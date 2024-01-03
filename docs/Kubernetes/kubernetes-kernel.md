---
tags:
  - k8s
share: "true"
title: Модули ядра Linux на ноде Kubernetes
---

Была задача мигрировать три ноды Kubernetes на ip-адреса из сервисной подсети. Эти ноды использовались для работы gitlab-runner и работали на виртуальных машинах (не железных серверах).

Последовательность:
- `kubectl cordon имя_ноды`
- на самих нодах `systemctl stop kubelet`
- дополнительно на нодах поставить пакет vmware tools:
  ```bash
  dnf install -y open-vm-tools
  systemctl enable --now vmtoolsd
  ```
- установить новый ip  (применяется после перезагрузки Linux или перезагрузки сетевого интерфейса):
```bash
nmcli con mod ens192 ipv4.address адрес/23 ipv4.gateway шлюз
```
- в веб-интерфейсе VMware vcenter перенести виртуальные машины трех нод в другой кластер и сервисную подсеть
- перегрузить (обновилось в том числе ядро Linux)
- вернуть в сервис: kubelet сам запустился после перезагрузки, осталось только `kubectl uncordon имя_ноды`
- дополнительно обновить helm чарт [gitlab-runner](https://gitlab.com/gitlab-org/charts/gitlab-runner), чтобы он заодно перерегистрировался в Gitlab с новым ip-адресом.

Казалось бы, всё заработало. Захожу в Gitlab, вроде раннер есть и даже запускаются джобы. Но у нас в части джобов используется сервис dind (docker inside docker) для сборки образов docker, которые потом запускаются в Kubernetes. И вот на этом шаге стала появляться ошибка:

```bash
Cannot connect to the Docker daemon at tcp://localhost:2375. Is the docker daemon running?
```

Анализ проблемы, копание в логах привело к тому, что надо запустить модули ядра iptable_nat (этот особо критичен), iptable_filter. Запустили через modprobe:

```bash
modprobe iptables_nat
modprobe iptables_filter
```

Но этот запуск работает только до следующей перезагрузки. Возник вопрос: а почему эти модули не загрузились автоматически после перезагрузки Linux, кто это всё настраивал? Есть подозрение, что настраивали, но НЕ ПРОВЕРИЛИ.

Итак, оба модуля были прописаны в */etc/modules* и по идее должны были загрузиться. Однако я нашла [документацию](https://wiki.archlinux.org/title/Kernel_module_(%D0%A0%D1%83%D1%81%D1%81%D0%BA%D0%B8%D0%B9))

> Сегодня все необходимые загрузки модулей делаются автоматически с помощью udev, поэтому если вам не нужно загружать какие-либо модули, не входящие в стандартное ядро, вам не придётся прописывать модули, требующиеся для загрузки в каком-либо конфигурационном файле. Однако, бывают случаи, когда вам необходимо загружать свой модуль в процессе загрузки или наоборот не загружать какой-то стандартный модуль, чтобы ваш компьютер правильно функционировал.

> Чтобы дополнительные модули ядра загружались автоматически в процессе загрузки, создаются статические списки в конфигурационных файлах в директории /etc/modules-load.d/. Каждый конфигурационный файл называется по схеме /etc/modules-load.d/program.conf. Эти файлы просто содержат список названий модулей ядра, которые необходимо грузить, разделённых переносом строки. Пустые строки и строки, в которых первым непробельным символом является # или ;, игнорируются.

Т.е. в новых версиях ядра файл */etc/modules*  не обрабатывается, а обрабатываются файлы из каталога */etc/modules-load.d/*. К сожалению, версия ядра Linux, начиная с которой это работает, не уточнена в документации. Я так и не нашла. Но ядро 4 мажорной версии стояло и ранее, и скорее всего просто после того как добавили модули в */etc/modules*, никто не перегружал Linux и не проверял работу.

Также можно самостоятельно узнать, откуда будут читаться файлы конфигурации: открыть файл */usr/lib/systemd/system/systemd-modules-load.service*

```ini title="/usr/lib/systemd/system/systemd-modules-load.service" hl_lines="8-12"
[Unit]
Description=Load Kernel Modules
Documentation=man:systemd-modules-load.service(8) man:modules-load.d(5)
DefaultDependencies=no
Conflicts=shutdown.target
Before=sysinit.target shutdown.target
ConditionCapability=CAP_SYS_MODULE
ConditionDirectoryNotEmpty=|/lib/modules-load.d             # (1)!
ConditionDirectoryNotEmpty=|/usr/lib/modules-load.d
ConditionDirectoryNotEmpty=|/usr/local/lib/modules-load.d
ConditionDirectoryNotEmpty=|/etc/modules-load.d
ConditionDirectoryNotEmpty=|/run/modules-load.d
ConditionKernelCommandLine=|modules-load
ConditionKernelCommandLine=|rd.modules-load

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/lib/systemd/systemd-modules-load
TimeoutSec=90s
```

1.   В строках, которые начинаются c `ConditionDirectoryNotEmpty`, перечислены все каталоги, из которых будут читаться файлы конфигурации.

B Ubuntu ещё могут быть модули, которые указаны в черном списке, обычно это в одном из файлов в каталоге /etc/modprobe.d/. Строка с черным списком выглядит так:
`blacklist имя_модуля`.
У нас в CentOS8 не было черного списка.

Решение: добавить файл `/etc/modules-load.d/iptable_nat.conf` с содержимым:

```bash
iptable_nat
iptable_filter
```

Обязательно перегрузить Linux для проверки. Проверить после перезагрузки:

```bash
# lsmod | grep iptable
iptable_filter         16384  1
iptable_nat            16384  1
ip_tables              28672  2 iptable_filter,iptable_nat
nf_nat                 45056  4 ipt_MASQUERADE,xt_nat,nft_chain_nat,iptable_nat
```