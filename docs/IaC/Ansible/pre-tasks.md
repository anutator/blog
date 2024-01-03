---
title: Определение порядка выполнения тасков в Ansible с помощью pre_tasks и post_tasks
share: "true"
---

Если задать **pre_tasks** в плейбуке, таски из него будут выполняться до всех остальных тасков (задач), включая роли. Если задать **post_tasks** в плейбуке, они будут выполняться после всего остального, включая все хендлеры, заданные в других тасках. Рассмотрим примеры полезного использования этих опций.

Можно использовать роли тремя способами:
- на уровне плея через опцию `roles` : это классический способ использования роли в плее.
- на уровне тасков через опцию включения роли `include_role`: можно переиспользовать роли динамически в любом месте секции тасков `tasks` плея.
- на уровне тасков через опцию импорта роли `import_role`: можно переиспользовать роль статически в любом месте в секции тасков `tasks` плея.
    
Стандартный способ включения ролей в плейбуке с помощью опции `roles`:

```yaml
---
- hosts: webservers
  roles:
  - common
  - webservers
```

При использовании опции `roles` на уровле плея, для каждой роли ‘x’:
- Если *roles/x/tasks/main.yml* существует, Ansible добавляет таски из этого файла в плей.
- Если *roles/x/handlers/main.yml* существует, Ansible добавляет хендлеры из этого файла в плей.
- Если *roles/x/vars/main.yml* существует, Ansible добавляет переменные из этого файла в плей.
- Если *roles/x/defaults/main.yml* существует, Ansible добавляет переменные из этого файла в плей.
- Если *roles/x/meta/main.yml* существует, Ansible добавляет все зависимости роли из этого файла в список ролей.
- Любые таски `copy`, `script`, `template` или `include` (в роли) могут ссылаться на файл роли из каталогов *roles/x/{files,templates,tasks}/* (каталог зависит от таски) необходимости задания абсолютного или относительного пути.

При использовании ролей `roles` на уровне плея в плейбуке, Ansible обрабатывает роли как статический импорт и вставляет их во время парсинга плейбука. Ansible выполняет каждый плей в следующем [порядке](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html#using-roles-at-the-play-level):
- Все таски из `pre_tasks` .
- Все хендлеры, которые триггерились тасками из pre_tasks.
- Каждая роль, перечисленная в `roles:`, выполняется в порядке перечисления сверху-вниз. Все зависимости ролей, заданные в файле `meta/main.yml` роли выполняются первыми, учитывая теги и условия. Смотреть  [использование зависимостей ролей](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html#role-dependencies) .
- Все таски `tasks`, прописанные в плее.
- Все хендлеры, которые триггерятся ролями или тасками из tasks.
- Все `post_tasks`, прописанные в плее.
- Все хендлеры, которые триггерятся из post_tasks.
## Управление хостами в балансировщике нагрузки
Мы всегда хотим минимизировать простои (перерывы) в продакшн, и многие архитекторы используют балансировку нагрузки для этого. Хорошая идея расположить системы бэкенда за балансировщиком и исключить конкретный бекенд из работы при выполнении работ, требующих перерывов, а по окончании работ вернуть его обратно в работу. Плейбук patch-webservers.yml выведет сервер из балансировщика HAProxy, выполнит обновления патчей и полную перезагрузку, а затем снова добавит сервер в балансировщик:

```yaml title="patch-webservers.yml"
---

- hosts: webservers
  pre_tasks:
    - name: Выключаем web сервер в балансировщике
      community.general.haproxy:
        state: disabled
        host: '{{ inventory_hostname_short }}'
        fail_on_not_found: yes
      delegate_to: loadbalancer.example.com
  roles:
    - full_patches
  post_tasks:
    - name: Включаем web сервер в балансировщике
      community.general.haproxy:
        state: enabled
        host: '{{ inventory_hostname_short }}'
        fail_on_not_found: yes
      delegate_to: loadbalancer.example.com
```

## Выполнение задач первичной настройки
Секция **pre_tasks** также полезна для задания фактов значениями, полученными во время прогона роли. Например, предположим, для некоторых из ролей нам надо получить последние доступные версии софта. Можно получить эту информация с сервера артифактов в секции **pre_tasks** и задать факт, который может быть доступен таскам нашей роли. В примере факт **latest_app_version** задан при вызове [API](https://www.redhat.com/en/topics/api/what-are-application-programming-interfaces) endpoint. Т.к. вызов API выполняется только если **latest_app_version** не задан, он все еще позволяет пользователю перезаписать этот факт, который затем становится доступным всем ролям плейбука:

```yaml title="software-version.yml"
---

- hosts: webservers
  pre_tasks:
    - name: Получить последнюю версию софта с сервера артифактов
      ansible.builtin.uri:
        url: http://artifact-server.example.com:8080/software/latest
        return_content: yes
      delegate_to: localhost
      register: uri_output
      when: latest_app_version is not defined

    - name: Задать факт latest_software_version
      ansible.builtin.set_fact:
        latest_app_version: "{{ uri_output.json.version }}"
      when: latest_app_version is not defined

    - name: Отобразить последнюю версию latest_app_version
      ansible.builtin.debug:
        msg: "{{ latest_app_version }}"
  roles:
    - appserver
    - apache
    - monitored_host
```

## Временное отключение системы от мониторинга
Пример похож на пример с балансировщиком нагрузки. Перед началом работ надо исключить хост из мониторинга, чтобы не создавать ложных алармов. По окончании работ хост включается в мониторинг. Если система мониторинга поддерживает HTTP запросы, можно создать такой плейбук:

```yaml title="patch-databases.yml"
---
- hosts: database_servers
  pre_tasks:
    - name: Выключение хоста из мониторинга
      ansible.builtin.uri:
        url: "http://monitoring-server.example.com:8080/hosts/{{ inventory_hostname }}/schedule_downtime"
        method: POST
        body_format: json
        body:
          downtime_duration: 30m
      delegate_to: localhost
  roles:
    - full_patches
  post_tasks:
    - name: Включение хоста в мониторинг
      ansible.builtin.uri:
        url: "http://monitoring-server.example.com:8080/hosts/{{ inventory_hostname }}/clear_downtime"
        method: POST
        body_format: json
      delegate_to: localhost
```

## Использование включений (includes)
В предыдущих примерах таски прописаня прямо в плейбуке. Для одной-двух тасков это удобно, но при использовании большого количества тасков и тем более если эти таски используются в разных плейбуках, лучше сделать их переиспользуемыми, для чего используем [include](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_module.html) . В примере задачи (таски) мониторинга расположены в их собственном каталоге `tasks/`. Это делает код чище и позволяет переиспользовать код:

```yaml title="patch-databases-with-include.yml"
---
- hosts: database_servers
  pre_tasks:
    - name: Исключение хоста их мониторинга
      ansible.builtin.include: tasks/monitoring/silence-host.yml
  roles:
    - full_patches
  post_tasks:
    - name: Включение хоста в мониторинг
      ansible.builtin.include: tasks/monitoring/enable-host.yml
```

Note that this `tasks/` directory is in the same directory as my main playbook and **not** the `tasks/` directory under any particular role. To make this point clearer, the directory structure looks like this:

```plaintext
$ tree --noreport
├── ansible.cfg
├── inventory.ini
├── patch-databases-with-include.yml
├── patch-databases.yml
├── patch-webservers.yml
├── roles
│   ├── apache
│   │   └── tasks
│   │       └── main.yml
│   ├── appserver
│   │   └── tasks
│   │       └── main.yml
│   ├── full_patches
│   │   └── tasks
│   │       └── main.yml
│   └── monitored_host
│       └── tasks
│           └── main.yml
├── software-version.yml
├── tasks
│   └── monitoring
│       ├── enable-host.yml
│       └── silence-host.yml
└── templates
    └── hosts.j2
```

[Источник](https://www.redhat.com/sysadmin/ansible-pretasks-posttasks)