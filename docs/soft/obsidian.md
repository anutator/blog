---
tags:
  - soft
share: "true"
title: База знаний Obsidian
---
https://obsidian.md/download — скачать пакет для своей ОС, например `.deb`.
https://github.com/OliverBalfour/obsidian-pandoc/issues/118

Плагины:
- Шифрование: Meld Encrypt (я использую его, шифрует выделенный текст или указанный файл), Cryptsidian. Не плагины, шифровать полностью каталог: [veracrypt](https://www.veracrypt.fr/code/VeraCrypt/), [cryptomator](https://cryptomator.org/).
- Экспорт из Obsidian в Word: скачать свежую версию [pandoc](https://github.com/jgm/pandoc/releases) для Ubuntu,  [pandoc-3.1.2-1-amd64.deb](https://github.com/jgm/pandoc/releases/download/3.1.2/pandoc-3.1.2-1-amd64.deb), установить. Установить в Obsidian плагин Pandoc plugin. В редакторе `Ctrl+P` (в Mac `Cmd+P`), начать вводить Pan, выбрать куда надо конвертировать.
- [Github Publisher](https://github.com/ObsidianPublisher/obsidian-github-publisher) — публикация избранных статей через Material for MkDocs. [Документация](https://obsidian-publisher.netlify.app/). Этот сайт публикуется этим плагином.
- Пробелы типа `&nbsp` или `&emsp` пришлось убрать через плагин VSCode [fix irregular whitespace](https://github.com/karlito40/fix-irregular-whitespace).
  https://jkorpela.fi/chars/spaces.html
  https://allalmohamedlamine.medium.com/fixing-and-removing-irregular-spaces-984389e17132
  http://shpargalkablog.ru/2016/02/space-html.html

Пример списка плагинов:
SYSTEM INFO:
Obsidian version: v0.15.9
Installer version: v0.15.9
Operating system: Darwin Kernel Version 21.6.0: Sat Jun 18 17:07:22 PDT 2022; root:xnu-8020.140.41~1/RELEASE_ARM64_T6000 21.6.0
Login status: logged in
Catalyst license: insider
Insider build toggle: off
Live preview: on
Legacy editor: off
Base theme: light
Community theme: Typomagical
Snippets enabled: 0
Restricted mode: off
Plugins installed: 32
Plugins enabled: 31
1: Advanced Tables v0.17.3
2: Sliding Panes (Andy’s Mode) v3.3.0
3: Dataview v0.5.41
4: Templater v1.12.0
5: Natural Language Dates v0.6.1
6: Note Refactor v1.7.1
7: Paste URL into selection v1.7.0
8: Better Word Count v0.8.1
9: Tag Wrangler v0.5.3
10: Ozan’s Image in Editor Plugin v2.1.3
11: Find orphaned files and broken links v1.6.1
12: PDF to Markdown v0.0.7
13: Dictionary v2.21.1
14: Folder Note v0.7.3
15: Calendar v1.5.10
16: Mind Map v1.1.0
17: Checklist v2.2.7
18: Emoji Toolbar v0.3.1
19: Tasks v1.11.1
20: Hotkeys for templates v1.4.3
21: Obsidian Orthography v2.0.4
22: Footnote Shortcut v0.0.9
23: Tidy Footnotes v0.1.1
24: Periodic Notes v0.0.17
25: Readwise Official v2.0.1
26: Smart Typography v1.0.18
27: Wikilinks to MDLinks v0.0.12
28: cMenu v1.1.2
29: Auto Link Title v1.2.5
30: Local REST API v1.3.9
31: Excalidraw v1.7.12

RECOMMENDATIONS:
Custom theme: for cosmetic issues, please first try updating your theme to latest. If still not fixed, please try to make the issue happen in the Sandbox Vault or disable community theme and snippets.
Community plugins: for bugs, please first try updating all your plugins to latest. If still not fixed, please try to make the issue happen in the Sandbox Vault or disable community plugins.

## Изменение ширины экрана
Сделать ширину на полный экран
В проекте в *docs/assets/css/custom_attributes.css* добавить

```css title="docs/assets/css/custom_attributes.css"
.md-grid {
  max-width: initial;
}
```

Этот файл уже включен в *mkdoc.yml* в `extra_css`, поэтому ничего дополнительно редактировать не надо.

## Pandoc Plugin
Как настроить Pandoc Plugin для импорта картинок, если картинки храним не в корневом каталоге vault, а во вложенных каталогах внутри каждого подкаталога. Например у меня в настройках Obsidian картинки хранятся в подкаталоге `assets`. Т.е. я создаю раздел. Obsidian создаёт каталог с названием раздела, например DevOps, Gitlab, Docker, Cisco. Внутри этих каталогов тоже могут быть подкаталоги. Я пишу статью, создается файл `.md`, а картинки хранятся в каталоге `assets`, который находится там же, где файлы `.md`.

Зайдем в основной каталог vault и в скрытом каталоге `.obsidian`, где хранятся настройки Obsidian и настройки плагинов, откроем настройки плагина Pandoc.
`vim .obsidian/plugins/obsidian-pandoc/main.js `

По слову `inputFile` ищем кусок кода
`file: inputFile, format: 'markdown',` и меняем его на
`file: inputFile, format: 'markdown+rebase_relative_paths',`.

```js
    case 'md': {
        const result = yield pandoc({
            file: inputFile, format: 'markdown',
            pandoc: this.settings.pandoc, pdflatex: this.settings.pdflatex
        }, { file: outputFile, format }, this.settings.extraArguments.split('\n'));
        error = result.error;
        command = result.command;
        break;
    }
```

Получаем такое:

```js
       case 'md': {
          const result = yield pandoc({
              file: inputFile, format: 'markdown+rebase_relative_paths',
              pandoc: this.settings.pandoc, pdflatex: this.settings.pdflatex
          }, { file: outputFile, format }, this.settings.extraArguments.split('\n'));
          error = result.error;
          command = result.command;
          break;
      }   
```

```js
    args.push('--resource-path');
    args.push(path.dirname(input.file) + "/assets");
    args.push('--resource-path');
    args.push("/home/bestann/Documents/bestann_docs/GitLab/assets");
```

Обязательно нужен этот плагин [Obsidian Link Converter](https://github.com/ozntel/obsidian-link-converter) (смотреть [issue](https://github.com/jgm/pandoc/issues/8853)) , найти в списке плагинов.

```bash
# Проверить свежие версии https://github.com/typst/typst/releases
wget https://github.com/typst/typst/releases/download/v0.3.0/typst-x86_64-unknown-linux-musl.tar.xz
sudo tar xf typst-x86_64-unknown-linux-musl.tar.xz -C /usr/local/bin --strip-components=1
```

Перевести word в markdown и сохранить картинки в каталог media:

```bash
pandoc -o output.md --extract-media=./ inputfile.docx
```
## Поиск в Obsidian
- можно ли в Obsidian сделать поиск только по каталогу, в котором сейчас нахожусь. Например у меня слева в панели каталоги Ansible, Docker, Git, Kubernetes, Linux и пр. (на диске они являются каталогами, а внутри каждого из них .md файлы). Например мне надо искать слово 'git' в каталоге Ansible, а не по всему vault: `path:Ansible git`
- по умолчанию поиск зависит от регистра, например поиск Git даст один результаты, поиск git — другие. Выключить зависимость от регистра — кнопка `Aa`.
- [Плагин Obisidian Outliner](https://github.com/vslinko/obsidian-outliner) для работы с таблицами
- Показать в правой панели навигацию по заголовкам документа как в Word — плагин Outline. Проверить, что он включен в Core Plugins. Один раз вызвать его для текущего документа `Ctrl+S`. И перетащить мышкой из отдельной панели в правую панель, где теги, обратные ссылки и пр.

## Синхронизация с помощью Rclone
У Obsidian есть встроенная возможность синхронизации, но это платный функционал. Поэтому можно либо архивировать весь свой каталог в zip и переносить вручную между устройствами (для смартфона также есть приложение Obsidian), либо поставить [Rclone](https://rclone.org/)  — синхронизацию локальных папок с облаками google, dropbox и другими (полный список [здесь](https://rclone.org/#providers)).  При использовании шифрования можно не переживать за конфиденциальность данных.

Пример настройки сихронизации через *google docs* (у кого есть почта на *gmail*):

```bash
# настраиваем хранилище, даем ему имя my
rclone config
# первичное копирование локального каталога в хранилище (можно сразу использовать команду синхронизации)
rclone copy /media/bestann/Toshiba/LINUX/00_DevOps/bestann_docs my:bestann_docs

rclone ls my:bestann_docs
rclone ls my:

# не путать!! Здесь мы изменения с локальной папки переносим в Google docs
# -i — флаг для интерактивности, чтобы не накосячить. Подтверждаем изменение каждого файла!!! Для скорости убрать флаг.
# rclone sync папка_ОТКУДА_копируем папка_КУДА_вносим_изменения
# все измененные и удаленные локально файлы так же будут изменены на удаленном хранилище
rclone sync -i /media/bestann/Toshiba/LINUX/00_DevOps/bestann_docs my:bestann_docs

# если наоборот хотим из облака принести себе на компьютер изменения:
rclone sync -i my:bestann_docs /media/bestann/Toshiba/LINUX/00_DevOps/bestann_docs

# Чтобы не перепутать синхронизацию, сделать алиасы
alias rbackup="rclone sync /media/bestann/Toshiba/LINUX/00_DevOps/bestann_docs my:bestann_docs"
alias rrestore= "rclone sync -i my:bestann_docs /media/bestann/Toshiba/LINUX/00_DevOps/bestann_docs"
```

## Дебаг плагинов
(перевод)
I would like to share some tricks. Some of them I even discovered on my own, as I could not find them by googling.
As Obsidian is an Electron app, it’s essentially the wrapper on Google Chrome, so if you know how to debug things there, you already know the most essential parts.
Нажать `Ctrl + Shift + I` для открытия `Developer Tools`.

### Plugin ID
Your plugin logic resides in `[Vault Path]\.obsidian\plugins\[Plugin ID]\main.js`
Usually if you know the plugin name, it’s simple to to find the `Plugin ID` but if it is not the case, just go to `Settings > Community Plugins > [Plugin Name] .> Copy share link`

![](image-2022-05-10-12-29-01.png)

And the link looks like `obsidian://show-plugin?id=[Plugin Id]`

### Debug without sources
Plugins are loaded dynamically using `eval` JavaScript function, therefore it is impossible to search for the plugin code during execution in `Sources` pane in `Developer Tools`.

So this requires a little trick. Go to `Developer Tools` and type in the console

```
app.plugins.plugins["[Plugin ID]"].constructor
```

Here obviously `[Plugin ID]` has to be replaced with the real `Plugin ID`.

As the result you will get a function in the console

![](image-2022-05-10-12-57-24.png)

If you click on that function, you will be redirected to the `Sources` pane with the code of `eval`\-ed plugin.

![](image-2022-05-10-15-16-17.png)

Here you can debug normally, set breakpoints etc.

If you want to make permanent changes you can modify `[Vault Path]\.obsidian\plugins\[Plugin ID]\main.js` accordingly.

### TypeScript sources
Actually the main language for Obsidian plugins is TypeScript. It is being later on compiled into JavaScript `[Vault Path]\.obsidian\plugins\[Plugin ID]\main.js`.

For more advanced changes, dealing with compiled JavaScript is troublesome. Async functions are being compiled into JavaScript generator functions. Overall, the experience is not as pleasant.

To get TypeScript sources you have to find `[plugin-repo-url]`

![](image-2022-05-10-15-28-38.png)

```bash
git clone [plugin-repo-url]
cd [plugin-dir]
npm install # installs all the dependencies required to compile the plugin
npm run dev # start the daemon that watches changes in the TypeScript source code and compiles them into main.js
```

Then you can make your changes, `main.js` will be automatically recompiled every time you save TypeScript changes.

When you are done, copy `main.js` into `[Vault Path]\.obsidian\plugins\[Plugin ID]\main.js`
### Reload changed plugin
You have multiple ways to reload the plugin after changes
1.  Just restart Obsidian app.
2.  Manually turn on and off the plugin

![](image-2022-05-10-15-36-21.png)

3.  Reload all plugins at once

![](image-2022-05-10-15-37-08.png)

4.  Reload via code in `Developer Tools` console.

```js
await app.plugins.disablePlugin("[Plugin ID]");
await app.plugins.enablePlugin("[Plugin ID]");
```

5.  Use `Hot Reload` plugin
[Hot Reload](https://github.com/pjeby/hot-reload) plugin automatically reloads plugins under certain conditions. See its documentation.
Also note that this plugin doesn’t exist in the standard community plugins registry, so it has to be installed manually.