---
tags:
  - virtualization
title: Настройка OpenStack mail.ru
share: "true"
---
Рассмотрим настройку подключения к OpenStack Mail.ru, простую установку Python в Windows, подключение к mail.ru из Windows CLI и PowerShell, настройку xshell для работы с Windows CLI и OpenStack, некоторые команды OpenStack.

Варианты кроме Openstack:
- Qemu/KVM
- CoreOS + K8 + Terraform или OpenShift

### Введение
Throughout the years, corporate computing has seen many developments, which eventually have lead to the rise of cloud computing as we know it:
- In the 1990s, corporate computing was centered around servers in a datacenter.
- In 2000s, corporate computing was largely based on virtualization.
- In the 2010s, we are seeing the rise of cloud computing to leverage corporate computing.

The concept of "cloud computing" is very broad, to the point that we can affirm that there is no such thing as "cloud computing". If you ask an end-user to explain what cloud computing is, and then ask a system administrator the same question, you will get two different descriptions. In general, there are three important approaches when it comes to cloud computing:
- **IaaS (Infrastructure as a Service**): an infrastructure that is used to provide virtual machines
- **PaaS (Platform as a Service**): the provider supplies the network, servers, storage, OS and middleware to host an application
- **SaaS (Software as a Service**): the provider gives access to an application.
OpenStack belongs to the IaaS cloud computing category. However, OpenStack is continuously evolving, broadening its scope. On occasion, the focus of OpenStack goes beyond IaaS.

### Сети
Networking is a big topic in OpenStack. In the past, networking was defined by Nova. However, as networking became more and more complex, a separate project was created to deal with it: Neutron. Several processes can be involved in making Neutron Networking happening in OpenStack:
- neutron-openvswitch-agent
- neutron-dhcp-agent
- neutron-l3-agent
- neutron-metadata-agent
- neutron-ovs-cleanup
- neutron-lbaas-agent.

## Подключение к mail.ru
Условно порт (`port`) — это сетевая карта, или сетевой интерфейс. По умолчанию при создании инстанса (виртуальной машины) создается один порт, обычно из диапазона dhcp.

Инстанс (он же `server`) — виртуальная машина, сервер с операционной системой Linux (разные), Windows. Инстансы разворачиваются из общедоступных образов операционных систем, которые уже есть. Но можно и свой образ залить.

Фиксированные занятые ip-адреса в Mail.Ru:
`10.112.203.250` — основной адрес маршрутизатора Openstack, сейчас он указан для инстансов в dhcp в качестве шлюза. Через него доступен Интернет. На маршрутизаторе прописан статический маршрут на 10.0.0.0/8 через 10.112.203.251 (OpenBSD)
`10.112.203.19`  — дополнительный адрес маршрутизатора. Сказали, что он выделяется автоматом, но мне непонятно почему.
`10.112.203.15` — основной адрес DHCP сервера OpenStack. DNS-суффикс подключения — openstacklocal (жаль, что средствами OpenStack нельзя поменять на msk.quadra.ru.
`10.112.203.16` — дополнительный адрес DHCP сервера OpenStack

Веб-интерфейс:
Для выполнения команд выполнить `mail.ru.source`, затем `mail.ru.sh`

Скрипт `mail.ru.source` добавляет переменные окружения в Linux или Mac.

```bash title="mail.ru.source"
#!/usr/bin/env bash

# To use an OpenStack cloud you need to authenticate against the Identity
# service named keystone, which returns a **Token** and **Service Catalog**.
# The catalog contains the endpoints for all services the user/tenant has
# access to - such as Compute, Image Service, Identity, Object Storage, Block
# Storage, and Networking (code-named nova, glance, keystone, swift,
# cinder, and neutron).
#
# *NOTE*: Using the 3 *Identity API* does not necessarily mean any other
# OpenStack API is version 3. For example, your cloud provider may implement
# Image API v1.1, Block Storage API v2, and Compute API v2.0. OS_AUTH_URL is
# only for the Identity API served through keystone.
export OS_AUTH_URL=https://infra.mail.ru:35357/v3/

# With the addition of Keystone we have standardized on the term **project**
# as the entity that owns the resources.
export OS_PROJECT_ID=МногоБуквЦифр
export OS_PROJECT_NAME="pupkin_av@example.ru"
export OS_USER_DOMAIN_NAME="users"
if [ -z "$OS_USER_DOMAIN_NAME" ]; then unset OS_USER_DOMAIN_NAME; fi
export OS_USER_DOMAIN_ID="users"
if [ -z "$OS_USER_DOMAIN_ID" ]; then unset OS_USER_DOMAIN_ID; fi
export OS_PROJECT_DOMAIN_ID="МногоБуквЦифр"
if [ -z "$OS_PROJECT_DOMAIN_ID" ]; then unset OS_PROJECT_DOMAIN_ID; fi

# unset v2.0 items in case set
unset OS_TENANT_ID
unset OS_TENANT_NAME

# In addition to the owning entity (tenant), OpenStack stores the entity
# performing the action as the **user**.
export OS_USERNAME="pupkin_av@example.ru"

# With Keystone you pass the keystone password.
#echo "Please enter your OpenStack Password for project $OS_PROJECT_NAME as user $OS_USERNAME: "
#read -sr OS_PASSWORD_INPUT

export OS_PASSWORD="ТутПароль"

# If your configuration has multiple regions, we set that information here.
# OS_REGION_NAME is optional and only valid in certain environments.
export OS_REGION_NAME="RegionOne"
# Don't leave a blank variable, unset it if it was empty
if [ -z "$OS_REGION_NAME" ]; then unset OS_REGION_NAME; fi

export OS_INTERFACE=public
export OS_IDENTITY_API_VERSION=3
```
Скрипт mail.ru.sh
```bash
source mail.ru.source
openstack
```

## Настройки серверов OpenBSD на стороне Mail.ru
OSPF
```bash title="/etc/ospfd.conf"
# cat /etc/ospfd.conf
router-id 10.112.203.252
rfc1583compat yes
fib-update yes
stub router no
router-priority 0

redistribute 10.112.203.0/24
redistribute 10.112.203.252/24

area 0.0.0.0 {
        interface gre8 {
            metric 10
        }
}
```

Файрвол Packer Filter:
```bash title="/etc/pf.conf"
table <grey_lan> persist const {10.0.0.0/8,172.16.0.0/12,192.168.0.0/16}
table <sshspammers> persist

set skip on { lo bridge vether vxlan gre gif enc}
set block-policy drop
set optimization aggressive
set reassemble yes
match all scrub (max-mss 1420)

block log quick inet from <sshspammers>

# Iperf|iperf3 server access
pass in quick inet proto tcp from any to self port {5001,5201}


pass out quick inet from 10.112.203.0/24 to !<grey_lan> nat-to 95.163.182.60
pass in quick inet from 10.112.203.0/24 to ! <grey_lan>

block drop in on ! lo0 proto tcp to port 6000:6010


pass in quick inet proto tcp from any port >1024 to self port 6022 \
rdr-to 127.0.0.1 port 22 

pass out quick inet from 127.0.0.1 to any nat-to self
pass quick inet proto {carp,esp,pfsync} keep state (no-sync)
block drop log all
pass proto icmp
pass out inet from self to any
pass out inet from 10.0.0.0/8 to 10.112.203.0/24
pass in inet from 10.112.203.0/24 to 10.0.0.0/8
```

VPN

```bash title="/etc/npppd/npppd.conf"             
# $OpenBSD: npppd.conf,v 1.2 2014/03/22 04:32:39 yasuoka Exp $
# sample npppd configuration file.  see npppd.conf(5)

authentication LOCAL type local {
        users-file "/etc/npppd/npppd-users"
}
#authentication RADIUS type radius {
#	authentication-server {
#		address 192.168.0.1 secret "hogehoge"
#	}
#	accounting-server {
#		address 192.168.0.1 secret "hogehoge"
#	}
#}

tunnel L2TP protocol l2tp {
	listen on 0.0.0.0
	listen on ::
}

ipcp IPCP {
        pool-address 10.0.0.2-10.0.0.254
        dns-servers 8.8.8.8
}

# use pppx(4) interface.  use an interface per a ppp session.
interface pppx0 address 10.0.0.1 ipcp IPCP
bind tunnel from L2TP authenticated by LOCAL to pppx0

# use tun(4) interface.  multiple ppp sessions concentrate one interface.
#interface tun0  address 10.0.0.1 ipcp IPCP
#bind tunnel from L2TP authenticated by LOCAL to tun0
```

```bash title="/etc/npppd/npppd-users"
# $OpenBSD: npppd-users,v 1.1 2012/09/20 12:51:43 yasuoka Exp $
# sample npppd-users file.  see npppd-users(5)

#taro:\
#	:password=taro's password:\
#	:framed-ip-address=10.0.0.101:
#hana:\
#	:password=hana's password:\
#	:framed-ip-address=10.0.0.102:
```

**CARP** is the Common Address Redundancy Protocol. Its primary purpose is to allow multiple hosts on the same network segment to share an IP address. CARP is a secure, free alternative to the [Virtual Router Redundancy Protocol](https://www.ietf.org/rfc/rfc3768.txt) (VRRP) and the [Hot Standby Router Protocol](https://www.ietf.org/rfc/rfc2281.txt) (HSRP).

```bash title="/etc/sysctl.conf"
#	$OpenBSD: sysctl.conf,v 1.4 2015/04/03 15:50:28 millert Exp $
#
# This file contains a list of sysctl options the user wants set at
# boot time.  See sysctl(3) and sysctl(8) for more information on
# the many available variables.
#
net.inet.ip.forwarding=1	# 1=Permit forwarding (routing) of IPv4 packets
net.inet.ip.mforwarding=1	# 1=Permit forwarding (routing) of IPv4 multicast packets
net.inet.ip.multipath=1	# 1=Enable IP multipath routing
net.inet.icmp.rediraccept=1	# 1=Accept ICMP redirects
net.inet6.ip6.forwarding=1	# 1=Permit forwarding (routing) of IPv6 packets
net.inet6.ip6.mforwarding=1	# 1=Permit forwarding (routing) of IPv6 multicast packets
net.inet6.ip6.multipath=1	# 1=Enable IPv6 multipath routing
net.inet.tcp.always_keepalive=1 # 1=Keepalives for all connections (e.g. hotel/airport NAT)
net.inet.tcp.keepidle=100	# 100=send TCP keepalives every 50 seconds
#net.inet.esp.enable=0		# 0=Disable the ESP IPsec protocol
#net.inet.ah.enable=0		# 0=Disable the AH IPsec protocol
#net.inet.esp.udpencap=0	# 0=Disable ESP-in-UDP encapsulation
#net.inet.ipcomp.enable=1	# 1=Enable the IPCOMP protocol
net.inet.etherip.allow=1	# 1=Enable the Ethernet-over-IP protocol
net.inet.tcp.ecn=1		# 1=Enable the TCP ECN extension
net.inet.carp.preempt=1	# 1=Enable carp(4) preemption
net.inet.carp.log=3		# log level of carp(4) info, default 2
#net.pipex.enable=1		# 1=Enable pipex(4) for npppd(8)
ddb.panic=0			# 0=Do not drop into ddb on a kernel panic
#ddb.console=1			# 1=Permit entry of ddb from the console
#ddb.log=1			# 1=Log ddb output in kernel message buffer
#fs.posix.setuid=0		# 0=Traditional BSD chown() semantics
#vm.swapencrypt.enable=0	# 0=Do not encrypt pages that go to swap
#vfs.nfs.iothreads=4		# Number of nfsio kernel threads
#net.inet.ip.mtudisc=0		# 0=Disable tcp mtu discovery
#kern.splassert=2		# 2=Enable with verbose error messages
#kern.nosuidcoredump=3		# 3=Put suid coredumps in /var/crash/progname
kern.watchdog.period=32	# >0=Enable hardware watchdog(4) timer if available
#kern.watchdog.auto=0		# 0=Disable automatic watchdog(4) retriggering
#hw.allowpowerdown=0		# 0=Disable power button shutdown
#machdep.allowaperture=2	# See xf86(4)
#machdep.kbdreset=1		# permit console CTRL-ALT-DEL to do a nice halt
#machdep.lidaction=0		# 1=suspend, 2=hibernate laptop upon lid closing
#machdep.pwraction=1		# ACPI power button action: 0=none, 2=suspend
net.inet.gre.allow=1
net.inet.gre.wccp=1
net.inet.ip.ifq.maxlen=2048
```

На первом 10.112.203.252
```bash
# ifconfig carp
pfsync0: flags=41<UP,RUNNING> mtu 1500
	index 6 priority 0 llprio 3
	pfsync: syncdev: vio1 maxupd: 128 defer: off
	groups: carp pfsync
carp203: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> mtu 1500
	lladdr 00:00:5e:00:01:cb
	description: INT_203
	index 8 priority 15 llprio 3
	carp: MASTER carpdev vio2 vhid 203 advbase 1 advskew 0
	groups: carp
	status: master
	inet 10.112.203.254 netmask 0xffffff00 broadcast 10.112.203.255
```

На втором 10.112.203.251
```bash
obsd1-mail-ru# ifconfig carp
pfsync0: flags=41<UP,RUNNING> mtu 1500
	index 6 priority 0 llprio 3
	pfsync: syncdev: vio0 maxupd: 128 defer: off
	groups: carp pfsync
carp203: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> mtu 1500
	lladdr 00:00:5e:00:01:cb
	description: INT_203
	index 8 priority 15 llprio 3
	carp: BACKUP carpdev vio2 vhid 203 advbase 1 advskew 100
	groups: carp
	status: backup
	inet 10.112.203.254 netmask 0xffffff00 broadcast 10.112.203.255
```

Первый – полностью `ifconfig`

```bash
obsd2-mail-ru# ifconfig          
lo0: flags=8049<UP,LOOPBACK,RUNNING,MULTICAST> mtu 32768
	index 5 priority 0 llprio 3
	groups: lo
	inet6 ::1 prefixlen 128
	inet6 fe80::1%lo0 prefixlen 64 scopeid 0x5
	inet 127.0.0.1 netmask 0xff000000
vio0: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> mtu 1450
	lladdr fa:16:3e:a0:b4:2b
	index 1 priority 0 llprio 3
	groups: egress
	media: Ethernet autoselect
	status: active
	inet 95.163.182.60 netmask 0xfffffe00 broadcast 95.163.183.255
vio1: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> mtu 1450
	lladdr fa:16:3e:04:e6:64
	index 2 priority 0 llprio 3
	media: Ethernet autoselect
	status: active
	inet 192.168.254.2 netmask 0xfffffffc broadcast 192.168.254.3
vio2: flags=8b43<UP,BROADCAST,RUNNING,PROMISC,ALLMULTI,SIMPLEX,MULTICAST> mtu 1450
	lladdr fa:16:3e:4d:77:c5
	index 3 priority 0 llprio 3
	media: Ethernet autoselect
	status: active
	inet 10.112.203.252 netmask 0xffffff00 broadcast 10.112.203.255
enc0: flags=0<>
	index 4 priority 0 llprio 3
	groups: enc
	status: active
pfsync0: flags=41<UP,RUNNING> mtu 1500
	index 6 priority 0 llprio 3
	pfsync: syncdev: vio1 maxupd: 128 defer: off
	groups: carp pfsync
gre8: flags=8051<UP,POINTOPOINT,RUNNING,MULTICAST> mtu 1400
	description: obsd1.quadra.ru
	index 7 priority 0 llprio 6
	encap: vnetid none txprio payload
	groups: gre
	tunnel: inet 95.163.182.60 -> 94.228.246.38 ttl 64 nodf ecn
	inet 172.16.8.2 --> 172.16.8.1 netmask 0xffffffff
carp203: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> mtu 1500
	lladdr 00:00:5e:00:01:cb
	description: INT_203
	index 8 priority 15 llprio 3
	carp: MASTER carpdev vio2 vhid 203 advbase 1 advskew 0
	groups: carp
	status: master
	inet 10.112.203.254 netmask 0xffffff00 broadcast 10.112.203.255
pflog0: flags=141<UP,RUNNING,PROMISC> mtu 33136
	index 9 priority 0 llprio 3
	groups: pflog
```

Второй – полностью `ifconfig`

```bash
# ifconfig
lo0: flags=8049<UP,LOOPBACK,RUNNING,MULTICAST> mtu 32768
	index 5 priority 0 llprio 3
	groups: lo
	inet6 ::1 prefixlen 128
	inet6 fe80::1%lo0 prefixlen 64 scopeid 0x5
	inet 127.0.0.1 netmask 0xff000000
vio0: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> mtu 1500
	lladdr fa:16:3e:40:db:68
	index 1 priority 0 llprio 3
	media: Ethernet autoselect
	status: active
	inet 192.168.254.1 netmask 0xfffffffc broadcast 192.168.254.3
vio1: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> mtu 1450
	lladdr fa:16:3e:24:ca:c8
	index 2 priority 0 llprio 3
	groups: egress
	media: Ethernet autoselect
	status: active
	inet 95.163.182.69 netmask 0xfffffe00 broadcast 95.163.183.255
vio2: flags=8b43<UP,BROADCAST,RUNNING,PROMISC,ALLMULTI,SIMPLEX,MULTICAST> mtu 1450
	lladdr fa:16:3e:29:7e:ac
	index 3 priority 0 llprio 3
	media: Ethernet autoselect
	status: active
	inet 10.112.203.251 netmask 0xffffff00 broadcast 10.112.203.255
enc0: flags=0<>
	index 4 priority 0 llprio 3
	groups: enc
	status: active
pfsync0: flags=41<UP,RUNNING> mtu 1500
	index 6 priority 0 llprio 3
	pfsync: syncdev: vio0 maxupd: 128 defer: off
	groups: carp pfsync
gre7: flags=8051<UP,POINTOPOINT,RUNNING,MULTICAST> mtu 1400
	description: obsd1.quadra.ru
	index 7 priority 0 llprio 6
	encap: vnetid none txprio payload
	groups: gre
	tunnel: inet 95.163.182.69 -> 94.228.246.38 ttl 64 nodf ecn
	inet 172.16.7.2 --> 172.16.7.1 netmask 0xffffffff
carp203: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> mtu 1500
	lladdr 00:00:5e:00:01:cb
	description: INT_203
	index 8 priority 15 llprio 3
	carp: BACKUP carpdev vio2 vhid 203 advbase 1 advskew 100
	groups: carp
	status: backup
	inet 10.112.203.254 netmask 0xffffff00 broadcast 10.112.203.255
pflog0: flags=141<UP,RUNNING,PROMISC> mtu 33136
	index 9 priority 0 llprio 3
	groups: pflog
```

## Командная строка OpenStack CLI в Windows. Облако mail.ru
Для комфортной работы можно изменить ширину окна для Windows CLI и Powershell, например на 300. Щелкнуть правой кнопкой мыши на название окна, Свойства, вкладка Расположение. При следующем открытии ширина сохранится.
В xshell ширину 300 столбцов указываем во вкладке Терминал.
### Простая установка Python в Windows
Скачать последнюю версию WinPyton (Download latest version) https://sourceforge.net/projects/winpython/files/
В итоге у нас на диске файл `Winpython64-3.7.4.0.exe` или подобный (если появится более новая версия). Это не установщик, а просто распаковщик архива `7zip`, т.к. софт полностью Portable и не нуждается в установке.
Перемещаем каталог, который появился после распаковки, в нужное место на диске. У меня после распаковки имя каталога *WPy64-3740*, я его переместила в свой рабочий каталог WORK на диске C:\.
Если хотим переименовать каталог, надо сделать это сразу, т.к. установленные для OpenStack пакеты будут «привязаны» к имени папки. Я не переименовывала.
Заходим в каталог, в моем случае это *WPy64-3740*. Запускаем либо командную строку `Powershell WinPython Powershell Prompt.exe`, либо обычную командную строку `WinPython Command Prompt.exe`.

Ставим дополнительные пакеты для OpenStack:

```powershell
pip install -U buildtools
pip install python-openstackclient
pip install python-neutronclient
```

Пока командную строку можно закрыть.

Хотя у нас Python полностью Portable, т.е. не требует установки и может быть перемещен куда угодно, в дальнейшем возможно захочется не заходить в каталог c python, а запускать python или команды OpenStack вызвав cmd стандартным образом в любом месте. Для этого добавить каталоги, где находится Python, в переменную Path — это некое подобие такой же переменной PATH в Linux, хотя не совсем то. В Windows это некоторые нестандартные каталоги, откуда запускаются программы.

На клавиатуре нажимаем `Win+Pause`, откроется окно *Панель управления→ Все элементы панели управления→ Система*. Вариант 2: на Рабочем столе найти иконку *Компьютер*, нажать на неё правой кнопкой мыши и нажать *Свойства*.
В левом меню нажать *Дополнительные параметры системы*. В окне нажать *«Переменные среды...»*. В нижнем окошке *«Системные переменные»* будут перечислены пары Параметр—Значение. Выделяем строку с переменной Path и жмем «Изменить...».
Здесь через точку с запятой надо добавить два каталога. Т.к. я поместила разархивированный каталог *WPy64-3740* в папку *WORK*, то у меня это будет:

```powershell
C:\WORK\WPy64-3740\python-3.7.4.amd64
C:\WORK\WPy64-3740\python-3.7.4.amd64\Scripts
```

Пример моей итоговой строки (у вас может отличаться), куда я добавила два последних пути, просто скопировав их из проводника.

```powershell
C:\Program Files\VanDyke Software\Clients\;C:\Program Files(x86)\NetSarang\Xftp 6\;C:\Program Files (x86)\NetSarang\Xshell 6\;C:\Program Files (x86)\Common Files\Oracle\Java\javapath;%SystemRoot%\system32;%SystemRoot%;%SystemRoot%\System32\Wbem;%SYSTEMROOT%\System32\WindowsPowerShell\v1.0\;C:\WORK\WPy64-3740\python-3.7.4.amd64;C:\WORK\WPy64-3740\python-3.7.4.amd64\Scripts
```

Сохраняем всё, нажав Ок.

Как проверить: в Windows CLI  все текущие переменные, в том числе и Path, отображаются командой `set` без аргументов (аналог `env` в Linux):

```powershell
C:\Users\Pupkin_AV>set
ALLUSERSPROFILE=C:\ProgramData
APPDATA=C:\Users\Pupkin_AV\AppData\Roaming
CommonProgramFiles=C:\Program Files\Common Files
CommonProgramFiles(x86)=C:\Program Files (x86)\Common Files
CommonProgramW6432=C:\Program Files\Common Files
COMPUTERNAME=PUPKIN-AV
ComSpec=C:\windows\system32\cmd.exe
FP_NO_HOST_CHECK=NO
HOMEDRIVE=C:
HOMEPATH=\Users\Pupkin_AV
LOCALAPPDATA=C:\Users\Pupkin_AV\AppData\Local
...
NUMBER_OF_PROCESSORS=4
OS=Windows_NT
Path=C:\Program Files\VanDyke Software\Clients\;C:\Program Files (x86)\NetSarang\Xftp 6\;C:\Program Files (x86)\NetSarang\Xshell 6\;C:\Program Files (x86)\Common Files\Oracle\Java\javapath;C:\windows
system32;C:\windows;C:\windows\System32\Wbem;C:\windows\System32\WindowsPowerShell\v1.0\;C:\windows\System32\WindowsPowerShell\v1.0\;C:\windows\System32\WindowsPowerShell\v1.0\;C:\WORK\WPy64-3740\pyt
on-3.7.4.amd64;C:\WORK\WPy64-3740\python-3.7.4.amd64\Scripts;C:\Program Files\Bandizip\
PATHEXT=.COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC
PROCESSOR_ARCHITECTURE=AMD64
PROCESSOR_IDENTIFIER=Intel64 Family 6 Model 42 Stepping 7, GenuineIntel
PROCESSOR_LEVEL=6
PROCESSOR_REVISION=2a07
ProgramData=C:\ProgramData
ProgramFiles=C:\Program Files
ProgramFiles(x86)=C:\Program Files (x86)
ProgramW6432=C:\Program Files
PROMPT=$P$G
...
```

Теперь скопировать свои данные для подключения к Mail.Ru. В веб-интерфейсе mail.ru (INFRA) в меню слева выбираем *Вычислительные ресурсы→ Доступ к API*. Это ссылка https://infra.mail.ru/dashboard/project/api_access/
Жмем *Скачать OpenStack RC-файл версии 3*.
Скачанный скрипт c расширением `.sh` позволит импортировать переменные окружения в Linux. Для Windows надо сделать другой файл, но с теми же значениями переменных. Причем значения будем указывать без кавычек.

!!! warning
    Для обычной командной строки Windows и для Powershell переменные импортируются по-разному!

### Подключение к mail.ru из Windows CLI
Открыть командную строку Windows CLI, варианты:
* `Win+R`, ввести `cmd`, Enter
* Пуск, в окне ввести `cmd`, Enter
* В любой папке `Shift`+щелчок правой кнопкой мыши, в меню выбрать «Открыть окно команд». Преимущество этого способа в том, что командная строка открывается сразу с нужной папке, а не в каталоге по умолчанию.

У Windows CLI нет своих собственных переменных. Есть только переменные окружения. Создаем файл `mailru.bat`:
```powershell title="mailru.bat"
@echo off
set OS_AUTH_URL=https://infra.mail.ru:35357/v3/
set OS_PROJECT_ID=МногоБуквЦифр
set OS_PROJECT_NAME=pupkin_av@example.ru
set OS_USER_DOMAIN_NAME=users
set OS_PROJECT_DOMAIN_ID=МногоБуквЦифр
set OS_USERNAME=pupkin_av@example.ru
set OS_REGION_NAME=RegionOne
set OS_IDENTITY_API_VERSION=3
set OS_INTERFACE=public
set OS_PASSWORD=Тут_пароль
```

Первая строка `@echo off` : все, что будет выполнено ниже, не будет отображаться на экране. Переменные задаются без кавычек и в формате:

```powershell
 set ПЕРЕМЕННАЯ=значение
```

Чтобы этот файл исполнялся одной командой `mailru`, закинем его в каталог `C:\WORK\WPy64-3740\python-3.7.4.amd64\Scripts`, который у нас уже указан в Path. Введенные из файла значения переменных окружения действуют только на время текущей сессии. Как только окно работы с командной строкой закрыли, они недействительны.

!!! warning
    Чтобы никто не украл пароли, надо либо шифровать каталог *Scripts*, либо сделать отдельный безопасный каталог для таких секретных данных и добавить его в Path. Для безопасного шифрования каталогов можно использовать кроссплатформенный софт [Pismo File Mount Audit Package](https://pismotec.com/download/), позволяющий в одном шифрованном файле держать свои секретные каталоги. При необходимости использовать каталоги файл монтируется с вводом пароля.

После того как поместили *mailru.bat* в каталог *Scripts* (или другой шифрованный), процедура работы с Mail.Ru такова: открываем Windows CLI любым способом, вводим `mailru`, чтобы прописать все переменные окружения для подключения к mail.ru, после чего можно вводить команды OpenStack, например:

```powershell
Microsoft Windows [Version 6.1.7601]
(c) Корпорация Майкрософт (Microsoft Corp.), 2009. Все права защищены.

C:\Users\Pupkin_AV>mailru

C:\Users\Pupkin_AV>openstack server list
+--------------------------------------+--------------+---------+----------------------------------------------+-------+-------------------+
| ID                                   | Name         | Status  | Networks                                     | Image | Flavor            |
+--------------------------------------+--------------+---------+----------------------------------------------+-------+-------------------+
| cd4ae102-9c93-4b8d-a574-788ea41966d9 | TEST         | SHUTOFF | internal=10.112.203.20                       |       | Standard-2-4-50   |
| 67ac5f50-6071-4fd8-bf79-fde99c89e656 | centos_01    | ACTIVE  | ext-net=95.163.182.60; internal=10.112.203.4 |       | Basic-1-1-10      |
| 938de5a2-4811-4605-8944-ab67ad40988e | 1c-licsrv    | ACTIVE  | internal=10.112.203.2                        |       | Standard-2-4-40   |
| 6c3b615e-2cb3-41b0-ac09-39751c336759 | 1c-pgsqltest | ACTIVE  | internal=10.112.203.7                        |       | Advanced-16-32-50 |
| 6143a993-4399-47cb-a445-b5fb488921fd | 1c-apptest   | ACTIVE  | internal=10.112.203.11                       |       | Advanced-8-16-100 |
| 9c03abb6-4511-46f4-9130-d3d3b5847b97 | storage-test | ACTIVE  | internal=10.112.203.12                       |       | Standard-4-16-50  |
| f46b0cea-b6f1-4583-b7a8-e22500456a0c | openbsd2     | ACTIVE  | internal=10.112.203.198                      |       | Standard-2-4-50   |
| 74d0d220-ca1d-4edb-bb5a-b2be10cd2d86 | openbsd      | ACTIVE  | ext-net=95.163.182.69                        |       | Basic-1-4-50      |
+--------------------------------------+--------------+---------+----------------------------------------------+-------+-------------------+
```

### Подключение к mail.ru из PowerShell
Открыть командную строку PowerShell:
* `Win+R`, ввести `powershell`, Enter
* Пуск, в окне ввести `powershell`, Enter. Преимущество в том, что можно начать вводить слово `pow`, и вверху появятся возможные варианты.  Также можно запустить PowerShell от имени администратора, этот режим выбирается правой кнопкой мыши при наведении на программу.
* Можно сначала зайти в Windows CLI, а там ввести `powershell`, и мы в том же окне перейдем в PowerShell. Или ввести `start powershell`, чтобы открылось отдельное окно PowerShell.
* В Windows 10 есть скрытое меню WinX для продвинутых пользователей. Самый быстрый способ открыть это меню — `Win+X` на клавиатуре или щелкнуть правой кнопкой мыши на логотипе Windows в левом нижнем углу или щелкнуть левой кнопкой и удерживать. Из меню можно открыть PowerShell от своего имени либо от администратора.

Здесь все существующие переменные можно просмотреть командой `Get-ChildItem env:` или сокращенной командой `gci env:` (аналог команды `set` в Windows CLI или `env` в Linux):

```powershell
PS C:\Users\Pupkin_AV> Get-ChildItem env:

Name                           Value
----                           -----
ALLUSERSPROFILE                C:\ProgramData
APPDATA                        C:\Users\Pupkin_AV\AppData\Roaming
CommonProgramFiles             C:\Program Files\Common Files
CommonProgramFiles(x86)        C:\Program Files (x86)\Common Files
CommonProgramW6432             C:\Program Files\Common Files
COMPUTERNAME                   PUPKIN-AV
ComSpec                        C:\windows\system32\cmd.exe
FP_NO_HOST_CHECK               NO
HOMEDRIVE                      C:
HOMEPATH                       \Users\Pupkin_AV
LOCALAPPDATA                   C:\Users\Pupkin_AV\AppData\Local
...
NUMBER_OF_PROCESSORS           4
OS                             Windows_NT
Path                           %SystemRoot%\system32\WindowsPowerShell\v1.0\;C:\Program Files\VanDyke Software\Clients\;C:\Program Files (x86)\NetSarang\Xftp 6\;C:\Program Files (x86)\NetSarang\Xs...
PATHEXT                        .COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC;.CPL
PROCESSOR_ARCHITECTURE         AMD64
PROCESSOR_IDENTIFIER           Intel64 Family 6 Model 42 Stepping 7, GenuineIntel
PROCESSOR_LEVEL                6
PROCESSOR_REVISION             2a07
ProgramData                    C:\ProgramData
ProgramFiles                   C:\Program Files
ProgramFiles(x86)              C:\Program Files (x86)
ProgramW6432                   C:\Program Files
...
windows_tracing_logfile        C:\BVTBin\Tests\installpackage\csilogfile.log
```

Посмотреть значение конкретной переменной, например Path:

```powershell
PS C:\Users\Pupkin_AV> Get-Item Env:Path

Name        Value
----        -----
Path        %SystemRoot%\system32\WindowsPowerShell\v1.0\;C:\Program Files\VanDyke Software\Clients\;C:\Program Files (x86)\NetSarang\Xftp 6\;C:\Program Files (x86)\NetSarang\Xs...
```

Формат установки переменной окружения для PowerShell. Кавычки не нужны.

```powershell
set-env env:ПЕРЕМЕННАЯ -value значение
```

Можно упростить:

```powershell
set-env env:ПЕРЕМЕННАЯ значение
```

Также возможна другая форма записи, но здесь обязательно значение переменной заключить в **двойные или одинарные кавычки**. Только если значение цифровое, можно обойтись без кавычек.

```powershell
$env:ПЕРЕМЕННАЯ="значение"
```

Здесь не рассматриваю варианты создания JSON, XML файлов или сложные способы.

Импорт переменных окружения из файла может вызвать затруднения, т.к. исполнение скриптов PowerShell может быть запрещено.
Итак, создадим файл `mailru.ps1` и скопируем его в тот же каталог `C:\WORK\WPy64-3740\python-3.7.4.amd64\Scripts`, который добавили ранее в переменную окружения *Path*, значение которой будет сохраняться даже при перезагрузке компьютера.

Содержимое `mailru.ps1`, вариант 1 (кавычки можно пропустить там где значение переменной цифровое):

```powershell title="mailru.ps1"
$env:OS_AUTH_URL="https://infra.mail.ru:35357/v3/"
$env:OS_PROJECT_ID="МногоБуквЦифр"
$env:OS_PROJECT_NAME="pupkin_av@example.ru"
$env:OS_USER_DOMAIN_NAME="users"
$env:OS_PROJECT_DOMAIN_ID="МногоБуквЦифр"
$env:OS_USERNAME="pupkin_av@example.ru"
$env:OS_REGION_NAME="RegionOne"
$env:OS_IDENTITY_API_VERSION=3
$env:OS_INTERFACE="public"
$env:OS_PASSWORD="ТутПароль"
```

Содержимое `mailru.ps1`, вариант 2:

```powershell title="mailru.ps1"
set-item env:OS_AUTH_URL https://infra.mail.ru:35357/v3/
set-item env:OS_PROJECT_ID МногоБуквЦифр
set-item env:OS_PROJECT_NAME pupkin_av@example.ru
set-item env:OS_USER_DOMAIN_NAME users
set-item env:OS_PROJECT_DOMAIN_ID МногоБуквЦифр
set-item env:OS_USERNAME pupkin_av@example.ru
set-item env:OS_REGION_NAME RegionOne
set-item env:OS_IDENTITY_API_VERSION 3
set-item env:OS_INTERFACE public
set-item env:OS_PASSWORD ТутПароль
```

!!! info
    Когда мы находимся в Windows CLI и выполняем `mailru`, будет автоматом выполняться `mailru.bat`, а когда мы находимся в Powershell и вводим `mailru`, будет автоматом выполняться `mailru.ps1`. Таким образом, способ подключения к Mail.Ru для нас совершенно одинаков в этих двух вариантах, хотя внутри двух файлов одни и те же переменные окружения задаются разным синтаксисом.

Пробуем выполнить `mailru` и получаем сообщение о запрете:

```powershell
PS C:\Users\Pupkin_AV> mailru
mailru : File C:\WORK\WPy64-3740\python-3.7.4.amd64\Scripts\mailru.ps1 cannot be loaded because running scripts is disabled on this system. For more information, see about_Execution_Policies at http://go.microsoft.com/fwlink/?LinkID=135170.
At line:1 char:1
+ mailru
+ ~~~~~~
    + CategoryInfo          : SecurityError: (:) [], PSSecurityException
    + FullyQualifiedErrorId : UnauthorizedAccess
```

Проверим текущую политику. Действительно она запрещающая:

```powershell
PS C:\Users\Pupkin_AV> Get-ExecutionPolicy
Restricted
```

Самый простой способ — сменить политику исполнения скриптов на **Bypass** или **RemoteSigned**. Смена политики действует не только на время текущей сессии, т.е. это можно сделать один раз.

```powershell
PS C:\Users\Pupkin_AV> Set-ExecutionPolicy Bypass

Execution Policy Change
The execution policy helps protect you from scripts that you do not trust. Changing the execution policy might expose you to the security risks described in the about_Execution_Policies help topic at http://go.microsoft.com/fwlink/?LinkID=135170. Do you want to change the execution policy?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"): Y
```

Теперь можем спокойно выполнять `mailru`:

```
PS C:\Users\Pupkin_AV> mailru
PS C:\Users\Pupkin_AV> openstack server list
+--------------------------------------+--------------+---------+----------------------------------------------+-------+-------------------+
| ID                                   | Name         | Status  | Networks                                     | Image | Flavor            |
+--------------------------------------+--------------+---------+----------------------------------------------+-------+-------------------+
| cd4ae102-9c93-4b8d-a574-788ea41966d9 | TEST         | SHUTOFF | internal=10.112.203.20                       |       | Standard-2-4-50   |
| 67ac5f50-6071-4fd8-bf79-fde99c89e656 | centos_01    | ACTIVE  | ext-net=95.163.182.60; internal=10.112.203.4 |       | Basic-1-1-10      |
| 938de5a2-4811-4605-8944-ab67ad40988e | 1c-licsrv    | ACTIVE  | internal=10.112.203.2                        |       | Standard-2-4-40   |
| 6c3b615e-2cb3-41b0-ac09-39751c336759 | 1c-pgsqltest | ACTIVE  | internal=10.112.203.7                        |       | Advanced-16-32-50 |
| 6143a993-4399-47cb-a445-b5fb488921fd | 1c-apptest   | ACTIVE  | internal=10.112.203.11                       |       | Advanced-8-16-100 |
| 9c03abb6-4511-46f4-9130-d3d3b5847b97 | storage-test | ACTIVE  | internal=10.112.203.12                       |       | Standard-4-16-50  |
| f46b0cea-b6f1-4583-b7a8-e22500456a0c | openbsd2     | ACTIVE  | internal=10.112.203.198                      |       | Standard-2-4-50   |
| 74d0d220-ca1d-4edb-bb5a-b2be10cd2d86 | openbsd      | ACTIVE  | ext-net=95.163.182.69                        |       | Basic-1-4-50      |
+--------------------------------------+--------------+---------+----------------------------------------------+-------+-------------------+
```

Пример команды загрузки образа:
```bash
openstack image create --private --container-format bare --disk-format raw --file имя_файла.raw --property hw_qemu_guest_agent=yes --property os_require_quiesce=yes имя_образа
```

### Настройка xshell для работы с Windows CLI и Openstack
В Xshell удобно журналировать сессии, а также подключаться к удаленным серверам. В случае с Windows CLI мы никуда не подключаемся, поэтому протокол LOCAL и никаких паролей.

![Соединение](xshell-Connection.png)
При входе можно автоматически ввести `cmd`, чтобы попасть в командную строку Windows. Кстати если мы выполняем `ping` и затем для остановки нажали `Ctrl+C`, надо будет уже руками набрать `cmd`, чтобы вернуться в командную строку. Включаем журналирование сессий. Путь к файлу можно отредактировать, но стандартно он состоит из имени сессии `%n`, далее год, месяц, день и время. Если надо поменять каталог для логов (например логи сессий можем так же распределить по папкам, как и слева дерево сессий), нажать на троеточие и выбрать путь.
![Соединение→ Скрипты входа](xshell-ScriptsEnter.png)
![Расширенные→ Журнал сессии](xshell-SessionHistory.png)
![Терминал](xshell-Terminal.png)

Как видим ниже, при подключении команда cli уже ввелась. Далее набираем mailru, как из Windows CLI (текущий вариант у нас это Windows CLI) или Powershell и можем вводить команды OpenStack.

![Управление сессиями](xshell-Sessions.png)
Что пока не работает: регулярные выражения как в оболочке Linux bash. Для исправления этой ситуации скачать программу **cmder** и использовать вместо Xshell.
## Команды OpenStack
Формат команды и примеры для установки фиксированного ip-адреса, если его не было:
```bash
openstack port set --no-fixed-ip --fixed-ip subnet=<subnet>,ip-address=<ip-address> <port-id>
openstack port set --no-fixed-ip --fixed-ip subnet=int_203,ip-address=10.112.203.251 2038c49f-200c-4111-93f1-68c2b62ee718
openstack port set --no-fixed-ip --fixed-ip subnet=pfcync,ip-address=192.168.254.1 2f6a51b9-4606-4c11-a66d-a967b145d592
openstack port set --no-fixed-ip --fixed-ip subnet=int_203,ip-address=10.112.203.252 OBSD2_int203
```

Установка группы безопасности (типа правил файрвола) для порта работает только из командной строки. Т.е. группы можно задать при создании инстанса, а потом только посмотреть, какие назначены, но не изменить.

```bash
openstack port set --security-group gre 4313ed09-743d-4661-be64-86997b24d70c
```

По умолчанию имена портам не задаются, поэтому в дальнейшем для наглядности можно их задать самостоятельно.

```bash
openstack port set --name ROUTER_aux 7e3dd13a-d5dd-416b-891e-fdda66bd4498
```

Как косвенно узнать версию OpenStack. Здесь версия 13 Queens. Или в Horizon Admin→ System Information.

```powershell
C:\Distr\xshell\Xshell\Sessions>openstack keystone-manage --version
openstack keystone-manage --version
openstack 3.19.0
```

Список серверов (инстансов):

```bash
C:\Distr\xshell\Xshell\Sessions>openstack server list
openstack server list
+--------------------------------------+--------------+---------+----------------------------------------------------------------------+-------+-------
| ID                                   | Name         | Status  | Networks                                                             | Image | Flavor            |
+--------------------------------------+--------------+---------+----------------------------------------------------------------------+-------+-------------------+
| df6f17d1-8066-4a5c-a9a5-9dc0b297bd42 | anna-test    | ACTIVE  | internal=10.112.203.18                                               |       | Standard-2-4-40   |
| 1ddd12a1-badd-4a82-95d8-2573ad3d7286 | centos-test  | ACTIVE  | internal=10.112.203.24                                               |       | Basic-1-2-40      |
| cd4ae102-9c93-4b8d-a574-788ea41966d9 | TEST         | SHUTOFF | internal=10.112.203.20                                               |       | Standard-2-4-50   |
| 67ac5f50-6071-4fd8-bf79-fde99c89e656 | centos_01    | ACTIVE  | ext-net=95.163.182.60; internal=10.112.203.4                         |       | Basic-1-1-10      |
| 938de5a2-4811-4605-8944-ab67ad40988e | 1c-licsrv    | ACTIVE  | internal=10.112.203.2                                                |       | Standard-2-4-40   |
| 6c3b615e-2cb3-41b0-ac09-39751c336759 | 1c-pgsqltest | ACTIVE  | internal=10.112.203.7                                                |       | Advanced-16-32-50 |
| 6143a993-4399-47cb-a445-b5fb488921fd | 1c-apptest   | ACTIVE  | internal=10.112.203.11                                               |       | Advanced-8-16-100 |
| 9c03abb6-4511-46f4-9130-d3d3b5847b97 | storage-test | ACTIVE  | internal=10.112.203.12                                               |       | Standard-4-16-50  |
| f46b0cea-b6f1-4583-b7a8-e22500456a0c | openbsd2     | ACTIVE  | internal=10.112.203.252                                              |       | Standard-2-4-50   |
| 74d0d220-ca1d-4edb-bb5a-b2be10cd2d86 | openbsd      | ACTIVE  | ext-net=95.163.182.69; internal=10.112.203.251; pfsync=192.168.254.1 |       | Basic-1-4-50      |
+--------------------------------------+--------------+---------+----------------------------------------------------------------------+-------+------

```

Список портов (сетевых интерфейсов). Здесь я уже задала имена портам для удобства (для тестовых инстансов не задавала, поэтому Name пустой), но изначально список не очень удобен, т.к. непонятно, какому инстансу какие порты принадлежат.

```bash
C:\Distr\xshell\Xshell\Sessions>openstack port list
openstack port list
+--------------------------------------+--------------------+-------------------+----------------------------------------------------------------------------+--------+
| ID                                                      | Name                   | MAC Address           | Fixed IP Addresses                                                                                             | Status |
+--------------------------------------+--------------------+-------------------+----------------------------------------------------------------------------+--------+
| 02c52cbb-6b28-4c6c-8db0-d077eb8fa813 |                          | fa:16:3e:5f:50:0b | ip_address='10.112.203.18', subnet_id='dd63f499-a14c-4437-af4a-7cd590de1cd4'    | ACTIVE |
| 0712c5c4-057a-40ad-9718-2e9878104078 |                          | fa:16:3e:da:8a:a2 | ip_address='10.112.203.20', subnet_id='dd63f499-a14c-4437-af4a-7cd590de1cd4'    | ACTIVE |
| 2038c49f-200c-4111-93f1-68c2b62ee718 | OBSD1_int203     | fa:16:3e:29:7e:ac | ip_address='10.112.203.251', subnet_id='dd63f499-a14c-4437-af4a-7cd590de1cd4'  | ACTIVE |
...
и т.д.
```

Список портов, принадлежащих конкретному серверу:

```bash
C:\Distr\xshell\Xshell\Sessions>openstack port list --server openbsd
openstack port list --server openbsd
+--------------------------------------+--------------+-------------------+-------------------------------------------------------------------------------+--------+
| ID                                                       | Name               | MAC Address           | Fixed IP Addresses                                                          | Status |
+--------------------------------------+--------------+-------------------+-------------------------------------------------------------------------------+--------+
| 2038c49f-200c-4111-93f1-68c2b62ee718 | OBSD1_int203 | fa:16:3e:29:7e:ac | ip_address='10.112.203.251', subnet_id='dd63f499-a14c-4437-af4a-7cd590de1cd4' | ACTIVE |
| 2f6a51b9-4606-4c11-a66d-a967b145d592 | OBSD1_pfsync | fa:16:3e:40:db:68 | ip_address='192.168.254.1', subnet_id='d8174518-8545-416e-8fef-148681c51d29'    | ACTIVE |
| c76ad8de-5fad-49d9-8b31-23beb7977661 | OBSD1_extif    | fa:16:3e:24:ca:c8 | ip_address='95.163.182.69', subnet_id='ec5d4a62-5039-460c-833f-7084a19794d2'    | ACTIVE |
+--------------------------------------+--------------+-------------------+-------------------------------------------------------------------------------+--------+
```

Список всех портов с распределением по серверам не получается отобразить стандартной командой, поэтому только из API, где работает grep и пр., но в ответе будут uuid инстанcов, а не имена:

```bash
TOKEN=$(openstack token issue | grep ' id '| awk '{print $4}')
curl -s -g -X GET "https://infra.mail.ru:9696/v2.0/ports?project_id=3a29058e451244e1be88d8a6b8012da5&fields=id&fields=device_id&fields=fixed_ips&fields=network_id&fields=status&fields=mac_address&fields=device_owner"
 -H "X-Auth-Token: $TOKEN" -H "Content-Type: application/json" -H 
"Accept: application/json" -H "User-Agent: None"| python -m json.tool
```

Задать имя для порта (сетевого интерфейса):

```bash
openstack port set --name confluence_int d20d4e1c-db61-4c0a-aeb1-be436f793731
```

Список volumes:

```bash
C:\Distr\xshell\Xshell\Sessions>openstack volume list
openstack volume list
+--------------------------------------+-------------+-----------+------+---------------------------------------+
| ID                                   | Name        | Status    | Size | Attached to                           |
+--------------------------------------+-------------+-----------+------+---------------------------------------+
| cfb2feca-fe6d-4b03-ade2-9d1b4b81d62e |             | in-use    |   40 | Attached to anna-test on /dev/vda     |
| 0685f042-01b1-46ef-a448-592c530a366d |             | in-use    |   10 | Attached to centos-test on /dev/vda   |
| 09c33ac5-a502-44fb-bb9a-145b1d016c8a |             | in-use    |   35 | Attached to TEST on /dev/vda          |
| ef77e1ba-0190-498e-9fea-6895e4416853 |             | in-use    |   10 | Attached to centos_01 on /dev/vda     |
| 00fbcb96-39da-4e8b-a328-24b430726b93 |             | in-use    |   40 | Attached to 1c-licsrv on /dev/vda     |
| d3bd0777-40e5-47ff-a5d8-6c22c98ad8f1 | test-volume | available |   32 |                                       |
| 31f67a9f-f8f2-4225-86d7-7f7195331079 | None        | in-use    |  800 | Attached to 1c-pgsqltest on /dev/vda  |
| 9b936472-4c1d-4881-9da6-2ff73dad32bd | None        | in-use    |  100 | Attached to 1c-apptest on /dev/vda    |
| 8c561084-687e-4dc5-98fc-df782d06ea7e | None        | in-use    | 2100 | Attached to storage-test on /dev/vda  |
| ae0c8189-6090-4798-a2d1-fc336e1e1113 |             | in-use    |   18 | Attached to openbsd2 on /dev/vda      |
| 65ed56d7-8ce4-43dc-860f-aa08d0acdd92 |             | in-use    |   18 | Attached to openbsd on /dev/vda       |
```

Список образов, из которых устанавливается операционная система. Но отобразит не только закаченные нами образы, а все, включая публичные:

```bash
C:\Distr\xshell\Xshell\Sessions>openstack image list --private
openstack image list --private
+--------------------------------------+-------------------+--------+
| ID                                   | Name              | Status |
+--------------------------------------+-------------------+--------+
| f8b380b3-ba28-4f20-a533-de1e25893bad | OpenBSD-amd64-6.5 | active |
| 009884e2-793d-455e-ad6e-d559ea6bc5e2 | centos-7.6-1907   | active |
| 107c1276-d4a1-47d0-b987-8c070400258a | target_dc2        | active |
| fe9a9f4e-9532-465e-9f34-3e2e714c030c | vids_temp         | active |
+--------------------------------------+-------------------+--------+
```

Закачать образ с локального компьютера в на Mail.Ru. Если мы не указываем дополнительные параметры, то visibility будет *shared*. Просто можно добавить флаг `--private`. Здесь файл образа `truedeb.raw` лежит в каталоге `C:/Distr`. Назвали образ так же как и файл — `truedeb`, но это необязательно.

```bash
C:\Distr\xshell\Xshell\Sessions>openstack image create truedeb --file C:\Distr\truedeb.raw
openstack image create truedeb --file C:\Distr\truedeb.raw
+------------------+-----------------------------------------------------------------+
| Field            | Value                                                           |
+------------------+-----------------------------------------------------------------+
| checksum         | 2b3ab7019a6102eeddfc414ceb0fdb14                                |
| container_format | bare                                                            |
| created_at       | 2019-10-02T10:12:49Z                                            |
| disk_format      | raw                                                             |
| file             | /v2/images/33c504c6-ce67-4206-b7fb-641a2d3fc2ba/file            |
| id               | 33c504c6-ce67-4206-b7fb-641a2d3fc2ba                            |
| min_disk         | 0                                                               |
| min_ram          | 0                                                               |
| name             | truedeb                                                         |
| owner            | 3a29058e451244e1be88d8a6b8012da5                                |
| properties       | direct_url='rbd://b0e441fc-c317-4acf-a606-cf74683978d2/images/33c504c6-ce67-4206-b7fb-641a2d3fc2ba/snap', locations='[{'url': 'rbd://b0e441fc-c317-4acf-a606-cf74683978d2/images/33c504c6-ce67-4206-b7fb-641a2d3fc2ba/snap', 'metadata': {}}]' |
| protected        | False                                                           |
| schema           | /v2/schemas/image                                               |
| size             | 21474836480                                                     |
| status           | active                                                          |
| tags             |                                                                 |
| updated_at       | 2019-10-02T10:44:46Z                                            |
| virtual_size     | None                                                            |
| visibility       | shared                                                          |
+------------------+-----------------------------------------------------------------+
```

Сложная команда:

```bash
openstack image create --os-auth-url https://infra.mail.ru:35357/v3/ --os-identity-api-version 3 --private --disk-format raw --container-format bare --file <полный_путь_к_файлу> имя_образа.
```

### Закачиваем новый образ.
Закачала самую свежую версию CentOS7 на mail.ru. Назначила:

```bash
openstack image create --container-format bare --public --file ./CentOS-7-x86_64-GenericCloud-1907.raw centos-7.6-1907
openstack image set --property short-id=centos centos-7.6-1907
```

### Переназначение внешнего адреса
Был например порт, который был заранее создан и потом назначен в качестве внешнего ip для инстанса `centos_01`. После удаления инстанса порт с ip-адресом `95.163.182.60` DOWN, но при этом из веб-интерфейса не получается его назначить на другую виртуальную машину, например в качестве внешнего интерфейса OpenBSD (будет интерфейсом `OBSD2_ext`).

```bash
C:\Distr\xshell\Xshell\Sessions>openstack port list
openstack port list
+--------------------------------------+--------------------+-------------------+-------------------------------------------------------------------------------+--------+
| ID                                   | Name               | MAC Address       | Fixed IP Addresses                                                            | Status |
+--------------------------------------+--------------------+-------------------+-------------------------------------------------------------------------------+--------+
| 2038c49f-200c-4111-93f1-68c2b62ee718 | OBSD1_int          | fa:16:3e:29:7e:ac | ip_address='10.112.203.251', subnet_id='dd63f499-a14c-4437-af4a-7cd590de1cd4' | ACTIVE |
| 2e8384b1-8cf3-4f98-8255-b9858694a6de | ROUTER_main        | fa:16:3e:ae:96:da | ip_address='10.112.203.250', subnet_id='dd63f499-a14c-4437-af4a-7cd590de1cd4' | ACTIVE |
| 2f6a51b9-4606-4c11-a66d-a967b145d592 | OBSD1_pfsync       | fa:16:3e:40:db:68 | ip_address='192.168.254.1', subnet_id='d8174518-8545-416e-8fef-148681c51d29'  | ACTIVE |
| 4c610117-4f5d-4308-9612-a198fc3ecfa4 | 1c-licsrv_int      | fa:16:3e:4e:fa:80 | ip_address='10.112.203.2', subnet_id='dd63f499-a14c-4437-af4a-7cd590de1cd4'   | ACTIVE |
| 529ae566-48ba-40d0-a0a5-00664dcf82e6 | OBSD2_int203       | fa:16:3e:4d:77:c5 | ip_address='10.112.203.252', subnet_id='dd63f499-a14c-4437-af4a-7cd590de1cd4' | ACTIVE |
| 7e3dd13a-d5dd-416b-891e-fdda66bd4498 | ROUTER_aux         | fa:16:3e:07:55:7b | ip_address='10.112.203.19', subnet_id='dd63f499-a14c-4437-af4a-7cd590de1cd4'  | ACTIVE |
| 80b2eaa3-6e14-4255-8366-54291fb7e82f | DHCP_2             | fa:16:3e:41:00:59 | ip_address='10.112.203.16', subnet_id='dd63f499-a14c-4437-af4a-7cd590de1cd4'  | ACTIVE |
| 8b16b953-37df-430f-9b14-53c0bd8c23e0 | DHCP_1             | fa:16:3e:bd:03:e0 | ip_address='10.112.203.15', subnet_id='dd63f499-a14c-4437-af4a-7cd590de1cd4'  | ACTIVE |
| a192002a-f3c6-4635-b10e-281d4dc57e21 | 1c-apptest_int     | fa:16:3e:f7:f1:4f | ip_address='10.112.203.11', subnet_id='dd63f499-a14c-4437-af4a-7cd590de1cd4'  | ACTIVE |
| add9c7be-b9c5-430d-98ae-75db9f99697e | 1c-storagetest_int | fa:16:3e:5b:df:d9 | ip_address='10.112.203.12', subnet_id='dd63f499-a14c-4437-af4a-7cd590de1cd4'  | ACTIVE |
| c76ad8de-5fad-49d9-8b31-23beb7977661 | OBSD1_ext          | fa:16:3e:24:ca:c8 | ip_address='95.163.182.69', subnet_id='ec5d4a62-5039-460c-833f-7084a19794d2'  | ACTIVE |
| d20d4e1c-db61-4c0a-aeb1-be436f793731 | confluence_int     | fa:16:3e:6a:41:f0 | ip_address='10.112.203.24', subnet_id='dd63f499-a14c-4437-af4a-7cd590de1cd4'  | ACTIVE |
| d4dbe38e-b4e1-462c-8c79-0059014648ce | 1c-pgsqltest_int   | fa:16:3e:77:6c:00 | ip_address='10.112.203.7', subnet_id='dd63f499-a14c-4437-af4a-7cd590de1cd4'   | ACTIVE |
| da3fa0af-6764-494c-8adb-979639eb3a22 | 95.163.182.60      | fa:16:3e:52:4f:1c | ip_address='95.163.182.60', subnet_id='ec5d4a62-5039-460c-833f-7084a19794d2'  | DOWN   |
| e65d075e-13f9-423e-8696-6f13971c4692 | OBSD2_pfsync       | fa:16:3e:04:e6:64 |                                                                               | ACTIVE |
| ffe388fa-7836-47a0-9908-0d32676e16c2 |                    | fa:16:3e:bc:2c:0d | ip_address='10.112.203.25', subnet_id='dd63f499-a14c-4437-af4a-7cd590de1cd4'  | ACTIVE |
+--------------------------------------+--------------------+-------------------+-------------------------------------------------------------------------------+--------+
```

Переименуем интерфейс:

```bash
openstack port set --name OBSD2_ext da3fa0af-6764-494c-8adb-979639eb3a22
```

Почему-то присвоить порт серверу не удалось у меня, а на 4 версии получилось:

```bash
openstack server add port f46b0cea-b6f1-4583-b7a8-e22500456a0c da3fa0af-6764-494c-8adb-979639eb3a22

interface_attach() missing 1 required positional argument: 'server'
```

История:

```bash
(openstack) server list
+--------------------------------------+--------------+--------+-------------------------------------+-------+-------------------+
| ID                                   | Name         | Status | Networks                            | Image | Flavor            |
+--------------------------------------+--------------+--------+-------------------------------------+-------+-------------------+
| 938de5a2-4811-4605-8944-ab67ad40988e | 1c-licsrv    | ACTIVE | internal=10.112.203.2, 10.112.203.7 |       | Standard-2-4-40   |
| 6c3b615e-2cb3-41b0-ac09-39751c336759 | 1c-pgsqltest | ACTIVE |                                     |       | Advanced-16-32-50 |
| 6143a993-4399-47cb-a445-b5fb488921fd | 1c-apptest   | ACTIVE | internal=10.112.203.11              |       | Advanced-8-16-100 |
| 9c03abb6-4511-46f4-9130-d3d3b5847b97 | storage-test | ACTIVE |                                     |       | Standard-4-16-50  |
| f46b0cea-b6f1-4583-b7a8-e22500456a0c | openbsd2     | ACTIVE | ext-net=95.163.182.60               |       | Standard-2-4-50   |
| 74d0d220-ca1d-4edb-bb5a-b2be10cd2d86 | openbsd      | ACTIVE | ext-net=95.163.182.69               |       | Basic-1-4-50      |
+--------------------------------------+--------------+--------+-------------------------------------+-------+-------------------+

+--------------------------------------+-----------------------+-------------------+------------------------------------------------------------------------------+--------+
| ID                                   | Name                  | MAC Address       | Fixed IP Addresses                                                           | Status |
+--------------------------------------+-----------------------+-------------------+------------------------------------------------------------------------------+--------+
| 2038c49f-200c-4111-93f1-68c2b62ee718 | OBSD1_int203          | fa:16:3e:29:7e:ac |                                                                              | ACTIVE |
| 2f6a51b9-4606-4c11-a66d-a967b145d592 | OBSD1_pfsync          | fa:16:3e:40:db:68 |                                                                              | ACTIVE |
| 4c610117-4f5d-4308-9612-a198fc3ecfa4 |                       | fa:16:3e:4e:fa:80 | ip_address='10.112.203.2', subnet_id='dd63f499-a14c-4437-af4a-7cd590de1cd4'  | ACTIVE |
| 529ae566-48ba-40d0-a0a5-00664dcf82e6 | OBSD2_int203          | fa:16:3e:4d:77:c5 |                                                                              | ACTIVE |
| 542a7728-6386-470c-9a6f-50f3d0bea81f | fixed                 | fa:16:3e:27:c1:3d |                                                                              | ACTIVE |
| 6a7c3c1b-542c-4d9d-954c-c7008aa771aa |                       | fa:16:3e:7c:5f:58 | ip_address='10.112.203.7', subnet_id='dd63f499-a14c-4437-af4a-7cd590de1cd4'  | ACTIVE |
| a192002a-f3c6-4635-b10e-281d4dc57e21 | int_203_10.112.203.11 | fa:16:3e:f7:f1:4f | ip_address='10.112.203.11', subnet_id='dd63f499-a14c-4437-af4a-7cd590de1cd4' | ACTIVE |
| ab2b66d7-c124-4b5d-a1e7-cb77f4b57595 | OBSD2_extif           | fa:16:3e:a0:b4:2b | ip_address='95.163.182.60', subnet_id='ec5d4a62-5039-460c-833f-7084a19794d2' | ACTIVE |
| add9c7be-b9c5-430d-98ae-75db9f99697e | 10.112.203.12         | fa:16:3e:5b:df:d9 |                                                                              | ACTIVE |
| c76ad8de-5fad-49d9-8b31-23beb7977661 | OBSD1_extif           | fa:16:3e:24:ca:c8 | ip_address='95.163.182.69', subnet_id='ec5d4a62-5039-460c-833f-7084a19794d2' | ACTIVE |
| d4dbe38e-b4e1-462c-8c79-0059014648ce |                       | fa:16:3e:77:6c:00 |                                                                              | ACTIVE |
| e65d075e-13f9-423e-8696-6f13971c4692 | OBSD2_pfsync          | fa:16:3e:04:e6:64 |                                                                              | ACTIVE |
+--------------------------------------+-----------------------+-------------------+------------------------------------------------------------------------------+--------+

(openstack) subnet list
+--------------------------------------+--------------------+--------------------------------------+-------------------+
| ID                                   | Name               | Network                              | Subnet            |
+--------------------------------------+--------------------+--------------------------------------+-------------------+
| 01009166-1de2-413d-995c-8c2272f1bc19 | ext-subnet10       | 298117ae-3fa4-4109-9e08-8be5602be5a2 | 85.192.32.0/22    |
| 1ea7f321-4ed0-4ae7-a136-a0226b9c5969 | ext-subnet11       | 298117ae-3fa4-4109-9e08-8be5602be5a2 | 89.208.208.0/22   |
| 2d3d92bf-acc1-41db-8e84-bf1fd9052cf3 | int-subnet1        | 19fcebaa-5b40-445e-837d-2e751579a2b1 | 192.168.1.0/24    |
| 47786b6d-1135-4735-9c5b-376976bc7034 | int-subnet1-poll6  | 19fcebaa-5b40-445e-837d-2e751579a2b1 | 192.168.6.0/24    |
| 7f876978-01fe-43ab-8c77-7e6e32cd28c4 | ext-subnet8        | 298117ae-3fa4-4109-9e08-8be5602be5a2 | 89.208.196.0/22   |
| 8708d714-b54e-4394-b08e-69b56f1d6672 | int-subnet1-poll5  | 19fcebaa-5b40-445e-837d-2e751579a2b1 | 192.168.5.0/24    |
| 8e0231a8-944a-44df-898e-2b1740d80a78 | int-subnet1-poll9  | 19fcebaa-5b40-445e-837d-2e751579a2b1 | 192.168.9.0/24    |
| 9357da81-c8f3-4c9b-ad50-df4f01e76acf | ext-subnet3        | 298117ae-3fa4-4109-9e08-8be5602be5a2 | 128.140.175.96/29 |
| 94640c6b-6298-40d0-8c71-6aab8716d48f | ext-subnet4        | 298117ae-3fa4-4109-9e08-8be5602be5a2 | 95.163.212.0/22   |
| aa2689f9-a208-4bf2-bed0-c20dab001467 | ext-subnet7        | 298117ae-3fa4-4109-9e08-8be5602be5a2 | 89.208.84.0/22    |
| af7e6915-4dbb-4868-924e-bffde931c02a | int-subnet1-16/20  | 19fcebaa-5b40-445e-837d-2e751579a2b1 | 192.168.16.0/20   |
| be8539d5-eeff-4eaa-8048-9f7c3dbc8804 | ext-subnet6        | 298117ae-3fa4-4109-9e08-8be5602be5a2 | 95.163.180.0/23   |
| c4f89da6-529f-4a08-9df1-6b95842a07b9 | ext-subnet1        | 298117ae-3fa4-4109-9e08-8be5602be5a2 | 95.163.248.0/22   |
| c6fafdba-deb7-4ad0-83fd-ec893dedfb69 | ext-subnet2        | 298117ae-3fa4-4109-9e08-8be5602be5a2 | 79.137.174.0/23   |
| c8865df7-892b-43ac-a274-9d9141453d5f | int-subnet1-poll2  | 19fcebaa-5b40-445e-837d-2e751579a2b1 | 192.168.2.0/24    |
| d5f70b09-6d49-445b-99f1-184d366decf6 | ext-subnet5        | 298117ae-3fa4-4109-9e08-8be5602be5a2 | 95.163.208.0/22   |
| d8174518-8545-416e-8fef-148681c51d29 | pfsync0            | ae0b721c-e0e8-48f9-b8f7-7136387975a1 | 192.168.254.0/30  |
| dd01624c-649f-4fd5-b1a7-e3660adb8f21 | int-subnet1-poll10 | 19fcebaa-5b40-445e-837d-2e751579a2b1 | 192.168.10.0/24   |
| dd63f499-a14c-4437-af4a-7cd590de1cd4 | int_203            | 7c9f921e-ef3b-4df0-a35e-9a2d0f97fcd2 | 10.112.203.0/24   |
| ec5d4a62-5039-460c-833f-7084a19794d2 | ext-subnet9        | 298117ae-3fa4-4109-9e08-8be5602be5a2 | 95.163.182.0/23   |
+--------------------------------------+--------------------+--------------------------------------+-------------------+


(openstack) network list
+--------------------------------------+----------+- ------------------------------------------------------------------------------------------------------------------+
| ID                                   | Name     | Subnets                                                                                                            |
+--------------------------------------+----------+--------------------------------------------------------------------------------------------------------------------+
| 19fcebaa-5b40-445e-837d-2e751579a2b1 | int-net1 | 2d3d92bf-acc1-41db-8e84-bf1fd9052cf3, 47786b6d-1135-4735-9c5b-376976bc7034, 8708d714-b54e-4394-b08e-69b56f1d6672, 8e0231a8-944a-44df-898e-2b1740d80a78, af7e6915-4dbb-4868-924e-bffde931c02a, c8865df7-892b-43ac-a274-9d9141453d5f, dd01624c-649f-4fd5-b1a7-e3660adb8f21                                                       |
| 298117ae-3fa4-4109-9e08-8be5602be5a2 | ext-net  | 01009166-1de2-413d-995c-8c2272f1bc19, 1ea7f321-4ed0-4ae7-a136-a0226b9c5969, 7f876978-01fe-43ab-8c77-7e6e32cd28c4, 9357da81-c8f3-4c9b-ad50-df4f01e76acf, 94640c6b-6298-40d0-8c71-6aab8716d48f, aa2689f9-a208-4bf2-bed0-c20dab001467, be8539d5-eeff-4eaa-8048-9f7c3dbc8804, c4f89da6-529f-4a08-9df1-6b95842a07b9, c6fafdba-deb7-4ad0-83fd-ec893dedfb69, d5f70b09-6d49-445b-99f1-184d366decf6, ec5d4a62-5039-460c-833f-7084a19794d2 |
| 7c9f921e-ef3b-4df0-a35e-9a2d0f97fcd2 | internal | dd63f499-a14c-4437-af4a-7cd590de1cd4                                                                               |
| ae0b721c-e0e8-48f9-b8f7-7136387975a1 | pfsync   | d8174518-8545-416e-8fef-148681c51d29                                                                               |
```

### Траблшутинг
Пока не нужно. Ранее нужно было, когда по два порта висело на инстансе, и один из портов занимал ip-адрес чужого инстанса. Здесь вместо `$1` поставить `id` инстанса. Либо в Linux это выполняется скриптом и id указывается в качестве аргумента скрипта.

```bash
openstack port set $1 --no-fixed-ip
openstack port set $1 --no-allowed-address
openstack port set $1 --no-security-group
openstack port set $1 --disable-port-security
```

## OpenBSD

Primary:
```bash
ext_if: 95.163.182.60/23 default route 95.163.183.254 mtu
1450
доступ извне ssh -p 6022 loki@ 95.163.182.60
int_if: 10.112.203.252/24 on vio2 mtu 1450
carp203: 10.112.203.254/24 on vio2 (MASTER)
pfsync0: on vio1 192.168.254.2/24
FW: /etc/pf.conf
IPSEC: /etc/ipsec.conf
OSPF: /etc/ospfd.conf
gre8 to Звенигородское ш.
```
Secondary:
```bash
ext_if: on vio1 95.163.182.69/23 default route
95.163.183.254 mtu 1450
доступ извне ssh -p 6022 loki@ 95.163.182.60
int_if: 10.112.203.251/24 on vio2 mtu 1450
carp203: 10.112.203.254/24 on vio2 (BACKUP)
pfsync0: on vio0 192.168.254.1/24
FW: /etc/pf.conf
IPSEC: /etc/ipsec.conf
OSPF: /etc/ospfd.conf
gre7 to Звенигородское ш.
```

Проверка интерфейса
```bash
obsd2-mail-ru# ifconfig carp
pfsync0: flags=41<UP,RUNNING> mtu 1500
	index 6 priority 0 llprio 3
	pfsync: syncdev: vio1 maxupd: 128 defer: off
	groups: carp pfsync
carp203: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> mtu 1500
	lladdr 00:00:5e:00:01:cb
	description: INT_203
	index 8 priority 15 llprio 3
	carp: MASTER carpdev vio2 vhid 203 advbase 1 advskew 0
	groups: carp
	status: master
	inet 10.112.203.254 netmask 0xffffff00 broadcast 10.112.203.255
```