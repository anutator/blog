---
title: Настройка СО линий Avaya
share: "true"
---
## Отбой на аналоговых платах (защита от подвисания Сошек)
Прошивки аналоговых плат последние.

Убедиться, что в `disp sys cu` в Optional Features стоит `Global Call Classification? y`
```console title="disp sys cu" hl_lines="16"
display system-parameters customer-options                      Page   4 of  11
                                OPTIONAL FEATURES

   Emergency Access to Attendant? y                              IP Stations? y
           Enable 'dadmin' Login? y
           Enhanced Conferencing? y                        ISDN Feature Plus? y
                  Enhanced EC500? y        ISDN/SIP Network Call Redirection? y
    Enterprise Survivable Server? n                          ISDN-BRI Trunks? y
       Enterprise Wide Licensing? n                                 ISDN-PRI? y
              ESS Administration? y               Local Survivable Processor? n
          Extended Cvg/Fwd Admin? y                     Malicious Call Trace? y
     External Device Alarm Admin? y                 Media Encryption Over IP? n
  Five Port Networks Max Per MCC? n     Mode Code for Centralized Voice Mail? n
                Flexible Billing? y
   Forced Entry of Account Codes? y                 Multifrequency Signaling? y
      Global Call Classification? y         Multimedia Call Handling (Basic)? y
             Hospitality (Basic)? y      Multimedia Call Handling (Enhanced)? y
Hospitality (G3V3 Enhancements)? y               Multimedia IP SIP Trunking? n
                       IP Trunks? y

           IP Attendant Consoles? y
        (NOTE: You must logoff & login to effect the permission changes.)
```

В параметрах страны `ch sys cou` (не везде есть этот экран), ставим `Enable Busy Tone Disconnect for Analog loop-start Trunks? y`

```console title="disp sys cou" hl_lines="9"
display system-parameters country-options
                       SYSTEM PARAMETERS COUNTRY-OPTIONS

                                 Set Layer 1 timer T1 to 30 seconds? n
                                            * Display Character Set: Cyrillic
                                        Directory Search Sort Order: Cyrillic
                                             Howler Tone After Busy? n
                               Disconnect on No Answer by Call Type? n
           Enable Busy Tone Disconnect for Analog loop-start Trunks? y

TONE DETECTION PARAMETERS
   Tone Detection Mode: 6
      Interdigit Pause: short

* WARNING: Changing the display character set will cause Non-ASCII data to
            appear incorrectly
```

`ch sys ocm` на 1 странице `USA Default Algorithm? n`, на второй странице настраиваем длительность сигналов, попробовать 225 и 525, 250 и 550. По ГОСТу в России стандартная частота сигнала отбоя 425 Гц при длительности 350 мс через 350 мс тишины, но может быть и другой вариант. В Египте обычно On 250, off 550. Для точного определения интервалов можно включить запись разговора (предварительно настроив Audix) и после ответа положить трубку на удаленной стороне, немного подождать для записи сигналов «занято». Скачать файл и открыть в Audacity или SoundForge. Проверить интервалы по временной шкале.

``` title="ch sys ocm" hl_lines="6"
change system-parameters ocm-call-classification                Page   1 of   9
                   SYSTEM PARAMETERS OCM-CALL-CLASSIFICATION

         TONE DETECTION PARAMETERS:
       Global Classifier Adjustment (dB): 0
                   USA Default Algorithm? n
                       USA SIT Algorithm? y
     Global Busy Tone Detection Adj (db):  5
     Cadence Classification After Answer? n
```

``` title="ch sys ocm"
change system-parameters ocm-call-classification                Page   2 of   9
                   SYSTEM PARAMETERS OCM-CALL-CLASSIFICATION

         Tone Name  Instance   Tone     Cadence  Duration  Duration
                             Continuous   Step   Minimum   Maximum
        busy            4        n       1: on     225       525
                                         2: off    225       525
                                         3: on     225       525
                                         4: off    225       525
                                         5: on
                                         6: off
                                         7: on
                                         8: off

(NOTE: Custom tones (pages 2-9) will only be used if USA Default is 'n'.)
```

В свойствах транк-группы `ch tru xxx` поставить `Busy Tone Disconnect? y`

```console title="ch tru 1" hl_lines="11"
change trunk-group 1                                          Page 4 of 22
ADMINISTRABLE TIMERS
            Send Incoming/Outgoing Disconnect Timers to TN465 Ports? n
        Incoming Disconnect(msec): 500     Outgoing Disconnect(msec): 500
                                          Outgoing Dial Guard(msec): 1600 
  Incoming Glare Guard(msec): 1500        Outgoing Glare Guard(msec): 1500
                               Outgoing Rotary Dial Interdigit(msec): 800
      Ringing Monitor(msec): 5200             Incoming Seizure(msec): 500
  Outgoing End of Dial(sec): 10       Outgoing Seizure Response(sec): 5
Programmed Dial Pause(msec): 1500                 Flash Length(msec): 540 
Busy Tone Disconnect? y 

END TO END SIGNALING
  Tone(msec): 350   Pause(msec): 150 

OUTPULSING INFORMATION
    PPS: 10    Make(msec): 40   Break(msec): 60   PPM? N
```

Здесь же на 2 странице поэкспериментировать с ` Disconnect Supervision - In`. Выключить его. Но это может вызвать проблемы при переадресации с транка на транк, поэтому надо подобрать или y, или n.

```console title="ch tru 1" hl_lines=11"
change trunk-group 1                                Page 2 of 22
Group Type: co                      Trunk Type: loop-start

TRUNK PARAMETERS
     Outgoing Dial Type: rotary                         Cut-Through? n
      Trunk Termination: rc                 Disconnect Timing(msec): 500 
           Auto Guard? n   Call Still Held? n     Sig Bit Inversion: none 
    Analog Loss Group: 6                         Digital Loss Group: 11 
                            Trunk Gain: high 

Disconnect Supervision - In? n Out? n                 Cyclical Hunt? y 
Answer Supervision Timeout: 10           Receive Answer Supervision? n 
        Administer Timers? y
```

В `ch sys fea` поставить:
`Trunk-to-Trunk Transfer? none`	 — запрет отправить внешний звонок вовне (отключили возможность переадресации, но возможно с другими подобранными настройками и при разрешенной переадресации с транка на транк всё будет работать)
`Call Park Timeout Interval (minutes): 10`время парковки вызова (если вызов отправлен на парковку и не обработан в течение этого времени, он вернётся к тому ext, который его на парковку отправил, или к attendant - зависит от `Deluxe Paging and Call Park Timeout to Originator? y/n` )
`Off-Premises Tone Detect Timeout Interval (seconds): 20`

```console title="ch sys fea" hl_lines="5 8"
change system-parameters features Page 1 of 17

                             FEATURE-RELATED SYSTEM PARAMETERS
                                Self Station Display Enabled? n
                                     Trunk-to-Trunk Transfer: none
     Automatic Callback - No Answer Timeout Interval (rings): 3
                        Call Park Timeout Interval (minutes): 10
         Off-Premises Tone Detect Timeout Interval (seconds): 20
                                  AAR/ARS Dial Tone Required? y
                              Music/Tone on Hold: music Type: ext   2270
               Music (or Silence) on Transferred Trunk Calls? all
                        DID/Tie/ISDN/SIP Intercept Treatment: attd
     Internal Auto-Answer of Attd-Extended/Transferred Calls: transferred
                   Automatic Circuit Assurance (ACA) Enabled? n

              Abbreviated Dial Programming by Assigned Lists? n
        Auto Abbreviated/Delayed Transition Interval (rings): 2
                     Protocol for Caller ID Analog Terminals: Bellcore
     Display Calling Number for Room to Room Caller ID Calls? n

                Controlled Toll Restriction Replaces: none
```

на 4 странице
`Deluxe Paging and Call Park Timeout to Originator? y`	когда звонок приняли на т/а 1, отправили на парковку и забыли о нем, звонок возвращается на т/а 1 по прошествии времени? указанного в Call Park Timeout Interval (minutes)

на 6 странице:
`Night Service Disconect Timer 180` —	если звонок пришёл в ночное время (реально от времени не зависит,  включается/выключается кнопкой на телефоне или командой SAT), отбить его через 3 минуты, если никто не ответил
`Station Call Transfer Recall Timer (seconds):`	когда звонок приняли на т/а 1, button transfer - ext т/а 2 -button transfer, положили трубку, а там не ответили, звонок возвращается на т/а 1 по прошествии установленного количества секунд. Если 0, эта функция отключена.

на 10 странице:
`Wait Answer Supervision Timer: y`	звонок не отвеченный в течение 50 секунд отбивать, если не изменён Unanswered DID Call Timer (проверить!)

Может понадобиться подстройка `"Global Busy Tone Detection Adj (db): X" from (0 to 5)`. Попробовать 0, если не работает 1. Пока не заработает. Но это скорее всего на новых версиях CM не нужно.

Возможно прописать, у каждого телефона Coverage Path, конечной точкой в котором переход на VDN->vector, где прописано busy
Можно прописать time-out исходящего вызова, по истечение которого звонок отобьётся даже если реально идёт разговор.

## Направление входящих вызовов
Если звонок приходит по ISDN PRI или IP транку, то УАТС видит на какой номер пришёл вызов.
- если в вашу УАТС приходит не много номеров на эту TrunkGroup, то в `ch inc tru xxx` (`change inc-call-handling-trmt trunk-group номерГруппы`) надо указать вызываемый номер, и преобразовать его в какой-то extension, на который он направляется далее (операции DEL и INS оперируют с левой частью номера. Т.е. если `1234567` DEL `3`, INS `567`, получится `5674567`
- если в вашу УАТС приходит много номеров на эту транковую группу, и они просто не влезают в таблицу INCOMING CALL HANDLING TREATMENT, всем входящим номерам добавить (INS) Auto Route Selection (ARS) Access Code (указанный в `ch fea`, например 9), а далее в таблице ARS DIGIT CONVERSION TABLE, в которой больше места, ( `ch ars dig`) указать, по какому вызываемому номеру на какой extension направить звонок

Провайдер возможно отправляет вам не все цифры номера, это можно проверить в трассировке. В INCOMING CALL HANDLING TREATMENT или ARS DIGIT CONVERSION TABLE надо прописывать только реально пришедшие цифры (или часть). Например абонент набрал 84955223564, а пришли только цифры 3564.

### Особенности настройки входящих вызовов СО линий
Если звонок приходит по CO линии, УАТС не знает, какой именно входящий номер был набран, воспринимая входящие вызовы только по изменению напряжения в линии.

По умолчанию в стандартном режиме всю группу CO линий, прописанных в одной транковой группе, можно направить только на один extension. Т.е. если разные городские должны приходить на разные внутренние телефоны или на приветствие, надо создавать отдельную транковую группу для каждой СО линии (притом в названии транка или в названии единственной линии транка, надо не забыть прописать какой номер надо набрать где-то в городе, чтобы звонок пришёл на эту линию, чтобы не забыть, какой транк за какой номер отвечает. Далее на первой странице каждого CO транка в `Incoming Destination:` указывается внутреннийНомер (extension), куда направить далее звонок, пришедший по этой линии.

**Обходной вариант**
В ночном режиме Avaya позволяет направлять аналоговые линии одной транковой группы на разные extensions (номера). В `add tru xxx` на странице, где прописываются порты аналоговых плат, к которым подключены CO линии, прописать, куда направляются вызовы в ночном режиме: GROUP MEMBER ASSIGNMENTS, поле Night. Теперь надо принудительно переключить транковую группу в ночной режим. Любому цифровому аппарату (можно даже виртуальному с портом x) назначить кнопку `trunk-ns Grp номерГруппы` и включить эту кнопку из SAT (ASA) командой `enable night-service trunk-group номерГруппы`

## Personal CO-Line
Настройка Personal CO-line позволяет совместить корпоративный и  городской номер телефонной сети в одном телефонном аппарате.
Для этого нужна плата TN747 и приходящие прямые телефонных пары с ГТС. После установки на кросс расшивается кабель Telko 50. Так как плата TN747 имеет в себе только 8 портов, в кабеле используется 8 пар в таком порядке:

```
 01 - - 04 - -07 - - 10 - - 13 - - 16 - - 19 - - 22
 т.е 
 01 пара - 1 порт
 04 пара - 2 порт
 07 пара - 3 порт
 10 пара - 4 порт
 13 пара - 5 порт
 16 пара - 6 порт
 19 пара - 7 порт 
 22 пара - 8 порт
```

Берем пару с городским номером, кроссируем в первый порт. Далее выполняем команду `add per n` где n — номер группы.

```
change personal-CO-line 1                                       Page   1 of   3
                            PERSONAL CO LINE GROUP
 
 Group Number: 1                    Group Type: wats           CDR Reports? y
   Group Name: OUTSIDE CALL                                            TAC: 015
Security Code: 1234                  Coverage Path: 1         Data Restriction? n
                              Outgoing Display? n
TRUNK PARAMETERS  
                 Trunk Type: loop-start             Trunk Direction: two-way
                 Trunk Port: 02A1201        Disconnect Timing(msec): 500
                 Trunk Name: Ext67000             Trunk Termination: rc
         Outgoing Dial Type: tone                 Analog Loss Group: 6                                          
         Digital Loss Group: 11
Disconnect Supervision - In? n                      Call Still Held? n
 Answer Supervision Timeout: 10          Receive Answer Supervision? n
                 Trunk Gain: high                           Country: 1
          Charge Conversion: 1  
              Decimal Point: none  
            Currency Symbol:  
                Charge Type: units


change personal-CO-line 1                                       Page   2 of   3
                            PERSONAL CO LINE GROUP
  
ASSIGNED MEMBERS (Stations with a button for this PCOL Group)
  
    Extension       Name  
 1: 6-хх-хх           Senior Engineer
 2:  
 3: 6-хх-хх           Middle Engineer
 4:
  
ADMINISTRABLE TIMERS
  
   Incoming Disconnect(msec): 500             Outgoing Disconnect(msec): 500
                                              Outgoing Dial Guard(msec): 1600
  Incoming Glare Guard(msec): 1500           Outgoing Glare Guard(msec): 1500
  
                                  Outgoing Rotary Dial Interdigit(msec): 800
       Ringing Monitor(msec): 5200               Incoming Seizure(msec): 500
   Outgoing End of Dial(sec): 10         Outgoing Seizure Response(sec): 5
 Programmed Dial Pause(msec): 1500  
          Flash Length(msec): 540  
  
END TO END SIGNALING  
    Tone(msec): 350      Pause(msec): 150
  
OUTPULSING INFORMATION  
     PPS: 10    Make(msec): 40   Break(msec): 60      PPM? n
```

На телефоне абонента создать  кнопку `per-COline Grp:1` с указанием номера группы.