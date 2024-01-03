---
title: SMGR — Avaya System Manager
share: "true"
---

## Постоянно приходится менять пароль к SMGR CLI
Пароль для пользователя `smgradmin` на вход по ssh по умолчанию надо менять каждые 2 месяца. Убираем или настраиваем нужный срок.
Вариант 1. Из веб-интерфейса.
*Users→ Administrators→ Administrative Users→ Security→ Policies*. Раздел *Password Policy*. Поставить **Aging: Password policy for aging is disabled**. Как сделать: зайти в это меню и снять галку *Enforce password aging policies*.

Вариант 2. Из Linux CLI

```
root> cat /etc/shadow
…..
insights:!!:17669::::::
admin:$1$FiO.sG..$m7Hqu/q8kdSMj/.iBseJa/:17737::99999:::99999:
smgr::17674:1:60:7:::
nortel:!!:17674:1:60:7:::
csadmin:!!:17760:0:99999:7:::
smgradmin:$1$fWHRaY/s$.NaOt.JIH.7ZFFdpyrleR/:17806:1:60:7:::

root >chage smgradmin
Changing the aging information for smgradmin
Enter the new value, or press ENTER for the default

        Minimum Password Age [1]:
        Maximum Password Age [60]: 99999
        Last Password Change (YYYY-MM-DD) [2018-10-02]:
        Password Expiration Warning [7]:
        Password Inactive [-1]:
        Account Expiration Date (YYYY-MM-DD) [-1]:
```

## SDM (Solution Deployment Manager)
`C:\Program Files\Avaya\AvayaSDMClient\Default_Artifacts` — все образы из этого каталога будут видны в SW Library.
### Настройка Software Library
В качестве библиотеки софта для обновления плат MM и шлюзов G450, G430 и др. настраивается локальный FTP сервер.

!!! warning "Внимание"
    Для обновления прошивок плат TN нельзя использовать System Manager, нужен удаленный FTP сервер.

В веб-интерфейсе *Services→ Solution Deployment Manager→ Software Library Management*. Выбрать **SMGR_DEFAULT_LOCAL** и нажать **Edit**. Появится страница *Edit Software Library*.

На вкладке Library Server Details (L) в Default Protocol нажить FTP.
Во вкладке FTP Configuration (F) поставить галочку Enable FTP.
User Name — admuser
Password и Confirm Password — пароль.
Нажать **Commit**.

На левой панели нажать *Download Management* и скачать необходимую прошивку плат MM в **SMGR_DEFAULT_LOCAL** библиотеку.

Путь к каталогу локальной библиотеки: */swlibrary/swLib/local*

![](image211.png)

### Как вручную скачать образы в библиотеку
Скачать образ OVA в каталог */swlibrary/staging/sync* через WinSCP

Зайти в веб-интерфейс SMGR: *Services→ Solution Deployment Manager→ Software Library Management*. Выбрать требуемую библиотеку. Нажать *Manage Files*. В *Sync Files from directory* поставить галочку за именем образа OVA файла. Ввести контрольную сумму OVA файла в поле MD5 Checksum.

В поле *Software Library* выбрать локальную System Manager библиотеку.
Product Family field — выбрать Avaya Aura Media Server.
Device Type — выбрать OVA.
Software Type field — выбрать OVA.
Нажать **Sync**.

Дождаться окончания процесса upload.

(Опционально) выполнить шаги для просмотра закачанного файла в библиотеке (проверка):
*Services→ Software Library Management*.
Галочку за именем библиотеки, содержащей OVA. Нажать *Manage Files*.
### Переполнение библиотеки swlibrary
Периодически надо очищать папку */swlibrary*. Но есть нюанс: даже при очистке через `rm -rf каталог` команда `df -h` показывает, что занято очень много места. Но `du -sh` показывает, что каталоги внутри swlibrary практически не заняты.

Причина: в Linux удаление файла через `rm` или файловый менеджер удаляет ссылку на файл из структуры каталогов файловой системы, тем не менее, если файл все ещё открыт (используется исполняющимся процессом), он будет доступен для процесса и продолжит занимать место на диске.
Решение: перезагрузить SMGR или, что более правильно, только сервис **jboss**:

``` hl_lines="27"
root >systemctl restart jboss
Perform cleanup ....
Stopping System Manager JBoss - PID: 3170
.............................................
Stopped System Manager JBoss
              total        used        free      shared  buff/cache   available
Mem:       12138328     7913792     2732236      514444     1492300     3231096
Swap:       4194300     1807108     2387192
              total        used        free      shared  buff/cache   available
Mem:       12138328     2250180     9111056      514444      777092     9012400
Swap:       4194300     1263108     2931192
Starting System Manager JBoss
Redirecting to /bin/systemctl restart postgresql.service
Redirecting to /bin/systemctl status postgresql.service
.
Service started successfully - PID: 7703

root >df -h
Filesystem                             Size  Used Avail Use% Mounted on
/dev/mapper/smgrvg01-lv_root           4.2G  1.9G  2.3G  45% /
devtmpfs                               5.8G     0  5.8G   0% /dev
tmpfs                                  5.8G  8.0K  5.8G   1% /dev/shm
tmpfs                                  5.8G  610M  5.2G  11% /run
tmpfs                                  5.8G     0  5.8G   0% /sys/fs/cgroup
/dev/sdc                                15G  222M   15G   2% /emdata
/dev/sdb                                25G  254M   25G   1% /perfdata
/dev/sdd                                23G  330M   22G   2% /swlibrary    # а было 59%
/dev/sda1                              509M  139M  370M  28% /boot
/dev/mapper/smgrvg01-lv_var            4.5G  1.2G  3.4G  25% /var
/dev/mapper/smgrvg01-lv_var_log        1.9G  757M  1.2G  40% /var/log
/dev/mapper/smgrvg01-lv_home           1.9G  343M  1.6G  18% /home
/dev/mapper/smgrvg01-lv_data            14G  883M   13G   7% /var/lib/pgsql/data
/dev/mapper/smgrvg01-lv_var_log_audit  1.9G  197M  1.7G  11% /var/log/audit
/dev/mapper/smgrvg01-lv_opt            8.7G  3.5G  5.2G  41% /opt
/dev/mapper/smgrvg01-lv_tmp            1.5G   35M  1.5G   3% /tmp
tmpfs                                  1.2G     0  1.2G   0% /run/user/778

root>lsof | grep deleted
```
## Переполнение каталога /var в SMGR 7.0.0.0
Патч 1 для SMGR 7.0.1.3 — [PSN004806u](https://downloads.avaya.com/css/P8/documents/101043232).

Признаки переполнения:
- невозможно выполнить никакие отчеты
- не работает веб-интерфейс SMGR
- При залогинивании в System Manager получаем ошибку: «Some internal error has occurred in the service. Please contact the support team».
- Иногда не работает локальный бэкап: «line 34: echo: write error: No space left on device».

Видим, что диск переполнен, причем из-за каталога */var*:

```bash hl_lines="3 7"
[root@SMGR-SYM ~]# df -kh
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda3       4.8G  4.8G     0 100% /
tmpfs           5.9G     0  5.9G   0% /dev/shm
/dev/sda6       2.0G  326M  1.5G  18% /home
/dev/sda1       6.8G  5.0G  1.5G  78% /opt
/dev/sda2       6.8G  6.4G   23M 100% /var
/dev/sda7        15G  4.6G  9.3G  34% /var/lib/pgsql
/dev/sdc         15G   63M   14G   1% /emdata
/dev/sdb         25G  4.1G   20G  18% /perfdata
/dev/sdd         25G  2.7G   21G  12% /swlibrary
```

Временное решение проблемы: проверить, какой каталог внутри /var занимает больше всего места:

```bash
du -h --max-depth=1 /var
```
Очистить старые heap dump и ненужные большие файлы. Если диск полностью забит, найдем файлы более 100MB в корневом разделе SMGR:

```bash
find / -type f -size +100M -exec ls -lh {} \; 2>> /dev/null
```

Пример удаления. Старые файлы heap dump были внутри каталога */opt/Avaya/HeapDump*, альтернативное расположение */var/jboss/HeapDump/*.

```
-rw------- 1 admin admin 6.0G Sep 24  2013 /opt/Avaya/HeapDump/SMGR_JBOSS_HEAP_2013Sep24_181506.hprof
-rw------- 1 admin admin 6.0G May 11  2014 /opt/Avaya/HeapDump/SMGR_JBOSS_HEAP_2014May11_000507.hprof
```
## Не работает CM Notify Sync (при изменении данных на CM в ASA и не обновляет их в SMGR)
Пример: меняем `COR` на station через Avaya Site Administration. При дальнейшем изменении этого же station в SMGR выясняется, что там все еще старые данные. Т.е. если какой-то другой пункт меняем в SMGR, параллельно с этим мы вернем старый COR для station.

В истории команд после изменений через ASA автоматически должен залогиниться специально выделенный логин для smgr, но этого не происходит. Ниже меняем два раза station 21107, логинимся в ASA как `cmadmin`. Далее в истории ничего не появляется.

```
list history                                                           Page   1

                                  HISTORY
          Date of Loaded Translation: 12:45pm Fri Nov 16, 2018

 Date  Time Port      Login    Actn Object       Qualifier

12/26 15:52 MAINT     cmadmin  cha  station      21107
12/26 15:51 MAINT     cmadmin  cha  station      21107
12/26 15:51 MAINT     cmadmin  cha  secondary-us
12/26 15:50 MAINT     cmadmin  logn
12/26 15:24 PHONE     chgsta   cha  station      21248
12/26 14:35 PHONE     chgsta   cha  station      21108
12/26 14:11 PHONE     chgsta   cha  station      21108
12/26 14:02 PHONE     chgsta   cha  station      21108
12/26 13:57 PHONE     chgsta   cha  station      21108
12/26 13:57 PHONE     chgsta   cha  station      21108
12/26 13:49 PHONE     chgsta   cha  station      21108
12/26 13:37 MAINT     cmadmin  logf
12/26 13:37 MAINT     cmadmin  cha  secondary-us
12/26 13:37 MAINT     cmadmin  logn
```

В документации функции:
- Оne way CM notify sync feature
- Two way CM notify sync feature

We restored the original configuration and then specified the one-way TLS configuration by adding SMGR certicate in the Communication Manager. Finally we modified the /etc/hosts file on MSGR server by removing the FQDN name of Communication Manager and left short hostname for CM. After restarting the jboss service on SMGR and CM Notify sync started to work.

When you perform an administrative task from System Manager, the local database is immediately updated. If you execute the action through a Communication Manager SAT screen, or through aphone, or from any of the several management applications such as Site Administration, MultiSite Administration, Native Configuration Manager, or MyPhone, it is not immediately reflected in SystemManager. This scenario creates an out-of-sync condition between the Communication Manager andSystem Manager.

 The CM notify sync feature provides near-real time notifications from Communication Manager to System Manager whenever you execute certain tasks against a Communication Manager objectfrom a system other than System Manager. The CM notify sync feature also provides notifications whenever the tti-m, tti-s, psa-u, psa-a, or psa-d logins perform their predefined actions against a Communication Manager station object.

 After a Communication Manager sends notifications to System Manager, System Manager discovers the complete details of the task you preformed. The transmission of notifications in the form of event messages from Communication Manager to System Manager is based on the Communication Manager’s existing rsyslog capability. The Communication Manager’s rsyslog uses UDP or TCP to send event messages from the originating Communication Manager to the System Manager.

!!! note
    The existing daily default synchronization and any other scheduled synchronization operations are unaffected by the CM notify sync feature.
    You need Communication Manager with version 6.2 or above for to enable the CM notify sync feature. System Manager 6.3 supports both one-way and two-way TLS.

Enabling the CM notify sync feature :
You can enable and disable the CM notify sync feature on a per Communication Manager basis.

You can activate the CM notify sync feature from a new System Manager using Manage Elements. Select Communication Manager 6.2 or a higher version, and select Enable Notifications in the Attributes section. As a system administrator, you must specify the IPs of one or two System Managers to which the Communication Managers send the event data using rsyslog. If your configuration includes two System Managers, the standby System Manager ignores the syslog messages until it becomes active.

You can have one-way and two-way TLS.
Please follow [link](https://downloads.avaya.com/css/P8/documents/100182973) for how to activate this feature. Page No 1184.
## Не работает High Availability
Пропадает адрес второго *CM Home→ Services→ Inventory→ Manage Elements*→ открыть *Name* - общее имя двух CM.
Проблема наблюдается при включенной опции SNMP. Если выключить SNMP, Alternate IP Address не пропадает.
Как решила Avaya: под root-ом в */etc/sysconfig/network* прописали hostname. Пример.

```ini title="/etc/sysconfig/network"
NETWORKING=yes
NETWORKING_IPV6=no
GATEWAY=10.8.12.1
IPV6_AUTOCONF=no
HOSTNAME=sng-nv-abk1-ucacm1   (или ucacm2)
```

Как решить без прав root-a. Зайти в веб-интерфейс CM1 в *Server Configuration→ Network Configuration*. В поле *Host Name* ввести собственное имя сервера sng-nv-abk1-ucacm1, в *Alias Host Name* ввести общее имя для двух CM sng-nv-abk1-ucacm. Аналогично сделать в веб-интерфейсе CM2, введя свое имя sng-nv-abk1-ucacm2 и общее имя sng-nv-abk1-ucacm.
## Список доступных команд SMGR (/etc/hosts и другие)
Т.к. нет доступа root, то изменение некоторых системых файлов возможно через специальные команды, которые Avaya заранее обеспечила. Например с логином `cust` выполнением команды `editHosts`  изменяем файл */etc/hosts*. Надо скопировать и вставить оригинальное содержимое файла */etc/hosts* в первую строку, затем добавить IP адрес и FQDN к записи, которую хотим дополнить. При сохранении полностью ввести **Yes** вместо одной буквы **y**.

В SMGR CLI ввести `alias` для вывода списка доступных конечному пользователю команд. При их выполнении будет запрошен для `sudo` тот же пароль, который используем при входе в SMGR CLI:

```bash title="вывод команды alias"
EASGManage='/opt/Avaya/vsp/toggleEASGState.sh'
EASGSiteCertManage='sudo /sbin/EASGSiteCertManage'
SMGRPatchdeploy='sudo /opt/Avaya/vsp/VMWrapper/applyPatch.sh'
changeIPFQDN='sudo /opt/Avaya/vsp/VMWrapper/changeIPFQDN.sh'
changePublicIPFQDN='sudo /opt/Avaya/vsp/OOBM/changePublicIPFQDN.sh'
changeVFQDN='sudo /opt/Avaya/vsp/SMGRVirtualFqdnUtility.sh'
collectLogs='sudo /opt/vsp/collectLogs.sh'
configureNTP='sudo /opt/Avaya/vsp/VMWrapper/configureNTP.sh'
configureOOBM='sudo /opt/Avaya/vsp/OOBM/configureOOBM.sh'
configureTimeZone='sudo /opt/Avaya/vsp/VMWrapper/configureTimeZone.sh'
createCA='sudo /opt/Avaya/Mgmt/7.1.11/trs/utility/ca_renewal/createCA.bin'
editHosts='/opt/Avaya/vsp/setHostsFile.sh'
egrep='egrep --color=auto'
enableMUDGHardening='sh /opt/Avaya/setSecurityProfile.sh --enabledod'
enableOOBMMultiTenancy='sudo /opt/Avaya/vsp/OOBM/enableMultitenancyInPublicInterface.sh'
fgrep='fgrep --color=auto'
getUserAuthCert='sudo /opt/Avaya/Mgmt/7.1.11//trs/utility/getUserAuthIDCert.sh'
grep='grep --color=auto'
l.='ls -d .* --color=auto'
ll='ls -l --color=auto'
ls='ls --color=auto'
mountNFS_PerfData.sh='sudo /opt/Avaya/Session_Manager/perf/mountNFS_PerfData.sh'
move_to_perfdata-remote.sh='sudo /opt/Avaya/Session_Manager/perf/move_to_perfdata-remote.sh'
move_to_perfdata.sh='sudo /opt/Avaya/Session_Manager/perf/move_to_perfdata.sh'
pairIPFQDN='sudo /opt/Avaya/vsp/VMWrapper/pairIPFQDN.sh'
poweroffVM='/opt/Avaya/vsp/rebootSMGR'
refreshDbKS='sh /opt/Avaya/Mgmt/7.1.11/infra/bin/refreshDBKeyStore.sh'
serviceJBossRESTART='sudo systemctl restart jboss.service'
serviceJBossSTART='sudo systemctl start jboss.service'
serviceJBossSTATUS='sudo systemctl status jboss.service'
serviceJBossSTOP='sudo systemctl stop jboss.service'
setSecurityProfile='sudo /opt/Avaya/setSecurityProfile.sh'
smgr='sudo service jboss'
smgr-db='sudo service postgresql'
status_vm='/opt/Avaya/vsp/SMGREnvoirnment.sh'
swversion='sh /opt/Avaya/vsp/swversion.sh'
toggleWeblmOldcert='sudo /opt/Avaya/Mgmt/7.1.11//utils/toggleWeblmOldcert.sh'
unmountNFS_PerfData.sh='sudo /opt/Avaya/Session_Manager/perf/unmountNFS_PerfData.sh'
updateASG='sudo /opt/Avaya/vsp/VMWrapper/updateASG.sh'
upgradeSMGR='sudo /opt/Avaya/vsp/VMWrapper/upgradeSMGR.sh'
vi='vim'
which='| /usr/bin/which --tty-only --read---show-dot --show-tilde'
```

!!! note "Примечание"
    Команды serviceJBossRESTART, serviceJBossSTART,serviceJBossSTATUS и serviceJBossSTOP не работают в 7.1.2 и 7.1.3, хотя алиасы к ним остались. Алиасы удалены в 7.1.3.1. Вместо их алиасов использовать команды проверки/перезапуска сервиса JBoss:

```bash
smgr status
smgr stop
smgr start
smgr restart
```

Change the directory to the above, i.e. */opt/Avaya/vsp/* for editHosts, once there type `./setHostsFile.sh` , you will then be prompted for the `sudo` password

Начиная с SMGR 7.1+ у Customer/BP не будет аккаунта с привилегиями админа. UserID, который мы создадим во время установки, будет ID для входа в CLI и веб-интерфейс (в нашем случае smgradmin). Also admin CANNOT be a user that is configured when deploying SMGR, or it will cause problems.

The [SMGR Deployment guide](https://downloads.avaya.com/css/P8/documents/101038429) on page 76 states it. For web interface, admin access would be still available.
Please used below [link](https://downloads.avaya.com/css/P8/documents/101038429) from page 160

## Обновление SMGR
Сначала ставим сервис-пак, затем патч. Сервис-пак `System_Manager_7.1.3.2_r713208362.bin`

```
System Manager 7.1.3.2
Security Mode – Standard Hardening
Build No. - 7.1.0.0.1125193
Software Update Revision No: 7.1.3.2.058362
Service Pack 2
```

Установка патча из командной строки System Manager CLI для VMWare илиAvaya Virtualization Platform. Если несколько SMGR, инструкции поустановке патча не отличаются.
1. Снять **snapshot** виртуальной машины System Manager. Это может повлиять на сервис!
2. Скопировать установочный файл патча `System_Manager_7.1.3.3_r713309127.bin` на SMGR в каталог */swlibrary/*

```
smgradmin >ls
AnnouncementData
AVP_Bulk_Import_Spreadsheet_Template.xlsx
cdmount
HeapDump
lost+found
reports
sdm-api
staging
System_Manager_7.1.2.0_r712007353.bin
System_Manager_7.1.3.3_r713309127.bin  
System_Manager_R7.1_r710006654_mandatoryPatch.bin
tmp
twiddle.log
```

3. Зайти по ssh в командную строку с логином пользователя, который был создан при установке 7.1 OVA.
4. Проверить контрольную сумму файла патча в соответствии с данными наAvaya PLDS, для этого патча (63524a14792ffc5aa6ea25d28b4b940d)
5. Выполнить установку патча. Примечание: будет запрос на принятие EULA. Согласиться с EULA дляустановки. Также повторно запросится пароль smgradmin.

```bash
SMGRPatchdeploy <абсолютный путь к файлу System_Manager_7.1.3.3_r713309127.bin>
```

Если не получаетcя, можно через стандартный вариант Linux

```bash
sh <абсолютный путь к файлу System_Manager_7.1.3.3_r713309127.bin>
```

6. Ждать процесса окончания установки. Если окно ssh неожиданно закрылось во время установки патча, значит система повреждена или перешла в несовместимое состояние. Тогда лучше всего восстановить исходное состояние, вернувшись к снапшоту и попытавшись снова запустить установку патча.
7. Залогиниться в веб-интерфейче для проверки. Если появится предупреждение « Virtual Machine needs to be rebooted as System Manager Patch installation updated the Kernel», надо перезагрузить виртуальную машину.

![](image212.png)

То же самое будет после перелогинивания в командной строке:

```
WARNING! The remote SSH server rejected X11 forwarding request.
Last login: Sat Mar  2 11:23:42 2019 from 172.31.12.32

	* * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * *
	*							              *
	* !! Virtual Machine needs to be rebooted as System Manager Patch     *
        *    installation updated the Kernel. !!                              *
	*							              *
	* * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * *
```

Можно вывести список alias (см. список доступных команд), там есть команда `rebootVM`

```bash
alias rebootVM='/opt/Avaya/vsp/rebootSMGR -r'
```

После ввода `rebootVM` надо ввести пароль для `smgradmin` и для соглашения обязательно **Yes**, а не просто **y**.

```
smgradmin >rebootVM
[sudo] password for smgradmin: 

This Script will reboot the VM

Do you want to continue ? .. (Yes/No)
Yes

Broadcast message from root@mrc-krl15-ucasmgr2.rtrn.ru (pts/0) (Sat Mar  2 12:33:28 2019):

Rebooting the VM
PolicyKit daemon disconnected from the bus.
We are no longer a registered authentication agent.

Socket error Event: 32 Error: 10053.
Connection closing...Socket close.

Connection closed by foreign host.

Disconnected from remote host(mrc-krl15-ucasmgr2) at 12:35:46.
```

В правом верхнем углу нажать на иконку и выбрать "About". Убедиться, что отображается следующая информация о версии:

```
System Manager 7.1.3.2
Security Mode – Standard Hardening
Build No. - 7.1.0.0.1125193
Software Update Revision No: 7.1.3.2.058700
Service Pack 2
```

!!! note
    Значение для Security Mode может зависеть от профиля безопасности Security Profile. "Standard Hardening" — по умолчанию для Security Mode.

![](image213.png)

8. Удалить снапшот, сделанный в первом шаге, после полной проверки функционала.
 
!!! warning
    Влияет на сервис.
### Не хватает места для обновления SMGR

```
smgradmin >SMGRPatchdeploy System_Manager_7.1.3.3_r713309127.bin 
 [sudo] password for smgradmin: 
Verifying the patch binary.....
Checking if patchplugin.log exists!
StrictModes yes
AllowTcpForwarding no
There is not enough space available to install this patch.  Please free up some space in /var & /opt partitions and try again.
Check /var/log/Avaya/SMGR_Patch.log for details.
```

Проверка показала, что забит каталог */opt*

``` hl_lines="10"
smgradmin >df -h
Filesystem                             Size  Used Avail Use% Mounted on
/dev/mapper/smgrvg01-lv_root           3.9G  1.8G  2.2G  44% /
devtmpfs                               4.4G     0  4.4G   0% /dev
tmpfs                                  4.4G  4.0K  4.4G   1% /dev/shm
tmpfs                                  4.4G  426M  3.9G  10% /run
tmpfs                                  4.4G     0  4.4G   0% /sys/fs/cgroup
/dev/sdc                                15G  139M   15G   1% /emdata
/dev/sdb                                25G  197M   25G   1% /perfdata
/dev/mapper/smgrvg01-lv_opt            6.8G  6.5G  249M  97% /opt
/dev/mapper/smgrvg01-lv_var            4.3G  1.2G  3.1G  27% /var
/dev/mapper/smgrvg01-lv_home           1.8G  387M  1.4G  22% /home
/dev/mapper/smgrvg01-lv_var_log        1.8G  1.3G  474M  74% /var/log
/dev/sdd                                25G  2.2G   22G  10% /swlibrary
/dev/mapper/smgrvg01-lv_data            14G  1.1G   13G   8% /var/lib/pgsql/data
/dev/mapper/smgrvg01-lv_var_log_audit  1.8G   34M  1.8G   2% /var/log/audit
/dev/mapper/smgrvg01-lv_tmp            1.5G   35M  1.4G   3% /tmp
/dev/sda1                              509M  148M  362M  29% /boot
tmpfs                                  884M     0  884M   0% /run/user/779
```

Здесь выведем список каталогов по убыванию их размера:

```
smgradmin> du -a | sort -rn | head -20
… многие каталоги не читаются
du: cannot read directory ‘./emdata/ipoffice’: Permission denied
13425484	.
6512780	./opt
6484216	./opt/Avaya
3858584	./opt/Avaya/JBoss/6.1.0
3858584	./opt/Avaya/JBoss
3858480	./opt/Avaya/JBoss/6.1.0/jboss-as
3754760	./opt/Avaya/JBoss/6.1.0/jboss-as/server/avmgmt
3754760	./opt/Avaya/JBoss/6.1.0/jboss-as/server
2303840	./opt/Avaya/SPIRIT/7.1.11
2303840	./opt/Avaya/SPIRIT
2249564	./swlibrary
2228936	./opt/Avaya/SPIRIT/7.1.11/LogTail_5049017714981803363.logbuff
2183080	./var
2026664	./opt/Avaya/JBoss/6.1.0/jboss-as/server/avmgmt/tmp
2026128	./opt/Avaya/JBoss/6.1.0/jboss-as/server/avmgmt/tmp/vfs
2026072	./opt/Avaya/JBoss/6.1.0/jboss-as/server/avmgmt/tmp/vfs/automountea9818f7a36fca50
1678304	./usr
1241592	./swlibrary/System_Manager_7.1.3.3_r713309127.bin
```

Также вариант поиска больших файлов:

```bash
find / -type f -size +100M -exec ls -lh {} \; 2>> /dev/null-
```

Узнать полный размер каталога:

```bash
ls -ltrh
```

Прав на удаление нет. Пробуем перезапустить сервис jboss:

```bash
smgr restart
```

Если снова в каталоге */opt* занято 97%, перегрузим виртуальную машину:

```bash
rebootVM
```

У меня после перезагрузки стало 65% вместо 97%.
### SMGR 7.1 : Upgrading SMGR from 7.1.2 to 7.1.3 returns CA Initialization failed. Unable to contact CA.
Doc ID:     SOLN326626
Updated:     13 Jul 2018
System Manager 7.1 Build 7.1.0.0.1125193 Feature Pack 3mLatest Build 7.1.3.0.037763

Upgrading SMGR from 7.1.2 to 7.1.3, The SMGR does get upgraded to 7.1.3 but when menu of Inventory→ Managed Elements→SMGR→Trusted or Identity certificate is clicked it gives the error
" Error while communicating to remote application.Please verify that the remote application is up and running".

It does not happen for any other element but only for SMGR. They are using Third Party certificates.

Cause
Customer had configured the 3rd party certificates , CRL ( `ldap:///CN=TelenorDKIssuingCA1,CN=PKI,CN=CDP,CN=Public%20Key%20Services,CN=Services,CN=Configuration,DC=int,DC=sonofon,DC=dk?certificateRevocationList?base?objectClass=cRLDistributionPoint`) download associated with this cert was not reachable from SMGR So CRL was not getting downloaded locally.

And also if you look at CRL URL it does not have CA server(like https://pki.telenor.dk/) details which is also strange.

If CRL would have got downloaded to system then there would not have been an issue over here.

Solution
1. login to web console
2. Security → Security Configuration
3. change settings of Certificate Revocation Validation from "BEST_EFFORT" to "NONE" and click commit button
4. After 10 minutes jboss will restart
5. Wait for 20 minutes Login to SMGR web console
6. if success then start upgrade to 7.1.3
### SMGR: Cannot install patch 7.1.2 (or 7.1.3) on top of 7.1.0
Doc ID:     SOLN324393
cannot install 7.1.2 getting patch signature verification fails when attempting

Cause
Security on 7.1 disabled port forwarding, when using SAL to connect directly to the smgr the patch install would fail.

Solution
Connect to another device, i.e. asm, cm, etc and ssh over to the smgr and perform the smgrpatchdeploy command

Example:
Connect to another Avaya product, e.g., CM → SSH into System Manager → Enable enrollment password → install patch with command:

```
SMGRPatchdeploy /swlibrary/patchname
```
