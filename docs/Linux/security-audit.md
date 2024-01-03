---
share: "true"
title: Автоматизация аудита безопасности
tags:
  - security
---

Cредства автоматизированного аудита проверят проблемы в Linux и могут отобразить, что именно необходимо исправить. Также есть разнообразные бенчмарки. Здесь опишу только CIS Benchmark. Средствами Ansible можно не только проверить проблемы согласно бенчмарку, но и произвести необходимые настройки автоматически.
## Lynis
[Lynis](https://cisofy.com/lynis/) — утилита аудита безопасности для UNIX деривативов типа Linux, macOS, BSD, Solaris, AIX и других, а также файлов Dockerfile. Бесплатная версия не строит красивые отчеты.
Как добавлять репозитории для разных дистрибутивов Linux, описано здесь https://packages.cisofy.com/community/

**Установка Lynis в Ubuntu.**

```bash
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 013baa07180c50a7101097ef9de922f1c2fde6c4
sudo apt install apt-transport-https
echo "deb https://packages.cisofy.com/community/lynis/deb/ stable main" | sudo tee /etc/apt/sources.list.d/cisofy-lynis.list
sudo apt update
sudo apt install lynis
sudo lynis update info
lynis -h                      # подсказки по командам
sudo lynis audit system       # аудит операционной системы
sudo lynis audit dockerfile   # аудит файла docker
```

Например установили версию и проверили:

```bash
$ sudo lynis update info

 == Lynis ==

  Version            : 3.0.7
  Status             : Up-to-date
  Release date       : 2022-01-18
  Project page       : https://cisofy.com/lynis/
  Source code        : https://github.com/CISOfy/lynis
  Latest package     : https://packages.cisofy.com/

2007-2021, CISOfy - https://cisofy.com/lynis/

$ lynis show commands   # список команд

Commands:
lynis audit
lynis configure
lynis generate
lynis show
lynis update
lynis upload-only
```

**Установка Lynis в CentOS, Oracle Linux, RHEL, Rocky Linux.**
Добавить файл репозитория */etc/yum.repos.d/cisofy-lynis.repo*

```ini title="/etc/yum.repos.d/cisofy-lynis.repo"
[lynis]
name=CISOfy Software - Lynis package
baseurl=https://packages.cisofy.com/community/lynis/rpm/
enabled=1
gpgkey=https://packages.cisofy.com/keys/cisofy-software-rpms-public.key
gpgcheck=1
priority=2
```

Установить Lynis и проверить версию:

```bash
sudo yum makecache fast
sudo yum update ca-certificates curl nss openssl
sudo yum install lynis
sudo lynis update info
```

Пример вывода части результата теста. В реальности проблемы подсвечиваются оранжевым и красным цветом.

```bash
$ sudo lynis audit system
...
 -[ Lynis 3.0.7 Results ]-

  Warnings (3):
  ----------------------------
  ! Found one or more vulnerable packages. [PKGS-7392] 
      https://cisofy.com/lynis/controls/PKGS-7392/

  ! Nameserver 192.168.42.129 does not respond [NETW-2704] 
      https://cisofy.com/lynis/controls/NETW-2704/

  ! Couldn't find 2 responsive nameservers [NETW-2705] 
      https://cisofy.com/lynis/controls/NETW-2705/

  Suggestions (42):
  ----------------------------
  * Set a password on GRUB boot loader to prevent altering boot configuration (e.g. boot in single user mode without password) [BOOT-5122] 
      https://cisofy.com/lynis/controls/BOOT-5122/

  * Consider hardening system services [BOOT-5264] 
    - Details  : Run '/usr/bin/systemd-analyze security SERVICE' for each service
      https://cisofy.com/lynis/controls/BOOT-5264/

  * If not required, consider explicit disabling of core dump in /etc/security/limits.conf file [KRNL-5820] 
      https://cisofy.com/lynis/controls/KRNL-5820/

  * Check PAM configuration, add rounds if applicable and expire passwords to encrypt with new values [AUTH-9229] 
      https://cisofy.com/lynis/controls/AUTH-9229/
....

  * Harden the system by installing at least one malware scanner, to perform periodic file system scans [HRDN-7230] 
    - Solution : Install a tool like rkhunter, chkrootkit, OSSEC
      https://cisofy.com/lynis/controls/HRDN-7230/

  Follow-up:
  ----------------------------
  - Show details of a test (lynis show details TEST-ID)
  - Check the logfile for all details (less /var/log/lynis.log)
  - Read security controls texts (https://cisofy.com)
  - Use --upload to upload data to central system (Lynis Enterprise users)

================================================================================

  Lynis security scan details:

  Hardening index : 64 [############        ]
  Tests performed : 254
  Plugins enabled : 0

  Components:
  - Firewall               [V]
  - Malware scanner        [X]

  Scan mode:
  Normal [V]  Forensics [ ]  Integration [ ]  Pentest [ ]

  Lynis modules:
  - Compliance status      [?]
  - Security audit         [V]
  - Vulnerability scan     [V]

  Files:
  - Test and debug information      : /var/log/lynis.log
  - Report data                     : /var/log/lynis-report.dat

================================================================================

  Lynis 3.0.7

  Auditing, system hardening, and compliance for UNIX-based systems
  (Linux, macOS, BSD, and others)

  2007-2021, CISOfy - https://cisofy.com/lynis/
  Enterprise support available (compliance, plugins, interface and tools)
```

Например, были найдены уязвимые пакеты, но какие именно — непонятно. По каждой проблеме можно отображить детальную информацию, например здесь рекомендуется обновить rsyslog, libsqlite3-0, libsqlite3-0:i386.

```bash
$ sudo lynis show details PKGS-7392
2022-05-08 20:59:54 Performing test ID PKGS-7392 (Check for Debian/Ubuntu security updates)
2022-05-08 20:59:54 Action: updating package repository with apt-get
2022-05-08 21:00:07 Result: apt-get finished
2022-05-08 21:00:07 Test: Checking if /usr/lib/update-notifier/apt-check exists
2022-05-08 21:00:07 Result: apt-check (update-notifier-common) not found
2022-05-08 21:00:12 Result: found vulnerable package(s) via apt-get (-security channel)
2022-05-08 21:00:12 Found vulnerable package: libsqlite3-0
2022-05-08 21:00:12 Found vulnerable package: libsqlite3-0:i386
2022-05-08 21:00:12 Found vulnerable package: rsyslog
2022-05-08 21:00:12 Warning: Found one or more vulnerable packages. [test:PKGS-7392] [details:-] [solution:-]
2022-05-08 21:00:12 Suggestion: Update your system with apt-get update, apt-get upgrade, apt-get dist-upgrade and/or unattended-upgrades [test:PKGS-7392] [details:-] [solution:-]
2022-05-08 21:00:12 ====
```

## OpenSCAP
[OpenSCAP](https://www.open-scap.org/)  — аналог Lynis
## Suricata
[Suricata](https://suricata.io/) — free and open source, mature, fast and robust network threat detection engine. The Suricata engine is capable of real time intrusion detection (IDS), inline intrusion prevention (IPS), network security monitoring (NSM) and offline pcap processing. Suricata inspects the network traffic using a powerful and extensive rules and signature language, and has powerful Lua scripting support for detection of complex threats.
## CIS Benchmark
Центр интернет-безопасности (CIS — Center of Internet Security) является некоммерческой организацией, которая разрабатывает собственные контрольные показатели и рекомендации, которые позволяют организациям совершенствовать свои программы обеспечения безопасности и соответствия требованиям. Эта инициатива направлена на создание базовых уровней конфигурации безопасности систем, которые обычно встречаются во всех организациях.

[Контрольные показатели CIS](https://www.cisecurity.org/cis-benchmarks/) представляют собой базовые показатели конфигурации и рекомендации для безопасной настройки системы. В каждой из рекомендаций содержатся ссылки на одну или несколько [точек управления CIS](https://www.cisecurity.org/controls/), которые были разработаны, чтобы помочь организациям улучшить свои возможности киберзащиты. Контрольные точки CIS соответствуют многим установленным стандартам и нормативным положениям, включая NIST Cybersecurity Framework (CSF) и NIST SP 800-53, серию стандартов ISO 27000, PCI DSS, HIPAA и другие.

Каждый контрольный показатель проходит два этапа консенсусного анализа. Первый имеет место во время первоначальной разработки, когда эксперты собираются для обсуждения, создания и тестирования рабочих проектов, и продолжается до тех пор, пока они не достигнут консенсуса по контрольному показателю. На втором этапе, после публикации контрольного показателя, консенсусная группа рассматривает отзывы интернет-сообщества для включения в контрольную точку.

В контрольных показателях CIS предусмотрены два уровня настроек безопасности:
- **Уровень 1** рекомендует основные базовые требования безопасности, которые могут быть настроены в любой системе и должны вызывать незначительное или нулевое прерывание обслуживания либо снижение функциональности.
- **Уровень 2** рекомендует параметры безопасности для сред, требующих повышенной безопасности, что может привести к некоторому снижению функциональности.

https://downloads.cisecurity.org/#/ — здесь можно скачать документы CIS Benchmark в формате `.pdf` для проверки Linux (есть общий файл для Linux, а также файлы для конкретных дистрибутивов), Windows, Apple macOS, VMware, MongoDb, Apache Tomcat, Microsoft SQL Server, Nginx, PostgreSQL, Apache Cassandra, Apache HTTP Server, Docker, Oracle Database, Kubernetes, MIT Kerberos, Oracle MySQL, Cisco, Juniper и пр.

В документах содержится информация о том, что проверять, что должно отображаться в результате проверки и как исправить. Для того, чтобы не делать все операции вручную, можно использовать автоматизацию с помощью Ansible. На сайте https://galaxy.ansible.com/  в окно поиска ввести "CIS Benchmark", будет найдено около 60 ролей. Надо смотреть на роли с хорошими оценками ближе к 5 и желательно. Если есть роль официального производителя, например от RedHat, можно скачать её. В компаниях обычно скачивают роль и самостоятельно её улучшают под собственные нужды.
Обычно в компании есть центральный сервер в установленным Ansible или же Ansible запускается в виде docker контейнера.

Установка Ansible: 

```bash
sudo python get-pip.py
sudo python -m pip install ansible
sudo python3 -m pip install --upgrade ansible  # обновление Ansible

# Проверка версии
$ ansible --version
ansible [core 2.12.5]
  config file = /home/bestann/ansible.cfg
  configured module search path = ['/home/bestann/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.9/dist-packages/ansible
  ansible collection location = /home/bestann/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/local/bin/ansible
  python version = 3.9.7 (default, Sep 10 2021, 14:59:43) [GCC 11.2.0]
  jinja version = 3.0.3
  libyaml = True
```

Настройки конфигурационного файла `ansible.cfg`:

```ini title="ansible.cfg"
[defaults]
roles_path = ~/roles:~/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles
inventory=~/hosts
forks=10
remote_user=autom
```

Установка роли https://galaxy.ansible.com/redhatofficial/rhel8_cis

```bash
ansible-galaxy install redhatofficial.rhel8_cis # установка роли
ansible-galaxy install redhatofficial.rhel8_cis --force # обновление роли до последней версии, если уже была установлена
```

Роль была установлена в домашнем каталоге пользователя в подкаталог roles. Лучше на её основе создать свою роль, чтобы изменять параметры для своих систем. Находясь в домашнем каталоге скопируем скаченную роль в свою роль cis.

```bash
cp -r roles/{redhatofficial.rhel8_cis,cis}
```

Пример автоматизации установки пароля на загрузчик Linux через Ansible:

```yaml
# работает только для CentOS 8 или OracleLinux 8, где уже стоит python3
# если стоит по умолчанию python2, как в CentOS7, пакета python3-pexpect нет
# по идее все равно надо CentoS7 переводить на python3, т.к. python2 уже не поддерживается
- name: 1.5.2 Ensure bootloader password is set | install grub2-tools & pexpect
  package:
    name: "{{ item }}"
    state: present
  loop:
    - python3-pexpect
    - grub2-tools
    - grub2-tools-minimal
  when: ansible_python.version.major == 3

# grub2-mkpasswd-pbkdf2 — генерирует пароль
# grub2-setpassword — генерирует пароль и пишет в /boot/grub2/user.cfg
# Заранее генерируем пароль через grub2-mkpasswd-pbkdf2 и помещаем в переменную в уже зашифрованном виде
# TODO: переменную лучше держать не в роли, а в плейбуке или отдельно подключаемом файле
# защита паролем включена в этом файле /etc/grub.d/01_users
# можно и модуль copy использовать
- name: 1.5.2 Ensure bootloader password is set
  ini_file:
    path: /boot/grub2/user.cfg
    section: null
    option: GRUB2_PASSWORD
    value: "{{ cis_bootloader_password }}"
    mode: 0600
    no_extra_spaces: yes
  notify: run grub2-mkconfig
```

Пример части автоматизации настроек безопасного ssh соединения:

```yaml
# Поиск без учета регистра (строчные или прописные)
# как вариант — сделать шаблон на будущее целиком для файла конфига
- name: |
    5.2.5 Ensure SSH LogLevel is appropriate
    5.2.6 Ensure SSH X11 forwarding is disabled
    5.2.7 Ensure SSH MaxAuthTries is set to 4 or less
    5.2.8 Ensure SSH IgnoreRhosts is enabled
    5.2.9 Ensure SSH HostbasedAuthentication is disabled
    5.2.10 Ensure SSH root login is disabled
    5.2.11 Ensure SSH PermitEmptyPasswords is disabled
    5.2.12 Ensure SSH PermitUserEnvironment is disabled
    5.2.13 Ensure SSH Idle Timeout Interval is configured
    5.2.14 Ensure SSH LoginGraceTime is set to one minute or less
    5.2.15 Ensure SSH warning banner is configured
    5.2.16 Ensure SSH PAM is enabled
    5.2.17 Ensure SSH AllowTcpForwarding is disabled
    5.2.18 Ensure SSH MaxStartups is configured
    5.2.19 Ensure SSH MaxSessions is set to 4 or less
  lineinfile:
    path: /etc/ssh/sshd_config
    regexp: (?i)^\s*{{item.key }}\s+
    line: "{{ item.key }} {{ item.value }}"
    insertbefore: ^[#\s]*Match
    validate: /usr/sbin/sshd -t -f %s
  loop: "{{ cis_sshd_config | dict2items }}"
  notify: reload sshd
```

Переменные роли берутся из файла переменных *defaults/main.yml* внутри роли, например переменная `cis_sshd_config`

```yaml title="defaults/main.yml"
...
cis_sshd_config:
  LogLevel: VERBOSE
  X11Forwarding: no
  MaxAuthTries: 4
  IgnoreRhosts: yes
  HostbasedAuthentication: no
  PermitRootLogin: no
  PermitEmptyPasswords: no
  PermitUserEnvironment: no
  ClientAliveInterva: 900
  ClientAliveCountMax: 0
  LoginGraceTime: 60
  Banner: /etc/issue.net
  UsePAM: yes
  AllowTcpForwarding: no
  MaxStartups: "10:30:60"
  MaxSessions: 10
```

Для применения роли необходимо создать файл `hosts` со списком всех серверов Linux, например для теста создадим:

```ini title="hosts"
[proxy]
ansible2.hl.local ansible_host=192.168.122.121

[webservers]
ansible3.hl.local ansible_host=192.168.122.37
ansible4.hl.local ansible_host=192.168.122.193

[database]
ansible5.hl.local ansible_host=192.168.122.122

[ol]
ol8.hl.local ansible_host=192.168.122.53
```

Каждый сервер может состоять в своей группе, указанной в квадратных скобках. Применять роль можно ко всем хостам (all) или к какой-то группе (например webservers). Создадим плейбук *cis.yml* для применения роли:

```yaml title="cis.yml"
---
- hosts: all
  become: yes
  roles:
    - cis
```

Выполнение роли позволит настроить сразу все необходимые серверы компании:

```bash
ansible-playbook cis.yml
```
