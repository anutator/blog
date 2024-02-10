---
title: Работа с аудиофайлами
share: "true"
---
## Подготовка аудио в Audacity
Формат аудиофайлов для Communication Manager:
- CCITT A-law или Mu-law (у нас Mu-law, он же U-law)  
- 8 KGz sample rate
- 8 bits per sample
- Mono

*Разделить стерео дорожку в моно*, после чего нижнюю дорожку удалить, нажав на крестик.

![cid:image003.png@01D451C2.B183D7E0](image223.png)

*Правка→ Параметры→ Качество*. Частота дискретизации о умолчанию 8000 Hz

![cid:image002.png@01D451C4.40DE0900](image224.png)

*Файл→ Экспорт→ Экспортировать как WAV*
Выбрать *Прочие несжатые параметры*, заголовок WAV, кодирование U-law.

![cid:image001.png@01D451C3.CF9F6770](image225.png)

Установить программу [MediaInfo](https://mediaarea.net/ru/MediaInfo) для проверки форматов файлов.
Было:
![cid:image005.png@01D451C4.8E5BAE80](image226.png)
Стало:
![cid:image006.png@01D451C4.8E5BAE80](image227.png)

## Список приветствий
Все имена приветствий должны соответствовать именам залитых файлов. Расширение файлов `.wav` в именах приветствий не указывается.

```console title="li anno"
list announcement
                           ANNOUNCEMENTS/AUDIO SOURCES
Announcement                                                            Num of
Extension             Type       Name                        Source     Sources
71000                 integ-mus  moh                          G1          4
71001                 integrated main                         G1          4
71002                 integrated busy                         G1          4
71003                 integrated poseshenie                   G1          4
71004                 integrated main-night                   G1          4
71005                 integrated record                       G1          4
71006                 integrated poseshenie1                  G1          4
71007                 integrated poseshenie2                  G1          4
71008                 integrated poseshenie3                  G1          4
71009                 integrated poseshenie4                  G1          4
71010                 integrated donabor                      G1          4
71011                 integrated posesh-night                 G1          4
71012                 integrated Privetstvie1                 G1          4
71014                 integrated nedostupen                   G1          4
71015                 integrated Privetstvie6150000           G1          4
```

**G1** — это объединенный пул из четырех медиашлюзов, т.е. любое приветствие должно существовать на каждом медиашлюзе, хранится на виртуальной плате с порядковым номером 9.

```console title="disp audio 1"
display audio-group 1                                           Page   1 of   5
                                AUDIO GROUP 1

                          Group Name: G1

AUDIO SOURCE LOCATION
  1: 001V9    16:          31:          46:          61:          76:
  2: 002V9    17:          32:          47:          62:          77:
  3: 003V9    18:          33:          48:          63:          78:
  4: 004V9    19:          34:          49:          64:          79:
  5:          20:          35:          50:          65:          80:
……
```

При удалении приветствий предварительно надо удалять ссылку на него из вектора (можно заменить его номер на номер другого приветствия).
Как узнать, в каком векторе используется приветствие:

``` title="li usa ext 71015"
list usage extension 71015

                             LIST USAGE REPORT
Used By
Announcement
Vector               Vector Number        5        Step 3
```

Иначе при попытке удаления приветствия будет ошибка:

![](image228.png)

Временно заменяю в 5 векторе `71015` на `71012`.

``` title="ch ve 5" hl_lines="11"
change vector 5                                                 Page   1 of   6
                                  CALL VECTOR

    Number: 5                Name: 615-00-00 HG75099
Multimedia? y      Attendant Vectoring? n    Meet-me Conf? n           Lock? n
     Basic? y   EAS? y   G3V4 Enhanced? y   ANI/II-Digits? y   ASAI Routing? y
 Prompting? y   LAI? y  G3V4 Adv Route? y   CINFO? y   BSR? y   Holidays? y
 Variables? y   3.0 Enhanced? y
01 goto step    7             if ani              in     table 1
02 wait-time    2   secs hearing ringback
03 collect      5    digits after announcement 71012    for none
04 goto step    6             if digits           not-in table 2
05 route-to     digits with coverage y
06 route-to     number 75099            with cov y if unconditionally
07 disconnect   after announcement 71014
```
## Подключение к меню работы с аудиофайлами
На скриншоте видно, что соединение с CM активно, но экран для ввода команд (либо синий `Ctrl+E` либо графический `Ctrl+G`) закрыт. *System→ Announcements→ Announcements* или `Ctrl+A`.

![](image229.png)

При нажатии на `1 [G1]` видим список приветствий системы (аналог `list announcements`) и свойства. `Protected - n` означает, что приветствие можно удалить, перезаписать с телефона или перезалить файлы. При использовании в продакшн рекомендуется после тестов векторов установить `Protected - y`, чтобы кто-то случайно не изменил приветствия.

![](image230.png)
## Удаление приветствия (и аудиофайлов)
Для примера я скопировала тестовое приветствие к себе на компьютер. Клиенты смогли его залить на `002V9`, `003V9`, `004V9`, а на `001V9` его нет. Чтобы проверить как всё работает, сначала удалим его. Находясь в пункте `1 [G1]` жмем *Delete*, выбираем *Delete both*, т.к. я хочу удалить одновременно и файлы со шлюзов G450, и информацию о приветствии в CM.

![](image231.png)

Отмечаем галочками одинаковые приветствия на трех G450 (`002V9`, `003V9`, `004V9`) и это же приветствие без привязки к шлюзам (это запись о приветствии в СМ). Кнопкой с одинарной стрелкой перемещаем вправо, *Далее→ Далее→ Готово*. Внизу ожидаем статус *Complete* (см. выше, приветствие не должно быть задействовано в векторе).

![](image232.png)

![](image233.png)

![](image234.png)

Для правильного отображения текущего статуса приветствий в группе **G1** и на платах нажать *View*.
## Добавление приветствия
Находясь на `1 [G1]` жмем *Add*. Вводим номер `71015` и имя `Privetstvie6150000`

![](image235.png)

Жмем *Далее→ Далее→ Готово*. Ждем выполнения операции. Теперь жмем *View* находясь на `1 [G1 ]`. Как видно из скриншота, запись о приветствии создалась в CM, но в столбце **Length(sec)** длительность приветствия равна **0**, т.к. мы не залили файлы.

![](image236.png)

Заливаем файлы на медиашлюзы G450.
Переходим в раздел *Media-Gateways* и жмем на виртуальную плату `001V9` первого медиашлюза. Жмем *View*, видим список всех приветствий шлюза, жмем *Add*. Вводим номер `71015`, ставим галочку *Upload File* и выбираем файл `Privetstvie6150000.wav` в браузере.

![](image237.png)

Во время заливки приветствия внизу появится ошибка, что номер 71015 уже был назначен. Нам это не важно, но при добавлении приветствия, если не прописать номер, оно не зальётся. Такое только при использовании пула шлюзов.

![](image238.png)

Аналогично добавляем файл на `002V9`, `003V9`, `004V9`.

У меня правильная длительность приветствия после загрузки приветствия на все шлюзы отобразилась только после выхода и повторного входа в меню *Announcements*. Видим 15 секунд (округляется). Если же смотреть информацию на платах, то длительность приветствия не округляется и равна 15,1 секунд.

![](image239.png)

Теперь можно спокойно номер `71015` занести в вектор 5.

``` title="ch ve 5" hl_lines="11"
change vector 5                                                 Page   1 of   6
                                  CALL VECTOR

    Number: 5                Name: 615-00-00 HG75099
Multimedia? y      Attendant Vectoring? n    Meet-me Conf? n           Lock? n
     Basic? y   EAS? y   G3V4 Enhanced? y   ANI/II-Digits? y   ASAI Routing? y
 Prompting? y   LAI? y  G3V4 Adv Route? y   CINFO? y   BSR? y   Holidays? y
 Variables? y   3.0 Enhanced? y
01 goto step    7             if ani              in     table 1
02 wait-time    2   secs hearing ringback
03 collect      5    digits after announcement 71015    for none
04 goto step    6             if digits           not-in table 2
05 route-to     digits with coverage y
06 route-to     number 75099            with cov y if unconditionally
07 disconnect   after announcement 71014
```
