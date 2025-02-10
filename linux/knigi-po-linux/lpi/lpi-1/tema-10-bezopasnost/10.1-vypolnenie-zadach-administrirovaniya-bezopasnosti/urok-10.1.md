# Урок 10.1

## Введение <a href="#sec.110.1_01-in" id="sec.110.1_01-in"></a>

Безопасность является обязательным условием при системном администрировании. Как хороший системный администратор Linux, вы должны следить за несколькими вещами, такими как специальные разрешения для файлов, устаревание паролей пользователей, открытые порты и сокеты, ограничение использования системных ресурсов, работа с пользователями, вошедшими в систему, и повышение привилегий с помощью `su` и `sudo`. В этом уроке мы рассмотрим каждую из этих тем.

## Проверка наличия файлов с установленными SUID и SGID <a href="#checking_for_files_with_the_suid_and_sgid_set" id="checking_for_files_with_the_suid_and_sgid_set"></a>

Помимо традиционного набора разрешений _на чтение_, _запись_ и _выполнение_, файлы в системе Linux могут также иметь специальные разрешения, такие как _SUID_ или _SGID_.

Бит SUID позволяет запускать файл с правами владельца файла. Он обозначается цифрой `4000` и символом `s` или `S` в бите _выполнения_ владельца. Классический пример исполняемого файла с установленным битом SUID — `passwd`:

```bash
carol@debian:~$ ls -l /usr/bin/passwd
-rwsr-xr-x 1 root root 63736 jul 27  2018 /usr/bin/passwd
```

Нижний регистр `s` в `rws` указывает на наличие SUID в файле — вместе с разрешением на _выполнение_. Заглавная буква `S` вместо (`rwS`) означала бы, что базовое разрешение на _выполнение_ не установлено.

{% hint style="info" %}
Вы узнаете о `passwd` в следующем разделе. Утилита в основном используется `root` для установки/изменения паролей пользователей (например, `passwd carol`). Однако обычные пользователи могут использовать её и для изменения собственных паролей. Поэтому она поставляется с набором SUID
{% endhint %}

С другой стороны, бит SGID может быть установлен как для файлов, так и для каталогов. Для файлов его поведение аналогично SUID, но привилегии соответствуют привилегиям владельца группы. Однако если он установлен для каталога, то все созданные в нём файлы будут наследовать владельца группы каталога. Как и SUID, SGID символически представлен либо `s` или `S` в бите _выполнения_ группы. Числовым значением он представлен `2000`. Вы можете установить SGID для каталога с помощью `chmod`. Вам нужно добавить `2` (SGID) к традиционным разрешениям (`755` в нашем случае):

```bash
carol@debian:~$ ls -ld shared_directory
drwxr-xr-x 2 carol carol 4096 may 30 23:55 shared_directory
carol@debian:~$ sudo chmod 2755 shared_directory/
carol@debian:~$ ls -ld shared_directory
drwxr-sr-x 2 carol carol 4096 may 30 23:55 shared_directory
```

Чтобы найти файлы с SUID или SGID, вы можете использовать команду `find` и опцию `-perm` . Вы можете использовать как числовые, так и символьные значения. Значения, в свою очередь, могут передаваться отдельно или с тире (`-`) или косой чертой (`/`). Это означает следующее:

`-perm`` `_`numeric-value`_ или `-perm`` `_`symbolic-value`_

находить файлы, имеющие специальное разрешение _исключительно_

`-perm`` `_`-numeric-value`_ или `-perm`` `_`-symbolic-value`_

поиск файлов, имеющих специальное разрешение и другие разрешения

`-perm /`_`numeric-value`_ или `-perm /`_`symbolic-value`_

поиск файлов, имеющих любое из специальных разрешений (и другие разрешения)

Например, чтобы найти файлы с _только_ установленным SUID в текущем рабочем каталоге, используйте следующую команду:

```
carol@debian:~$ find . -perm 4000
carol@debian:~$ touch file
carol@debian:~$ chmod 4000 file
carol@debian:~$ find . -perm 4000
./file
```

Обратите внимание, что, поскольку не было файлов с исключительно SUID-правами, мы создали один такой файл, чтобы показать результат. Вы можете выполнить ту же команду в символической форме:

```bash
carol@debian:~$ find . -perm u+s
./file
```

Чтобы найти файлы с SUID (независимо от других разрешений) в каталоге `/usr/bin/` , вы можете использовать одну из следующих команд:

```bash
carol@debian:~$ sudo find /usr/bin -perm -4000
/usr/bin/umount
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/chfn
/usr/bin/mount
/usr/bin/passwd
/usr/bin/chsh
/usr/bin/sudo
/usr/bin/su
carol@debian:~$ sudo find /usr/bin -perm -u+s
/usr/bin/umount
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/chfn
/usr/bin/mount
/usr/bin/passwd
/usr/bin/chsh
/usr/bin/sudo
/usr/bin/su
```

Если вы ищете файлы в том же каталоге с установленным битом SGID, вы можете выполнить `find /usr/bin/ -perm -2000` или `find /usr/bin/ -perm -g+s`.

Наконец, чтобы найти файлы с одним из двух специальных разрешений, добавьте `4` и `2` и используйте `/`:

```bash
carol@debian:~$ sudo find /usr/bin -perm /6000
/usr/bin/dotlock.mailutils
/usr/bin/umount
/usr/bin/newgrp
/usr/bin/wall
/usr/bin/ssh-agent
/usr/bin/chage
/usr/bin/dotlockfile
/usr/bin/gpasswd
/usr/bin/chfn
/usr/bin/mount
/usr/bin/passwd
/usr/bin/chsh
/usr/bin/expiry
/usr/bin/sudo
/usr/bin/bsd-write
/usr/bin/crontab
/usr/bin/su
```

## Управление паролями и устаревание <a href="#password_management_and_aging" id="password_management_and_aging"></a>

Как отмечалось выше, вы можете использовать утилиту `passwd` для смены пароля обычного пользователя. Кроме того, вы можете передать параметр `-S` или `--status` для получения информации о статусе вашей учётной записи:

```bash
carol@debian:~$ passwd -S
carol P 12/07/2019 0 99999 7 -1
```

Вот разбивка по семи полям, которые вы получаете в выходных данных:

`carol`

Имя пользователя для входа в систему.

`P`

Это означает, что у пользователя есть действующий пароль (`P`); другие возможные значения — `L` для заблокированного пароля и `NP` для отсутствия пароля.

`12/07/2019`

Дата последней смены пароля.

`0`

Минимальный срок в днях (минимальное количество дней между сменами пароля). Значение `0` означает, что пароль можно сменить в любое время.

`99999`

Максимальный срок действия в днях (максимальное количество дней, в течение которых действителен пароль). Значение `99999` отключает истечение срока действия пароля.

`7`

Срок предупреждения в днях (количество дней до истечения срока действия пароля, за которые пользователь будет предупреждён).

`-1`

Период бездействия пароля в днях (количество дней бездействия после истечения срока действия пароля до блокировки учетной записи). Значение `-1` устранит бездействие учетной записи.

Помимо отчётов о состоянии учётных записей, вы будете использовать команду `passwd` от имени пользователя root для выполнения основных действий по обслуживанию учётных записей. Вы можете блокировать и разблокировать учётные записи, заставить пользователя сменить пароль при следующем входе в систему и удалить пароль пользователя с помощью опций `-l`, `-u`, `-e` и `-d` соответственно.

Чтобы протестировать эти параметры, на этом этапе удобно ввести команду `su`'. С помощью `su` вы можете переключаться между пользователями во время сеанса входа в систему. Так, например, давайте воспользуемся `passwd`' для блокировки пароля `carol`'. Затем мы переключимся на `carol`' и проверим статус нашей учётной записи, чтобы убедиться, что пароль действительно заблокирован (`L`) и не может быть изменён. Наконец, вернувшись к пользователю root, мы разблокируем пароль `carol`':

```bash
root@debian:~# passwd -l carol
passwd: password expiry information changed.
root@debian:~# su - carol
carol@debian:~$ passwd -S
carol L 05/31/2020 0 99999 7 -1
carol@debian:~$ passwd
Changing password for carol.
Current password:
passwd: Authentication token manipulation error
passwd: password unchanged
carol@debian:~$ exit
logout
root@debian:~# passwd -u carol
passwd: password expiry information changed.
```

Кроме того, вы можете заблокировать и разблокировать пароль пользователя с помощью команды `usermod`:

Заблокировать пароль пользователя `carol`

`usermod -L carol` или`usermod --lock carol`.

Разблокировать пароль пользователя `carol`

`usermod -U carol` или`usermod --unlock carol`.

{% hint style="info" %}
С помощью переключателей `-f` или `--inactive` `usermod` также можно задать количество дней до отключения учётной записи с истёкшим сроком действия пароля (например, `usermod -f 3 carol`).
{% endhint %}

Помимо `passwd` и `usermod`, наиболее прямой командой для решения проблемы старения пароля и учетной записи является `chage` (“изменить возраст”). Как пользователь root, вы можете передать `chage` переключатель `-l` (или `--list`), за которым следует имя пользователя, чтобы на экране отображался текущий пароль этого пользователя и информация об истечении срока действия учетной записи; как обычный пользователь, вы можете просматривать свою собственную информацию:

```bash
carol@debian:~$ chage -l carol
Last password change					: Aug 06, 2019
Password expires					: never
Password inactive					: never
Account expires						: never
Minimum number of days between password change		: 0
Maximum number of days between password change		: 99999
Number of days of warning before password expires	: 7
```

При запуске без параметров и только с указанием имени пользователя `chage` будет работать в интерактивном режиме:

```bash
root@debian:~# chage carol
Changing the aging information for carol
Enter the new value, or press ENTER for the default

	Minimum Password Age [0]:
	Maximum Password Age [99999]:
	Last Password Change (YYYY-MM-DD) [2020-06-01]:
	Password Expiration Warning [7]:
	Password Inactive [-1]:
	Account Expiration Date (YYYY-MM-DD) [-1]:
```

Ниже приведены варианты изменения различных `chage` настроек:

`-m`` `_`days`_ _`username`_ или `--mindays`` `_`days`_ _`username`_

Укажите минимальное количество дней между сменами пароля (например: `chage -m 5 carol`). Значение `0` позволит пользователю сменить пароль в любое время.

`-M`` `_`days`_ _`username`_ или `--maxdays`` `_`days`_ _`username`_

Укажите максимальное количество дней, в течение которых будет действителен пароль (например: `chage -M 30 carol`). Чтобы отключить истечение срока действия пароля, обычно устанавливают для этого параметра значение `99999`.

`-d`` `_`days`_ _`username`_ или `--lastday`` `_`days`_ _`username`_

Укажите количество дней, прошедших с момента последней смены пароля (например, `chage -d 10 carol`). Значение `0` заставит пользователя сменить пароль при следующем входе в систему.

`-W`` `_`days`_ _`username`_ или `--warndays`` `_`days`_ _`username`_

Укажите, через сколько дней пользователю будет напоминаться об истечении срока действия его пароля.

`-I`` `_`days`_ _`username`_ или `--inactive`` `_`days`_ _`username`_

Укажите количество дней бездействия после истечения срока действия пароля (например, `chage -I 10 carol`). Это то же самое, что `usermod -f` или `usermod --inactive`. По истечении указанного количества дней учётная запись будет заблокирована. Однако при значении `0` учётная запись не будет заблокирована.

`-E`` `_`date`_ _`username`_ или `--expiredate`` `_`date`_ _`username`_

Укажите дату (или количество дней с _начала эпохи_ — 1 января 1970 года), когда аккаунт будет заблокирован. Обычно она указывается в формате `YYYY-MM-DD` (например, `chage -E 2050-12-13 carol`).

{% hint style="info" %}
Вы можете узнать больше о `passwd`, `usermod` и `chage` - и их возможностях — на соответствующих страницах руководства.
{% endhint %}

## Обнаружение Открытых Портов <a href="#discovering_open_ports" id="discovering_open_ports"></a>

Для отслеживания открытых портов в большинстве систем Linux есть четыре мощные утилиты: `lsof`, `fuser`, `netstat` и `nmap`. Мы рассмотрим их в этом разделе.

`lsof` означает «список открытых файлов list open files», что немаловажно, учитывая, что в Linux всё является файлом. На самом деле, если вы введёте `lsof` в терминале, вы получите большой список обычных файлов, файлов устройств, сокетов и т. д. Однако в этом уроке мы сосредоточимся в основном на портах. Чтобы вывести список всех сетевых файлов «Интернет», запустите `lsof` с опцией `-i`:

```bash
root@debian:~# lsof -i
COMMAND  PID     USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
dhclient 357     root    7u  IPv4  13493      0t0  UDP *:bootpc
sshd     389     root    3u  IPv4  13689      0t0  TCP *:ssh (LISTEN)
sshd     389     root    4u  IPv6  13700      0t0  TCP *:ssh (LISTEN)
apache2  399     root    3u  IPv6  13826      0t0  TCP *:http (LISTEN)
apache2  401 www-data    3u  IPv6  13826      0t0  TCP *:http (LISTEN)
apache2  402 www-data    3u  IPv6  13826      0t0  TCP *:http (LISTEN)
sshd     557     root    3u  IPv4  14701      0t0  TCP 192.168.1.7:ssh->192.168.1.4:60510 (ESTABLISHED)
sshd     569    carol    3u  IPv4  14701      0t0  TCP 192.168.1.7:ssh->192.168.1.4:60510 (ESTABLISHED)
```

Помимо службы `bootpc` — которая используется DHCP — в выводе показаны две службы, ожидающие подключения, — `ssh` и веб-сервер Apache (`http`), — а также два установленных SSH-соединения. Вы можете указать конкретный хост с помощью `@`_`ip-address`_ для проверки его подключений:

```bash
root@debian:~# lsof -i@192.168.1.7
COMMAND PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
sshd    557  root    3u  IPv4  14701      0t0  TCP 192.168.1.7:ssh->192.168.1.4:60510 (ESTABLISHED)
sshd    569 carol    3u  IPv4  14701      0t0  TCP 192.168.1.7:ssh->192.168.1.4:60510 (ESTABLISHED)
```

{% hint style="info" %}
Чтобы вывести на печать только сетевые файлы IPv4 и IPv6, используйте параметры `-i4` и `-i6` соответственно.
{% endhint %}

Аналогичным образом вы можете выполнить фильтрацию по порту, передав в качестве аргумента `-i` (или `-i@`_`ip-address`_) параметр _`:port`_:

```bash
root@debian:~# lsof -i :22
COMMAND PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
sshd    389  root    3u  IPv4  13689      0t0  TCP *:ssh (LISTEN)
sshd    389  root    4u  IPv6  13700      0t0  TCP *:ssh (LISTEN)
sshd    557  root    3u  IPv4  14701      0t0  TCP 192.168.1.7:ssh->192.168.1.4:60510 (ESTABLISHED)
sshd    569 carol    3u  IPv4  14701      0t0  TCP 192.168.1.7:ssh->192.168.1.4:60510 (ESTABLISHED)
```

Несколько портов разделяются запятыми (а диапазоны указываются с помощью тире):

```bash
root@debian:~# lsof -i@192.168.1.7:22,80
COMMAND PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
sshd    705  root    3u  IPv4  13960      0t0  TCP 192.168.1.7:ssh->192.168.1.4:44766 (ESTABLISHED)
sshd    718 carol    3u  IPv4  13960      0t0  TCP 192.168.1.7:ssh->192.168.1.4:44766 (ESTABLISHED)
```

{% hint style="info" %}
Количество доступных для `lsof` опций довольно велико. Чтобы узнать больше, обратитесь к руководству.
{% endhint %}

Следующей в списке сетевых команд является `fuser`. Её основная цель — найти «пользователя файла», то есть узнать, какие процессы получают доступ к каким файлам. Она также предоставляет некоторую другую информацию, например, о типе доступа. Например, чтобы проверить текущий рабочий каталог, достаточно выполнить `fuser .` Однако, чтобы получить немного больше информации, удобно использовать параметр подробного вывода (`-v` или `--verbose`):

```bash
root@debian:~# fuser .
/root:                 580c
root@debian:~# fuser -v .
                     USER        PID ACCESS COMMAND
/root:               root        580 ..c.. bash
```

Давайте разберем выходные данные:

Файл

Файл, о котором мы получаем информацию (`/root`).

`USER`

Владелец файла (`root`).

`PID`&#x20;

Идентификатор процесса (`580`).

`ACCESS`

Тип доступа (`..c..`). Один из:

`c`

Текущий каталог.

`e`

Исполняемый файл запускается.

`f`

Открыт файл (в режиме отображения по умолчанию он опущен).

`F`

Открыт файл для записи (в режиме отображения по умолчанию он опущен).

`r`

корневой каталог.

`m`

файл mmap'ed или общая библиотека.

`.`

Заполнитель (опущен в режиме отображения по умолчанию).

`COMMAND` Колонна

Команда, связанная с файлом (`bash`).

С помощью опции `-n` (или `--namespace`) вы можете найти информацию о сетевых портах/сокетах. Вам также необходимо указать сетевой протокол и номер порта. Таким образом, чтобы получить информацию о веб-сервере Apache, выполните следующую команду:

```
root@debian:~# fuser -vn tcp 80
                     USER        PID ACCESS COMMAND
80/tcp:              root        402 F.... apache2
                     www-data    404 F.... apache2
                     www-data    405 F.... apache2
```

{% hint style="info" %}
`fuser` Также можно использовать `-k` или `--kill` для завершения процессов, обращающихся к файлу (например, `fuser -k 80/tcp`). Более подробную информацию см. в руководстве.
{% endhint %}

Давайте обратимся к `netstat` теперь. `netstat` это очень универсальный сетевой инструмент, который в основном используется для печати "сетевой статистики”.

При запуске без параметров `netstat` отобразит как активные интернет-соединения, так и сокеты Unix. Из-за большого объёма списка вы можете захотеть перенаправить его вывод через `less`:

```bash
carol@debian:~$ netstat |less
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 192.168.1.7:ssh         192.168.1.4:55444       ESTABLISHED
Active UNIX domain sockets (w/o servers)
Proto RefCnt Flags       Type       State         I-Node   Path
unix  2      [ ]         DGRAM                    10509    /run/systemd/journal/syslog
unix  3      [ ]         DGRAM                    10123    /run/systemd/notify
(...)
```

Чтобы вывести список только «прослушиваемых» портов и сокетов, используйте параметры `-l` или `--listening`. Параметры `-t`/`--tcp` и `-u`/`--udp` можно добавить для фильтрации по протоколу TCP и UDP соответственно (их также можно комбинировать в одной команде). Аналогично, `-e`/`--extend` отобразит дополнительную информацию:

```bash
carol@debian:~$ netstat -lu
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
udp        0      0 0.0.0.0:bootpc          0.0.0.0:*
carol@debian:~$ netstat -lt
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN
tcp        0      0 localhost:smtp          0.0.0.0:*               LISTEN
tcp6       0      0 [::]:http               [::]:*                  LISTEN
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN
tcp6       0      0 localhost:smtp          [::]:*                  LISTEN
carol@debian:~$ netstat -lute
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       User       Inode
tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN      root       13729
tcp        0      0 localhost:smtp          0.0.0.0:*               LISTEN      root       14372
tcp6       0      0 [::]:http               [::]:*                  LISTEN      root       14159
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN      root       13740
tcp6       0      0 localhost:smtp          [::]:*                  LISTEN      root       14374
udp        0      0 0.0.0.0:bootpc          0.0.0.0:*                           root       13604
```

Если вы опустите `-l` опцию, будут показаны _только_ установленные соединения:

```bash
carol@debian:~$ netstat -ute
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       User       Inode
tcp        0      0 192.168.1.7:ssh         192.168.1.4:39144       ESTABLISHED root       15103
```

Если вас интересует только числовая информация о портах и хостах, вы можете использовать опцию `-n` или `--numeric` для вывода только номеров портов и IP-адресов. Обратите внимание, что `ssh` превращается в `22` при добавлении `-n` к приведенной выше команде:

```
carol@debian:~$ netstat -uten
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       User       Inode
tcp        0      0 192.168.1.7:22          192.168.1.4:39144       ESTABLISHED 0          15103
```

Как видите, вы можете создавать очень полезные и продуктивные `netstat` команды, комбинируя некоторые из их параметров. Просмотрите справочные страницы, чтобы узнать больше и найти сочетания, которые лучше всего соответствуют вашим потребностям.

Наконец, мы познакомим вас с `nmap` — или «network mapper». Это ещё одна очень мощная утилита, которая сканирует порты, если указать IP-адрес или имя хоста:

```bash
root@debian:~# nmap localhost
Starting Nmap 7.70 ( https://nmap.org ) at 2020-06-04 19:29 CEST
Nmap scan report for localhost (127.0.0.1)
Host is up (0.0000040s latency).
Other addresses for localhost (not scanned): ::1
Not shown: 998 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 1.58 seconds
```

Помимо отдельного хоста, `nmap` позволяет вам сканировать:

несколько хостов

разделяя их пробелами (например: `nmap localhost 192.168.1.7`).

диапазоны хостов

с помощью тире (например: `nmap 192.168.1.3-20`).

подсети

с помощью подстановочного знака или обозначения CIDR (например, `nmap 192.168.1.*` или `nmap 192.168.1.0/24`). Вы можете исключить определённые хосты (например, `nmap 192.168.1.0/24 --exclude 192.168.1.7`).

Чтобы просканировать конкретный порт, используйте переключатель `-p` с последующим указанием номера порта или имени службы (`nmap -p 22` и `nmap -p ssh` выдадут одинаковый результат):

```bash
root@debian:~# nmap -p 22 localhost
Starting Nmap 7.70 ( https://nmap.org ) at 2020-06-04 19:54 CEST
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000024s latency).
Other addresses for localhost (not scanned): ::1

PORT   STATE SERVICE
22/tcp open  ssh

Nmap done: 1 IP address (1 host up) scanned in 0.22 seconds
```

Вы также можете просканировать несколько портов или диапазонов портов, используя запятые и тире соответственно:

```bash
root@debian:~# nmap -p ssh,80 localhost
Starting Nmap 7.70 ( https://nmap.org ) at 2020-06-04 19:58 CEST
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000051s latency).
Other addresses for localhost (not scanned): ::1

PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 0.22 seconds
```

```bash
root@debian:~# nmap -p 22-80 localhost
Starting Nmap 7.70 ( https://nmap.org ) at 2020-06-04 19:58 CEST
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000011s latency).
Other addresses for localhost (not scanned): ::1
Not shown: 57 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 1.47 seconds
```

Двумя другими важными и удобными `nmap` опциями являются:

`-F`

Выполните быстрое сканирование 100 наиболее распространенных портов.

`-v`

Получите подробный вывод (`-vv` выведет еще более подробный вывод).

{% hint style="info" %}
&#x20;С помощью типов сканирования можно выполнять довольно сложные команды. Однако эта тема выходит за рамки данного урока.
{% endhint %}

