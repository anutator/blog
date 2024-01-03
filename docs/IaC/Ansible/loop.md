---
tags:
  - ansible
share: "true"
title: Использование loop в Ansible
---

loop в loop-е
```yaml
- name: test loop
  debug:
    msg: '{{ item.0.key }} - {{ item.1.name }}:{{ item.1.chmod }}'
  loop: >-
    {{ auth | dict2items | subelements('value') }}
  loop_control:
    label: '{{ item.0.key }}'
```

## loop двух элементов
playbook test.yml

```yaml title="test.yml"
- hosts: localhost
  gather_facts: no
  connection: local
  vars:
    type: st1
    size: 2000
    instance_ids:
      - xxx
      - yyy
      - zzz
    objs:
      - { key1: /dev/sdb1, key2: "{{instance_ids}}" }
      - { key1: /dev/sdc1, key2: "{{instance_ids}}" }
      - { key1: /dev/sdd1, key2: "{{instance_ids}}" }

  tasks:
    - name: 2nd --------
      debug: msg="{{ item.0.key1 }} and {{ item.1 }} and {{ size }} and {{ type }}"
      with_subelements:
        - "{{ objs }}"
        - key2
```

Использование:
```bash
ansible-playbook test.yml -i hosts/all | grep -i "msg"

"msg": "/dev/sdb1 and xxx and 2000 and st1"
"msg": "/dev/sdb1 and yyy and 2000 and st1"
"msg": "/dev/sdb1 and zzz and 2000 and st1"
"msg": "/dev/sdc1 and xxx and 2000 and st1"
"msg": "/dev/sdc1 and yyy and 2000 and st1"
"msg": "/dev/sdc1 and zzz and 2000 and st1"
"msg": "/dev/sdd1 and xxx and 2000 and st1"
"msg": "/dev/sdd1 and yyy and 2000 and st1"
"msg": "/dev/sdd1 and zzz and 2000 and st1"
```

Если вы уверены, что ваши списки синхронизированы, использовать [`zip` filter](https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters.html#zip-and-zip-longest-filters), как показано ниже в `test.yml` MVCE плейбуке.

```yaml title="test.yml"
- name: Zip demo
  hosts: localhost
  gather_facts: false

  vars:
    "dev_names": [
      "nameisone",
      "nameistwo"
    ]

    "dev_types": [
      "type_one",
      "type_two"
    ]

  tasks:
    - name: demonstrate how to use the zip filter with a loop
      debug:
        msg: "Element from first list: {{ item.0 }}. Element from second list: {{ item.1 }}"
      loop: "{{ dev_names | zip(dev_types) | list }}"
```

Выполнение плейбука:

```bash
$ ansible-playbook test.yml 

PLAY [Zip demo] ************************************************************************************************************************************************************************************************************************

TASK [demonstrate how to use the zip filter with a loop] *******************************************************************************************************************************************************************************
ok: [localhost] => (item=['nameisone', 'type_one']) => {
    "msg": "Element from first list: nameisone. Element from second list: type_one"
}
ok: [localhost] => (item=['nameistwo', 'type_two']) => {
    "msg": "Element from first list: nameistwo. Element from second list: type_two"
}

PLAY RECAP *****************************************************************************************************************************************************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

## Loop по атрибутам словаря
Есть словарь, где мы хотим сделать loop только по конкретным атрибутам, а не всем значениям. Пример описания интерфейсов:

```yaml
interfaces:
  eth0:
    ip: 10.0.0.10
    mask: 16
  eth1:
    ip: 192.168.1.100
    mask: 24
```

Как сделать loop только по всем IP адресам? Нужен фильтр `map`. Просмотрим все значения из словаря `interfaces` (`eth0`, `eth1`) и затем извлечем атрибут `ip` из них и сформируем список.

```yaml
vars:
  ip_addresses: "{{ interfaces.values() | map(attribute='ip') | list }}"
```

[Ansible filters](https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters.html) are a very powerful tool, so I would recommend you take some time to read it thoroughly. If you are looking for more Ansible tips, such as [how to make a playbook distribution agnostic](https://radeksprta.eu/posts/make-ansible-playbook-distribution-agnostic) or [setup your laptop with Ansible](https://radeksprta.eu/posts/automatically-setup-computer-ansible-playbook), look [here](https://radeksprta.eu/categories/ansible/).