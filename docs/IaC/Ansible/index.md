---
title: Ansible Cheetsheat
share: "true"
---

## Модули, которые необходимо знать
Модули, которые надо знать:
- Работа непосредственно с командной строкой Linux, разница между  `command` (модуль по умолчанию в Ansible), `raw`, `shell`, [`script`](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/script_module.html). Расширенный модуль `expect` позволит интерактивно работать с командой — не только ввести её, но и ввести ответы (необходим модуль `pexpect`, можно установить через pip).
- установка программ `yum`/`dnf` для CentOS/RedHat/Scientific/Rocky Linux, `apt` для `Debian/Ubuntu/Pop!_OS`, `slackpkg` для Slackware, `package` (общий, берет значение пакета установки программ из факта `ansible_pkg_mgr`).
- управление репозиториями: `yum_repository`, `apt_repo`, `rpm_key`
- запуск, остановка, перезапуск, добавление в автозагрузку сервисов: `service`, `systemd`
- управление пользователями: `user`, `group`, `authorized_key` (добавление или удаление авторизованных ssh ключей для индивидуальных аккаунтов пользователей), `openssh_keypair` (генерирование ssh ключей)
- копирование файлов: `copy`, `fetch`, `synchronize`
- создание файлов из шаблонов: `template`
- создание  конфигурационных файлов из фрагментов: `assemble`
- создание каталогов, файлов, изменение атрибутов, создание мягких ссылок: `file`
- дебаг во время выполнения тасков (задач): `debug`
- Поиск в файле строки по регулярному выражению и изменение строки при необходимости. Удаление, создание, изменение строки:
    - `lineinfile` изменяет только последнюю найденную в соответствии с регулярным выражением строку или добавляет строку, если её нет, (зависит от параметра backrefs),
    - `blockinfile`
    - `replace` — если изменять надо несколько строк, которые будут найдены в соответствии с регулярным выражением
- Клонирование репозиториев (но не может делать push): `git`
- работа с сетевыми устройствами (не Linux): `cli_command`
- создание архивов bz2, **gz** (по умолчанию), tar, xz, zip: `archive`
- работа с дисками: `parted` (создание разделов утилитой parted, которая является аналогом fdisk), `lvg` (создание, удаление, изменение размеров volume groups), `lvol` (создание и изменение логических томов Logical Volume, `filesystem` (создаёт файловые системы btrfs, ext2, ext3, ext4, ext4dev, f2fs, lvm, ocfs2, reiserfs, xfs, vfat, swap), `mount` (монтирование через */etc/fstab*, размонтировать можно как временно, так и удалить запись в fstab)
- задачи по расписанию: `cron`
- скачивает файлы с удаленного сервера (из Интернета) по HTTP, HTTPS, или FTP: `get_url`
- проверка доступности машин (nodes): `ping` (не является прямым аналогом команды ping)
- бесшовное обновление ПО, держит несколько предыдущих версий ПО в подкаталогах: `deploy_helper`
- настройки ядра Linuх: `sysctl`
- настройки SELinux: `selinux`
- работа с файрволами: `iptables`, `iptables_state`, `firewalld`
- ожидание условия перед продолжением действий (проверяем, появился ли определенный файл в каталоге, доступен ли определенный порт, ожидание окончания работы процесса, ожидание удаления lock файла, ожидание появления определенного слова в определенном файле): `wait_for`, `wait_for_connection`
- проверка, являются ли утверждения правдой, отображение сообщений при успешной или неуспешной проверке: `assert`
- поиск файлов по определенным критериям: `find`. Можно сохранять список найденных файлов через `register` и использовать в дальнейшем в другой таске.
- работа с базой данных: [mysql_db](https://docs.ansible.com/ansible/latest/collections/community/mysql/mysql_db_module.html#ansible-collections-community-mysql-mysql-db-module) (удаление и добавление баз), [mysql_info](https://docs.ansible.com/ansible/latest/collections/community/mysql/mysql_info_module.html#ansible-collections-community-mysql-mysql-info-module) (сбор информации о MySQL серверах), [mysql_query](https://docs.ansible.com/ansible/latest/collections/community/mysql/mysql_query_module.html#ansible-collections-community-mysql-mysql-query-module) (выполнение запроса MySQL), [mysql_replication](https://docs.ansible.com/ansible/latest/collections/community/mysql/mysql_replication_module.html#ansible-collections-community-mysql-mysql-replication-module) (репликация), [mysql_user](https://docs.ansible.com/ansible/latest/collections/community/mysql/mysql_user_module.html#ansible-collections-community-mysql-mysql-user-module) (управление пользователями), [mysql_variables](https://docs.ansible.com/ansible/latest/collections/community/mysql/mysql_variables_module.html#ansible-collections-community-mysql-mysql-variables-module) (управление глобальными переменными). Есть похожие команды для других баз данных.
- взаимодействие с веб-сервисами HTTP и HTTPS и поддержка Digest, Basic и WSSE HTTP механизмов аутентификации (проверка сайтов, наличия определенных слов в содержимом сайта, проверка аутентификации, автоматическое создание инцидентов в Jira и пр.): `uri`

## Плейбуки
`ansible-playbook file.yaml`  — выполнить плейбук с именем file.yaml

### Опции аутентификации

| Параметр                                            | Описание                                                                                                                                                                                 |
| --------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `--user, -u <username>`                             | залогиниться как пользователь                                                                                                                                                            |
| `--private-key, --key-file <key>`                   | Залогиниться используя ключ SSH (обычно в каталоге ~/.ssh)                                                                                                                               |
| `--ssh-extra-args`                                  | Добавить дополнительные аргументы команды SSH                                                                                                                                            |
| `--vault-id <id>`                                   | использовать идентификатор vault (шифрование), если есть несколько идентификаторов                                                                                                       |
| `--vault-password-file <key>`                       | указать файл, содержащий пароль vault для расшифровки переменных                                                                                                                         |
| `--ask-vault-pass`                                  | Будет использоваться строка приглашения для ввода пароля vault вручную                                                                                                                   |
| `--become`                                          | Эскалация привилегий (например пользователь получает права sudo)                                                                                                                         |
| `--ask-become-pass`                                 | Строка приглашения ввода пароля для эскалации привилегий вручную                                                                                                                         |
| `--become-method`                                   | Эскалация привилегий используя указанный метод (не тот, что указан в файле конфигурации ansible)                                                                                         |
| `ansible-doc -l`                                    | Список опций для become, соединения и других опций Ansible (список всех модулей).                                                                                                        |
| `ansible-doc -t become -l`                          | Список плагинов типа become. Все варианты для опции `-t`: become, cache, callback, cliconf, connection, httpapi, inventory, lookup, netconf, shell, vars, module, strategy, role, keyword |
| `ansible-doc -t connection -l`                      | Список плагинов соединения (connection). В плейбуке можно не указывать целиком [плагин соединения](https://acozine.github.io/html/plugins/connection.html), а писать только `grpc`, `httpapi`, `local` и пр.                                                                                                                                                   |
| `ansible-doc имя_модуля`, например `ansible-doc find` | Документация по модулю, предварительно показать список модулей `ansible-doc -t module -l`. Аналог документации [на сайте Ansible](https://docs.ansible.com/ansible/latest/index.html)                                                                                                                                                                  |

```shellsession
$ ansible-doc -t vars -l
community.sops.sops Loading sops-encrypted vars files
host_group_vars     In charge of loading group_vars and host_vars

$ ansible-doc -t become -l
ansible.netcommon.enable         Switch to elevated permissions on a network device
community.general.doas           Do As user
community.general.dzdo           Centrify's Direct Authorize
community.general.ksu            Kerberos substitute user
community.general.machinectl     Systemd's machinectl privilege escalation
community.general.pbrun          PowerBroker run
community.general.pfexec         profile based execution
community.general.pmrun          Privilege Manager run
community.general.sesu           CA Privileged Access Manager
community.general.sudosu         Run tasks using sudo su -
containers.podman.podman_unshare Run tasks using podman unshare
runas                            Run As user
su                               Substitute User
sudo                             Substitute User DO

$ ansible-doc -t connection -l
ansible.netcommon.grpc         Provides a persistent connection using the gRPC protocol
ansible.netcommon.httpapi      Use httpapi to run command on network appliances
ansible.netcommon.libssh       Run tasks using libssh for ssh connection
ansible.netcommon.napalm       Provides persistent connection using NAPALM
ansible.netcommon.netconf      Provides a persistent connection using the netconf protocol
ansible.netcommon.network_cli  Use network_cli to run command on network appliances
ansible.netcommon.persistent   Use a persistent unix socket for connection
community.aws.aws_ssm          execute via AWS Systems Manager
community.docker.docker        Run tasks in docker containers
community.docker.docker_api    Run tasks in docker containers
community.docker.nsenter       execute on host running controller container
community.general.chroot       Interact with local chroot
community.general.funcd        Use funcd to connect to target
community.general.iocage       Run tasks in iocage jails
community.general.jail         Run tasks in jails
community.general.lxc          Run tasks in lxc containers via lxc python library
community.general.lxd          Run tasks in lxc containers via lxc CLI
community.general.qubes        Interact with an existing QubesOS AppVM
community.general.saltstack    Allow ansible to piggyback on salt minions
community.general.zone         Run tasks in a zone instance
community.libvirt.libvirt_lxc  Run tasks in lxc containers via libvirt
community.libvirt.libvirt_qemu Run tasks on libvirt/qemu virtual machines
community.okd.oc               Execute tasks in pods running on OpenShift
community.vmware.vmware_tools  Execute tasks inside a VM via VMware Tools
community.zabbix.httpapi       Use httpapi to run command on network appliances
containers.podman.buildah      Interact with an existing buildah container
containers.podman.podman       Interact with an existing podman container
kubernetes.core.kubectl        Execute tasks in pods running on Kubernetes
local                          execute on controller
paramiko_ssh                   Run tasks via python ssh (paramiko)
psrp                           Run tasks over Microsoft PowerShell Remoting Protocol
ssh                            connect via SSH client binary
winrm                          Run tasks over Microsoft's WinRM  
```

### Опции контроля

| Параметр                      | Описание                                                                                                                                                        |
| ----------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `--syntax-check`              | Проверить синтаксис плейбука, но не выполнять его                                                                                                               |
| `--list-hosts`                | Список хостов, перечисленных в плейбуке                                                                                                                         |
| `--list-tasks `               | Список тасков (задач), содержащихся в плейбуке                                                                                                                  |
| `--start-at-task <task_name>` | Выполнить плейбук, начиная с задачи с именем, указанным здесь                                                                                                   |
| `--check` или `-C`            | Выполнить плейбук, но не делать изменения в реальности                                                                                                          |
| `--diff`                      | Показать изменения, который вносит плейбук при исполнении                                                                                                       |
| `--module-path` или `-M`      | Добавить путь к библиотеке модулей точку с запятой перед путем по умолчанию, по умолчанию default=~/.ansible/plugins/modules:/usr/share/ansible/plugins/modules |
| `--connection <method>`       | Использовать указанный метод соединения. Полный список плагинов: `ansible-doc -t connection -l`                                                                 |

Структура простого плейбука. Плейбук связывает, ЧТО будет выполняться и ГДЕ будет выполняться и может состоять из нескольких плеей. В свою очередь плей состоит из задач, но он может содержать ссылку на роль или несколько ролей. Можно сочетать роли и таски используя `pre_tasks` и `post_tasks`, что будет рассмотрено отдельно.

```yaml
---  # (1)!      
- name: “My play” # (2)!
  hosts: all      # (3)!

  tasks:          # (4)!
  - name: "My task"  # (5)!
    some_module:     # (6)!
      path: '/пример/'
  
  - name: "My other task"  # (7)!
    other_module:
      foo: ‘bar’
```

1. YAML файл начинается с трех дефисов
2. Имя плея (плейбук может содержать несколько плеев)
3. Список хостов, на которых будет выполняться плейбук. Это может быть как имя хоста, так и имя группы. По умолчанию список берется из инвентори /etc/ansible/hosts, но обычно указывается конкретно какой инвентори использовать, например `ansible-playbook -i  company/inventory ...`
4. Открывает перечисление списка задач. Вместо списка задач плейбук может ссылаться на роль.
5. Каждой задаче надо давать имя, которое потом будет отображаться при выполнении плейбука.
6. Указываем имя модуля. У модуля есть обязательные и необязательные параметры. Например модуль поиска find. Список параметров найти в Интернет или через `ansible-doc модуль`
7. Следующая задача (таска). Аналогично указываем модуль и его параметры.


Стандартная модель работы Ansible — **push**, но можно использовать [pull](https://docs.ansible.com/ansible/latest/cli/ansible-pull.html), используя cron и команду `ansible-pull` вместо `ansible-playbook`.  Плейбук установки можно настроить для изменения расписания cron, расположения логов, параметров ansible-pull. Это полезно как для экстремального масштабирования, так и для периодического исправления. Использования модуля `fetch` позволит централизованно собирать логи исполнения ansible-pull с разных хостов.

```console
ansible-pull -U URL-репозитория [опции] имя_плейбука.yml

ansible-pull [-h] [--version] [-v] [--private-key PRIVATE_KEY_FILE]
                 [-u REMOTE_USER] [-c CONNECTION] [-T TIMEOUT]
                 [--ssh-common-args SSH_COMMON_ARGS]
                 [--sftp-extra-args SFTP_EXTRA_ARGS]
                 [--scp-extra-args SCP_EXTRA_ARGS]
                 [--ssh-extra-args SSH_EXTRA_ARGS]
                 [-k | --connection-password-file CONNECTION_PASSWORD_FILE]
                 [--vault-id VAULT_IDS]
                 [--ask-vault-password | --vault-password-file VAULT_PASSWORD_FILES]
                 [-e EXTRA_VARS] [-t TAGS] [--skip-tags SKIP_TAGS]
                 [-i INVENTORY] [--list-hosts] [-l SUBSET] [-M MODULE_PATH]
                 [-K | --become-password-file BECOME_PASSWORD_FILE]
                 [--purge] [-o] [-s SLEEP] [-f] [-d DEST] [-U URL] [--full]
                 [-C CHECKOUT] [--accept-host-key] [-m MODULE_NAME]
                 [--verify-commit] [--clean] [--track-subs] [--check]
                 [--diff]
                 [playbook.yml ...]
```
