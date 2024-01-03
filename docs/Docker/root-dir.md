---
title: Изменение корневого каталога Docker
share: "true"
tags:
  - docker
---

Обычно каталог _/var/lib/docker_ разрастается и может неожиданно занять 100% места в корневом разделе, поставив под угрозу всю работу системы. Приходится перезапускать Linux, чтобы сервисы заработали. Поэтому рекомендуется вынести этот каталог на отдельный раздел на основном или дополнительном диске. Варианты того, как мы можем обезопасить себя:
- создать заранее отдельный раздел для */var/lib/docker* и монтировать его в */etc/fstab*.
- Переназначить каталог Docker: 
  - изменить каталог в файле конфигурации
  - создать ссылку  */var/lib/docker* на другой каталог

Как проверить текущий рабочий корневой каталог Docker:

```sh
$ sudo docker info | grep "Docker Root Dir"
 Docker Root Dir: /var/lib/docker
```

## Изменение каталога в файле конфигурации
В общем случае можно выделить целиком один раздел под _/var/lib/docker_. Тогда в fdisk, gdisk или gparted создается раздел на диске и форматируется. Затем настраивают монтирование раздела в каталог _/var/lib/docker_ непосредственно в файле _/etc/fstab_. Если такой возможности нет, некоторые используют символические ссылки или _bind mount_, но в случае с docker это может иметь негативные побочные явления при работе контейнеров, т.к. работа docker связана с ядром Linux. Поэтому при необходимости переноса нескольких каталогов от разных программ на другой диск все каталоги этих программ можем монтировать как *bind mount* или делать ссылки, а для docker сделаем исключение: изменим конфигурацию самого Docker. 

Например, есть диск _/dev/sdb_, который я разметила как логический том (Logical Volume), чтобы иметь возможность увеличить его размер в дальнейшем. Этот том монтирован в каталог _/data_ в файле _/etc/fstab_. Настройки описаны в соответствующем разделе (общие настройки серверов). Внутри каталога _/data_ есть разные каталоги, часть из которых монтируется с помощью _bind mount_, а на какие-то каталоги созданы символические ссылки. Создадим каталог docker внутри каталога _/data_.

```sh
mkdir /data/docker
```

Остановим работу docker.

```sh
systemctl stop docker
sudo systemctl stop docker.socket # для свежих Linux
sudo systemctl stop containerd    # для свежих Linux
```

Перенесем все содержимое _/var/lib/docker_ в новый каталог _/data/docker_ c сохранением прав:

```sh
rsync -a /var/lib/docker/* /data/docker
# или
cp -a /var/lib/docker /data/       # (1)! 
```

 1.  `-a` или `--archive` — аналог комбинации атрибутов `-dR --preserve=all`
     `--preserve[=ATTR_LIST]` — preserve the specified attributes (default: mode, ownership, timestamps), if possible additional attributes: context, links, xattr, all. Соответственно у нас копирование с сохранением всех указанных атрибутов (права, владелец, дата создания, контекст, ссылки, xattr)
     `-d` — аналог `--no-dereference --preserve=links`  (never follow symbolic links in SOURCE)
     `-R`, `-r`, `--recursive` — копировать каталоги рекурсивно

Сохранить бэкап каталога */var/lib/docker*:
```bash
mv -u /var/lib/docker /var/lib/docker.bak  # (1)!
```

1.   `-u`, `--update` —  перемещать только если файл источника свежее, чем назначения, или если файл назначения отсутствует.

**Для Docker  v17.05.0+ (АКТУАЛЬНО)**
Создать, если такого еще нет, или отредактировать файл */etc/docker/daemon.json*  Здесь всего одна строка, но могут содержаться и другие настройки:

```json title="/etc/docker/daemon.json"
{
  "data-root": "/data/docker",
  "storage-driver": "overlay2"
}
```

**Устаревший вариант для Docker <v17.05.0**
**dockerd** мог задавать путь к образам и контейнерам через флаг `--graph` (или `-g`) при запуске сервиса Docker, например: `-g=/var/lib/docker`. Поэтому нужно было заменить его в конфигурационном файле Docker, который задает большинство параметров фонового процесса. В Ubuntu файл расположен в `/etc/default/docker`, в CentOS — в `/etc/sysconfig/docker`.

=== "CentOS"

    ```bash title="/etc/sysconfig/docker"
    OPTIONS=-g="/store/software/docker" --selinux-enabled -H fd://
    ```

=== "Ubuntu"

    ```bash title="/etc/default/docker"
    OPTIONS=-g="/store/software/docker" -H fd://
    DOCKER_OPTS="-g /store/software/docker"
    ```

Запустим docker:
```bash
systemctl start docker
```

Проверим, что теперь именно каталог _/data/docker_ является корневым для docker. Ищем строку, которая начинается со слов **Docker Root Dir**. Или выводим переменную DockerRootDir.

```shellsession
$ docker info | grep "Docker Root Dir"
 Docker Root Dir: /data/docker

$ docker info -f '{{ .DockerRootDir}}'
/var/lib/containerd/docker
```

Теперь docker не будет забивать корневой раздел данными. При этом в каталоге _/data_ можно создавать папки и хранить данные, относящиеся не только к docker, но и другим сервисам.
## Изменение каталога через ссылку
Создание мягкой ссылки — самый быстрый и простой способ. Действия те же самые: остановить сервис, скопировать содержимое */var/lib/docker* в новый каталог, создать бэкап. Только вместо изменения файла конфигурации создать ссылку:

```bash
sudo ln -fs /data/docker /var/lib/docker
```

После запуска сервиса Docker каталог "Docker Root Dir" будет, как и прежде, */var/lib/docker*, но в реальности запись будет идти в новое место.

После завершения миграции любым из способов и проверки работоспособности Docker удалить бэкап:

```bash
rm -rf /var/lib/docker.bak
```

