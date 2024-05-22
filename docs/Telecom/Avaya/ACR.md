---
title: "ACR: запись разговоров"
share: "true"
---
ACR — Avaya Contact Recorder, устанавливается на Linux.
## Новое начиная с версии ACR 15.1
On Demand и Meeting Recording

До релиза 15.1FP1 оба режима записи использовали DMCC feature (известен как Call Information Services) для тегирования записей в зависимости от участников разговора. У этого интерфейса был ряд ограничений (например можно было идентифицировать только одного оператора, который разговаривал). Интерфейс заменен TSAPI мониторингом, который используется для потоковой записи Bulk Recording.

As with Bulk recording, however, to tag calls with the phone number of the original (especially external) parties on the call, TSAPI must have visibility of the call when it is initially handled. Note that this may require you to add additional stations and/or VDNs to your configuration so that TSAPI observers can be placed on them. These settings are on the General Setup administration page for your Communication Manager. The stations that take or make the calls that are to be recorded and/or the VDN(s) these go through should be added.

Архивы на DVD & Blu-ray устарели. Теперь бэкапы сохраняются на файловый сервер (NAS, mounted drive или ссылка symlink, SFTP или EMC).

Patch Tool

The previous, manual, patch application process has been replaced with an automated tool that supports rollback and patch prerequisites. Ensure that you are familiar with this tool. The release notes provide instructions on its use. These must be followed exactly.

**Changes since ACR 15.1FP2**

Требования к серверу:
- Windows 2012 или Windows 2012 R2
- RHEL 7 Update 2 или выше

Требования к клиентскому ПО:
- Windows или Mac OS X с IE11, Chrome, Firefox или Safari
- Может понадобиться Java Runtime Environment 1.8 с последним обновлением

Поддерживаются:
- Avaya Aura Communication Manager 7
- Avaya Aura Communication Manager 6.3.8 and Avaya Enablement Server 6.3.3 running the latest loads
- Communication Server 1000 7.6.6 (SP) running the latest loads
- Avaya Aura Contact Center 6.4.2 (FP) and Avaya Aura Contact Center 7.0 running the latest loads
- VMware ESXi versions 5.0, 5.1, 5.5, и 6.0 (also referred to as VSphere 5.0, 5.1, 5.5 и 6.0)

Что нового в FP2
- Использует TLSv1.2 для связи с AES.
- Обновленная утилита Kickstart для улучшения безопасности.
- Улучшены настройки дублирования серверов High Availability и производительность.
- Интерфейс внешнего управления WebX API через HTTP/HTTPS.

Если в системе только один CM или один CS1000, смотреть Appendix Dруководства по установке ACR.
## Установка RHEL 7. Создание kickstart скрипта
[Скачать](%3Chttps://developers.redhat.com/products/rhel/get-started/%3E)
Залогиниться под аккаунтом Facebook.

При установке на новом железе сначала можно не использовать kickstart скрипт, а начать установку RHEL стандартно, чтобы определить точные имена устройств:
- С экрана Network и Hostname записать названия сетевых контроллеров, например **ens192**.
- С экрана Installation Destination записать имя логических дисков, например **sda**.

![](19-test-install_check-disk.png)

![](19-test-install_check-network.png)

После получения данных отменить процесс установки (Quit).

**Подготовка скрипта kickstart для автоматизированной установки RHEL:**

Скопировать с образа **acr-151fp2-linux.iso** файл **ks3.jar**, который позволяет сгенерировать подходящий для ACR kickstart файл. На компьютере(Windows, Linux, Mac OS) должна быть версия Java Version 7 или выше. Этот компьютер станет сервером «Kickstart server» на время установкиRHEL.

Открыть порт 8080 в файрволе Kickstart сервера. Новый ACR должен иметь доступ к серверу по сети (либо настроена маршрутизация, либо они в одной подсети).

Двойным щелчком запускаем файл **ks3.jar**. Выбираем соответствующую версию Red Hat. Заполняем форму:
- Клавиатурная раскладка **Keyboard** — us
- Часовой пояс **Timezone** — Moscow/Europe
- Корпоративный сервер NTP (IP адрес или полное доменное имя). Если ничего не указано, по умолчанию останутся публичные NTP сервера. Это важно для правильного времени записей разговоров и одинакового времени в CMS и ACR.
- IP адрес и маска первой сетевой карты. IP-адрес роутера из той же подсети, что и адрес сетевой карты. **NIC** — ens192. **Netmask** — 255.255.255.192.
- IP адреc сервера DNS.
- hostname, предпочтительнее полное доменное имя (например acrmaster.bigcorp.com).
- Опционально задать пароли для пользователей root и witness. Если забудешь, поддержку Avaya не получить без переустановки ACR. **Root password (optional)** — свой. **Witness password (optional**) — свой.
- Галочку на чекбоксе **CRS**, если будет установлен отдельный сервер для проигрывания. Нам не нужно, т.к. всё устанавливаем на одной виртуальной машине.
- Нажать **Generate**.

Посмотреть и проверить скрипт, чтобы были правильно внесены адреса. В нашем случае будем вносить дополнительные правки, которые описаны в расширенных настройках.

Запустить сервер http сервер нажав **Run HTTP Server**. Теперь можно проверить доступность скрипта из браузера подсоединившись с другого компьютера по адресу `http://<IP-адрес_KickstartServer>:8080/ks.cfg`. Если хотим сохранить файл, жмем Save As.

![](kickstart.png)

**Расширенные настройки Expert kickstart (опционально)**

Выбрать Allow Edits box и сделать изменения. Расширенные настройки указаны ниже.

*NIC bonding ( только RHEL 7), не нужно*
1. Раскомментировать строку **un-bonded network** и **bonded network alternative**.
2. Изменить имена бэкапных сетевых карт чтобы соответствовать железу.
3. После установки проверить файлы в */etc/sysconfig/network-scripts*.
4. Удалить все что ссылается на ens0 или ens1. Оставить ifcfg-bond\*файлы. Пример:

```bash title="ks.cfg"
network --device=bond0 --bondslaves=ens1,ens2 --bootproto=static --ip=10.10.11.44 --netmask=255.255.255.0 --gateway=10.10.11.1 --nameserver=10.10.11.11 --hostname=acrmaster.bigcorp.com
```

*Вторая сетевая карта в другой подсети, не нужно*

Раскомментировать prototype second NIC line, вручную изменить IP адрес, маску, имя. Пример

```bash title="ks.cfg"
network --device=ens3 --bootproto=static --ip=10.10.12.123 --netmask=255.255.255.0
```


*Два и более дисков (designated disks), не нужно*
You must designate a disk for each partition. The typical way to do this is to create */calls* on the largest disk and the other partitions on thesmaller one. Allow /var/lib/pgsql to grow to fill the smaller disk. Addthe on disk suffix to each partition

Пример: (здесь размер каталога */opt/witness* увеличен с 10000 Мб до 33000 Мб для разрешения больших файлов логов уровня дебаг):

```bash title="ks.cfg"
part /boot --fstype=ext4 --size=200 --ondisk=sda
part / --fstype=ext4 --size=10000 --ondisk=sda
part /opt/witness --fstype=ext4 --size=33000 --ondisk=sda
part /var/lib/pgsql --fstype=ext4 --size=50000 --grow --ondisk=sda
part swap --recommended --ondisk=sda
part /calls --fstype=ext4 --size=1000 --grow --ondisk=sdb
```

*Один и более дисков (volume group), не нужно*

Red Hat Enterprise Linux can join multiple disks into one large volume. This allows greater flexibility going forward. To create a logicalvolume group, comment out one set of part lines and uncomment theothers.

Allow one part pv.xx line per physical disk. Set the ondisk to the device name for each one.

Make sure that each pv.xx appears on the volgroup line. This joins them together into one large volume group

Пример: (note how the default sizes of */opt/witness* and */var/lib/pgsqlhave* been increased to allow for larger logs and a larger database):

```bash title="ks.cfg"
part /boot --fstype=ext4 --size=200 --ondisk=sda
part pv.01 --size=1 --grow --ondisk=sda
part pv.02 --size=1 --grow --ondisk=sdb
volgroup acrvg pv.01 pv.02
logvol swap --recommended --vgname=acrvg --name=swap
logvol / --fstype=ext4 --size=10000 --vgname=acrvg --name=root
logvol /opt/witness --fstype=ext4 --size=30000 --vgname=acrvg --name=witness
logvol /var/lib/pgsql --fstype=ext4 --size=100000 --vgname=acrvg --name=postgres
logvol /calls --fstype=ext4 --size=1000 --grow --vgname=acrvg --name=calls
```


**Удаляем графическую оболочку из установки (делаем)**
По умолчанию kickstart устанавливает рабочий стол Gnome. Если рабочий стол и X windows система не нужны, удалить пакеты, которые к ним относятся, оставив минимальный необходимый набор. Начинающиеся с дефиса "-" строки говорят kickstart скрипту НЕ УСТАНАВЛИВАТЬ пакет. Это трогать нельзя. Пример:

```bash hl_lines="4-7" title="ks.cfg"
%packages
@ base
@ core
@ x11  # (1)!
@ fonts
@ internet-browser
@ gnome-desktop
perl
mkisofs
perl-libwww-perl
chrony или ntp в зависимости от версии (ничего не менять)
systemd-libs
tigervnc-server
-redhat-lsb
-ash
-aspell
-cups
-cups-libs
-finger
-irda-utils
-lftp
-mtr
-nano
-nc
-pax
-pdksh
-rdate
-redhat-config-network-tui
-rp-pppoe
-rsh
-jfsutils
-jwhois
-setuptool
-sendmail
-sharutils
-stunnel
-tftp
-wireless-tools
-rdist
-openoffice-libs
-cyrus-sasl-gssapi
-cyrus-sasl-plain
-wvdial
-dvd+rw-tools
-gnome-boxes
%end
%post
```

1. Удаляем выделенные строки

![](kickstart2.png)

**Уменьшаем требуемое место на диске (делаем)**

Значения по умолчанию приведены ниже:

```bash title="ks.cfg"
# Edit partition sizes as desired to meet the minimums in the installation manual
part /boot --fstype=ext4 --size=200
part / --fstype=ext4 --size=10000
part /opt/witness --fstype=ext4 --size=10000
part /var/lib/pgsql --fstype=ext4 --size=10000
part  swap --recommended
part /calls --fstype=ext4 --size=1000 --grow
```

Т.к. у нас минимальные требования, настроить скрипт установки kickstartсогласно таблице:

| Точка монтированиz | Использование                 | Размер                                                                                                               |
| ------------------ | ----------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| /boot              | Bootstrap                     | 200 Мб                                                                                                               |
| /                  | Операционная система Linux    | Как минимум 5 Гб                                                                                                     |
| Swap               | Виртуальная память            | Зависит от количества оператиной памяти                                                                              |
| /opt/witness       | Avaya Contacr Recorder и логи | Минимум 10Гб, рекомендуется 100Гб, т.т. логи уровня дебаг могут использовать несколько Гб в день для большой системы |
| /var/lib/pgsql     | База данных PostgreSQL        | Примерно 2 Кб на один разговор. Умножаем кол-во разговоров recordings/day x кол-во дней, которое хотим хранить.      |
| /calls             | Аудиофайоы записей разговоров | Примерно 7.2MB (G.729 стерео) или 3.6MB (G.729 моно) на 1 час записи. Оставшееся место на диске.                     |

```bash title="ks.cfg"
# Edit partition sizes as desired to meet the minimums in the installation manual
part /boot --fstype=ext4 --size=200
part / --fstype=ext4 --size=5500                 # поменяли size
part /opt/witness --fstype=ext4 --size=10000
part /var/lib/pgsql --fstype=ext4 --size=10000
part  swap --recommended
part /calls --fstype=ext4 --size=1000 --grow
```

## Подготовка kickstart, если нет доступа по http

**Способ 1. В образ iso добавить ks.cfg и другие необходимые пакеты**

Минусы: при изменении файла *ks.cfg* надо целиком пересобрать iso, установка может не состояться из-за неправильной пересборки RedHat iso в **UltraISO**:

```bash
Anaconda exception report
No such group: core
https://bugzilla.redhat.com/show_bug.cgi?id=981902
```

Возможно при переcборке образа из Linux CLI будет работать, непроверялось.

Строка со ссылкой на kickstart файл во время установки Linux:

```bash
ks=cdrom:/ks.cfg
```

**Способ 2. Лучше. Создание виртуальной загрузочной дискеты с kickstart файлом.**
*Преимущество*: точно работает, никак не меняем скачанный установочный диск с RHEL, можно создавать заново при изменении ks.cfg без необходимости менять и заливать iso целиком.

Ставим WinImage, можно пробную версию. Жмем New (левая иконка с листом)→Ок (оставляем размер по умолчанию).

![](kickstart-to-flp_WinImage0.png)

Добавляем на дискету kickstart файл *ks.cfg*: нажать клавишу **Insert** или кнопку Inject и выбрать файл.

![](kickstart-to-flp_WinImage1.png)

Вот что получается:

![](kickstart-to-flp_WinImage2.png)

Жмем на иконку дискетки (сохранить), выбираем тип файла Virtual Floppy Image (`*.vfd`, `*.flp`) и задаем название `kickstart.flp`

![](kickstart-to-flp_WinImage3.png)

Получившийся образ дискеты закачиваем в Datastore.

![](kickstart-to-flp_vCenter.png)

В свойствах виртуальной машины **rhel-server-7.4-x.86_64-dvd.iso** (неизмененный файл, скачан с сайта RHEL) указываем для CDROM, в свойствах Floppy указываем **kickstart.flp**. Галочки Connect On Poweron.

![](vm-cdrom-floppy.png)

Т.к. по умолчанию система будет пытаться загрузиться не с CDROM, а сFloppy, надо в *VM Option→ Boot Options* включить принудительную загрузку BIOS (действует только во время следующей загрузки), чтобы опустить Floppy ниже всех устройств Hard, CDROM).

![](vm-force-BIOS.png)

Как сослаться на дискету. Во время установки Linux сразу жмем Tab, после строки, которая оканчивается на quiet, ввести пробел и ссылку на расположение файла kickstart:

```bash
ks=hd:fd0:/ks.cfg
```

Примечание. Не сработал такой способ указания флоппи:

```bash
ks=floppy
```

**Создание виртуальной дискеты с помощью BFI (Build Floppy Image)**

Программа **bfi.exe** больше не поддерживается, поэтому можно скачать её с моего сайта. В примере мы положили файл *ks.cfg* в каталог `С:\temp\floppy` — это будущее содержимое виртуальной дискеты. Вкаталоге может быть и несколько файлов.

```bash
bfi -f=c:\temp\kickstart.flp c:\temp\floppy # (1)!
```
1.  Опция `-f` указывает имя образа виртуальной дискеты.
## Установка RHEL 7 с помощью kickstart скрипта
После проверки доступности kickstart файла из браузера:
1. Загрузить сервер дистрибутив RHEL 7 не ниже 2 апдейта согласно Release Notes.
2. На экране инициализации выбрать Install Red Hat.
3. Нажать **TAB** для попадания в командную строку по умолчанию.
4. **Удалить** последнее слово **quiet**, добавить пробел и параметр в конце строки:

```bash
ks=http://<ip-адрес_KickstartServer>:8080/ks.cfg
```

Адрес моего компьютера 10.0.45.51. Получаем:

```bash
ks=http://10.0.45.51:8080/ks.cfg
```

![](RHEL-start-kickstart.png)

![](RHEL-start-kickstart3.png)

Возможно, если не считывается kickstart по http, поможет явное прописывание в этой же строке ip-адреса, маски и шлюза:

```bash
ks=http://xxx.xxx.xxx.xxx/ks.cfg ksdevice=eth0 ip=xxx.xxx.xxx.xxx netmask=255.255.255.0 gateway=xxx.xxx.xxx.xxx noipv6
```

Если не работает доступ с виртуальной машины на мой компьютер, есть несколько вариантов: предварительно подготовить установочный диск RHEL **rhel-server-7.4-x86_64-dvd.iso,** добавив в него папку witness, куда и закинуть скрипт. И только после изменений закачать этот `.iso` на виртуальную машину. Тогда при установке RHEL строка для kickstart будет такой:

```ini
ks=cdrom:/witness/ks.cfg # (1)! 
ks=hd:fd0:/ks.cfg ks.cfg # (2)! 
```

1. `ks.cfg` на CDROM (внутри образа `.iso`)
2. на виртуальной дискете (образ `.flp`)

**Вариант 2.** Использовать уже установленный AES в качестве http сервера. В WinSCP зайти по SFTP и закинуть файл *ks.cfg* в домашний каталог */home/cust*. Оттуда копируем в каталог веб-сервера */var/www/html*

```bash
cp /home/cust/ks.cfg /var/www/html
```

Остальные файлы кстати являются ссылками на другой каталог

```bash
[root@example-ucaes cust]# ls -la /var/www/html
total 16
drwxr-xr-x 2 root root 4096 Mar 28 22:22 .
drwxr-xr-x 4 root root   31 Nov 10 15:10 ..
lrwxrwxrwx 1 root root   46 Nov 10 15:13 avaya_logo.gif -> /usr/share/tomcat5/webapps/ROOT/avaya_logo.gif
lrwxrwxrwx 1 root root   47 Nov 10 15:13 disclaimer.html -> /usr/share/tomcat5/webapps/ROOT/disclaimer.html
lrwxrwxrwx 1 root root   46 Nov 10 15:13 error_page.jsp -> /usr/share/tomcat5/webapps/ROOT/error_page.jsp
lrwxrwxrwx 1 root root   43 Nov 10 15:13 favicon.ico -> /usr/share/tomcat5/webapps/ROOT/favicon.ico
lrwxrwxrwx 1 root root   48 Nov 10 15:13 hd_squares_3.gif -> /usr/share/tomcat5/webapps/ROOT/hd_squares_3.gif
-rwxr-xr-x 1 root root  163 Nov 10 15:13 index.html
lrwxrwxrwx 1 root root   41 Nov 10 15:13 index.jsp -> /usr/share/tomcat5/webapps/ROOT/index.jsp
lrwxrwxrwx 1 root root   43 Nov 10 15:13 index.shtml -> /usr/share/tomcat5/webapps/ROOT/index.shtml
-rw-r--r-- 1 root root 4794 Mar 28 22:27 ks.cfg
lrwxrwxrwx 1 root root   48 Nov 10 15:13 logo-smaller.gif -> /usr/share/tomcat5/webapps/ROOT/logo-smaller.gif
```

При установке добавляем

```bash
ks=https://172.адрес/ks.cfg
```

![](RHEL-start-kickstart-from-AES.png)

5\. Если у нас более одной сетевой карты, добавить ещё один пробел и параметр:

```bash
ksdevice=xxxx  # (1)!
```

1.  xxxx — имя сетевой карты, которая имеет доступ к Kickstart server (one of the ones you noted earlier and that you have connected to the network):

6. Если нет сервера DHCP, также добавить три параметра с пробелом перед каждым, заменяя адреса на те же самые, которые вводили при генерировании kickstart скрипта.

```bash
ip=x.x.x.x
netmask=y.y.y.y
gateway=z.z.z.z
```

7. Нажать ENTER.
8. Ждать окончания установки. Возможно нужно будет подтвердить полную очистку диска.

## Установка ACR 15.1 FP2, PostgreSQL, Java и Tomcat на предварительно установленную RHEL
Пароли по умолчанию: root/ContactStore, root

В примере пользователm **witness**. Заходим по SSH (PuTTY) как пользователь witness. Переключить на root командой `su`. Как и в AES, нельзя сразу залогиниться как root.
Второй вариант — после входа под witness перейти в режим суперпользователя `sudo -s` и снова ввести пароль от witness.

Под пользователем **witness**, скопировать необходимые файлы с dvd в каталог */home/witness* сервера.

![](WinSCP-witness-login.png)

![](WinSCP-witness-files.png)

Проверяем из PuTTY:

```bash
[witness@example-ucacr1 ~]$ ls -la
total 170776
drwx------. 4 witness witness     4096 Mar 30 18:11 .
drwxr-xr-x. 3 root    root        4096 Mar 30 14:13 ..
-rw-rw-r--  1 witness witness 88343268 Sep  7  2017 acr-15.1fp2-0010.apa
-rw-rw-r--  1 witness witness 14152690 Feb 22 18:19 acr-15.1fp2-0019.apa
-rw-rw-r--  1 witness witness  4121548 Sep  7  2017 acr-15.1fp2-3.rhel7.x86_64.rpm
-rw-r--r--. 1 witness witness       18 Mar  7  2017 .bash_logout
-rw-r--r--. 1 witness witness      193 Mar  7  2017 .bash_profile
-rw-r--r--. 1 witness witness      231 Mar  7  2017 .bashrc
drwxrwxr-x  3 witness witness     4096 Mar 30 18:11 .cache
drwxrwxr-x  3 witness witness     4096 Mar 30 18:11 .config
-rw-rw-r--  1 witness witness 54576635 Jan 19 14:56 jdk1.8_161.run
-rw-rw-r--  1 witness witness  6232338 Jan 19 18:20 postgresql966_rh7.run
-rw-rw-r--  1 witness witness  7402536 Jan 19 15:13 tomcat8524.run
```

Названия этих файлов могут отличаться в новой версии или при обновлении.
- **acr-151fp2-linux.iso** — основной диск для Linux с пакетом ACR15.2FP2, установочными пакетами PostgreSQL, Java, Tomcat, документацией, MIB файлами для SNMP мониторинга, некоторые утилиты:
    - **/utils —** дополнительные утилиты: acr-csapi Java API Toolkit sdk, [ValidateFP](file:///C:\\Users\\ATOROP~1\\AppData\\Local\\Temp\\BNZ.5aba36b139acb4fb\\utils\\ValidateFP.zip) audio WAV file finger print validation utility, [WitsBSUserCredentials](file:///C:\\Users\\ATOROP~1\\AppData\\Local\\Temp\\BNZ.5aba36b139acb4fb\\utils\\WitsBSUserCredentials.zip) утилита для шифрования логинов и паролей, [WSSChangePassword](file:///C:\\Users\\ATOROP~1\\AppData\\Local\\Temp\\BNZ.5aba36b139acb4fb\\utils\\WSSChangePassword.zip) утилита для подготовки ключей безопасности для записи экранов операторов.
    - **/mibs** — Simple Network Management Protocol (SNMP) Management Information Base (MIB) файлы для мониторинга.
- **15.1fp2_ACR_Security_and_3rd_Party_Update_Kit_15.1.2.0003.zip** — обновления PostrgeSQL, Java и Tomcat
- **acr-15.1fp2-0019.zip** — новыq патч ACR версии 0019.

![](SupportAvaya_DownloadACR2.png)

Что в итоге ставим (из того что было на диске остался только rpm пакетACR15.1FP2 и патч 0010):
- **acr-15.1fp2-3.rhel7.x86_64.rpm** — пакет самого Avaya Contact Recorder.
- **postgresql963_rh7.run** — самоизвлекаемый установщик базы данных PostgreSQL 9.6. Вместо него новый **postgresql966_rh7.run**.
- **jdk1.8_131.run** — самоизвлекаемый установщик Java. Вместо него новый **jdk1.8_161.run**
- **tomcat8515.run** — самоизвлекаемый файл Tomcat. Вместо него новый **tomcat8524.run**
- **/patch/acr-15.1fp2-0010.apa** — патч версии 0010. Сначала установить его, т.к. это полный патч! Затем ставим новый патч **acr-15.1fp2-0019.apa**, скачанный отдельным архивом.

**Установка основных компонентов**.

Зайти под **root** и установить ACR и базу данных:

```bash
rpm acr-15.1fp2-3.rhel7.x86_64.rpm -Uvh
sh ./postgresql966_rh7.run
```

Вернуться в пользователя **witness** и установить Java (jdk) и tomcat:

```bash
sh ./jdk1.8_161.run
sh ./tomcat8524.run
```

Если забыли и установили эти два последние пакета под root, поменяем владельца всего что есть в каталоге */opt/witness* на **witness** (рекурсивно, т.е. все каталоги и подкаталоги). На самом деле там уже все папки принадлежат witness кроме того что мы забыли. Если этого не сделать, будет проблема при установке патча (подробнее описывается ниже)

```bash
chown -R witness. /opt/witness
```


Пример моей установки:

```bash
[root@example-ucacr1 witness]# rpm acr-15.1fp2-3.rhel7.x86_64.rpm -Uvh
Preparing...                          ################################# [100%]
Updating / installing...
   1:acr-15.1fp2-3.rhel7              ################################# [100%]
Created symlink from /etc/systemd/system/multi-user.target.wants/acr.service to /usr/lib/systemd/system/acr.service.

root@example-ucacr1 witness]# sh ./postgresql966_rh7.run
Verifying archive integrity...  100%   All good.
Uncompressing ACR PostgreSQL for RH7 version 9.6.6  100%
warning: postgresql96-libs-9.6.6-1PGDG.rhel7.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID 442df0f8: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:postgresql96-libs-9.6.6-1PGDG.rhe################################# [ 33%]
   2:postgresql96-9.6.6-1PGDG.rhel7   ################################# [ 67%]
   3:postgresql96-server-9.6.6-1PGDG.r################################# [100%]
Initializing database ... OK

CREATE ROLE
Created symlink from /etc/systemd/system/multi-user.target.wants/postgresql-9.6.service to /usr/lib/systemd/system/postgresql-9.6.service.

[root@example-ucacr1 witness]$ sh ./jdk1.8_161.run
Verifying archive integrity...  100%   All good.
Uncompressing Server JDK 1.8_161  100%

[root@example-ucacr1 witness]$ sh ./tomcat8524.run
Verifying archive integrity...  100%   All good.
Uncompressing Tomcat for ACR version 8.5.24  100%
```

Дополнительно при необходимости интеграции с Workforce Optimizationи/или Quality Monitoring развернуть Java unlimited strength policyфайлы.

Каталог размещения вспомогательной утилиты Patch Tool Utility для установки патча:
- Linux — `/opt/witness/patches`
- Windows — папка установки `\Avaya\ACR151FP2\patches`

Утилита проверяет текущего пользователя, проверит что сервис ACRостановлен, создаст патч для возврата предыдущего состояния (RollbackPatch), применит новый патч и удалит рабочий каталог.

Обычно можно применить любой патч, версия которого выше установленного.В редких случаяx предварительно надо установить патч более ранней версии, тогда утилита выдаст предупреждение, и нужно будет скачать и установить более старый патч.

**Проверка установленных версий PostgreSQL, Tomcat, Java**

Текущая версия PostgreSQL:

```bash
$ su -l postgres -c "/usr/pgsql-9.6/bin/psql -V"
psql (PostgreSQL) 9.6.6
```

Текущая версия Java и Tomcat:

```bash
$ ls /opt/witness/packages
apache-tomcat-8.5.24 jdk1.8.0_161
```

**Переустановка PostgreSQL**

```bash
systemctl stop acr.service              # Остановить сервис записи разговоров
systemctl stop postgresql-9.6.service   # Остановить сервис базы данных PostgreSQL
sh ./postgresql96<номер_версии>_rh7.run # Обновить версию PostgreSQL
systemctl start acr.service             # PostgreSQL запустится автоматически. Осталось запустить запись разговоров
```

**Переустановка Java**

Для пользователей WFO или других функций, требующих установить опции шифрования неограниченной силы (описано в Installing Unlimited Strength Encryption разделе, appendix F руководства по установке PIA—Programming, Installation and Administration), требуется обновить процедуру после обновления версии Java.

Для пользователей WFO где настроен SSL, требуется заново импортировать сертификат безопасности после обновления Java, иначе EMA не сможетзагрузиться. Процедура описана в 'WFO Security/SSL Configuration' разделе руководства «ACR Integration To WFO Guide» (нам не нужно).

```bash
su -c "systemctl stop acr.service"   # Остановить сервис записи
### sh ./jdk1.8_nnn.run              # Установить новую версия Java (заменить nnn на номер текущей версии)
su -c "systemctl start acr.service"  # Запустить сервис записи
```

**Переустановка Tomcat**

```bash
su -c "systemctl stop acr.service"   # Выключить сервис записи разговоров
sh ./tomcatnnn.run                   # Установить новую версию Tomcat (заменить nnn на номер текущей версии)
su -c "systemctl start acr.service"  # Запустить сервис записи
```
## Системные файлы для бэкапа

Обязательно скопировать после установки */etc/sysconfig/network-scripts/ifcfg-ens192*
Мак-адрес надо скопировать, если случайно изменится потом.

```ini
# Generated by parse-kickstart
UUID="2c0e8a7e-73d2-4817-9cf5-example"
DNS2="первый"
DNS1="второй"
IPADDR="ip-адрес"
GATEWAY="шлюз"
NETMASK="255.255.255.192"
BOOTPROTO="static"
DEVICE="ens192"
ONBOOT="yes"
IPV6INIT="yes"
```

Опционально: проверка связи с Avaya Proactive Contact server до ACR. Убедиться, что есть запись ESTABLISHED между Avaya Proactive Contact Event Server и Avaya Contact Recorder.

```bash
$ netstat | grep enserver
tcp 0 0 apc:enserver_ssl wfo-acr:46336 ESTABLISHED
tcp 0 0 apc:enserver_ssl apc:29482 ESTABLISHED
tcp 0 0 apc:29482 apc:enserver_ssl ESTABLISHED
```
## Установка патча 0010, затем 0019
Патчи кумулятивные, т.е. патч 0019 содержит все обновления патчей 0000-0018. Но не совсем так. При установке патча 0019 предварительно надо поставить **полный патч 0010**, входящий в состав **acr-151fp2-linux.iso**, при установке патча 0023, который я ставила через два месяца после установки ACR, необходим патч 0022. Если не установлен полный патч, ничего страшного — появится сообщение, что его надо установить.

Скопировать патч с расширением .apa на сервер ACR сервер (виртуалку) поSFTP под пользователем witness через WinSCP во временный каталог /tmp(лично я скачивала в домашний каталог /home/witness).

Залогиниться в PuTTY под пользователем witness, остановить ACR сервис, будет запрошен ввод пароля от root:

```bash
systemctl stop acr
cd /opt/witness/patches  # Перейти в каталог патчей
../jdk8/bin/java -cp patchtool.jar Patcher  # Проверить установленные ранее патчи утилитой Patch Tool Utility.
../jdk8/bin/java -cp patchtool.jar Patcher/tmp/acr-15.1fp2-НомерПатча.apa  # Применить новые патчи (от пользователя witness)
```

Пример установки нового патча 0023 (ранее был установлен 0019).Появилось предупреждение, что нужен патч 0022.

```bash
[witness@example-ucacr1 ~]$ cd /opt/witness/patches
[witness@example-ucacr1 patches]$ ls
baseline.json  patches.db  patchtool.jar  rollback
[witness@example-ucacr1 patches]$ systemctl stop acr
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ===
Authentication is required to manage system services or units.
Authenticating as: root
Password:
==== AUTHENTICATION COMPLETE ===
[witness@example-ucacr1 patches]$ pwd
/opt/witness/patches
[witness@example-ucacr1 patches]$ ../jdk8/bin/java -cp patchtool.jar Patcher
The current baseline is 15.1fp2
The current patch is 0019
The following patches are installed:
Patch 0010 installed on Fri Mar 30 19:02:54 MSK 2018
Patch 0019 installed on Fri Mar 30 19:14:58 MSK 2018

$ ../jdk8/bin/java -cp patchtool.jar Patcher /home/witness/acr-15.1fp2-0023.apa
Patch 0022 is required before this patch can be applied
```

Скачиваем патч 0022 с сайта Avaya и копируем на сервер. Устанавливаем.

```bash
[witness@example-ucacr1 patches]$ ../jdk8/bin/java -cp patchtool.jar Patcher /home/witness/acr-15.1fp2-0022.apa
The changes since the last patch are:

0020
   ESR 4984362: Avoid EventLock problem when AACC Agent state changes.
   ESR 4984322: Linking and unlinking calls safeRefreshRels explicitly rather than rely on checkObserver().
   ESR 4987448: Configure maximum number of envelopes to add to an MDL job using property setting mdl.maxenvsperjob with a default of 20.
   ESR 4986686: Catch error that led to multiple uid files created in existing archive folder.
   ESR 4827490: Correct boundary condition matching INum against session's inums and subsequent changes from 15.1FP1 1022-3.
   Correct data type of variable kmsSetting from Setting to SettingBoolean to avoid unexpected values.
   QC 209265: Added null check around the session about to prevent exception in the wrapper log.
   ESR 4991916: DMCC Port register/unregister events not being sent.
   Log http response codes (other than 200) from MDL at WARN level.
   ESR 4993596: EQC Device_not_found - Try to match station if agent not logged-on.
0021
   Treat missing or empty parameter as OperatorType.NONE rather than an NFE (Validation in S&R API for example).
   ESR 4993472: Correctly escape special characters when writing to XML in Session for CTI info.
   ESR 4999002: Store and use the kerberos username as it is required in AES key creation.
   ESR 4991512: Attempt to get a lock on the XML when archiving to avoid conflicts with post call tagging.
   ESR 5000566: Avoid exception in wrapper log if delayed deletion call is last one in its session.
   ESR 5000250: Resolve spurious second screen recording (triggered by DPA command) which runs indefinitely and prevents contact closure and push to WFO.
0022
   QC 209264: Added null check for inum to prevent exception in the wrapper log.
   Re-queue calls for archive: correct oldest unarchived call age correctly, and make batch size configurable using archive.maxautocorrect calls (default 100).
   ESR 4990004: Replace SecureBlackBox sFTP libraries with SSHJ Open Source libraries.
This patch removes the following files:
lib/cscmmain.jar
lib/cscmres.jar
webapps/ROOT.war
lib/SecureBlackbox.Base.jar
lib/SecureBlackbox.SFTPClient.jar
lib/SecureBlackbox.SFTPCommon.jar
lib/SecureBlackbox.SSHClient.jar
lib/SecureBlackbox.SSHCommon.jar
and adds the following files:
lib/cscmmain.jar
lib/cscmres.jar
webapps/ROOT.war
lib/eddsa.jar
lib/slf4j-api.jar
lib/bcprov-jdk15on.jar
lib/sshj.jar
deleting lib/cscmmain.jar
deleting lib/cscmres.jar
deleting webapps/ROOT.war
deleting lib/SecureBlackbox.Base.jar
deleting lib/SecureBlackbox.SFTPClient.jar
deleting lib/SecureBlackbox.SFTPCommon.jar
deleting lib/SecureBlackbox.SSHClient.jar
deleting lib/SecureBlackbox.SSHCommon.jar
expanding lib/cscmmain.jar
expanding lib/cscmres.jar
expanding webapps/ROOT.war
expanding lib/eddsa.jar
expanding lib/slf4j-api.jar
expanding lib/bcprov-jdk15on.jar
expanding lib/sshj.jar
success
```

Теперь применяем патч 0023.

```bash
[witness@example-ucacr1 patches]$ ../jdk8/bin/java -cp patchtool.jar Patcher /home/witness/acr-15.1fp2-0023.apa
The changes since the last patch are:

0023
   ESR 4990004: Use the SSHJ random to speed up SFTP session creation.
   Correct oldest unarchived call query to return archive.maxautocorrect + 1 calls.
   ESR 4990004: Replace SecureBlackBox sFTP libraries with SSHJ Open Source libraries.
   ESR 5009324: Delete very short on-demand calls if there is no cti information.
   ESR 5011070: Remove beacon softphone owner from job queue on premature 'Media Stopped' event.
This patch removes the following files:
lib/cscmmain.jar
lib/cscmres.jar
webapps/ROOT.war
and adds the following files:
lib/cscmmain.jar
lib/cscmres.jar
webapps/ROOT.war
deleting lib/cscmmain.jar
deleting lib/cscmres.jar
deleting webapps/ROOT.war
expanding lib/cscmmain.jar
expanding lib/cscmres.jar
expanding webapps/ROOT.war
success
```

**Проблема во время установки, если jdk и tomcat случайно были установлены от лица root**.

Выполнять установку патча нужно под пользователем wintess, но я получала сообщение, что нет доступа:

```bash
[witness@example-ucacr1 patches]$ ../jdk8/bin/java -cp patchtool.jar Patcher /home/witness/acr-15.1fp2-0010.apa
-bash: ../jdk8/bin/java: Permission denied
```

Если же пробуем поставить под root, то выходит предупреждение, что патчер должен выполняться от лица witness.

```bash
[root@example-ucacr1 patches]# ../jdk8/bin/java -cp patchtool.jar Patcher /home/witness/acr-15.1fp2-0010.apa
Incorrect user: the patcher must be run as witness (Linux)
```

Причина проблемы: установка jdk и tomcat выполнялась под root-ом, поэтому права на эти каталоги принадлежали root (обратить внимание на строки с root). Исправляется сменой владельца.

```bash hl_lines="7 13 14 17"
[witness@example-ucacr1 patches]$ ls -la ..
total 76
drwxr-xr-x. 16 witness witness  4096 Mar 30 18:23 .
drwxr-xr-x.  4 root    root     4096 Mar 30 14:12 ..
drwxrwx---   2 witness witness  4096 Mar 30 18:20 bin
drwxrwx---   5 witness witness  4096 Mar 30 18:20 ema
lrwxrwxrwx   1 root    root       34 Mar 30 18:23 jdk8 -> /opt/witness/packages/jdk1.8.0_161
drwxrwx---   3 witness witness  4096 Mar 30 18:20 keystore
drwxrwx---   4 witness witness  4096 Mar 30 18:20 kms
drwxrwx---   2 witness witness  4096 Jun 14  2017 lib
drwxrwx---   2 witness witness  4096 Mar 30 18:20 lib64
drwxrwx---   2 witness witness  4096 Jun 14  2017 logs
drwx------.  2 root    root    16384 Mar 30 14:10 lost+found
drwx------   4 root    root     4096 Mar 30 18:23 packages
drwxrwx---   2 witness witness  4096 Mar 30 18:20 patches
drwxrwx---   3 witness witness  4096 Mar 30 18:20 properties
lrwxrwxrwx   1 root    root       42 Mar 30 18:23 tomcat8 -> /opt/witness/packages/apache-tomcat-8.5.24
drwxrwx---   2 witness witness  4096 Mar 30 18:20 wav
drwxrwx---   2 witness witness  4096 Jun 14  2017 webapps
drwxrwx---   2 witness witness  4096 Mar 30 18:20 wrapper
```

Пример моей установки (я пользователь **witness**):

```bash
$ cd /opt/witness/patches
$ ../jdk8/bin/java -cp patchtool.jar Patcher /home/witness/acr-15.1fp2-0010.apa
$ ../jdk8/bin/java -cp patchtool.jar Patcher /home/witness/acr-15.1fp2-0019.apa
```

Проверяем что сейчас стоит:

```bash
[witness@example-ucacr1 patches]$ ../jdk8/bin/java -cp patchtool.jar Patcher
The current baseline is 15.1fp2
The current patch is 0019
The following patches are installed:
Patch 0010 installed on Fri Mar 30 19:02:54 MSK 2018
Patch 0019 installed on Fri Mar 30 19:14:58 MSK 2018
```

**Проверка ACR prosess id**

```bash
ps -Af | gerp witness.home
```
## Создание сертификатов

**Сертификат для HTTPS доступа**
If you would like to test this implementation, you can practice this procedure with a certificate authority\'s 30-day trial certificate. Then, to implement real certificates, you can start over from this point.

1. Выбрать алгоритм шифрования: RSA (Rivest—Shamir—Adleman) или EC (elliptic curve). RSA больше подходит, EC меньше и быстрее. При выборе EC убедиться, что наш CA может подписывать ECDSA сертификаты.

2. Залогиниться на сервер и перейти в каталог:

```bash
cd <installdir>/keystore
```

Пример:

```bash
[witness@example-ucacr1 witness]$ cd /opt/witness/keystore/
[witness@example-ucacr1 keystore]$ ls
avayapcs511.ks  cacerts  srtpcert.jks  verint.jks
```

3\. В командах ниже синтаксис зависит от операционной системы.

=== "Linux"

    ```bash
    ../jdk8/bin/keytool Linux # т.е. утилита здесь /opt/witness/jdk8/bin/keytool
    ```

=== "Windows"

    ```bash
    ..\jdk8\bin\keytool
    ```

4\. Если файл уже существует, удалить его (заменив `<keystorename>` на соответствующее имя сертификата).

=== "Linux"

    ```bash
    rm <keystorename>
    ```

=== "Windows"

    ```bash
    del <keystorename>
    ```

5. Запустить java keytool утилиту

```bash
keytool -genkeypair -keystore <keystorename> -alias <alias> -keyalg RSA
```

Если хотим выбрать шифрование EC, заменяем RSA в конце на него. Наша итоговая строка:

```bash
../jdk8/bin/keytool -genkeypair -keystore keystore.jks -alias tomcat -keyalg RSA
```

Проверка существования *keystore.jks*. До создания:

```bash
[witness@example-ucacr1 keystore]$ pwd
/opt/witness/keystore
[witness@example-ucacr1 keystore]$ ../jdk8/bin/keytool -list -keystore keystore.jks
keytool error: java.lang.Exception: Keystore file does not exist: keystore.jks
```

6. Заполнить следующие данные:

Password: **ContactStorPass55** — пароль должен быть именно c этим соблюдением заглавных и строчных букв

i First & Last Name: полное доменное имя, IP адрес или внутреннее имя
ii Organizational Unit: дивизион
iii Organization: название компании (RTRS)
iv City/Location: расположение (Moscow)
v State/Province: штат
vi Country Code: код страны (RU для России, GB для Великобритании).

7. Нажать **yes** для подтверждения.
8. Нажать **Enter** при запросе второго пароля.

**Сгенерировать запрос на получение сертификата Certificate Signing Request (CSR)**

После получения CSR он вставляется на веб-страницу CertificateAuthority\'s. Генерирование CSR:

1\. Заново запустить команду keytool с другими параметрами (если мы всё еще в директории keystore)

```bash
keytool -certreq -keystore <keystorename> -alias <alias>
```

Наша команда:

```bash
keytool -certreq -keystore keystore.jks -alias tomcat
```

2. Ввести пароль **ContactStorPass55**.
3. Скопировать и вставить вывод команды на веб-страницу CA. (включая строки BEGIN и END).
4. Завершить процесс верификации.
5. Ответить на письма verification emails и сделать другие шаги для получения подписанного сертификата от CA.

**Импорт CA сертификатов**
До того как мы сможем импортировать certificate reply, надо импортировать корневой сертификат certificate

Authority и любые промежуточные сертификаты между корневым и нашим.
1\. Скачать эти сертификаты с сайта certificate authority.
2\. Сохранить корневой сертификат как rcert.crt и любой промежуточный как icert.crt. Если промежуточных два или более, задать им разные имена.

Для импорта всех сертификатов:

1. Импорт корневого сертификата:

```bash
keytool -importcert -keystore <keystorename> -alias root -file rcert.crt
```

2\. Ввести пароль ContactStorPass55.

3\. Импорт промежуточного сертификата (если требуется).

```bash
keytool -importcert -keystore <keystorename> -alias inter -file icert.crt
```

Следующие промежуточные импортировать как inter1 и т.д..

4\. Импортировать наш подписанный сертификат. Сохранить файл, присланныйCA, как cert.crt. Выполнить:

```bash
keytool -importcert -keystore <keystorename> -alias <alias> -file cert.crt
```

**Бэкап keystore файла**

Этот файл сейчас содержит два значения, которые невозможно повторно генерировать, поэтому важно сделать бэкап. При потере любого из компонентов надо перегенерировать сертификат и заново заплатить, чтобы его подписали.
- Случайный ключ private key, уникальный для данного веб-сервера
- Подписанный сертификат, за который мы заплатили

**Добавление дополнительных AES CA Root сертификатов в ACR**

На сервере **AES** зайти **Security→Certificate Management→CA TrustedCertificates**.

Выбрать каждый из новых CA Root сертификатов через кружочек.

![](AES-Export_caSMGR_list.png)

Выберем сгенерированный ранее в SMGR для AES сертификат caSMGR, жмем **Export**, в окне содержащем CA Root cert в текстовом формате, скопировать все содержимое (включая строки `-----BEGIN CERTIFICATE-----` и `-----END CERTIFICATE-----`) и вставить в имя файла, которое мы выберем, например *CAname.pem* (типа *inhouseca.pem*)

![](AES-Export_caSMGR.png)

Дополнительно сертификат caSMGR надо скопировать в ACR в **/opt/witness/keystore/cacerts/caSMGR.pem**. Как вариант, зайти в этот каталог и создать в редакторе vi файл, вставив туда содержимое буфера обмена. Если не скопировать, будет ошибка на ACR и запись работать не будет:

```bash
cd /opt/witness/keystore/cacerts
vi caSMGR.pem
```

![](AES-Export_caSMGR_import_ACR.png)
## Завершение установки
Выполнить необходимые изменения файла настроек, отображенные в таблице фиксированных issues. Также сюда добавляются расширенные настройки, например уровень логирования `log.level=debug`. Например увеличим количество аудиозаписей, которое находится при поиске:

```ini title="/opt/witness/properties/acr.properties"
viewerx.limit=1000 # по умолчанию 100, а это значит что по факту выводится 10 записей
```

Снова запустить ACR:

```bash
systemctl start acr
```


Файл целиком:
```ini title="/opt/witness/properties/acr.properties"
log.level=ERROR
viewerx.limit=1000
acr.logkeepdays=2
acr.dialerlist=PCS
PCS.class=com.swhh.cti.pcscon.PCSDialer
#PDS.switchnumber=9
PCS.blockagentids=false
PCS.hostname=apc
PCS.username=client1
PCS.password=server1
PCS.secure=true
PCS.replyip=10.10.204.17
```

Пример работы через proxy.

```ini title="/opt/witness/properties/acr.properties"
acr.dialerlist=PCS
PCS.class=com.swhh.cti.pcscon.PCSDialer
PCS.hostname=pusqaHP09
PCS.username=client00
PCS.password=server00
PCS.secure=true
PCS.blockagentids=false
PCS.switchnumber=345
PCS.replyip=10.0.1.205
```


Проверить из веб-интерфейса статус сервера: *Status→ Server*. Проверить алармы.

Создать Java keystore внутри каталоге keystore по инструкции *Planning, Installation and Administration Guide*.
## Свойства H.323 и SIP телефонов, которые хотим записывать

Для стандартных H.323 телефонов должен быть COR, в котором `Can be Service Observed? y`, а также включен `IP SoftPhone? Y`. Для SIP телефонов дополнительно надо включать *Type of 3PCC Enabled -Avaya* (страница 6 в `change station` либо из *SMGR → User Management→ Endpoint Editor*):

```bash title="disp st 22096 странице 6" hl_lines="5"
display station 22096                                           Page   6 of   6
                                     STATION
SIP FEATURE OPTIONS

         Type of 3PCC Enabled: Avaya
                    SIP Trunk: aar
                    
 Enable Reachability for Station Domain Control: s
```

![](SIP-Station-EnableRecording.png)
## Настройка AES для стыка с ACR

![](AEC-acr-dmcc-user-UnrestrictedAccess.png)

![](AES-acr-dmcc-user.png)

![](AES-CM-Interface.png)

![](AES-DMCC-ports.png)

![](AES-TSAPI.png)

![](AES-SecurityDatabase.png)


![](AES-tlink.png)

![](AES-Status-Switch-Conn-Summary.png)

На ACR ввести адрес AE-сервера.

![](ACR-AES-address.png)



## Настройка ACR
### Лицензия
![](ACR-activation.png)

*Система→ Лицензия*
По умолчанию:

![](ACR-without-license.png)

Ввели ключ активации лицензии:

![](ACR-with-license_1.png)

### Общие настройки — Сервер

![](ACR-server-ip-address.png)

*Общие настройки→ Сервер*
Окно по умолчанию:

![](ACR-settings-server-default.png)

Начиная с ACR 10.1 SP2 FP (Feature pack), логический TLINK удален и заменен на настройку имени TSAPI Switch, где надо задать имя AES Switch соединения.
Add switch connection name configured on AES under AES TSAPI Switch Name(s) on ACR 10.1 SP2 FP (or ACR 11) instead of adding TLINK

| Параметр                                                                                 | Значение     |
| ---------------------------------------------------------------------------------------- | ------------ |
| Пул устройств записи                                                                     | main         |
| Справедливая доля неназначенных записей                                                  | Справедливая |
| Максимальное количество одновременных записей                                            | 1200         |
| Сигнальный программный телефон                                                           | 73005        |
| Предупреждать при достижении нижнего порога количества доступных лицензированных каналов | 1            |
| Предупреждать, когда число доступных лицензий на запись экрана падает НИЖЕ               | 4            |
| IP-адрес данного сервера для записей (RTP, запись экранов и т.д.)                        | тут_адрес    |
| Используйте стереоформат аудио, когда это возможно                                       | Да           |
| Максимальная продолжительность фрагмента записи (мин.)                                   | 120          |
| SNMP Read Community                                                                      | Не задано    |
| Местоположения уведомлений SNMP                                                          | Не задано    |
| Версия SNMP                                                                              | V1           |
| Сервер(ы) воспроизведения (по умолчанию первичный и резервный)                           | тот_же_адрес |
| Сервер управления ключами                                                                | Не задано    |
| URL-адреса внешних портов управления для подключения                                     | Не задано    |

![](ACR-settings-main.png)
### Общие настройки→ ucacm
Виртуальным телефонам надо задать один и тот же пароль в CM, т.к. в ACR можно прописать только 1 пароль.

*Формат записи*
Поддерживается два аудиокодека: G.711 m-law (64kbps) и G.729A (8kbps моно или 16 kbps стерео). Выбор формата зависит от типа свича и сетевой топологии. Если ACR получает аудио в G711 m-law, он будет самостоятельно по умолчанию сжимать его в G729A. Я сделала сетевой регион 11 с codec-set 6. Поставила G.711 m-law (**g711U**), т.к. при тесте на слух оказалось, что если на ACR приходит G.711 m-law и затем уже сжимается (это по умолчанию) в G.729, то звук менее приглушенный. 60мс вместо 20 — требование системы записи.

!!! warning "Внимание"
    поддерживается только G711m-law и не поддерживается a-law вне зависимости от страны (в России стандарт a-law).

Если сжатие не нужно, можно его выключить принудительно в настройках:

```ini
acr.disablecompress=true
```

Отрицательные моменты получения аудио в G.711 (и последующего сжимания) по сравнения с получение аудио сразу в G.729:
- Между ACR и источником аудио необходима полоса пропускания в несколько раз больше.
- Производительность потоковой записи на сервере будет уменьшена примерно на 30%, если сервер не поддерживает **SSSE3**.
- Если получает G.729A, он аудио не сжимает.

*Общие настройки→ ucacm* по умолчанию:

![](ACR-settings-new-data-default.png)

| Параметр                                                                                                  | Значение                                                            |
| --------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| Тип источника данных                                                                                      | Communication Manager                                               |
| Количество минут, после которых информация о вызове, с присвоенным идентификатором вызова, не учитывается | 180                                                                 |
| Использовать звуковой сигнал                                                                              | Нет                                                                 |
| Интервал между сигналами (сек.)                                                                           | 5                                                                   |
| Включены дополнительные страницы администрирования                                                        | Запись по требованию, Запись встречи, Воспроизведение через телефон |
| Формат аудио                                                                                              | g711U                                                               |
| Имя Avaya Communication Manager                                                                           | ucacm                                                               |
| Максимальная общая продолжительность вызова (часов)                                                       | 10                                                                  |
| Адрес(а) AE-сервера                                                                                       | тут_адрес_AES                                                       |
| Имя пользователя DMCC                                                                                     | acr  (пользователь acr из AES)                                      |
| Пароль DMCC                                                                                               | `*****` (пароль задан в AES)                                        |
| Шифрование медиа-потоков                                                                                  | none (отсутствует)                                                  |
| Код безопасности внутреннего IP-номера                                                                    | `*****`   (пароль задан в CM)                                       |
| Сервер(ы) AES TSAPI                                                                                       | тот_же_адрес_AES                                                    |
| Имя (имена) коммутатора AES TSAPI                                                                         | ucacm                                                               |
| Идентификатор входа службы AES TSAPI                                                                      | tsapiuser (пользователь tsapiuser из AES)                           |
| Пароль службы AES TSAPI                                                                                   | `*****`  (пароль задан в AES)                                       |
| Незаписываемые станции/порты IVR для наблюдения через TSAPI                                               | Не задано                                                           |
| Группа(ы) приема вызовов с учетом квалификации агентов, отслеживаемые через TSAPI                         | 75100                                                               |
| VDN(s) для наблюдения через TSAPI                                                                         | 72000                                                               |
| VDN, используемый для пометки вызовов                                                                     | Первый                                                              |
| Добавить номер VDN в качестве дополнительного "владельца" вызовов                                         | Нет                                                                 |
| Адрес Communication Manager                                                                               | Не задано                                                           |
| Avaya Oceana™                                                                                             | Не задано                                                           |
| Записывать с помощью пассивного перехвата IP                                                              | Нет                                                                 |
| Порты 73001-73005  (73005 программный)                                                                    | 5                                                                   |

![](ACR-type-of-data.png)

![](ACR-settings-new-data.png)

### Настройки массовой записи
*Операции→ ucacm Массовая запись*. Сейчас записываем номера телефонов, т.е. запись будет идти даже если телефоны разлогинены. Если надо записывать разговоры только когда операторы залогинены, ввести диапазон номеров агентов 22000-22003. Перезагрузка ACR не требуется.

| Параметр                                                                                           | Значение                                                              |
| -------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| Владельцы записи                                                                                   | Не задано                                                             |
| Записывать внутренние вызовы?                                                                      | Да                                                                    |
| Блокирование IP-записи (принудительное использование TDM)                                          | Нет                                                                   |
| Процент записываемых вызовов                                                                       | 100                                                                   |
| Процент записываемых экранов                                                                       | 0                                                                     |
| Управление записью                                                                                 | По умолчанию (автоматический запуск, без ручного/внешнего управления) |
| Укажите целевые объекты для записи                                                                 | по внутренним номерам, агентам, номерам VDN или навыкам               |
| Удалить запись при вводе                                                                           | Не задано                                                             |
| Удержание записи при вводе                                                                         | Не задано                                                             |
| Количество целевых адресов (в таблице ниже)                                                        | 4                                                                     |
| Адреса 22096-22099 (пишем по номерам телефонов, звонки записываются даже если телефон разлогинен). | 4                                                                     |

!!! warning "Внимание"
    у всех объектов (телефоны, VDN) должны быть заданы имена, иначе ACR ругается.

![](ACR-settings-ucacm.png)

Проверка статуса соединения: *Состояние→ Сервер*. DMCC обязательно **АКТИВНЫЙ**, TSAPI включен.

| Параметр                                                                                               | Значение                                                                                                               |
| ------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------- |
| Канал к DMCC на адрес_AES                                                                              | АКТИВНЫЙ                                                                                                               |
| Канал к TSAPI на адрес_AES                                                                             | ВКЛЮЧЕН                                                                                                                |
| Всего вызовов отслежено через CTI с момента запуска                                                    | 1 585                                                                                                                  |
| Всего мультимедийных файлов записано на настоящий момент                                               | 921                                                                                                                    |
| Всего вызовов отслежено через CTI сегодня (или после перезапуска, если он был произведен сегодня)      | 90                                                                                                                     |
| Всего мультимедийных файлов записано за сегодня (либо с момента перезапуска, если он выполнен сегодня) | 46                                                                                                                     |
| Всего записей за сегодня по типам (или с момента перезапуска, если в течение сегодняшнего дня)         | DMCC: 47                                                                                                               |
| Дата самого раннего вызова, сохраненного на диске                                                      | 04/04/18 18:10:30                                                                                                      |
| Сервер 848033                                                                                          | Рекомендовано записей (с момента запуска этого сервера): 231. Самые последние 1 мин назад (и завершенные 1 мин назад). |

![](ACR-settings-alarms.png)
### Воспроизведение записей
Пример вкладки воспроизведения. Обязательно установить Java. Возможен поиск по дате, АОНу, номеру оператора или телефона.

![](ACR-list-records.png)

## Резервное копирование ACR

### Резервное копирование аудио и метаданных по SFTP
Аудиофайлы *.wav* и файлы метаданных *.xml* из каталога */calls* можно архивировать несколькими способами:
- Общий каталог Windows (SMB)
- Локальный (или смонтированный) диск
- Сервер SFTP
- EMC Centera

Для Windows: [https://www.bitvise.com/ssh-server-download](https://www.bitvise.com/ssh-server-download) (простой вариант) или  [https://github.com/PowerShell/Win32-OpenSSH/releases скачать OpenSSH-Win64.zip](https://github.com/PowerShell/Win32-OpenSSH/releases%20скачать%20OpenSSH-Win64.zip) (более сложный).
Для Linux: [http://www.proftpd.org/](http://www.proftpd.org/) — бесплатный SFTP для Linux, поддерживает extensions.

Бэкап инкрементный, ежедневно добавляются новые файлы. Требование к SFTP серверу для ACR отличаются от других систем, они описаны в Administration Guide (страница 182):
Требование к SFTP серверу для ACR отличаются от других систем, они описаны в Administration Guide (страница 182):
- SFTP V3 или выше
- Поддержка расширения <statvfs@openssh.com>. Этот extension доступен, если `Can check available space — Yes`. Почти все SFTP серверы под Windows (Core FTP mini SFTP server, SolarWinds Free SFTP/SCP Server, Filezilla Server и пр.) не поддерживают extensions, соответственно <statvfs@openssh.com> недоступен. [Bitvise SSH Server](https://www.bitvise.com/ssh-server-download) —простой вариант ssh server для Windows, [Win32-OpenSSH](https://github.com/PowerShell/Win32-OpenSSH/)  —более сложный. [ProFTPD](http://www.proftpd.org/) — бесплатный SFTP для Linux, поддерживает extensions.

Проверить функций, поддерживаемые SFTP сервером, можно из WinSCP после подключения к нему: *Session→ Server/Protocol Information*.

![](WinSCP-SFTP-Server-Protocol-Info.png)

На первой вкладке Protocol версия SFTP протокола:

![](WinSCP-SFTP-Server-Protocol-Version.png)

На второй вкладке Capabilities. Внизу должна отобразиться дополнительная информация:

```txt hl_lines="3" title="Capabilites, проверка поддержки расширения statvfs@openssh.com"
The server supports these SFTP extensions:
  posix-rename@openssh.com="1"
  statvfs@openssh.com="2"
  fstatvfs@openssh.com="2"
  hardlink@openssh.com="1"
  fsync@openssh.com="1"
```

The server supports these SFTP extensions:

![](WinSCP-SFTP-Server-Protocol-Capabilities.png)

Скачать [**Bitvise SSH Server**](https://www.bitvise.com/ssh-server-download)  — выбрать **Personal** или платный вариант. Работает с аккаунтами Windows или виртуальными. Функционал :

![cid:image002.png@01D3FFF9.DB7C91B0](image27.png)

![cid:image003.png@01D3FFF9.DB7C91B0](image28.png)

После проверки доступности SFTP сервера настраиваем ACR: *Операции→ Задания Ввода-Вывода→ Добавить архив*. Заголовок задания произвольный.

![](ACR-settings-sftp.png)

При успешном сохранении файлов видно, сколько свободного места осталось на SFTP сервере, сколько файлов туда записано и какого размера.

![](ACR-backup.png)

С правами root проверим из консоли, что копируется на SFTP (проще WinSCP). 848033 — корневая папка с архивом ACR. Внутри неё папки с годом, месяцами и днями.

```bash
[root@example-ucacr1 ~]# sftp avaya@адрес_сервервера_backup
The authenticity of host 'адрес_сервервера_backup (адрес_сервервера_backup)' can't be established.
ECDSA key fingerprint is SHA256:iEM6JQV+UCzzRw6HU17bGrSluhOA3mhWYiCHckT/iI.
ECDSA key fingerprint is MD5:ba:aa:91:82:5a:08:af:ef:59:9b:6e:2d:52:e8:81:49.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'адрес_сервервера_backup' (ECDSA) to the list of known hosts.
avaya@172.21.40.43's password:
Connected to адрес_сервервера_backup.

sftp> ls
848033                                         SMGR-backup_2018_Jun_09_02_00_24_841.zip
SMGR-backup_2018_Jun_15_11_30_31_288.zip       full_example-ucacm1_234501_20180615.tar.gz
full_example-ucacm1_234501_20180622.tar.gz   full_example-ucacm2_140719_20180609.tar.gz
full_example-ucacm2_141010_20180609.tar.gz

sftp> ls 848033
848033/0010330001.uid  848033/2018
sftp> ls 848033/2018/06/
10/  11/  12/  14/  15/  16/  17/  18/  19/  20/  21/  22/  23/  24/
```

Всё скачать к себе с sftp сервера:

```bash
get -r *
```

Архив одного дня хранится я в одном файле *.tar*. Он содержит аудиофайлы wav и метаданные с информацией о звонке xml:

```bash
sftp> get 848033/2018/06/22/10/12.tar
Fetching /848033/2018/06/22/10/12.tar to 12.tar
/848033/2018/06/22/10/12.tar                                100%   10MB  11.0MB/s   00:00
sftp> exit

[root@example-ucacr1 ~]# ls
12.tar  848033  anaconda-ks.cfg  original-ks.cfg  SMGR-backup_2018_Jun_09_02_00_24_841.zip

[root@example-ucacr1 ~]# tar -tvf 12.tar
-r--r--r-- avaya/avaya    6204 2018-06-21 10:08 848033000002364.xml
-r--r--r-- avaya/avaya  103994 2018-06-21 10:08 848033000002364.wav
-r--r--r-- avaya/avaya    6201 2018-06-21 10:10 848033000002365.xml
-r--r--r-- avaya/avaya   56114 2018-06-21 10:10 848033000002365.wav
-r--r--r-- avaya/avaya    6203 2018-06-21 10:14 848033000002366.xml
-r--r--r-- avaya/avaya   66074 2018-06-21 10:14 848033000002366.wav
-r--r--r-- avaya/avaya    6202 2018-06-21 10:15 848033000002367.xml
-r--r--r-- avaya/avaya   35354 2018-06-21 10:15 848033000002367.wav
-r--r--r-- avaya/avaya    6203 2018-06-21 10:29 848033000002368.xml
-r--r--r-- avaya/avaya   97574 2018-06-21 10:29 848033000002368.wav
-r--r--r-- avaya/avaya    6202 2018-06-21 10:30 848033000002369.xml
...
```

Разархивируем одну запись и её матаданные.

```bash
[root@example-ucacr1 ~]# tar xvf 12.tar 848033000002364.*
848033000002364.xml
848033000002364.wav
[root@example-ucacr1 ~]# ls
12.tar  848033000002364.wav  anaconda-ks.cfg  SMGR-backup_2018_Jun_09_02_00_24_841.zip
848033  848033000002364.xml  original-ks.cfg
```

Файл xml c информацией о звонке:

```bash
[root@example-ucacr1 ~]# cat 848033000002364.xml
<?xml version="1.0" encoding="UTF-8"?>
..... тут много текста
</cti>
</ctiinfo> timestamp="2018-06-21T09:57:43.269+03:00" eventtype="Disconnected"></tags></cti>
</session>
</sessions>
</contact>
</cti>
</recording>
```

### Win32-OpenSSH portable 64 bit
В Win32-OpenSSH привязка к Windows аккаунтам, т.е. вход по логину пользователей AD. Там есть то что надо.

Установим SFTP/SSH сервер на Windows используя OpenSSH согласно [инструкции](https://winscp.net/eng/docs/guide_windows_openssh_server)

Отсюда скачать [Win32-OpenSSH](https://github.com/PowerShell/Win32-OpenSSH/), выбирать последний релиз и качать архив с portable версиями для 64-битной системы, например https://github.com/PowerShell/Win32-OpenSSH/releases/download/v9.2.2.0p1-Beta/OpenSSH-Win64.zip. Распаковать в *C:/Program Files/OpenSSH* (папку создала).
![](ProgramFiles-OpenSSH.png)

Заходим в командную строку с правами администратора. Переходим в вышеуказанный каталог и устанавливаем сервисы *sshd* и *ssh-agent.*

```powershell
C:\windows\system32>cd /

C:\>cd "Program Files"/OpenSSH

C:\Program Files\OpenSSH>powershell.exe -ExecutionPolicy Bypass -File install-sshd.ps1
[SC] SetServiceObjectSecurity: успех
[SC] ChangeServiceConfig2: успех
[SC] ChangeServiceConfig2: успех
sshd and ssh-agent services successfully installed

C:\Program Files\OpenSSH>
```

В файрволе разрешить соединения к SSH серверу. У них открыто, т.к. другой SFTP сервер был доступен. Можно пропустить.

Вариант открытия 22 порта из командной строки с правами администратора. Доступен в Window 8, 2012 или более новых.

```powershell
New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
```

Стандартный способ. Control Panel (Панель управления)→ System and Security (Система и Безопасность, при мелких значках не надо)→ WindowsFirewall1 (Брандмауэр Windows) → Advanced Settings (Дополнительные параметры) → Inbound Rules (Правила для входящих подключений)→ правой кнопкой Создать правило (new rule) для порта 22 (TCP).

![](windows-firewall1.png)

![](windows-firewall2.png)

![](windows-firewall3.png)

Запустить сервис и/или настроить на автоматический запуск:

Control Panel (Панель управления)→ System and Security (Система и безопасность)→ Administrative Tools (Администрирование)→ Services(Службы). Проще Пуск → набрать `services.msc` и Enter.

Найти OpenSSH SSH Server (имя службы sshd).

Если нужен автоматический запуск при старте компьютера, в свойствах меняем Action (Тип запуска)→ Properties (вручную) на Automatic(Автоматически).

Если же необходимо просто потестировать, запустим правой кнопкой мыши.

![](windows-firewall4.png)

Если не генерировать публичные ключи ssh, то работает в WinSCP вход со своей учетной записью в Windows:

Логин@домен и пароль, соответствующий нашему паролю в Windows. Если так зайти, то будут доступны **все жесткие диски** нашего компьютера.

Настройка каталога по умолчанию. Откроем *С:/ProgramFiles/OpenSSH/sshd_config_default*

```bash title="С:/ProgramFiles/OpenSSH/sshd_config_default"
# override default of no subsystems
Subsystem	sftp	sftp-server.exe
```

Хотим сделать по умолчанию каталог C:/DISTR

```bash title="С:/ProgramFiles/OpenSSH/sshd_config_default"
# override default of no subsystems
Subsystem	sftp	sftp-server.exe -C C:\DISTR
```

### Резервное копирование базы PostgreSQL по расписанию

Бэкап базы данных нельзя настроить из веб-интерфейса. Из Linux CLI под root создать скрипт */opt/witness/bin/pgbackup.sh*:

```bash title="/opt/witness/bin/pgbackup.sh"
#!/bin/bash
# переменные задаем по необходимости
BACKUP_DIR="/calls/pgbackups"
NUM_TO_KEEP="5"                 # сколько последних бэкапов хранить, число файлов
DAYS_TO_KEEP="3"                # для второго варианта - число дней хранения
#time=`date '+%F'-'%H''%M''%S'` # это не используем, т.к. надо для postgres, а не root
#
# Удаление старых бэкапов основанное на количестве файлов бэкапа, которое надо оставить.
# Все файлы кроме пяти последних удаляются.
ls -1t ${BACKUP_DIR}/pgbackup* | tail -n +${NUM_TO_KEEP} | xargs -i -n1 rm -f "{}"
# Второй вариант — удаление старых бэкапов на основании количества дней, в течение которых надо хранить бэкап
# Этот вариант не рекомендуется т.к. при неуспешном бэкапе могут удалиться все файлы.
# Переменная NUM_TO_KEEP всегда сохранит определенное число файлов независимо от даты создания бэкапов.
# Для второго варианта раскомментировать строку ниже.
#find ${BACKUP_DIR}/pgbackup* -type f -mtime +${DAYS_TOKEEP} -exec rm {} \;
#
### Сам скрипт бэкапа
# Параметр exclude-table=\*_i исключает бэкап таблиц segment_i и/или session_i
# если таковые существуют. Таблицы нужны для турбо-режима просмотра. Они пересоздаются
# из других таблиц при старте сервера ACR c восстановленной базой данных
# автоматически в фоновом режиме.
# --compress=5 средняя степень сжатия. Максимально 9. При увеличении числа требуется
# больше ресурсов, скорость ниже, но итоговый файл меньше по размеру
# Дамп выполняем от лица postrges (нет роли для root), иначе пишет
# ls: cannot access /calls/pgbackups/pgbackup*: No such file or directory
# pg_dump: [archiver (db)] connection to database "eware" failed: FATAL:  role "root" does not exist
# Здесь не могу использовать переменную BACKUP_DIR и time
su - postgres -c 'pg_dump --format=c --compress=5 eware > /calls/pgbackups/pgbackup.$(date '+%F'-'%H''%M''%S').dmp'
sleep 3
```

Меняем владельца файла на postgres и даем всем право на исполнение скрипта. Создать каталог **/calls/pgbackups**, где будут сохраняться бэкапы(дампы) базы данных, сделать пользователя postgres (группа тоже postgres) владельцем каталога. Проверяем наличие каталога и его владельца.

```bash hl_lines="16"
chown postgres. /opt/witness/bin/pgbackup.sh
chmod +x /opt/witness/bin/pgbackup.sh

# ls -la /opt/witness/bin/pgbackup.sh
-rwxr-xr-x 1 postgres postgres 2248 Jun 25 01:43 /opt/witness/bin/pgbackup.sh

mkdir -p /calls/pgbackups
chown postgres. /calls/pgbackups

# ls -la /calls
total 32
drwxr-xr-x.  5 witness  witness   4096 Jun 25 01:45 .
dr-xr-xr-x. 19 root     root      4096 Mar 30 14:16 ..
drwxrwx---   3 witness  witness   4096 Apr  4 18:10 848033
drwx------.  2 root     root     16384 Mar 30 14:10 lost+found
drwxr-xr-x   2 postgres postgres  4096 Jun 25 01:45 pgbackups
```

Скрипт **pgbackup.sh** выполнять вручную либо по расписанию. Ручная проверка. При первом исполнении скрипта появится ошибка, т.к. нет еще ни одного файла в каталоге для поиска кандидатов на удаление:

```bash
/opt/witness/bin/pgbackup.sh

ls: cannot access /calls/pgbackups/pgbackup*: No such file or directory
```

Задать расписание. В примере делаем бэкап ежедневно ночью в три часа пять минут (минутыуказываются перед часами):

```bash
crontab -e

05 03 * * * /opt/witness/bin/pgbackup.sh
```

При повторном исполнении скрипта ошибок не будет. В каталоге появился дамп после исполнения:

```bash
# ls -la /calls/pgbackups/
total 348
drwxr-xr-x  2 postgres postgres   4096 Jun 25 03:22 .
drwxr-xr-x. 5 witness  witness    4096 Jun 25 01:45 ..
-rw-r--r--  1 postgres postgres 344651 Jun 25 03:22 pgbackup.2018-06-25-032254.dmp
```

Пример, когда уже 5 дампов сохранилось.

![](image36.png)

Возможная ошибка при исполнении скрипта:

```bash
pg_dump: [archiver (db)] connection to database "eware" failed: could not connect to server: No such file or directory
        Is the server running locally and accepting
        connections on Unix domain socket "/tmp/.s.PGSQL.5432"?
```

Причина: postgresql использует Unix domain сокеты для соединения от лица пользователя postgres. Другие соединения используют TCP/IP сокеты. В Linux сокеты Unix domains — текстовые файлы, в примере выше */tmp/.s.PGSQL.5432*. Ошибка обозначает, что процесс postmaster process держал сокет домена открытым с помощью `lsof`, но файла не было. Скорее всего он случайно удален, каталог */tmp* пустой, что подозрительно. Возможно какой-то скрипт Cron очищал временный каталог. Самый простой способ решить проблему —перезагрузиться командой reboot под пользователем root. Также можно вручную перезагрузить сервис базы данных:

```bash
service cscm stop (или acr)
service postgresql restart
service cscm start ( или acr).

ls -a /tmp  # проверить содержимое временного каталога. Убедиться, что сокет домена пересоздан.
```

!!! note "Примечание"
    В стандартном варианте скрипта, который предлагает Avaya, дамп базы данных не сжимается архиватором. Желательно добавить архивирование, т.к. база будет разрастаться.

**Восстановление базы из бэкапа**
Переустановить операционную систему. Залогиниться как root, установать софт ACR.

```bash
systemctl stop acr  # остановить сервис записи
su - postgres       # переключиться на пользователя postgres
dropdb eware        # Удалить существующую базу
createdb eware      # Создать пустую копию базы PostgreSQL
pg_restore --dbname=eware --use-set-session-authorization имя_файла_бэкапа # Восстановить данные из бэкапа
systemctl start acr
```

Проверить успешное восстановление из бэкапа после запуска ACR,

!!! note "Примечание"
    В процессе восстановления могут появиться многочисленные ошибка «Error from TOC entry», относящиеся к внутренним функциям PostgreSQL. Также могут быть сообщения, что функции или языки уже существуют. Не обращать внимания.
## Проблемы
### Не пишет звонки, т.к. длинный внутренний номер принимает за внешний.

If you want the recording rule to fire then you will need to set `tsapi.numplanlength=9` as the station is 9 digits long which is longer than the default of 8 which are considered internal stations anything longer is considered external.

### Не пишет звонки: нет места на диске
Настроен бэкап на sftp. Бэкап, настроенный на SMGR, работает. При указании того же сервера в ACR бэкап не работает.

В логе ACR:

```bash
2018-06-03 23:57:51,311 [IO-1] INFO  com.swhh.cs.uarchive.v - Trying to activate a destination
2018-06-03 23:58:55,918 [IOJobs Manager thread] ERROR com.swhh.cs.uarchive.aa - Folder sftp://borisova.rtrs.local/: net.schmizz.sshj.sftp.SFTPException - Operation unsupported
```

Ручная проверка sftp дает положительный результат, можно зайти насервер, другие бэкапы видны.

```bash
[witness@example-ucacr1 logs]$ sftp avaya@borisova.rtrs.local
avaya@borisova.rtrs.local's password:
Connected to borisova.rtrs.local.
sftp> ls
221212.xlsx                                       22222.txt
222222                                            SMGR-bacup_2018_May_30_01_16_11_872.zip
SMGR-bacup_2018_May_31_11_00_48_208.zip           backup_SMGR-test_2018_Jun_01_01_33_10_793.zip
full_example-ucacm1_233001_20180601.tar.gz        full_example-ucacm2_130943_20180530.tar.gz
full_example-ucacm2_232001_20180601.tar.gz

sftp> version
SFTP protocol version 3
```

Включаем уровень DEBUG для логов в */opt/witness/properties/acr.properties*:

```ini title="/opt/witness/properties/acr.properties"
log.level=DEBUG
```

Применяем изменения, сохранив бэкап старого лога.

```bash
systemctl stop acr
mv /opt/witness/logs/acr.log{,.old}
systemctl start acr
```

По логу понятно, что не осталось свободного места на диске:

```bash hl_lines="5"
...
2018-06-05 12:18:27,661 [IO-1] DEBUG com.swhh.cs.uarchive.v - Prevmedia=null
2018-06-05 12:18:27,661 [IO-1] DEBUG com.swhh.cs.uarchive.v - Searching for blank media
2018-06-05 12:18:27,661 [IO-1] DEBUG com.swhh.cs.uarchive.v - Considering sftp://ivanova.mycompany.local isUsable=false
2018-06-05 12:18:27,661 [IO-1] DEBUG com.swhh.cs.uarchive.v - No blank media left to try
2018-06-05 12:18:27,988 [IOJobs Manager thread] ERROR com.swhh.cs.uarchive.aa - Folder sftp://ivanova.mycompany.local: net.schmizz.sshj.sftp.SFTPException - Operation unsupported
2018-06-05 12:18:28,441 [DMCC_poll_адрес_AES] DEBUG com.swhh.c.g - sending message: com.swhh.c.a.bv 30
2018-06-05 12:18:28,473 [DiskManager thread] DEBUG com.swhh.cs.f.o - purgeto=0 lowestInum=848033000000100
```
### Логи

**/opt/mvap/logs/log.дата**

=== "Linux"

    /opt/witness/logs/acr.log.yyyy-mm-dd

=== "Windows"

    D:\Program Files\Avaya\ContactRecorder\logs

### ACR не пишет звонки после добавления дополнительных bulk recording лицензий на массовую запись.

cscm-11.0-2patch110014, ACR version 12

Причина: после применения новой лицензии ACR перешел в состояние «Not Viable» (не жизнеспособен) и не мог найти ни одного жизнеспособного сервера, кудf можно записывать вызовы. Он не мог выделить ни одного порта для записи. Все текущие вызовы записывались и сохранялись в базе данных. Только входящие вызовы не писались.

```bash
2013-02-18 10:25:18,385 [http-8080-10] INFO com.swhh.utils.servlets.AuthServlet - POST license set
2013-02-18 10:25:18,385 [http-8080-10] INFO com.swhh.cs.d.y - Clearing remotes list
2013-02-18 10:25:18,386 [http-8080-10] INFO com.swhh.cs.b.ai - Setting flags.clientids changed to 
2013-02-18 10:25:18,386 [http-8080-10] DEBUG com.swhh.os.d - Found interface eth0 B4:B5:2F:5E:FD:5C
2013-02-18 10:25:18,386 [http-8080-10] DEBUG com.swhh.cs.b.bl - macaddress=B4:B5:2F:5E:FD:5C
2013-02-18 10:25:18,407 [http-8080-10] INFO com.swhh.cs.b.ai - Setting license.liccheckkey changed to 521

2013-02-18 10:25:25,911 [DMCC_event_172.20.9.111] DEBUG com.swhh.cs.c.j - Closing: 821010000102983.wav G.729A 204000
2013-02-18 10:25:25,913 [queue.name.recordingend-0] INFO com.swhh.cs.a.ac - Call ID:00555027531361154109 INUM:821010000102983 inserted into database OK. 2ms.

2013-02-18 10:25:33,353 [TSAPIEvent-1] WARN com.swhh.f.a.b.k - 00555027821361154267 could not be tapped. No appropriate viable servers connected
2013-02-18 10:25:47,752 [queue.name.recordingend-0] INFO com.swhh.cs.a.ac - Call ID:00555027791361154245 INUM:821010000102991 inserted into database OK. 2ms.

2013-02-18 10:25:52,274 [SnapTimer] WARN com.swhh.f.a.b.k - 00555027021361153895 could not be tapped. No appropriate viable servers connected
```

Решение: система работает так как и должна. Начиная с релиза 10.1 SP2 и выше (версия 11 включены) необходима перезагрузка сервиса cscm для применения лицензии.
### Не работает массовая запись Bulk Recording
Не виртуальная отдельная система записи ACR под управлением Linux. Причина: версия ACR **cscm-11.0-2patch1100104** должна быть обновлена до patch 212 ACR 12.x.

**Не работает массовая запись (Bulk recording), но алармы не появляются.**
Причина: все виртуальные телефоны DMCC были настроены для записи по требованию (on-demand).
### Verint Call Recording and Quality Monitoring, Witness ContactStore: ACR не пишет разговоры, но статус порта активный
KB01126108. ACR 10.0 Patch 100068. CTI monitors показывает, что звонок в состоянии Established state. Алармов нет 5 дней, последний найденный аларм «Error on link to Master at 127.0.0.1:1209».

Причина: неправильный владелец и права файлов **cscmmain.jar**, **cscmres.jar** и **ROOT.war**.
Решение: перегрузить сервис CSCM, залогиниться под root, сменить владельца и группу на witness:

```bash
chown witness. /opt/witness/tomcat5525/shared/lib/cscm{main,res}.jar  # или отдельно сhgrp witness cscmmain.jar
chown witness. /opt/witness/tomcat5525/webapps/ROOT.war
```
### После апгрейда AES до версии 7.0.1 ACR не может залогиниться на нем
ACR  v12.x, v15.x. AES обновлен до 7.0.1, на ACR импортировали сертификат с AES, больше ничего не менялось. Но не устанавливается связь между ACR и AES. В логах AES найдена ошибка «unknown protocol» при попытке ACR залогиниться на AES.

`error:140760FC:SSL routines:SSL23_GET_CLIENT_HELLO:unknown protocol`

Причина: в AES 7.0.1 по умолчанию включен только TLS 1.2, а текущая версия ACR использует TLS1.0 для связи. Надо включить TLS1.0 на AES: *Networking→ TLS/TCP Settings→ отметить галочкой TLS1.0* (или отметить все три версии TLS при желании). Перегрузить сервис AES.
### Версия TLS на Avaya WFO 12/15.1.
WFO 12 работает с TLS 1.1 (если установка enterprise, то поддерживается только TLS1.0, [смотреть](https://downloads.avaya.com/css/P8/documents/100173073): страницу: 13 SSLпротоколы), WFO 15.1 работает с TLS 1.2. Из документа *security guide* по версии 15.1: все WFO сервера включая ACR поддерживают 1.2 (но не 1.3).

KMS может использовать только TLS1.0 как видно ниже (хотя команда разработчиков уже работает над проблемой). PH: "Communication between system servers and KMS in 15.1 FP0 is over TLS 1.0 as per the official documentation in the security configuration guide.

TLS 1.1 / TLS 1.2 are used in communication that is going via Secure Gateway, e.g. browsers to application servers, or server to server communication (excluding KMS where Secure Gateway is not deployed)."

**Security Configuration:**
**TLS protocols and cipher suites**
The system natively supports the following protocols:
- TLS v1.0
- TLS v1.1
- TLS v1.2

TLS cipher suite refers to a collection of algorithms used for tasks such as authentication and encryption. The specific algorithm collectionused for these tasks is selected during the SSLhandshake, when thesecurity settings used for SSL are agreed upon by both computers. Duringthe handshake process between system components, the system establishesthe most secure connection supported for that connection pair.

**TLS Protocols and Cipher Suites for the Secure Gateway**
The SSL mode in EM determines the specific TLS protocols and cipher suites that are used to encrypt communications with the Secure Gateway.
The available settings are High (strongest security), Medium (default),and Custom.
These modes are set when enabling SSL in the Avaya Suite.
Setting the SSL mode to High configures the security level of the SecureGateway to the highest level available.

**TLS Protocols and Cipher Suites for KMS**
As the KMS does not have a secure gateway, the TLS protocols and ciphersuites should be set according to Microsoft's guidelines.
Note, that regardless of which SSL mode you set for the secure gateway,communication between system servers and the KMS is over TLS 1.0.
Might also help to know that although we support SHA2 for SSL, the keyexchange between WFO and KMS (KMC-KMS mutual authentication only workswith SHA1)

**Moreover:**
There are 2 types of certificates for KMS:
- SSL сертификат, используемый для связи между system servers и KMS — поддерживает SHA2 (customer provided cert).
- KMS client сертификат, используемый для взаимной аутентификации с KMS — не поддерживает SHA2 (Verint provided cert).

The client certificate is used only for internal for KMC-KMS mutual authentication.

**This is not a SSL (HTTPS) communication certificate.**
It is used by RSA for internal validation, and could as well be a token or passphrase, only in this case they chose to implement it using a certificate.
When passed over the network it is done via secured HTTPS communication that is SHA2.
### ACR не пишет звонки — Major alarm неправильный vdn и/или не может наблюдаться через AES TSAPI

ACR 12.0 Patch 125 на Linux на виртуальном сервере VMWare Master. Сотни таких алармов:


```bash
Minor Alarm Address 86563 not recognized by TSAPI.  
Major Alarm 70668 is not a valid vdn and/or cannot be observed via AES TSAPI.
```

Причина. Изучаем лог *acr.log*:

```bash
2017-12-13 00:00:00,235 [TSAPI thread] ERROR com.swhh.cscm.h.a -TsapiException: TsapiException querying device 29439 -ACSUniversalFailureConfEvent error=tserverDriverCongestion
```

Эта ошибка отображает проблему с сервисом AES TSAPI и потоком ACR—AES не может выполнить запросы ACR TSAPI.

Решение: остановить сервис ACR (ранее cscm) (если есть резервный серверStandby и/или Slave, сначала остановить на Standby, затем на Slave,затем на Master). Затем на AES перезапустить TSAPI и DMCC сервисы(подтвердить адрес на Master, Standby т.к. они используют разные AESсервера — сделать reset на обоих AES, порядок не имеет значения).Запустить сервис ACR сначала на Master (!), затем на Slave, и в конце наStandby.

### ACR 12, 12.1, 15.1: DMCC порты записи не регистрируются на AES
Звонки не пишутся, DMCC не регистрируются на AES. ACR Alarm:

```bash
AES TSAPI Service Observation of 79818 reports error:ACSUniversalFailureConfEvent error=tserverDeviceNotSupported
```

Чтобы ACR записывал звонки, он должен регистрировать назначенные порты на **AES over DMCC**.

**Из лога acr.log:**

```bash title="acr.log"
2017-10-12 20:48:05,336 [TSAPI thread] INFO  com.swhh.csta.a.k - Attempting to add TSAPI observer to dmcc 22323
2017-10-12 20:48:05,339[TSAPI thread] INFO  com.swhh.cs.a.p - Filtered out alarm alarms.tsapi.observer [22323] [ACSUniversalFailureConfEvent
 error=tserverDeviceNotSupported]
2017-10-12 20:48:05,339[TSAPI thread] ERROR com.swhh.csta.d.c.k - Cannot Observe dmcc 22558: ACSUniversalFailureConfEvent
 error=tserverDeviceNotSupported
2017-10-12 20:48:05,343[TSAPI thread] INFO  com.swhh.csta.a.k - Attempting to add TSAPI observer to dmcc 22559
2017-10-12 20:48:05,347 [TSAPI thread] INFO  com.swhh.cs.a.p - Filtered out alarm alarms.tsapi.observer [22559] [ACSUniversalFailureConfEvent
 error=tserverDeviceNotSupported]
```

**Из лога AES DMCC error.log:**

```bash title="error.log"
2017-10-12 20.48.05,439 :T-100: com.avaya.platform.broker.impl.AsyncServiceMethodImpl invoke
WARNING: Exception when calling method.invoke
java.lang.reflect.InvocationTargetException
 at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
.....
 at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
 at java.lang.Thread.run(Thread.java:724)
Caused by: ch.ecma.csta.errors.InvalidDeviceIDException: [22323:swlink1:0.0.0.0:0] DeviceID='[22323:swlink1:0.0.0.0:0] ' does not exist.  Call DeviceServices.getDevice to create a device
 at com.avaya.cmapi.intsvc.CstaTerminalMgrImpl.findExistingStation(CstaTerminalMgrImpl.java:173)
 at com.avaya.cmapi.intsvc.CstaTerminalMgrImpl.findExistingStation(CstaTerminalMgrImpl.java:185)
 at com.avaya.cmapi.extsvc.H323RegistrationServices.registerTerminal(H323RegistrationServices.java:99)
 ... 22 more
```

Причина: на AES был неправильно настроен аккаунт пользователя CTI userдля ACR TSAPI и для DMCC в веб-интерфейсе **AES Security→ Security Database→ CTIUsers→ List all users** поставить галочку **UNRESTRICTED ACCESS.** Перезапустить сервис ACR.

### Nice is unable to monitor a station, csta log in AES shows error tserverDeviceNotSupported

AES 5.2.1 It seems that Nice keeps getting the error that it can\'tmonitor a device, checking csta logs in AES confirms error as:

```bash
02/15/2012 12:03:45.949:TSAPI:Thread 0xabf03b90: value ACSUniversalFailureConfEvent ::=
02/15/2012 12:03:45.949:TSAPI:Thread 0xabf03b90: {
02/15/2012 12:03:45.949:TSAPI:Thread 0xabf03b90: error tserverDeviceNotSupported
Verified the device is in the Nice devicegroup
```

Nice is unable to monitor a station, csta log in AES shows error \"tserverDeviceNotSupported\".

Причина: Have Avaya verify database values are set correctly for tlinkgroupid...

`-1 = NONE` (for tlink group)
`0 = ANY` (for tlink group)

This is configured upon adding a new station/device to the SDB in AES, select Tlink group ANY in order to be able to monitor the station.

Решение: неправильное значение в базе данных psql db, где устройство имеет колонку tlinkgroupid с неправильным символом `-1` для обработки запросов, а должно быть `0`.

Пример. Правильная настройка:

```bash
mvap=# select * from device where deviceid= '1716528';
devicepk_id | deviceid | devicetype | location    | telephonenumber | tlinkgroupid | hostname
-------------+---------+------------+-------------+-----------------+--------------+-----------
   12358    | 1716528  |    PHONE   | nicecti ext |                 |       0      | localhost
(1 row)
```

Неправильная настройка:

```bash
mvap=# select * from device where deviceid= '1716529';
devicepk_id | deviceid | devicetype | location    | telephonenumber | tlinkgroupid | hostname
------------+----------+------------+-------------+-----------------+--------------+-----------
   12363    |  1716529 |    PHONE   | nicecti ext |                 |      -1      | localhost
(1 row)
```

Выполнить команду в psql, обновить устройство device set `tlinkgroupid='0'` где` tlinkgroupid= '-1'`;

```bash
mvap=# update device set tlinkgroupid= '0' where tlinkgroupid= '-1';
UPDATE 1
```

Customer needs to select tlink group ANY (0) upon adding the station/device in AES SBD in order to be able to monitor them.
### ACR — address not recognized by TSAPI for one extension
ACR12  'Address not recognized by TSAPI' error только для одного номера 8015592. SOLN314865.
Product Affected: Avaya Aura Workforce Optimization
Additional technical details and information related to the request:

```bash
2017-08-14 22:32:41,232 [TSAPI thread] DEBUG com.swhh.csta.a.ngetDeviceViaTsapi : 8015592 acceptexternal=false
2017-08-14 22:32:41,251 [TSAPI thread] INFO com.swhh.cs.a.p - Filtered out alarm
alarms.tsapi.addressunknown [8015592] [Query Device Info failed:
genericSubscribedResourceAvailability]
2017-08-14 22:32:41,252 [TSAPI thread] ERROR com.swhh.cscm.i.a - TsapiException:
TsapiException querying device 8015592 - Query Device Info failed:
genericSubscribedResourceAvailability
2017-08-14 22:32:41,252 [TSAPI thread] INFO com.swhh.cs.a.p - Filtered out alarm
alarms.tsapi.addressunknown [8015592] [null]
2017-08-14 22:32:47,261 [DMCC_poll_10.88.192.33] DEBUG com.swhh.d.g - sending
message: com.swhh.d.a.bt 5053
```

What is the software version/ Patch/service pack that you are currently using? 12
When did the issue start? 11 Aug 2017
How many are affected? One. Ext 8011592
Is this new installation? No
Any changes made to the system before the issue occurred? Extension wasmigrated from ACR v10 to v12. Extension has no issue in ACRv10

Причина: телефон H.323 с настроенным off-pbx mapping.

Add other tested stations with the same configuration in Off-pbxmappings. All good to observe.
Suspecting Corruption in CM station. BP didn't delete/re-add the station to change the UCID, but just removing the station from thepbx-mapping list.

Решение: удалить номер в off-pbx, и TSAPI определил и смог записывать.
### Ошибка Major Alarm Error running once a second tasks: 1
Не работал бэкап на SFTP.

![](image23.png)
### Denial event 1246: Svc obsrv exceed max
Не пишет разговоры, ошибка в `list trace station`:

```text
06:49:18     active station      7099 cid 0xd0 
06:49:18     Connected party   not in private table
06:49:18   denial event 1246: Svc obsrv exceed max D1=0x9fc4 D2=0xd0
06:49:18   denial event 1246: Svc obsrv exceed max D1=0x9fc4 D2=0xd0
06:49:18 SIP>SIP/2.0 200 OK 
06:49:18     Call-ID: e_5b869649-6aea9c1e61555j6j505j526h4j134441_I7
06:49:18     098
06:49:18     add service observer station      7998 cid 0xd0
```

Разрешить на 11 странице `ch sys fea` двух обозревателей звонка (нижняя строка):

```console title="ch sys fea" hl_lines="22"
change system-parameters features                               Page  11 of  19
                        FEATURE-RELATED SYSTEM PARAMETERS
CALL CENTER SYSTEM PARAMETERS
  EAS
         Expert Agent Selection (EAS) Enabled? n
        Minimum Agent-LoginID Password Length:
          Direct Agent Announcement Extension:                    Delay:
    Message Waiting Lamp Indicates Status For: station
                           Work Mode On Login: aux
  VECTORING
                    Converse First Data Delay: 0      Second Data Delay: 2
               Converse Signaling Tone (msec): 100         Pause (msec): 70
                     Prompting Timeout (secs): 10
                 Interflow-qpos EWT Threshold: 2
    Reverse Star/Pound Digit For Collect Step? n
          Available Agent Adjustments for BSR? n
                             BSR Tie Strategy: 1st-found
   Store VDN Name in Station's Local Call Log? n
  SERVICE OBSERVING
              Service Observing: Warning Tone? n     or Conference Tone? n
    Allowed with Exclusion: Service Observing? n                    SSC? n
             Allow Two Observers in Same Call? y
```

## Создание пользователя или сброс пароля веб-интерфейса в PostgreSQL
Зайти в Linux CLI сервера (виртуальной машины ACR) с логином **witness**. По умолчанию пароль так же witness, если не был задан другой пароль во время создания kickstart скрипта для установки RHEL7.

```bash
login as: witness
witness@ip-адрес's password:
Last login: Thu May 31 12:38:35 2018 from ip-адрес-откуда-захожу.
[witness@имя-хоста ~]$
```
### Лирическое отступление про права суперпользователя
Так же при создании kickstart скрипта задается пароль root-а (по умолчанию по идее должен быть ContactStore). Нельзя сразу зайти в Linux CLI под **root**-ом, сначала надо залогиниться как **witness**. То же самое сделано в AES, где сначала логинимся как **cust**.

Иногда при самостоятельной установке RHEL не используют kickstart скрипт, пользователю witness дают права выполнять все команды в файле настроек суперпользователя */etc/sudoers*, после чего можно получать права суперпользователя командой `sudo -s` и повторным вводом пароля от **witness**,а не root. **Как** это выглядит, видно в примере. В файле */etc/passwd* есть пользователи root, witness, rhel, postgres. Каждый из них является участником основной группы, которая создается автоматически и совпадает с именем пользователя.


```bash title="/etc/passwd" hl_lines="1 20 21 22"
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
systemd-network:x:192:192:systemd Network Management:/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
polkitd:x:999:997:User for polkitd:/:/sbin/nologin
postfix:x:89:89::/var/spool/postfix:/sbin/nologin
chrony:x:998:996::/var/lib/chrony:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
rhel:x:1000:1000:rhel:/home/rhel:/bin/bash
witness:x:1001:1001::/home/witness:/bin/bash
postgres:x:26:26:PostgreSQL Server:/var/lib/pgsql:/bin/bash
```

Внимание: пароль witness для входа по ssh указан в /etc/shadow в зашифрованном виде. В то же время мы можем завести для входа в веб-интерфейс пользователя **witness** с другим паролем, который будет указан в базе данных PostgreSQL. Т.е. в системе могут существовать два пользователя **witness**, которые не пересекаются.

Каждый пользователь может состоять в дополнительных группах кроме своей основной, что определяется в файле */etc/group*. Как видно, пользователи **rhel** и witness **дополнительно** состоят в группе wheel.

```bash title="/etc/group" hl_lines="11"
root:x:0:
bin:x:1:
daemon:x:2:
sys:x:3:
adm:x:4:
tty:x:5:
disk:x:6:
lp:x:7:
mem:x:8:
kmem:x:9:
wheel:x:10:rhel,witness
cdrom:x:11:
mail:x:12:postfix
...
```

**Wheel** — специальная группа администраторов, существующая воFreeBSD и некоторых версиях Linux. Имя wheel было аналогом учетнойзаписи root в операционной системе TOPS-20 (она же Tenex для DECPDP-10). Название группы не имеет отношения к дословному переводу wheel— колесо, а пришло из разговорного английского: wheel — big wheel— важный человек. Проверить, что у группы wheel есть права sudo, можно в файле */etc/sudoers*, где гибко настраиваются права суперпользователя. Открыть для редактирования можно как стандартным способом (естественно только с правами root), так и удобной командой `visudo`. Видно, что группе **wheel** разрешено выполнять все команды, как и пользователю root.Строка, начинающаяся со знака %, обозначает настройки группы, а неконкретного пользователя.

```bash title="/etc/sudoers"
## The COMMANDS section may have other options added to it.
##
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL

## Allows members of the 'sys' group to run networking, software,
## service management apps and more.
# %sys ALL = NETWORKING, SOFTWARE, SERVICES, STORAGE, DELEGATING, PROCESSES, LOCATE, DRIVERS


## Allows people in group wheel to run all commands
%wheel  ALL=(ALL)       ALL

## Same thing without a password
# %wheel        ALL=(ALL)       NOPASSWD: ALL
```

Таким образом, чтобы быстро дать пользователю права **sudo**, добавляем его в дополнительную группу **wheel**:

```bash
usermod -aG wheel имя_пользователя
```

Параметры -a и -G описаны в стандартной подсказке. Если мы точно знаем, что пользователь не состоит ни в каких дополнительных группах, достаточно `-G`. Чаще оба параметра используются вместе.
`-G`, `--groups ГРУППЫ`  список дополнительных ГРУПП
`-a`, `--append`   добавить пользователя в дополнительные ГРУППЫ, указанные в параметре -G не удаляя пользователя из других групп

Ниже информация о том, в каких группах состоит пользователь witness. Для информации по собственным группам залогиненного в текущий момент пользователя достаточно набрать `id` без указания имени. Uid (user id) — номер пользователя witness, gid (group id) —первичная (основная, главная) группа пользователя witness, groups —полный список групп пользователя witness, где основная группа указана первой в списке, а далее перечислены дополнительные группы.

Здесь пользователь witness состоит в группе wheel и может выполнять все команды root-а со своим собственным паролем, что удобно при установке пакетов программ для Avaya Contact Recorder.

Здесь пользователь witness не состоит в группе wheel и следовательно не может получить права суперпользователя со своим паролем командой sudo-s:

```bash
[witness@ACR ~]$ id
uid=1001(witness) gid=1001(witness) группы=1001(witness),10(wheel)

# Вот как мы получаем права суперпользователя состоя в группе wheel:
login as: witness
witness@имя-хоста's password:
Last login: Fri May 18 14:21:30 2018 from адрес-откуда-входим
[witness@ACR ~]$ sudo -s
[sudo] пароль для witness:                      пароль для witness
[root@ACR witness]#         все, мы можем выполнять команды от root-а
```

В моей инсталляции со скриптом kickstart в группе wheel никого не было, т.е. пользователь witness состоял только в своей основной группе и группе cdrom.

```bash title="/etc/group" hl_lines="11 12 23"
root:x:0:
bin:x:1:
daemon:x:2:
sys:x:3:
adm:x:4:
tty:x:5:
disk:x:6:
lp:x:7:
mem:x:8:
kmem:x:9:
wheel:x:10:
cdrom:x:11:witness
mail:x:12:postfix
man:x:15:
dialout:x:18:
floppy:x:19:
games:x:20:
tape:x:30:
....
stapsys:x:157:
stapdev:x:158:
tcpdump:x:72:
witness:x:1000:witness    # обычно тот пользователь, для кого группа является основной, здесь не указан. Но у меня такое отобразилосось в ACR.
postgres:x:26:
```

Пользователя rhel не было.
### Вход в PostgreSQL
Для залогинивания в базу данных PostgreSQL достаточно зайти под **witness** без получения дополнительных прав суперпользователя. Теперь заходим в базу с логином и паролем **eware**.

```bash
$ psql -U eware -h localhost
Password for user eware:  вводим пароль eware
psql (9.6.6)
Type "help" for help.
```

`psql` — команда входа в базу Postgresql из Linux CLI.
`-U` — имя пользователя, под которым хотим залогиниться в базе данных. У нас пользователь eware.
`-h` — hostname/IP-адрес сервера базы данных. Т.к. у нас база на том же сервере, где мы залогинены, пишем localhost или 127.0.0.1.

Убедиться, что у нас есть база **eware**, которую использует AvayaContact Recorder (ACR):

Вывести список всех баз командой `\list` или сокращенно `\l`

```sql
eware=> \list
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
-----------+----------+----------+-------------+-------------+-----------------------
 eware     | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
(4 rows)
```

Переключение между базами. Не удалось только в **template0** попасть. Префикс командной строки показывает, в какой базе мы сейчас находимся.

```sql
eware=> \connect postgres
You are now connected to database "postgres" as user "eware".
postgres=>
postgres=> \connect template0
FATAL:  database "template0" is not currently accepting connections
Previous connection kept
postgres=> \connect template1
You are now connected to database "template1" as user "eware".
template1=> \connect eware
You are now connected to database "eware" as user "eware".
eware=>
```

Выполним SQL запрос для отображения списка всех пользователей из таблицы **users**. Внимание: все команды должны оканчиваться точкой с запятой "; ". В примере ниже нет ни одного пользователя:

```sql
eware=> select * from users;
 userid | username | password |    comment    | email |         passwordsetat
--------+----------+----------+---------------+-------+------------------------------
      0 |          | x        | DO NOT DELETE |       | 2016-03-10 12:46:23.416852-07
(1 row)
```

Просто информация. Проверить флаги.

```sql
eware=> select * from licencetokens;
 tokenid |  tokenname   | tokenvalue 
---------+--------------+------------
       0 | expiry       |          0
       2 | qualitychans |          0
       3 | backupchans  |          0
       5 | replaychans  |          0
       7 | screenchans  |          0
       6 | monchans     |         56
       4 | fullchans    |          4
       1 | servertype   |          3
(8 rows)

eware=> select * from settings where settingkey like 'flags%';
        settingkey         |      value      
---------------------------+-----------------
 flags.licenseaccepted     | false
 flags.clientids           | 
 flags.licenseseen         | false
 flags.distribstandbyseen  | false
 flags.mainstandbyseen     | false
 flags.master              | 
 flags.controllers         | 
 flags.purgedinums         | 0
 flags.oldestarcpending    | 848033000030774
 flags.primary             | 
 flags.DMCC.адрес_AES      | true
 flags.TSAPI.адрес_AES     | true
(12 rows)
```

Остановить ACR. Зайти в psql как eware, выполнить:

```sql
eware=> delete from settings where settingkey='flags.oldestarcpending';
DELETE 1
```

Запустить ACR.
На рабочей системе:

```sql
eware=> select * from licencetokens;
 tokenid |  tokenname   | tokenvalue 
---------+--------------+------------
       2 | qualitychans |          0
       3 | backupchans  |          0
       5 | replaychans  |          0
       7 | screenchans  |          0
       6 | monchans     |         63
       0 | expiry       |      17757
       4 | fullchans    |         75
       1 | servertype   |          7
(8 строк)

eware=> select * from settings where settingkey like 'flags%';
        settingkey        | value 
--------------------------+-------
 flags.licenseaccepted    | false
 flags.clientids          | 
 flags.licenseseen        | false
 flags.distribstandbyseen | false
 flags.mainstandbyseen    | false
 flags.master             | 
 flags.controllers        | 
 flags.purgedinums        | 0
 flags.oldestarcpending   | 0
 flags.primary            | 
 flags.DMCC.172.31.43.16  | true
 flags.TSAPI.172.31.43.16 | true
(12 строк)
```

**Вопросы по базе данных**
1. If this DB postgres is used on all these servers?
   all server has its own database (slaves too), so 3 server means youhave 3 databases. One / ACR server. In your case the CRS is feeded bythe others trough a kind of SOAP interface.
2. If we can change the password for eware login?
   you can, but password has to be added to acr.properties (look fordb.password in PIA)
3. Apart from eware any other logins are there to access postgres DB?
   by default no, but on linux if you switch to postgres (from root, `su - postgres`) you can reach it without password (WaD, not tested if it hurt the operation)
4. Apart from postgres DB any other DB are there?
   No other DB used for ACR, if there is any other DB on the serverthat is not part of the ACR. Also in postgres only eware is used forACR, so no other db instance in postgres used by ACR.
5. When you say access of it not approved for non-certified engineersdo you mean it can only be access by Avaya Certified Engineers or Any Linux Certified Engineer?
   I meant: Avaya certified engineer.. so Linux engineers has to stayaway, despite they know the postgres
6. What information is stored on postgres DB?
   settings and calldetails like: audio/video recording details assegments (a call can have more than one audio + screen segments), forthe segments the system stores in DB the segment details: like parties,times, sizes, ID-s. theser are in different tables, does not store theRTP (audio/video payload),.. Also ACR store almost all settings there(some exception are in acr.properties), again in different tables.
### Создание аккаунта для веб-интерфейса ACR.
Обязательно убедиться, что номер аккаунта (userid) ещё не был задействован. В примере еще нет никакого аккаунта, поэтому добавляем аккаунт с номером 1. Проверим, что аккаунт создан.

```sql
eware=> insert into users (userid, username, comment) VALUES  ('1', 'witness', 'Customer Account');
INSERT 0 1

eware=> select * from users;
 userid | username | password |     comment      | email |         passwordsetat
--------+----------+----------+------------------+-------+------------------------------
      0 |          | x        | DO NOT DELETE    |       | 2016-03-10 12:46:23.416852-07
      1 | witness  |          | Customer Account |       | 
 (2 rows)
```
### Установка пароля
ACR шифрует только пароль, а не комбинацию логин-пароль. Поэтому для пользователя с любым именем можно сбросить пароль на указанный ниже.

Пароль **Pa\$\$w0rd** (в примере) в зашифрованном виде выглядит как **3cc31cd246149aec68079241e71e98f6**

Пароль **avaya1** в зашифрованном виде выглядит как **46a2f99c7cb2ecb846804569c39037b1**

```sql
eware=> update users set password = '3cc31cd246149aec68079241e71e98f6' where userid = '1';
```

Зададим дату создания пароля:

```sql
eware=> update users set passwordsetat = now() WHERE userid = '1';
```

Проверим, как теперь выглядит таблица пользователей users:

```sql
eware=> select * from users;
 userid | username | password                        |     comment      | email |         passwordsetat
--------+----------+---------------------------------+------------------+-------+------------------------------
      0 |          | x                               | DO NOT DELETE    |       | 2016-03-10 12:46:23.416852-07
      1 | witness  | 3cc31cd246149aec68079241e71e98f6| Customer Account |       | 2016-03-10 12:46:23.416852-07
 (2 rows)
```

Отобразим содержимое только таблицы **userpasswords**. Пока что онапустая:

```sql
eware=> select * from userpasswords;
 userid | password | setat
--------+----------+-------
(0 rows)
```

Если пароль успешно добавился в таблицу **users**, копируем из таблицы **users** значения полей **userid**, **passwordsetat** и **password** в таблицу **userpasswords**.

```sql
eware=> insert into userpasswords (userid, setat, password) select userid, passwordsetat, password from users where userid = '1';
INSERT 0 1
```

Примечание: для залогинивания пользователя через веб-интерфейс ACR надо обновить обе таблицы (users и userpasswords).

Повторно отобразим содержимое таблицы userpasswords. Теперь должно быть:

```sql
ewara=> select * from userpasswords;
 userid |             password             |             setat
--------+----------------------------------+-------------------------------
      1 | 3cc31cd246149aec68079241e71e98f6 | 2016-03-10 12:46:23.416852-07
(1 row)
```
### Сброс пароля пользователя
Процедура сброса схожа с добавлением нового пользователя за исключением того, что мы только *обновим* (update) существующие записи вместо вставки (insert into) новых записей. Сначала обновим таблицу users.

```sql
eware=> update users set password = '3cc31cd246149aec68079241e71e98f6' WHERE userid = '1';
```

Лучше добавить одинарные скобки вокруг номера userid.

Обновим дату создания пароля для нашего пользователя в таблице users:

```sql
eware=> update users set passwordsetat = now() WHERE userid = '1';
```

Скопируем поля password и passwordsetat из таблицы users и обновим поляpassword и setat таблицы userpasswords:

```sql
eware=> update userpasswords set password = (SELECT password FROM users WHERE userid = '1'), setat = (SELECT passwordsetat FROM users WHERE userid = 1) WHERE userid = '1';
UPDATE 1
```

Теперь проверим содержимое таблицы userpasswords, оно должно соответствовать содержимому users:

```sql
eware=> select * from userpasswords;
  userid |             password             |             setat
 --------+----------------------------------+-------------------------------
       1 | 3cc31cd246149aec68079241e71e98f6 | 2016-03-10 12:46:23.416852-07
(1 row)

eware=> select * from users;
 userid | username |              password            |     comment      | email |         passwordsetat
--------+----------+----------------------------------+------------------+-------+------------------------------
      0 |          | x                                | DO NOT DELETE    |       | 2016-03-10 12:46:23.416852-07
      1 | witness  | 3cc31cd246149aec68079241e71e98f6 | Customer Account |       | 2016-03-10 12:46:23.416852-07
(2 rows)
```

## Пример /etc/sudoers
Можно открыть командой visudo

```bash title="/etc/sudoers"
## Sudoers allows particular users to run various commands as
## the root user, without needing the root password.
##
## Examples are provided at the bottom of the file for collections
## of related commands, which can then be delegated out to particular
## users or groups.
##
## This file must be edited with the 'visudo' command.

## Host Aliases
## Groups of machines. You may prefer to use hostnames (perhaps using
## wildcards for entire domains) or IP addresses instead.
# Host_Alias     FILESERVERS = fs1, fs2
# Host_Alias     MAILSERVERS = smtp, smtp2

## User Aliases
## These aren't often necessary, as you can use regular groups
## (ie, from files, LDAP, NIS, etc) in this file - just use %groupname
## rather than USERALIAS
# User_Alias ADMINS = jsmith, mikem


## Command Aliases
## These are groups of related commands...

## Networking
# Cmnd_Alias NETWORKING = /sbin/route, /sbin/ifconfig, /bin/ping, /sbin/dhclient, /usr/bin/net, /sbin/iptables, /usr/bin/rfcomm, /usr/bin/wvdial, /sbin/iwconfig, /sbin/mii-tool

## Installation and management of software
# Cmnd_Alias SOFTWARE = /bin/rpm, /usr/bin/up2date, /usr/bin/yum

## Services
# Cmnd_Alias SERVICES = /sbin/service, /sbin/chkconfig, /usr/bin/systemctl start, /usr/bin/systemctl stop, /usr/bin/systemctl reload, /usr/bin/systemctl restart, /usr/bin/systemctl status, /usr/bin/systemctl enable, /usr/bin/systemctl disable

## Updating the locate database
# Cmnd_Alias LOCATE = /usr/bin/updatedb

## Storage
# Cmnd_Alias STORAGE = /sbin/fdisk, /sbin/sfdisk, /sbin/parted, /sbin/partprobe, /bin/mount, /bin/umount

## Delegating permissions
# Cmnd_Alias DELEGATING = /usr/sbin/visudo, /bin/chown, /bin/chmod, /bin/chgrp

## Processes
# Cmnd_Alias PROCESSES = /bin/nice, /bin/kill, /usr/bin/kill, /usr/bin/killall

## Drivers
# Cmnd_Alias DRIVERS = /sbin/modprobe

# Defaults specification

#
# Refuse to run if unable to disable echo on the tty.
#
Defaults   !visiblepw

#
# Preserving HOME has security implications since many programs
# use it when searching for configuration files. Note that HOME
# is already set when the the env_reset option is enabled, so
# this option is only effective for configurations where either
# env_reset is disabled or HOME is present in the env_keep list.
#
Defaults    always_set_home
Defaults    match_group_by_gid

Defaults    env_reset
Defaults    env_keep =  "COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS"
Defaults    env_keep += "MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE"
Defaults    env_keep += "LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES"
Defaults    env_keep += "LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE"
Defaults    env_keep += "LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY"

#
# Adding HOME to env_keep may enable a user to run unrestricted
# commands via sudo.
#
# Defaults   env_keep += "HOME"

Defaults    secure_path = /sbin:/bin:/usr/sbin:/usr/bin

## Next comes the main part: which users can run what software on
## which machines (the sudoers file can be shared between multiple
## systems).
## Syntax:
##
##      user    MACHINE=COMMANDS
##
## The COMMANDS section may have other options added to it.
##
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL

## Allows members of the 'sys' group to run networking, software,
## service management apps and more.
# %sys ALL = NETWORKING, SOFTWARE, SERVICES, STORAGE, DELEGATING, PROCESSES, LOCATE, DRIVERS

## Allows people in group wheel to run all commands
%wheel  ALL=(ALL)       ALL

## Same thing without a password
# %wheel        ALL=(ALL)       NOPASSWD: ALL

## Allows members of the users group to mount and unmount the
## cdrom as root
# %users  ALL=/sbin/mount /mnt/cdrom, /sbin/umount /mnt/cdrom

## Allows members of the users group to shutdown this system
# %users  localhost=/sbin/shutdown -h now

## Read drop-in files from /etc/sudoers.d (the # here does not mean a comment)
#includedir /etc/sudoers.d
```