---
share: "true"
title: Equinox
tags:
  - avaya
---
Опции установки софтфона Equinox.

| Опция                                                                                                                                                                                               | Комментарии                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `"Avaya Equinox Setup.msi" /qn`                                                                                                                                                                       | Установка в тихом режиме (silent installation).<br><br>Установить на ПК стенда Avaya Equinox 3.5 в этом режиме с указанными ниже параметрами                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `"Avaya Equinox Setup.msi" AUTOCONFIG="https://sng-nv-abk1-ucaads1.smn.rosneft.ru:8443/acs/resources/configurations"` | Установка с автоматическим конфигурированием из URL. (чтобы пользователя не запрашивали e-mail). Почему-то не подхватил этот URL автоматом, хотя отображал его в Equinox при первом подключении. При повторном ручном вводе этого URL в самом Equinox сработало.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| `"Avaya Equinox Setup.msi" NOQOS=0`                                                                                                                                                                   | Включить установку драйвера DSCP — Differentiated Services Code Point (по умолчанию выключено). DSCP — поле в IP пакете для маркировки различных уровней сервиса сетевого трафика. Стандартно: голос Audio media streams как **Expedited** **Forwarding** (EF) — DSCP 46, видео Video media streams как Class 4 **Assured** **Forwarding** (AF41) — DSCP 34, сигнализация Signaling streams SIP, XMPP и др. как **Class** **Selector** **3** (CS3) — DSCP 24 (эти поля видны в Wireshark).<br><br>Настройки через веб-интерфейс SMGR→ Elements→ Session Manager→ Device and Location Configuration→ Device Settings Groups, параметр DIFFSERV/QOS Parameters:<br><br>Call Control PHB Value, Audio PHB Value, Video PHB Value. Можно настроить из 46xxsettings.txt:<br>SET DSCPAUD "46"<br>SET DSCPSIG "24"<br>SET DSCPVID "34"<br><br>При установке драйвера DSCP настройки Avaya из SMGR или файлов телефонов имеют приоритет над Microsoft QWAVE API (MS QoS policy) и преимущество, т.к. у Microsoft ограничение: нельзя разделить трафик аудио и видео.
| `"Avaya Equinox Setup.msi" OP=0  `                                                                                                                                                                    | Выключить установку плагина Outlook. Он нужен только организаторам конференций (имеется 104 лицензии организаторов конференций). С флагом Outlook plugin не ставится (см. скриншот).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `"Avaya Equinox Setup.msi" BP=0`                                                                                                                                                                      | Выключить установку плагина браузера. Используется для инициирования набора номеров с web-страниц. Необходимо уточнить, как выглядят номера при использовании плагина (некоторых пользователей раздражает выделение номеров на web-страницах)<br><br>Стандартно плагин интегрируется с Internet Explorer, для интеграции с Google Chrome необходимы. С флагом Browser plugin не ставится (см. скриншот).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `msiexec /i "Avaya Equinox Setup.msi" /L*v install_log.txt`                                                                                                                                           | Во время установки лог создается в текущем каталоге (находимся в каталоге с установщиком)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `msiexec /qn /x "Avaya Equinox Setup.msi"`                                                                                                                                                            | Удаление программы Equinox в «тихом» режиме                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `msiexec /x "Avaya Equinox Setup.msi" /L*v uninstall_log.txt`                                                                                                                                         | Удаление Equinox с сохранением лога в текущий каталог в файл `uninstall_log.txt `                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |

## Настройка Wireshark для отображения DSCP поля
Использование DSCP-драйвера обеспечивает возможность маркировки генерируемого приложением голосового, видео и трафика телефонной сигнализации значениями DSCP в соответствии с указанными в подсистеме управления СТСиУК значениями.

Правой кнопкой мыши щелкаем на названии одной из колонок и жмем на Column Preferences. Нажав знак `+` добавим дополнительную колонку с именем DSCP, задаем ей тип Custom и вводим имя поля `ip.dsfield.dscp`:

![](Wireshark-Add-DSCP-Column.png)

Сохраняем — Ok. Колонку передвигаем влево от колонки Info.
Видно, что трафик маркируется. Вот звонок, где только аудио. Видно **Expedited Forwarding** (аудио, DSCP 46) и **Class Selector 3** (сигнализация, DSCP 24).

![](7097-7093-only-audio.png)

При видеозвонке между двумя Equinox добавляется Assured Forwarding 42 (в скриншоте), а не Assured Forwarding 41, потому что я ошиблась и в SMGR ввела в поле Video PHB Value 36 вместо 34. После исправления будет как положено Assured Forwarding 41. Зато мы точно знаем, что трафик маркируется согласно настройкам веб-интерфейса SMGR.

DSCP значения согласно стандарту RFC 2475.

| DSCP значение | Десятичное значение | Описание                                    | Drop Probability | Эквивалент значения критичности IP Precedence |
| ------------- | ------------------- | ------------------------------------------- | ---------------- | --------------------------------------------- |
| **101 110**   | **46**              | **High Priority Expedited Forwarding (EF)** | **N/A**          | **101 - Critical**                            |
| 000 000       | 0                   | Best Effort                                 | N/A              | 000 - Routine                                 |
| 001 010       | 10                  | AF11                                        | Low              | 001 - Priority                                |
| 001 100       | 12                  | AF12                                        | Medium           | 001 - Priority                                |
| 001 110       | 14                  | AF13                                        | High             | 001 - Priority                                |
| 010 010       | 18                  | AF21                                        | Low              | 010 - Immediate                               |
| 010 100       | 20                  | AF22                                        | Medium           | 010 - Immediate                               |
| 010 110       | 22                  | AF23                                        | High             | 010 - Immediate                               |
| 011 010       | 26                  | AF31                                        | Low              | 011 - Flash                                   |
| 011 100       | 28                  | AF32                                        | Medium           | 011 - Flash                                   |
| 011 110       | 30                  | AF33                                        | High             | 011 - Flash                                   |
| **100 010**   | **34**              | **AF41**                                    | **Low**          | **100 - Flash Override**                      |
| 100 100       | 36                  | AF42 (ошибочно ставила)                     | Medium           | 100 - Flash Override                          |
| 100 110       | 38                  | AF43                                        | High             | 100 - Flash Override                          |
| 001 000       | 8                   | CS1                                         |                  | 1                                             |
| 010 000       | 16                  | CS2                                         |                  | 2                                             |
| **011 000**   | **24**              | **CS3**                                     |                  | **3**                                         |
| 100 000       | 32                  | CS4                                         |                  | 4                                             |
| 101 000       | 40                  | CS5                                         |                  | 5                                             |
| 110 000       | 48                  | CS6                                         |                  | 6                                             |
| 111 000       | 56                  | CS7                                         |                  | 7                                             |
| 000 000       | 0                   | Default                                     |                  |                                               |

Первые три двоичные цифры поля DSCP — значение приоритета IP трафика

| Значение | Описание                                             |
| -------- | ---------------------------------------------------- |
| 000 (0)  | Routine or Best Effort                               |
| 001 (1)  | Priority                                             |
| 010 (2)  | Immediate                                            |
| 011 (3)  | Flash (mainly used for voice signaling or for video) |
| 100 (4)  | Flash Override                                       |
| 101 (5)  | Критичный (в основном для голоса RTP)                |
| 110 (6)  | Internet                                             |
| 111 (7)  | Network                                              |

![](7097-outgoing-to-7093-video.png)

## Какие ошибки периодически появляются в Equinox
Периодически теряет соединение

![](Equinox-LostConnection1.png)

![](Equinox-LostConnection2.png)

### Логи на шлюзах
`UCCLOG.log` — логи Equinox. При проблемах с сертификатом обход по списку SIP контроллеров завершится.

## Установка Equinox с флагами и без
Для установка переименовать название скаченной версии `Avaya Equinox Setup 3.5.0.52.37.msi` на `Avaya Equinox Setup.msi`.
Тихая установка без плагинов (дополнительно сохраняется лог установки в каталоге программы):

```powershell
Microsoft Windows [Version 6.1.7601]
<c> Корпорация Майкрософт (Microsoft Corp.>, 2009. Все права защищены.

C:\Windows\system32>cd C:\Equinox

C:\Equinox>msiexec /i "Avaya Equinox Setup.msi" /L*v install_log.txt /qn NOQOS=0 OP=0 BP=0 AUTOCONFIG="https://sng-nv-abk1-ucaads1.smn.rosneft.ru:8443/acs/resources/configurations"

C:\Equinox>
```

Установка с плагинами Outlook и браузера (сохраняется лог установки):

```sh
msiexec /i "Avaya Equinox Setup.msi" /L*v install_log.txt /qn NOQOS=0 AUTOCONFIG="https://sng-nv-abk1-ucaads1.smn.rosneft.ru:8443/acs/resources/configurations"
```

Установка драйвера DSCP требует дополнительного подтверждения (на Windows 10 автоматом):

![](Windows2012-conform-dscp.png)

Аналогичное на русском:

![](Windows2012-conform-dscp-russian.png)

![](equinox-enter1.png)

![](equinox-enter1-2.png)


![](equinox-enter2.png)

![](equinox-calls.png)

Диспетчер учетных данных, Общие учетные данные.

![](equinox-uchetnie-dannie.png)

Реестр — DoNotDisturbDisableAddinList

![](equinox-dnd.png)
## Плагины браузера и Outlook
Плагины браузера и Outlook при установке Equinox c флагами `OP=0 BP=0`:
Включить плагины невозможно, т.к. их вообще нет!

![](equinox-no-plugins.png)
### Плагин браузера
Когда плагин установлен, он находится в каталоге fTarget внутри каталога **Avaya Equinox**
`C:/Program%20Files%20(x86)/Avaya/Avaya%20Equinox/fTarget`

![](equinox-BrowserExtension.png)

Установка через `accs_crx_update_manifest.xml`
В Internet Explore плагин ставится автоматом.

![](equinox-IE-AvayaBrowserExtension.png)

![](equinox-IE-AvayaBrowserExtension2.png)

В Google Chrome настройка добавляется дополнительными действиями.

При флаге `BP=0` каталог **fTarget** вообще не создается:

![](equinox-no-fTarget.png)

### Плагин Outlook
Плагин Outlook, когда он установлен:

![](equinox-Outlook-with-Extension.png)

![](equinox-Outlook-nadstroyki.png)

После установки Equinox с флагом `OP=0` (НЕ ставить расширение для Outlook) пусто:

![](equinox-Outlook-without-Extension.png)