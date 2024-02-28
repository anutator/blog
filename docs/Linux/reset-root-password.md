---
tags:
  - security
title: Восстановление пароля root и защита загрузчика
share: "true"
---
При наличии физического доступа к серверу или доступа к веб-интерфейсу VMWare (актуально для виртуальных машин), откуда можно перезагрузить Linux и попасть в консоль загрузки, злоумышленник может из консоли прервать стандартную загрузку Linux, зайти в режим аварийного восстановления rescue и установить свой пароль для root. Как это сделать официальным способом, показано ниже. Есть немного измененные неофициальные варианты, позволяющие избавиться от длинного процесса relabel SELinux.
## Сброс пароля в RedHat
Перезагрузить систему. Во время появления меню загрузчика GRUB2 **нажать любую клавишу** для прерывания процесса, затем нажать **e** английскую для редактирования (edit).

!["Прервать процесс загрузки"](rhel7.1-boot.png)

С помощью курсора или клавиши End перейти в конец строки, которая начинается с `linux16 /vmlinuz-…`, добавить туда `rd.break enforcing=0`. Директива `rd.break` запрашивает прерывание процесса загрузки на ранней стадии, `enforcing=0` помещает систему в режим SELinux Permissive вместо SELinux Enforcing (по умолчанию, если ранее не выключали). Не путать с `selinux=0`, который полностью выключает SELinux.

 `rd.break` не является командой GRUB загрузчика; это аргумент [Dracut](https://dracut.wiki.kernel.org/), который говорит Dracut попасть в оболочку shell. The version of GRUB is also different than in previous versions of RHEL. It is what was formerly know as GRUB2. It is a seriously over-engineered and complex multiboot loader which uses a completely different command syntax than GRUB Legacy.

Нажать `Ctrl-X` для загрузки с измененной конфигурацией:

```bash
    insmod part_msdos
    insmod xfs
    set root='hde,msdos1'
    if [x$feature_platform_search_hint = xy ]; then
      search --no-floppy --fs-uuid --set=root --hint='hd0,msdos1' dfa2f4ce-ab99-4e78-bc96-ebb73fa2922c
    else
      search --no-floppy --fs-uuid --set=root dfa2f4ce-ab99-4e78-bc96-ebb73fa2922c
    fi
    linux16 /umlinuz-3.10.0-229.el7.x86_64 root=/dev/mapper/rhel_rhel7-root ro crashkernel=auto rd.lvm.lv=rhel_rhel7/root rd.lvm.lv=rhel_rhel7/swap rhgb quiet LANG=en_US.UTF-8    # тут добавить rd.break enforcing=0
    initrd16 /initramfs-3.10.0-229.e17.x86_64.img
  
  Press Ctrl-x to start, Ctrl-c for a command prompt or Escape to
  discard edits and return to the menu. Pressing Tab lists
  possible completions.
```

После загрузки в строке слева написано `switch_root` перемонтируем файловую систему `/sysroot ` в режиме read-write (запись и чтение), командой `chroot` заходим в ловушку (jail— тюрьма, тюремное заключение). Пароль для root меняем стандартной командой `passwd`.

```bash
Generating "/run/initramfs/rdsosreport.txt"

Entering emergency mode. Exit the shell to continue.
Type "journalctl" to view system logs.
You might want to save "/run/initramfs/rdsosreport.txt" to a USB stick or /boot
after mounting them and attach it to a bug report.

switch_root:/# mount -oremount,rw /sysroot
switch_root:/# chroot /sysroot
sh-4.2# passwd root
Changing password for user root.
New password:

# в продакшн системах после установки пароля снова перемонтировать 
# в режиме read-only (только чтение) для безопасности.

sh-4.2# mount -o remount,ro    
sh-4.2# exit   либо Ctrl-D
exit
switch_root:/# exit   либо Ctrl-D
logout
```

После того как набрали два раза exit, наблюдаем за процессом загрузки в консоли, ни в коем случае не перезагружаемся пока не зайдем под root c новым паролем и не доделаем.

```bash
...
[  OK  ] Started Network Manager Script Dispatcher Service.
[  OK  ] Started Crash recovery kernel arming.
[  OK  ] Reached target Multi-User System.

CentOS Linux 7 (Core)
Kernel 3.10.0-229.14.1.el7.x86_64 on an x86_64

vm login: root
Password: новый_пароль             вводим новый пароль

# restorecon /etc/shadow       # Доделываем! Пояснение ниже
# reboot # или setenforce enforcing   или  setenforce 1   (перезагружаться необязательно)
```

При строгом действии в соответствии с этой процедурой не надо делать SELinux relabel (# `touch /.autorelabel` или # `fixfiles onboot`) или загружать SELinux policy (`/usr/sbin/load_policy -i`).

Пояснения:
Команда `passwd` изменяет содержимое файла `/etc/shadow`, где хранятся пароли: создается новый файл с измененным паролем. Далее применяется ссылка SELinux к новому файлу, копируется содержимое старого файла `/etc/shadow` и записывается новый пароль. В конце новый файл переименовывается в `/etc/shadow`. Добавленная нами директива `rd.break` прерывает процесс загрузки и на этой фазе SELinux не работает, команда `passwd` не может правильно переразметить файл. В результате при загрузке системы у файла `/etc/shadow` будет неправильная SELinux метка. Если система будет в режиме SELinux Enforcing, она не сможет получить доступ к файлу `/etc/shadow` и будет невозможно залогиниться. Вот пример выдержки важных строк из лога аудита:

```bash
type=AVC msg=audit(1471855249.615:42): avc: denied { open } for pid=2056 comm=”unix_chkpwd” path=”/etc/shadow” dev=”dm-1″ ino=1112495 scontext=system_u:system_r:chkpwd_t:s0-s0:c0.c1023 tcontext=system_u:object_r:unlabeled_t:s0 tclass=file

type=AVC msg=audit(1471855249.615:42): avc: denied { read } for pid=2056 comm=”unix_chkpwd” name=”shadow” dev=”dm-1″ ino=1112495 scontext=system_u:system_r:chkpwd_t:s0-s0:c0.c1023 tcontext=system_u:object_r:unlabeled_t:s0 tclass=file
```

Команда `chroot` позволяет выполнить другую команду с измененным путем к корневой директории. Хотя она и может пригодиться для выполнения различных административных задач, чаще всего о ней упоминают как о методе защиты отдельных служб или процессов, заключающемся в их запуске с использованием новой нестандартной корневой директории. Преимущество данного метода заключается в том, что служба или процесс исполняется в окружении `chroot`, изолированном от таких важных хранилищ системных файлов, как директория `/etc` или других не менее важных файлов, хранящихся в системных директориях. Вместе с данным методом защиты используется еще один метод, заключающийся в запуске служб или процессов с использованием учетных записей пользователей с минимальными наборами привилегий.

Например, при активации поддержки `chroot` служба доменных имен named использует в качестве корневой директории директорию `/var/named` и запускается от лица локального пользователя с именем «named» (с минимальными привилегиями) в отличие от корневой директории системы и учетной записи пользователя root. Поэтому если сервер named будет скомпрометирован взломщиком или вредоносной программой, будут повреждены лишь файлы из директории `/var/named `(новой корневой директории сервера), ведь служба `named` не может получить доступ к файлам, находящимися за приделами окружения chroot, а пользователь с именем named не имеет достаточных прав для чтения каких-либо других системных файлов.

Проверить режим ДО восстановления пароля можно только в системе, к которой уже имеется доступ хотя бы от стандартного пользователя:
```bash
$ getenforce
Disabled
```

### Вариант2. Тоже быстрый
The method I demonstrate here does not require a second reboot and relabels only `/etc/shadow` since this is the only file whose security content is incorrect after you change the password for _root_. The net result of using this method is several minutes of time saved during the RHCSA exam.

First get to the GRUB command line by rebooting the operating system and add `init=/bin/bash` to the end of the GRUB command which starts with `linux16` in the case of non-UEFI firmware or _linuxefi_ in the case of [UEFI](http://www.uefi.org)-enabled firmware. By the way, _linux16_ means load the binary in 16-bit mode. UEFI does not have a 16–bit mode; hence the different GRUB commands. Do not bother removing the existing _rhgb_ or _quiet_ arguments unless you run into a problem while booting.

!["Добавить init=/bin/bash"](grub2-edit-init.png)

Как обычно жмем `Ctrl-X` чтобы выйти из GRUB и продолжить процесс загрузки. В итоге мы окажемся root в bash shell, где можно изменить пароль для root, загрузить политику SELinux, переразметить (relabel) файл `/etc/shadow` и загрузить операционную систему полностью без второй перезагрузки.

```bash
bash-4.2# /bin/mount -o remount,rw /

bash-4.2# passwd
Changing password for user root.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.

bash-4.2# ls -lZ /etc/{passwd,shadow}  # проверяем контексты SELinux
-rw-r--r--. root root ?				/etc/passwd
----------. root root ?				/etc/shadow

bash-4.2# /sbin/load_policy -i  # загружаем текущую политику безопасности в ядро

bash-4.2# ls -lZ /etc/{passwd,shadow}  # проверяем контексты SELinux
-rw-r--r--. root root system_u:object_r:passwd_file_t:s0 /etc/passwd
----------. root root system_u:object_r:unlabeled_t:s0 /etc/shadow

bash-4.2# /bin/restorecon -v /etc/shadow
bash: /bin/restorecom: No such file or directory

bash-4.2# /sbin/restorecon -v /etc/shadow  # восстанавливаем правильный контекст безопасности (shadow_t) в /etc/shadow
/sbin/restorecon reset /etc/shadow context system_u:object_r:unlabeled_t:s0->system_u:object_r:shadow_t:s0

bash-4.2# ls -lZ /etc/{passwd,shadow}  # проверяем контексты SELinux
-rw-r--r--. root root system_u:object_r:passwd_file_t:s0 /etc/passwd
----------. root root system_u:object_r:shadow_t:s0    /etc/shadow

bash-4.2# exec /sbin/init
```

Также читать: [Fastest Way to Gain Root Access in RHCSA7 Exam](https://blog.fpmurphy.com/2016/10/fastest-way-to-gain-root-access-in-rhcsa7-exam.html)

### Вариант 3. Официальный вариант от Red Hat. Менее предпочтительный, т.к. медленнее.
Перезагрузить систему. Во время появления меню загрузчика GRUB2 **нажать любую клавишу** для прерывания процесса загрузки. Убедившись, что выбрана правильная версия ядра, нажать английскую **e** для редактирования (edit).

Получили доступ к GRUB boot-loader. В строке, которая начинается на `linux16 …` нажать `Ctrl-E` или `End`, чтобы сразу попасть в конец строки, и добавить `rd.break`. Часть `rd` — это сокращение от ramdisk (initial ramdisk (initrd) окружение).

Нажать `Ctrl-X` для продолжения прерванной загрузки с измененными параметрами.
Система загрузится в аварийный режим *emergency.target*. Здесь нужно будет:
- перемонтировать каталог */sysroot* с правами записи и чтения
- Переключиться в root jail (jail — тюрьма) через `chroot`
- задать новый пароль для root
- Сделать `relabel` для SELinux. Процесс смены меток занимает много времени, т.е. все файлы перелинкуются, хотя нас интересует только */etc/shadow*

```bash
Generating "/run/initramfs/rdsosreport.txt"

Entering emergency mode. Exit the shell to continue.
Type "journalctl" to view system logs.
You might want to save "/run/initramfs/rdsosreport.txt" to a USB stick or /boot
after mounting them and attach it to a bug report.

switch_root:/# mount -oremount,rw /sysroot
switch_root:/# chroot /sysroot
sh-4.2# passwd root
Changing password for user root.
New password:
…
sh-4.2# touch /.autorelabel   
sh-4.2# exit   либо Ctrl-D
exit
switch_root:/# exit   либо Ctrl-D
logout
```

## Сброс пароля Debian
Перезагрузить компьютер (виртуальную машину). Во время загрузки появится экран GNU GRUB с обратным счетчиком (примерно 5 секунд). Если ничего не выбрать, активируется стандартная опция загрузки. Во время появления меню загрузчика GRUB2 **нажать любую клавишу** для прерывания процесса загрузки. Убедившись, что выбрана правильная версия ядра, нажать английскую **e** для редактирования (edit) команд перед загрузкой.

Найти строку, начинающуюся на слово _linux_, после которого указана версия загрузчика `/boot/vmlinuz-*` (тут конкретная версия) и далее `root=UUID=(идентификатор загрузочного раздела)`. Переходим в конец строки либо стрелкой, либо клавишей End. Обычно строка заканчивается так: `ro quiet`, Добавляем после этих слов `init=/bin/bash `или `init=/bin/sh`. Жмем `Ctrl+x` или `F10` для продолжения процесса загрузки с измененной опцией. Debian загрузится в однопользовательский режим, где корневая файловая система смонтирована в режиме "только чтение" (read-only). Перемонтируем в режиме чтения-записи read-write:
`mount -n -o remount,rw /`
Стандартной командой `passwd` изменяем пароль root.

Перезагружаем машину. Заходим под root из консоли, меняем пароль для стандартного пользователя pupkin. Т.к. удаленный доступ по ssh для root закрыт, надо сначала заходить под другим пользователем.

## Защита загрузчика boot-loader
Как показано выше, если есть доступ к консоли, мы можем прервать процесс загрузки Linux, поэтому если дополнительно не защитить загрузчик паролем, будет достаточно просто поменять пароль root и любого пользователя (список пользователей может в том же меню восстановления посмотреть в */etc/passwd*).

Во-первых, установим права для файла конфигурации загрузчика:
- владелец и группа-владельца файла *grub.cfg* — root
- если есть файл *user.cfg*, то также его владелец и группа — root
- отнимаем все права (чтение, запись, исполнение) у группы и для всех остальных. Только владелец root сможет просматривать или изменять файл.

```bash
chown root. /boot/grub2/grub.cfg
test -f /boot/grub2/user.cfg && chown root. /boot/grub2/user.cfg
chmod og-rwx /boot/grub2/grub.cfg
test -f /boot/grub2/user.cfg && chmod og-rwx /boot/grub2/user.cfg
```

Для виртуальных машин есть дополнение. Если система использует UEFI, отредактировать */etc/fstab* и добавить опцию `fmask=0077`:

```bash title="/etc/fstab"
<устройство> /boot/efi vfat defaults,umask=0027,fmask=0077,uid=0,gid=0 0 0
```

Создать зашифрованный пароль для загрузчика.

```bash
# grub2-setpassword
Enter password: <password>
Confirm password: <password>
```

Обновить конфигурацию grub2:

```bash
grub2-mkconfig -o /boot/grub2/grub.cfg
```

Убедиться, что требуется аутентификация для однопользовательского режима (rescue mode), который используется для восстановления, когда система находит ошибку во время загрузки или во время ручного выбора в меню загрузчика. Это позволит не попасть в rescue mode без пароля, а именно этот режим был рассмотрен ранее для получения всех привилегий и заданий своего пароля для root.

Изменить или добавить строку в файле */usr/lib/systemd/system/rescue.service*:

```ini title="/usr/lib/systemd/system/rescue.service"
ExecStart=-/usr/lib/systemd/systemd-sulogin-shell rescue
```

Изменить или добавить сроку в файле */usr/lib/systemd/system/emergency.service*:

```ini title="/usr/lib/systemd/system/emergency.service"
ExecStart=-/usr/lib/systemd/systemd-sulogin-shell emergency
```

!!! warning "Внимание"
    Если после всех вышеуказанных действий забыть и пароль root, и пароль загрузчика, восстановить доступ к системе будет очень сложно. Поэтому надо их сохранить в безопасном месте.