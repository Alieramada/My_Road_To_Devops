# Урок 8.2.2.

## Введение <a href="#sec.108.2_02-in" id="sec.108.2_02-in"></a>

С повсеместным внедрением `systemd` во всех основных дистрибутивах демон журнала (`systemd-journald`) стал стандартной службой ведения журнала. В этом уроке мы обсудим, как он работает и как его можно использовать для различных целей: запрашивать информацию, фильтровать её по разным критериям, настраивать хранилище и размер, удалять старые данные, извлекать данные из резервной системы или копии файловой системы и, что не менее важно, понимать его взаимодействие с `rsyslogd`.

## Основы `systemd` <a href="#basics_of_systemd" id="basics_of_systemd"></a>

Впервые представленный в Fedora, `systemd` постепенно заменил SysV Init в качестве _фактического_ системного и сервисного менеджера в большинстве основных дистрибутивов Linux. Среди его преимуществ можно выделить следующие:

* Простота настройки: файлы модулей в отличие от скриптов SysV Init.
* Универсальное управление: помимо демонов и процессов, он также управляет устройствами, сокетами и точками монтирования.
* Обратная совместимость как с SysV Init, так и с Upstart.
* Параллельная загрузка во время запуска: службы загружаются параллельно, в отличие от последовательной загрузки Sysv Init.
* Он включает службу ведения журнала под названием _journal_, которая обладает следующими преимуществами:
  * Он объединяет все журналы в одном месте.
  * Не требует ротации журналов.
  * Журналы можно отключить, загрузить в оперативную память или сделать постоянными.

Units и targets

`systemd` действует на _units_. Unit — это любой ресурс, которым `systemd` можно управлять (например, сеть, Bluetooth и т.д.). Units, в свою очередь, управляются _файлами unitов_. Это обычные текстовые файлы, которые хранятся в `/lib/systemd/system` и включают параметры конфигурации — в виде _разделов_ и _директив_ — для конкретного ресурса, которым нужно управлять. Существует несколько типов юнитов: `service`, `mount`, `automount`, `swap` `timer`, `device` `socket` `path`, `timer` `snapshotslice`, `scope` `target`. Таким образом, каждое имя unit файла следует шаблону `<resource_name>.<unit_type>` (например, `reboot.service`).

target — это особый тип единицы, напоминающий классические уровни выполнения SysV Init. Это связано с тем, что _target_ объединяет различные ресурсы для представления определённого состояния системы (например, `graphical.target` похожа на `runlevel 5` и т. д.). Чтобы проверить текущий target в вашей системе, используйте команду `systemctl get-default`:

```bash
carol@debian:~$ systemctl get-default
graphical.target
```

С другой стороны, target и уровни выполнения отличаются тем, что первые являются взаимоисключающими, а вторые — нет. Таким образом, target может вызывать другие targets, что невозможно при использовании уровней выполнения.

## Системный журнал: `systemd-journald` <a href="#the_system_journal_systemd_journald" id="the_system_journal_systemd_journald"></a>

`systemd-journald` Это системная служба, которая отвечает за получение информации из различных источников: сообщений ядра, простых и структурированных системных сообщений, стандартного вывода и стандартных ошибок служб, а также записей аудита из подсистемы аудита ядра (подробнее см. в руководстве по `systemd-journald`). Её задача — создавать и поддерживать структурированный и индексированный журнал.

Его файл конфигурации — `/etc/systemd/journald.conf` и, как и в случае с любой другой службой, вы можете использовать команду `systemctl` для _запуска_, _перезапуска_, _остановки_ или — просто — проверки его _состояния_:

```bash
root@debian:~# systemctl status systemd-journald
 systemd-journald.service - Journal Service
   Loaded: loaded (/lib/systemd/system/systemd-journald.service; static; vendor preset: enabled)
   Active: active (running) since Sat 2019-10-12 13:43:06 CEST; 5min ago
     Docs: man:systemd-journald.service(8)
           man:journald.conf(5)
 Main PID: 178 (systemd-journal)
   Status: "Processing requests..."
    Tasks: 1 (limit: 4915)
   CGroup: /system.slice/systemd-journald.service
           └─178 /lib/systemd/systemd-journald
(...)
```

Также возможны файлы конфигурации типа `journal.conf.d/*.conf` — которые могут включать конфигурации для конкретных пакетов (подробнее см. на странице руководства `journald.conf`).

Если эта функция включена, журнал может храниться либо постоянно на диске, либо в энергозависимой файловой системе на основе оперативной памяти. Журнал — это не обычный текстовый файл, он двоичный. Поэтому для чтения его содержимого нельзя использовать инструменты для анализа текста, такие как `less` или `more`; вместо этого используется команда `journalctl`

## **Запрос содержимого журнала**

`journalctl` Это утилита, которую вы используете для запроса журнала `systemd` . Для её запуска вам нужно либо быть пользователем root, либо использовать `sudo` . Если вы запрашиваете журнал без параметров, он будет выводиться в хронологическом порядке (сначала самые старые записи):

```bash
root@debian:~# journalctl
-- Logs begin at Sat 2019-10-12 13:43:06 CEST, end at Sat 2019-10-12 14:19:46 CEST. --
Oct 12 13:43:06 debian kernel: Linux version 4.9.0-9-amd64 (debian-kernel@lists.debian.org) (...)
Oct 12 13:43:06 debian kernel: Command line: BOOT_IMAGE=/boot/vmlinuz-4.9.0-9-amd64 root=UUID=b6be6117-5226-4a8a-bade-2db35ccf4cf4 ro qu
(...)
```

Вы можете выполнять более конкретные запросы, используя ряд переключателей:

`-r`

Сообщения журнала будут напечатаны в обратном порядке:

```bash
root@debian:~# journalctl -r
-- Logs begin at Sat 2019-10-12 13:43:06 CEST, end at Sat 2019-10-12 14:30:30 CEST. --
Oct 12 14:30:30 debian sudo[1356]: pam_unix(sudo:session): session opened for user root by carol(uid=0)
Oct 12 14:30:30 debian sudo[1356]:    carol : TTY=pts/0 ; PWD=/home/carol ; USER=root ; COMMAND=/bin/journalctl -r
Oct 12 14:19:53 debian sudo[1348]: pam_unix(sudo:session): session closed for user root
(...)
```

`-f`

&#x20;Выведет на печать самые последние сообщения журнала и продолжит выводить новые записи по мере их добавления в журнал — почти как `tail -f`:

```bash
root@debian:~# journalctl -f
-- Logs begin at Sat 2019-10-12 13:43:06 CEST. --
(...)
Oct 12 14:44:42 debian sudo[1356]: pam_unix(sudo:session): session closed for user root
Oct 12 14:44:44 debian sudo[1375]:    carol : TTY=pts/0 ; PWD=/home/carol ; USER=root ; COMMAND=/bin/journalctl -f
Oct 12 14:44:44 debian sudo[1375]: pam_unix(sudo:session): session opened for user root by carol(uid=0)

(...)
```

`-e`

Перейдёт в конец журнала, чтобы последние записи были видны на странице:

```bash
root@debian:~# journalctl -e
(...)
Oct 12 14:44:44 debian sudo[1375]:    carol : TTY=pts/0 ; PWD=/home/carol ; USER=root ; COMMAND=/bin/journalctl -f
Oct 12 14:44:44 debian sudo[1375]: pam_unix(sudo:session): session opened for user root by carol(uid=0)
Oct 12 14:45:57 debian sudo[1375]: pam_unix(sudo:session): session closed for user root
Oct 12 14:48:39 debian sudo[1378]:    carol : TTY=pts/0 ; PWD=/home/carol ; USER=root ; COMMAND=/bin/journalctl -e
Oct 12 14:48:39 debian sudo[1378]: pam_unix(sudo:session): session opened for user root by carol(uid=0)
```

`-n <value>, --lines=<value>`

Выведет _значение_ последних строк (если `<value>` не указано, по умолчанию используется значение 10):

```bash
root@debian:~# journalctl -n 5
(...)
Oct 12 14:44:44 debian sudo[1375]:    carol : TTY=pts/0 ; PWD=/home/carol ; USER=root ; COMMAND=/bin/journalctl -f
Oct 12 14:44:44 debian sudo[1375]: pam_unix(sudo:session): session opened for user root by carol(uid=0)
Oct 12 14:45:57 debian sudo[1375]: pam_unix(sudo:session): session closed for user root
Oct 12 14:48:39 debian sudo[1378]:    carol : TTY=pts/0 ; PWD=/home/carol ; USER=root ; COMMAND=/bin/journalctl -e
Oct 12 14:48:39 debian sudo[1378]: pam_unix(sudo:session): session opened for user root by carol(uid=0)
```

`-k`, `--dmesg`

Эквивалентно использованию команды `dmesg`:

```bash
root@debian:~# journalctl -k
-- Logs begin at Sat 2019-10-12 13:43:06 CEST, end at Sat 2019-10-12 14:53:20 CEST. --
Oct 12 13:43:06 debian kernel: Linux version 4.9.0-9-amd64 (debian-kernel@lists.debian.org) (gcc version 6.3.0 20170516 (Debian 6.3.0-18
Oct 12 13:43:06 debian kernel: Command line: BOOT_IMAGE=/boot/vmlinuz-4.9.0-9-amd64 root=UUID=b6be6117-5226-4a8a-bade-2db35ccf4cf4 ro qu
Oct 12 13:43:06 debian kernel: x86/fpu: Supporting XSAVE feature 0x001: 'x87 floating point registers'
Oct 12 13:43:06 debian kernel: x86/fpu: Supporting XSAVE feature 0x002: 'SSE registers'
(...)
```

## **Навигация и поиск по журналу**

Вы можете перемещаться по выходу журнала с помощью:

* Клавиши PageUp, PageDown и стрелки для перемещения вверх, вниз, влево и вправо.
* <kbd>></kbd> для перехода к концу вывода.
* <kbd><</kbd> для перехода к началу вывода.

Вы можете выполнять поиск строк как в прямом, так и в обратном направлении от вашего текущего положения просмотра:

* Прямой поиск: нажмите <kbd>/</kbd> и введите строку для поиска, затем нажмите Enter.
* Обратный поиск: нажмите <kbd>?</kbd> и введите строку для поиска, затем нажмите Enter.

Чтобы перемещаться по совпадениям в результатах поиска, используйте <kbd>N</kbd> для перехода к следующему совпадению и <kbd>Shift</kbd>+<kbd>N</kbd> для перехода к предыдущему.

## **Фильтрация данных журнала**

Журнал позволяет фильтровать данные журнала по различным критериям:

Номер загрузки`--list-boots`

В нём перечислены все доступные загрузки. Вывод состоит из трёх столбцов: первый указывает номер загрузки (`0` относится к текущей загрузке, `-1` — к предыдущей, `-2` — к предыдущей предыдущей и так далее); второй столбец — это идентификатор загрузки; в третьем столбце указаны временные метки:

```bash
root@debian:~# journalctl --list-boots
 0 83df3e8653474ea5aed19b41cdb45b78 Sat 2019-10-12 18:55:41 CEST—Sat 2019-10-12 19:02:24 CEST
```

`-b`, `--boot`

Отображаются все сообщения текущей загрузки. Чтобы увидеть сообщения журнала предыдущих загрузок, просто добавьте параметр смещения, как описано выше. Например, чтобы отобразить сообщения предыдущей загрузки, введите `journalctl -b -1`. Однако помните, что для восстановления информации из предыдущих журналов необходимо включить сохранение журнала (вы узнаете, как это сделать, в следующем разделе):

```
root@debian:~# journalctl -b -1
Specifying boot ID has no effect, no persistent journal was found
```

Приоритет`-p`

Интересно, что вы также можете отфильтровать сообщения по степени важности/приоритетности с помощью опции `-p`:

```
root@debian:~# journalctl -b -0 -p err
-- No entries --
```

Журнал сообщает нам, что на данный момент не было сообщений с приоритетом `error` (или выше) от текущей загрузки. Примечание: `-b -0` можно опустить, если речь идёт о текущей загрузке.

## Временной интервал

Вы можете использовать `journalctl` для вывода только сообщений, записанных в течение определенного периода времени, с помощью переключателей `--since` и `--until`. Дата должна быть указана в формате `YYYY-MM-DD HH:MM:SS`. Если опустить компонент времени, будет принята полночь. Аналогично, если опустить дату, будет принята текущая дата. Например, чтобы увидеть сообщения, записанные с 19:00 до 19:01, введите:

```bash
root@debian:~# journalctl --since "19:00:00" --until "19:01:00"
-- Logs begin at Sat 2019-10-12 18:55:41 CEST, end at Sat 2019-10-12 20:10:50 CEST. --
Oct 12 19:00:14 debian systemd[1]: Started Run anacron jobs.
Oct 12 19:00:14 debian anacron[1057]: Anacron 2.3 started on 2019-10-12
Oct 12 19:00:14 debian anacron[1057]: Normal exit (0 jobs run)
Oct 12 19:00:14 debian systemd[1]: anacron.timer: Adding 2min 47.988096s random time.
```

Аналогичным образом вы можете использовать немного другую спецификацию времени: `"integer time-unit ago"`. Таким образом, чтобы увидеть сообщения, отправленные две минуты назад, вы вводите `sudo journalctl --since "2 minutes ago"`. Также можно использовать `+` и `-` для указания времени относительно текущего времени, поэтому `--since "-2 minutes"` и `--since "2 minutes ago"` эквивалентны.

Помимо числовых выражений, вы можете указать ряд ключевых слов:

`yesterday`

По состоянию на полночь дня, предшествующего текущему дню.

`today`

По состоянию на полночь текущего дня.

`tomorrow`

По состоянию на полночь следующего за текущим дня.

`now`

Текущее время.

Давайте посмотрим все сообщения с полуночи прошлого дня до сегодняшнего вечера в 21:00:

```bash
root@debian:~# journalctl --since "today" --until "21:00:00"
-- Logs begin at Sat 2019-10-12 20:45:29 CEST, end at Sat 2019-10-12 21:06:15 CEST. --
Oct 12 20:45:29 debian sudo[1416]:    carol : TTY=pts/0 ; PWD=/home/carol ; USER=root ; COMMAND=/bin/systemctl r
Oct 12 20:45:29 debian sudo[1416]: pam_unix(sudo:session): session opened for user root by carol(uid=0)
Oct 12 20:45:29 debian systemd[1]: Stopped Flush Journal to Persistent Storage.
(...)
```

{% hint style="info" %}
Чтобы узнать больше о различных синтаксисах для указания времени, обратитесь к странице руководства `systemd.time`.
{% endhint %}

## Программа

Чтобы просмотреть сообщения журнала, относящиеся к определённому исполняемому файлу, используется следующий синтаксис: `journalctl`` `_`/path/to/executable`_:

```bash
root@debian:~# journalctl /usr/sbin/sshd
-- Logs begin at Sat 2019-10-12 20:45:29 CEST, end at Sat 2019-10-12 21:54:49 CEST. --
Oct 12 21:16:28 debian sshd[1569]: Accepted password for carol from 192.168.1.65 port 34050 ssh2
Oct 12 21:16:28 debian sshd[1569]: pam_unix(sshd:session): session opened for user carol by (uid=0)
Oct 12 21:16:54 debian sshd[1590]: Accepted password for carol from 192.168.1.65 port 34052 ssh2
Oct 12 21:16:54 debian sshd[1590]: pam_unix(sshd:session): session opened for user carol by (uid=0)
```

## Unit

Помните, что Unit — это любой ресурс, обрабатываемый `systemd` и по которому вы тоже можете выполнять фильтрацию.

`-u`

Он показывает сообщения об указанном устройстве:

```bash
root@debian:~# journalctl -u ssh.service
-- Logs begin at Sun 2019-10-13 10:50:59 CEST, end at Sun 2019-10-13 12:22:59 CEST. --
Oct 13 10:51:00 debian systemd[1]: Starting OpenBSD Secure Shell server...
Oct 13 10:51:00 debian sshd[409]: Server listening on 0.0.0.0 port 22.
Oct 13 10:51:00 debian sshd[409]: Server listening on :: port 22.
(...)
```

{% hint style="info" %}
Чтобы вывести на печать все загруженные и активные модули, используйте `systemctl list-units`; чтобы просмотреть все установленные файлы модулей, используйте `systemctl list-unit-files`.
{% endhint %}

## Поля

Журнал также можно отфильтровать по определённым _полям_ с помощью любого из следующих синтаксисов:

* `<field-name>=<value>`
* `_<field-name>=<value>_`
*   `__<field-name>=<value>`

    `PRIORITY=`

    Одно из восьми возможных `syslog` значений приоритета, отформатированных в виде десятичной строки:

```bash
root@debian:~# journalctl PRIORITY=3
-- Logs begin at Sun 2019-10-13 10:50:59 CEST, end at Sun 2019-10-13 14:30:50 CEST. --
Oct 13 10:51:00 debian avahi-daemon[314]: chroot.c: open() failed: No such file or directory
```

Обратите внимание, что вы могли бы получить тот же результат, используя команду `sudo journalctl -p err` из примера выше.

`SYSLOG_FACILITY=`

Любой из возможных номеров объектов, представленных в виде десятичной строки. Например, чтобы просмотреть все сообщения на уровне пользователя:

```bash
root@debian:~# journalctl SYSLOG_FACILITY=1
-- Logs begin at Sun 2019-10-13 10:50:59 CEST, end at Sun 2019-10-13 14:42:52 CEST. --
Oct 13 10:50:59 debian mtp-probe[227]: checking bus 1, device 2: "/sys/devices/pci0000:00/0000:00:06.0/usb1/1-1"
Oct 13 10:50:59 debian mtp-probe[227]: bus: 1, device: 2 was not an MTP device
Oct 13 10:50:59 debian mtp-probe[238]: checking bus 1, device 2: "/sys/devices/pci0000:00/0000:00:06.0/usb1/1-1"
Oct 13 10:50:59 debian mtp-probe[238]: bus: 1, device: 2 was not an MTP device
```

`_PID=`

Показать сообщения, созданные с помощью определённого идентификатора процесса. Чтобы увидеть все сообщения, созданные с помощью `systemd`, введите:

```bash
root@debian:~# journalctl _PID=1
-- Logs begin at Sun 2019-10-13 10:50:59 CEST, end at Sun 2019-10-13 14:50:15 CEST. --
Oct 13 10:50:59 debian systemd[1]: Mounted Debug File System.
Oct 13 10:50:59 debian systemd[1]: Mounted POSIX Message Queue File System.
Oct 13 10:50:59 debian systemd[1]: Mounted Huge Pages File System.
Oct 13 10:50:59 debian systemd[1]: Started Remount Root and Kernel File Systems.
Oct 13 10:50:59 debian systemd[1]: Starting Flush Journal to Persistent Storage...
(...)
```

`_BOOT_ID`

На основе идентификатора загрузки вы можете выделить сообщения от конкретной загрузки, например: `sudo journalctl _BOOT_ID=83df3e8653474ea5aed19b41cdb45b78`.

`_TRANSPORT`

Показать сообщения, полученные с помощью определённого транспорта. Возможные значения: `audit` (подсистема аудита ядра), `driver` (сгенерировано внутри), `syslog` (сокет syslog), `journal` (собственный протокол журнала), `stdout` (стандартный вывод или стандартная ошибка службы), `kernel` (кольцевой буфер ядра — то же самое, что `dmesg`, `journalctl -k` или `journalctl --dmesg`):

```bash
root@debian:~# journalctl _TRANSPORT=journal
-- Logs begin at Sun 2019-10-13 20:19:58 CEST, end at Sun 2019-10-13 20:46:36 CEST. --
Oct 13 20:19:58 debian systemd[1]: Started Create list of required static device nodes for the current kernel.
Oct 13 20:19:58 debian systemd[1]: Starting Create Static Device Nodes in /dev...
Oct 13 20:19:58 debian systemd[1]: Started Create Static Device Nodes in /dev.
Oct 13 20:19:58 debian systemd[1]: Starting udev Kernel Device Manager...
(...)
```

## **Объединение полей**

Поля не являются взаимоисключающими, поэтому вы можете использовать несколько полей в одном запросе. Однако будут отображаться только те сообщения, которые соответствуют значениям обоих полей одновременно:

```bash
root@debian:~# journalctl PRIORITY=3 SYSLOG_FACILITY=0
-- No entries --
root@debian:~# journalctl PRIORITY=4 SYSLOG_FACILITY=0
-- Logs begin at Sun 2019-10-13 20:19:58 CEST, end at Sun 2019-10-13 20:21:55 CEST. --
Oct 13 20:19:58 debian kernel: acpi PNP0A03:00: fail to add MMCONFIG information, can't access extended PCI configuration (...)
```

&#x20;`+` для объединения двух выражений по принципу логического _ИЛИ_:

```bash
root@debian:~# journalctl PRIORITY=3 + SYSLOG_FACILITY=0
-- Logs begin at Sun 2019-10-13 20:19:58 CEST, end at Sun 2019-10-13 20:24:02 CEST. --
Oct 13 20:19:58 debian kernel: Linux version 4.9.0-9-amd64 (debian-kernel@lists.debian.org) (...9
Oct 13 20:19:58 debian kernel: Command line: BOOT_IMAGE=/boot/vmlinuz-4.9.0-9-amd64 root=UUID= (...)
(...)
```

С другой стороны, вы можете указать два значения для одного и того же поля, и будут показаны все записи, соответствующие любому из этих значений:

```bash
root@debian:~# journalctl PRIORITY=1
-- Logs begin at Sun 2019-10-13 17:16:24 CEST, end at Sun 2019-10-13 17:30:14 CEST. --
-- No entries --
root@debian:~# journalctl PRIORITY=1 PRIORITY=3
-- Logs begin at Sun 2019-10-13 17:16:24 CEST, end at Sun 2019-10-13 17:32:12 CEST. --
Oct 13 17:16:27 debian connmand[459]: __connman_inet_get_pnp_nameservers: Cannot read /pro
Oct 13 17:16:27 debian connmand[459]: The name net.connman.vpn was not provided by any .se
```

{% hint style="info" %}
Поля журнала относятся к одной из следующих категорий: «Поля журнала пользователя», «Поля доверенного журнала», «Поля журнала ядра», «Поля от имени другой программы» и «Поля адреса». Дополнительную информацию по этой теме, включая полный список полей, см. на странице `man` для `systemd.journal-fields(7)`.
{% endhint %}

## **Ручные записи в Системном журнале: `systemd-cat`**

Подобно тому, как команда `logger` используется для отправки сообщений из командной строки в системный журнал (как мы видели в предыдущем уроке), команда `systemd-cat` служит аналогичной, но более универсальной цели в системном журнале. Она позволяет отправлять в журнал стандартный ввод (_stdin_), вывод (_stdout_) и ошибки (_stderr_).

Если вы не укажете параметры, программа отправит в журнал всё, что прочитает из _stdin_. Когда закончите, нажмите <kbd>Ctrl</kbd>+<kbd>C</kbd>:.

```bash
carol@debian:~$ systemd-cat
This line goes into the journal.
^C
```

Если это вывод команды, передаваемой по конвейеру, он также будет отправлен в журнал:

```
carol@debian:~$ echo "And so does this line." | systemd-cat
```

Если за этим следует команда, то вывод этой команды также будет отправлен в журнал вместе с _stderr_ (если он есть):

<pre><code><strong>ccarol@debian:~$ systemd-cat echo "And so does this line too."
</strong></code></pre>

Также можно указать уровень приоритета с помощью опции `-p`:

```
carol@debian:~$ systemd-cat -p emerg echo "This is not a real emergency."
```

Обратитесь к `systemd-cat` справочной странице, чтобы узнать о других ее параметрах.

Чтобы просмотреть последние четыре строки в журнале:

```bash
carol@debian:~$ journalctl -n 4
(...)
-- Logs begin at Sun 2019-10-20 13:43:54 CEST. --
Nov 13 23:14:39 debian cat[1997]: This line goes into the journal.
Nov 13 23:19:16 debian cat[2027]: And so does this line.
Nov 13 23:23:21 debian echo[2030]: And so does this line too.
Nov 13 23:26:48 debian echo[2034]: This is not a real emergency.
```

{% hint style="info" %}
В большинстве систем записи в журнале с уровнем приоритета _«срочно»_ будут выделяться жирным красным цветом.
{% endhint %}

## **Постоянное Хранилище журналов**

Как упоминалось ранее, у вас есть три варианта расположения журнала:

* Ведение журнала можно полностью отключить (однако перенаправление в другие средства, такие как консоль, по-прежнему возможно).
* Храните его в памяти, что делает его непостоянным, и удаляйте журналы при каждой перезагрузке системы. В этом случае каталог `/run/log/journal` будет создан и использован.
* Сделайте его постоянным, чтобы он записывал журналы на диск. В этом случае сообщения журнала будут отправляться в каталог `/var/log/journal`

Поведение по умолчанию следующее: если `/var/log/journal/` не существует, журналы будут сохраняться в нестабильном режиме в каталоге `/run/log/journal/` и, следовательно, будут потеряны при перезагрузке. Имя каталога — `/etc/machine-id` — представляет собой шестнадцатеричную 32-значную строку в нижнем регистре с завершающей новой строкой:

```bash
carol@debian:~$ ls /run/log/journal/8821e1fdf176445697223244d1dfbd73/
system.journal
```

Если вы попытаетесь прочитать его с помощью less, вы получите предупреждение, поэтому вместо этого используйте команду `journalctl`:

```bash
root@debian:~# less /run/log/journal/9a32ba45ce44423a97d6397918de1fa5/system.journal
"/run/log/journal/9a32ba45ce44423a97d6397918de1fa5/system.journal" may be a binary file.  See it anyway?
root@debian:~# journalctl
-- Logs begin at Sat 2019-10-05 21:26:38 CEST, end at Sat 2019-10-05 21:31:27 CEST. --
(...)
Oct 05 21:26:44 debian systemd-journald[1712]: Runtime journal (/run/log/journal/9a32ba45ce44423a97d6397918de1fa5) is 4.9M, max 39.5M, 34.6M free.
Oct 05 21:26:44 debian systemd[1]: Started Journal Service.
(...)
```

Если `/var/log/journal/` существует, журналы будут сохраняться там постоянно. Если этот каталог будет удалён, `systemd-journald` не создаст его заново, а вместо этого будет записывать данные в `/run/log/journal` . Как только мы снова создадим `/var/log/journal/` и перезапустим демон, постоянное ведение журнала будет восстановлено:

```bash
root@debian:~# mkdir /var/log/journal/
root@debian:~# systemctl restart systemd-journald
root@debian:~# journalctl
(...)
Oct 05 21:33:49 debian systemd-journald[1712]: Received SIGTERM from PID 1 (systemd).
Oct 05 21:33:49 debian systemd[1]: Stopped Journal Service.
Oct 05 21:33:49 debian systemd[1]: Starting Journal Service...
Oct 05 21:33:49 debian systemd-journald[1768]: Journal started
Oct 05 21:33:49 debian systemd-journald[1768]: System journal (/var/log/journal/9a32ba45ce44423a97d6397918de1fa5) is 8.0M, max 1.1G, 1.1G free.
Oct 05 21:33:49 debian systemd[1]: Started Journal Service.
Oct 05 21:33:49 debian systemd[1]: Starting Flush Journal to Persistent Storage...
(...)
```

{% hint style="info" %}
По умолчанию для каждого вошедшего в систему пользователя будут создаваться специальные файлы журнала, расположенные в `/var/log/journal/`, поэтому — наряду с файлами `system.journal` — вы также найдёте файлы типа `user-1000.journal`.
{% endhint %}

Помимо того, что мы только что упомянули, способ хранения журналов в демоне журнала можно изменить после установки, настроив его файл конфигурации: `/etc/systemd/journald.conf`. Ключевой параметр — `Storage=` и может принимать следующие значения:

`Storage=volatile`

Данные журнала будут храниться исключительно в памяти — в папке `/run/log/journal/`. Если она не существует, каталог будет создан.

`Storage=persistent`

По умолчанию данные журнала будут храниться на диске — в папке `/var/log/journal/` — с возможностью резервного копирования в память (`/run/log/journal/`) на ранних этапах загрузки и если диск не поддерживает запись. При необходимости будут созданы оба каталога.

`Storage=auto`

`auto` похоже на `persistent`, но каталог `/var/log/journal` не создаётся, если он не нужен. Это значение по умолчанию.

`Storage=none`

Все данные журнала будут удалены. Однако возможна переадресация на другие цели, такие как консоль, буфер журнала ядра или сокет системного журнала.

Например, чтобы `systemd-journald` создать `/var/log/journal/` и переключиться на постоянное хранилище, вы можете отредактировать `/etc/systemd/journald.conf` и установить `Storage=persistent`, сохранить файл и перезапустить демон с помощью `sudo systemctl restart systemd-journald`. Чтобы убедиться, что перезапуск прошёл успешно, вы всегда можете проверить состояние демона:

```bash
root@debian:~# systemctl status systemd-journald
 systemd-journald.service - Journal Service
   Loaded: loaded (/lib/systemd/system/systemd-journald.service; static; vendor preset: enabled)
   Active: active (running) since Wed 2019-10-09 10:03:40 CEST; 2s ago
     Docs: man:systemd-journald.service(8)
           man:journald.conf(5)
 Main PID: 1872 (systemd-journal)
   Status: "Processing requests..."
    Tasks: 1 (limit: 3558)
   Memory: 1.1M
   CGroup: /system.slice/systemd-journald.service
           └─1872 /lib/systemd/systemd-journald

Oct 09 10:03:40 debian10 systemd-journald[1872]: Journal started
Oct 09 10:03:40 debian10 systemd-journald[1872]: System journal (/var/log/journal/9a32ba45ce44423a97d6397918de1fa5) is 8.0M, max 1.2G, 1.2G free.
```

{% hint style="info" %}
Файлы журнала в `/var/log/journal/<machine-id>/` или `/run/log/journal/<machine-id>/` имеют суффикс `.journal` (например, `system.journal`). Однако, если они будут повреждены или демон будет остановлен некорректным образом, они будут переименованы с добавлением `~` (например, `system.journal~`), и демон начнёт записывать данные в новый, чистый файл.
{% endhint %}

## **Удаление старых данных журнала: Размер журнала**

Журналы сохраняются в _файлах журналов_, имена которых заканчиваются на `.journal` или `.journal~` и находятся в соответствующем каталоге (`/run/log/journal` или `/var/log/journal` в зависимости от настроек). Чтобы проверить, сколько дискового пространства в данный момент занимают файлы журналов (как архивные, так и активные), используйте переключатель `--disk-usage`:

```bash
root@debian:~# journalctl --disk-usage
Archived and active journals take up 24.0M in the filesystem.
```

`systemd` По умолчанию размер журналов не должен превышать 10% от размера файловой системы, в которой они хранятся. Например, в файловой системе размером 1 ГБ они не будут занимать более 100 МБ. Как только этот лимит будет достигнут, старые журналы начнут удаляться, чтобы приблизиться к этому значению.

Однако ограничением размера хранимых файлов журнала можно управлять с помощью ряда параметров конфигурации в `/etc/systemd/journald.conf`. Эти параметры делятся на две категории в зависимости от используемого типа файловой системы: постоянная (`/var/log/journal`) или в памяти (`/run/log/journal`). В первой используются параметры, начинающиеся со слова `System` и применимые только в том случае, если постоянное ведение журнала включено должным образом и система полностью загружена. Имена параметров во второй категории начинаются со слова `Runtime` и применимы в следующих случаях:

`SystemMaxUse=`, `RuntimeMaxUse=`

Они контролируют объём дискового пространства, которое может занимать журнал. По умолчанию он занимает 10% от размера файловой системы, но его можно изменить (например, `SystemMaxUse=500M`) до тех пор, пока он не превысит 4 ГБ.

`SystemKeepFree=`, `RuntimeKeepFree=`

Они контролируют объём дискового пространства, которое должно оставаться свободным для других пользователей. По умолчанию он составляет 15% от размера файловой системы, но его можно изменить (например, `SystemKeepFree=500M`), если он не превышает 4 ГБ.

Что касается приоритета `*MaxUse` и `*KeepFree`, то `systemd-journald` удовлетворит оба запроса, используя меньшее из двух значений. Кроме того, имейте в виду, что удаляются только архивные (а не активные) файлы журналов.

`SystemMaxFileSize=`, `RuntimeMaxFileSize=`

Они контролируют максимальный размер, до которого могут увеличиваться отдельные файлы журнала. Значение по умолчанию — 1/8 от `*MaxUse`. Уменьшение размера выполняется синхронно, и значения могут быть указаны в байтах или с помощью `K`, `M`, `G`, `T`, `P`, `E` для килобайтов, мегабайтов, гигабайтов, терабайтов, петабайтов и эксабайтов соответственно.

`SystemMaxFiles=`, `RuntimeMaxFiles=`

Они устанавливают максимальное количество отдельных и архивных файлов журнала для хранения (активные файлы журнала не затрагиваются). По умолчанию установлено значение 100.

Помимо удаления и ротации сообщений журнала на основе их размера, `systemd-journald` также позволяет использовать критерии на основе времени с помощью следующих двух параметров: `MaxRetentionSec=` и `MaxFileSec=`. Дополнительную информацию об этих и других параметрах см. на странице руководства `journald.conf`

{% hint style="info" %}
Всякий раз, когда вы изменяете поведение `systemd-journald` по умолчанию, раскомментируя и редактируя параметры в `/etc/systemd/journald.conf`, вам необходимо перезапустить демон, чтобы изменения вступили в силу.
{% endhint %}

## Чистка журнала ручным способом

Вы можете вручную очистить архивные файлы журналов в любое время одним из следующих трёх способов:

`--vacuum-time=`

Эта опция, основанная на времени, удалит все сообщения в файлах журнала с меткой времени, превышающей указанный период. Значения должны быть записаны с использованием любого из следующих суффиксов: `s`, `m`, `h`, `days` (или `d`), `months`, `weeks` (или `w`) и `years` (или `y`). Например, чтобы удалить все сообщения в архивных файлах журнала старше 1 месяца:

<pre class="language-bash"><code class="lang-bash"><strong>root@debian:~# journalctl --vacuum-time=1 месяц
</strong>Удален архивный журнал /var/log/journal/7203088f20394d9c8b252b64a0171e08/system@27dd08376f71405a91794e632ede97ed-0000000000000001-00059475764d46d6.journal (16,0 МБ).
Удален архивный журнал /var/log/journal/7203088f20394d9c8b252b64a0171e08/user-1000@e7020d80d3af42f0bc31592b39647e9c-000000000000008e-00059479df9677c8.journal (8,0 МБ).
</code></pre>

`--vacuum-size=`

Эта опция, основанная на размере, будет удалять архивные файлы журналов до тех пор, пока их размер не станет меньше указанного. Значения должны быть записаны с одним из следующих суффиксов: `K`, `M`, `G` или `T`. Например, чтобы удалять архивные файлы журналов до тех пор, пока их размер не станет меньше 100 мегабайт:

<pre class="language-bash"><code class="lang-bash"><strong>root@debian:~# journalctl --vacuum-size=100M
</strong> Выполнена очистка, освобождено 0 байт архивных журналов из /run/log/journal/9a32ba45ce44423a97d6397918de1fa5.
</code></pre>

`--vacuum-files=`

Эта опция позаботится о том, чтобы осталось не более указанного количества архивных файлов журнала. Значение является целым числом. Например, чтобы ограничить количество архивных файлов журнала до 10:

<pre class="language-bash"><code class="lang-bash"><strong>root@debian:~# journalctl --vacuum-files=10
</strong> Очистка завершена, освобождено 0 байт архивных журналов из /run/log/journal/9a32ba45ce44423a97d6397918de1fa5.
</code></pre>

При очистке удаляются только архивные файлы журналов. Если вы хотите избавиться от всего (включая активные файлы журналов), вам нужно использовать сигнал (`SIGUSR2`), который запрашивает немедленную ротацию файлов журналов с помощью опции `--rotate`. Другие важные сигналы можно вызвать с помощью следующих опций:

`--flush` (`SIGUSR1)`

Он запрашивает очистку файлов журнала от `/run/` до `/var/` для сохранения журнала. Для этого необходимо включить постоянное ведение журнала и подключить `/var/`

`--sync` (`SIGRTMIN+1`)

Используется для запроса записи всех незаписанных данных журнала на диск.

\


## **Извлечение данных журнала из системы восстановления**

Как системный администратор, вы можете оказаться в ситуации, когда вам потребуется доступ к файлам журналов на жёстком диске неисправного компьютера с помощью аварийной системы (загрузочного компакт-диска или USB-накопителя с дистрибутивом Linux).

`journalctl` ищет файлы журнала в `/var/log/journal/<machine-id>/`. Поскольку идентификаторы машин в системе восстановления и неисправной системе будут разными, необходимо использовать следующую опцию:

`-D </path/to/dir>`, `--directory=</path/to/dir>`

С помощью этой опции мы указываем путь к каталогу, в котором `journalctl` будет искать файлы журналов, вместо стандартных мест хранения и системных папок.

Таким образом, необходимо подключить `rootfs` (`/dev/sda1`) неисправной системы к файловой системе системы восстановления и приступить к чтению файлов журнала следующим образом:

```bash
root@debian:~# journalctl -D /media/carol/faulty.system/var/log/journal/
-- Logs begin at Sun 2019-10-20 12:30:45 CEST, end at Sun 2019-10-20 12:32:57 CEST. --
oct 20 12:30:45 suse-server kernel: Linux version 4.12.14-lp151.28.16-default (geeko@buildhost) (...)
oct 20 12:30:45 suse-server kernel: Command line: BOOT_IMAGE=/boot/vmlinuz-4.12.14-lp151.28.16-default root=UUID=7570f67f-4a08-448e-aa09-168769cb9289 splash=>
oct 20 12:30:45 suse-server kernel: x86/fpu: Supporting XSAVE feature 0x001: 'x87 floating point registers'
oct 20 12:30:45 suse-server kernel: x86/fpu: Supporting XSAVE feature 0x002: 'SSE registers'
(...)
```

Другими опциями, которые могут быть полезны в этом сценарии, являются:

`-m`, `--merge`

Он объединяет записи из всех доступных журналов в разделе `/var/log/journal`, включая удаленные.

`--file`

Он покажет записи в определенном файле, например: `journalctl --file /var/log/journal/64319965bda04dfa81d3bc4e7919814a/user-1000.journal`.

`--root`

Путь к каталогу, то есть к корневому каталогу, передаётся в качестве аргумента. `journalctl` будет искать там файлы журналов (например, `journalctl --root /faulty.system/`).

Смотрите `journalctl` справочную страницу для получения дополнительной информации.

## **Пересылка данных журнала традиционному `syslog` демону**

Данные журнала могут быть доступны традиционному демону `syslog` следующим образом:

* Пересылка сообщений в файл сокета `/run/systemd/journal/syslog` для `syslogd` чтения. Эта функция включена с помощью опции `ForwardToSyslog=yes`
* Наличие демона `syslog` с поведением `journalctl`, позволяющего считывать сообщения журнала непосредственно из файлов журнала. В этом случае подходящей опцией является `Storage`; она должна иметь значение, отличное от `none`.

{% hint style="info" %}
Аналогичным образом вы можете пересылать сообщения журнала в другие места с помощью следующих параметров: `ForwardToKMsg` (буфер журнала ядра — `kmsg`), `ForwardToConsole` (системная консоль) или `ForwardToWall` (все вошедшие в систему пользователи через `wall`). Для получения дополнительной информации обратитесь к справочной странице `journald.conf`.
{% endhint %}

## Упражнения с руководством <a href="#sec.108.2_02-ge" id="sec.108.2_02-ge"></a>

1.  Предполагая, что являетесь`root`, заполните таблицу соответствующей `journalctl` командой:

    | Цель                                                                                      | Команда |
    | ----------------------------------------------------------------------------------------- | ------- |
    | Печать записей ядра                                                                       |         |
    | Печать сообщения со второй загрузки, начиная с начала журнала                             |         |
    | Печать сообщения со второй загрузки, начиная с конца журнала                              |         |
    | Печать самых последних сообщений журнала и продолжить следить за появлением новых         |         |
    | С этого момента печать только новых сообщений и постоянно обновляйте выходные данные      |         |
    | Распечатать сообщения из предыдущей загрузки с приоритетом `warning` и в обратном порядке |         |
2.  Поведение демона журнала в отношении хранилища в основном контролируется значением параметра `Storage` в `/etc/systemd/journald.conf`. Укажите, какое поведение соответствует какому значению в следующей таблице:

    | Поведение                                                                                                                   | `Storage=auto` | `Storage=none` | `Storage=persistent` | `Storage=volatile` |
    | --------------------------------------------------------------------------------------------------------------------------- | -------------- | -------------- | -------------------- | ------------------ |
    | Данные журнала выбрасываются, но пересылка возможна.                                                                        |                |                |                      |                    |
    | После загрузки системы данные журнала будут храниться в `/var/log/journal`. Если каталог ещё не создан, он будет создан.    |                |                |                      |                    |
    | После загрузки системы данные журнала будут храниться в `/var/log/journal`. Если каталог ещё не создан, он не будет создан. |                |                |                      |                    |
    | Данные журнала будут сохранены в `/var/run/journal`, но не переживут перезагрузок.                                          |                |                |                      |                    |
3. Как вы узнали, журнал можно очистить вручную в зависимости от времени, размера и количества файлов. Выполните следующие задачи, используя `journalctl` и соответствующие параметры:
   * Проверьте, сколько места на диске занимают файлы журнала:
   * Уменьшите объём памяти, выделяемой для архивных файлов журнала, и установите его на 200 МБ:
   * Еще раз проверьте наличие свободного места на диске и объясните результаты:

### Исследовательские упражнения <a href="#sec.108.2_02-ee" id="sec.108.2_02-ee"></a>

1.  Какие параметры нужно изменить в `/etc/systemd/journald.conf` для переадресации сообщений на `/dev/tty5`? Какие значения должны быть у этих параметров?

    |   |
    | - |
    |   |
2.  Укажите правильный `journalctl` фильтр для печати следующего:

    | Цель                                                                 | Фильтр + Значение |
    | -------------------------------------------------------------------- | ----------------- |
    | Печать сообщений, принадлежащих определенному пользователю           |                   |
    | Печать сообщений с хоста с именем `debian`                           |                   |
    | Печать сообщений, принадлежащих к определенной группе                |                   |
    | Печатать сообщения, принадлежащие `root`                             |                   |
    | В зависимости от пути к исполняемому файлу выводите `sudo` сообщения |                   |
    | На основе названия команды выводите `sudo` сообщения                 |                   |
3. При фильтрации по приоритету в список также будут включены журналы с более высоким приоритетом, чем указано. Например, `journalctl -p err` выведет сообщения _ошибки_, _критической ситуации_, _предупреждения_ и _аварийной ситуации_. Однако вы можете настроить `journalctl` так, чтобы он отображал только определенный диапазон. Какую команду вы бы использовали, чтобы `journalctl` выводил только сообщения с уровнями _предупреждения_, _ошибки_ и _критической ситуации_?
4. Уровни приоритета также можно указать в числовом формате. Перепишите команду из предыдущего упражнения, используя числовое представление уровней приоритета:

## Краткие сведения <a href="#sec.108.2_02-su" id="sec.108.2_02-su"></a>

На этом уроке вы усвоили:

* Преимущества использования `systemd` в качестве системного и сервисного менеджера.
* Основы работы с `systemd` модулями и целями.
* Откуда `systemd-journald` получает данные журнала.
* Параметры, которые вы можете передать `systemctl` для управления `systemd-journald`: `start`, `status`, `restart` и `stop`.
* Где находится файл конфигурации журнала — `/etc/systemd/journald.conf` — и его основные параметры.
* Как запрашивать журнал в целом и конкретные данные с помощью фильтров.
* Как перемещаться по журналу и выполнять поиск.
* Как работать с файлами журнала: в памяти или на диске.
* Как полностью отключить ведение журнала.
* Как проверить, сколько места на диске занимает журнал, установить ограничения на размер хранимых файлов журнала и очистить архивные файлы журнала вручную (_очистка_).
* Как получить данные журнала из резервной системы.
* Как перенаправить данные журнала в традиционный демон `syslog`.

Команды, используемые в этом уроке:

`systemctl`

Управляйте `systemd` системой и менеджером обслуживания.

`journalctl`

Запросите `systemd` журнал.

`ls`

Перечислите содержимое каталога.

`less`

Просмотр содержимого файла.

`mkdir`

Создать каталог.

## Ответы на Упражнения с Руководством <a href="#sec.108.2_02-age" id="sec.108.2_02-age"></a>

1.  Предполагая, что это так `root`, заполните таблицу соответствующей `journalctl` командой:

    | Цель                                                                                      | Команда                                         |
    | ----------------------------------------------------------------------------------------- | ----------------------------------------------- |
    | Печать записей ядра                                                                       | `journalctl -k` или `journalctl --dmesg`        |
    | Печать сообщения со второй загрузки, начиная с начала журнала                             | `journalctl -b 2`                               |
    | Печать сообщения со второй загрузки, начиная с конца журнала                              | `journalctl -b -2 -r` или `journalctl -r -b -2` |
    | Распечатать самые последние сообщения журнала и продолжайте следить за появлением новых   | `journalctl -f`                                 |
    | С этого момента печатать только новые сообщения и постоянно обновляйте выходные данные    | `journalctl --since "now" -f`                   |
    | Распечатать сообщения из предыдущей загрузки с приоритетом `warning` и в обратном порядке | `journalctl -b -1 -p warning -r`                |
2.  Поведение демона журнала в отношении хранилища в основном контролируется значением параметра `Storage` в `/etc/systemd/journald.conf`. Укажите, какое поведение соответствует какому значению в следующей таблице:

    | Поведение                                                                                                                  | `Storage=auto` | `Storage=none` | `Storage=persistent` | `Storage=volatile` |
    | -------------------------------------------------------------------------------------------------------------------------- | -------------- | -------------- | -------------------- | ------------------ |
    | Данные журнала выбрасываются, но пересылка возможна                                                                        |                | x              |                      |                    |
    | После загрузки системы данные журнала будут храниться в `/var/log/journal`. Если каталог ещё не создан, он будет создан    |                |                | x                    |                    |
    | После загрузки системы данные журнала будут храниться в `/var/log/journal`. Если каталог ещё не создан, он не будет создан | x              |                |                      |                    |
    | Данные журнала будут храниться в разделе`/var/run/journal`, но не выдержат перезагрузок                                    |                |                |                      | x                  |
3. Как вы узнали, журнал можно очистить вручную в зависимости от времени, размера и количества файлов. Выполните следующие задачи, используя `journalctl` и соответствующие параметры:
   *   Проверьте, сколько места на диске занимают файлы журнала:

       ```
       journalctl --disk-usage
       ```
   *   Уменьшите объём памяти, выделяемой для архивных файлов журнала, и установите его на 200 МБ:

       ```
       journalctl --vacuum-size=200M
       ```
   *   Еще раз проверьте наличие свободного места на диске и объясните результаты:

       ```
       journalctl --disk-usage
       ```

       Взаимосвязи нет, потому что `--disk-usage` показывает пространство, занимаемое как активными, так и архивными файлами журнала, в то время как `--vacuum-size` относится только к архивным файлам.

### Ответы на Исследовательские упражнения <a href="#sec.108.2_02-aee" id="sec.108.2_02-aee"></a>

1.  Какие параметры нужно изменить в `/etc/systemd/journald.conf` для переадресации сообщений на `/dev/tty5`? Какие значения должны быть у этих параметров?

    ```
    ForwardToConsole=yes
    TTYPath=/dev/tty5
    ```
2.  Укажите правильный `journalctl` фильтр для печати следующего:

    | Цель                                                                 | Фильтр + Значение    |
    | -------------------------------------------------------------------- | -------------------- |
    | Печать сообщений, принадлежащих определенному пользователю           | `_ID=<user-id>`      |
    | Печать сообщений с хоста с именем `debian`                           | `_HOSTNAME=debian`   |
    | Печать сообщений, принадлежащих к определенной группе                | `_GID=<group-id>`    |
    | Печатать сообщения, принадлежащие `root`                             | `_UID=0`             |
    | В зависимости от пути к исполняемому файлу выводите `sudo` сообщения | `_EXE=/usr/bin/sudo` |
    | На основе названия команды выводите `sudo` сообщения                 | `_COMM=sudo`         |
3.  При фильтрации по приоритету в список также будут включены журналы с более высоким приоритетом, чем указано. Например, `journalctl -p err` выведет сообщения _ошибки_, _критической ситуации_, _предупреждения_ и _аварийной ситуации_. Однако вы можете настроить `journalctl` так, чтобы он отображал только определенный диапазон. Какую команду вы бы использовали, чтобы `journalctl` выводил только сообщения с уровнями _предупреждения_, _ошибки_ и _критической ситуации_?

    ```
    journalctl -p warning..crit
    ```
4.  Уровни приоритета также можно указать в числовом формате. Перепишите команду из предыдущего упражнения, используя числовое представление уровней приоритета:

    ```
    journalctl -p 4..2
    ```
