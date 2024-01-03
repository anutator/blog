---
tags:
  - network
  - security
title: fail2ban
share: "true"
---

**Fail2ban** — простой в использовании локальный сервис, который отслеживает log–файлы запущенных программ, и на основании различных условий блокирует по IP найденных нарушителей, используя правила файрвола. Fail2ban может «из коробки» защищать ssh–серверы от атак типа «brute-force» (перебор паролей), а также умеет бороться с атаками на все популярные сервисы: Apache, Nginx, ProFTPD, vsftpd, Exim, Postfix, named, Asterisk (ip-телефония) и т.д.   
## Установка в CentOS7
Рассмотрим, как защитить с помощью Fail2ban ip-телефонию Asterisk, ssh, файловые серверы.
Если Asterisk имеет внешний ip адрес, либо на него сделан проброс портов с внешнего ip, то необходимо обеспечить защиту от перебора учетных записей и прочих множественных подключений.

Стандартная установка Fail2ban:

```bash
yum -y install epel-release
yum -y install fail2ban{,-systemd}
# Если включен SELinux, нужно обновить его политики
yum -y update selinux-policy*
```

Если доступ в интернет закрыт или ограничен, установка немного усложняется. Например стандартные репозитории открыты, а доступ к дополнительным закрыт. Тогда надо поставить из скачанного rpm (скачать на другой сервер и скопировать по ftp). Для установки rpm с одновременной установкой недостающего пакета используем `yum localinstall`.

```bash
yum localinstall fail2ban-{server-0.9.6-3.el7.noarch.rpm,sendmail-0.9.6-3.el7.noarch.rpm,firewalld-0.9.6-3.el7.noarch.rpm,0.9.6-3.el7.noarch.rpm}
```

Также можно установить самую свежую версию fail2ban из исходников, хотя сейчас это не нужно — через репозиторий доступна самая свежая версия  2020 года 0.11.2.

```bash
yum update && yum install -y git python gamin-python   # зависимости
wget https://github.com/fail2ban/fail2ban/archive/refs/tags/0.11.2.tar.gz

tar xf 0.11.2.tar.gz  (или zxpvf)
# или 
git clone https://github.com/fail2ban/fail2ban.git

cd fail2ban*
python -V # проверить версию Python. Не ниже 2.6 или 3.2.
sudo python setup.py install # установи Fail2Ban в каталог библиотеки Python
```

Исполняемые скрипты разместятся в */usr/bin*, а файлы конфигурации в */etc/fail2ban*. Проверка установки:

```bash
fail2ban-client -h # помощь по опциям
fail2ban-client -V # версия
```

Всегда будем использовать **fail2ban-client** и никогда **fail2ban-server** напрямую.

При стандартной установке Fail2ban был установлен пакет **fail2ban-systemd** для запуска службы fail2ban.
При установке из файлов rpm не забыть установить такой же пакет.
При установке из исходников для того, чтобы fail2ban стал службой, скопировать скрипт из каталога files дистрибутива в /etc/init.d:

```bash
cp files/redhat-initd /etc/init.d/fail2ban
chmod 755 /etc/init.d/fail2ban   (опционально, т.к. права уже 755)
chkconfig --add fail2ban && chkconfig fail2ban on
```

## Установка в Oracle Linux 8
Стандартная установка в Oracle Linux 8.

```bash
dnf config-manager --set-enabled powertools            # CentOS 8
dnf config-manager --set-enabled ol8_codeready_builder # Oracle Linux 8
dnf install epel-release
dnf install fail2ban
```

Проверить установку так же, как в CentOS7.

##  Файлы конфигурации
У Fail2ban 2 основных файла конфигурации:
- `/etc/fail2ban/fail2ban.conf` — отвечает за настройки запуска процесса Fail2ban.
- `/etc/fail2ban/jail.conf` — содержит настройки защиты конкретных сервисов, в том числе sshd. Для включения защиты добавлять строку  `enabled = true` в разделе соответствующего сервиса. Описания фильтров для сервисов содержатся в `/etc/fail2ban/filter.d/имя_сервиса.conf`.

Не будем менять эти файлы конфигурации, т.к. при обновлении Fail2ban перезаписывает их, а скопируем их в файлы `.local` и в дальнейшем меняем настройки только в файлах `.local`. Эти файлы имеют приоритет выше, чем стандартные файлы `.conf`, они будут подключены автоматически.

```bash
cp /etc/fail2ban/fail2ban.{conf,local}  
cp /etc/fail2ban/jail.{conf,local}
```

**Особенности настроек для разных файрволов**
Стандартным файрволом в `/etc/fail2ban/jail.local` является **iptables**:

```ini title="/etc/fail2ban/jail.local"
banaction = iptables-multiport
banaction_allports = iptables-allports
```

В CentOS8, если используется файрвол **nftables**, заменить **iptables** на **nftables**:

```ini title="/etc/fail2ban/jail.local"
banaction = nftables-multiport
banaction_allports = nftables-allports
```

В CentOS7 и Oracle Linux 8, если используется firewalld (надстройка над iptables), заменить **iptables** на **firewallcmd**:

```ini title="/etc/fail2ban/jail.local"
banaction = firewallcmd-multiport
banaction_allports = firewallcmd-allports
```

!!! note
    По умолчанию fail2ban в CentOS 7 использует команды **firewallcmd-rich-rules**, а не то что мы указали выше. Это прописано в файле `/etc/fail2ban/jail.d/00-firewalld.conf`:
    ```bash title="/etc/fail2ban/jail.d/00-firewalld.conf"
    # This file is part of the fail2ban-firewalld package to configure the use of
    # the firewalld actions as the default actions.  You can remove this package
    # (along with the empty fail2ban meta-package) if you do not use firewalld
    [DEFAULT]
    banaction = firewallcmd-rich-rules[actiontype=<multiport>]
    banaction_allports = firewallcmd-rich-rules[actiontype=<allports>]
    ```

Для использования тех действий, которые мы сами определили в `banaction` и `banaction_allports`, необходимо удалить этот файл и перезапустить **fail2ban**. Если хотим использовать **iptables**, так же удалить этот файл.

Настройки всех возможных действий (файрволов и прочих) в каталоге `/etc/fail2ban/actions.d/`. Например, **firewallcmd-multiport** описан в `/etc/fail2ban/action.d/firewallcmd-multiport.conf`. Любой файл можно использовать как есть или составить свой файл с кастомизированными правилами.

- **ignoreip** — используется для установки списка IP-адресов, которые не будут забанены (белый список). Список IP-адресов следует указывать через пробел.
- **bantime** — время блокировки, в секундах.
- **maxretry** — количество попыток перед перед блокировкой.
- **findtime** — время, на протяжении которого рассчитывается количество попыток перед баном (maxretry).

В конфиге по умолчанию прописано, что пользователь будет забанен на 10 минут, если в течение 10 минут будет совершено 5 неудачных попыток. Мы увеличим стандартное время блокировки до получаса, уменьшим максимальное число ошибок с 5 до 3:

```ini title="/etc/fail2ban/action.d/firewallcmd-multiport.conf"
# "bantime" is the number of seconds that a host is banned.
bantime  = 30m
# A host is banned if it has generated "maxretry" during the last "findtime" seconds.
findtime  = 10m
# "maxretry" is the number of failures before a host get banned.
maxretry = 3
```

## Защита ssh сессий
Стандартный шаблон jail для sshd в `/etc/fail2ban/jail.local`

```ini title="/etc/fail2ban/jail.local"
[sshd]
# To use more aggressive sshd modes set filter parameter "mode" in jail.local:
# normal (default), ddos, extra or aggressive (combines all).
# See "tests/files/logs/sshd" or "filter.d/sshd.conf" for usage example and details.
#mode   = normal
port    = ssh
logpath = %(sshd_log)s
backend = %(sshd_backend)s
```

Изменим его (вместо изменения `/etc/fail2ban/jail.local` можно создать отдельный файл `/etc/fail2ban/jail.d/sshd.local`)

```ini title="/etc/fail2ban/jail.d/sshd.local"
[sshd]
enabled  = true
port     = ssh
filter   = sshd
action   = firewallcmd-ipset
logpath  = %(sshd_log)s
maxretry = 5
bantime  = 1440m
```

- `action` используется для получения IP-адреса, который необходимо заблокировать, используя фильтр, доступный в /etc/fail2ban/action.d/firewallcmd-ipset.conf.
- `logpath` — путь, где хранится файл журнала. Этот файл журнала сканируется Fail2Ban.
- `maxretry` — после какого количества неудачных входов в систему будет заблокирован ip-адрес.
- `bantime` — время блокировки (24 часа).

При изменении порта ssh с 22 на нестандартный заменить `port = ssh` на конкретный номер порта.
## Защита ip-телефонии Asterisk
Максимальное число попыток регистрации ip-телефона на свой вкус, по умолчанию 10, можно уменьшить до 3. Изменила logpath. Добавила порт 5062.  `filter=asterisk` указывает на готовый шаблон фильтров из файла `/etc/fail2ban/filter.d/asterisk.conf`

```ini title="/etc/fail2ban/filter.d/asterisk.conf"
[asterisk]
enabled  = true
filter   = asterisk
port     = 5060,5061,5062
action_  = %(default/action_)s[name=%(__name__)s-tcp, protocol="tcp"]
           %(default/action_)s[name=%(__name__)s-udp, protocol="udp"]
# logpath  = /var/log/asterisk/messages
logpath  = /var/log/asterisk/security
maxretry = 10
# опционально
bantime = 86400
```

Т.к. logpath изменили, включаем запись в лог файл событий типа security в файле `/etc/asterisk/logger.conf`:

```ini title="/etc/asterisk/logger.conf"
security => security
```

Перечитываем настройки хранения логов:

```bash
asterisk -rx "logger reload"
```

Если не получается перегрузить logger, зайдем в консоль Астериск, чтоб увидеть причину:

```bash
astergren*CLI> logger reload  
Failed to reload the logger  
Command 'logger reload' failed.  
[May 31 18:12:08]   == Parsing '/etc/asterisk/logger.conf': Found  
ERROR: Unable to open log file '/var/log/asterisk/messages': Permission denied'  
ERROR: Unable to create log channel 'messages'  
[May 31 18:12:08] ERROR[30040]: logger.c:852 logger_queue_restart: Unable to create queue log: Permission denied  
[May 31 18:12:08]  Asterisk Queue Logger restarted
```

Смотрим что в каталоге */var/log/asterisk* владелец каталоге messages root:

```bash
[root@astergren asterisk]# pwd  
/var/log/asterisk  
[root@astergren asterisk]# ls –l  
total 11  
drwxr-xr-x 2 asterisk asterisk   24 Apr  3 14:57 cdr-csv  
drwxr-xr-x 2 asterisk asterisk    6 Mar 31 14:10 cdr-custom  
drwxr-xr-x 2 asterisk asterisk    6 Mar 31 14:10 cel-custom  
-rw-r--r-- 1 root     root        0 Mar 31 17:09 h323_log  
-rw-r--r-- 1 root     root     3209 Mar 31 17:09 messages  
-rw-r--r-- 1 root     root       38 Mar 31 17:09 queue_log
```

Даем права пользователю asterisk, после чего `logger reload` работает.

```bash
chown -R asterisk. /var/{lib,log,spool}/asterisk
```

В папке _/var/log/asterisk_ появился новый файл **security**:

```bash
[root@astergren asterisk]# ls –l  
total 12  
drwxr-xr-x 2 asterisk asterisk   24 Apr  3 14:57 cdr-csv  
drwxr-xr-x 2 asterisk asterisk    6 Mar 31 14:10 cdr-custom  
drwxr-xr-x 2 asterisk asterisk    6 Mar 31 14:10 cel-custom  
-rw-r--r-- 1 asterisk asterisk    0 Mar 31 17:09 h323_log  
-rw-r--r-- 1 asterisk asterisk 3340 May 31 18:15 messages  
-rw-r--r-- 1 asterisk asterisk   78 May 31 18:15 queue_log  
**-rw-r--r-- 1 asterisk asterisk 1840 May 31 18:18 security**
```

## Защита веб-сервера Nginx
Стандартные jail (ловушки) для Nginx в `/etc/fail2ban/jail.local`:

```ini title="/etc/fail2ban/jail.local"
[nginx-http-auth]
port    = http,https
logpath = %(nginx_error_log)s

# To use 'nginx-limit-req' jail you should have `ngx_http_limit_req_module` 
# and define `limit_req` and `limit_req_zone` as described in nginx documentation
# http://nginx.org/en/docs/http/ngx_http_limit_req_module.html
# or for example see in 'config/filter.d/nginx-limit-req.conf'
[nginx-limit-req]
port    = http,https
logpath = %(nginx_error_log)s

[nginx-botsearch]
port     = http,https
logpath  = %(nginx_error_log)s
maxretry = 2
```

Настроим свои собственные «кастомные» jail в `/etc/fail2ban/jail.local`:

```ini title="/etc/fail2ban/jail.local"
[nginx-403]
 enabled  = true
 port     = http,https
 filter   = nginx-403
 action   = firewallcmd-allports
 logpath  = %(nginx_access_log)s
 bantime  = 1440m # 1 день
 findtime = 1440m # 1 день
 maxretry = 4

[nginx-404]
 enabled  = true
 port     = http,https
 filter   = nginx-404
 action   = firewallcmd-allports
 logpath  = %(nginx_access_log)s
 bantime  = 1440m # 1 день
 findtime = 1440m # 1 день
 maxretry = 30 # мониторить и настраивать, все серверы разные

[nginx-noagent]
 enabled  = true
 port     = http,https
 filter   = nginx-noagent
 action   = firewallcmd-allports
 logpath  = %(nginx_access_log)s
 bantime  = 1440m # 1 день
 findtime = 1440m # 1 день
 maxretry = 3

[nginx-noauth]
 enabled  = true
 filter   = nginx-noauth
 action   = firewallcmd-allports
 logpath  = %(nginx_error_log)s
 bantime  = 1440m # 1 день
 findtime = 1440m # 1 день
 maxretry = 5

[nginx-no-x-spam]
 enabled  = true
 filter   = nginx-no-x-spam
 action   = firewallcmd-allports
 logpath  = %(nginx_access_log)s
 bantime  = 1440m # 1 день
 findtime = 1440m # 1 день
 maxretry = 5

[nginx-noscript]
 enabled  = true
 action   = firewallcmd-allports
 filter   = nginx-noscript
 logpath  = %(nginx_access_log)s
 bantime  = 1440m # 1 день
 findtime = 1440m # 1 день
 maxretry = 3

[nginx-noproxy]
 enabled  = true
 action   = firewallcmd-allports
 filter   = nginx-noproxy
 logpath  = %(nginx_access_log)s
 bantime  = 1440m # 1 день
 findtime = 1440m # 1 день
 maxretry = 0

[nginx-nowordpress]
 enabled  = true
 action   = firewallcmd-allports
 filter   = nginx-nowordpress
 logpath  = %(nginx_access_log)s
 bantime  = 1440m # 1 день
 findtime = 1440m # 1 день
 maxretry = 3

[portscan-block]
 enabled  = true
 action   = firewallcmd-allports
 filter   = portscan-block
 logpath  = /var/log/firewalld  # путь к логу файрвола
 bantime  = 1440m # 1 day
 findtime = 1440m # 1 day
 maxretry = 5
 
# блокирует ip-адреса, которые обращаются к скриптам php, asp, exe, pl, cgi, scgi
[nginx-noscript]
enabled  = true
filter   = nginx-noscript
port     = http,https
logpath  = /var/log/nginx/access.log
bantime  = 600
maxretry = 6


[nginx-noproxy]
enabled  = true
filter   = nginx-noproxy
port     = http,https
logpath  = /var/log/nginx/access.log
bantime  = 600
maxretry = 2

# блокирует ip-адреса, которые используют метод Connect
[nginx-noconnect]
enabled  = true
filter   = nginx-noconnect
port     = http,https
logpath  = /var/log/nginx/access.log
bantime  = 600
maxretry = 2
```

Добавить файлы с соответствующими фильтрами в каталог `/etc/fail2ban/filter.d/`

```ini title="/etc/fail2ban/filter.d/..."
# /etc/fail2ban/filter.d/nginx-403.conf
[Definition]
failregex = ^<HOST> -.*"(GET|POST|HEAD).*HTTP.*" 403
ignoreregex =

# /etc/fail2ban/filter.d/nginx-404.conf
[Definition]
failregex = ^<HOST> -.*"(GET|POST|HEAD).*HTTP.*" 404
ignoreregex =

# /etc/fail2ban/filter.d/nginx-noagent.conf
[Definition]
failregex = ^<HOST> -.*"-" "-"$
             ^<HOST> -.*"-" "curl.*"$
ignoreregex =

# /etc/fail2ban/filter.d/nginx-noauth.conf
 [Definition]
failregex = no user/password was provided for basic authentication.client: user . was not found in.client:              user . password mismatch.*client: 
ignoreregex =

# /etc/fail2ban/filter.d/nginx-no-x-spam.conf
[Definition]
failregex   = {"log":"<HOST> .* ".*\\x.*" .*$
ignoreregex =

# /etc/fail2ban/filter.d/nginx-noscript.conf
[Definition]
failregex = ^<HOST> -.*GET.*(\.asp|\.exe|\.pl|\.cgi|\.scgi)
ignoreregex =

# /etc/fail2ban/filter.d/nginx-noproxy.conf
[Definition]
failregex = ^<HOST> -.*GET http.*
             ^<HOST> -.*CONNECT.*
ignoreregex =

# /etc/fail2ban/filter.d/nginx-nowordpress.conf
[Definition]
 failregex = ^<HOST> -.*"(GET|POST|HEAD) /+(?i)(wp(-|/)|xmlrpc.php|\?author=1)
             ^<HOST> -.* "(GET|POST|HEAD|PROPFIND) /+(?i)    (a2billing|admin|apache|axis|blog|cfide|cgi|cms|config|etc|.git|hnap|inc|jenkins|jmx-|joomla|lib|linuxsucks|msd|muieblackcat|mysql|myadmin|n0w|owa-autodiscover|pbxip|pma|recordings|sap|sdk|script|service|shell|sqlite|vmskdl44rededd|vtigercrm|w00tw00t|webdav|websql|wordpress|xampp|xxbb)
             ^<HOST> -.* "(GET|POST|HEAD) /[^"]+.(asp|cgi|exe|jsp|mvc|pl)( |\?)
             ^<HOST> -.*(?i)(/bash|burger-imperia|changelog|hundejo|hvd-store|jorgee|masscan|pizza-imperia|pizza-tycoon|servlet|testproxy|uploadify)
ignoreregex  =
journalmatch = _SYSTEMD_UNIT=nginx.service + _COMM=nginx

# /etc/fail2ban/filter.d/portscan-block.conf
[Definition]
failregex = .*\[UFW BLOCK\] IN=.* SRC=<HOST>
ignoreregex = SRC=(10.|172.1[6-9].|172.2[0-9].|172.3[0-1].|192.168.|fe\w:). DST=(static.ip.address.here|224.0.0.). PROTO=(2|UDP)(\s+|.* DPT=(1900|3702|5353|5355) LEN=\d*\s+)$

# /etc/fail2ban/filter.d/nginx-noscript.conf
[Definition]
failregex = ^<HOST> -.*GET.*(\.php|\.asp|\.exe|\.pl|\.cgi|\.scgi)
ignoreregex =

# /etc/fail2ban/filter.d/nginx-noproxy.conf
[Definition]
failregex = ^<HOST> -.*GET http.*
ignoreregex =


# /etc/fail2ban/filter.d/nginx-noconnect.conf
[Definition]
failregex = ^<HOST> -.*CONNECT
ignoreregex =
```
### Фильтр для WebExploit-ов
Создадим фильтр, который блокирует URL злонамеренных бот-сканеров. Фильтр требует настройки, потому что может заблокировать реальных пользователей, но в основном он будет банить нормальных ботов SEO, которые могут случайно сканировать ту же самую ловушку. Можно скачать несколько созданных сообществом и обновленных списков, но лучший способ — мониторить свои логи и добавить свои собственные списки, когда будет замечено сканирование. Через какое-то время вы накопите большой собственный список, и фильтр будет работать без ложных срабатываний.
Добавить jail в `/etc/fail2ban/jail.local`:

```ini title="/etc/fail2ban/jail.local"
[webexploits]
enabled  = false
logpath  = %(nginx_access_log)s
filter   = webexploits
port     = http,https
bantime  = 1440m
findtime = 1440m
maxretry = 0
```

Создать фильтр `/etc/fail2ban/filter.d/webexploits.conf`, от которого можно отталкиваться, добавляя собственные регулярные выражения.

```ini title="/etc/fail2ban/filter.d/webexploits.conf"
[Definition]
failregex = 
            ^<HOST> -.*(GET|POST|HEAD).*(/data/admin/)
            ^<HOST> -.*(GET|POST|HEAD).*(/dnt-policy.txt)
            ^<HOST> -.*(GET|POST|HEAD).*(/gpc.json)
            ^<HOST> -.*(GET|POST|HEAD).*(/wlwmanifest.xml)
            ^<HOST> -.*(GET|POST|HEAD).*(/adminer)
            ^<HOST> -.*(GET|POST|HEAD).*(/_adminer)
            ^<HOST> -.*(GET|POST|HEAD).*(/user/login)
            ^<HOST> -.*(GET|POST|HEAD).*(/.env)
ignoreregex =
```

Пример добавления выражения в фильтр, например  WordPress XMLRPC.

```ini title="/etc/fail2ban/filter.d/webexploits.conf"
            ^<HOST> -.*(GET|POST|HEAD).*(/xmlrpc.php)
```

Также можно менять опции **GET|POST|HEAD**, но если мы хотим банить абсолютно любые запросы на указанные URL, опции менть не надо. Готовые списки с эксплоитами можно взять [здесь](https://github.com/bigalownz/Fail2Ban.WebExploits). Но нельзя просто копировать и вставлять. Это требует тестирования и анализа собственных логов.
### Безопасные страницы Nginx и Honeypot
Другая интересная опция — создание фильтра 444, при создании геолокаций, чтобы, например спрятать такие страницы как login, которые не должны открывать ни хорошие боты, ни реальные пользователи.
Добавить jail в `/etc/fail2ban/jail.local`:

```ini title="/etc/fail2ban/jail.local"
[nginx-444]
 enabled  = true
 port     = http,https
 filter   = nginx-444
 action   = firewallcmd-allports
 logpath  = %(nginx_access_log)s
 bantime  = 1440m # 1 день
 findtime = 1440m # 1 день
 maxretry = 1
```

Добавление jail в этот раз, моментально баним его. Надо добавить страницу в robots.txt в качестве исключения, чтобы хорошие SEO боты типа Google не попадали на неё. Надо создать фейковые страницы и также исключить их в robots.txt в качестве honeypot-ов. Пример работает для актуальных страниц или honeypot-ов.
Добавить фильтр `/etc/fail2ban/filter.d/nginx-444.conf`

```ini title="/etc/fail2ban/filter.d/nginx-444.conf"
[Definition]
failregex = ^<HOST> -.*"(GET|POST|HEAD).*HTTP.*" 444
ignoreregex =
```

Создать файл с белым списком, куда добавить ваш ip-адрес или диапазон ip-адресов, которые мы хотим исключить из адресов, которые могут попасть на страницу. Обратить внимание на false в конце.

```bash
sudo nano /etc/nginx/whitelist.txt

IP Address false;
# или
195.168.50.1 false;
```

Добавить настройки в `/etc/nginx.conf`.  Отметим, что исключение трафика по умолчанию установлено в **true**.

```nginx title="/etc/nginx.conf"
geo $remote_addr $exclude_trafic {
 default true;
include /etc/nginx/whitelist.txt;
}
```

В этом же файле в блоке, относящемся к настройкам сервера, добавить:

```nginx title="/etc/nginx.conf"
if ( $allowed_trafic = 'true'){
    return 444;
}
```

Для сайтов на PHP, например WordPress, разблокировать например  wp-login.php. Возможно страница будет другой в зависимости от версии PHP и версии WordPress. Итак, в примере исключаем весь трафик с этой страницы, если вы не находитесь в белом списке.

```nginx  title="/etc/nginx.conf"
  location ~ /wp-login.php {
   if ( $exclude_trafic = 'true'){
    return 444;
} 
 fastcgi_split_path_info ^(.+\.php)(/.+)$;
    fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_param SCRIPT_NAME $fastcgi_script_name;
    include fastcgi_params;
    include snippets/fastcgi-php.conf;
    fastcgi_intercept_errors on;
}
```

Убедитесь, что вы добавили html ссылки на фейковые страницы, чтобы их видели только боты, и исключите их в robots.txt, чтобы реальные полезные боты, которые следуют вашим правилам, не попадали на фейковые страницы (злонамеренные боты не обрабатывают файл robots.txt).
## Отработка рецидивов
Если ip-адрес был пойман повторно, он обрабатывается в jail **recidive**. При рецидиве ip-адрес банится на неделю, здесь я ничего не меняла.

```ini title="/etc/fail2ban/jail.local"
[recidive]
enabled = true
logpath  = /var/log/fail2ban.log
banaction = %(banaction_allports)s
bantime  = 1w
findtime = 1d
```

## Включение и тестирование работы фильтров и ловушек fail2ban
Включим fail2ban и проверим статус:

```bash
systemctl enable --now fail2ban  
systemctl status fail2ban
```

Не забываем настроить разрешение для SELinux (там где nftables в CentOS8):

```bash
ausearch -c 'nft' --raw | audit2allow -M my-nft  
9462 semodule -X 300 -i my-nft.pp
```

После изменени правил Fail2ban достаточно перезагрузить конфигурацию:

```bash
# fail2ban-client reload  
OK
```

Проверка статуса конкретной ловушки (jail). Видим, что забанено 95 ip-адресов, а всего 107 (в это число включены активные и неактивные адреса, которые уже были разблокированы).

```bash
sudo fail2ban-client status nginx-noscripts

Status for the jail: nginx-noscripts
 |- Filter
 | |- Currently failed: 0
 | |- Total failed: 0
 | - File list: /var/log/nginx/access.log - Actions
 |- Currently banned: 95
 |- Total banned: 107
 `- далее отобразится список заблокированных IP-адресов
```

Проверка лога fail2ban:

```bash
2022-01-16 12:47:55,481 fail2ban.filter  [26054]: INFO    [asterisk] Found 199.127.63.186 - 2022-01-16 12:47:55
2022-01-16 12:47:55,486 fail2ban.actions [26054]: NOTICE  [asterisk] Ban 77.247.110.199
2022-01-16 12:47:55,661 fail2ban.filter  [26054]: INFO    [recidive] Found 77.247.110.199 - 2022-01-16 12:47:55
2022-01-16 12:48:01,491 fail2ban.filter  [26054]: INFO    [asterisk] Found 134.119.214.251 - 2022-01-16 12:48:01
2022-01-16 12:48:01,549 fail2ban.actions [26054]: NOTICE  [asterisk] Ban 134.119.214.251
2022-01-16 12:48:01,670 fail2ban.filter  [26054]: INFO    [recidive] Found 134.119.214.251 - 2022-01-16 12:48:01
2022-01-16 12:48:56,166 fail2ban.filter  [26054]: INFO    [asterisk] Found 37.49.230.28 - 2022-01-16 12:48:56
2022-01-16 12:48:56,291 fail2ban.actions [26054]: NOTICE  [asterisk] Ban 37.49.230.28
2022-01-16 12:48:56,345 fail2ban.filter  [26054]: INFO    [recidive] Found 37.49.230.28 - 2022-01-16 12:48:56
2022-01-16 12:49:06,187 fail2ban.filter  [26054]: INFO    [asterisk] Found 77.66.176.94 - 2022-01-16 12:49:06
2022-01-16 12:49:06,791 fail2ban.filter  [26054]: INFO    [asterisk] Found 77.66.176.94 - 2022-01-16 12:49:06
2022-01-16 12:49:06,792 fail2ban.filter  [26054]: INFO    [asterisk] Found 77.66.176.94 - 2022-01-16 12:49:06
2022-01-16 12:49:06,977 fail2ban.actions [26054]: NOTICE  [asterisk] Ban 77.66.176.94
2022-01-16 12:49:07,165 fail2ban.filter  [26054]: INFO    [recidive] Found 77.66.176.94 - 2022-01-16 12:49:06
2022-01-16 13:12:48,551 fail2ban.filter  [26054]: INFO    [asterisk] Found 164.132.63.169 - 2022-01-16 13:12:47
2022-01-16 13:14:59,935 fail2ban.filter  [26054]: INFO    [asterisk] Found 77.247.108.241 - 2022-01-16 13:14:59
2022-01-16 13:17:33,207 fail2ban.actions [26054]: NOTICE  [asterisk] Unban 77.247.110.199
2022-01-16 13:17:55,320 fail2ban.actions [26054]: NOTICE  [asterisk] Unban 199.127.63.186
2022-01-16 13:18:01,419 fail2ban.actions [26054]: NOTICE  [asterisk] Unban 134.119.214.251
2022-01-16 13:18:11,203 fail2ban.filter  [26054]: INFO    [asterisk] Found 134.119.214.251 - 2022-01-16 13:18:10
2022-01-16 13:18:25,827 fail2ban.filter  [26054]: INFO    [asterisk] Found 134.119.214.251 - 2022-01-16 13:18:25
2022-01-16 13:18:51,865 fail2ban.filter  [26054]: INFO    [asterisk] Found 134.119.214.251 - 2022-01-16 13:18:51
2022-01-16 13:18:52,165 fail2ban.actions [26054]: NOTICE  [asterisk] Ban 134.119.214.251
2022-01-16 13:18:52,211 fail2ban.filter  [26054]: INFO    [recidive] Found 134.119.214.251 - 2022-01-16 13:18:52
2022-01-16 13:18:56,270 fail2ban.actions [26054]: NOTICE  [asterisk] Unban 37.49.230.28
2022-01-16 13:19:06,370 fail2ban.actions [26054]: NOTICE  [asterisk] Unban 77.66.176.94
2022-01-16 13:20:02,162 fail2ban.filter  [26054]: INFO    [asterisk] Found 77.247.110.58 - 2022-01-16 13:20:02
2022-01-16 13:20:18,188 fail2ban.filter  [26054]: INFO    [asterisk] Found 77.247.110.58 - 2022-01-16 13:20:17
2022-01-16 13:20:46,631 fail2ban.filter  [26054]: INFO    [asterisk] Found 37.49.230.28 - 2022-01-16 13:20:46
```

Пример ручной разблокировки забаненного ip-адреса. Имя jail — asterisk, это видно по логу.

```bash
sudo fail2ban-client set asterisk unbanip 37.49.230.28
```

Пример лога при перечитывании конфига:

```bash
2022-01-16 13:21:00,247 fail2ban.server   [26054]: INFO    Reload all jails
2022-01-16 13:21:00,248 fail2ban.server   [26054]: INFO    Reload jail 'asterisk'
2022-01-16 13:21:00,249 fail2ban.filter   [26054]: INFO      maxRetry: 3
2022-01-16 13:21:00,250 fail2ban.filter   [26054]: INFO      findtime: 600
2022-01-16 13:21:00,250 fail2ban.actions  [26054]: INFO      banTime: 1800
2022-01-16 13:21:00,250 fail2ban.filter   [26054]: INFO      encoding: UTF-8
2022-01-16 13:21:00,251 fail2ban.server   [26054]: INFO    Reload jail 'recidive'
2022-01-16 13:21:00,251 fail2ban.server   [26054]: INFO    Jail recidive is not a JournalFilter instance
2022-01-16 13:21:00,251 fail2ban.filter   [26054]: INFO      maxRetry: 3
2022-01-16 13:21:00,252 fail2ban.filter   [26054]: INFO      findtime: 86400
2022-01-16 13:21:00,252 fail2ban.actions  [26054]: INFO      banTime: 604800
2022-01-16 13:21:00,252 fail2ban.filter   [26054]: INFO      encoding: UTF-8
2022-01-16 13:21:00,252 fail2ban.server   [26054]: INFO    Jail 'asterisk' reloaded
2022-01-16 13:21:00,253 fail2ban.server   [26054]: INFO    Jail 'recidive' reloaded
2022-01-16 13:21:00,254 fail2ban.server   [26054]: INFO    Reload finished.
```

Информация о версии при входе в интерактивную консоль:

```bash
# fail2ban-client -i  
Fail2Ban v0.10.4 reads log file that contains password failure report  
and bans the corresponding IP addresses using firewall rules.  
fail2ban
```

Команды в сочетании с `fail2ban-client`:

| Command          | Description                                                |
| ---------------- | ---------------------------------------------------------- |
| `start`          | запуск сервера и всех jail                                 |
| `reload`         | перезагружает конфигурацию                                 |
| `reload <jail> ` | перезагружает jail `<jail>`                                |
| `status`         | текущий статус сервера                                     |
| `status <jail>`  | текущий статус конкретного jail (например status asterisk) |
| `stop`           | остановка всех jails и сервера                             |
| `ping`           | проверка, жив ли сервер                                    |
| `help`           | отображение подсказки после входа в интерактивное меню     |

Как протестировать фильтр fail2ban на примере фильтра sshd. Команда проанализирует файл лога и отобразит то что будет забанено, если мы включим фильтр. Это обязательно надо протестировать на фильтрах, которые могут вызвать ложные срабатывания, например веб эксплоиты.

```bash
fail2ban-regex /var/log/auth.log /etc/fail2ban/filter.d/sshd.conf
```

