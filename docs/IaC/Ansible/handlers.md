---
tags:
  - ansible
share: "true"
---
**handler** обычно используется для перезапуска каких-то сервисов при изменении файлов конфигурации. handler выполняются в самом конце плейбука.
Плюсы: если несколько событий notify вызвали один и тот же handler, он выполнится только один раз (не будет нескольких перезагрузок).
Про минусы handler надо читать отдельно: например, если какая-то таска завершилась с ошибкой, при этом она не имеет отношения к handler, все равно он не будет выполнен, т.к. плейбук не выполнился до конца. При повторном выполнении плейбука задача, где менялся файл конфигурации, уже не будет иметь статус changed, поэтому при повторном выполнении handler так же не будет выполнен. Проблему можно обойти с помощью flush handlers.
## Универсальный handler и Loop
Как сделать универсальный handler, если например есть задача, которая через loop меняет файл одного из сервисов (Ansible сверяет контрольные сумммы и изменяет только те файлы, которые отличаются от файлов на сервере). При этом мы хотим перегрузить только сервис, конфигурация которого изменилась, а отдельные handler для каждого делать не хотим.

```yaml title="tasks/main.yml"
# одна из тасков. Копируем шаблоны. Добавляем register и сохраняем результат выполнения команды в переменную 'podman_services'
- name: create podman-compose files for services
  template:
    src: "{{ item }}/{{ item }}.yml.j2"
    dest: "~/{{ item }}/{{ item }}.yml"
  loop: "{{ services }}"
  register: podman_services
  notify: restart podman service
```

```yaml title="handlers/main.yml"  
 # в handlers/main.yml используем loop, где вытаскивает из переменной 'podman_services' только список измененных сервисов
 # Перезапуск от лица пользователя с его переменными окружения
- name: restart podman service
  become_user: "{{ user_name }}"
  become_flags: -iS
  systemd:
    name: "podman-compose@{{ item }}"
    scope: user
    state: restarted
  loop: "{{ podman_services.results | selectattr('changed') | map(attribute='item') | list }}"
```

## Несколько задач на один хендлер
### Способ 1 — общий listen
handlers могут использовать директиву прослушивания “listen”. Пример не совсем жизненный, т.к. здесь можно было в handler использовать одну task с loop, где перечислены memcached и apache, но общая схема ясна — слушаем один и тот же notify. Плюс в том, что хендлер можно использовать как целиком, перезапуская 2 сервиса, так и использовать каждый хендлер в отдельности

```yaml
tasks:
  - name: restart everything
    command: echo "this task will restart the web services"
    notify: "restart web services"

....
handlers:
  - name: restart memcached
    service:
      name: memcached
      state: restarted
    listen: "restart web services"
  
  - name: restart apache
    service:
      name: apache
      state: restarted
    listen: "restart web services"
```

Вот такой пример handler с двумя тасками более жизненный. Здесь вторая таска зависит от первой:

```yml
- name: Check if restarted
  shell: check_is_started.sh
  register: result
  listen: Restart processes

- name: Restart conditionally step 2
  service:
    name: service
    state: restarted
  when: result
  listen: Restart processes
```

В задаче пишем `notify: Restart processes` для вызова хендлера.

Source:
-   https://stackoverflow.com/a/43451315/3151055
-   http://docs.ansible.com/ansible/playbooks_intro.html#handlers-running-operations-on-change

### Способ 2 — перечисление хендлеров

```yaml title="handlers/main.yml"
# handlers file for zabbix_agent
- name: ensure selinux tools are installed
  ansible.builtin.package:
    name:
    - checkpolicy
    - policycoreutils-python
    state: latest

- name: create selinux mod for zabbix_agent
  ansible.builtin.command: checkmodule -M -m -o /etc/zabbix/my-zabbixagent.mod /etc/zabbix/my-zabbixagent.te

- name: create selinux pp for zabbix_agent
  ansible.builtin.command: semodule_package -o /etc/zabbix/my-zabbixagent.pp -m /etc/zabbix/my-zabbixagent.mod

- name: load selinux pp for zabbix_agent
  ansible.builtin.command: semodule -i /etc/zabbix/my-zabbixagent.pp

...
```

Перечисляем несколько хенлеров в *tasks/main.yml*:

```yaml title="tasks/main.yml"
- name: place selinux type enforcement
  ansible.builtin.copy:
    src: my-zabbixagent.te
    dest: /etc/zabbix/my-zabbixagent.te
    mode: "0644"
  notify:
    - ensure selinux tools are installed
    - create selinux mod for zabbix_agent
    - create selinux pp for zabbix_agent
    - load selinux pp for zabbix_agent
  when:
    - ansible_selinux.status is defined
    - ansible_selinux.status == "enabled
```

Выполнение:

```bash
TASK [robertdebock.zabbix_agent : place selinux type enforcement] ************************************************************
Friday 06 August 2021  00:43:24 +0300 (0:00:00.824)       0:00:13.035 ********* 
changed: [preprod-db1]

TASK [robertdebock.zabbix_agent : start and enable zabbix agent] *************************************************************
Friday 06 August 2021  00:43:24 +0300 (0:00:00.713)       0:00:13.749 ********* 
ok: [preprod-db1]

RUNNING HANDLER [robertdebock.zabbix_agent : ensure selinux tools are installed] *********************************************
Friday 06 August 2021  00:43:25 +0300 (0:00:00.600)       0:00:14.349 ********* 
changed: [preprod-db1]

RUNNING HANDLER [robertdebock.zabbix_agent : create selinux mod for zabbix_agent] ********************************************
Friday 06 August 2021  00:43:30 +0300 (0:00:05.444)       0:00:19.794 ********* 
changed: [preprod-db1]

RUNNING HANDLER [robertdebock.zabbix_agent : create selinux pp for zabbix_agent] *********************************************
Friday 06 August 2021  00:43:31 +0300 (0:00:00.446)       0:00:20.240 ********* 
changed: [preprod-db1]

RUNNING HANDLER [robertdebock.zabbix_agent : load selinux pp for zabbix_agent] ***********************************************
Friday 06 August 2021  00:43:31 +0300 (0:00:00.294)       0:00:20.535 ********* 
changed: [preprod-db1]

PLAY RECAP *******************************************************************************************************************
preprod-db1                : ok=16   changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

##  meta
- Meta tasks are a special kind of task which can influence Ansible internal execution or state.  
- Meta tasks can be used anywhere within your playbook.
- This module is also supported for Windows targets.
- `meta` is not really a module nor action_plugin as such it cannot be overwritten.
- `clear_facts` will remove the persistent facts from [ansible.builtin.set_fact](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/set_fact_module.html#ansible-collections-ansible-builtin-set-fact-module) using `cacheable=True`, but not the current host variable it creates for the current run.
- Looping on meta tasks is not supported.
- Skipping `meta` tasks with tags is not supported before Ansible 2.11.
 
This module takes a free form command, as a string. There is not an actual option named "free form". See the examples!
- `flush_handlers` makes Ansible run any handler tasks which have thus far been notified. Ansible inserts these tasks internally at certain points to implicitly trigger handler runs (after pre/post tasks, the final role execution, and the main tasks section of your plays).
- `refresh_inventory` (added in Ansible 2.0) forces the reload of the inventory, which in the case of dynamic inventory scripts means they will be re-executed. If the dynamic inventory script is using a cache, Ansible cannot know this and has no way of refreshing it (you can disable the cache or, if available for your specific inventory datasource (e.g. aws), you can use the an inventory plugin instead of an inventory script). This is mainly useful when additional hosts are created and users wish to use them instead of using the [ansible.builtin.add_host](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/add_host_module.html) module.
- `noop` (added in Ansible 2.0) This literally does 'nothing'. It is mainly used internally and not recommended for general use.
- `clear_facts` (added in Ansible 2.1) causes the gathered facts for the hosts specified in the play's list of hosts to be cleared, including the fact cache.
- `clear_host_errors` (added in Ansible 2.1) clears the failed state (if any) from hosts specified in the play's list of hosts.
- `end_play` (added in Ansible 2.2) causes the play to end without failing the host(s). Note that this affects all hosts.
- `reset_connection` (added in Ansible 2.3) interrupts a persistent connection (i.e. ssh + control persist)
- `end_host` (added in Ansible 2.8) is a per-host variation of `end_play`. Causes the play to end for the current host without failing it.

```yaml
# Выполнение хендлеров по требованию, а не в конце плея
- template:
    src: new.j2
    dest: /etc/config.txt
  notify: myhandler

- name: Force all notified handlers to run at this point, not waiting for normal sync points
  meta: flush_handlers

# Как обновить инвентори во время выполнения плея
- name: Reload inventory, useful with dynamic inventories when play makes changes to the existing hosts
  cloud_guest:            # this is fake module
    name: newhost
    state: present

- name: Refresh inventory to ensure new instances exist in inventory
  meta: refresh_inventory

# Как очистить все существующие факты на целевых хостах
- name: Clear gathered facts from all currently targeted hosts
  meta: clear_facts

# Как продолжить выполнение на хостах плея даже если там есть ошибка
- name: Bring host back to play after failure
  copy:
    src: file
    dest: /etc/file
  remote_user: imightnothavepermission

- meta: clear_host_errors

# Как сбросить существующее соединение, здесь поменяли свойства пользователя ansible_user (того, под кем логинится ansible) и надо перелогиниться, чтобы изменения вступили в силу
- user:
    name: '{{ ansible_user }}'
    groups: input

- name: Reset ssh connection to allow user changes to affect 'current login user'
  meta: reset_connection

# Как завершить выполнение плея на особых хостах, например здесь плей завершается, если операционная система CentOS6. При этом ошибка не генерируется.
- name: End the play for hosts that run CentOS 6
  meta: end_host
  when:
  - ansible_distribution == 'CentOS'
  - ansible_distribution_major_version == '6'
```

