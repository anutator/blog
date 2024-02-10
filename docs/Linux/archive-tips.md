---
tags:
  - linux
title: Архивирование и дамп бд в скриптах
share: "true"
---
Ускоряем архивирование и снятие дампа базы данных в скриптах. Для проверки, насколько мы оптимизировали работу скрипта, подсчитываем время исполнения скрипта.
## Подсчитать время исполнения скрипта

```bash
date=$(date '+%Y-%m-%d_%H-%M-%S')  # дата в читабельном формате
start=$SECONDS  # для подсчета, сколько времени занимает бэкап
..... 
тут всякие команды, например архивирование и дампы
.....
end=$SECONDS
echo "Длительность бэкапа youtrack: $((end-start)) секунд."
```
## zstd — ускорение архивирования
Эффективно архивировать большие каталоги можно с помощью **zstd** — продвинутого архиватора от Facebook. Я тестировала — скорость реально намного выше стандартного gzip, сжимает лучше. Проверьте свою версию архиватора `tar --version`. Если она `>= 1.31`, то **zstd** уже поддерживается. Если нет (обычно в CentOS 7 старая версия 1.26), обновите из исходников:

```bash
$ tar --version
tar (GNU tar) 1.26
Copyright (C) 2011 Free Software Foundation, Inc....

# установка
cd
wget https://ftp.gnu.org/gnu/tar/tar-latest.tar.gz
tar xf tar-latest.tar.gz
cd tar-1.34/
./configure --prefix=/usr # предварительно установить Development Tools или хотя бы пакеты make и gcc
make
sudo make install
# Стереть каталог и скаченный архив
cd ..
rm -rf tar*

# проверка
$ tar --version
tar (GNU tar) 1.34
Copyright (C) 2021 Free Software Foundation, Inc.

# руководство
man tar
```

Сама `Pop!_OS` (Ubuntu) имеет также не самую свежую версию zstd. Обновим zstd из [исходников](https://github.com/facebook/zstd/releases):

```sh
# Стандартная версия из пакетов Pop!_OS, Ubuntu
$ zstd -V
*** zstd command line interface 64-bits v1.4.8, by Yann Collet ***

$ which zstd
/usr/local/bin/zstd

wget https://github.com/facebook/zstd/releases/download/v1.5.5/zstd-1.5.5.tar.gz
tar xf zstd-1.5.5.tar.gz
cd zstd-1.5.5/
make
sudo make install

$ zstd -V
*** Zstandard CLI (64-bit) v1.5.5, by Yann Collet ***
```

Теперь можно использовать zstd при архивировании добавляя флаг `--zstd`:

```bash
tar -C /sites --zstd -cf /backup/youtrack/youtrack_$date.tar.zst youtrack
```

Но ещё удобнее использовать флаг `-a` (`--auto-compress`) для автоматического определения типа архива по суффиксу, т.е. достаточно будет правильно назвать имя файла архива, для zstd это `.zst` или `.tzst`. Пример:

```bash
tar -C /sites -caf /backup/youtrack/youtrack_$date.tzst youtrack
```

Суффиксы (расширения файлов), распознаваемые архиватором **tar**:

| Суффикс         | Архиватор | Суффикс                 | Архиватор | Суффикс     | Архиватор |
| --------------- | --------- | ----------------------- | --------- | ----------- | --------- |
| .gz, .tgz, .taz | **gzip**      | .bz2, .tz2, .tbz2, .tbz | **bzip2**     | .lzma, .tlz | **lzma**      |
| .Z, .taZ        | **compress**  | .zst, .tzst             | **zstd**      | .lz         | **lzip**      |
| .lzo            | **lzop**      | .xz                     | **xz**        |             |           |

!!!note
    флаг `-С` не имеет отношения к типу сжатия, а указывает на переход в определенный каталог перед сжатием. Мне надо сжать каталог `/sites/youtrack`, но чтобы внутри архива был не полный путь, а только каталог youtrack, я заранее перехожу в каталог `/sites` и указываю, что архивировать буду папку youtrack.


### 7z — ещё один вариант архивирования
[Графический](https://axelstudios.github.io/7z/#!/) интерфейс 7z поможет подобрать команду для архивирования 7z архива. Например степерь сжатия Ultra (или 9), формат LZMA2 (подходит для видео), размер словаря 1024Mb, указывается имя архива, затем каталог или файл, который архивируем.

```sh
sudo apt install -y p7zip-full
7z a -mx9 -md1024m windows-server.7z windows-server
```

Лучшие опции:
- method : LZMA2
- dictionary size: 1536 Mo
- word size: 273  
- block : solid (This means only one block. As 7-zip will use one thread per block, this means the number of threads will be ignore and only one thread will be used)

```sh
7z a -mx9 -md1536m -mfb273 windows-server.7z windows-server
```

При тесте сжатия видеофайлов показал практически тот же результат по итоговому объему архива, что и zstd.

### Пример. Бэкап Youtrack и nginx
Бэкапы Youtrack хранятся в каталоге **/backup/youtrack** (каталог */backup* — монтированный NFS). Это полная копия каталога */sites/youtrack*, который используется docker контейнером Youtrack Jetbrains.

```bash title="/root/scripts/backup_youtrack.sh"
#!/bin/bash
#####################
## Бэкап Youtrack ###
#####################
date=$(date '+%Y-%m-%d_%H-%M-%S')  # дата в читабельном формате
start=$SECONDS   # для подсчета, сколько времени занимает бэкап

# чтоб заработал флаг --zstd, установить новую версию tar. Я поставила 1.32 из исходников.
tar -C /sites -caf /backup/youtrack/youtrack_$date.tzst youtrack

# Ротация старых бэкапов.
find /backup/youtrack/ -type f -mtime +5 -exec rm -rf {} +

end=$SECONDS
echo "Длительность бэкапа youtrack: $((end-start)) секунд."

##################
## Бэкап Nginx ###
##################
tar -C /etc -caf /backup/nginx/nginx_$date.tzst nginx
# Ротация старых бэкапов
find /backup/nginx/ -type f -mtime +6 -exec rm -rf {} +
```
## Массив в tar вместо списка каталогов
Когда-то делала интересный вариант архивирования для Asterisk, который практически нигде в Интернете не описан, но очень удобен, потому что лаконичен. Я уверена, что мало кто так делал архивирование. Обычно в команде tar пишут простыню.

Дополню: дамп SQL не всегда целесообразно делать через `mysqldump`. Если база очень большая, нужно использовать новый способ через `mysqlsh` с автоматическим архивированием в **zstd** архивы. Этот способ намного быстрее и не требует блокировки бд.

Сначала создается массив **astfiles**, где перечисляем в каждой строке папку или файл, который нужно архивировать. Это удобно, потому что ненужные компоненты всегда можно закомментировать решеткой в начале строки. При этом сам элемент массива **astfiles** может быть переменной. Потом архивируем вызывая этот массив. Часть из скрипта бэкапа:

```bash
bkdir="/backup"                                        # каталог бэкапа
timestamp=$(date +%Y%m%d_%H%M)                         # текущие дата и время в удобном формате
hostname=$(hostname -s)                                # имя хоста (удобно, когда собираем архивы с нескольких хостов)
astPath="$bkdir/astbackup-$hostname-$timestamp.tzst"   # полный путь архива бэкапа, сразу расширение для zstd архивирования.
sqlPath="$bkdir/sqldump-$timestamp.sql"                # полный путь к дампу sql

# Снимаем дамп базы (одной или нескольких) SQL со статистикой звонков
# mysqldump — более медленный вариант, mysqlsh — быстрее.
mysqldump asterisk > $sqlPath

# Перечисляем каталоги или отдельные файлы для бэкапа в массиве astfiles. Ненужное закомментировать
astfiles=(    
# /etc/dahdi
$sqlPath              # дамп базы данных MariaDB со статистикой по звонкам
/etc/asterisk         # файлы конфигурации Asterisk
/etc/zabbix           # файлы конфигурации Zabbix
/etc/profile.d        # каталог с алиасами Linux, скриптами, настройками редактора vim и пр.
/etc/yum.repos.d      # каталог со списком репозиториев
/var/www/html/acdr    # каталог веб-сервера, скрипт статистики звонков
... тут любые каталоги и папки, ненужное закомментировать.
)

### отобразим на экране весь список каталогов и файлов, которые бэкапим
echo 'Список файлов и каталогов бэкапа:'
echo ${astfiles[@]}

### Сжатие всех каталогов и файлов в один архив.
tar -caf $astPath ${astfiles[@]}

### Удаляем дамп базы CDR, т.к. он теперь есть в сжатом виде внутри общего архива.
# Работает, если даже несколько баз бэкапили, для этого ставим *
rm -f $bkdir/sqldump-*.sql
```

Далее у меня в файле описано копирование на сетевое хранилище NAS и удаление старых копий:
Удаляем бэкапы старше `keep_days`.
Удаляем старые бэкапы, количество которых превышает `keep_copies`.
Отображаем статус и конечные каталоги.
## mysqlsh — ускорение дампа базы данных MySQL 8.0.21+
В MySQL 8.0.21 появились утилиты:
- [`util.dumpInstance()`](https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-utilities-dump-instance-schema.html): полный дамп инстанса базы данных, включая пользователей
- [`util.dumpSchemas()`](https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-utilities-dump-instance-schema.html): дамп набора схем (баз данных).
- [`util.loadDump()`](https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-utilities-load-dump.html): заливка дампа на целевой сервер MySQL.
- [`util.dumpTables()`](https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-utilities-dump-instance-schema.html) : добавлено в **8.0.22**. Дамп конкретных таблиц или views схемы (базы).

Преимущества нового дампа:
- многопоточный дамп, разделяющий большие таблицы на маленькие порции данных, благодаря чему скорость существенно выше однопоточного **mysqdump** достигает 3Гб/с!
- параллельная загрузка порций данных при заливке дампа (как мы помним, импорт дампа занимал намного больше времени, чем создание дампа) в совокупности с возможностью выключения [InnoDB Redo Log в MySQL Server 8.0.21](https://dev.mysql.com/doc/refman/8.0/en/innodb-redo-log.html#innodb-disable-redo-logging) увеличивает скорость до 200Мб/с. Импорт дампа наконец использует возможности многоядерных процессоров.
- можно загружать дамп на другой хост одновременно со снятием дампа
- можно остановить и продолжить загрузку данных
- Сжатие уже встроено. **zstd** — продвинутый архиватор от Facebook, он сжимает отдельные файлы, gzip.
- отложенное создание вторичных индексов после загрузки данных
- дамп и загрузка непосредственно на/с OCI Object Storage
- режимы совместимости для [importing into OCI MySQL Database Service](https://docs.cloud.oracle.com/en-us/iaas/mysql-database/doc/importing-and-exporting-databases.html), что облегчает миграцию в облако.

Полный дамп всех баз через API. Предварительно в стандартной командной строке MySQL создать пользователя, который может делать бэкапы.

```sql
mysql> create user 'enterprise_dump'@'%' identified by 'ПАРОЛЬ';
mysql> grant backup_admin, select, lock tables, event, reload on *.* to 'enterprise_dump'@'%';
mysql> flush privileges;
```

Опционально можно использовать пользователя, который имеет привилегии **all** ко всем базам. Но на деле такой пользователь нужен, когда заливаем дамп обратно в базу, т.к. нужны права на запись.

Если подключаемся к стандартному порту 33060 (это дополнительный порт, не считая стандартного 3306), его можно не указывать в строке mysqlsh. Порт `33060/tcp` должен быть открыт в файрволе на том сервере, с которого делаем дамп (где база данных). Он не входит в стандартный сервис mysql, поэтому надо либо отредактировать порты сервиса mysql, либо дополнительно к открытию сервиса mysql открыть этот порт (надо проверять новые версии Linux, в них уже возможно добавили порт 33060 в сервис mysql):

```bash
firewall-cmd --add-port=33060/tcp --permanent
firewall-cmd --reload
```

Если же порт другой, надо указать его через двоеточие после доменного имени или ip-адреса. 
### Бэкап Еnterprise Architect. Пример полного дампа.
Дамп сохранится в каталог */backup/ea/dump_ea*, а затем дополнительно архивируется в 1 файл */backup/ea/ea_$date.tzst*. Представлен весь скрипт, но при ручном бэкапе достаточно выполнить строку, которая начинается с mysqlsh.

```bash title="dump_ea.sh"
#!/bin/bash
# Бэкап базы Enterprise Architect целиком (вся база данных).
date=$(date '+%Y-%m-%d_%H-%M-%S')  # дата в читабельном формате, специально её после дампа берем, чтоб понять, сколько времени занял дамп

# по умолчанию подключение к порту 33060
# Здесь пользователь с именем enterprise_dump. Пример создания пользователя с необходимыми правами выше.
# Особенность: дапм собирается в каталог, а не файл, zstd сжимает каждый файл дампа бд автоматически.
mysqlsh enterprise_dump@ip-адрес_сервера --password=ПАРОЛЬ -- util dump-instance /backup/ea/dump_ea

# чтобы на диске было меньше файлов и занималось меньше inode, дополнительно сжимаем и удаляем каталог c дампом бд
# опция -C /backup/ea позволяет перейти в каталог во время операции tar,
# чтобы внутри архива не сохранять полный путь к папке ea_dump
# tar -C /backup/ea -czf /backup/ea/ea_$date.tgz dump_ea && rm -rf /backup/ea/dump_ea
tar -C /backup/ea -caf /backup/ea/ea_$date.tzst dump_ea && rm -rf /backup/ea/dump_ea

# Ротация старых бэкапов. Очиста каталогов на текущем сервере и на nfs.
# поиск каталогов можно будет вскоре убрать
# find /backup/ea/ -mindepth 1 -maxdepth 1 -type d -ctime +10 -exec rm -rf {} +
find /backup/ea/ -type f -mtime +24 -exec rm -rf {} +
```

Для Enterprise Architect необходимо делать дампы почаще, поэтому храним побольше копий. Для работы расписания:

```bash
00 9,12,15,18 * * 1-5 root /root/scripts/dump_ea.sh
00 2 * * *  root /root/scripts/dump_ea.sh
```

Для заливки полного дампа нужен пользователь, который имеет полный доступ ко всем базам. Если это пользователь root, то для него нельзя вводить пароль в командной строке. Указывается путь к каталогу, а не файлу, как в случае с файлом дампа, полученного через mysqldump. Предварительно разархивировать нужный дамп.

```bash
tar xf /backup/ea/ea_дата.tszt
mysqlsh ПОЛЬЗОВАТЕЛЬ@ip-адрес_сервера --password=ПАРОЛЬ -- util load-dump /backup/ea/dump_ea
```

### Бекап Bookstack. Пример дампа конкретной базы (схемы)
В качестве примера рассмотрим бэкап [BookStack](https://www.bookstackapp.com/) (бесплатный софт для wiki) . Будем сохранять каждый бэкап BookStack в каталоге **/backup/wiki** в двух архивах:
- дамп базы **bookstackdb**, пример имени архива бэкапа `bookstack_db_2020-11-30_02-05-01.tgz`. Пользователь базы данных должен иметь права на бэкап (backup_admin, select, lock tables, event, reload).
- Каталог BookStack обычно находится внутри _/var/www/_, но если его там нет, посмотреть в настройках сайта _/etc/nginx/conf.d/bookstack.conf_ путь к нему). Пример имени архива бэкапа пример `bookstack_2020-11-30_02-05-01.tzst`. Что архивировать из этого каталога:
  - **.env** — файл, содержащий важную информацию с настройками (переменные).
  - **public/uploads** — каталог, в котором находятся все загруженные картинки (если только не используется внешнее хранилище amazon s3).
  - **storage/uploads** — каталог, содержащий Folder, скачанные вложения страниц (существует только начиная с версии v0.13).

Скрипт бэкапа */root/scripts/backup_wiki.sh*

```bash title="/root/scripts/backup_wiki.sh"
#!/bin/bash
######################
## Бэкап BookStack ###
######################
date=$(date '+%Y-%m-%d_%H-%M-%S')  # дата в читабельном формате
start=$SECONDS               # для подсчета, сколько времени занимает бэкап
wikidir=/var/www/BookStack   # рабочий каталог, внутри которого находятся папки и файлы BookStack
# Путь /backup/wiki также можно вынести в переменную.

# Бэкап базы bookstackdb. По умолчанию подключение к порту 33060
# Cтарый вариант через mysqldump в фвйл bookstack-db.sql
# mysqldump -uroot bookstackdb > bookstack-db.sql
# Новый вариант через mysqlsh: дапм собирается в каталог '/backup/wiki/bookstack', а не файл 'bookstack-db.sql'. zstd сжимает каждый файл дампа бд автоматически.
mysqlsh ПОЛЬЗОВАТЕЛЬ@localhost --password=ПАРОЛЬ -e 'util.dumpSchemas(["bookstackdb"], "/backup/wiki/bookstack")'

# чтобы на диске было меньше файлов и занималось меньше inode, дополнительно сжимаем gzip и удаляем каталог
# опция -C /backup/wiki позволяет перейти в каталог во время операции tar,
# чтобы внутри архива не сохранять полный путь к папке bookstack
tar -C /backup/wiki -caf /backup/wiki/bookstack_db_$date.tgz bookstack && rm -rf /backup/wiki/bookstack

# чтоб заработал флаг --zstd, установить новую версию tar. Я поставила 1.32 из исходников.
# Можно даже обойтись без флага --zstd, если у архивов расширение .zst или .tzst
# Тогда вместо -cf пишем -caf. Флаг -a (он же --auto-compress) использует суффикс архива для определения программы сжатия
# Бэкап каталогов Bookstack: public/uploads, storage/uploads и файла настроек .env
tar -C $wikidir -caf /backup/wiki/bookstack_$date.tzst public/uploads storage/uploads .env

# Ротация старых бэкапов.
#find /tmp/backup/dumpdb/s5 -mindepth 1 -maxdepth 1 -type d -ctime +60 | xargs rm -rf
#find /tmp/backup/dumpdb/s5 -mindepth 1 -maxdepth 1 -type d -ctime +10 -exec rm -rf {} +
find /backup/wiki/ -type f -mtime +10 -exec rm -rf {} +

end=$SECONDS
echo "Длительность бэкапа BookStack: $((end-start)) секунд."
```

**Восстановление данных из бэкапа**
Восстановим каталоги из архива, предварительно скопировав архив с другого сервера в рабочий каталог BookStack на новом сервере.

```bash
tar xf /backup/wiki/bookstack_2020-11-30_02-05-01.tzst -C /var/www/Bookstack
chown -R nginx. /var/www/BookStack
cd /var/www/BookStack
chmod -R 755 bootstrap/cache public/uploads storage
```

После выхода заливаем дамп старой базы
```bash
tar xf /backup/wiki/bookstack_db_2020-11-30_02-05-01.tgz

# старый вариант
# mysql -uroot -p bookstackdb < bookstack-db.sql
```

### Пример дампа конкретной таблицы внутри конкретной базы (схемы)
У нас внутри инстанса есть база данных **staging**, в ней таблица **source_records**. Сохраняем дамп таблицы **source_records** в каталог */backup/dumpdb/srec*. Для сохранения дампа пользователю не обязательно иметь права на запись.

```bash
# вариант 1
mysqlsh ПОЛЬЗОВАТЕЛЬ@ip-адрес_сервера --password=ПАРОЛЬ -e 'util.dumpTables("staging", ["source_records"], "/backup/dumpdb/srec")'

# вариант 2. Через API.
# не дописала ещё
```

Для снятия дампа используется целых три утилиты, а для заливки дампа только одна утилита `loadDump` или `load-dump`, если использовать API. Для заливки дампа нужен ПОЛЬЗОВАТЕЛЬ, который имеет доступ на запись. Ниже пример ошибки при попытке залить дамп с таблицей **source_records**, если эта таблица уже существует.

```bash
$ mysqlsh ПОЛЬЗОВАТЕЛЬ@localhost --password=ПАРОЛЬ -- util load-dump /backup/dumpdb/srec --schema=staging
WARNING: Using a password on the command line interface can be insecure.
Loading DDL and Data from '/backup/dumpdb/srec' using 4 threads.
Opening dump...
Target is MySQL 8.0.22. Dump was produced from MySQL 8.0.22               
Checking for pre-existing objects...
ERROR: Schema `cimp_staging` already contains a table named source_records
ERROR: One or more objects in the dump already exist in the destination database. You must either DROP these objects or exclude them from the load.
```

Если данные в таблице **source_records** уже есть, добавить аргумент `--ignoreExistingObjects=true` для игнорирования существующих объектов:

```bash
# вариант 1. Пример заливки дампа: залить всю базу staging за исключением таблиц source_records и source_records_old
mysqlsh ПОЛЬЗОВАТЕЛЬ@ip-адрес_сервера -e 'util.loadDump("/backup/dumpdb/s5/s5_dump", {excludeTables: ["staging.source_records","staging.source_records_old"], ignoreExistingObjects: true})'

# вариант 2. Через API
mysqlsh gpnsmonitor@localhost --password=ПАРОЛЬ -- util load-dump /backup/dumpdb/srec --schema=staging --ignoreExistingObjects=true
```

> `ignoreExistingObjects: [ true | false ]`
> 
> Import the dump even if it contains objects that already exist in the target schema in the MySQL instance. The default is `false`, meaning that an error is issued and the import stops when a duplicate object is found, unless the import is being resumed from a previous attempt using a progress state file, in which case the check is skipped. When this option is set to `true`, duplicate objects are reported but no error is generated and the import proceeds. This option should be used with caution, because the utility does not check whether the contents of the object in the target MySQL instance and in the dump files are different, so it is possible for the resulting import to contain incorrect or invalid data. An alternative strategy is to use the `excludeTables` option to exclude tables that you have already loaded where you have verified the object in the dump files is identical with the imported object in the target MySQL instance. The safest choice is to remove duplicate objects from the target MySQL instance before restarting the dump.

Продолжение следует.
