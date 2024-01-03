---
tags:
  - databases
share: "true"
title: Обновление PostgreSQL в RedOS 7.3.1 с 14 до 15 версии
---

Нас обязывают использовать российские ОС, и приходится заморачиваться, т.к. новые пакеты добавляют только по заявкам (мы оставили, но ждать долго — RedOS работает с Posgres PRO, тоже российской компанией, а там пока максимальная мажорная версия 14). На текущий момент в RedOS максимальная версия PostgreSQL 14.5.

Версия RedOS может ввести в заблуждение. Кажется, что это аналог CentOS 7, но пакеты для CentOS 7 не подходят, и надо ставить пакеты от Redhat (CentOS, Oracle) 8:
```bash
$ cat /etc/*release
RED OS release MUROM (7.3.1) MINIMALNAME="RED OS"
VERSION="MUROM (7.3.1)"
PLATFORM_ID="platform:el7"
ID="redos"
ID_LIKE="rhel centos fedora"
VERSION_ID="7.3.1"
PRETTY_NAME="RED OS MUROM (7.3.1)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:redos:redos:7"
HOME_URL="http://red-soft.ru/ru/main_products.html#redos"
BUG_REPORT_URL="http://redos-support.red-soft.ru"
EDITION="Standard"

RED OS release MUROM (7.3.1) MINIMALRED OS release MUROM (7.3.1) MINIMALRED OS release MUROM (7.3.1) MINIMAL
```

Официальные репозитории PostgreSQL — https://www.postgresql.org/download/linux/redhat/
Выбираем:
- Select Version — 15
- Select Platform — RedHat Enterprise, Rocky or Oracle version 8
- Select architecture — x86_64

```bash
sudo dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm
```

Команда добавит файл со списком репозиториев */etc/yum.repos.d/pgdg-redhat-all.repo*, но т.к. версия у нас 7.3.1, а она нам совершенно не подходит, надо через sed или вручную заменить переменную `$releasever` на последнюю версию из 8, которую увидим на странице https://download.postgresql.org/pub/repos/yum/common/redhat/, адрес которой подсмотрели в файле .repo. Здесь я ставлю версию релиза 8.5. В файле также будут репозитории версий 14 и ниже, например все что в секции `[pgdg13-source]`. Их можно удалить вручную, но удобнее это сделать автоматически:

```bash
# Удалить всё что до строки, содержащей pgdg13 - мне не подходит
sed '/pgdg13/,$!d' /etc/yum.repos.d/pgdg-redhat-all.repo

# оставить секции, которые содержат pgdg15
# в качестве разделителя на секции используется pgdg в начале блока; обязательны одинарные кавычки вокруг pgdg
awk 'BEGIN { RS='pgdg' } $0 ~ /pgdg15/ { print RS$0 "\n" }' /etc/yum.repos.d/pgdg-redhat-all.repo > tmp && mv -f tmp /etc/yum.repos.d/pgdg-redhat-all.repo

# заменить все $releasever на 8.5
sed -i 's/$releasever/8.5/g' /etc/yum.repos.d/pgdg-redhat-all.repo
```

Обновить релиз 14 на 15 нельзя через `dnf update`. Надо сначала установить 15 версию тех же пакетов, которые стоят:

```bash
dnf install postgresql15{,-server}
```

Проблема: не все пакеты удается установить. Так, если попытаемся поставить postgre15-contrib, появится ошибка. Хорошо, что нам этот пакет не нужен был.

```bash
$ sudo dnf install postgresql15{,-server,-contrib}
Последняя проверка окончания срока действия метаданных: 0:00:28 назад, Чт 20 окт 2022 18:58:12.
Ошибка: 
 Проблема: conflicting requests
  - nothing provides libpython3.6m.so.1.0()(64bit) needed by postgresql15-contrib-15.0-1PGDG.rhel8.x86_64
  - nothing provides libperl.so.5.26()(64bit) needed by postgresql15-contrib-15.0-1PGDG.rhel8.x86_64
(попробуйте добавить «--skip-broken» для пропуска удаления пакетов или «--nobest», чтобы использовать не только наилучшие варианты пакетов)
```

После установки 15 версии, если ранее использовался нестандартный каталог `/opt/pgsql/14` для хранения данных, надо изменить каталог и для новой версии:

```bash
systemctl edit postgresql-15.service 
```

```ini
[Service]
Environment=PGDATA=/opt/pgsql/15
```

Применить изменения:

```bash
systemctl daemon-reload
```

Теперь создадим этот каталог (владелец postgres) и инициализируем базу данных, после чего в пустом каталоге появятся файлы.

```bash
$ sudo mkdir /opt/pgsql/15
$ sudo chmod postgres. /opt/pgsql/15

# проверим
$ ls -lah /opt/pgsql
итого 16K
drwxr-xr-x.  4 root     root     4,0K окт 19 01:11 .
drwxr-xr-x.  6 root     root     4,0K июл 22 16:39 ..
drwx------. 20 postgres postgres 4,0K окт 19 11:26 14
drwx------. 20 postgres postgres 4,0K окт 20 10:52 15

# инициализируем бд
/usr/pgsql-15/bin/postgresql-15-setup initdb
```

Выполнить проверку совместимости старого и нового кластера. Если не всё Ок, то потребуются дополнительные действия по исправлению пунктов.

```bash
$ /usr/pgsql-15/bin/pg_upgrade -b /usr/pgsql-14/bin -B /usr/pgsql-15/bin -d /opt/pgsql/14 -D /opt/pgsql/15 -U postgres -c
Проверка целостности на старом работающем сервере
-------------------------------------------------
Checking cluster versions                                   ok
Checking database user is the install user                  ok
Checking database connection settings                       ok
Checking for prepared transactions                          ok
Checking for system-defined composite types in user tables  ok
Checking for reg* data types in user tables                 ok
Checking for contrib/isn with bigint-passing mismatch       ok
Checking for presence of required libraries                 ok
Checking database user is the install user                  ok
Checking for prepared transactions                          ok
Checking for new cluster tablespace directories             ok

*Кластеры совместимы*
```

Пояснения по параметрам команды `pg_upgrade`:

```bash
Параметры:
  -b, --old-bindir=КАТ_BIN      каталог исполняемых файлов старого кластера
  -B, --new-bindir=КАТ_BIN      каталог исполняемых файлов нового кластера (по умолчанию каталог программы pg_upgrade)
  -c, --check                   только проверить кластеры, не меняя никакие данные
  -d, --old-datadir=КАТ_DATA    каталог данных старого кластера (база и файны конфигурации), может быть нестандартным, как в примере ниже, потому что добавлялся отдельный диск для данных, который был смонтирован в /opt.
  -D, --new-datadir=КАТ_DATA    каталог данных нового кластера
  -j, --jobs=ЧИСЛО              число одновременно используемых процессов или потоков
  -k, --link                    устанавливать ссылки вместо копирования файлов в новый кластер
  -N, --no-sync                 не ждать завершения сохранения данных на диске
  -o, --old-options=ПАРАМЕТРЫ   параметры старого кластера, передаваемые серверу
  -O, --new-options=ПАРАМЕТРЫ   параметры нового кластера, передаваемые серверу
  -p, --old-port=ПОРТ           номер порта старого кластера (по умолчанию 50432)
  -P, --new-port=ПОРТ           номер порта нового кластера (по умолчанию 50432)
  -r, --retain                  сохранить файлы журналов и SQL в случае успеха
  -s, --socketdir=КАТАЛОГ       каталог сокетов (по умолчанию текущий)
  -U, --username=ИМЯ            суперпользователь кластера (по умолчанию "root")
  -v, --verbose                 включить вывод подробных внутренних сообщений
  -V, --version                 показать версию и выйти
  --clone                       клонировать, а не копировать файлы в новый кластер
  -?, --help                    показать эту справку и выйти
```

При тесте проблем не выявлено, поэтому может остановить кластер postgresql-14 (я заодно удаляю из автозагрузки) и выполнить `pg_upgrade` уже без режима `--check`. Проверить, что хватит места в `/opt` для хранения нового каталога, иначе можно опцией `-k` установить ссылки вместо копирования. Кроме утилиты `pg_upgrade` есть и другие способы переноса старого сластера на новую версию, например полный дамп, но здесь он не рассматривается.

```bash
$ systemctl disable --now postgresql-14

$ /usr/pgsql-15/bin/pg_upgrade -b /usr/pgsql-14/bin -B /usr/pgsql-15/bin -d /opt/pgsql/14 -D /opt/pgsql/15 -U postgres
Проведение проверок целостности
-------------------------------
Checking cluster versions                                   ok
Checking database user is the install user                  ok
Checking database connection settings                       ok
Checking for prepared transactions                          ok
Checking for system-defined composite types in user tables  ok
Checking for reg* data types in user tables                 ok
Checking for contrib/isn with bigint-passing mismatch       ok
Creating dump of global objects                             ok
Creating dump of database schemas                           
                                                            ok
Checking for presence of required libraries                 ok
Checking database user is the install user                  ok
Checking for prepared transactions                          ok
Checking for new cluster tablespace directories             ok

Если работа pg_upgrade после этого прервётся, вы должны заново выполнить initdb
для нового кластера, чтобы продолжить.

Выполнение обновления
---------------------
Analyzing all rows in the new cluster                       ok
Freezing all rows in the new cluster                        ok
Deleting files from new pg_xact                             ok
Copying old pg_xact to new server                           ok
Setting oldest XID for new cluster                          ok
Setting next transaction ID and epoch for new cluster       ok
Deleting files from new pg_multixact/offsets                ok
Copying old pg_multixact/offsets to new server              ok
Deleting files from new pg_multixact/members                ok
Copying old pg_multixact/members to new server              ok
Setting next multixact ID and offset for new cluster        ok
Resetting WAL archives                                      ok
Setting frozenxid and minmxid counters in new cluster       ok
Restoring global objects in the new cluster                 ok
Restoring database schemas in the new cluster               
                                                            ok
Copying user relation files                                 
                                                            ok
Setting next OID for new cluster                            ok
Sync data directory to disk                                 ok
Creating script to delete old cluster                       ok
Checking for extension updates                              ok

Обновление завершено
--------------------
Статистика оптимизатора утилитой pg_upgrade не переносится.
Запустив новый сервер, имеет смысл выполнить:
    /usr/pgsql-15/bin/vacuumdb -U postgres --all --analyze-in-stages

При запуске этого скрипта будут удалены файлы данных старого кластера:
    ./delete_old_cluster.sh
```

Внимание:  проверить файлы /opt/pgsql/15/postgresql.conf и /opt/pgsql/15/pg_hba.conf. Если они не совпадают со старыми, скопировать их из старого каталога, иначе потом может выясниться, что у вас была настроена авторизация через LDAP сервер, а она перестала работать, и никто кроме postgres не может залогиниться.

```bash
cp /opt/pgsql/14/{postgresql,pg_hba}.conf /opt/pgsql/15/
```

Включить и добавить в автозагрузку новый кластер:

```bash
systemctl enable --now postgresql-15
```

После включения выполнить рекомендуемую очистку (подсказка по которой была при обновлении), выполнять от лица пользователя root:

```bash
# sudo -u postgres /usr/pgsql-15/bin/vacuumdb -U postgres --all --analyze-in-stages
vacuumdb: обработка базы данных "имя_базы": Вычисление минимальной статистики для оптимизатора (1 запись)
...
vacuumdb: обработка базы данных "имя_базы": Вычисление средней статистики для оптимизатора (10 записей)
...
vacuumdb: обработка базы данных "имя_базы": Вычисление стандартной (полной) статистики для оптимизатора
```

Т.к. мы порт не меняли, то postgres должен работать на порту 5432:

```bash
$ sudo netstat -tulpn | grep post
tcp        0      0 0.0.0.0:5432            0.0.0.0:*               LISTEN      214485/postmaster   
tcp6       0      0 :::5432                 :::*                    LISTEN      214485/postmaster 

# через новую команду ss
ss -nlpt |grep post
LISTEN   0         244                 0.0.0.0:5432             0.0.0.0:*        users:(("postmaster",pid=214485,fd=6))                                         
LISTEN   0         244                    [::]:5432                [::]:*        users:(("postmaster",pid=214485,fd=7)) 
```

Проверить вход и вывести список баз:

```bash
# psql -U postgres
psql (15.0)
Введите "help", чтобы получить справку.

postgres=# \l
...тут отобразится список баз...
```
Обязательно проверить вход не только из Linux, но извне от какого-то пользователя, который ранее заходил.

Если все хорошо работает, очистить Linux от старых пакетов и удалить старый каталог с данными. При выполнении pg_upgrade выше был создан файл `/var/lib/pgsql/delete_old_cluster.sh` , на самом деле там всего одна строка с удалением старого каталога с данными. Можно через скрипт, можно и так:

```bash
dnf remove postgresql14{,-server,-libs}

rm -rf /opt/pgsql/14
# безопаснее, чтобы не перепутать версии при вводе команды выше:
/var/lib/pgsql/delete_old_cluster.sh
```

Проверим, что остались только свежие пакеты 15 версии:

```bash
$ rpm -qa | grep postgres
postgresql15-server-15.0-1PGDG.rhel8.x86_64
postgresql15-libs-15.0-1PGDG.rhel8.x86_64
postgresql15-15.0-1PGDG.rhel8.x86_64
```

Дополнительные материалы:
- https://www.gnu.org/software/gawk/manual/html_node/Comparison-Operators.html
- http://www.dbaglobe.com/2021/10/upgrade-postgresql-13-to-postgresql-14.html
- https://postgrespro.ru/docs/postgresql/9.6/pgupgrade
- https://www.paulox.net/2022/04/28/upgrading-postgresql-from-version-13-to-14-on-ubuntu-22-04-jammy-jellyfish/
- https://docs.percona.com/postgresql/14/major-upgrade.html#on-debian-and-ubuntu-using-apt
