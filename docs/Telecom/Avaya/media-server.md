---
title: Avaya Aura Media Server (виртуальный шлюз G450)
share: "true"
---

## Заливка патчей AMS
need patches staged for the customer. Нужен доступ root
1. Переслать файлы .zip на сервер AMS, можно утилитой WinSCP.
2. Подключиться к серверу по ssh с логином craft используя EASG (сначала включить, если выключен)  и перейти в пользователя sroot (`su – sroot`)
3. Скопировать все zip файлы в каталог `$MASHOME/qfe`
4.  Выполнить `amspatch apply all` для установки всех QFE’s
## Установка обновлений из Linux CLI

```bash  title="stageUpdate"
[cust@sng-nv-abk3-ucams3 ~]$ stageUpdate -hh

This script provides a command line method of managing the staging of
updates for this mediaserver appliance. Updates are delivered separately for
the mediaserver application and the system layer (operating system). An update
must be staged before it can be installed. Staging can be performed
prior to the installation maintenance window.

The staging process verifies the integrity and authenticity of an update,
and passing these checks, consumes the update content into internal storage
areas, making it available for later installation. An update will be
rejected if it fails these checks. The staging process does not apply any
changes to the system. To install updates that have been staged, use
the following command:

    installUpdate

More help on the above command is available as follows:
    installUpdate -help

The following command line arguments are available for this command.

-h     Provides terse help information (invocation syntax).
-hh    Provides verbose help information (this help information).
-upload <filename> Copies the given filename into the internal upload directory.

-stage [ -f <filename> ] [ -verifyOnly ]
        Stages the file uploaded by the -upload argument. The upload and staging step can be combined by using the optional
       "-f <filename>" argument. The staging process verifies the integrity and authenticity of the update. The optional argument "-verifyOnly" allows the update to be verified without actually being staged onto the system (consumed).

    -list
        Lists the currently uploaded and staged updates.

    -clean < -uploaded | -staged [ ams | sys ] | -all >
        Deletes the stored content in the upload area, or staged contents for the given update type. If "-all" is specified, then all upload and staged content is deleted.
```

Закачать из WinSCP в */opt/avaya/app/pub* патч `MediaServer_Update_7.8.0.410_2018.09.12.iso`
Перейти в публичный каталог с помощью алиаса **cdpub** или полной команды. Установить из веб-интерфейса или из командной строки:

```bash
cd /opt/avaya/app/pub
stageUpdate -stage -f MediaServer_Update_7.8.0.410_2018.09.12.iso
installUpdage      # без аргументов
```
### Не удается установить обновление
Как получить лог, который можно скопировать на свой компьютер через WinSCP.
```bash
$ sysTool -e
Generated file: sng-nv-abk3-ucams3-event-log.zip

[cust@sng-nv-abk3-ucams3 pub]$ ls
MediaServer_System_Update_7.8.0.16_2018.09.14.iso
MediaServer_Update_7.8.0.410_2018.09.12.iso
sng-nv-abk3-ucams3-event-log.zip
```
## Установка обновлений из веб-интерфейса
*Tools→ Manage Software→ Updates→ Choose File* (`MediaServer_System_Update_7.8.0.16_2018.09.14 .iso`)→ *Upload*. При закачивании принять лицензионное соглашение. Аналогично закачиваем на сервер `MediaServer_Update_7.8.0.410_2018.09.12.iso`

Остановить обработку новых сессий: *System Status→ Element Status*,  в выпадающем меню *More Actions* выбрать *Lock→ Confirm*.

Для обновления возвращаемся в *Tools→ Manage Software→ Updates*. В правом нижнем углу жмем *Install Updates*. Появится предупреждение о том, что обновление займет примерно 7 минут.
После установки *System Status→ Element Status*,  в выпадающем меню *More Actions* выбрать *Unlock→ Confirm*.
## Пароль по умолчанию AMS 8
Вход в веб-интерфейс https://ipAddress:8443/emlogin
Username: `admin`  
Password: `Admin123$`
## Настройка
*System Configuration→ Network Settings→ IP Interface Assignment*

В разделе IPv4 Interfaces для Signaling, Media и Cluster выбрать `10.8.13.8 [eth0]`, для OAM `0.0.0.0` All Network Interfaces. *Save→ Confirm*. Для вступления в силу требуется перезагрузка медиасервера.

*Licensing →  General Settings* вместо `localhost` установить `10.8.12.147`.
## Установка сертификатов
*Security→ Certificate Management→ Trust Store*. Нажать *Import* и выбрать в проводнике файл **SystemManagerCA.cacert.pem**. Нажать *Upload*. Задать имя *System Manager* (у меня sng-nv-abk1-ucasmgr1) → *Save→ Restart Now*.

*Security→ System Manager→ Advanced Settings*
Во время Enrollment

**Certificate Fields**
Certificate friendly name:  sng-nv-abk3-ucams3_sng-nv-abk1-ucasmgr1_signed
Key bit length:
Signature algorithm:  SHA256
Organization:  ROSNEFT
Organization unit: MGMT
Common name: sng-nv-abk3-ucams3.smn.rosneft.ru
Country (ISO 3166): RU
State/Province (full name): KhMAO
City/Locality: Nizhnevartovsk
Include Subject Alternative Name with IP address  10.8.13.8
Include Subject Alternative Name with FQDN    sng-nv-abk3-ucams3.smn.rosneft.ru

**Trust Management**
Provide the enrollment password that the media server must use to acquire a System Manager-signed certificate from System Manager Trust Management.

System Manager trust management enrollment password:
••••••••
Confirm password:
••••••••
System Manager Enrollment

Step 4 of 4: Verify the System Manager enrollment information.
Summary

The following tasks are performed when you enroll:
The system stores System Manager parameters in the media servers.
The system acquires the System Manager-signed certificate and stores it in the media server key store for use by the OAM and EM service profiles.
The Element Manager authentication and authorization source is set to System Manager.
Management related services will be restarted to apply the changes.
The media server node and cluster configuration are added o the configured System Manager server.

**Cluster and Elements**
The administrative names and descriptions of the media server cluster and elements.

Cluster administrative name: AAMS3
Cluster administrative description: Media Server 3
Server Name Server Role Element Administrative Name Element Administrative Description
sng-nv-abk3-ucams3 Primary AAMS3 Media Server 3

**System Manager Configuration**
Fully qualified domain name (FQDN) of System Manager server: sng-nv-abk1-ucasmgr1.smn.rosneft.ru
System Manager server port: 443
System Manager registration username: adminnn@smn.rosneft.ru
System Manager registration password: тут_пароль

**Certificate Setup**
The certificate fields for a new System Manager-signed certificate are listed below.

Certificate friendly name: sng-nv-abk3-ucams3_sng-nv-abk1-ucasmgr1_signed
Key bit length: 2048
Signature algorithm: SHA256
Organization: ROSNEFT
Organization unit: MGMT
Common name: sng-nv-abk3-ucams3.smn.rosneft.ru
Country (ISO 3166): RU
State/Province (full name): KhMAO
City/Locality: Nizhnevartovsk
Subject Alternative Name: 10.8.13.8, sng-nv-abk3-ucams3.smn.rosneft.ru
System Manager trust management enrollment password: тут_пароль

Нажать *Enroll*. По окончании процесса появится сообщение:

>The System Manager enrollment is complete.
>All Element Manager sessions will end momentarily. You must close your Element Manager browser window or tab. You can connect to Element Manager again in about three minutes.

Теперь из SMGR разрешим Communication Manager доступ к ресурсам Media Server:
*Media Server→ Application Assignment*, поставить галочку *Communication Manager* в строке соответствующего имени кластера, который создавали в *Enrollment*, нажать *Commit*.
## Включаем доступ для Avaya

```bash
$ EASGManage --enableEASG
By enabling Avaya Services Logins you are granting Avaya access to
your system.  This is required to maximize the performance and value
of your Avaya support entitlements, allowing Avaya to resolve product
issues in a timely manner.

The product must be registered using the Avaya Global Registration
Tool (GRT, see https://grt.avaya.com) to be eligible for Avaya remote
connectivity.  Please see the Avaya support site (https://support.avaya.com/
registration) for additional information for registering products and
establishing remote access and alarming.

Do you want to continue [yes/no]? yes

EASG Access is enabled. Performed by user ID: 'cust', on Oct 17 2018 - 15:04
```