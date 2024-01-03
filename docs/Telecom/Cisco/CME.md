---
title: Руководство по CME
share: "true"
---

CME — Cisco Unified Communications Manager Express. Настройки собраны в один документ.
## Сайты
https://cciev.wordpress.com/ - тут много
http://benmorgan.com.au/blog/cme-shared-lines-quick-note/
http://www.unifiedguru.com/category/categories/cme/
https://networkexpertblog.wordpress.com/2011/10/10/registering-cisco-unified-9971-ip-phone-to-cme/
http://www.cciemagazine.in/free-learning-videos/sip-phone-registration-to-cucme/
http://www.cisco.com/c/en/us/support/docs/voice-unified-communications/sip-ip-phone-software/113048-reg-9971ip-cucm.html
https://voicetalks.wordpress.com/page/4/
http://abelorus.blogspot.ru/2013/05/cisco-call-manager-express-sip.html
http://silver-golem.livejournal.com/426023.html
blog.alakin.org
https://voiceonbits.com/category/cme/ — хорошие статьи!!!
http://admindoc.ru/1082/cucme-i-class-of-restrictions/

http://www.netcraftsmen.com/sip-endpoints-in-cisco-communications-manager-call-manager-express-x-lite/

http://www.ucguerrilla.com/ 
http://x-ccie.blogspot.ru          это продвинутые сайты

http://abelorus.blogspot.ru/2013/05/configuring-ip-cisco-sccp-sip.html#more

http://www.ianseno.com/
[здесь](http://ccievoice.ksiazek.be/) много ссылок на другие CCIE блоги
[пример лабораторки CCIE Voice](http://www.i-1.nl/blog/?p=176)

http://www.ccierants.com
http://chilli-net.blogspot.ru/
http://my-voice-journey.blogspot.ru/
cisco-notes.blogspot.ru

https://dreamforccie.wordpress.com
http://q931.blogspot.ru/
[блог](http://www.markholloway.com) CISCO CCIE Voice специалиста
http://www.voicecerts.com/2011/04/voice-translation-rules-examples.html
## Шаблоны телефонов
http://x-ccie.blogspot.ru/2012/09/cme-additional-features-part-2.html

```bash
show voice register template all        # статус свойства шаблонов SIP телефонов
show telephony-service ephone-template  # статус свойств шаблонов SIP телефонов
```

## Настройка SecureCRT и азы навигации по IOS
### Логирование сессий SecureCRT в текстовые файлы
Options→ Session Options→ раздел Log File
- **Log file name**: включает путь к файлу и имя самого файла, куда собирается лог
- **Upon connect**: текст внутри файла лога, когда открываем сессию.
- **Upon disconnect**: текст внутри файла лога когда отключаемся от сессии. 
- **On each line**: текст добавляемый в каждой строке файла лога.
- **Start log upon connect**: автоматически начинаем записывать данные в лог при открытии сессии (когда соединяемся с Cisco)
- **Append to file**: добавлять данные к файлу, overwrite file —перезаписывать файл.

Для заполнения полей используем переменные:
`%H` – hostname, ip-адрес соединения
`%S` – session name, текстовое имя соединения (сессии)
`%Y` – four-digit year, год в четырехзначном формате
`%M` – two-digit month, месяц в двухзначном формате
`%D` – two-digit day of the month , день
`%h` – two-digit hour, час в двухзначном 24-часовом формате
`%m` – two-digit minute, минут.
`%s` – two-digit seconds, секунды
`%t` – three-digit milliseconds, миллисекунды
`%%` – percent (%) 
`%envvar%` – environment variable, переменная (например имя пользователя  %USERNAME%).
При первом изменении спросит, применить ли ко всем сессиям или только к новым сессиям, которые будут созданы после внесения изменений.

Пример. Хотим хранить логи в общем каталоге C:\SecureCRT logs. Далее для каждой сессии у нас своя папка с её именем (имя сессии), внутри папки файл для каждой даты, файл содержит имя и ip-адрес в скобках, затем дата.

```bash
Log file name	C:\SecureCRT logs\%S\%S (%H) -- %Y-%M-%D.log
```

Если за день несколько раз заходили на маршрутизатор, то время начала и окончания сессий мы увидим в файле:

```
Custom log data:
Upon connect:  %S (%H) - %h:%m:%s
Upon disconnect: %S (%H) - %h:%m:%s
```

Пример. Хотим хранить файл в общем логе в каталоге C:\SecureCRT logs, при этом создаем папки не для каждой сессии, а по датам (год-месяц-день), а уже внутри папки с датой хранятся файлы сессий, причем каждая сессия хранится в отдельном файле, что достигается добавлением часа и минуты начала сессии:

```
C:\SecureCRT logs\%Y-%M-%D\%S (%H) -- %h-%m.log
```

Внутри сессии в начала точное время начала сессии, в конце время окончания сессии. Так же в начале каждой строки точное время вплоть до указания миллисекунд, причем добавляется справа символ “§”, чтобы при импорте в Excel можно было использовать его в качестве разделителя, другие символы типа запятой использовать нельзя, т.к. могут встречаться в самой конфигурации:

```
Custom log data:
Upon connect: %S (%H) - %h:%m:%s
Upon disconnect: %S (%H) - %h:%m:%s
On each line: %h:%m:%s.%t §
```
### Горячие клавиши Cisco IOS
Сокращать команды можно до тех символов, которые однозначно идентифицируют слово.
**Ctrl+T**:  поменять местами текущий символ с предыдущим
**Ctrl+D**: удалить текущий символ
**Ctrl+K**: удалить все символы начиная с текущего (где курсор) и до конца строки
**Ctrl+X** или **Ctrl+U**: удалить все символы с начала строки до текущего не включая его
**Ctrl+L**: перепечатать текущую строку
**Ctrl+A**: переметить курсор в начало строки
**Ctrl+E**: переместить курсор в конец строки
**Ctrl+F** или →: перейти на символ вправо 
**Ctrl+B** или ←: перейти на символ влево
**Ctrl+R**: перепечатать текущую строку в новой. Полезно, когда неожиданно вывод каких-то данных  появился как раз во время ввода команды.
**Ctrl+U**: стереть строку
**Ctrl+W**: стереть слово слева от курсора
**Esc+D**: стереть слово справа от курсора
**Ctrl+Z**: применить текущую команду и выйти из режима конфигурирования (configure terminal), возвращает нас в привилегированный EXEC режим
**Ctrl+C**: выйти из режима конфигурирования (почти аналог **Ctrl+Z**)
**Ctrl+Y**: вставить последний удаленный командами редактирования текст
**Ctrl+P** (или стрелка вверх): прокручивать  вперед предыдущие команды
**Ctrl+N** (или стрелка вниз): прокручивать назад предыдущие команды
**Ctrl+Shift+6** или **Ctrl+6+x** (называется escape sequence): отменить запущенный traceroute и вернуться в командную строку. Также остановка любых длительных процессов. Можно заменить на другую команду, например на `Ctrl+W:  terminal escape-character 23 ! 23 = Ctrl+W`
**Tab** или **Ctrl+I**: заканчивает частично введенную команду
**Esc, F**: передвижение на одно слово вперед (нажимать последовательно)
**Esc, B**: передвижение на одно слово назад
**Esc, C**: перевести текущий символ в верхний регистр `undebug all`
**Esc, U**: перевести слово справа от курсора в верхний регистр
**Esc, L**: перевести слово справа от курсора в нижний регистр

## Полезные команды Cisco
#### Загрузка оперативной памяти и процессора
```bash
show proc mem       # куча процессов и сколько каждый занимает памяти
show proc cpu sort  # загрузка процессора с сортировкой по убыванию
```
#### Перезагрузка Cisco
Если сконфигурировать и не сохранить настройки (wr или copy run sta), после перезагрузки вернутся прежние настройки.
Перегрузить маршрутизатор/ коммутатор:

```bash
reload
reload in 5     # перезагрузка через 5 минут, также можно 4:30 через 4 часа 30 мин
reload cancel   # отмена отложенной перезагрузки
```

Посмотреть сколько времени прошло с момента последней перезагрузки можно командой просмотра версии маршрутизатора:

```
OTK-MSK-c3925-1-VG#sh ver
Cisco IOS Software, C3900 Software (C3900-UNIVERSALK9_NPE-M), Version 15.4(3)M3, RELEASE SOFTWARE (fc2)
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 1986-2015 by Cisco Systems, Inc.
Compiled Fri 05-Jun-15 15:47 by prod_rel_team

ROM: System Bootstrap, Version 15.0(1r)M16, RELEASE SOFTWARE (fc1)

OTK-MSK-c3925-1-VG uptime is 22 weeks, 1 day, 18 hours, 8 minutes # (1)!
....
```
 
 1. 22 недели 1 день 18 часов и 8 минут маршрутизатор работает без перезагрузки
#### Сброс настроек маршрутизатора
```
write erase
reload
```

#### Просмотр конфигурации
При просмотре конфигурации `sh run` или других длинных выводов команд и появлении` ---More---` :
**Enter**  показывает следующую строку
**Пробел** показывает следующий экран
**/** позволяет ввести слово для поиска в` sh run` (например вводим `/tele` и попадаем в начало раздела телефонии)
**Любой другой символ** возвращает в основной режим EXEC.

Примечание: для вывода всей конфигурации без разделителей `---More---`  сделать так:

```bash
sh run | tee http:// 1.1.1.1    # якобы отправить конфигу на несуществующий адрес.
```

Для больших конфигураций, IOS12.2(25)+. Если в конфиг добавить:
`parser config cache interface`
то указанные команды будут выполняться быстрее:

```
copy system:running-configuration
show running-configuration
write terminal
```
#### Состояние интерфейсов, трафик на интерфейсе, кто больше всех грузит интерфейс, что качается в данный момент
Состояние интерфейсов, показывать только назначенные:

```bash
show ip int brief  | ex unass]
```

Можно сделать сокращение (псевдоним alias) для этой команды (в режиме конфигурирования):

```bash
alias exec ipconfig show ip interface brief | exclude unassigned
```

Теперь вводя `ipconfig` получим вывод состояния назначенных интерфейсов.

Для слежения за интерфейсом включить на физическом интерфейсе Netflow:

```bash
interface giga0/0
ip address 10.0.0.1 255.255.255.252
ip flow ingress
ip flow egress
```

Теперь можно смотреть что происходит на интерфейсе:

```bash
sh ip cache flow
sh ip cache flow | i 10.0.1  # (1)!
```

1.  то же самое с выборкой

Узнать кто создает максимальный трафик.

```bash
ip flow-top-talkers     # настроить предварительно
top 10
sort-by bytes
cache-timeout 100
sh ip flow top-talkers  # смотреть качающие хосты
```

Если есть прокси, смотреть на нем.

Сброс настроек интерфейса:

```bash
default interface fa0/0
```
#### Версия прошивки маршрутизатора
```txt title="show version"
OTK-MSK-c3925-1-VG#sh ver
Cisco IOS Software, C3900 Software (C3900-UNIVERSALK9_NPE-M), Version 15.4(3)M3, RELEASE SOFTWARE (fc2)
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 1986-2015 by Cisco Systems, Inc.
Compiled Fri 05-Jun-15 15:47 by prod_rel_team

ROM: System Bootstrap, Version 15.0(1r)M16, RELEASE SOFTWARE (fc1)

OTK-MSK-c3925-1-VG uptime is 20 weeks, 3 days, 20 hours, 46 minutes
System returned to ROM by reload at 21:59:03 MSK Mon May 16 2016
System restarted at 22:01:20 MSK Mon May 16 2016
System image file is "flash0:c3900-universalk9_npe-mz.SPA.154-3.M3.bin"
Last reload type: Normal Reload
Last reload reason: Reload Command
```
 
Обозначение версий и дерево функционала:
Внимание: версии 13 и 14 пропущены по «религиозным» причинам.

Цикл релизов:
- **EOS Notice** Уведомление о приближении EOS
- **End of Sale (EOS)** Окончание продажи
- **End of Engineering (EOE)** Окончание исправления ошибок
- **End of Life (EOL)** Окончание поддержки
#### Версия софта телефонии CUCME
Версия CME

```
sh telephony-service (кратко sh telephony)
OTK-MSK-c3925-1-VG#sh telephony
CONFIG (Version=10.5)
=====================
Version 10.5
Max phoneload sccp version 17
Max dspfarm sccp version 18
Cisco Unified Communications Manager Express
For on-line documentation please see:
```
#### Навигация по настройкам и статусам SCCP телефонов
Список всех зарегистрированных и незарегистрированных SCCP телефонов

```bash
sh ep reg su      # зарегистрированные
sh ep unreg su    # незарегистриорванные. Кратко sh ep un su.
sh ep sub r       # все прописанные телефоны, зарегистрированные и нет
```

Показать конфигурацию SCCP телефона по его dn: 

```bash
sh run | s ephone  15   # (1)!
```

1. обязательно 2 пробела перед 15

Данных немного, мак-адрес и dn телефона. Пример:

```bash
sh ep te 3984      # (1)!
DP tag: 0, primary
Tag 15, Normal or Intercom dn
  ephone 15, mac-address E8BA.70FB.AEF4, line 1
```

1. полный вид show ephone telephone-number ххх
#### Список файлов конфигурации для телефонов

```bash
show voice register tftp-binding      # Для SIP. Кратко: sh voice r tf
show telephony-service tftp-binding   # для SCCP. Кратко: sh telephony t
```

## Регистрация SIP телефона
http://anton.fly7.ru/
Подключал Gigaset a610 IP к Cisco 3925
Cложность заключалась в не очевидности того, что Username должен быть такой же как номер телефона. Потратил на это несколько дней… До того как прописал логин = номеру телефона, Gigaset мог звонить на внутренние номера и внешние, причем на внутренних номерах нормально определялся как 120й. Звонок на него не проходил.
В Gigsset прописать (веб-интерфейс) адрес Cisco (в трех местах), порт 5060 и логин/пароль (120/120 в моем случае)
В Cisco вот такой код

```
voice register dn 20
 number 120
 allow watch
 no-reg
!
voice register pool 20
 id mac 7C2F.805D.xxxx
 number 1 dn 20
 template 1
 voice-class codec 1
 username 120 password 120
 no vad
!
```

несколько команд, которые мне подсказали )

```
debug ccsip all
debug voice register
ter mon
undebug all
show dial-peer voice summary
```

Даже команда reset перегружает SIP телефон не во время разговора, а сразу по окончании (в Avaya предварительно смотрим статус телефона, чтоб не прервать разговор, только в Utility Server при прошивке  есть варианты).

### Навигация по настройкам и статусам SIP телефонов
Показать конфигурацию SIP телефона по его dn:

```bash
sh run | s voice register pool  14    # обязательно два пробела перед 14 и писать полностью
```

Статус  SIP телефона по его dn. Выводится много данных: версия телефона, статус, настройки, автоматически созданный для него dial-peer и пр.

```bash
sh voice r pool 14    # можно с одним пробелом
```

Проверить, зарегистрирован ли  SIP телефон, по его номеру:

```
sh voice r pool tel <номер телефона>
```

Здесь данных немного, пример:

```
OTK-MSK-c3925-1-VG#sh voice reg pool tel 6855
Pool ID              IP Address      Ln DN  Number               State
==== =============== =============== == === ==================== ============
14   006C.BCA9.C42C  10.174.84.163   1  14  6855$                REGISTERED
```


Статусы всех SIP телефонов: `show voice register pool all`

```
after-hour-exempt        Show details of the phone with after hours exempt
all                      Show info of all pools
attempted-registrations  Show details of the pools that have attempted to register
cfa                      Show details of pool with call-forward-all set
connected                Show details of the phone that are currently in conversation
dnd                      Show details of pool with DND set
ip                       Show details of the phone with IP address
mac                      Show details of the pool with MAC id
on-hold                  Show details of the pool that are currently on hold
phone-load               Show the phone load details of the phones
registered               Show details of the pools that are registered
remote                   Show details of the remote phones (with no arp entry)
ringing                  Show details of the phones that are currently ringing
telephone-number         Show details of the pools with the telephone-number
type                     Show description of pools of a phone type and properties if it is FastTrack phone
unregistered             Show details of the pools that are unregistered
```

```bash
show voice register dial-peers    # (кратко  sh voice r dial-) показать SIP номера
show voice register statistics    # статистика по регистрациям и разрегистрациям SIP телефонов
sh voice register all             # конфигурационная инфа SIP телефонов
sh voiceregister pool phone-load  # текущие и прошлые версии прошивок телефона
sh run | inc voice register dn |name  # списком dn sip телефонов и фамилии
```

### Замена прошивки SIP телефонов 78хх
Текущая прошивка SIP телефонов 10.3.1. Архив прошивки состоит из 7 файлов. 

```
28    39321936 Jul 14 2016 11:53:34 +03:00 phone/78xx/rootfs78xx.10-3-1-12.sbn
29      596536 May 13 2016 13:11:50 +03:00 phone/78xx/sboot2.78xx.10-3-1-12.sbn
30     2511358 May 13 2016 13:11:26 +03:00 phone/78xx/kern2.78xx.10-3-1-12.sbn
31     3092888 May 13 2016 13:41:24 +03:00 phone/78xx/kern78xx.10-3-1-12.sbn
32      370348 May 13 2016 13:41:40 +03:00 phone/78xx/sboot78xx.10-3-1-12.sbn
33    41419092 Jul 14 2016 12:02:46 +03:00 phone/78xx/rootfs2.78xx.10-3-1-12.sbn
34        1206 May 12 2016 11:01:04 +03:00 phone/78xx/sip78xx.10-3-1-12.loads
```

Скачана прошивка `11.5.1 cmterm-78xx.11-5-1-18.zip`, в архиве так же 7 файлов.

Файл `sip78xx.11-5-1-18.loads` должен быть указан для типа телефонов (7821, 7841 и т.д.) в глобальных настройках SIP телефонии `voice register global`.

### Регистрация нового типа ip телефона, если он еще не поддерживается, как старого типа.

```
voice register pool-type 7841
  description Cisco IP Phone 7841
  reference-pooltype 6941
```

## Коды доступа к функциям (Feature Access Code)
Коды функций SCCP и аналоговых телефонов

```
telephony-service
  fac {standard | custom {alias alias-tag custom-fac to existing-fac [extra-digits]} | feature custom-fac}}
```

```bash
telephony-service
  fac standard        # по умолчанию коды функций выключены, включаем стандартные
telephony-service
  no fac standard     # выключаем стандартные коды функций
```

Если задано fac standard, то все коды функций CME по умолчанию отобразятся здесь:

```bash
OTK-MSK-c3925-1-VG#show telephony-service fac
  telephony-service fac standard
    callfwd all **1     # переадресация всех вызовов
    callfwd cancel **2  # отмена переадресации
    pickup local **3
    pickup group **4
    pickup direct **5
    park **6
    dnd **7             # режим «не беспокоить»
    redial **8
    voicemail **9
    ephone-hunt join *3
    ephone-hunt cancel #3
    ephone-hunt hlog *4
    ephone-hunt hlog-phone *5
    trnsfvm *6
    dpark-retrieval *0
    cancel call waiting *1
    ephone-hunt unjoin #4
```

Пример. Изменим стандартный код для переадресации всех вызовов `**1` на свой `#45`

```
telephony-service
  fac custom callfwd all #45
```

Параметр alias позволяет создать альтернативный код доступа функции вдобавок к существующему или добавить к коду набор еще каких-то цифр. Таких альтернативных котом можно создавать до 10 (от 0 до 9) для одного кода функции. Например, создадим короткий набор `#44`, позволяющий сразу делать переадресацию всех вызовов на `1111`, чтоб не надо было вводить `#451111`

```
Telephony-service
  fac custom callfwd all #45
  fac custom alias 0 #44 to #451111
  fac alias0 code has been configurated to #44!
  alias0 map code has been configurated to #451111!
```

Пример. Создадим короткие наборы alias для трех групп перехвата вызова. 

```bash
telephony-service
 fac custom pickup group **4      # код для перехвата групп (надо еще набирать номер группы)
 fac custom alias 1 #1 to **4121  # перехват группы 121
 fac custom alias 2 #2 to **4122  # перехват группы 122
 fac custom alias 4 #4 to **4124  # перехват группы 124
```

## Соответствие версий IOS и CUME. Максимальное количество абонентов CUCME для разных маршрутизаторов.
Support →Product Support →Unified Communications→Cisco Unified Communications Manager Express → Compatibility Information
Например для версии [10.5](http://www.cisco.com/c/en/us/td/docs/voice_ip_comm/cucme/requirements/guide/cme105spc.html)
## Время и настройка часовых поясов

Показать текущее время и дату на маршрутизаторе:

```
show clock
```

Для SCCP телефонов:

```bash
telephone-service
 time-zone 32         # Россия или 31 Саудовская Аравия, у них одинаково +180)
 time-format 24       # время в 24-часовом формате
 date-format dd-mm-yy # день-месяц-год (для года только вариант последние 2 цифры)
```
 
Для SIP телефонов:

```
voice register global
 timezone 32
 time-format 24
 date-format D/M/Y
```

## Dial-Peers
Виды dial-peer:
- **POTS (Plain old telephone service)** — для маршрутизации на аналоговые и SCCP телефоны, городские CO линии FXO, цифровой поток ISDN PRI используется команда port. Только в POTS для destination-pattern действует автоматическое обрезание цифр , которые определены точно (не масками). Для телефонов SCCP (Skinny) dial-peer создаются автоматически при прописывании телефона ephone.
- **VoIP dial peer** — использование команды session target с ip-адресом удаленной станции или такого же маршрутизатора. По умолчанию ничего не обрезается в destination-pattern. Кодек по умолчанию G729.
- **Multimedia Mail overlap (MMolP)** — третий вид dial-peer, маршрутизируется на адрес электронной почты SMTP сервера. Используется для хранения и переадресации факсов (on-ramp и off-ramp faxing).
### Маски (wildcards) для множества номеров в destination-pattern
As always, feel free to correct anything that needs correcting or add anything that needs adding. There is a lot more to the full definition of wildcards, but these are the basics. Note to Linux guys: This isn't regex as you understand it. Yes, the use of curly braces would be nice, but we don't get that here.

**T** — любой диапазон любых цифр от 0 до 32 цифр максимум. Номера с T имеют наименьший приоритет, даже если у них указано большее количество первых цифр в паттерне: например  если описан `98.......` и `98011Т`, то набор номера `9801112345` отправится туда, где паттерн `98........`

```bash
destination-patter 9T   # 9, за которой может не быть ничего или любое кол-во цифр до 32 включительно. После набора цифр будет ожидание таймера interdigit timeout (T302 timer). Для ускорения набора можно нажать #. Но правильнее более подробно описать правила исходящей маршрутизации на разные направления (город, межгород, мобильные, международные).

destination-pattern .T  # набор любого количества любых цифр. Указываем впереди точку, чтобы для срабатывания правила была введена хотя бы одна цифра. В противном случае правило сработает даже если пользователь просто поднял трубку и долгое время ничего не набирал.
```

Точка (Period) — одна точка соответствует одной цифре или *.

```bash
destination-pattern 3...          # четырехзначный номер, начинающийся на 3.
destination-pattern 91802.......  # 12-значный номер (5 точных цифр и 6 любых), начинающийся на 91802
```

Знак плюс `+`  — повтор от 1 до 32 раз предыдущей цифры или паттерна из нескольких цифр. В начале строки знак + обозначает номер в формате E164.

```bash
destination-pattern 85+   # 85, 855, 8555, 85555 и т.д. до 32 пятерок
destination-pattern 1+    # 1, 11, 111, 1111 и т.д. до 32 единиц.
destination-pattern 5+23  # cоответствует 5523, 55523, 555523 и т.д.
```

Процент `%` или знак вопроса `?` — повтор от 0 до 32 раз предыдущей цифры или паттерна из нескольких цифр

```bash
destination-pattern 74%   # цифра 7, за которой следует цифра 4 от 0 од 32 раз, т.е. 7, 74, 744, 7444, 7444 и т.д.
destination-pattern 84?
```

Квадратные скобки `[brackets]` — одна цифра из указанного диапазона

```bash
destination-pattern [2-4]...  # четырехзначный номер начинается на 2, 3 или 4. Остальные три цифры номера любые.
destination-pattern [159]...  # четырехзначный номер начинается на 1, 5 или 9.
destination pattern [04-6]... # четырехзначный номер начинается на 0, 4, 5 или 6.
destination-pattern 555[25]   # два номера 5552 и 5555.
destination pattern 1[1–3]........[68]  # номер состоит из 11 цифр, первая цифра 1, вторая цифра 1, 2 или 3, последняя цифра 6 или 8, остальные — любые
```

Круглые скобки `(parenthesis)` — группа цифр (паттерн), которая будет использована совместно с  +, ? или %.

```bash
destination-pattern (61)+   # сочетание цифр 61, которое может повторяться от 1 до 32 раз, т.е. 61, 6161, 616161 и т.д.
destination-pattern(555)+   # 555, 555555, 555555555 и т.д. до 32 раз повторения 555.
```

Запятая (Comma) — добавляет паузу 1 секунда между набранными цифрами.
Символ `#` не используется для масок. Нажатие решетки обозначает окончание набора номера без ожидания набора следующей цифры.
### Манипуляция АОНами или набранными цифрами в translation-profile
Для более тонких преобразований можно использовать Translation Profiles. Также с помощью них можно сделать все что делается через forward-digits или prefix.
Создание Translation Profiles включает три этапа:
1. Определяем Translation Rule, состоящий из одного или нескольких (до 15) правил.
2. Ассоциируем Translation Rule с translation profile. Здесь определяем, к чему применится правило: called — изменение набранного номера, calling — изменение АОНа.
3. Добавляем translation profile к dial-peer

Синтаксис Translation Rule:

```bash
Router(config)# voice translation-rule номер_правила
Router(cfg-translation-rule)# rule 1 /match-pattern/ /replace-pattern/
Router(cfg-translation-rule)# rule 2 /match-pattern/ /replace-pattern/
Router(cfg-translation-rule)# rule 3 /match-pattern/ /replace-pattern/ ...и т.д.
```

Мы можем предварительно тестировать Translation Rule командой:

```
test voice translation-rule номер_правила набранный_номер_или_аон
```

`*`	— Цифры (цифра) встречается 0 или более раз.
`+`	— Цифры (цифра) встречаются 1 или более раз
`?`	— Цифры встречаются 0 или 1 раз.
`.`	— Любая цифра
`[0-9]`	— Диапазон цифр
`.*`	— Любая цифра, встречающаяся 0 или более раз. Правило работает всегда, включая случаи, когда нет набора.
`.+`	— Любая цифра, встречающаяся 0 или более раз.  Правило работает при любой цифровой комбинации кроме отсутствия набора.
`^$`	— Сочетание знака начала и окончания ввода дает комбинацию: отсутствие набора.
`/___ /` —	Все что заключено в правый слеш обозначает номер целиком — или исходный номер (match-pattern), или итоговый номер после модификации (replace pattern)
`^`	— Если указана в начале, обозначает четко первую или первые цифры номера.
`^4` — номера, начинающиеся на 4 (разной длины), ^455 — номера начинаются на 455. Если кроме этого ничего не указано для исходного паттерна, то в итоговом заменены будут только указанные цифры. Пример:
`rule 1 /^4/  /823/` —   Если исходный номер 4520, то итоговый будет 82320
Если для исходного номера указано его полное описание ( например ниже пятизначный номер, начинающийся на 40), то заменяться будет номер целиком.
`rule 1 /^40.../ /6666000/` —  Если исходный 40123, то итоговый 6666000 (от исходного ничего не осталось)
`\`	— В исходном match-pattern обозначает места, где номер будет разделен на составные части. После обратного слеша следует либо точная последовательность цифр, либо множество, определенное с помощью масок
В итоговом replacement-pattern обозначает вставляемую часть из первого. После обратного слеша следует порядковый номер этого кусочка из исходного паттерна. Пример `\1`
`()` —	Круглые скобки выделяют те составные части, которые будут сохранены для вставки в итоговый паттерн. То, что без скобок — использоваться будет только для срабатывания правила.
`(a\)` — сохраняем выражение а
`b\`  — не сохраняем выражение b, т.е. b должно присутствовать в исходном номере, но в итоговом номере его не будет.

http://www.cisco.com/c/en/us/support/docs/voice/call-routing-dial-plans/61083-voice-transla-rules.html
http://www.dasblinkenlichten.com/destination-patterns-the-basics/
#### Примеры без использования масок
**Пример 1.** Замена первого совпадения паттерна на другой паттерн. Номер может начинаться с любой цифры, не обязательно с указанного паттерна.

```bash
voice translation-rule 1
rule 1 /123/ /890/                            # как только встретится 123, заменяем на 890

router#test voice translation-rule 1 123      # тестируем правило
Matched with rule 1
Original number: 123 Translated number: 890

router#test voice translation-rule 1 1234
Matched with rule 1
Original number: 1234 Translated number: 8904

router#test voice translation-rule 1 1123
Matched with rule 1
Original number: 1123 Translated number: 1890

router#test voice translation-rule 1 1123123
Matched with rule 1
Original number: 1123123 Translated number: 1890123
```

**Пример 2.** Правило применится только если исходный паттерн для сравнения будет находиться в начале номера (АОН звонящего или набранные цифры). Для этого правила используется символ  `^` в исходном паттерне.

```bash
voice translation-rule 1
rule 1 /^123/ /890/                             # если номер начинается на 123, заменяем 123 на 890

router#test voice translation-rule 1 123 
Matched with rule 1
Original number: 123 Translated number: 890

router#test voice translation-rule 1 1234
Matched with rule 1
Original number: 1234 Translated number: 8904

router#test voice translation-rule 1 1234123
Matched with rule 1
Original number: 1234123 Translated number: 8904123 

router#test voice translation-rule 1 1123
No match. "123" does not occur at beginning.     # правило не сработает, т.к. 123 не в начале номера
```

**Пример 3**. Исходный паттерн будет заменен только если он находится в конце номера (знак "$")

```bash
voice translation-rule 1
rule 1 /123$/ /890/                                      # если в конце номера 123, то заменяем их на 890

router#test voice translation-rule 1 123 
Matched with rule 1
Original number: 123 Translated number: 890

router#test voice translation-rule 1 1234
No match. "123" does not occur at the end of the number.  # 123 в начале, а не конце

router#test voice translation-rule 1 1234123
Matched with rule 1
Original number: 1234123 Translated number: 1234890       # в номере два совпадения, но заменится последнее

router#test voice translation-rule 1 1123
Matched with rule 1
Original number: 1123 Translated number: 1890 
```

Пример 4. Паттерн будет заменен только при полном совпадении с исходным паттерном, что достигается символом "^" в начале и "$" в конце.

```bash
voice translation-rule 1
rule 1 /^123$/ /890/                    # только при наборе 123 мы отправим 890. Всего одно совпадение.

router#test voice translation-rule 1 123 
Matched with rule 1
Original number: 123 Translated number: 890               # правило сработало!

router#test voice translation-rule 1 1234
No match. "123" does not occur at the end of the number.  # 123 не в конце

router#test voice translation-rule 1 1123
No match. "123" does not occur at the beginning.          # 123 не в начале
```

#### Примеры с использованием масок
**Пример 1**. Заменяем любой пятизначный номер, начинающийся на 88, на номер 9999000. Replace any number that is five digits in length that begins with "88" with the number "9999000".

```bash
voice translation-rule 1
rule 1 /^88.../ /9999000/

router#test voice translation-rule 1 88222
Matched with rule 1
Original number: 88222 Translated number: 9999000   # номер 88222 заменен на 9999000

router#test voice translation-rule 1 89123
No Match. The number does not begin with 88.    # номер не начинается на 88, правило не сработает

router#test voice translation-rule 1 881234
No Match. The number contains too many digits.  # номер содержит больше 5 цифр (хоть и начинается на 88), правило не сработает

router#test voice translation-rule 1 8812
No Match. The number contains too few digits.   # номер содержит меньше 5 цифр (хоть и начинается на 88), правило не сработает.
```

**Пример 2**. Заменяем любой номер на `9995000`. Точка со звездочкой — это маска, включающая в себя все цифры, включая даже отсутствие номера (например пустой АОН или поднятие трубки с ожиданием).

```
voice translation-rule 2
rule 1 /.*/ /9995000/

router#test voice translation-rule 2 123
Matched with rule 1
Original number: 123 Translated number: 9995000

router#test voice translation-rule 2 86573
Matched with rule 1
Original number: 86573 Translated number: 9995000

router#test voice translation-rule 2 ""
Matched with rule 1
Original number: <NULL> Translated number: 9995000
```

**Пример 3**. Заменяем любой номер, кроме отсутствия номера, на 9950000.

```bash
voice translation-rule 1
rule 1 /.+/ /9995000/

router#test voice translation-rule 1 89555
Matched with rule 1
Original number: 89555 Translated number: 9995000 

router#test voice translation-rule 1 212
Matched with rule 1
Original number: 212 Translated number: 9995000

router#test voice translation-rule 1 ""
No match. <NULL> is not included in match pattern.    # (1)!
```

1. только в этом случае не сработает

**Пример 4**. Заменяем любой номер, начинающийся на `8` или несколько восьмерок, тремя десятками вместо восьмерок.

```bash
voice translation-rule 5
rule 1 /^8+/ /999/

router#test voice translation-rule 5 85551212
Matched with rule 1
Original number: 85551212 Translated number: 9995551212

router#test voice translation-rule 5 885551212
Matched with rule 1
Original number: 885551212 Translated number: 9995551212

router#test voice translation-rule 5 8885551212
Matched with rule 1
Original number: 888123456 Translated number: 9995551212

router#test voice translation-rule 5 5551212 
No match. Number does not begin with eight.     # (1)!
```

1. правило не работает, т.к. начинается на 5

**Пример 5**. Любой номер заменяется на пустой, включая пустой. Маска `^.*` включает любые цифры, включая их отсутствие. Если же нужно применить правило только к пустому номеру (например хотим заменить пустой АОН на какой-то другой), используем маску `^$` (т.е. обозначены символы начала и конца, между которыми ничего нет).

```
voice translation-rule 1
rule 1 /^.*/ / /

router#test voice translation-rule 1 2001
Matched with rule 1
Original number: 2001 Translated number: <NULL>

router#test voice translation-rule 1 5551212
Matched with rule 1
Original number: 5551212 Translated number: <NULL>

router#test voice translation-rule 1 
Matched with rule 1
Original number: <NULL> Translated number: <NULL>
```

**Пример 6**. У всех набранных номеров (called) убираем префикс выхода на город `9`. Все АОНы (calling) должны быть заменены на `3467361869`.

```bash
voice translation-rule 9        # будущее правило для набираемых номеров
 rule 1 /^9/ //

voice translation-rule 361869   # будущее правило для АОНоа
 rule 1 /.*/ /3467361869/
```

Ассоциируем Translation Rule с translation profile

```bash
voice translation-profile kmg-pbx
 translate calling 361869           # правило для АОНов
 translate called 9                 # правило для набранных номеров.
```
 
Добавляем translation-profile к dial-peer:

```bash
dial-peer voice 361869 voip
 corlist outgoing call-pbx
 description kmg -- UgrTel PBX-style calls
translation-profile outgoing kmg-pbx        # (1)!
 preference 3
 destination-pattern 9T                     # (2)!
 session protocol sipv2
 session target sip-server
 voice-class codec 1
 voice-class sip dtmf-relay force rtp-nte
 dtmf-relay rtp-nte
 no vad
```

1. добавили профиль
2. для звонков начинающихся на 9
#### Сложные правила с разбиванием номера на несколько составных частей
Техника number slicing — разрезание номера на несколько составных частей, которые потом можно использовать в итоговом номере.
Пример 1.
```bash
/ (x\) y\ (z\) / / a \1 \2 / 
```

Разделяем  анализируемый номер на три составные части: x, y и z. Знак обратного слэша `\` указывает, где именно разрезать номер. Круглые скобки выделяют только те составные части, которые будут использованы не только для сравнения, но и как части итогового после замены. Буква a обозначает дополнительные цифры (префикс, суффикс) в полученном номер represents.

Set 1 inherits the value of x.
Set 2 inherits the value of z.
Expression y is not reused.

Итоговый номер после применения правила (concatenated number): `axz`

В примере номер `55866` заменим на `85566`. Символы `^` и `$` использованы, чтобы обозначить, что паттерны должны располагаться в начале и конце номера (`55` в начале и `66` в конце). При этом самым точным совпадением будет номер 55866, но может быть и номер большей длины, содержащий в начале две пятерки, в любом месте в середине восьмеру, а в конце две шестерки.

```
voice translation-rule 1
rule 1 /^\(55\)8\(66\)$/ /8\1\2/
```

Set 1: 55 — используется в итоговом номере как первая составная часть
Set 2: 66 — используется в итоговом номере как вторая составная часть

Игнорировать: 8, т.к. используется только для сравнения, чтобы правило сработало, но не в итоговом номере.

```bash
router#test voice translation-rule 1 55866
Matched with rule 1
Original number: 55866 Translated number: 85566

router#test voice translation-rule 1 12345
No match. The match pattern is not satisfied. # правило не сработало

router#test voice translation-rule 1 55867
No match. The match pattern is not satisfied. # правило не сработало, т.к. номер заканчивается не на 66, а на 67.

router#test voice translation-rule 1 55466
No match. The match pattern is not satisfied. # правило не сработало, т.к. в середине номера нет восьмерки.
```

#### Обрезаем или добавляем цифры в номере (частный случай составных частей как аналог forward-digits и prefix в dial-peer)
**Пример 1**. Отрезаем все цифры в номере, кроме четырех последних. Обязательно все цифры, которые оставляем, будут выделены круглыми скобками, как в случае использования составных частей.

```
rule 1 /^.*\(....\)/ /\1/

router#test voice translation-rule 1 5551212
Matched with rule 1
Original number: 5551212 Translated number: 1212

router#test voice translation-rule 1 2125551212
Matched with rule 1
Original number: 2125551212 Translated number: 1212
```

**Пример 2**. Замена called вида `[1-5]..` на `70[1-5]..`  в translation-rule.

```bash
voice translation-rule 333
rule 1 /\([1-5]..\)/ /70\1/    # (1)!
!
voice translation-profile short_calls  # (2)!
translate called 333       # (3)!
!
dial-peer voice 3001 voip
description Short calls
translation-profile outgoing short_calls  # (4)!
destination-pattern [1-5]..
session target ipv4:172.16.2.11
dtmf-relay h245-alphanumeric
codec g711ulaw
no vad
```

1. весь номер заключен в скобки и используется как составная часть для того, чтобы в итоговом номере был префикс 70 плюс исходный номер
2. название правила пишем произвольно
3. правило 333 применяется к набранным номерам (если бы было calling, то применялось бы к АОНам)
4. номер оправляется на правило преобразования

То же самое даже проще сделать с помощью prefix в самом dial-peer.

```
dial-peer voice 3001 voip
description Short calls
destination-pattern [1-5]..
prefix 70
session target ipv4:172.16.2.11
dtmf-relay h245-alphanumeric
codec g711ulaw
no vad
```
#### Изменение типа набора и номерного плана
**Пример 1**. Если номер начинается на 4 и его тип "national",  добавим впереди префикс "99"x. Если набран номер начинающийся с 4 и его тип "international", добавим впереди префикс "999". Это пример сценария, когда оператор связи требует код доступа при междугородных и международных звонках.

```bash
voice translation-rule 1
rule 1 /^4/ /994/ type national national
rule 2 /^4/ /9994/ type international international

router#test voice translation-rule 1 442195555 type national
Matched with rule 1
Original number: 442195555 Translated number: 99442195555
Original number type: national Translated number type: national
Original number plan: none Translated number plan: none

router#test voice translation-rule 1 442195555 type international
Matched with rule 2
Original number: 442195555 Translated number: 999442195555
Original number type: international Translated number type: international
Original number plan: none Translated number plan: none

router#test voice translation-rule 1 842195555 type international
No Match. Does not begin with "4".  # (1)!
```

1. правило не применяется, т.к. номер не начинается на 4.

**Пример 2**.  В любом  четырехзначном номере, начинающемся на `8`, сама первая цифра `8` будет заменена на префикс `04421955`, тип будет установлен "national", а план `isdn`.

```bash
voice translation-rule 1 
rule 1 /^8\(...$\)/ /04421955\1/ type unknown national plan unknown isdn

router#test voice translation-rule 1 8150 type unknown plan unknown
Matched with rule 1
Original number: 8150 Translated number: 04421955150
Original number type: unknown Translated number type: national
Original number plan: unknown Translated number plan: isdn

router#test voice translation-rule 1 6150 type unknown plan unknown
No Match. Does not begin with 8.      # (1)!

router#test voice translation-rule 1 81501 type unknown plan unknown
No Match. Does not contain the correct number of digits (4).
```

1. первая цифра не 8, правило не сработает
#### Черный список
Пример. Отбиваем все вызовы, начинающиеся на 900.

```bash
rule 1 reject /^900/

router#test voice translation-rule 10 8005551212
No Match. Does not begin with 900.    # совпадений нет, номер не начинается на 900

router#test voice translation-rule 10 9005551212
Call blocked by rule 1                # номер блокируется первым правилом
```

## Флеш-память, прошивки и конфигурации телефонов
Посмотреть содержимое флеш-памяти: `show flash`
### SCP для копирования файлов на флеш-память из GUI (графического интерфейса)
http://www.ccierants.com/2011/06/great-way-to-copy-files-on-cisco.html
http://blog.prorouting.com/2013/12/easy-transfer-of-files-tofrom-cisco.html

Проверка показала, что WinSCP не работает с марштрутизаторами Cisco (а также ASA) по SCP из-за того что поддерживает только `bash/ksh: Error message "Error skipping startup message. Your shell is probably incompatible with the application (Bash is recommended)"`
- Для ASA: ASDM (Cisco Adaptive Security Device Manager) →Tools→File Management
- Графический интерфейс в Windows: программа SecureFX→ SCP. Лучше использовать программу«два в одном»  SecureCRT + SecureFX Bundle и для командной строки Cisco, и для закачки файлов.
- Командная строка Windows: утилита для Putty – pscp.exe
- SCP в MAC/Linux уже доступен из командной строки по умолчанию

```bash
ip scp server enable    # Включение SCP на маршрутизаторах Cisco
feature scp-server      # Включение SCP на Cisco Nexus OS
```

Должна быть настроена AAA аутентификация и авторизация (возможно уже есть):

```bash
aaa new-model
aaa authentication login default local
aaa authorization exec default local
aaa authentication attempts login 3
username user privilege 15 password 0 securepass # (1)!
```

1. логин с привилегией 15. Можно не настраивать, если используется авторизация TACACS Узнать можно sh run | i tacacs

Вариант, если авторизация через tacacs:

```
aaa group server tacacs+ CiscoACS
aaa authentication login default group CiscoACS local
aaa authorization exec default group CiscoACS local
aaa authorization commands 15 default group CiscoACS local
```

Должен быть включен и настроен ssh:

```bash
ip ssh version 2
line vty 0 15                     # или 0 4  (зависит от своих настроек)
transport input telnet ssh
ip ssh time-out 120               # опционально
ip ssh authentication-retries 3   # опционально
exec-timeout 800 0                # опционально
```

При попытке открыть удаленный хост по ssh/telnet, а он не доступен. Тогда приходится ждать 30 секунд и никакие **ctrl+shift+6** не спасают. Спастись можно если добавить в конфиг предварительно:

```bash
ip tcp synwait-time 5   # (1)!
```

1. уменьшить время ожидания до 5

На маршрутизаторе должно быть задано доменное имя и hostname для генерирования ключа RSA, необходимого для SSH:

```bash
ip domain-name company.com
hostname имя_маршрутизатора
crypto key zeroize rsa      # (1)!
crypto key generate rsa general-keys modulus 2048 # (2)!
```

1. опционально. Удалить все существующие ключи, если были.
2. ждем генерации ключа

Пример подключения клиента из Linux. Скопируем образ IOS (файл `.bin`) на флеш-памать маршрутизатора с адресом. Опция -2 для использования SCP версии 2.

```bash
scp -2 ./c2800nm-adventerprisek9-mz.151-2.T1.bin username@10.1.1.1:/c2800nm-adventerprisek9-mz.151-2.T1.bin

scp c2900-universalk9-mz.SPA.151-4.M7.bin username@10.1.1.1:flash:c2900-universalk9-mz.SPA.151-4.M7.bin
```

Наоборот скачиваем с маршрутизатора в текущий каталог файл callfail.

```bash
scp username@5.5.5.5:flash:callfail .           # точка в конце обозначает текущий каталог
scp username@5.5.5.5:flash:callfail Documents/  # то же самое, но скачиваем в папку Documents
```

Программа SecureCRT+SecureFX:

```
Install settings
The installation directory is
C:\Program Files\VanDyke Software\Clients

SecureCRT will be installed with the following protocols:
SSH2, SSH1, Telnet/SSL, telnet, Rlogin, Serial, TAPI, Raw

SecureFX will be installed with the following protocols:
SFTP, SCP, FTP/SSL, FTP
```

### HTTP копирование

```bash
copy http://192.168.0.10/test.vxml flash0:/AA/test.vxml
```

### Удаление непустого каталога с содержимым
Удалить каталог, в котором есть подкаталоги и файлы, нельзя обычным способом:

```
2951-router#delete /force flash0:CME9.1.0GUI 
%Error deleting flash0:CME9.1.0GUI (Can't delete a directory that has files in it)
```

Необходимо рекурсивное удаление содержимого начиная с файлов самого нижнего уровня:

```
2951-router#delete /force /recursive flash0:CME9.1.0GUI
```

### Перемещение каталогов внутри флеш-памяти
Например, нужно было закачать определенный каталог в корневую директорию, а закачали в каталог внутри корневой директории, и теперь надо переместить файлы. Самый простой путь — удалить каталог и закачать заново в корневую директорию. Но если большой объем или удаленно закачиваем не из локальной сети, целесообразнее переместить каталог. Это делается переименованием, т.к. команда rename не только изменяет имя каталога, но и запрашивает его новое назначение:

```bash
2951-router#rename flash0:phone_loads/CME9.1.0GUI flash0:CME9.1.0GUI # (1)!
Destination filename [CME9.1.0GUI]? 

2951-router#dir     # (2)!
Directory of flash0:/

  1 -rw- 89934456 May 28 2013 22:07:24 +07:00 c2951-universalk9-mz.SPA.152-4.M3.bin
  2 -rw- 3064 May 28 2013 22:17:54 +07:00 cpconfig-29xx.cfg
240 drw- 0 Aug 9 2013 04:34:26 +07:00 phone_loads
  3 drw- 0 May 28 2013 22:18:12 +07:00 ccpexp
239 -rw- 2464 May 28 2013 22:19:54 +07:00 home.shtml
241 drw- 0 Aug 9 2013 04:34:58 +07:00 ringtones
242 drw- 0 Aug 9 2013 04:35:46 +07:00 CME9.1.0GUI
246 -rw- 496521 Aug 9 2013 05:04:12 +07:00 music-on-hold.au
```

1. перемещаем каталог CME9.1.0GUI из каталога *phone_loads* в корневой
2. показать каталоги флеш-памяти и файлы в корневом каталоге
## Локализация, русский язык и тоны
Localization is used to define phone parameters on country bases. Those parameters can be language to use for text displays (user-local) or country-specific tones and cadences (network-local). CME has built-in system-locals for 16 countries and 12 languages. 
For SCCP phones: 
For 7905, 7912, 7940, and 7960 phones, system-locals files are preloaded in IOS. Only remaining is to apply the desired localization. 
For 6921, 6945, 7906, 7911, 7921, 7931, 7941, 7961, 7970, 7971, 8941, 8945, and Cisco IP Communicator phones, system-locals files should be uploaded to IOS (obtained from cisco.com site). The next step will be applying the desired local.
For SIP phones localization has been introduced in CME 8.6. But you need to upload the local files for system-built locals (this is applicable for all SIP Phones types). 
CME has an option to create user-defined locals. Those will allow to create custom parameters (user/network locals) and upload their own files. 
Important Note about User Localization 
Phone displays (menus and prompts) are managed by CME Dictionary & IP Phone Local File. Those displays can be localized by User-Local. Other types of displays which are configured by IOS can't be localized (English display only). 
The following display items are localized by the IP phone (.jar file): 
- System menus accessed with feature buttons (for example, messages, directories, services, settings, and information) 
- Call processing messages 
- Soft keys (for example, Redial and CFwdALL) 

The following display items are localized by the dictionary file for Cisco Unified CME: 
- Directory Service (Local Directory, Local Speed Dial, and Personal Speed Dial) 
- Status Line 

Display options configured through Cisco IOS commands are not localized and can only be displayed in English. For example, this includes features such as: 
- Caller ID 
- Header Bar 
- Phone Labels 
- System Message 

Configuration 
Download the tar file which contain all localization files.
Extract the file in routers flash (archive tar /xtract source-url flash:/file-url)
Create TFTP alias for localization files (for phones that require manual upload of the files). Two files should be assigned to TFTP server which are user-local and network-local:

```
tftp-server flash:/jar_file alias directory_name/jar_file 
tftp-server flash:/g3-tones.xml alias directory_name/g3-tones.xml
```

Apply the required local globally to SCCP/SIP CME. This will be applied to all phones.

```bash
telephony-service         # (1)!
  user-locale RU
  network-locale RU
!
voice register global     # (2)!
  user-locale RU
  network-locale RU
```

1. общие настройки SCCP телефонов
2. общие настройки SIP телефонов

Если языковых файлов на TFTP сервере нет, будет использована английская локаль English US (user-locale 0).
CME 4.0+ одновременно поддерживает до 5 локалей, каждой из которых нужно присвоить свой порядковый номер в общих настройках телефонии. Далее  для шаблонов телефонов (ephone-templates/ voice-register-templates) прописать свои локали, если будут мультиязычные пользователи. Затем шаблоны присваиваются самим телефонам. Если у телефона нет шаблона с определенной локалью, он использует общие глобальные настройки.

```bash
telephony-service 
  user-locale 1 RU
! 
ephone-template 1 
  user-locale 1 
! 
ephone 1
  ephone-template 1   # (1)!
```

1. язык будет русским только если присвоить шаблон телефону

Процесс установки локалей упрощен в CME 7.0.1+ для SCCP телефонов и в 9.0+ для SIP телефонов. Необходимо указать язык/страну и имя архивного tar файла, который должен находиться в том же каталоге, где и файлы конфигураций для телефонов. CME автоматически извлечет файлы JAR и tg3-tones.xml из архива и создаст ссылки на них (alias).

```bash
telephony-service # для SCCP
  user-locale [user-locale-tag] country-code load TAR-filename
  user-locale U2 load CME-locale-fi_FI-7.0.1.1.tar        # (1)!

! 
voice register global // для SIP
 network-locale [user-locale-tag] country-code load TAR-filename
```

1. установка дополнительного языка. Стандартные: DE—Germany, DK—Denmark, ES—Spain, FR—France, IT—Italy, JP—Japan, NL—Netherlands, NO—Norway, PT—Portual, RU—Russia, SE—Sweden, US—United States. Дополнительные: U1, U2, U3, U4, U5.

## Назначение кнопок своих линий
Для SIP телефона:

```bash
voice register dn  14    # порядковый номер directory number
 number 6855             # это номер самой линии (потом закрепляем за телефоном)
 allow watch             # разрешение отслеживать статус занятости этого телефона другими
 name Ivanov R.G.
 label Ivanov R.G.

voice register pool xx  # назначение самого телефона
 number 1 dn 14         # на первую кнопку назначен номер, указанный в voice register dn 14
```

Для SCCP телефона:

```bash
ephone хх
 button 1:14      # на первую кнопку назначен номер, указанный в ephone-dn 14
```

Виды разделителей:
- `:` обычные телефонные линии. Пример:   `button 1:2 2:5`
- `s` silent ring, ringer muted, call waiting beep muted. Пример:  `button 1s2 2s5`
- `b` silent ring, ringer muted, call waiting beep not muted. Пример:   `button 1b2 2b5`
- `f` feature ring. Пример:    button 1f2 2f5. Также смотреть `no dnd feature-ring`
- `m` monitor line, silent ring, call waiting display suppressed. Пример:   `button 1m2 2m5`. Также смотреть `transfer-system full-consult dss`
- `w` watch line (BLF), watch the phone offhook status via the phone's primary ephone-dn. Пример:   `button 1w2 2w5`
- `o` overlay lines, combine multiple lines per physical button. Пример:  `button 1o2,3,4,5`
- `c` overlay call-waiting, combine multiple lines per physical button. Пример:   `button 1c2,3,4,5`. Также смотреть  `huntstop channel' for ephone-dn dual-line`
- `x` expansion/overflow, define additional expansion lines that are used when the primary line for an overlay button is occupied by an active call. Expansion works with 'button o' and not with 'button c'
Примеры:
`button 4o21,22,23,24,25`
`button 5x4`
`button 6x4`

В одной строке через пробелы можно назначать несколько кнопок с разными разделителями:
`button 1:2 2s5 3b7 4f9 5m22 6w10`

## BLF. Кнопки набора внутреннего номера с индикацией занятости и статусы в адресной книге, списках пропущенных, принятых и набранных номеров
Отображаемые состояния:
- линия абонента свободна, при этом индикация кнопки быстрого набора погашена, напротив кнопки и в списках вызовов отображается иконка телефона с положенной трубкой;
- линия абонента занята, кнопка прямого вызова светится красным цветом, напротив кнопки и в списках вызовов отображается иконка телефона со снятой трубкой;
- состояние линии неизвестно, телефон либо не зарегистрирован, либо просмотр его состояния запрещен. При этом индикация кнопки погашена, но высвечивается иконка в виде кубиков.

Не поддерживается отображение занятости на телефонах моделей:
- BLF Call-List: 7905, 7906, 7911, 7912, 7931, 7940, 7960, or 7985, Cisco Unified IP Phone Expansion Modules или Cisco Unified IP Conference Stations.
- BLF Speed-Dial: 7905, 7906, 7911, 7912, 7985 или Cisco Unified IP Conference Stations.

Для ephone-dn или voice register dn, за состоянием которого предполагается следить, должен стоять параметр allow watch — разрешение отслеживания состояния телефона сервисом presence.

```bash
ephone-dn 1
 allow watch
 !
voice register dn 1
 allow watch
Должен быть настроен сервис presence:
sip-ua
 presence enable      # открываем presence в настройках sip ua
!
presence
 presence call-list   # глобально разрешаем просмотр состояния в списках вызовов
 server 177.1.10.10   # дополнительная настройка если надо отследить состояние соединенной с CME АТС, например это может быть CUCM или presense сервер Avaya
 allow subscribe      # для того же
 watcher all          # для того же
```

Этот параметр недостаточно открыть глобально. Нужно открывать для каждого телефона, если хотим разрешить ему видеть состояние в списке пропущенных, принятых, набранных.

```bash
ephone 1                # настройка SCCP телефона
    presence call-list  # разрешаем отслеживать статус номеров в списках вызовов
!
voice register pool 1   # настройка SIP телефона
    presence call list
```

Кнопка blf назначается одинаково для ephone и voice register pool:

```
blf-speed-dial <номер_кнопки от 1 до 113>  <номер_телефона> label <Подпись> [device]
```

Если в подписи два слова, например имя и фамилия, ввести в кавычках. Максимальное количество кнопок зависит от версии софта (версия CME), но отобразится  столько сколько поддерживается телефоном, а также его шаблоном.
Параметр device — опциональный начиная с версии CME 7.1, включает мониторинг всех линий телефона (phone-based), для которого <номер_телефона> является первой и основной линией. Для корректной работы такой кнопки все directory numbers, ассоциированные с отслеживаемым телефоном, должны иметь параметр allow watch, иначе наблюдателю может быть передан неправильный статус.
Посмотреть presence статусы телефонов, где watcher — телефон с настроенной кнопкой blf, presentity — номер, за которым следят:

```
OTK-MSK-c3925-1-VG#sh presence subs sum

Presence Active Subscription Records Summary: 6 subscription

  D indicate as device-based blf-speed-dial monitoring

Watcher                    Presentity               SubID  Expires SibID  Status
========================== ======================== ====== ======= ====== ======
  6721@10.174.84.46        6963@10.174.82.152       241350 3600    0      idle
  6721@10.174.84.46        6967@10.174.82.152       241352 3600    0      idle
  6721@10.174.84.46        6971@10.174.82.152       241354 3600    0      unknown
  6721@10.174.84.46        6577@10.174.82.152       241356 3600    0      unknown
  6721@10.174.84.46        6869@10.174.82.152       241358 3600    0      idle
  3984@10.174.84.31        6855@10.174.82.152       252541 3600    0      idle
```

Посмотреть общий статус presence:

```
sh presence global
```

### Занятость на SIP телефонах

```
blf-speed-dial 1 3984 label "Ivanov Pavel"
```

### Занятость на SCCP телефоне
Способ 1. Как для SIP.

```
blf-speed-dial 1 6855 label "Petrov Sergey"
```

После назначения кнопки в telephony-service ввести create cnf. Снова зайти в телефон и restart. Пример:

```
OTK-MSK-c3925-1-VG(config)#telephony-s
OTK-MSK-c3925-1-VG(config-telephony)#create cnf
Creating CNF files
OTK-MSK-c3925-1-VG(config-telephony)#ephone 15
OTK-MSK-c3925-1-VG(config-ephone)#restart
restarting E8BA.70FB.AEF4
```

Способ 2. Monitor-Line Button for Speed Dial. Старый. Без использования presence. Для мониторинга никаких дополнительных настроек типа allow watch и прочая не требуется.

```bash
ephone-dn 1
number 2311

ephone-dn 22
number 2322

ephone 1
button 1:11       # на этом телефоне назначена только своя линия 2311 на первой кнопке

ephone 2
button 1:22 2m11  # своя линия 2322 на первой кнопке и мониторинг линии 2311 на второй кнопке.
```
### Настройка отображения занятости при объединении двух CME
So tonight I sat down and thought you know what I have never tried to get two CME box's to share presence across each other... So I pulled a configuration together and got it working... Figured I would share in case anyone hadn't seen how it was done or had any questions...

Сценарий
SIP/SCCP enpoints → CME1 → WAN ← CME2 ← SIP/SCCP endpoints

```
Config
-------- 
CME1

presence
  presence call-list (used for Directory Lookup presence status)
  server 10.10.110.2 (IP address of CME2)
  watcher all (allows for external watcher)
   allow subscribe (all for external subscribe)
!
sip-ua
 presence enable (used for Sip phones ..not needed if only SCCP environment)
!
ephone-dn 1
  allow watch (allows for others to monitor your dn)
!
ephone 1
  presence call-list (Directory lookup)
!
voice register dn 1
  allow watch
!
voice register pool 1
 presence call-list
!
dial-peer voice 100 voip
  destination-pattern (CME2-DN's)
  incoming called (CME1-DN's)
  session protocol sipv2
  session target (CME2-IP)
  codec/dtmf/vad - optional
!
====
CME 2
!
Та же конфигурация, только другие dn и ip-адреса 
!
============
Debug commands that are useful

Debug voice dialpeer inout
Debug ccsip messages 
```

## Перевод вызова одной клавишей (DSS)
DSS — Direct Station Select. Обычно актуален для колл-центров. Включить сервис dss в свойствах. После этого во время разговора перевести вызов на другого абонента можно просто нажатием кнопки blf или кнопки мониторинга типа 2m11.

```
telephony-service
    service dss
```

В Avaya реализовано по-другому: кнопка `inst-trans` (вместо кнопки `busy-ind`).
## Разница между Max calls, Busy Trigger и HuntStop
*Как это отображается в статусах телефонов?*
Posted: August 14, 2010 in CME 
The concept of max-calls per button and busy trigger is quite easy but has been made complex.
`max-calls-per-button` — максимальное количество звонков на кнопке, учитывая входящие и исходящие.
`busy-trigger-per-button` — дает «занято» для входящих звонков после достижения максимума, установленного этим параметров. Например, если для ephone установлен параметр `busy-trigger-per-button` равный  2 и на телефоне есть 2 активных разговора, третий звонок придет на голосовую почту или переадресуется на мобильный или другой номер в зависимости от настроек телефона по занятости. Или переадресуется по настройкам `call-forward noan`  (точно???).
`huntstop channel` is used on a shared line for incoming calls, rest of the channels are reserved for outgoing calls.

Пример конфигурации:
- Phone 1 может принять 4 входящих вызова
- Phone 2 может принять 2 входящих вызова
- Общая линия (Shared line) первого и второго телефонов может принять 5 входящих вызовов

```bash
ephone-dn 1 octo-line              # выделенный dn максимум восьмиканальный
  number 3001 no-reg primary
  description +56576513001
  name “Phone 1”
  call-forward busy 3220            # переадресация по занятости с 3001 на 3220
  call-forward noan 3220 timeout 20 # переадресация по неответу через 20 секунд с 3001 на 3220
!
!
ephone-dn 2 octo-line 
  number 3002 no-reg primary
  description +56576513002
  name “Phone 2”
  call-forward busy 3220
  call-forward noan 3220 timeout 20
!
ephone-dn 3 octo-line
  number 3003 no-reg primary
  description +56576513003
  huntstop channel 5      # Общая линия будет использоваться только для входящих, до 5 вызовов
!
!
ephone 1
  privacy-buton
  mac-address XXXX.XXXX.XXXX
  ephone-template 1
  max-calls-per-button 5      # любая кнопка телефона имеет 5 каналов для входящих или исходящих
  busy-trigger-per-button 4   # при этом входящих может быть максимум 4 на любой кнопке
  button 1:1 2:3    # на первой кнопке свой номер 3001, на второй общая линия 3003
!
ephone 2
  privacy-button
  mac-address XXXX.XXXX.XXXX
  ephone-template 1
  max-calls-per-button 5      # любая кнопка телефона имеет 5 каналов для входящих или исходящих
  busy-trigger-per-button 2   # при этом входящих может быть максимум 2 на любой кнопке
  button 1:2 2:3              # на первой кнопке свой номер 3002, на второй общая линия 3003
```

Если будет 2 активных вызова на номер 3003 на втором телефоне, третий входящий придет на ephone 1, т.к. на втором сработает ограничение, установленное busy-trigger. Третий, четвертый и пятый вызовы поступят на ephone 1, но шестой вызов никуда не придет, т.к. сработает ограничение huntstop channel для shared line, настроенное в ephone-dn 3.
## Приветствие с донабором внутреннего номера или меню
### Скрипты VXML
Чтобы работал перевод вызова на телефоне (как обычный, так и full-consult), который ответил после IVR (секретарь или тот, кого донабрали из голосового меню), нужно добавить в общую конфигурацию маршрутизатора (не конкретно настройки телефонии):

```
vxml version 2.0
vxml allow-star-digit
```

При этом для работы перевода вызова в режиме full consult также в общих настройках телефонии стоит:

```
telephony-service
 transfer-system full-consult
 transfer-pattern .T
```

### Пауза между голосовыми сообщениями `<break>`
Элемент `<break>` предназначен для создания паузы в синтезируемом тексте или между файлами аудиозаписи внутри `<prompt></prompt>`. Размер паузы может быть указан в миллисекундах, секундах или с использованием эквивалентных значений в атрибуте size или strength.

```html
<break
     size="(none|small|medium|large)"
     strength="(none|x-weak|weak|medium|strong|x-strong)"
     time="CDATA"/>
```

**size** (опционально) — продолжительность паузы в секундах:
- none — эквивалентно отсутствию паузы
- small — пауза 2 секунды
- medium — пауза 5 секунд (по умолчанию, если ничего не указано)
- large — 10 секунд

**strength** (опционально, более высокий приоритет чем у size и time)— продолжительность паузы в миллисекундах до 1 секунды включительно:
- none - эквивалентно отсутствию паузы
- x-weak, weak — 350 миллисекунд
- medium — 700 миллисекунд
- strong, x-strong — 1 секунда

**time** (опционально)— точное указание длительности паузы в секундах  (s) или миллисекундах (ms). 
**ЗАМЕЧАНИЕ №1**: некоторые голосовые платформы не могут интерпретировать значение атрибута size, выводя сообщение об ошибке. Рекомендуется использовать иные атрибуты для регулирования длительности паузы. 
**ЗАМЕЧАНИЕ №2**: некоторые голосовые платформы не могут интерпретировать значение атрибута time без указания единиц измерения длительности паузы (например, time="10000"). Поэтому рекомендуется всегда указывать единицы измерения (time="10000ms" или time="10s").
## Hunt группы
Полезные команды просмотра:

```bash
show voice hunt-g br        # краткий список всех voice групп в виде таблички
show ephone-hunt summarys   # не очень краткий список ephone групп
sh run | s voice hunt       # Просмотр конфигурации voice хант групп
sh run | s ephone-hunt      # Просмотр конфигурации ephone хант групп
```

## Перехват вызова
По умолчанию горячие клавиши PickUp и GPickUp отображаются на поддерживаемых SCCP и SIP телефонах. Если перед этим они были выключены, нужно включить командой softkeys idle. Для телефонов SCCP функция прямого перехвата Directed Call Pickup доступна нажатием на кнопку PickUp, для телефонов SIP функция выключена. Поддержка для SIP начиная с CME 7.1 добавлением параметра gpickup.

SIP телефоны, не поддерживающие горячие клавиши, могут пользоваться кодами доступа. Разные номера dn, имеющие одинаковый номер, должны иметь идентичные настройки перехвата. Один dn может состоять только в одной группе перехвата одновременно. Номера групп перехвата могут быть разной длины, но первые цифры должны отличаться, т.е. если назначена группа 17, нельзя так же назначать группу 177. Иначе перехват в группе 17 сработает раньше, чем пользователь наберет  последнюю семерку в 177. Звонки с транков H.323 не поддерживаются на SIP телефонах.

| Синтаксис команды IOS                                                                              | Телефоны SCCP                    | Телефоны SIP            |
| -------------------------------------------------------------------------------------------------- | -------------------------------- | ----------------------- |
| service directed-pickup (подразумевается по умолчанию, если ничего не введено в telephony-service) |                                  |                         |
| Directed Call Pickup (перехват звонка, поступившего на любой номер)                                | PickUp + номер телефона          | Не работает             |
| Local Group Pickup – своя группа перехвата (звонок на телефон из той же группы перехвата)          | GPickUp + *                      | GPickUp + * или PickUp  |
| Other Group Pickup — чужая группа перехвата (звонок на телефон из другой группы перехвата)         | GPickUp + номер группы перехвата | то же                   |
| service directed-pickup gpickup                                                                    |                                  |                         |
| Directed Call Pickup                                                                               | GPickUp + номер телефона         | то же                   |
| Local Group Pickup                                                                                 | GPickUp + * или PickUp           | то же                   |
| Other Group Pickup                                                                                 | GPickUp + номер группы перехвата | то же                   |
| no service directed-pickup                                                                         |                                  |                         |
| Directed Call Pickup                                                                               | нет                              | нет                     |
| Local Group Pickup                                                                                 | нет                              | GPickUp + * или PickUp |
| Other Group Pickup                                                                                 | нет                              | нет                     |

Глобальное выключение функции Directed Call Pickup. Изменяется поведение softkey PickUp так, что при нажатии активируется Local Group Pickup вместо Directed Call Pickup.

```
telephony-service
 no service directed-pickup 
```

Directed Call Pickup, Group Pickup и Local Group Pickup можно делать клавишей GPickUp:

```
telephony-service
 service directed-pickup gpickup
```

Для выборочного удаления softkep Pickup с SCCP или аналоговых телефонов, нужно создать шаблон, в котором запрещена Pickup  (так же можно удалить кнопку GPickup).

```
ephone-template 3
 features blocked Pickup|GPickup
```

## Кодеки
### Не работает тональный донабор на внутренний номер CME (телефон SIP)
Сами кодеки прописаны в конфигурации, но кодек не был привязан к телефону:

```bash
voice class codec 1
 codec preference 1 g711ulaw
 codec preference 2 g711alaw
 codec preference 3 g729r8
!
voice class codec 2
 codec preference 1 g729r8
 codec preference 2 g729br8
 codec preference 3 g723r63
 codec preference 4 g723ar63
 codec preference 5 g723r53
 codec preference 6 g723ar53
 codec preference 7 g711alaw
 codec preference 8 g711ulaw
!
voice class h323 1
  h225 timeout tcp establish 3
  h225 timeout setup 3

voice register pool  1
 busy-trigger-per-button 2
 id mac 00CC.FC17.E4A4
 type 7821
 number 1 dn 1
 voice-class codec 1          # забыли присвоить класс кодека 1 SIP телефону
 username 5974 password 5974
```
### Иногда не работает тональный донабор при звонках в город
Не были указаны необходимы типы dtmf набора в dial-peer для городских звонков (смотреть строку dtmf-relay):

``` hl_lines="8"
dial-peer voice 105 voip
 description *** External Calls to QUCM Greenatom - Moscow ***
 translation-profile outgoing CLID_to_City_4953570014
 preference 1
 destination-pattern ^849[589][1-9]......$
 no modem passthrough
 session target ipv4:10.161.224.4
 dtmf-relay h245-signal h245-alphanumeric cisco-rtp rtp-nte
 codec g711ulaw
 fax rate disable
 ip qos dscp cs3 signaling
```

## DTMF Relay
Виды DTMF:
- in-band (передающиеся внутри разговорного тракта)
- out-of-band(предающиеся вне разговорного тракта по средствам какого либо сигнального канала).

Назначение типа dtmf relay:

```
Router(config-dial-peer)#dtmf-relay ?
  cisco-rtp          Cisco Proprietary RTP
  h245-alphanumeric  DTMF Relay via H245 Alphanumeric IE
  h245-signal        DTMF Relay via H245 Signal IE
  rtp-nte            RTP Named Telephone Event RFC 2833
  sip-kpml DTMF Relay via KPML over SIP SUBCRIBE/NOTIFY
  sip-notify DTMF Relay via SIP NOTIFY messages
```

### H323
- **h245-signal**: Метод передачи DTMF по средствам канала сигнализации h.245 в протоколе h.323. Данный метод передает сообщения, которые содержат в себе не только само значение передаваемой цифры, но и его продолжительность
- **h245-alphanumeric**: передает сообщения в канале h.245 протокола h.323, но его отличие от предыдущего в том, что число передается в виде символа ASCII с фиксированной продолжительностью в 500ms. 

Трассировка

```
debug h245 asn1
```

Пример вывода:

```
Aug 21 21:07:37.902: h245_decode_one_pdu: more_pdus = 0, bytesLeftToDecode = 7
Aug 21 21:07:37.902: H245 MSC INCOMING ENCODE BUFFER::= 6D810447200063
Aug 21 21:07:37.902:
Aug 21 21:07:37.902: H245 MSC INCOMING PDU ::=
value MultimediaSystemControlMessage ::= indication : userInput : signal :
{
signalType “9”
duration 100
}
```
### Проприетарный Cisco-RTP
Как видно из названия- этот метод придуман инженерами cisco, а значит и рекомендован он для передачи между оборудованием cisco. Он передает сообщения по тому же каналу что и голосовой траффик, но сами сообщения закодированы иным образом и идентифицируются различными индексами Payload Type.
### SIP INFO
Метод протокола SIP. пердает сообщения по сигнальному каналу протокола SIP а значит является out-of-band методом. По умолчанию включен, но не всегда используется из-за приоритета использования протоколов, поэтому чтобы его увидеть нужно включить в функйии dtmf-relay какой нибудь из h.245 методов. Трассировка производится командой
Прописать:

```
dtmf-relay sip-notify
dtmf-relay sip-kpml
```

Дебаг:
```
debug ccsip messages
```

### RTP-NTE
Описанный в стандарте RFC2833. in-band метод, который использует для передачи DTMF специальные NTE (named telephone events), которые передаются в одном потоке с RTP траффиком, но кодируются несколько иначе. Не зависит от протокола. Описания этих событий можно найти по выше указанной ссылке на стандарт RFC2833 а так же в стандарте 4733. Наше событие имеет type 101 ( Line lockout tone ).
Настройка:

```
dtmf-relay rtp-nte
```

Если нужно переназначить со 101, в dial-peer прописать:

```
rtp payload-type …
  cisco-cas-payload           Cisco CAS RTP payload
  cisco-clear-channel         Cisco clear channel RTP payload
  cisco-codec-aacld           Cisco codec AACLD
  cisco-codec-fax-ack         Cisco codec fax ack
  cisco-codec-fax-ind         Cisco codec fax indication
  cisco-codec-gsmamrnb        Cisco codec GSM AMR Narrow Band
  cisco-codec-ilbc            Cisco codec iLBC
  cisco-codec-isac            Cisco codec ISAC
  cisco-codec-mp4a-latm       Cisco codec MP4A-LATM
  cisco-codec-video-h263+     RTP video codec H263+ payload type
  cisco-codec-video-h264      RTP video codec H264 payload type
  cisco-fax-relay             Cisco fax relay
  cisco-pcm-switch-over-alaw  Cisco RTP PCM codec switch over ind (a-law)
  cisco-pcm-switch-over-ulaw  Cisco RTP PCM codec switch over ind (u-law)
  cisco-rtp-dtmf-relay        cisco-rtp dtmf relay
  comfort-noise               RTP comfort noise payload type
  g726r16                     Using dynamic payload for g726r16 in H323 -- SIP interop
  g726r24                     Using dynamic payload for g726r24 in H323 -- SIP interop
  lmr-tone                    Land Mobile Radio Tone Event
  nse                         Named Signaling Event
  nte                         Named Telephone Event
  nte-tone                    Named Telephone Tone Event

debug voip rtp session named-event
```

Пример вывода

```
Aug 21 21:07:37.902: Pt:101 Evt:9 Pkt:04 00 00 <Snd>>>
Aug 21 21:07:37.902: Pt:101 Evt:9 Pkt:04 00 00 <Snd>>>
Aug 21 21:07:37.902: Pt:101 Evt:9 Pkt:04 00 00 <Snd>>>
Aug 21 21:07:37.902: Pt:101 Evt:9 Pkt:04 01 90 <Snd>>>
Aug 21 21:07:37.902: Pt:101 Evt:9 Pkt:84 03 20 <Snd>>>
Aug 21 21:07:37.902: Pt:101 Evt:9 Pkt:84 03 20 <Snd>>>
Aug 21 21:07:37.902: Pt:101 Evt:9 Pkt:84 03 20 <Snd>>>
```

https://supportforums.cisco.com/document/144711/understanding-dtmf-negotiation-and-troubleshooting-sip-trunks
