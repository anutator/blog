---
title: CMS — Call Management System
share: "true"
---

Eth0 прикрепляется к Network Adapter 4.
После установки входим как root, пароля нет! Пароль задать самостоятельно:

```bash
# passwd
Changing password for user root.
New password: ввести
Retype new password: ввести
passwd: all authentication tokens updated successfully.
```

Требования для виртуальных машин:
1. Small configuration: The CMS OVA for the small configuration is based on the medium configuration. Provide a minimum of 600 GB of physical disk space even though the small configuration storage reservation is only 200 GB.
2. Large configuration: See Requirements for expanding large configurations.

При установке виртуальной машины из `CMS-R18.0.2.0-ma.k-e65-00.ova` никакие данные не вводим (ни имя хоста, ни ip-адреса и пр.). Все это потом при входе из консоли.
Включаем виртуальную машину. Заходим в vCenter в виртуальную консоль. Делаем сетевые настройки, выполняем скрипт:

```bash
/cms/toolsbin/netconfig
```

Во время выполнения задаем все необходимые параметры:

```
(localhost.localdomain)-(root)=# /cms/toolsbin/netconfig
 WARNING: This tool only supports IPv4
 Enter the network interface name from following name(s): eth0 eth1 eth2 eth3 (default eth0)
 ENTER>    просто жмем Enter, т.к. соглашаемся с тем что по умолчанию
 You have entered [ eth0 ]. Is this correct? (y|n) y

 Enter the host name of the CMS system 
 ENTER> mrc-krl15-ucacms1
 You have entered [ mrc-krl15-ucacms1 ]. Is this correct? (y|n) y

 Enter the domain name of the CMS system
 ENTER> rtrn.ru
 You have entered [ rtrn.ru ]. Is this correct? (y|n) y

 Enter the IP address of the network interface
 ENTER> 172.21.103.17
 You have entered [ 172.21.103.17 ]. Is this correct? (y|n) y

 Enter the netmask for the subnet of the network interface
 ENTER> 255.255.255.192
 You have entered [ 255.255.255.192 ]. Is this correct? (y|n) y

 Enter the gateway for the network interface
 ENTER> 172.21.103.1
 You have entered [ 172.21.103.1 ]. Is this correct? (y|n) y
 Enter the DNS server(s) separated by space (up to three servers)
 ENTER> 172.31.42.14 172.21.40.6
 You have entered [ 172.31.42.14 172.21.40.6 ]. Is this correct? (y|n) y

 Enter the search domains separated by space (press enter for none)
 ENTER>  просто жмем Enter
 You have entered [  ]. Is this correct? (y|n) y
  Interface: eth0
  CMS Hostname: mrc-krl15-ucacms1
  Domainname: rtrn.ru
  CMS IP address: 172.21.103.17
  Netmask: 255.255.255.192
  Gateway: 172.21.103.1
  DNS Server1: 172.31.42.14
  DNS Server2: 172.21.40.6
  DNS Server3:
  Search domains:

  Are the above inputs correct? (y|n)y

 Bring the network up. Please wait...

Tue Mar 20 13:47:32 EDT 2018 /cms/toolsbin/netconfig successfully finished
```

Выполняем файл, в котором экспортируются переменные окружения в Linux:

```bash
. /opt/informix/bin/setenv
```

Выполняем скрипт инициализации базы данных

```bash
/opt/informix/bin/dbinit.sh
```

Обязательно смотрим на результат выполнения скрипта. Если скрипт выполнен неуспешно (failed), запустить повторно.
Пример успешного выполнения скрита. Достаточно один раз ввести **y** и наблюдать за текстом на экране:

```
(mrc-krl15-ucacms1)-(root)=# /opt/informix/bin/dbinit.sh
WARNING: /opt/informix/bin/dbinit.sh will initialize CMS database.
WARNING: All data will be lost!!!
Do you want to continue? (y or n) : y
Wed Mar 21 10:11:01 EDT 2018 Creating CMS database started
Wed Mar 21 10:11:01 EDT 2018 disk partition for CMS informix: /dev/sda11
Wed Mar 21 10:11:01 EDT 2018 Repartition the disk for VMware CMS
Wed Mar 21 10:11:01 EDT 2018 The new disk size: 1258291199 sectors
Wed Mar 21 10:11:02 EDT 2018 The old disk size: 1258291199 sectors
Wed Mar 21 10:11:02 EDT 2018 No need to repartition the disk
Wed Mar 21 10:11:02 EDT 2018 raw disk path: /dev/raw/raw1
Wed Mar 21 10:11:02 EDT 2018 Initializing Informix IDS started
Wed Mar 21 10:11:09 EDT 2018 Checking kernel parameters
Wed Mar 21 10:11:09 EDT 2018 Max shared Memory size: 16333324 KB
kernel.sem = 250 256000 32 4096
Reading configuration file '/opt/informix/etc/onconfig.cms'...succeeded
Creating /INFORMIXTMP/.infxdirs...succeeded
Allocating and attaching to shared memory...succeeded
Creating resident pool 4306 kbytes...succeeded
Creating infos file "/opt/informix/etc/.infos.cms_ol"...succeeded
Linking conf file "/opt/informix/etc/.conf.cms_ol"...succeeded
Initializing rhead structure...rhlock_t 16384 (512K)... rlock_t (2656K)... Writing to infos file...succeeded
Initialization of Encryption...succeeded
Initializing ASF...succeeded
Initializing Dictionary Cache and SPL Routine Cache...succeeded
Bringing up ADM VP...succeeded
Creating VP classes...succeeded
Forking main_loop thread...succeeded
Initializing DR structures...succeeded
Forking 1 'ipcstr' listener threads...succeeded
Forking 1 'ipcstr' listener threads...succeeded
Forking 1 'soctcp' listener threads...succeeded
Forking 1 'soctcp' listener threads...succeeded
Starting tracing...succeeded
Initializing 16 flushers...succeeded
Initializing log/checkpoint information...succeeded
Initializing dbspaces...succeeded
Opening primary chunks...succeeded
Validating chunks...succeeded
Creating database partition...succeeded
Initialize Async Log Flusher...succeeded
Starting B-tree Scanner...succeeded
Initializing DBSPACETEMP list...succeeded
Init ReadAhead Daemon...succeeded
Init Auto Tuning Daemon...succeeded
Checking database partition index...succeeded
Initializing dataskip structure...succeeded
Checking for temporary tables to drop...succeeded
Updating Global Row Counter...succeeded
Forking onmode_mon thread...succeeded
Creating periodic thread...succeeded
Creating periodic thread...succeeded
Starting scheduling system...succeeded
Verbose output complete: mode = 5
Wed Mar 21 10:11:37 EDT 2018 Initializing Informix IDS successfully finished
Wed Mar 21 10:11:37 EDT 2018 Creating CMS dbspaces started
Verifying physical disk space, please wait ...
Space successfully added.

** WARNING **  A level 0 archive of Root DBSpace will need to be done.
Verifying physical disk space, please wait ...
Space successfully added.

** WARNING **  A level 0 archive of Root DBSpace will need to be done.
Verifying physical disk space, please wait ...
Space successfully added.

** WARNING **  A level 0 archive of Root DBSpace will need to be done.
Verifying physical disk space, please wait ...
Space successfully added.

** WARNING **  A level 0 archive of Root DBSpace will need to be done.
Log operation started. To monitor progress, use the onstat -l command.
** WARNING ** Because the physical log has been modified, a level 0 archive
must be taken of the following spaces before an incremental archive will be
permitted for them: rootdbs physdbs
(see Dynamic Server Administrator's manual)
Log operation started. To monitor progress, use the onstat -l command.
Logical log successfully added.
Log operation started. To monitor progress, use the onstat -l command.
Logical log successfully added.
Log operation started. To monitor progress, use the onstat -l command.
Logical log successfully added.
Logical log file 1 has been pre-dropped.
It will be deleted from the log list and its space can be reused
once you take level 0 archives of all BLOBspaces, Smart BLOBspaces
and non-temporary DBspaces.
Logical log file 2 has been pre-dropped.
It will be deleted from the log list and its space can be reused
once you take level 0 archives of all BLOBspaces, Smart BLOBspaces
and non-temporary DBspaces.
Logical log file 3 has been pre-dropped.
It will be deleted from the log list and its space can be reused
once you take level 0 archives of all BLOBspaces, Smart BLOBspaces
and non-temporary DBspaces.
Wed Mar 21 10:11:44 EDT 2018 Adding disks to cmsdbs started
Wed Mar 21 10:11:44 EDT 2018 New disk(s) were successfully added to cmsdbs
Wed Mar 21 10:11:45 EDT 2018 Creating CMS dbspaces successfully finished
Wed Mar 21 10:11:45 EDT 2018 Configuring tape drive to /cmstape
Wed Mar 21 10:11:45 EDT 2018 Disabling IDS Automatic Statistics Updating
Wed Mar 21 10:11:45 EDT 2018 Disabling IDS Automatic Statistics Updating successfully finished
Wed Mar 21 10:11:45 EDT 2018 Creating CMS database successfully finished
```

Заходим в меню cmssvc. Пункты при успешной установке должны отобразиться сразу без ожидания. Для выбора пунктов меню нажимаем соответствующий номер.

``` hl_lines="6"
(mrc-krl15-ucacms1)-(root)=# cmssvc

 Avaya(TM) Call Management System Services Menu

Select a command from the list below.
   1) auth_display Display feature authorizations
   2) auth_set     Authorize capabilities/capacities
   3) run_ids      Turn Informix Database on or off
   4) run_cms      Turn Avaya CMS on or off
   5) setup        Set up the initial configuration
   6) swinfo       Display switch information
   7) swsetup      Change switch information
   8) uninstall    Remove the CMS rpm from the machine
   9) patch_rmv    Backout an installed CMS patch
  10) back_all     Backout all installed CMS patches from machine
  11) security     Administer CMS security features
Enter choice (1-11) or q to quit: 1
                                     Capability/Capacity   Authorization
                                     -------------------   -------------
                                            CMS hardware   not authorized
                                               vectoring   not authorized
                                             forecasting   not authorized
                                                graphics   not authorized
                                   external call history   not authorized
                                  expert agent selection   not authorized
                                    external application   not authorized
                            global dictionary/ACD groups   not authorized
                                           multi-tenancy   not authorized
                                                 Dual IP   authorized
                                    Avaya CMS Supervisor   not authorized
                                   Avaya Report Designer   not authorized
                   Maximum number of split/skill members   0
                                  Maximum number of ACDs   1
                Simultaneous Avaya CMS Supervisor logins   0
                       Number of authorized agents (RTU)   not authorized
                   Number of authorized ODBC connections   0
                                         FIPS 140-2 mode   off
                                                Firewall   inconsistent
```

После перезагрузки Linux скорее всего будет ожидание при вводе cmssvc. Надо включить Informix Database (включить CMS не получится до открытия лицензий).

``` hl_lines="11"
# cmssvc
cmssvc: Warning IDS off-line.  It will take approx 45 seconds to
start cmssvc.  IDS can be turned on with the run_ids command on
the cmssvc menu.

 Avaya(TM) Call Management System Services Menu

Select a command from the list below.
   1) auth_display Display feature authorizations
   2) auth_set     Authorize capabilities/capacities
   3) run_ids      Turn Informix Database on or off
   4) run_cms      Turn Avaya CMS on or off
   5) setup        Set up the initial configuration
   6) swinfo       Display switch information
   7) swsetup      Change switch information
   8) uninstall    Remove the CMS rpm from the machine
   9) patch_rmv    Backout an installed CMS patch
  10) back_all     Backout all installed CMS patches from machine
  11) security     Administer CMS security features
Enter choice (1-11) or q to quit: 3

Select one of the following
  1) Turn on IDS
  2) Turn off IDS
Enter choice (1-2): 1
```

Проверка `tail /cms/install/logdir/admin.log`

```bash title="/cms/install/logdir/admin.log"
(mrc-krl15-ucacms1)-(root)=# cat /cms/install/logdir/admin.log
Fri Jan 26 00:53:58 EST 2018 Modifying VPCLASS in /opt/informix/etc/onconfig.cms
Fri Jan 26 00:53:58 EST 2018 Changed SHMTOTAL setting in
   /opt/informix/etc/onconfig.cms from 0 to 13996705
Fri Jan 26 00:53:58 EST 2018 Modifying DS_TOTAL_MEMORY in /opt/informix/etc/onconfig.cms
Fri Jan 26 00:53:58 EST 2018 Modifying 8K BUFFERPOOL to 514584 in /opt/informix/etc/onconfig.cms
Fri Jan 26 00:53:58 EST 2018 Modifying 2K BUFFERPOOL to 12800 in /opt/informix/etc/onconfig.cms
Dual IP feature automatically authorized.
FIPS feature automatically authorized.
Database not set up, no upgrades run
Copied /cms/toolsbin/age_pw_exclude_template to /cms/db/age_pw_exclude
Begin modifications to ssh configuration files for security.
A copy of /etc/ssh/ssh_config and /etc/ssh/sshd_config has been made.
Modification of /etc/ssh/ssh_config and /etc/ssh/sshd_config to Ciphers and Macs is complete.
CMS Version r18ma.k installation successful Fri Jan 26 00:54:00 EST 2018
Wed Mar 21 09:57:39 EDT 2018 /cms/toolsbin/netconfig started
### old /etc/sysconfig/network-scripts/ifcfg-eth0:
DEVICE=eth0
HWADDR=00:50:56:92:BA:9C
TYPE=Ethernet
UUID=5974d974-8c93-4788-88d7-e51eb2be871a
ONBOOT=no
NM_CONTROLLED=yes
BOOTPROTO=dhcp
### new /etc/sysconfig/network-scripts/ifcfg-eth0:
DEVICE=eth0
TYPE=Ethernet
NM_CONTROLLED=yes
UUID="80a36e4f-d63e-4921-a370-66e41c60feda"
ONBOOT="yes"
BOOTPROTO="static"
HWADDR="00:50:56:8C:4F:44"
DOMAIN="rtrn.ru"
IPADDR="172.21.103.17"
NETMASK="255.255.255.192"
GATEWAY="172.21.103.1"
DNS1="172.31.42.14"
DNS2="172.21.40.6"
Wed Mar 21 10:01:55 EDT 2018 ifup eth0
Determining if ip address 172.21.103.17 is already in use for device eth0...
Wed Mar 21 10:01:59 EDT 2018 /cms/toolsbin/netconfig successfully finished

Wed Mar 21 13:58:51 MSK 2018 Changed SHMTOTAL setting in
   /opt/informix/etc/onconfig.cms from 13996705 to 13883325
Wed Mar 21 13:58:51 MSK 2018 Modifying DS_TOTAL_MEMORY in /opt/informix/etc/onconfig.cms
Wed Mar 21 13:58:51 MSK 2018 Modifying 8K BUFFERPOOL to 510416 in /opt/informix/etc/onconfig.cms
```

Перезагрузка CMS:

```bash
shutdown --r now                # Linux
/usr/sbin/shutdown -i6 -g0 -y   # Solaris
```
## Установка лицензии сотрудником Avaya
После установки CMS даем доступ сотруднику Avaya для активации лицензии. Пришлют LMI ссылку, дающую доступ к моему компьютеру. Я захожу через PuTTY в CMS. Далее делает сотрудник Avaya.

``` hl_lines="7 19"
(mrc-krl15-ucacms1)-(root)=# cmssvc

 Avaya(TM) Call Management System Services Menu

Select a command from the list below.
   1) auth_display Display feature authorizations
   2) auth_set     Authorize capabilities/capacities
   3) run_ids      Turn Informix Database on or off
   4) run_cms      Turn Avaya CMS on or off
   5) setup        Set up the initial configuration
   6) swinfo       Display switch information
   7) swsetup      Change switch information
   8) uninstall    Remove the CMS rpm from the machine
   9) patch_rmv    Backout an installed CMS patch
  10) back_all     Backout all installed CMS patches from machine
  11) security     Administer CMS security features
Enter choice (1-11) or q to quit: 2

Password: пароль вводит сотрудник AVAYA

Authorize installation of forecasting package? (y/n): (default: n) y
Authorize installation of vectoring package? (y/n): (default: n) y
Authorize use of graphics feature? (y/n): (default: n) y
Authorize use of external call history feature? (y/n): (default: n) y
Authorize use of expert agent selection feature? (y/n): (default: n) y
Authorize use of external application feature? (y/n): (default: n) y
Authorize use of global dictionary/ACD groups feature? (y/n): (default: n) y
Enter the number of simultaneous Avaya CMS Supervisor logins the customer has purchased (2-1600): (default: 2) 5
Has the customer purchased Avaya Report Designer? (y/n): (default: n) y
Enter the maximum number of split/skill members that can be administered (1-800000): 480
Enter the maximum number of ACDs that can be installed (1-8): (default: 1) 1
Enter the number of authorized agents (Right To Use): 4
Enter the number of authorized ODBC connection (0-10): (default: 0) 

# cmssvc
 Avaya(TM) Call Management System Services Menu

Select a command from the list below.
   1) auth_display Display feature authorizations
   2) auth_set     Authorize capabilities/capacities
   3) run_ids      Turn Informix Database on or off
   4) run_cms      Turn Avaya CMS on or off
   5) setup        Set up the initial configuration
   6) swinfo       Display switch information
   7) swsetup      Change switch information
   8) uninstall    Remove the CMS rpm from the machine
   9) patch_rmv    Backout an installed CMS patch
  10) back_all     Backout all installed CMS patches from machine
  11) security     Administer CMS security features
Enter choice (1-11) or q to quit: 1
	                             Capability/Capacity   Authorization
	                             -------------------   -------------
	                                    CMS hardware   authorized
	                                       vectoring   authorized
	                                     forecasting   authorized
	                                        graphics   authorized
	                           external call history   authorized
	                          expert agent selection   authorized
	                            external application   authorized
	                    global dictionary/ACD groups   authorized
	                                   multi-tenancy   authorized
	                                         Dual IP   authorized
	                            Avaya CMS Supervisor   authorized
	                           Avaya Report Designer   authorized
	           Maximum number of split/skill members   480
	                          Maximum number of ACDs   1
	        Simultaneous Avaya CMS Supervisor logins   5
	               Number of authorized agents (RTU)   4
	           Number of authorized ODBC connections   0
	                                 FIPS 140-2 mode   off
	                                        Firewall   inconsistent
```

Настройка CMS для линка с Communication Manager. Cmssvc→ swsetup (пункт 7) или cmsadm→ acd_create

```shellsession hl_lines="12"
(mrc-krl15-ucacms1)-(root)=# cmssvc

 Avaya(TM) Call Management System Services Menu

Select a command from the list below.
   1) auth_display Display feature authorizations
   2) auth_set     Authorize capabilities/capacities
   3) run_ids      Turn Informix Database on or off
   4) run_cms      Turn Avaya CMS on or off
   5) setup        Set up the initial configuration
   6) swinfo       Display switch information
   7) swsetup      Change switch information
   8) uninstall    Remove the CMS rpm from the machine
   9) patch_rmv    Backout an installed CMS patch
  10) back_all     Backout all installed CMS patches from machine
  11) security     Administer CMS security features
Enter choice (1-11) or q to quit: 5

Select the language for this server:

All languages are ISO Latin except Japanese. Selection of the
server language assumes that existing customer data is compatible.
(Upgrade from any ISO Latin language to any ISO Latin language
or from Japanese to Japanese is supported).

   1) English
   2) Dutch
   3) French
   4) German
   5) Italian
   6) Portuguese
   7) Spanish
   8) Japanese
Enter choice (1-8): (default: 1) 1

## Initializing Customer CMS data . . .
.......................
Customer CMS data successfully initialized.

Enter a name for this UNIX system (up to 64 characters): (default: mrc-krl15-ucacms1)

Select the type of backup device you are using
   1) Tape
   2) Other
Enter choice (1-2): 2

Enter the default backup device path: (default: 'none') /storage/backup
Invalid backup device path.

Enter the default backup device path: (default: 'none') /storage/backup

Enter number of ACDs being administered (1-1): (default: 1)

Information for ACD 1

Enter switch name (up to 20 characters): ucacm

Select the model of switch for this ACD
   1) Communication Mgr 5.2
   2) Communication Mgr 6.x
   3) Communication Mgr 7.x
Enter choice (1-3): 3
Is Vectoring enabled on the switch? (y/n): y
Is Expert Agent Selection enabled on the switch? (y/n): y
Does the Central Office have disconnect supervision? (y/n): (default: y) y
Enter the local port assigned to switch (1-64): 1
Enter the remote port assigned to switch (1-64): 1
Select the transport to the switch
   1) TCP/IP
Enter choice (1-1): 1
Enter switch host name or IP Address: mrc-krl15-ucacm
Enter switch TCP port number (5001-5999): (default: 5001)
Number of splits/skills (0-8000): (default: 350) 10
Total split/skill members, summed over all splits/skills (0-480): (default: 480)
Number of shifts (1-4): (default: 1)
Enter the start time for shift 1 (hh:mmXM): (default: 8:00 AM)
Enter the stop time for shift 1 (hh:mmXM): (default: 5:00 PM)
Number of agents logged into all splits/skills during shift 1 (0-480): (default: 480)
Number of trunk groups (0-2000): (default: 350)
Number of trunks (0-24000): (default: 1000)
Number of unmeasured facilities (0-12000): (default: 500)
Number of call work codes (1-1999): (default: 750)
Enter number of vectors (0-8000): (default: 350)
Enter number of VDNs (0-30000): (default: 2000)
Updating database.
Computing space requirements and dbspace availability.
Setup completed successfully.
```

Изменение настроек cmssvc—7 (swsetup)

```shellsession
…
Enter switch TCP port number (5001-5999): (default: 5001)
Switch administration for acd 1:
        Switch name: ucacm
        Switch model: Communication Mgr 7.x
        Vectoring: y
        Expert Agent Selection: y
        Central office disconnect supervision: y
        Local port: 1
        Remote port: 1
        Link: TCP/IP mrc-krl15-ucacm 5001

WARNING: Once you confirm this new switch administration, it may
not be possible to restore the previous administration without loss of data.

Is the above switch administration correct? (y/n): y

Switch configuration changed successfully. At this point
you should turn on CMS, go to the "Data Storage Allocation"
screen, and verify/modify the current administration.
You should also go to the "Free Space Allocation" screen
and verify/modify your existing free space.
```

Включение CMS

```shellsession
(mrc-krl15-ucacms1)-(root)=# cmssvc

 Avaya(TM) Call Management System Services Menu

Select a command from the list below.
   1) auth_display Display feature authorizations
   2) auth_set     Authorize capabilities/capacities
   3) run_ids      Turn Informix Database on or off
   4) run_cms      Turn Avaya CMS on or off
   5) setup        Set up the initial configuration
   6) swinfo       Display switch information
   7) swsetup      Change switch information
   8) uninstall    Remove the CMS rpm from the machine
   9) patch_rmv    Backout an installed CMS patch
  10) back_all     Backout all installed CMS patches from machine
  11) security     Administer CMS security features
Enter choice (1-11) or q to quit: 4

Select one of the following
  1) Turn on CMS
  2) Turn off CMS but Leave IDS running
  3) Turn off both CMS and IDS
Enter choice (1-3): 1

Please wait for initialization
. .

*** CMS is now up  ***
```

Включение веб-интерфейса по порту 8443

```
(mrc-krl15-ucacms1)-(root)=# cmsweb status
cmsweb is stopped

(mrc-krl15-ucacms1)-(root)=# cmsweb start
starting cmsweb ...

(mrc-krl15-ucacms1)-(root)=# cmsweb status
cmsweb is running
```

## Сбор логов CMS
SPI — язык Switch Protocol Interpreter (SPI), который использует Communication Manager для установки соединения с приложениями, по которым собираем отчеты.

| Версия CMS<br><br>ch sys fea, page 12, пункт CMS (appl mis): | Avaya IQ release setting<br><br>ch sys fea, page 12, пункт AAPC/IQ (appl ccr): | Версия языка SPI |
| ------------------------------------------------------------ | ------------------------------------------------------------------------------ | ---------------- |
| R15/R16                                                      | 5.0                                                                            | 22               |
| R16.1/R16.x/R17.0.                                           | 5.1/5.2                                                                        | 23               |
| R18                                                          | 5.2.7+                                                                         | 24               |

Всего CMS может собирать логи с восьми телефонных станций. Они настраиваются в CMS как ACD1, ACD2 .. ACD8. А в логе CMS линки с ACD соответствуют SPI 1-8.

В формате команды используем полный путь к исполняемому файлу spilog, как указано ниже, либо добавляем этот путь в переменную окружения PATH.

```bash
/cms/bin/spilog acd# [[+|-]trace]
```
Здесь tracе может быть таким: err, xln, adm, aud, tk, ag, ign, call, sw, sess. Если вводим all, то включаем все виды трассировки. Можно включить все виды, а потом исключить один из них.
Активируем сбор всех видов логов для первого ACD (у нас он единстевнный Manager):
```bash
# /cms/bin/spilog 1 all
SPI 1: err+xln+adm+aud+tk+ag+ign+call+sw
```

Исключаем из этого списка логи aud:
```bash
# /cms/bin/spilog 1 -aud
SPI 1: err+xln+adm+tk+ag+ign+call+sw     # aud пропала из строки
```

Для просмотра статуса логов вводим команду без аргументов. Здесь включены все логи, но такие навороченные системы, где CMS собирает статистику аж с восьми телефонных станций, думаю, крайне редки.

```
# /cms/bin/spilog
SPI 1: err+xln+adm+aud+tk+ag+ign+call+sw
SPI 2: err+xln+adm+aud+tk+ag+ign+call+sw
SPI 3: err+xln+adm+aud+tk+ag+ign+call+sw
SPI 4: err+xln+adm+aud+tk+ag+ign+call+sw
SPI 5: err+xln+adm+aud+tk+ag+ign+call+sw
SPI 6: err+xln+adm+aud+tk+ag+ign+call+sw
SPI 7: err+xln+adm+aud+tk+ag+ign+call+sw
SPI 8: err+xln+adm+aud+tk+ag+ign+call+sw
```

Здесь включены все логи для первого ACD:

```
# /cms/bin/spilog
SPI 1: err+xln+adm+aud+tk+ag+ign+call+sw
SPI 2:
SPI 3:
SPI 4:
SPI 5:
SPI 6:
SPI 7:
SPI 8:
```

Выключение лога для первого АСD. Указываем порядковый номер ACD и перед all дефис.

```
# /cms/bin/spilog 1 -all
SPI 1:
```

Выключение лога для второго АСD:

```
# /cms/bin/spilog 2 -all
SPI 2:
```
Собираемые логи находятся в каталогах */cms/pbx/acdx*, где x — номер ACD. Т.к. у меня подключен только один Communication Manager, логи в каталоге */cms/pbx/acd1*:

```bash
# ls /cms/pbx/acd1
ag.log      spi.err.02  spi.lnk.03  spi.log.02  spi.log.06  xln.log.03
ag.log.01   spi.lnk     spi.lnk.04  spi.log.03  xln.log
spi.err     spi.lnk.01  spi.log     spi.log.04  xln.log.01
spi.err.01  spi.lnk.02  spi.log.01  spi.log.05  xln.log.02
```

Эти логи можно сравнивать с MST логами CM, с мониторингом skill групп.

Все логи в каталоге */var/elog*

```bash title="/var/elog"
# ls -l /var/elog
total 500
-rw-rw-r--. 1 root cms    100 Apr  9 17:16 ag_conflicts
-rw-rw-r--. 1 root cms    585 Mar 26 19:55 cow_paste.log
drwxrwxr-x. 2 root cms   4096 Jan 19 22:37 cvsup
-rw-rw-r--. 1 root cms  28243 Apr  9 21:27 elog
-rw-rw-r--. 1 root cms 150071 Apr  6 14:04 elog.01
-rw-rw-r--. 1 root cms 150052 Apr  1 11:58 elog.02
-rw-rw-r--. 1 root cms 150161 Mar 31 01:58 elog.03
-rw-rw-r--. 1 root cms   2454 Mar 26 19:55 elog_if
-rw-rw-r--. 1 root cms      0 Jan 19 22:37 elog_if.01
-rw-rw-r--. 1 root cms      0 Jan 19 22:37 elog_if.02
```

Сообщения файла */var/log/messages*.  `tail -f` — в режиме реального времени последние 10 строк. Если надо одновременно показывать больше, например 20, добавляем `-n20`.

```bash title="/var/log/mesages"
# tail -f /var/log/messages
Apr  9 21:48:21 mrc-krl15-ucacms1 login[4187]: FAILED LOGIN 2 FROM localhost FOR , User not known to the underlying authentication module
Apr  9 21:48:22 mrc-krl15-ucacms1 pam_asg[4187]: Login for [] - rhost[localhost],tty[pts/3]
Apr  9 21:48:22 mrc-krl15-ucacms1 asglib[4187]: GetKey, Before calling DecKey: enc->key:O▒C#001, enckey:
Apr  9 21:48:22 mrc-krl15-ucacms1 pam_asg[4187]: Login  not an ASG login
Apr  9 21:48:25 mrc-krl15-ucacms1 login[4187]: FAILED LOGIN 3 FROM localhost FOR , User not known to the underlying authentication module
Apr  9 21:48:27 mrc-krl15-ucacms1 pam_asg[4187]: Login for [] - rhost[localhost],tty[pts/3]
Apr  9 21:48:27 mrc-krl15-ucacms1 asglib[4187]: GetKey, Before calling DecKey: enc->key:O▒C#001, enckey:
Apr  9 21:48:27 mrc-krl15-ucacms1 pam_asg[4187]: Login  not an ASG login
Apr  9 21:48:31 mrc-krl15-ucacms1 login[4187]: FAILED LOGIN SESSION FROM localhost FOR , User not known to the underlying authentication module
Apr  9 21:48:31 mrc-krl15-ucacms1 xinetd[2169]: EXIT: telnet status=0 pid=4186 duration=80(sec)
```

Диагностика веб-интерфейса CMSWeb

```bash
ps -ef | grep cmsweb
```
## Расширенный дебаг CMS Supervisor
The **Advanced Debugging** option under the CentreVu / Avaya CMS Supervisor *Tools* menu provides an interface to CentreVu / Avaya CMS Supervisor’s debugging features. Its use is for setting debug options and getting debug history, system, and error log files from the local PC for remote analysis. Only Avaya support organizations are expected to analyze information provided by Advanced Debugging. The regular end user will probably not gain any better understanding of a problem by viewing this data.

By clicking on the **Tools** option from the menu bar, then select **Advanced**, you will see the following screen. Select ‘**_No_**’.

![](image95.png)

After selecting No, the next screen seen will be the default Advanced Debugging screen.

![](image96.png)
In order to obtain the proper level on information, the default parameters need to be changed as follows:
**Error Level Settings** –  set it to 8.
Rollover count – this can remain at 3 provided you can turn off the trace or shut down the Supervisor application in time so as the files are not overwritten. I have had some set this as high as 6.
Rollover size – this can remain 256 kb.

Notice the Stop Date and Stop Time will automatically populate the current date and the top of the next hour. Only if you can duplicate the problem during that short time, it should not be necessary to check the box **Stop Time OverRide**. ==In a scenario trying to troubleshoot an intermittent issue, you will want to check the **Stop Time OverRide** box.==

You should have a screen looking like this:

![](image97.png)

Once the Trace Parameters have been defined, you then check the **Trace Log Options Enabled** box. Once the Trace Log Options Enabled box is checked you will notice the parameters have been grayed out preventing them from being changed.

![](image98.png)

In order to complete the initialization of the trace, you must click onOK, you will see the screen:

![](image99.png)

Click on ‘OK’ again and completely exit the Supervisor application. We want to ensure we have fresh data captured in the trace logs. The next steps will be done within the Windows Explorer feature. In Windows Explorer you want to navigate to the Logs sub-directory of the Supervisor directory, by default located in
`C:\Program Files\Avaya\CMS Supervisor V11 [or whichever version you have installed].`

![](image100.png)

Once at the Logs directory, **you need to highlight all of the files within that directory and delete them**.

![](image101.png)

When Supervisor restarts these files will be recreated and will continue to log data as long as the application is running. Once the problem has been duplicated, exit the Supervisor application and send these log files to your Avaya engineer for analysis and continued troubleshooting.
## Отправка на почту

```bash
# vi/.forward
myname@mycompany.ru

(mrc-krl15-ucacms1)-(root)=# ls -la /.forward
-rw-r--r--. 1 root root 17 Apr  9 21:00 /.forward

(mrc-krl15-ucacms1)-(root)=# chmod 600 /.forward

(mrc-krl15-ucacms1)-(root)=# ls -la /.forward
-rw-------. 1 root root 17 Apr  9 21:00 /.forward
```
## Время. Часовой пояс и NTP.
Немного отличаются настройки, поэтому проверяем версию Linux

```bash
# cat /etc/*release
Red Hat Enterprise Linux Server release 6.9 (Santiago)
```

Смотрим, какой у нас текущий часовой пояс:

```bash
# cat /etc/sysconfig/clock
ZONE="America/New_York"
```

Проверяем время:

```bash
# date
Wed Mar 21 10:40:21 EDT 2018
```

Проверяем доступные часовые пояса вообще и пояса в Европе:

```bash
(mrc-krl15-ucacms1)-(root)=# ls /usr/share/zoneinfo
Africa      Chile    GB         Indian       Mexico    posixrules  Universal
.....
Canada      Etc      HST        Libya        Portugal  tzdata.zi
CET         Europe   Iceland    MET          posix     UCT

(mrc-krl15-ucacms1)-(root)=# ls /usr/share/zoneinfo/Europe
Amsterdam   Busingen     Kiev        Moscow      Saratov     Vatican
Andorra     Chisinau     Kirov       Nicosia     Simferopol  Vienna
...
Budapest    Kaliningrad  Monaco      Sarajevo    Vaduz
```
 
 Редактируем файл с часовой зоной. После редактирования проверяем
 
```bash title="/etc/sysconfig/clock"
ZONE="Europe/Moscow"
```

Активируем обновление:

```bash
tzdata-update
```

Проверяем дату и время:

```bash
# date
Wed Mar 21 17:46:02 MSK 2018
```

Примечание: для изменения времени конкретного пользователя отредактировать *.bashrc* в его домашнем каталоге:

```bash
export TZ="/usr/share/zoneinfo/[timezone_directory]/[timezone_file]"
```

Например `[timezone_directory]` — Europe,  `[timezone_file]` — Kalinigrad.

Видно, что Москва, но время некорректное. Установить время:

```bash
# date +%T -s 13:56:00
13:56:00
```

Изменение даты:

```bash
date +%D -s YYYY-MM-DD
```

Для точности настроим NTP. Сервера по умолчанию комментируем или удаляем, добавляем свои NTP сервера в  */etc/ntp.conf* :
```bash title="/etc/ntp.conf"
# server 0.rhel.pool.ntp.org iburst
# server 1.rhel.pool.ntp.org iburst
# server 2.rhel.pool.ntp.org iburst
# server 3.rhel.pool.ntp.org iburst
server 172.21.40.117
server 172.21.40.189
```

Стартуем сервис ntpd и добавляем его в автозагрузку:
```bash
service ntpd start
chkconfig ntpd on

# Проверка
ntpq –p
ntpstat
```

Пока наши сервера недоступны, время не синхронизируется.

```bash
# ntpstat
unsynchronised
  time server re-starting
   polling server every 8 s
```

## Расписание, сервисы
Формат DataStore – VMFS6 (рекомендуется вероятно к предыдущей версии VMFS5).

```
(mrc-krl15-ucacms1)-(root)=# sfdisk -lq

Disk /dev/sda: 78325 cylinders, 255 heads, 63 sectors/track
Warning: extended partition does not start at a cylinder boundary.
DOS and Linux will interpret the contents differently.
Units = cylinders of 8225280 bytes, blocks of 1024 bytes, counting from 0

   Device Boot Start     End   #cyls    #blocks   Id  System
/dev/sda1   *      0+     72-     73-    583676+  83  Linux
/dev/sda2         72+   1378-   1306-  10485756+  83  Linux
/dev/sda3       1378+   2683-   1306-  10485756+  83  Linux
/dev/sda4       2683+  78325-  75642- 607589376    f  W95 Ext'd (LBA)
/dev/sda5       2683+   3728-   1045-   8388604+  82  Linux swap / Solaris
/dev/sda6       3728+   5002-   1275-  10239996+  83  Linux
/dev/sda7       5002+   9180-   4178-  33554428+  83  Linux
/dev/sda8       9180+  12574-   3395-  27262972+  83  Linux
/dev/sda9      12574+  14662-   2089-  16777212+  83  Linux
/dev/sda10     14662+  16221-   1559-  12517372+  83  Linux
/dev/sda11     16221+  78325-  62104- 498847744   83  Linux

(mrc-krl15-ucacms1)-(root)=# df -Th | grep sda
/dev/sda2      ext4   9.8G  1.3G  8.0G  14% /
/dev/sda1      ext4   546M   38M  480M   8% /boot
/dev/sda3      ext4   9.8G  340M  8.9G   4% /cms
/dev/sda7      ext4    32G   48M   30G   1% /export/home
/dev/sda10     ext4    12G  781M   11G   7% /opt
/dev/sda6      ext4   9.5G  2.1G  7.0G  23% /storage
/dev/sda9      ext4    16G  263M   15G   2% /tmp
/dev/sda8      ext4    26G  111M   25G   1% /var

(mrc-krl15-ucacms1)-(root)=# rpm -qa | grep sysstat
sysstat-9.0.4-33.el6.x86_64
```

Можно редактировать.
```bash
(mrc-krl15-ucacms1)-(root)=# cat /etc/cron.d/sysstat
# Run system activity accounting tool every 10 minutes
*/10 * * * * root /usr/lib64/sa/sa1 1 1
# 0 * * * * root /usr/lib64/sa/sa1 600 6 &
# Generate a daily summary of process accounting at 23:53
53 23 * * * root /usr/lib64/sa/sa2 -A
```

Посмотреть список текущих сервисов:

```
(mrc-krl15-ucacms1)-(root)=# chkconfig --list
abrt-ccpp       0:off   1:off   2:off   3:on    4:off   5:on    6:off
abrtd           0:off   1:off   2:off   3:on    4:off   5:on    6:off
acpid           0:off   1:off   2:on    3:on    4:on    5:on    6:off
...
udev-post       0:off   1:on    2:on    3:on    4:on    5:on    6:off
xinetd          0:off   1:off   2:off   3:on    4:on    5:on    6:off

xinetd based services:
        chargen-dgram:  off
        chargen-stream: off
        daytime-dgram:  off
        daytime-stream: off
        discard-dgram:  off
        discard-stream: off
        echo-dgram:     off
        echo-stream:    off
        rsync:          off
        tcpmux-server:  off
        telnet:         on
        telnet.orig:    off
        time-dgram:     off
        time-stream:    off
```

Экспортировать список текущих сервисов во временный каталог в файл `current_chkconfig.txt`:

```bash
chkconfig --list > /tmp/current_chkconfig.txt
```

Сохраним новый список сервисов во временный каталог в файл new_chkconfig.txt:

```bash
chkconfig --list > /tmp/new_chkconfig.txt
```

Сравним содержимое двух файлов:

```bash
diff /tmp/current_chkconfig.txt /tmp/new_chkconfig.txt
```

смотрим, какие сервисы надо включить на текущий момент. Общая команда:

```bash
chkconfig [--level levels] <Service name> <on|off|reset>
```

Пример:

```bash
chkconfig --level 2345 snmpd on
```

Выключение Linux для проведения процедур обслуживания:

```bash
shutdown —h 0
```

## Скачивание и установка AF для ASG
В CMS R17 добавлена ASG (Avaya Security Gateway) аутентификация для Avaya Services логинов. Во время установки CMS устанавливается файл аутентификации (AF) по умолчанию. Для увеличения безопасности можно скачать и установить уникальный AF файл.

[https://rfa.avaya.com](https://rfa.avaya.com) — файл `.xml` можно скачать или отправить себе на почту.

Я получила файл `AF-7001273069-180406-130625.xml`. Десятизначный номер, начинающийся с семерки, надо запомнить, он нужен будет для генерирования нового файла аутентификации при апгрейде.

Скопировать файл с помощью WinSCP на CMS в каталог */tmp*. Устанавливаем (`-l` это строчная буква L):

```bash
# /opt/cmsasg/usr/local/bin/loadauth -af -l /tmp/AF-7001273069-180406-130625.xml
Loading file /tmp/AF-7001273069-180406-130625.xml
useradd: warning: the home directory already exists.
Not copying any file from skel directory into it.
Creating mailbox file: File exists
copying the AFS file
```

## Создание пользователя для CMS Supervisor
У пользователя должны быть права:
- Informix ODBC Access on CMS
- Permissions to create custom tables in CMS
- Permissions to insert/update/delete information from custom tables    in CMS
- Permissions to read data from CMS standard tables
- Permissions to view and export CMS Supervisor reports
- Permissions to upload Afiniti modified reports on CMS Supervisor
- Permissions on all the Afiniti skills and VDNs.

Ошибка входа:

``` title="/var/log/messages"
(mrc-krl15-ucacms1)-(root)=# tail /var/log/messages
Apr  6 14:29:04 mrc-krl15-ucacms1 pam_asg[30279]: Login cms not an ASG login
Apr  6 14:29:08 mrc-krl15-ucacms1 xinetd[2206]: START: telnet pid=30296 from=::1
Apr  6 14:29:10 mrc-krl15-ucacms1 pam_asg[30297]: Login for [cms] - rhost[localhost],tty[pts/2]
Apr  6 14:29:10 mrc-krl15-ucacms1 asglib[30297]: GetKey, Before calling DecKey: enc->key:#177i#001, enckey:
Apr  6 14:29:10 mrc-krl15-ucacms1 pam_asg[30297]: Login cms not an ASG login
Apr  6 14:29:10 mrc-krl15-ucacms1  -- cms[30297]: LOGIN ON pts/2 BY cms FROM localhost
Apr  6 14:30:53 mrc-krl15-ucacms1 xinetd[2206]: EXIT: telnet status=0 pid=30296 duration=105(sec)
Apr  6 14:31:14 mrc-krl15-ucacms1 pam_asg[30437]: Login for [cms] - rhost[10.0.45.51],tty[ssh]
Apr  6 14:31:14 mrc-krl15-ucacms1 asglib[30437]: GetKey, Before calling DecKey: enc->key: \#007#177, enckey:
Apr  6 14:31:14 mrc-krl15-ucacms1 pam_asg[30437]: Login cms not an ASG login
```

Проверка текущей версии CMS:

```bash
(mrc-krl15-ucacms1)-(root)=# rpm -q cms
cms-R18.0.2.0-ma.k.x86_64
```
## Нельзя залогиниться в CMS Supervisor или TE, но Putty работает
CMS can be accessed by Putty using either SSH or TELNET but with Supervisor we receive a timeout message in auto mode or hangs in manual mode using both SSH or TELNET.

Looking at the users logged in server with command **who**, we noticed that users who tried to access from supervisor were listed by **who** even when supervisor threw a timeout error. This confirmed it wasn't a SSH issue.

```
# who
root     pts/0        2018-04-09 19:01 (10.0.45.51)
cms      pts/1        2018-04-09 19:28 (10.0.45.51)
```

Because Supervisor uses TELNET to communicate with CMS even when we select SSH, we trace this protocol with Wireshark and found that after the first telnet client request, server sends **Do Authentication Option** and supervisor doesn't reply with **Won't Authentication Option** as RFC1416 requires.

CMS Supervisor не может подключиться к CMS, т.к. не только 22 порт, но и 23 порт должен быть открыт для нового CMS R18. Для старых версий CMS скорее всего порт 23 (telnet) был открыт по умолчанию. На сервере CMS отредактировать файл */etc/hosts.allow*. По умолчанию в конце файла строка ALL : ALL : DENY, из-за чего все telnet сессии запрещены. При этом разрешены сессии ssh в строке sshd : ALL.

Для безопасности предлагается открывать сессии telnet только для определённых подсетей. Например:

```bash title="/etc/hosts.allow"
in.telnetd : 192.168.100.
in.telnetd : 10.0.45.       # это для моей компании
```

Настройки для Solaris:

```bash
# inetadm -l svc:/network/telnet:default
SCOPE    NAME=VALUE
         name="telnet"
         endpoint_type="stream"
         proto="tcp6"
         isrpc=FALSE
         wait=FALSE
         exec="/usr/sbin/in.telnetd -a off"
         user="root"
default  bind_addr=""
default  bind_fail_max=-1
...
```

Если нет параметра `-a off`, добавить:

```bash
inetadm -m svc:/network/telnet:default exec="/usr/sbin/in.telnetd -a off"
```

And verify with previous command, this change will take effect immediatly
Также проверить, что активен `AllowTcpForwarding` в файле */etc/ssh/sshd_config*:

```bash title="/etc/ssh/sshd_config"
….
#AllowAgentForwarding yes
AllowTcpForwarding yes
```

Установленные пакеты сервиса telnet:

```shellsession
$ rpm -qa | grep telnet
telnet-server-0.17-48.el6.x86_64
telnet-0.17-48.el6.x86_64
```

Проверка, включен ли telnet в автозагрузке:

```shellsession
$ chkconfig --list telnet
  telnet          on
```

Из сети заказчика тоже порт 23 не должен быть заблокирован.
Basic outlines as to requirements within an internal network firewall to consider.
ODBC connections could be linked to any PC on the network running a ODBC client to connect to CMS.
RCP traffic would be required to any PC running CMS Supervisor and or access CMS Web.

Проверить статус ssh:

```shellsession
$ service sshd status
openssh-daemon (pid  2158) is running...
```

Проверить работу telnet в Linux

```shellsession
(mrc-krl15-ucacms1)-(root)=# telnet localhost
Trying ::1...
Connected to localhost.
Escape character is '^]'.
Red Hat Enterprise Linux Server release 6.9 (Santiago)
Kernel 2.6.32-696.18.7.el6.x86_64 on an x86_64
mrc-krl15-ucacms1 login:
```

Проверить */etc/xinetd.d/telnet*:

```shellsession title="/etc/xinetd.d/telnet"   hl_lines="12"
# default: on
# description: The telnet server serves telnet sessions; it uses \
#       unencrypted username/password pairs for authentication.
service telnet
{
        flags           = REUSE
        socket_type     = stream
        wait            = no
        user            = root
        server          = /usr/sbin/in.telnetd
        log_on_failure  += USERID
        disable         = no
        instances       = 1650
        per_source      = 1600
}
```

Перегрузить telnet, если были изменены настройки:

```shellsession
# /etc/init.d/xinetd restart
Stopping xinetd:   [OK]
Starting xinetd:   [OK]
```

Записать DNS ip-адрес, и проверить, что он пингуется:

```shellsession
ping <DNS IPaddress>
```

Если DNS не пингуется, сделать правки в нескольких файлах.
Проверить что есть строки в файле  */etc/nsswitch.conf*:

```bash title="/etc/nsswitch.conf" hl_lines="3"
passwd: files
group: files
hosts: files dns
ipnodes: files
```

Удалить запись для  dns используемую в hosts, закомментировав:

```bash title="/etc/nsswitch.conf"
hosts: files # dns
```

Проверить настройки сети */etc/sysconfig/network-scripts/ifcfg-eth0*. Если DNS не пингуется, закомментировать. Не должно быть дублирующихся записей в файле. Если есть, удалить и рестартовать сервис.

```shellsession
$ service network restart
```

Проверить */etc/resolv.conf*. Закомментировать строки, начинающиеся с DNS, если они есть (у меня их не было)

```bash title="/etc/resolv.conf"
# Generated by NetworkManager
# DNS1=xxx.xxx.xxx.xxx
# DNS2=xxx.xxx.xxx.xxx
DOMAIN=lab.foo.com bar.foo.com
nameserver xxx.xxx.xxx.xxx
search sd.avaya.com
```

If it is not answering then ask customer to get it corrected. Or if they are not using DNS then ask them to remove the entry from the file and restart the network services again.
## Нельзя залогиниться в CMS Supervisor / TE (не важно, автоматический или ручной режимы). 
"The connection to the CMS server has timed out. Login cannot be completed."
Скриншот русской версии CMS Supervisor:

![](image102.png)

Проблема встречается в Windows 7, в то время как Windows 10 работали без проблем. Вход через Putty/Secure CRT работает без проблем. Также люди могут залогиниться с другого компьютера.

Причина: CMS Supervisor конфликтует с сессией PuTTY и ssh key, сохраненным для PuTTY в реестре.

Решение: очистить кэш в реестре. В Проводнике в каталоге `%APPDATA%\Avaya\CMS Supervisor RXX\Cache` удалить временный файл **CVS_Cache.tmp**, где **XX** — версия CMS Supervisor.

Пример: `C:\Users\atoropova\AppData\Roaming\Avaya\CMS SupervisorR18\Cache`

![](image103.png)

Также удалить *Sessions* и *SshHostKeys* в реестре в ветке, касающейся PuTTY:
1. С правами администратора открыть редактор реестра: Windows +R,ввести \"regedit\" в меню Выполнить.
2. Перейти в *HKEY_CURRENT_USER→ Software→ SimonTatham→ PuTTY*
3. Удалить папку **Sessions→CMS** и **SshHostKeys**→ключ для ip-адреса CMS.

![](image104.png)

![](image105.png)

**HKEY_CURRENT_USER→ Software→ Avaya→ PuTTY→SshHostKeys**→удалить ключ для ip-адреса CMS.

![](image106.png)

Увеличить параметр **SSHDelay** c 5000 до 25000
`HKEY_CURRENT_USER\Software\Avaya\Supervisor\17.0\Servers\135.27.xx.xx\SSH\SSHDelay`
Перегрузить компьютер
Залогиниться в CMS Supervisor а потом уже логиниться через PuTTY.

Вторая причина: у пользователя было слишком много попыток неправильного входа, и параметр `pam_tally2` мог зашкалить за максимальное значение для этого местоположения. Для проверки `pam_tally2`:

```shellsession
pam_tally2 --user=username
```

Отобразится число ошибок при входе с разных мест. В примере нет неправильных попыток входа:

```shellsession
# pam_tally2 --user=cms
Login           Failures Latest failure     From
cms                 0
# pam_tally2 --user=cmssvc
Login           Failures Latest failure     From
cmssvc              0
# pam_tally2 --user=root
Login           Failures Latest failure     From
root                0
```

Решение: сбросить параметр pam_tally2.

```shellsession
# pam_tally2 --user=username --reset
```

Желательно после этого для безопасности задать новые пароли пользователям.
## Ручной и автоматический вход в CMS Supervisor. Добавление администратора.
Логин **cms** работает только при **Manual Login**, т.к. использует стандартную оболочку Linux Korn Shell в Solaris (*/bin/ksh*) или Bash в RHEL (*/bin/bash*). Для автоматического входа в CMS Supervisor у пользователя должна быть специальная оболочка CMS shell (*/usr/bin/cms*).

Кто какие оболочки использует, видно в */etc/passwd*. У нас для логинов **cms** (это логин администратора) и **cmssvc** (это сервисный логин) по умолчанию используется оболочка Korn Shell, хоть это не Solaris, а RHEL. Т.е. залогиниться в CMS Supervisor они могут только в ручном режиме.

```shellsession title="/etc/passwd" hl_lines="6 7"
(mrc-krl15-ucacms1)-(root)=# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
....
tcpdump:x:72:72::/:/sbin/nologin
informix:x:1000:1000::/opt/informix:/bin/bash
cms:x:1001:1001:CMS:/export/home/cms:/bin/ksh
cmssvc:x:1002:1001:Services:/export/home/cmssvc:/bin/ksh
craft:x:780:500::/home/craft:/bin/bash
inads:x:779:500::/home/inads:/bin/bash
init:x:778:500::/home/init:/bin/bash
sroot:x:0:0::/home/sroot:/bin/bash
rasaccess:x:1003:1003::/export/home/rasaccess:/bin/bash
```

Для них так настраивать:

![](image107.png)

Добавляем пользователя anna в CMS Supervisor: *Tools→ User Permissions→ User Data* (*Инструменты→ Полномочия пользователя→ Данные пользователя*). Добавим нажав на плюс. Длина имени логина ID не более восьми символов. Параметры User Name, (Room Number, Telephone Number и Default Printer опционально). Maximum user window count (1-12) — сколько окон CMS Supervisor может открывать этот пользователь одновременно. Minimum refresh rate (seconds) — частота обновления экрана в отчетах реального времени. Login ACD — привязка к телефонной станции при входе.

В качестве логинов нельзя использовать служебные имена Linux: con, nul, aux, com1, com2, com3, com4, com5, com6, com7, com8, com9, lpt1, lpt2, lpt3, lpt4, lpt5, lpt6, lpt7, lpt8, lpt9.

![](image108.png)

У меня также есть все необходимые права.

![](image109.png)

Теперь в /etc/passwd видно, что оболочка для нового пользователя anna другая. И под этим логином можно будет автоматически заходить в CMS Supervisor.

```bash title="/etc/passwd" hl_lines="3 4"
…
informix:x:1000:1000::/opt/informix:/bin/bash
cms:x:1001:1001:CMS:/export/home/cms:/bin/ksh
cmssvc:x:1002:1001:Services:/export/home/cmssvc:/bin/ksh
craft:x:780:500::/home/craft:/bin/bash
inads:x:779:500::/home/inads:/bin/bash
init:x:778:500::/home/init:/bin/bash
sroot:x:0:0::/home/sroot:/bin/bash
rasaccess:x:1003:1003::/export/home/rasaccess:/bin/bash
anna:x:1004:1001::/export/home/anna:/usr/bin/cms
```

Пароль я себе задала в Linux CLI:

```shellsession
# passwd anna
Changing password for user anna.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```

Первый вход неудачный, второй уже Ок.
## Сравнение программы CMS Supervisor и веб-интерфейса CMS Supervisor

| Функционал             | CMS Supervisor PC Client                        | CMS Supervisor Web Client                                  |
| ---------------------- | ----------------------------------------------- | ---------------------------------------------------------- |
| Платформы              | Windows 7,8, 10                                 | IE, Firefox, Safari, Chrome (используют флэш)              |
| Установка              | Надо скачать и установить как обычную программу | Сразу доступен из браузера (при включенном сервисе cmsweb) |
| Тип соединения         | Ssh, порт 22                                    | Https, порт 8443                                           |
| Можно ли менять пароль | Нет (с оговоркой)                               | Да                                                         |

Other CMS Supervisor Web improvements are:
- Threshold high and low ranges can be of different colors.
- You can change the report input values without re-launching the report by using the Set Input button.
- Users can change their password from the Web interface by clicking the User tab.

!!! note
    CMS Supervisor Web uses Flash technology to display reports. Therefore, reports only run on devices that support Flash. Without Flash, the Report menu stays hidden. Other features of CMS Supervisor Web do not require Flash.

**What is CMS Mobile Supervisor**
CMS Mobile Supervisor is an iPad application that the supervisors and operations managers of a contact center can use to monitor the agents and the health of the contact center when the supervisors and operations managers are away from their desks. The contact center supervisors and operations managers get the current status of the contact center activities on their iPad.

CMS Mobile Supervisor does not replace CMS Supervisor PC client or CMS Supervisor Web.

CMS Mobile Supervisor is a real-time reporting tool that displays specific real-time summary views of skills and agent activity with the capability to drill through to individual agent details.

Admin Yes Supports the following administrative procedures:
- Change agent skills
- Multi-agent skill change

Report Designer
(report creation)
Yes No
View Custom Reports
(created on server)
Yes Yes
View Designer Reports
(created on PC Client)
Yes Yes
Real Time Report Execution Yes Yes
Historical Report Execution Yes Yes
Integrated Report Execution Yes Yes
1. The thick client can change the password only if it has expired
... тут ещё что-то было
## Настройка CMS Supervisor
### Добавление ACD (Communication Manager)
*Команды→ Словарь*. Добавляем только один раз. Изначально это сделано из Linux CLI.

![](image110.png)
### Добавление операторов (возможно только из программы CMS Supervisor)
*Словарь→ Имена пользователей* (вкладка *Операции*).

![](image111.png)

Редактировать операторов можно отсюда или из веб-интерфейса.

Включить трассировку операторов для записи всех событий: *Команды→ Администрирование операторов → Включить трассировку оператора* (вкладка Операции).

![](image112.png)

![](image113.png)

Добавление операторов в skill группы: *Команды→ Словарь→ Группы операторов*.

![](image114.png)
### Добавление VDN

![](image115.png)
### Добавление skill группы
*Команды→ Словарь→ Split/Skills* (вкладка Операторы)

![](image116.png)
### Добавление вектора
*Команды→ Словарь→ Векторы

![](image117.png)
### Коды перерыва
Communication Manager: `change reason codes`
CMS Supervisor: *Команды→ Словарь→ Коды причины ПЕРЕРЫВА*. Коды соответствуют настройкам CM.

![](image118.png)
### Редактирование вычислений (при необходимости)
Команды→ Словарь→ Вычисления

![](image119.png)
### Сбор статистики по каждому звонку (Записи о вызовах)
Увеличение значения Data Storage Allocation(DSA) влияет на сервис.
`su cms cms`
Tools→ System Setup→ Data Collection → Выбрать все ACD→ выбрать off →Modify (выключили)

![](image120.png)

*Tools→ System Setup→ CMS State→ Single User mode→ Modify* (переводим в однопользовательский режим).

![](image121.png)

Tools→ System Setup→ Data Storage allocation→ установить **Number of Call Records** (от 0 до 100000), по умолчанию 0. По умолчанию:

![](image122.png)

Стало:

![](image123.png)

Tools→ System Setup→ Data Collection → выбрать ACD →выбрать **on**→Modify
Tools→ System Setup→ CMS State→ Multi-user mode → modify
## Кастомные отчеты
Отчеты можно редактировать только в CMS Supervisor или в Linux CLI. В веб-интерфейсе можно  просматривать готовые отчеты. В категории Designer находятся редактируемые (видны всем или только тому кто редактирует) и готовые предустановленные отчеты. Сделано два отчета:
- суммарный отчет за день по группе Tour Desk
- детализированный отчет по пропущенным вызовам за выбранный интервал времени со списком городских и мобильных номеров, с которых звонили.

Звонок считается потерянным в группе Tour Desk только если он потеря после сообщения о записи разговоров (после попадания в очередь операторов queue). Если звонок потерялся в меню во время приветствия, донабора или во время сообщения о записи разговоров, он учитывается как потерянный в статистике по VDN (vector directory number) или вектору, но не по группе.

Если в отчете по пропущенным вызовам указан номер группы 100, значит звонок пропущен в очереди. Если указан номер оператора, значит звонящий положил трубку до того как оператор успел ответить. Если не указан ни номер группы, ни номер оператора, звонок потерян во время приветствия, донабора или сообщения о записи разговоров.

Набранный номер:
72000 — приветствие Tour Desk
72002 — общее приветствие с донабором внутреннего номера.

Данные можно экспортировать в csv файл.

!!! warning
    Если оператор не ответил в течение четырех гудков, но при этом абонент трубку не положил, звонок не теряется, а переходит на следующего свободного оператора, если такой есть, или в очередь с музыкой, если все операторы заняты. Звонок в статистике не считается потерянным.

![](image124.png)

### Пропущенные за интервал
Проблема отчета (не работал из веб-интерфейса)  из-за запроса к базе данных, который получается после редактирования в дизайнере отчетов. Вот что в оригинальном отчете Call Records:
```sql
where=where text="ACD= $acd and ((((ROW_DATE = $G 1 and ROW_DATE < $G 3 and ROW_TIME >= $G 2 ) or (ROW_DATE > $G 1 and ROW_DATE = $G 3 and ROW_TIME <= $G 4 ) or (ROW_DATE > $G 1 and ROW_DATE < $G 3))) or ((ROW_DATE = $G 3) and (ROW_DATE = $G 1 ) and (ROW_TIME >= $G 2 ) and (ROW_TIME <= $G 4)))" order="order by" ocol=CALLID,SEGMENT
```

Как он выглядит при редактировании для нас:

```sql
ACD= $acd and ((((ROW_DATE = [Start Date:] and ROW_DATE < [Stop Date:] and ROW_TIME >= [Start Time:] ) or (ROW_DATE > [Start Date:] and ROW_DATE = [Stop Date:] and ROW_TIME <= [Stop Time:] ) or (ROW_DATE > [Start Date:] and ROW_DATE < [Stop Date:] ))) or ((ROW_DATE = [Stop Date:] ) and (ROW_DATE = [Start Date:] ) and (ROW_TIME >= [Start Time:] ) and (ROW_TIME <= [Stop Time:] ))) ORDER BY CALLID,SEGMENT
```

Вот что получается после добавления диспозиции:

```sql
where=where text="ACD=$acd and $e((((ROW_DATE;1; and ROW_DATE < $G3  and ROW_TIME >=$G2 ) or (ROW_DATE > $G1  and $eROW_DATE;3; and ROW_TIME <=$G4 ) or (ROW_DATE > $G1  and ROW_DATE < $G3 ))) or $e((ROW_DATE;3;) and $e(ROW_DATE;1;) and (ROW_TIME >=$G2 ) and (ROW_TIME <=$G4 ))) AND DISPOSITION=3" 
order="order by" ocol=CALLID,SEGMENT
```

Значения DISPOSITION, которые можно использовать в отчетах по списку звонков. Выбирая ABAN мы получаем список пропущенных вызовов.

| Значение | Описание                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1        | Вызов соединен (CONN, сторонний вызов для измеряемого оператора). Подключенный вызов не является вызовом ACD измеряемого оператора, для которого CMS получает сигнал о том, что оператор ответил на вызов.                                                                                                                                                                                                                                                                                                      |
| 2        | Вызов принимается (вызов ANS, вызов группы split/навыки прямой вызов оператора, принятый оператором). Принятый вызов — это любой вызов группы split/навык или прямой вызов ACD оператора, для которого система CMS принимает индикацию того, что оператор ответил на этот вызов, и вызов не был псевдопотерянным.                                                                                                                                                                                               |
| 3        | Вызов теряется (ABAN). Потерянный вызов — это любой вызов ACD, при котором вызывающий абонент вешает трубку до поступления ответа от оператора и для которого система CMS принимает уведомление о потере вызывающего абонента. Псевдопотерянные вызовы (_PHANTOMABNS_) входят в состав потерянных вызовов.                                                                                                                                                                                                      |
| 4        | Вызов подвергается внешней переадресации (IFLOW). Переадресованный вызов - это вызов, переадресованный к пункту назначения вне сервера связи                                                                                                                                                                                                                                                                                                                                                                    |
| 5        | Вызов получает принудительную посылку сигнала «занято» (FBUSY). Вызов с принудительным ответом «занято» — это такой вызов, который система CMS регистрирует как _BUSYCALLS_ для группы СЛ, по которой он поступил. Такие вызовы могут быть вызовами VDN, которые приняли принудительную посылку сигнала «занято» от векторной команды, или вызовами группы split/навык для группы split с невекторным управлением, которые получают индикацию «занято» от сервера связи вследствие того, что очередь заполнена. |
| 6        | Вызов получает принудительный отбой (FDISC). Вызовы с принудительным отбоем - это те вызовы, которые были отсоединены коммуникационным сервером в результате выполнения векторной команды отключения disconnect. К вызовам, получившим принудительный отбой, также относятся вызовы, отключенные по причине истечения таймера прекращения векторной обработки или по причине завершения векторной обработки без постановки в очередь.                                                                           |
| 7        | Вызов имеет другое размещение (_OTHER_). К прочим вызовам относятся любые прочие вызовы, которые не попадают в перечисленные выше категории. Дополнительную информацию см. в определении _OTHERCALLS_ в этой главе.                                                                                                                                                                                                                                                                                             |

![](image125.png)

![](image126.png)

### Пример отчета
```sql
ACD=$acd and DISPOSITION=3 and  ((((ROW_DATE = [Дата начала:] and ROW_DATE < [Дата окончания:] and ROW_TIME >=0000) or (ROW_DATE > [Дата начала:] and  ROW_DATE = [Дата окончания:] and ROW_TIME <=2359) or (ROW_DATE > [Дата начала:] and ROW_DATE < [Дата окончания:] ))) or  ((ROW_DATE = [Дата окончания:] ) and  (ROW_DATE = [Дата начала:] ) and (ROW_TIME >=0000) and (ROW_TIME <=2359)))
```

![](image138.png)

![](image139.png)

![](image140.png)
### Отчет за день по группе со списком перерывов
Отчет за день — Tour Desk можно получить только за вчерашний день иранее. Данные по текущему дню в нем нельзя отобразить.

![](image127.png)

*Входящие вызовы* отображают общее число вызовов, поступивших на приветствие Tour Desk (vdn 72000).

*Потеряно включая очередь* — общее количество потерянных вызовов в группе (после сообщения о записи разговоров), включая потерянные вызовы на операторах и в очереди. Не включены потерянные вызовы, которые не дошли до группы операторов.

В первой строке таблицы сумма по всем операторам, в следующих строках данные по каждому из четырех операторов. Если оператор не был залогинен, данных нет.

*ACD-вызовы* — сколько вызовов принял оператор.

*Потер. На операторах* — абонент положил трубку во время звонка на операторе. Переадресация при неответе — оператор не ответил на вызов, и после четырех гудков звонок ушел на очередь или на другого оператора.

![](image128.png)

## Отчеты реального времени

### Отчет по очереди

| Сводка                             | Отображение состояния вызова для заданной группы                                                  |     |
| ---------------------------------- | ------------------------------------------------------------------------------------------------- | --- |
| **Состояние**                      | **Отобр. сост. навыка для выбр. группы, а также сост. оп-ра и причины перехода в режим ПЕРЕРЫВА** |     |
| Статус очереди/основного оператора | Отображение состояния всех основных операторов в выбранном навыке, а также состояния очереди      |     |

В основном нам необходим отчет о состоянии очереди. Он показывает состояние всех залогиненных операторов, сколько вызовов находится в очереди в ожидании, количество пропущенных вызовов.

![](cms33.png)

Минимальный интервал обновления данных — 3 секунды.

![](cms34.png)

![](cms35.png)

### Отчеты по группе skill/split

| Split/Skill-Группы по местоположению                 | Отображение действий оператора в группе и одного или нескольких ID местоположения.                                       |     |
| ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ | --- |
| Активные операторы - график                          | Отображение количества операторов различных типов, активных в выбранном навыке                                           |     |
| Выделенные операторы - график                        | Количество активных и процентное распределение занятых операторов, обслуживающих навык                                   |     |
| Граф. отчет осн. оп-ров ПЕРЕРЫВА                     | Отобр. осн. оп-ров с данным навыком, наход. в сост. ПЕРЕРЫВА, причины и продолжит. нахожд. в сост. ПЕРЕРЫВА              |     |
| Графический отчет операторов ПЕРЕРЫВА                | Отобр. всех оп-ров с данным навыком, наход. в сост. ПЕРЕРЫВА, причины и продолжит. нахожд. в сост. ПЕРЕРЫВА              |     |
| Графический профиль вызова                           | Отобр. выз., не выход. за пределы приемл. уровня обслуж., а также инт-ла обслуж. для прин. и потер. вызовов (для группы) |     |
| Графическое представление EWT                        | Отображение расчетного времени ожидания для одной или нескольких групп                                                   |     |
| Графическое представление очереди                    | Отобр.ожид. выз., самого раннего ожид. выз. и тенденции для одн. или неск. групп                                         |     |
| Графическое представление состояния                  | Отображение состояния оператора, продолжительности нахождения в этом состоянии и статистики группы                       |     |
| Графическое представление состояния основного навыка | Отображение состояния агента и группы, а также причин перехода в состояние ПЕРЕРЫВА (для операторов с основным навыком)  |     |
| Группа ПЕРЕРЫВА, отчет                               | Отображение количества операторов в режиме ПЕРЕРЫВА для каждого кода причины (для выбранных навыков)                     |     |
| Основной оператор навыка                             | Отобр. кол-ва основн. оп-ров в каждом режиме работы, а также состояния групп (для выбранных навыков)                     |     |
| Отчет операторов ПЕРЕРЫВА резерва 1                  | Отобр. оп-ров рез. 1 данного навыка, наход. в сост. ПЕРЕРЫВА, прич. и продолж. в сост. ПЕРЕРЫВА                          |     |
| Отчет операторов ПЕРЕРЫВА резерва 2                  | Отобр. оп-ров рез. 2 данного навыка, наход. в сост. ПЕРЕРЫВА, прич. и продолж. в сост. ПЕРЕРЫВА                          |     |
| Отчет по группе                                      | Отображение сводки операций текущей группы                                                                               |     |
| Перегрузка навыка - график                           | Отображение для выбранных навыков нормального и перегруженного состояний навыков, а также тенденций                      |     |
| Профиль вызова                                       | Отображение профиля производительности текущей группы                                                                    |     |
| Профиль занятости - график                           | Отображение типов операторов, обслуживающих указанный навык                                                              |     |
| Состояние навыка                                     | Отображение состояния операторов для выбранного навыка, а также состояния навыка                                         |     |
| Состояние оператора по местоположению                | Отображение состояния навыка для выбранной группы и одного или нескольких ID местоположения.                             |     |
| Статус основного оператора                           | Отображение состояния всех операторов с выбранным основным навыком, а также состояния группы                             |     |
| Фактический по отношению к заданному                 | Отображение производительности группы по отношению к заданному образцу.                                                  |     |

### Отчеты по VDN (Vector Directory Number)

| Графический профиль вызова | Отобр. выз. VDN принятых с приемл. ур. обслуж., а также инт-ла обслуж. для прин. и потер. вызовов |     |
| -------------------------- | ------------------------------------------------------------------------------------------------- | --- |
| Отчет VDN                  | Отображение количества обработанных вызовов для заданных VDN                                      |     |
| Приписывание группы к VDN  | Отображение информации об обработке вызовов для заданного VDN на основе приписывания групп        |     |
| Профиль вызова             | Отображение профиля производительности текущего VDN                                               |     |

### Отчеты по вектору
Только один отчет: отображение количества обработанных вызовов для заданных векторов

### Отчеты по оператору

| Графическая информация     | Отображение в реальном времени информации и статистики указанного оператора                    |     |
| -------------------------- | ---------------------------------------------------------------------------------------------- | --- |
| Отчет оператора            | Отобр. сост. оп-ров в выбр. группе и причин перехода в режим ПЕРЕРЫВА                          |     |
| Отчет подгруппы операторов | Отображение состояния операторов в группе, а также причин перехода операторов в режим ПЕРЕРЫВА |     |

## Хронологические отчеты
Отчеты за день, неделю и месяц можно выводить только за предыдущий срок. В отчетах за интервал времени можно получить данные по текущему дню.
### Отчеты по группе skill/split

| ASA за день (графич.)                                         | Отображение средней скорости ответа для одной или нескольких групп за каждый день.                                      |     |
| ------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- | --- |
| Split/Skill-Группы по местоположению                          | Отображение действий оператора в группе и одного или нескольких ID местоположения.                                      |     |
| Графический интерфейс ASA                                     | Отображение средней скорости ответа для одной или нескольких операторских групп за каждый интервал                      |     |
| Графический профиль вызова, за день                           | Отобр. вызовов с приемл. уровнем обслуж., а также инт-ла обслуж. для прин. и потер. вызовов (для группы)                |     |
| Графическое представление времени работы группы, за день      | Отображение времени работы оператора в каждом режиме и с каждой причиной перехода в режим ПЕРЕРЫВА за день (для навыка) |     |
| Графическое представление уровня обслуживания, за интервал    | Отобр. процента вызовов с приемл. ур. обслуж. и процента потер. вызовов для кажд.инт-ла                                 |     |
| Исходящие, за день                                            | Отображение (за день) количества и средней длительности исходящих вызовов каждого типа для группы                       |     |
| Исходящие, за интервал                                        | Отоб. (за интервал) кол-ва и средн. длит. исходящих вызовов каждого типа для группы                                     |     |
| Исходящие, за месяц                                           | Отображение (за месяц) количества и средней длительности исходящих вызовов каждого типа для группы                      |     |
| Исходящие, за неделю                                          | Отображение (за неделю) количества и средней длительности исходящих вызовов каждого типа для группы                     |     |
| Отчет, за день                                                | Отображение действий оператора в группе за один день                                                                    |     |
| Отчет, за месяц                                               | Отображение действий оператора в группе за один месяц                                                                   |     |
| Отчет, за неделю                                              | Отображение действий оператора в группе за одну неделю                                                                  |     |
| Перегрузка навыка за интервал - график                        | Отображение для выбранных навыков процента времени нахождения в состоянии перегрузки                                    |     |
| Предпочтительный уровень владения навыком, сводка за день     | Предоставляет сводку всех операций на предпочтительном уровне владения навыком за один или более дней                   |     |
| Предпочтительный уровень владения навыком, сводка за интервал | Предоставляет сводку всех операций на предпочтительном уровне владения навыком за выбранный интервал                    |     |
| Предпочтительный уровень владения навыком, сводка за месяц    | Предоставляет сводку всех операций на предпочтительном уровне владения навыком за один или более месяцев                |     |
| Предпочтительный уровень владения навыком, сводка за неделю   | Предоставляет сводку всех операций на предпочтительном уровне владения навыком за одну или более недель                 |     |
| Профиль вызова, за день                                       | Отображение количества принятых/потерянных вызовов в зависимости от времени ожидания за день                            |     |
| Профиль вызова, за месяц                                      | Отображение количества принятых/потерянных вызовов в зависимости от времени ожидания за месяц                           |     |
| Профиль вызова, за неделю                                     | Отображение количества принятых/потерянных вызовов в зависимости от времени ожидания за неделю                          |     |
| Сводка за день                                                | Сводка по всем операциям в группе за один или несколько дней                                                            |     |
| Сводка за интервал                                            | Сводка по всем операциям в группе за интервал в течение одного дня                                                      |     |
| Сводка за месяц                                               | Сводка по всем операциям в группе за один или несколько месяцев                                                         |     |
| Сводка за неделю                                              | Сводка по всем операциям в группе за одну или несколько недель                                                          |     |
| Среднее количество занятых позиций за интервал — график       | Отображение макс. количества занятых позиций в навыке и среднего количества занятых позиций                             |     |
| Уровень обслуживания мульти-ACD за день (графич.)             | Отображение процента уровня обслуживания для одной группы одного или нескольких ACD за каждый день.                     |     |
| Уровень обслуживания, за день                                 | Отображает уровень обслуживания для Split/Skill-группы за один или более дней                                           |     |
| Уровень обслуживания, за интервал                             | Отображает уровень обслуживания для Split/Skill-группы за выбранный интервал                                            |     |
| Уровень обслуживания, за месяц                                | Отображает уровень обслуживания для Split/Skill-группы за один или более месяцев                                        |     |
| Уровень обслуживания, за неделю                               | Отображает уровень обслуживания для Split/Skill- группы за одну или более недели                                        |     |
| Фактический по отношению к заданному интервалу                | Отображение производительности группы по отношению к заданному образцу.                                                 |     |
| Фактический по отношению к заданному, за день                 | Отображение производительности группы по отношению к заданному образцу.                                                 |     |

### Отчеты по оператору

|   |   |   |
|---|---|---|
|Split/Skill, за день|Сводка производительности одного оператора за один или несколько дней по группам||
|Split/Skill, за интервал|Сводка производительности одного оператора за один день по группам и по интервалу||
|Split/Skill, за месяц|Сводка производительности одного оператора за один или несколько месяцев по группам||
|Split/Skill, за неделю|Сводка производительности одного оператора за одну или несколько недель по группам||
|Вход в систему/выход из системы (навык)|Отображение данных о входе в систему/выходе из системы операторов по навыкам, а также причины выхода из системы||
|Входящие/исходящие, за день|Отображение всех входящих/исходящих вызовов, обработанных оператором за один или несколько дней||
|Входящие/исходящие, за интервал|Отобр. всех вход./исход. вызовов, обработанных оп-ром за интервал в течение одного дня||
|Входящие/исходящие, за месяц|Отображение всех входящих/исходящих вызовов, обработанных оператором за один или несколько месяцев||
|Входящие/исходящие, за неделю|Отображение всех входящих/исходящих вызовов, обработанных оператором за один или несколько недель||
|Графическое представление времени работы, за день|Время работы оператора в каждом режиме и с каждой причиной перехода в режим ПЕРЕРЫВА за день||
|Группа ПЕРЕРЫВА, за день|Отображение времени работы всех операторов группы для каждого кода причины перехода в режим ПЕРЕРЫВА за один или несколько дней||
|Группа ПЕРЕРЫВА, за месяц|Отображение времени работы всех операторов группы для каждого кода причины перехода в режим ПЕРЕРЫВА за один или несколько месяцев||
|Группа ПЕРЕРЫВА, за неделю|Отображение времени работы всех операторов группы для каждого кода причины перехода в режим ПЕРЕРЫВА за одну или несколько недель||
|Групповое обслуживание, за день|Отобр. общего времени работы каждого оп-ра выбранной группы в каждом состоянии за одну неделю||
|Групповое обслуживание, за месяц|Отобр. общего времени работы каждого оп-ра выбранной группы в каждом состоянии за один месяц||
|Групповое обслуживание, за неделю|Отобр. общего времени работы каждого оп-ра выбранной группы в каждом состоянии за одну неделю||
|Использование рабочего времени, за день|Отображение общего времени работы оператора в каждом состоянии за один или несколько дней||
|Использование рабочего времени, за месяц|Отображение общего времени работы оператора в каждом состоянии за один или несколько месяцев||
|Использование рабочего времени, за неделю|Отображение общего времени работы оператора в каждом состоянии за одну или несколько недель||
|ПЕРЕРЫВА за день|Отобр. врем. работы оп-ра для кажд. кода причины перехода в режим ПЕРЕРЫВА за один или неск. дней||
|ПЕРЕРЫВА за интервал|Отобр. врем. работы оп-ра для кажд. кода причины перехода в режим ПЕРЕРЫВА в течен. выбр. инт-лов||
|ПЕРЕРЫВА за месяц|Отобр. врем. работы оп-ра для кажд. кода причины перехода в режим ПЕРЕРЫВА за один или неск. месяцев||
|ПЕРЕРЫВА за неделю|Отображение времени работы оператора для каждого кода причины перехода в режим ПЕРЕРЫВА за одну или несколько недель||
|Сводка за день|Отобр. информации о производительности одного оператора за один или неск. дней для всех групп||
|Сводка за интервал|Отображение информации о производительности одного оператора за интервал для всех групп||
|Сводка за месяц|Отображение информации о производительности одного оператора по месяцам для всех групп||
|Сводка за неделю|Отображение информации о производительности одного оператора по неделям для всех групп||
|Сводка по группе, за день|Сводка за день по операциям оператора группы||
|Сводка по группе, за месяц|Сводка за месяц по операциям оператора группы||
|Сводка по группе, за неделю|Сводка за неделю по операциям оператора группы||
|Счетчик событий, за день|Отображение количества нажатий оператором каждой кнопки события за один или несколько дней||
|Счетчик событий, за интервал|Отображение количества нажатий оператором каждой кнопки события за интервал||
|Счетчик событий, за месяц|Отображение количества нажатий оператором каждой кнопки события за один или несколько месяцев||
|Счетчик событий, за неделю|Отображение количества нажатий оператором каждой кнопки события за одну или несколько недель||
|**Трассировка по местоположению**|Показать все действия одного оператора и время их выполнения. Полная статистика по действиям оператора, когда и что нажимал.||

Пример трассировки по местоположению:

![](cms36.png)
## Резервное копирование отчетов
Из категории Designer отчеты можно сохранить в файл с расширением `.rep` нажав на кнопку Копировать, чтобы потом восстановить не другом сервере или системе. В таком же виде отчет можно переслать в техподдержку Avaya для поиска ошибок.

![](image129.png)

Импортировать, в отличие от экспорта, файл `.rep` можно находясь в любой категории — всё равно он импортируется в Designer. Копировать→ Из файла ПК на сервер CMS.

![](image130.png)

![](image131.png)

![](image132.png)

## Бэкап CMS
### Full Maintenance Backup
#### Определяем необходимое место на диске
Установить переменные окружения для Informix:

```bash
. /opt/informix/bin/setenv
```

Отобразим информацию по текущему использованию базы данных Informix:

```
(mrc-krl15-ucacms1)-(root)=# onstat -d
IBM Informix Dynamic Server Version 12.10.FC2 -- On-Line -- Up 111 days 21:58:24 -- 2294960 Kbytes

Dbspaces
address          number   flags      fchunk   nchunks  pgsize   flags    owner    name
44b22028         1        0x70001    1        1        2048     N  BA    informix rootdbs
45d757f0         2        0x70001    2        1        2048     N  BA    informix physdbs
45d75998         3        0x60001    3        1        2048     N  BA    informix logdbs
45d75b40         4        0x60001    4        1        8192     N  BA    informix dbtemp
45d75ce8         5        0x60001    5        1        8192     N  BA    informix cmsdbs
 5 active, 2047 maximum

Chunks
address          chunk/dbs     offset     size       free       bpages     flags pathname
44b221d0         1      1      0          128000     109448                PO-B-- /cmsdisk
45f1e028         2      2      128000     327680     0                     PO-B-- /cmsdisk
45f1f028         3      3      455680     65536      0                     PO-B-- /cmsdisk
45f20028         4      4      521216     655360     655307                PO-B-- /cmsdisk
45f21028         5      5      3142656    61439232   61353596              PO-B-- /cmsdisk
 5 active, 32766 maximum

NOTE: The values in the "size" and "free" columns for DBspace chunks are
      displayed in terms of "pgsize" of the DBspace to which they belong.

Expanded chunk capacity mode: always
```

Из данных вывода заполним таблицу:

| Dbspaces asddress | numbers   | flags   | fchunk     | nchunks  | pgsize   | flags      | owner     | name         |
| ----------------- | --------- | ------- | ---------- | -------- | -------- | ---------- | --------- | ------------ |
| 45d75ce8          | 5         | 0x60001 | 5          | 1        | 8192     | N BA       | informix  | cmsdbs       |
| **Chunk address** | **chunk** | **dbs** | **offset** | **size** | **free** | **bpages** | **flags** | **pathname** |
| 45f21028          | 5         | 5       | 3142656    | 61439232 | 61353596 |            | PO-B--    | /cmsdisk     |

Пример из документации:

| Dbspaces asddress | numbers   | flags   | fchunk     | nchunks  | pgsize   | flags      | owner     | name         |
| ----------------- | --------- | ------- | ---------- | -------- | -------- | ---------- | --------- | ------------ |
| c64a5358          | 5         | 0x60001 | 5          | 1        | 8192     | N BA       | informix  | cmsdbs       |
| **Chunk address** | **chunk** | **dbs** | **offset** | **size** | **free** | **bpages** | **flags** | **pathname** |
| c64a5ac0          | 5         | 5       | 3142656    | 34679882 | 29584364 |            | PO-B--    | /cmsdisk     |

Полный размер базы:

```
cmsdbs = ((pgsize * size) / 1,073,741,824) = 264.59 GB   (1,073,741,824 = 1024*1024*1024)
cmsdbs = ((8,192 * 34,679,882) / 1,073,741,824) = 264.59 GB
cmsdbs = ((8192 * 61439232) / 1,073,741,824) = 468.744 Гб
```

Полный размер базы cmsdbs, доступный для полного бэкапа:

```
Full maintenance backups = (((8192 * 34,679,882) / 1,073,741,824) / 30) = 8.82 GB
Необходимое место для бэкапа = (((8,192 * (34,679,882 - 29,584,364)) / 1,073,741,824) / 30) = 14.01 GB
```

На основе примера рассчитываем, сколько у нас доступно для бэкапа:

```
FullMaintenance backup = (((8192 * 61439232)) / 1,073,741,824 / 30) = 15.625 Гб
```

Необходимое место для бэкапа:

```
cmsdbs = (((8192 * (61439232 - 61353596)) / 1,073,741,824 / 30) = 0.0218 Гб
```

Use the output generated from running this command and the formulas at the bottom of the tables to calculate how much database space is required for a CMS full maintenance backup.

The data in this table is dynamic and changes as database space is used.

| Платформа/cmsdbs | Dbspace pgsize | Full disk size of cmsdbs Dbspace | Total Disk cmsdbs Dbspace (в байтах) | Total Disk cmsdbs Dbspace (округлено в Гб) | Total Full Maintenance Backup space Required if cmsdbs Dbspace is full (GB)1 |
| ---------------- | -------------- | -------------------------------- | ------------------------------------ | ------------------------------------------ | ---------------------------------------------------------------------------- |
| Dell R220        | 8,192          | 43,844,355                       | 359,172,956,160                      | 334.5                                      | 11.15                                                                        |

If ontape is being used for binary backups this value must be multiplied by 30 since on tape does not compress data.
#### Процедура CMS Maintenance BackUp
Из главного меню CMS: Maintenance → Back Up Data→ List devices. F5 — закрыть окно списка устройств.
Ввести имя устройства USB. Выбрать Run.
Выбирать `Verification - y` не нужно, т.к. проверка работает только для кассет.
Выбрать y для продолжения. Проверка прогресса бэкапа:

```shellsession
$ tail -f /cms/maint/backup/back.log
```

В логе */cms/maint/backup/back.log* после завершения бэкапа появится примерно такой текст:

```
state: 1
/cms/install/bin/compress_backup successfullty finished:
<Day>, <timestamp>
error:
status: Last backup finished >, <timestamp>.
state: 0. 
```

#### Полный бэкап Full Maintenance Backup (проверить)

Ошибка в CMS Supervisor:
Из PuTTY:

```shellsession
$ su - cms cms
```

Then CTR +P+4 select timetable

![](image134.png)

Выбрать *Get contents* для полного бэкапа *fullBackup*

![](image135.png)

Выбрать Task номер 1 для выполнения полного бэкапа *maintenance backup*, выбрать *Modify*.

![](image136.png)

Поставить \"x\" в пункте *Local system administration data*, выбрать *Run* (исполнение).

![](image137.png)

### CMSADM backup
Ранее для ежемесячного бэкапа использовались кассеты. На текущий момент есть варианты бэкапа:
- Symantec Netbackup 7.5 Server, LAN backup
- Tivoli Storage Manager (TSM) Server 6.3.3 и Tivoli Storage Manager Client 6.2.1, LAN backup
- USB флешка (вместо кассет)
- NFS монтируемая файловая система (вместо кассет)

CMSADM backup не сохраняет таблицы базы данных Informix, за это отвечает Maintenance backup!!!  Во время бэкапа другие пользователи, кроме того, от лица которого делается бэкап, не могут залогиниться.
#### Определяем необходимое место на диске
Проще сразу узнать суммарный объем каталогов, которые будут сохраняться при бэкапе (интересует Used):

```shellsession
# df -h {/,/cms,/export/home,/opt,/var} --total
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2       9.8G  1.3G  8.0G  14% /
/dev/sda3       9.8G  373M  8.9G   4% /cms
/dev/sda7        32G   49M   30G   1% /export/home
/dev/sda10       12G  1.1G   10G  10% /opt
/dev/sda8        26G  166M   25G   1% /var
total            88G  2.9G   81G   4%
```

Место, необходимое для NFS Admin backup — 2,9Гб.
То же самое в Килобайтах:

```shellsession
# df {/,/cms,/export/home,/opt,/var} --total
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/sda2       10190132 1318396   8347452  14% /
/dev/sda3       10190132  381756   9284092   4% /cms
/dev/sda7       32896876   49352  31169804   1% /export/home
/dev/sda10      12189612 1117008  10446736  10% /opt
/dev/sda8       26704120  169832  25171140   1% /var
total           92170872 3036344  84419224   4%
```

Если нужна точность до нескольких знаков после запятой, скачать http://qalculate.sourceforge.net/
Вот размер в Гб с тремя знаками после запятой:

```bash
qalc -t -set "precision 3" 3036344B
```

Общая формула для qalc:

```bash
human_readable="$( qalc -t set "precision $precision" "${in_bytes}B" )"
```

В документации Avaya указан не очень удобный способ, т.к. суммировать занятый объем каждого раздела необходимо вручную. Также сумма, которую получили в предыдущем примере, более точная, т.к. здесь суммируем уже округленные ранее значения.

```shellsession
# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2       9.8G  1.3G  8.0G  14% /
tmpfs           3.9G     0  3.9G   0% /dev/shm
/dev/sda1       546M   38M  480M   8% /boot
/dev/sda3       9.8G  373M  8.9G   4% /cms
/dev/sda7        32G   49M   30G   1% /export/home
/dev/sda10       12G  1.1G   10G  10% /opt
/dev/sda6       9.5G  2.1G  7.0G  23% /storage
/dev/sda9        16G   45M   15G   1% /tmp
/dev/sda8        26G  166M   25G   1% /var
```

Суммируем необходимое место для CMSADM backup:

```
Каталог	      Сколько использовано
/	            1,3 Гб
/cms	        373 Мб = 373/1024 Гб =0,3643 Гб
/export/home	49 Мб = 49/1024 Гб = 0,048 Гб
/opt	        1,1 Гб
/var	        166 Мб = 166/1024 Гб = 0,1621 Гб
Сумма	2,9744 Гб
```

#### Когда надо делать CMSADM backup
- After the CMS is provisioned to backup the RHEL system files, system programs and CMS configuration data placed on the computer by Avaya Services provisioning personnel. These CMSADM backups can be to tape, a USB storage device or a network mount point and should also be saved and not reused or overwritten.\ Важно: CMS R16.2+ поддерживают CMS бэкапы на разные устройства. Avaya больше не поставляет кассеты с серверами CMS. If a customer chooses to use tape drives to back up the customer's CMS data then the customer is responsible for purchasing the tape drive and any supplies needed to operate the tape drive. The customer is responsible for backing up CMS after the system has been provisioned. The customer must store the CMSADM backup in a safe place in case the system needs to be restored.
- After the CMS server is provisioned\ This backup contains the RHEL system files and programs and CMS configuration data placed on the computer by Avaya Services provisioning personnel. These tapes should also be saved and not reused. In addition, field technicians should perform a CMS full maintenance backup before they turn a new system over to the customer. For more information, see Avaya Call Management System Administration.
- До и после обновления софта CMS (выполняется инженером)
- Раз в месяц (выполняется самим заказчиком).

!!! note
    Надо записать точную версию загрузчика CMS и информацию об устройстве бэкапа/восстановления для помощи в аварийном восстановлении CMS.

Версия CMS:

```shellsession
# rpm -q cms
cms-R18.0.2.0-ma.k.x86_64
```
#### Форматирование флешки (ext4)
Файловая система usb — ext4. На флешке можно держать несколько бэкапов.
Проверить, что CMS видит флешку (скорее всего она */dev/sdb*):

```bash
fdisk -l
```

Размер и доступное место на флешке:

```bash
df -kl
```

Создание файловой системы ext4 на флешке и монтирование в каталог */CMS_Backup* (в корне):

```shellsession
cd /
mkfs -t ext4 /dev/sdb
mkdir /CMS_Backup
mount /dev/sdb /CMS_Backup       #  точка монтирования /CMS/Backup
ls -l /CMS_Backup                #  проверка, что монтирование сделано
```

Создать любой файл на флешке для проверки, что работает запись и чтение.
Обновить права записи и чтения для созданных каталогов, чтобы любой авторизированный пользователь мог сделать бэкапы.
После вставки флешки монтировать, после вытаскивания размонтировать. При перезагрузке CMS снова монтировать:

```shellsession
mount /CMS_Backup
umount /CMS_Backup
```

Из CMS Supervisor или аналогичного меню из командной строки: *Maintenance → Backup/Restore Devices*
Ввести имя устройства и путь к нему */CMS_Backup*.
*Description→ Device Type - Other→ Add*
Если путь к USB отсутствует, появится сообщение об ошибке:

```
Path nor valid for type other
Press Return to continue
```

Убедиться, что флешка доступна.
Посмотреть весь список устройств бэкапа - List devices
#### Процедура бэкапа
Убедиться, что CMS в многопользовательском режиме (2 или 3).

```
who -r
```

USB флешка установлена и настроена. Проверить доступное место (должно быть не менее того что нам необходимо, определяли ранее этот объем):

```
df -kl
```

Залогиниться как root.
Ввести cmsadm, отобразится меню администрирования CMS
Ввести номер пункта меню, который отвечает за бэкап (номер 3). Выбрать тип Other (номер 2)

```shellsession hl_lines="8"
 (mrc-krl15-ucacms1)-(root)=# cmsadm

 Avaya(TM) Call Management System Administration Menu

Select a command from the list below.
   1) acd_create   Define a new ACD
   2) acd_remove   Remove all administration and data for an ACD
   3) backup       Filesystem backup
   4) pkg_install  Install a feature package
   5) pkg_remove   Remove a feature package
   6) run_pkg      Turn a feature package on or off
   7) run_ids      Turn Informix Database on or off
   8) run_cms      Turn Avaya CMS on or off
   9) passwd_age   Set password aging options
  10) dbaccess     Change Informix DB access permissions
Enter choice (1-10) or q to quit: 3

Choose a backup device:
  1) Tape
  2) Other
Enter choice (1-2): 2
Enter backup path (must not be located on CMS disk) [q]:
```

Выбрать путь на флешке, например */CMS_Backup*
Запустится CMSADM бэкап. Для мониторинга статуса ввести:

```
tail -f /cms/install/logdir/backup.log
```

Avaya рекомендует скопировать бэкап с флешки куда-то еще для подстраховки на случай её поломки. 
#### Проверка бэкапа на USB флешке
Вставить USB флешку и просмотреть список файлов.

```bash
ls -l /CMS_Backup
```

Проверка индивидуальных CMSADM файлов:

```bash
cpio -ivct -C 10240 -I /{mount_point}/<CMSADM_filename> | more
```

`<CMSADM_filename>` — имя CMSADM бэкапа. Пример:

```bash
cpio -ivct -C 10240 -I /CMS_Backup/CMSADM-r18ab.t-121116151708-digger | more
```

Имя бэкапа содержит в себе:
Тип бэкапа: CMSADM
CMS версия в момент создания бэкапа: r18ab.t
Дата создания бэкапа: 121116(формат yymmdd)
Уникальный идентификационный номер бэкапа: 151708
Имя хоста сервера CMS: digger
Для остановки просмотра содержимого архива с бэкапом нажать Delete.
## Увеличение места на диске для больших конфигураций
Для маленьких и средних конфигураций не нужно делать. Во время добавления дисков работа сервисов не затрагивается.
Убедиться что IDS включен

```bash
. /opt/informix/bin/setenv
onstat -
```
IDS is on if the output shows IDS is On-Line.
5. If IDS is not on-line, enter the following commands on the Linux console:
a. Enter:
```
cmssvc
```
The system displays a warning that IDS is off, then displays the CMS Services menu.
b. Enter the number associated with the run_ids option.
c. Enter the number associated with the Turn on IDS option.
6. Right-click the CMS virtual machine and select Edit Settings. The system displays the Virtual Machine Properties window.
7. Select Add.... The system displays the Add Hardware window.
8. Select Hard Disk.
9. Select Next.
10. Select Create a new virtual disk.
11. Select Next.
12. In the disk size field, enter the amount of additional disk space you require depending on the configuration shown in the following table:
Important: The 5.1 and 5.2 vSphere software only allows you to add disks 2 TB at a time. Because of this, you must not enter the entire disk size shown in the table. You must repeat this procedure until the space on the disks add up to the required size. For example, if you require 5.9 TB, you must add two 2-TB disks and a 1.9-TB disk.
13. Select Thin Provision.
14. Select Next. Maintenance operations
Important: The system displays a warning message if your VMware deployment does not yet have enough physical disks to support the size you are adding. You can select OK and continue with this procedure, but you must eventually add more physical disk storage to your deployment.
15. Select Next. The system displays the Device Node window.
16. Select Next.
17. Select Finish. The system displays a summary window.
18. Select OK.
The system adds the new disk. If you need to add more disks, repeat this procedure starting with Step 6 on page 34.
19. Select the Console tab. The system displays a console terminal window.
20. Log on as root.
21. Enter the following command to initialize the new disks:
```bash
/opt/informix/bin/dbinit.sh add_disks
```

Verify that the disks were added successfully. If the procedure fails, contact Avaya support.
## Проблемы
### CMS: Designer report not working on the CMS Web supervisor R18.
Doc ID: 	   	SOLN305626
Updated: 	   	07 Mar 2017

```shellsession
cmsweb status – cmsweb is running
rpm –qa cmsweb - cmsweb-R18.0.0.2-gc.a.x86_64
rpm –qa cms – cms-R18.0.0.2-gc.a.x86_64
```

A) Real-time→ Designer one report is not working on the CMS Web Supervisor while it run fine on the CMS supervisor.
B) Historical → Designer one report is not working on the CMS Web supervisor while it run fine on the CMS supervisor.

Cause
A) This issue with the customer’s R/T designer report not working in CMS Web client is known issue and is being addressed in JIRA CMS-599, which is targeted for the R19 CMS release.
B) The issue with the calculations used in the designer report. All the calculations were in small letter alphabets.
`gbjlaAHT_SUM`
`gbjlaEXT_SUM`
`gbjlaEXT_TIME_SUM`
Solution

A) JIRA CMS-599 raised to fix in the next release.
Workaround: The issue is that this report uses two data tables (not data base tables) on the report, one for the vdn data and one for the agent data. The problem is in the web client the table size is causing the first table from query to overlay on top of the second table, thus causing the error observed.  The customer can try to manipulate the table format to see if can get this not to occur, but may not be successful. Once suggestion I have for them is to try setting the table format from the current horizontal orientation to vertical, at least with the vdn table display.  Then both tables can fit on the single web window for viewing.
B) Product house will fix in the next release of the CMS version.
Workaround: Customer must used the capital letters to resolve this issue on the CMS Web supervisor.
GBJLAAHT_SUM
GBJLAEXT_SUM
GBJLAEXT_TIME_SUM
### Не работают отчеты из CMSWEB, команда cmsweb не работает
Doc ID: SOLN288729
CMS version: R18.0.0.1-fb.g.x86_64
CMS WEB version:cmsweb-R18.0.0.1-fb.g.x86_64
Getting error while accessing the reports in CMSWEB
"Failed to set the report input on Monday Error code=101 error message=internal error Call services”

While accessing cmsweb command getting error as:

```shellsession
(devcms)-(root)=# cmsweb status
-bash: cmsweb: command not found
```

Причина: каталог */cms* забился на 100%.
Очистить временные файлы и core дампы из каталога */cms*. Также очистить кэш браузера.
Проблема того что не работает команда cmsweb:

```shellsession
(devcms)-(root)=# whereis cmsweb
cmsweb: /opt/cmsweb/bin/cmsweb.sh
(devcms)-(root)=# which cmsweb
(devcms)-(root)=# file /usr/bin/cmsweb
/usr/bin/cmsweb: cannot open `/usr/bin/cmsweb' (No such file or directory)
```

Ссылка отсутствует, создать её. После этого команда заработает.

```shellsession
(devcms)-(root)=#ln -s /opt/cmsweb/bin/cmsweb.sh /usr/bin/cmsweb
```

