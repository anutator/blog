---
tags:
  - linux
share: "true"
title: Полезные алиасы
---

Алиасы — настраиваемые сокращения команд Linux. Здесь буду обновлять список полезных алиасов.

Некоторые из алиасов уже прописаны по умолчанию:

```bash
l='ls -lah' 
la='ls -lAh' 
ll='ls -lh' 
ls='ls -G' 
lsa='ls -lah' 
md='mkdir -p'
```

Создание алиаса на время текущей сессии: `alias k=kubectl`

*/etc/motd* — подсказки и предупреждения, которые будут отображаться при залогинивании на сервер.
*/etc/profile.d/alias.sh* — файл с полезными переменными окружения, алиасами, функциями.
 
```bash title="/etc/profile.d/alias.sh"
alias vi=vim

# список подключенных клиентов к серверу OpenVPN (в настройках сервера должен быть указан файл лога)
alias vpnlist='grep "Peer Connection" /var/log/openvpn.log'

# увеличиваем стандартный размер хранения истории команд. Добавляем дату (день и месяц) и время (часы и минуты) выполнения команды.
export HISTSIZE=10000
export HISTFILESIZE=30000
export HISTCONTROL=ignoredups
export HISTTIMEFORMAT="%d.%m %H:%M  "

# cписок имен контейнеров и их ip-адресов, если они формируются через сеть Rancher
alias ranchip='docker inspect --format='{{.Name}} {{ index .Config.Labels "io.rancher.container.ip"}}' $(docker ps -q)'
# cтандартный список контейнеров docker и их ip-адресов — только если установлен Docker
alias docip='docker inspect --format='{{.Name}} {{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -q)'

# пока не заработало в Oracle Linux 8
#podip () {
#  podman inspect -f '{{.Name}}\t{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}\t{{.NetworkSettings.Ports}}' $(podman ps -q)
#}
# или так
alias podip='podman inspect -f '{{.Name}}\\t{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}\\t{{.NetworkSettings.Ports}}' $(podman ps -q)'

# Стандартно не видно порты, которые открывает Rancher в iptables. Можно вывести их через сохранение:
alias ranchiptables='iptables-save | grep "\-A CATTLE_HOSTPORTS_POSTROUTING"'
# или полные данные просто iptables-save

# список подключенных клиентов к серверу OpenVPN (в настройках сервера должен быть указан файл лога)
alias vpnlist='grep "Peer Connection" /var/log/openvpn.log'

# сжатие всех png файлов из папок и подпапок текущего каталога. Установить pngquant, zopfli.
alias png1='find . -type f -name "*.png" -exec pngquant 64 --skip-if-larger --strip --ext=.png --force {} +'
alias png2='find . -type f -name "*.png" -print0 | xargs -0 -I '{}' zopflipng -y '{}' '{}''

# для телефонии
alias pass2='date +%s | sha256sum | base64 | head -c 32 ; echo'
alias pass='openssl rand -base64 32'
alias sip='vim /etc/asterisk/pjsip_wizard.conf'
alias sipadd='asterisk -rx "pjsip reload"'

# удалить все пакеты pip
# pip freeze > requirements.txt
# pip uninstall -r requirements.txt # подтверждать каждый
# pip uninstall -r requirements.txt -y # все
alias pipuninstallall="pip uninstall -y -r <(pip freeze)"

# Для Mattermost (из комплекта Gitlab Omnibus)
alias mattermost-cli="cd /opt/gitlab/embedded/service/mattermost && sudo /opt/gitlab/embedded/bin/chpst -e /opt/gitlab/etc/mattermost/env -P -U mattermost:mattermost -u mattermost:mattermost /opt/gitlab/embedded/bin/mattermost --config=/var/opt/gitlab/mattermost/config.json $1"
```
