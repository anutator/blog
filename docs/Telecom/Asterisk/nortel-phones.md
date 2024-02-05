---
title: Настройки телефонов Nortel
share: "true"
tags:
  - avaya
---

Конфигурация для Nortel от igorg (гуру по Астериск) с использованием шаблонов для разных моделей, файл *unistim.conf*
```ini title="unistim.conf"
; /etc/asterisk/unistim.conf
[i200x](!) ; шаблон
rtp_method=1
language=ru
country=ru
timeformat=2
extension=line
context=<имя контекста>
titledefault=Asterisk
maintext0="text 0"
maintext1="Good day"
callgroup=1
pickupgroup=1

[i2002](!)  ; шаблон
height=1

[701](i200x)
device=<мак-адрес телефона>
line => 701
bookmark=4@Igor Home@210@36
bookmark=5@Igor Mobile@номер_мобилы@36

[703](i200x)
rtp_method=3
ringvolume=1
context=<имя контекста>
dateformat=0
timeformat=2
contrast=8
ringstyle=3
callhistory=1
callerid="Имя Фамилия" <703>
device=мак_адрес
line => 703
line => 703
bookmark=4@Mailbox@*97@54
bookmark=5@Masha Mobile@номер_мобилы
hasexp=yes
extbookmark=3@Igor.G@101

[705](i200x,i2002)
device=мак_адрес
line => 705
line => 705
```

extensions.conf
```ini title="extensions.conf"
; /etc/asterisk/extenstions.conf

[globals]
; Основные опции для внутренних звонков, используемые в контексте Dial-Users
;Здесь определен только таймаут. Другие опции в документации приложения Dial
INTERNAL_DIAL_OPT=,30  ; звоним на телефонах 30 секунд

[Hints]
; динамически формируем хинты для каждого внутреннего четырехзначного номера, начинающегося на 11
exten = _11XX,hint,PJSIP/${EXTEN}

[Features]
; проверка голосовой почты. Пользователю не надо вводить свой пинкод
exten = 8000,1,Verbose(1, "User ${CALLERID(num)} dialed the voicemail feature.")
 same = n,VoiceMailMain(${CALLERID{num)}@example,s)
 same = n,Hangup()
 
; номер набора меню IVR изнутри
exten = 1100,1,Verbose(1, "User ${CALLERID(num)} dialed the IVR.")
 same = n,Goto(Main-IVR,2565551100,1)

; вход в конференцию только для внутренних сотрудников компании
exten = 6000,1,Verbose(1, "User ${CALLERID(num) entered employee conference.")
 same = Confbridge(employees)
 same = n,Hangup()

; вход в общую конференцию для сотрудников компании и заказчиков
exten = 6500,1,Verbose(1, "User ${CALLERID(num)} dialed the employee/customer mixed conference.")
 same = n,Confbridge(mixed)
 same = n,Hangup()
 
[Dialing-Errors]
; Обработка внутренних звонков на несуществующие номера.
; Опционально. Можно просто не обрабатывать.
exten = _X.,1,Verbose(1, "User ${CALLERID(num)} dialed an invalid number.")
 same = n,Playback(pbx-invalid)
 same = n,Hangup()
 
[Internal-Setup]
;Пример предварительной обработки звонка для выполнения любого действия
;перед тем как отправить звонок далее на внутренний номер.
;В примере ниже выключаем сбор статистики CDR для внутренних вызовов.
enten = _X.,1,NoOp()
 same = n,Set(CDR_PROP(disable)=1)
 same = n,Goto(Internal-Main),${EXTEN},1)
 
[Internal-Main]
; контекст дает возможность внутренним абонентам звонить на номера и функции
; внешних абонентов можно отправить на один из контекстов, которые включены сюда
; т.е. контексты не обязательно для внутренних пользователей

include =Hints
include = Features
include = Dial-Users
include = Dialing-Errors

[Dial-Users]
; Звонки на внутренние телефоны
; Поступающие звонки могут быть внутренними или внешними
exten = _11XX,1,Verbose(1, @User ${CALLERID(num)} dialed ${EXTEN}.")
 same = n,Set(SAC_DIALED_EXTEN=${EXTEN})
 same = n,Gotoif($[$DEVICE_STATE(PJSIP/${EXTEN})} = BUSY]?dialed-BUSY,1:)
 same = n,Dial(PJSIP/${EXTEN}${INTERNAL_DIAL_OPT) ; добавляется ,30
 same = n,Goto(dialed-${DIALSTATUS},1)
 
exten = dialed-NOANSWER,1,NoOp()
 same = n,Voicemail(${SAC_DIALED_EXTEN}@example,u)
 same = n,Hangup()
 
exten = dialed-BUSY,1,NoOp()
 same = n,Voicenail(${SAC_DIALED_EXTEN}@example,b)
 same = n,Hangup()

exten = dialed-CHANUNAVAIL,1,NoOp()
 same = n,Playback(pbx-invalid)
 same = n,Hangup()

[catchall]
;вылавливаем ip-адреса анонимных звонящих,если в sip.conf надо оставить анонимные звонки
;в дальнейшем адреса можно забанить в файрволе
exten = s,1,Verbose(Dead calls rising)
 same = n,Set(uri=${SIPCHANINFO(uri)})
 same = n,Verbose(3,Unknown call from ${uri} to ${EXTEN})
 same = n,System(echo "[${STRFTIME(${EPOCH},,%b %d %H:%M:%S)}] SECURITY[] Unknown Call from ${CALLERIDNUM} to ${FROM_DID} IPdetails ${uri}" >> /var/log/asterisk/sipsec.log)
 same = n,Hangup()
 
 ....
 
 [to-sip-iqtek]  ; звонки с телефонов Nortel
 exten = 000,1,Playback(beep)
  same = n,Echo()
  same = n,Hangup()
  
 exten = _7XX,1,Dial(USTM/${EXTEN}@${EXTEN})
  same = n,Hangup()
 
 exten = _[125]XX,1,Dial(SIP/${EXTEN}@pbx.iqtek.ru0
  same = n,Hangup()
  
exten = _00X,1,Dial(SIP/${EXTEN@pbx.iqtek.ru)
 same = n,Hangup()

exten = *X.,1,Dial(SIP/${EXTEN}@pbx.iqtek.ru)
 same = n,Hangup()
 
exten = _[02-9]XXX.,1,Dial(SIP/${EXTEN}@pbx.iqtek.ru)
 same = n,Hangup()
```