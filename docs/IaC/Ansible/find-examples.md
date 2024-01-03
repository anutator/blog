---
tags:
  - ansible
share: "true"
title: Поиск файлов локально и удаленно
---

## Нерекурсивный поиск файлов локально в заданном каталоге/каталогах. with_fileglob.
Ищет все файлы в одном каталоге, совпадающие с паттерном, нерекурсивно. Вызывает библиотеку glob Python-а.
- Шаблоны только для поисков файлов, нельзя искать каталоги или большие пути к каталогам.
- Поиск ведется на контроллере Ansible, т.е. на самой машине, где запускается плейбук.  Для итерации списка файлов на удаленной ноде использовать модуль [ansible.builtin.find](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/find_module.html#ansible-collections-ansible-builtin-find-module).    
- Список найденных файлов будет перечислен через запятую, либо будет пустой список, если файлы не найдены. Для реального списка добавить `wantlist=True` в строку поиска lookup.

```yaml
# показать все файлы .txt в каталоге /my/path
- name: Display paths of all .txt files in dir
  debug: msg={{ lookup('fileglob', '/my/path/*.txt') }}

# скопировать каждый найденный файл, который соответствует заданному шаблону поиска, в каталог /etc/fooapp/
- name: Copy each file over that matches the given pattern
  copy:
    src: "{{ item }}"
    dest: "/etc/fooapp/"
    owner: "root"
    mode: 0600
  with_fileglob:
    - "/playbooks/files/fooapp/*"
```

There are scenarios where instead of being explicit about each file name when generating templates from a role, you may want to take all the suffixed files in the ‘templates’ directory and dump their generated content into a destination directory.

As a quick example, if you are in a role and want every script in the templates directory ending with ‘.sh’ to generate a file then use ‘with_fileglob’ like below.

```yaml
 - name: create file out of every file in template directory
   template:
     src: "{{item}}"
     dest: /tmp/.
   with_fileglob: "{{role_path}}/templates/*.sh"
```

If your templates have an additional suffix tacked on (such as ‘.j2’), then you can use the jinja2 `regex_replace` to strip it off like below.

```yaml
 - name: create file out of every jinja2 template
   template:
     src: "{{item}}"
     dest: /tmp/{{ item | basename | regex_replace('\.j2$', '') }}
   with_fileglob: "{{role_path}}/templates/*.j2"
```

Here is a link to a playbook using the role, which can be invoked locally for testing:

```bash
ansible-playbook playbook-fileglob.yml --connection=local
```

Полная роль [fileglobtest](https://github.com/fabianlee/blogcode/tree/master/ansible/roles/fileglobtest).

### Поиск в нескольких известных каталогах. with_fileglob

В переменную with_fileglob можно передать список каталогов

```yaml
defaults/main.yml
---
work_dir:
- ../templates/*.j2
- ../templates/prod/*.j2

 - name: create file out of every file in template directory
   template:
     src: "{{item}}"
     dest: /tmp/.
   with_fileglob: "{{work_dir}}"
```

### Поиск в нескольких каталогах
Здесь добавить пример через `set_fact`.

## Поиск файлов на удаленном хосте
### Поиск в одном каталоге. find.
К сожалению, `with_fileglob` работает только с файлами хосте, который является контроллером Ansible. Он не будет составлять список файлов из удаленного каталога, даже если вы используете что-то типа модуля [copy](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html) с параметром `remote_source: true`.

В таком случае можно использовать модуль [find](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/find_module.html).  Соберем список картинок из каталога `/tmp` на удаленном хосте. В переменной `image_list `содержится словарь со списком файлов в `.files`.  И затем внутри этого списка в “`.path`” содержится полный путь файла. Копируем все найденные картинки в веб каталог `/var/www/html`.

```yaml
- name: get images in remote /tmp
  find:
    paths: "/tmp"
    file_type: file
    patterns: '*.png,*.jpg,*.jpeg'
  register: image_list

# Для изучения. Смотрим результат, что содержится в переменной image_list.
- debug: msg="{{image_list}}"

- name: copy each image to the Apache2 html directory
  copy:
    remote_source: yes
    src: "{{item.path}}"
    dest: "/var/www/html"
  loop: "{{ image_list.files }}"
```

В jinja2 есть фильтр «базовое имя» ([basename](https://docs.ansible.com/ansible/2.3/playbooks_filters.html#id24)), который выводит только имя файла из всего полного пути.

```bash
{{ item.path | basename }}
```
### Поиск файлов в разных каталогах
В предыдущем примере мы искали файлы только в каталоге `/tmp`. Усложним задачу. Параметр [`paths`](https://docs.ansible.com/ansible/latest/modules/find_module.html#parameter-paths) модуля **find** может принимать список путей, достаточно их добавить в переменную, например `work_dir`. Этот способ для моего следующего примера с переменной `services` не подходит, т.к. у нас нет готового списка каталога — он формируется при использовании дополнительной переменной `home`. Подошло бы, если бы был готовый список `work_dir`:

```yaml
work_dir:
- /home/tradematic/api
- /home/tradematic/backtest
- /home/tradematic/dataserver
- /home/tradematic/other
- /home/tradematic/execution
- /tmp

tasks/logs.yml
# поиск в нескольких каталогах по паттерну
- name: Find file in folders from variable 'work_dir'
  find:
    paths: "{{ work_dir }}"
    use_regex: yes
    patterns:
    - '.*\.\d+\.\d{4}-\d{2}-\d{2}@\d{2}:\d{2}:\d{2}~$'
    age: 1d
    recurse: yes
  register: fileToDelete

# Удаление найденных файлов
- name: Delete file
  file:
    path: "{{ item.path }}"
    state: absent
  loop: "{{ fileToDelete.files }}"
```

### Поиск файлов в разных каталогах. find & loop
Поиск всех архивов `*.tgz` в домашнем каталоге пользователя tradematic в подкаталогах, заданных в переменной `services` (список одновременно является списком сервисов, он может быть разным на разных хостах). Здесь самое сложное — правильно распарсить найденные имена файлов из списка `tgzlist`.

Переменные в *defaults/main.yml*:

```yaml title="defaults/main.yml"
home: /home/tradematic

services:
- api
- backtest
- dataserver
- other
- execution
```

Часть задач, относящихся к поиску логов. Здесь команда `make logs` вызывает скрипт из файла Makefile (отдельно есть информация по этому файлу)

```yaml title="tasks/logs.yml"
# Сбор логов. Скрипт из файла Makefile собирает логи в архивы *.tgz
- name: Make logs for {{ user_name }}
  become_user: "{{ user_name }}"
  become_flags: -iS
  shell: make logs
  args:
    chdir: "{{ home }}/{{ item }}"
  loop: "{{ services }}"

# поиск в нескольких каталогах
- name: register all logs-*.tgz files
  find:
    path: "{{ home }}/{{ item }}"
    recurse: no
    patterns: logs-*.tgz
  loop: "{{ services }}"
  register: tgzlist

# Опционально для изучения. Смотрим результат: что содержится в переменной tzglist.
- debug: msg="{{tgzlist}}"

# должен быть установлен jmespath
# копировать в каталог logs, который находится не внутри playbooks, а на уровне playbooks
- name: Copy collected logs *.tgz to localhost
  fetch:
    src: "{{ item }}"
    dest: ../logs/
    flat: yes
  loop: "{{ tgzlist | json_query('results[*].files') | flatten | map(attribute='path') }}"
```

Как привести это к предыдущему виду без loop:
- через `set_fact` создать переменную `work_dir`, содержащую готовый список каталогов
- loop убрать, поставить в `path` переменную `work_dir`.

Ещё пример. Аналогично предыдущему, но более грязный вывод экрана, т.к. отображается намного больше чем когда оставили только `map(attribute='path')`.

```yaml
- name: Find file
  find:
    paths: "{{ item }}"
    use_regex: yes
    patterns:
    - '.*\.\d+\.\d{4}-\d{2}-\d{2}@\d{2}:\d{2}:\d{2}~$'
    age: 1d
    recurse: yes
  register: fileToDelete
  loop: "{{ fileToFindInAllSubDirecotry }}"

- name: Delete file
  file:
    path: "{{ item.path }}"
    state: absent
  loop: "{{ fileToDelete | json_query('results[*].files') | flatten }}"
```



