---
tags:
  - ansible
share: "true"
title: Отображение результата выполнения команды или скрипта
---

К сожалению, в реальном времени отображать выполнение скрипта в Ansible не получается. Результат все же можно сохранить в переменную и далее в файл. При сохранении в файл на экране тоже отобразится. Пока использую `with_items`, как это делать с `loop`, надо отдельно тестировать, просто так заменить на `loop` нельзя.
## Выполнить на локальном сервере

### Выполнить команду
Выполним поиск файлов с расширением `*.yml` используя модуль `command`

```yaml title="find_results.yml"
- name: Play to run find command and capture its output to a file
  hosts: 127.0.0.1
  connection: local
  tasks:
  - name: 'Run find command to fetch file rights {{inventory_hostname}}'
    command: "find / -type f -name '*.yml'"
    register: find_results
    become: true
    become_user: root
    become_method: sudo

  - name: Print to verify it works
    debug:
      msg: '{{find_results.stdout}}'

  - name: Use copy module to create the file using output from the previous command.
    copy:
      dest: find_results.txt   # сохранится в каталог, где выполняется плейбук. Можно задать абсолютный путь /tmp/find_results.txt
      content: "{{ item }}"
    with_items: "{{ find_results.stdout }}"
    delegate_to: localhost
```

### Выполнить скрипт

```ini title="inventory"
[localhost]
localhost
```

Пример простого скрипта `/tmp/foo`, отображающего два слова foo и bar.

```bash title="/tmp/foo"
#!/usr/bin/env bash
echo "foo"
echo "bar"
```

Дали скрипту права на исполнение:

```bash
$ chmod +x /tmp/foo
```

Плейбук `debug.yml`

```yaml title="debug.yml"
- hosts: localhost
  tasks:
  - shell: /tmp/foo
    register: foo_result
    ignore_errors: True
  - local_action: copy content= dest=script_result.txt
```

Выполнение:

```bash
$ ansible-playbook -i inventory debug.yml

PLAY [localhost] ********************************************************************************************************************************************************************

TASK [shell] ************************************************************************************************************************************************************************
changed: [localhost]

TASK [copy] *************************************************************************************************************************************************************************
changed: [localhost -> localhost]

PLAY RECAP **************************************************************************************************************************************************************************
localhost                  : ok=2    changed=2    unreachable=0    failed=0

$ cat script_result.txt
foo
bar
```

## Выполнить команду на удаленном сервере и сохранить результат там или локально

### Выполнить команду

```yaml title="remote_find_results.yml"
- name: Play to run find command and capture its output to a file
  hosts: my-test-host
  tasks:
  - name: 'Run find command to fetch file rights {{inventory_hostname}}'
    command: "find / -type f -name '*.yml'"
    register: find_results
    become: true
    become_user: root
    become_method: sudo

  - name: Print to verify it works
    debug:
      msg: '{{find_results.stdout}}'

  - name: Use copy module to create the file using output from the previous command.
    copy:
      dest: find_results.txt   # сохранится в каталог, где выполняется плейбук. Можно задать абсолютный путь /tmp/find_results.txt
      content: "{{ item }}"
    with_items: "{{ find_results.stdout }}"  # осторожно, просто на loop заменить нельзя
    delegate_to: localhost    # без этого параметра сохранит в файл на сервере, где выполняется команда поиска
```

### Выполнить скрипт
Выполняет на удаленном сервере скрипт, заданный в переменной `install_script`, файл вывода скрипта сохраняет во временном каталоге, заданном через переменную `temp_dir`. В этом же каталоге лежит и сам скрипт `install_script`.

```yaml title="remote_script_results.yml"
- name: Install | Run script {{ install_script }}
  shell: "./{{ install_script }}"
  args:
    chdir: "{{ temp_dir }}"
  register: install

- name: Copy install log to file {{ temp_dir }}/install.log
  copy:
    dest: "{{ temp_dir }}/install.log"
    content: "{{ item }}"
  with_items: "{{ install.stdout }}"
```