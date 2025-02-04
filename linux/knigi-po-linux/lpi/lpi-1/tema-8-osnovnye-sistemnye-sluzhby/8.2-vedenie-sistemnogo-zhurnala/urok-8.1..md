# Урок 8.1.

## Введение <a href="#sec.108.2_01-in" id="sec.108.2_01-in"></a>

Журналы могут стать лучшим другом системного администратора. Журналы — это файлы (обычно текстовые), в которых все системные и сетевые события регистрируются в хронологическом порядке с момента загрузки вашей системы. Таким образом, в журналах можно найти практически любую информацию о системе: неудачные попытки аутентификации, ошибки программ и служб, хосты, заблокированные брандмауэром, и т. д. Как вы можете себе представить, журналы значительно облегчают жизнь системных администраторов, когда дело доходит до устранения неполадок, проверки ресурсов, выявления аномального поведения программ и так далее.

В этом уроке мы рассмотрим одну из наиболее распространённых функций ведения журналов, которая в настоящее время доступна в дистрибутивах GNU/Linux: `rsyslog`. Мы изучим различные типы существующих журналов, места их хранения, информацию, которую они содержат, и способы получения и фильтрации этой информации. Мы также обсудим, как журналы могут храниться на централизованных серверах в IP-сетях, ротацию журналов и кольцевой буфер ядра.

## Ведение журнала системы <a href="#system_logging" id="system_logging"></a>

В тот момент, когда ядро и различные процессы в вашей системе начинают работать и взаимодействовать друг с другом, генерируется много информации в виде сообщений, которые по большей части отправляются в журналы.

Без ведения журналов поиск события, произошедшего на сервере, стал бы головной болью для системных администраторов, поэтому важно иметь стандартизированный и централизованный способ отслеживания любых системных событий. Журналы играют важную роль в устранении неполадок и обеспечении безопасности, а также являются надёжными источниками данных для понимания системной статистики и прогнозирования тенденций.

Если не брать в расчёт `systemd-journald` (который мы обсудим в следующем уроке), ведение журналов традиционно осуществлялось тремя основными специализированными службами: `syslog`, `syslog-ng` (syslog нового поколения) и `rsyslog` («сверхбыстрая система обработки журналов»). `rsyslog` привнёс важные улучшения (например, поддержку RELP) и стал самым популярным выбором в наши дни. Каждая из этих служб собирает сообщения от других служб и программ и сохраняет их в файлах журналов, обычно в `/var/log`. Однако некоторые службы сами заботятся о своих журналах (например, веб-сервер Apache HTTPD или система печати CUPS). Аналогичным образом ядро Linux использует кольцевой буфер в памяти для хранения сообщений журнала.

{% hint style="info" %}
`RELP` Расшифровывается как _«Протокол надёжного ведения журнала событий Reliable Event Logging Protocol»_ и расширяет функциональность протокола системного журнала, обеспечивая надёжную доставку сообщений.
{% endhint %}

Поскольку `rsyslog` стал _де-факто_ стандартным средством ведения журнала во всех основных дистрибутивах, мы сосредоточимся на нём в этом уроке. `rsyslog` использует клиент-серверную модель. Клиент и сервер могут находиться на одном хосте или на разных компьютерах. Сообщения отправляются и принимаются в определённом формате и могут храниться на централизованных `rsyslog` серверах в IP-сетях. Демон rsyslog — `rsyslogd` — работает вместе с `klogd` (который управляет сообщениями ядра). В следующих разделах мы обсудим `rsyslog` и его инфраструктуру ведения журнала.

{% hint style="info" %}
Демон — это служба, которая работает в фоновом режиме. Обратите внимание на последний `d` в названиях демонов: `klogd` или `rsyslogd`.
{% endhint %}

## **Типы журналов**

Поскольку журналы являются _переменными_ данными, они обычно находятся в `/var/log`. Грубо говоря, их можно разделить на _системные журналы_ и _журналы служб или программ_.

Давайте посмотрим некоторые системные журналы и информацию, которую они хранят:

`/var/log/auth.log`

Действия, связанные с процессами аутентификации: зарегистрированные пользователи, `sudo` информация, задания cron, неудачные попытки входа в систему и т. д.

`/var/log/syslog`

Централизованный файл практически для всех журналов, собираемых `rsyslogd`. Поскольку он содержит много информации, журналы распределяются по другим файлам в соответствии с конфигурацией, указанной в `/etc/rsyslog.conf`.

`/var/log/debug`

Отладочная информация из программ.

`/var/log/kern.log`

Сообщения ядра.

`/var/log/messages`

Информативные сообщения, которые относятся не к ядру, а к другим службам. Это также место назначения журнала удалённого клиента по умолчанию при реализации централизованного сервера журналов.

`/var/log/daemon.log`

Информация, относящаяся к демонам или службам, работающим в фоновом режиме.

`/var/log/mail.log`

Информация, относящаяся к почтовому серверу, например postfix.

`/var/log/Xorg.0.log`

Информация, относящаяся к видеокарте.

`/var/run/utmp` и `/var/log/wtmp`

Успешный вход в систему.

`/var/log/btmp`

Неудачные попытки входа в систему, например, атака методом перебора через ssh.

`/var/log/faillog`

Неудачные попытки аутентификации.

`/var/log/lastlog`

Дата и время последнего входа пользователя в систему.

Теперь давайте посмотрим на несколько примеров служебных журналов:

`/var/log/cups/`

Каталог для журналов _Common Unix Printing System_. Обычно он включает следующие файлы журналов по умолчанию: `error_log`, `page_log` и `access_log`.

`/var/log/apache2/` или `/var/log/httpd`

Каталог для журналов _веб-сервера Apache_. Обычно он включает следующие файлы журналов по умолчанию: `access.log`, `error_log`, и `other_vhosts_access.log`.

`/var/log/mysql`

Каталог журналов _системы управления реляционными базами данных MySQL_. Обычно он включает следующие файлы журналов по умолчанию: `error_log`, `mysql.log` и `mysql-slow.log`.

`/var/log/samba/`

Каталог для журналов протокола _Session Message Block_ (SMB). Обычно он включает следующие файлы журналов по умолчанию: `log.`, `log.nmbd` и `log.smbd`.

{% hint style="info" %}
Точное название и содержимое файлов журналов могут различаться в разных дистрибутивах Linux. Существуют также журналы, характерные для конкретных дистрибутивов, например `/var/log/dpkg.log` (содержащие информацию, связанную с пакетами `dpkg` ) в Debian GNU/Linux и его производных.
{% endhint %}

## **Чтение журналов**

Чтобы прочитать файлы журналов, сначала убедитесь, что вы являетесь пользователем root или имеете права на чтение файла. Вы можете использовать различные утилиты, например:

`less` или `more`

Пейджеры, позволяющие просматривать и прокручивать по одной странице за раз:

```bash
root@debian:~# less /var/log/auth.log
Sep 12 18:47:56 debian sshd[441]: Received SIGHUP; restarting.
Sep 12 18:47:56 debian sshd[441]: Server listening on 0.0.0.0 port 22.
Sep 12 18:47:56 debian sshd[441]: Server listening on :: port 22.
Sep 12 18:47:56 debian sshd[441]: Received SIGHUP; restarting.
Sep 12 18:47:56 debian sshd[441]: Server listening on 0.0.0.0 port 22.
Sep 12 18:47:56 debian sshd[441]: Server listening on :: port 22.
Sep 12 18:49:46 debian sshd[905]: Accepted password for carol from 192.168.1.65 port 44296 ssh2
Sep 12 18:49:46 debian sshd[905]: pam_unix(sshd:session): session opened for user carol by (uid=0)
Sep 12 18:49:46 debian systemd-logind[331]: New session 2 of user carol.
Sep 12 18:49:46 debian systemd: pam_unix(systemd-user:session): session opened for user carol by (uid=0)
(...)
```

`zless` или `zmore`

То же, что и `less` и `more`, но используется для журналов, сжатых с помощью `gzip` (обычная функция `logrotate`):

```bash
root@debian:~# zless /var/log/auth.log.3.gz
Aug 19 20:05:57 debian sudo:    carol : TTY=pts/0 ; PWD=/home/carol ; USER=root ; COMMAND=/sbin/shutdown -h now
Aug 19 20:05:57 debian sudo: pam_unix(sudo:session): session opened for user root by carol(uid=0)
Aug 19 20:05:57 debian lightdm: pam_unix(lightdm-greeter:session): session closed for user lightdm
Aug 19 23:50:49 debian systemd-logind[333]: Watching system buttons on /dev/input/event2 (Power Button)
Aug 19 23:50:49 debian systemd-logind[333]: Watching system buttons on /dev/input/event3 (Sleep Button)
Aug 19 23:50:49 debian systemd-logind[333]: Watching system buttons on /dev/input/event4 (Video Bus)
Aug 19 23:50:49 debian systemd-logind[333]: New seat seat0.
Aug 19 23:50:49 debian sshd[409]: Server listening on 0.0.0.0 port 22.
(...)
```

`tail`

Просмотр последних строк в файле (по умолчанию отображается 10 строк). Возможности команды tail в значительной степени обусловлены параметром `-f`, который динамически отображает новые строки по мере их добавления:

`head`

Просмотр первых строк в файле (по умолчанию 10 строк):

```bash
root@suse-server:~# head -5 /var/log/mail
2019-06-29T11:47:59.219806+02:00 suse-server postfix/postfix-script[1732]: the Postfix mail system is not running
2019-06-29T11:48:01.355361+02:00 suse-server postfix/postfix-script[1925]: starting the Postfix mail system
2019-06-29T11:48:01.391128+02:00 suse-server postfix/master[1930]: daemon started -- version 3.3.1, configuration /etc/postfix
2019-06-29T11:55:39.247462+02:00 suse-server postfix/postfix-script[3364]: stopping the Postfix mail system
2019-06-29T11:55:39.249375+02:00 suse-server postfix/master[1930]: terminating on signal 15
```

`grep`

Утилита фильтрации, которая позволяет вам искать определенные строки:

```bash
root@debian:~# grep "dhclient" /var/log/syslog
Sep 13 11:58:48 debian dhclient[448]: DHCPREQUEST of 192.168.1.4 on enp0s3 to 192.168.1.1 port 67
Sep 13 11:58:49 debian dhclient[448]: DHCPACK of 192.168.1.4 from 192.168.1.1
Sep 13 11:58:49 debian dhclient[448]: bound to 192.168.1.4 -- renewal in 1368 seconds.
(...)
```

Как вы, возможно, заметили, выходные данные печатаются в следующем формате:

* Дата и время
* Имя хоста, с которого было отправлено сообщение
* Название программы/службы, которая сгенерировала сообщение
* PID программы, которая сгенерировала сообщение
* Описание выполненного действия

Есть несколько примеров, в которых логи представляют собой не текстовые, а двоичные файлы, и, следовательно, для их анализа необходимо использовать специальные команды:

`/var/log/wtmp`

Использование `who` (или `w`):

```bash
root@debian:~# who
root pts/0 2020-09-14 13:05 (192.168.1.75)
root pts/1 2020-09-14 13:43 (192.168.1.75)
```

`/var/log/btmp`

Использование `utmpdump` или `last -f`:

```bash
root@debian:~# utmpdump /var/log/btmp
Utmp dump of /var/log/btmp
[6] [01287] [    ] [dave     ] [ssh:notty   ] [192.168.1.75        ] [192.168.1.75   ] [2019-09-07T19:33:32,000000+0000]
```

`/var/log/faillog`

Использование `faillog`:

```
root@debian:~# faillog -a | less
Login       Failures Maximum Latest                   On

root            0        0   01/01/70 01:00:00 +0100
daemon          0        0   01/01/70 01:00:00 +0100
bin             0        0   01/01/70 01:00:00 +0100
sys             0        0   01/01/70 01:00:00 +0100
sync            0        0   01/01/70 01:00:00 +0100
games           0        0   01/01/70 01:00:00 +0100
man             0        0   01/01/70 01:00:00 +0100
lp              0        0   01/01/70 01:00:00 +0100
mail            0        0   01/01/70 01:00:00 +0100
(...)
```

`/var/log/lastlog`

Использование `lastlog`:

```bash
root@debian:~# lastlog | less
Username         Port     From             Latest
root                                       Never logged in
daemon                                     Never logged in
bin                                        Never logged in
sys                                        Never logged in
(...)
sync                                       Never logged in
avahi                                      Never logged in
colord                                     Never logged in
saned                                      Never logged in
hplip                                      Never logged in
carol            pts/1    192.168.1.75     Sat Sep 14 13:43:06 +0200 2019
dave             pts/3    192.168.1.75     Mon Sep  2 14:22:08 +0200 2019
```

{% hint style="info" %}
Существуют также графические инструменты для чтения файлов журналов, например: `gnome-logs` и `KSystemLog`.
{% endhint %}

## **Как сообщения превращаются в журналы**

Следующий процесс иллюстрирует, как сообщение записывается в файл журнала:

1. Приложения, службы и ядро записывают сообщения в специальные файлы (сокеты и буферы памяти), например `/dev/log` или `/dev/kmsg`.
2. `rsyslogd` получает информацию из сокетов или буферов памяти.
3. В зависимости от правил, указанных в `/etc/rsyslog.conf` и/или файлах в `/etc/ryslog.d/`, `rsyslogd` перемещает информацию в соответствующий файл журнала (обычно находится в `/var/log`).

{% hint style="info" %}
Сокет — это специальный файл, используемый для передачи информации между различными процессами. Чтобы вывести список всех сокетов в вашей системе, вы можете использовать команду `systemctl list-sockets --all`.
{% endhint %}

## **Возможности, приоритеты и действия**

`rsyslog` Файл конфигурации — `/etc/rsylog.conf` (в некоторых дистрибутивах вы также можете найти файлы конфигурации в `/etc/rsyslog.d/`). Обычно он разделён на три части: `MODULES`, `GLOBAL DIRECTIVES` и `RULES`. Давайте рассмотрим их, изучив файл `rsyslog.conf` на нашем хосте Debian GNU/Linux 10 (buster) — для этого можно использовать `sudo less /etc/rsyslog.conf`

`MODULES` включает поддержку модулей для ведения журнала, отправки сообщений и приёма журналов UDP/TCP

```bash
#################
#### MODULES ####
#################

module(load="imuxsock") # provides support for local system logging
module(load="imklog")   # provides kernel logging support
#module(load="immark")  # provides --MARK-- message capability

# provides UDP syslog reception
#module(load="imudp")
#input(type="imudp" port="514")

# provides TCP syslog reception
#module(load="imtcp")
#input(type="imtcp" port="514")
```

`GLOBAL DIRECTIVES` позволяет нам настроить ряд параметров, таких как журналы и права доступа к каталогам журналов:

```bash
###########################
#### GLOBAL DIRECTIVES ####
###########################

#
# Use traditional timestamp format.
# To enable high precision timestamps, comment out the following line.
#
$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat

#
# Set the default permissions for all log files.
#
$FileOwner root
$FileGroup adm
$FileCreateMode 0640
$DirCreateMode 0755
$Umask 0022

#
# Where to place spool and state files
#
$WorkDirectory /var/spool/rsyslog

#
# Include all config files in /etc/rsyslog.d/
#
$IncludeConfig /etc/rsyslog.d/*.conf
```

`RULES` именно здесь на помощь приходят _средства_, _приоритеты_ и _действия_. Настройки в этом разделе указывают демону ведения журнала фильтровать сообщения в соответствии с определенными правилами и регистрировать их или отправлять куда требуется. Чтобы понять эти правила, мы должны сначала объяснить концепции `rsyslog` средств и приоритетов. Каждому сообщению журнала присваивается номер _объекта_ и ключевое слово, которые связаны с внутренней подсистемой Linux, создающей это сообщение:

| Число     | Ключевое слово          | Описание                                              |
| --------- | ----------------------- | ----------------------------------------------------- |
| `0`       | `kern`                  | Сообщения ядра Linux                                  |
| `1`       | `user`                  | Сообщения на уровне пользователя                      |
| `2`       | `mail`                  | Почтовая система                                      |
| `3`       | `daemon`                | Системные демоны                                      |
| `4`       | `auth`, `authpriv`      | Сообщения о безопасности/Авторизации                  |
| `5`       | `syslog`                | системные сообщения                                   |
| `6`       | `lpr`                   | Подсистема линейного принтера                         |
| `7`       | `news`                  | Подсистема сетевых новостей                           |
| `8`       | `uucp`                  | Подсистема UUCP (протокол копирования из Unix в Unix) |
| `9`       | `cron`                  | Демон задач                                           |
| `10`      | `auth`, `authpriv`      | Сообщения о безопасности/Авторизации                  |
| `11`      | `ftp`                   | Демон FTP (Протокола передачи файлов)                 |
| `12`      | `ntp`                   | Демон NTP (Network Time Protocol)                     |
| `13`      | `security`              | Журнал Аудита                                         |
| `14`      | `console`               | Log alert                                             |
| `15`      | `cron`                  | Часовой демон                                         |
| `16 - 23` | `local0` через `local7` | Местное использование 0 - 7                           |

Кроме того, каждому сообщению присваивается определенный уровень _приоритета_:

| Код | Серьезность           | Ключевое слово    | Описание                                |
| --- | --------------------- | ----------------- | --------------------------------------- |
| `0` | Чрезвычайная ситуация | `emerg`, `panic`  | Система непригодна для использования    |
| `1` | Тревога               | `alert`           | Действия должны быть приняты немедленно |
| `2` | Критический           | `crit`            | Критические условия                     |
| `3` | Ошибка                | `err`, `error`    | Условия ошибки                          |
| `4` | Предупреждение        | `warn`, `warning` | Предупреждающие условия                 |
| `5` | УВЕДОМЛЕНИЕ           | `notice`          | Нормальное, но значимое состояние       |
| `6` | Информационный        | `info`            | Информационные сообщения                |
| `7` | Отлаживать            | `debug`           | Сообщения уровня отладки                |

Вот отрывок из `rsyslog.conf` нашей системы Debian GNU/Linux 10 (buster), включающий несколько примеров правил:

```bash
###############
#### RULES ####
###############

# First some standard log files.  Log by facility.
#
auth,authpriv.*                 /var/log/auth.log
*.*;auth,authpriv.none          -/var/log/syslog
#cron.*                         /var/log/cron.log
daemon.*                        -/var/log/daemon.log
kern.*                          -/var/log/kern.log
lpr.*                           -/var/log/lpr.log
mail.*                          -/var/log/mail.log
user.*                          -/var/log/user.log

#
# Logging for the mail system.  Split it up so that
# it is easy to write scripts to parse these files.
#
mail.info                       -/var/log/mail.info
mail.warn                       -/var/log/mail.warn
mail.err                        /var/log/mail.err

#
# Some "catch-all" log files.
#
*.=debug;\
        auth,authpriv.none;\
	news.none;mail.none     -/var/log/debug
*.=info;*.=notice;*.=warn;\
	auth,authpriv.none;\
	cron,daemon.none;\
	mail,news.none          -/var/log/messages
```

Формат правила выглядит следующим образом: `<facility>.<priority>` `<action>`

Селектор `<facility>.<priority>` фильтрует сообщения по соответствию. Уровни приоритета являются иерархически инклюзивными, что означает, что rsyslog будет соответствовать сообщениям с указанным приоритетом и выше. `<action>` показывает, какое действие следует предпринять (куда отправить сообщение журнала). Вот несколько примеров для наглядности:

```bash
auth,authpriv.* /var/log/auth.log
```

Независимо от их приоритета (`*`), все сообщения с объектов `auth` или `authpriv` будут отправляться на `/var/log/auth.log`.

```bash
*.*;auth,authpriv.none -/var/log/syslog
```

Все сообщения — независимо от их приоритета (`*`) — со всех устройств (`*`) — за исключением сообщений с `auth` или `authpriv` (отсюда суффикс `.none`), — будут записываться в `/var/log/syslog` (знак минус (`-`) перед путём предотвращает избыточную запись на диск). Обратите внимание на точку с запятой (`;`) для разделения селектора и запятую (`,`) для объединения двух устройств в одном правиле (`auth,authpriv`).

```bash
mail.err /var/log/mail.err
```

Сообщения с объекта `mail` с уровнем приоритета `error` или выше (`critical`, `alert` или `emergency`) будут отправляться на `/var/log/mail.err`.

```bash
*.=debug;\
        auth,authpriv.none;\
	news.none;mail.none     -/var/log/debug
```

Сообщения со всех объектов с приоритетом `debug` и без других (`=`) будут записываться в `/var/log/debug` — за исключением сообщений, поступающих с объектов `auth`, `authpriv`, `news` и `mail` (обратите внимание на синтаксис: `;\`).

## **Ручные записи в Системный журнал: `logger`**

Команда `logger` пригодится для написания сценариев оболочки или в целях тестирования. `logger` будет добавлять любое полученное сообщение в `/var/log/syslog` (или в `/var/log/messages` при ведении журнала на удалённом центральном сервере журналов, как вы увидите далее в этом уроке):

```bash
carol@debian:~$ logger this comment goes into "/var/log/syslog"
```

Чтобы вывести последнюю строку в `/var/log/syslog`, используйте команду `tail` с опцией `-1`:

```bash
root@debian:~# tail -1 /var/log/syslog
Sep 17 17:55:33 debian carol: this comment goes into /var/log/syslog
```

## **`rsyslog` в качестве Центрального сервера журналов**

Чтобы объяснить эту тему, мы добавим в нашу систему новый хост. Схема выглядит следующим образом:

Чтобы объяснить эту тему, мы добавим в нашу систему новый хост. Схема выглядит следующим образом:

| Роль                               | Имя хоста     | Операционная система         | IP - адрес  |
| ---------------------------------- | ------------- | ---------------------------- | ----------- |
| Центральный Сервер Ведения журнала | `suse-server` | openSUSE Leap 15.1           | 192.168.1.6 |
| Клиент                             | `debian`      | Debian GNU/Linux 10 (бастер) | 192.168.1.4 |

Давайте начнём с настройки сервера. Прежде всего, мы убедимся, что `rsyslog` работает:

```bash
root@suse-server:~# systemctl status rsyslog
 rsyslog.service - System Logging Service
   Loaded: loaded (/usr/lib/systemd/system/rsyslog.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2019-09-17 18:45:58 CEST; 7min ago
     Docs: man:rsyslogd(8)
           http://www.rsyslog.com/doc/
 Main PID: 832 (rsyslogd)
    Tasks: 5 (limit: 4915)
   CGroup: /system.slice/rsyslog.service
           └─832 /usr/sbin/rsyslogd -n -iNONE
```

В openSUSE есть специальный файл конфигурации для удалённого ведения журнала: `/etc/rsyslog.d/remote.conf`. Давайте включим получение сообщений от клиентов (удалённых хостов) через TCP. Мы должны раскомментировать строки, которые загружают модуль и запускают TCP-сервер на порту 514:

```bash
# ######### Receiving Messages from Remote Hosts ##########
# TCP Syslog Server:
# provides TCP syslog reception and GSS-API (if compiled to support it)
$ModLoad imtcp.so  # load module
##$UDPServerAddress 10.10.0.1  # force to listen on this IP only
$InputTCPServerRun 514  # Starts a TCP server on selected port

# UDP Syslog Server:
#$ModLoad imudp.so  # provides UDP syslog reception
##$UDPServerAddress 10.10.0.1  # force to listen on this IP only
#$UDPServerRun 514  # start a UDP syslog server at standard port 514
```

После этого мы должны перезапустить службу rsyslog и проверить, что сервер прослушивает порт 514:

```bash
root@suse-server:~# systemctl restart rsyslog
root@suse-server:~# netstat -nltp | grep 514
[sudo] password for root:
tcp        0      0 0.0.0.0:514             0.0.0.0:*               LISTEN      2263/rsyslogd
tcp6       0      0 :::514                  :::*                    LISTEN      2263/rsyslogd
```

Далее мы должны открыть порты в брандмауэре и перезагрузить конфигурацию:

```bash
root@suse-server:~# firewall-cmd --permanent --add-port 514/tcp
success
root@suse-server:~# firewall-cmd --reload
success
```

{% hint style="info" %}
С выходом openSUSE Leap 15.0 `firewalld` полностью заменил классический `SuSEFirewall2`
{% endhint %}

## **Шаблоны и условия фильтрации**

По умолчанию журналы клиента будут записываться в файл `/var/log/messages` сервера — вместе с журналами самого сервера. Однако мы создадим _шаблон_ и _условие фильтрации_, чтобы журналы нашего клиента хранились в отдельных каталогах. Для этого мы добавим следующее в `/etc/rsyslog.conf` (или `/etc/rsyslog.d/remote.conf`):

```
$template RemoteLogs,"/var/log/remotehosts/%HOSTNAME%/%$NOW%.%syslogseverity-text%.log"
if $FROMHOST-IP=='192.168.1.4' then ?RemoteLogs
& stop
```

Шаблон

Шаблон соответствует первой строке и позволяет задать формат для имён журналов с помощью динамической генерации имён файлов. Шаблон состоит из:

* Шаблонная директива (`$template`)
* Имя шаблона (`RemoteLogs`)
* Текст шаблона (`"/var/log/remotehosts/%HOSTNAME%/%$NOW%.%syslogseverity-text%.log"`)
* Параметры (необязательно)

Наш шаблон называется `RemoteLogs`, и его текст состоит из пути в `/var/log`. Все журналы нашего удаленного хоста будут отправлены в `remotehosts` каталог, где будет создан подкаталог на основе имени хоста компьютера (`%HOSTNAME%`). Каждое имя файла в этом каталоге будет состоять из даты (`%$NOW%`), серьезности (иначе говоря, приоритета) сообщения в текстовом формате (`%syslogseverity-text%`) и `.log` суффикса. Слова, заключённые в процентные знаки, являются _свойствами_ и позволяют получить доступ к содержимому сообщения журнала (дата, приоритет и т. д.). `syslog` Сообщение имеет ряд чётко определённых свойств, которые можно использовать в шаблонах. Доступ к этим свойствам и их изменение осуществляются с помощью так называемого _замещающего свойства_, который подразумевает их запись в процентных знаках.

Состояние фильтра

Остальные две строки соответствуют условию фильтра и связанному с ним действию:

* Фильтр на основе выражений (`if $FROMHOST-IP=='192.168.1.4'`)
* Действие (`then ?RemoteLogs`, `& stop`)

Первая строка проверяет IP-адрес удалённого хоста, отправляющего журнал, и, если он совпадает с IP-адресом нашего клиента Debian, применяет шаблон `RemoteLogs` . Последняя строка (`& stop`) гарантирует, что сообщения не будут отправляться одновременно в `/var/log/messages` (а только в файлы в каталоге `/var/log/remotehosts`).

{% hint style="info" %}
Чтобы узнать больше о шаблонах, свойствах и правилах, вы можете обратиться к странице руководства для `rsyslog.conf`.
{% endhint %}

После обновления конфигурации мы снова перезапустим `rsyslog` и убедимся, что  в `/var/log` пока нет каталога `remotehosts`:

```bash
root@suse-server:~# systemctl restart rsyslog
root@suse-server:~# ls /var/log/
acpid             chrony     localmessages   pbl.log          Xorg.0.log
alternatives.log  cups       mail            pk_backend_zypp  Xorg.0.log.old
apparmor          firebird   mail.err        samba            YaST2
audit             firewall   mail.info       snapper.log      zypp
boot.log          firewalld  mail.warn       tallylog         zypper.log
boot.msg          krb5       messages        tuned
boot.omsg         lastlog    mysql           warn
btmp              lightdm    NetworkManager  wtmp
```

Теперь сервер настроен. Далее мы настроим клиента.

Опять же, мы должны убедиться, что `rsyslog` установлен и запущен:

```bash
root@debian:~# sudo systemctl status rsyslog
 rsyslog.service - System Logging Service
   Loaded: loaded (/lib/systemd/system/rsyslog.service; enabled; vendor preset:
   Active: active (running) since Thu 2019-09-17 18:47:54 CEST; 7min ago
     Docs: man:rsyslogd(8)
           http://www.rsyslog.com/doc/
 Main PID: 351 (rsyslogd)
    Tasks: 4 (limit: 4915)
   CGroup: /system.slice/rsyslog.service
           └─351 /usr/sbin/rsyslogd -n
```

В нашей тестовой среде мы реализовали разрешение имён на стороне клиента, добавив строку `192.168.1.6 suse-server` в `/etc/hosts`. Таким образом, мы можем обращаться к серверу по имени (`suse-server`) или IP-адресу (`192.168.1.6`).

Наш клиент Debian не поставляется с файлом `remote.conf` в `/etc/rsyslog.d/`, поэтому мы применим наши настройки в `/etc/rsyslog.conf`. Мы напишем следующую строку в конце файла:

```bash
*.* @@suse-server:514
```

Наконец, мы перезапускаем`rsyslog`.

```bash
root@debian:~# systemctl restart rsyslog
```

Теперь давайте вернёмся к нашей машине `suse-server` и проверим наличие `remotehosts` в `/var/log`:

```bash
root@suse-server:~# ls /var/log/remotehosts/debian/
2019-09-17.info.log  2019-09-17.notice.log
```

У нас уже есть два журнала в `/var/log/remotehosts` как описано в нашем шаблоне. Чтобы завершить этот раздел, мы запускаем `tail -f` `2019-09-17.notice.log` на `suse-server` и отправляем журнал _вручную_ с нашего клиента Debian и подтверждаем, что сообщения добавляются в файл журнала, как и ожидалось (опция `-t` предоставляет тег для нашего сообщения):

```bash
root@suse-server:~# tail -f /var/log/remotehosts/debian/2019-09-17.notice.log
2019-09-17T20:57:42+02:00 debian dbus[323]: [system] Successfully activated service 'org.freedesktop.nm_dispatcher'
2019-09-17T21:01:41+02:00 debian anacron[1766]: Anacron 2.3 started on 2019-09-17
2019-09-17T21:01:41+02:00 debian anacron[1766]: Normal exit (0 jobs run)
```

```bash
carol@debian:~$ logger -t DEBIAN-CLIENT Hi from 192.168.1.4
```

```bash
root@suse-server:~# tail -f /var/log/remotehosts/debian/2019-09-17.notice.log
2019-09-17T20:57:42+02:00 debian dbus[323]: [system] Successfully activated service 'org.freedesktop.nm_dispatcher'
2019-09-17T21:01:41+02:00 debian anacron[1766]: Anacron 2.3 started on 2019-09-17
2019-09-17T21:01:41+02:00 debian anacron[1766]: Normal exit (0 jobs run)
2019-09-17T21:04:21+02:00 debian DEBIAN-CLIENT: Hi from 192.168.1.4
```

## **Механизм Log Rotation**

Журналы регулярно меняются, что служит двум основным целям:

* Не позволяет старым файлам журналов занимать больше места на диске, чем необходимо.
* Сохраняет журналы в удобном для просмотра формате.

Утилита, отвечающая за ротацию (или циклическую обработку) журналов, называется `logrotate` и выполняет такие действия, как перемещение файлов журналов под новое имя, их архивирование и/или сжатие, иногда отправка их по электронной почте системному администратору и, в конечном итоге, их удаление по мере устаревания. Существуют различные соглашения об именовании этих перемещаемых файлов журналов (например, добавление суффикса с датой к имени файла); однако обычно просто добавляется суффикс с целым числом:

```bash
root@debian:~# ls /var/log/messages*
/var/log/messages /var/log/messages.1 /var/log/messages.2.gz /var/log/messages.3.gz /var/log/messages.4.gz
```

Давайте теперь объясним, что произойдет при следующем вращении журнала:

1. `messages.4.gz`будет удалён и потерян.
2. Содержимое `messages.3.gz` будет перемещено в `messages.4.gz`.
3. Содержимое `messages.2.gz` будет перемещено в `messages.3.gz`.
4. Содержимое `messages.1` будет перемещено в `messages.2.gz`.
5. Содержимое `messages` будет перемещено в `messages.1` и `messages` будет пустым и готовым для регистрации новых записей в журнале.

Обратите внимание, что согласно `logrotate` директивам, которые вы вскоре увидите, три старых файла журнала сжимаются, а два самых последних - нет. Кроме того, мы сохраним журналы за последние 4-5 недель. Чтобы прочитать сообщения недельной давности, мы ищем в `messages.1` (и так далее).

`logrotate` запускается как автоматизированный процесс или задание cron ежедневно с помощью скрипта `/etc/cron.daily/logrotate` и считывает файл конфигурации `/etc/logrotate.conf`. Этот файл содержит некоторые глобальные параметры и хорошо прокомментирован: каждый параметр сопровождается кратким объяснением его назначения:

```bash
carol@debian:~$ sudo less /etc/logrotate.conf
# see "man logrotate" for details
# rotate log files weekly
weekly

# keep 4 weeks worth of backlogs
rotate 4

# create new (empty) log files after rotating old ones
create

# uncomment this if you want your log files compressed
#compress

# packages drop log rotation information into this directory
include /etc/logrotate.d

(...)
```

Как видите, файлы конфигурации `/etc/logrotate.d` для конкретных пакетов также включены. Эти файлы содержат — по большей части — локальные определения и указывают на необходимость ротации файлов журналов (помните, что локальные определения имеют приоритет над глобальными, а более поздние определения переопределяют более ранние). Ниже приведён фрагмент определения в `/etc/logrotate.d/rsyslog`

```bash
/var/log/messages
{
        rotate 4
        weekly
        missingok
        notifempty
        compress
        delaycompress
        sharedscripts
        postrotate
                invoke-rc.d rsyslog rotate > /dev/null
        endscript
}
```

Как видите, каждая директива отделяется от своего значения пробелами и/или необязательным знаком равенства (`=`). Однако строки между `postrotate` и `endscript` должны располагаться на отдельных строках. Объяснение следующее:

`rotate 4`

Храните журналы на 4 недели.

`weekly`

Еженедельно чередуйте файлы журналов.

`missingok`

Если файл журнала отсутствует, не выводите сообщение об ошибке, просто перейдите к следующему.

`notifempty`

Не поворачивайте журнал, если он пустой.

`compress`

Сжимайте файлы журналов с помощью `gzip` (по умолчанию).

`delaycompress`

Отложить сжатие предыдущего файла журнала до следующего цикла ротации (эффективно только в сочетании со сжатием). Это полезно, когда программе нельзя указать закрыть файл журнала и она может продолжать писать в предыдущий файл журнала в течение некоторого времени.

`sharedscripts`

Связано со сценариями _превращения в архив_ и _последующего преобразования в архив_. Чтобы предотвратить многократное выполнение сценария, запускайте сценарии только один раз, независимо от того, сколько файлов журнала соответствуют заданному шаблону (например, `/var/log/mail/*`). Сценарии не будут запускаться, если ни один из журналов, соответствующих шаблону, не нуждается в преобразовании в архив. Кроме того, если сценарии завершатся с ошибкой, остальные действия не будут выполнены ни для одного журнала.

`postrotate`

Укажите начало сценария _postrotate_.

`invoke-rc.d rsyslog rotate > /dev/null`

Используйте `/bin/sh` для запуска `invoke-rc.d rsyslog rotate > /dev/null` после поворота журналов.

`endscript`

Укажите конец скрипта _postrotate_.

{% hint style="info" %}
Полный список директив и пояснений см. на странице руководства для `logrotate.conf`.
{% endhint %}

## **Кольцевой буфер ядра**

Поскольку ядро генерирует несколько сообщений до того, как `rsyslogd` становится доступным при загрузке, возникает необходимость в механизме регистрации этих сообщений. Именно здесь в игру вступает _кольцевой буфер ядра_. Это структура данных фиксированного размера, и поэтому по мере поступления новых сообщений самые старые будут удаляться.

Команда `dmesg` выводит буфер кольца ядра. Из-за размера буфера эта команда обычно используется в сочетании с утилитой фильтрации текста `grep`. Например, для поиска сообщений, связанных с устройствами универсальной последовательной шины:

```bash
root@debian:~# dmesg | grep "usb"
[    1.241182] usbcore: registered new interface driver usbfs
[    1.241188] usbcore: registered new interface driver hub
[    1.250968] usbcore: registered new device driver usb
[    1.339754] usb usb1: New USB device found, idVendor=1d6b, idProduct=0001, bcdDevice= 4.19
[    1.339756] usb usb1: New USB device strings: Mfr=3, Product=2, SerialNumber=1
(...)
```

## Управляемые Упражнения <a href="#sec.108.2_01-ge" id="sec.108.2_01-ge"></a>

1.  Какие утилиты / команды вы бы использовали в следующих сценариях:

    | Назначение и файл журнала                        | Полезность |
    | ------------------------------------------------ | ---------- |
    | Читать `/var/log/syslog.7.gz`                    |            |
    | Читать `/var/log/syslog`                         |            |
    | Отфильтруйте слово `renewal` в `/var/log/syslog` |            |
    | Читать `/var/log/faillog`                        |            |
    | Читать `/var/log/syslog` динамически             |            |
2. Переставьте следующие записи в журнале таким образом, чтобы они представляли собой корректное сообщение в журнале с правильной структурой:
   * `debian-server`
   * `sshd`
   * `[515]:`
   * `Sep 13 21:47:56`
   *   `Server listening on 0.0.0.0 port 22`

       Правильный порядок таков:
3. Какие правила вы бы добавили в `/etc/rsyslog.conf` для выполнения каждого из следующих действий:
   * Отправьте все сообщения с объекта `mail` с приоритетом/важностью `crit` (и выше) на адрес `/var/log/mail.crit`:
   * Отправьте все сообщения с объекта `mail` с приоритетами `alert` и `emergency` на объект `/var/log/mail.urgent`:
   * За исключением сообщений, поступающих с серверов `cron` и `ntp`, отправляйте все сообщения — независимо от их сервера и приоритета — на `/var/log/allmessages`:
   * Сначала настройте все необходимые параметры, а затем отправьте все сообщения с устройства `mail` на удалённый хост, IP-адрес которого `192.168.1.88` , используя TCP и указав порт по умолчанию:
   * Независимо от их возможности, отправляйте все сообщения с `warning` приоритетом (_только с `warning` приоритетом_) в  `/var/log/warnings что бы` предотвратить чрезмерную запись на диск:
4.  Рассмотрите следующую строфу из `/etc/logrotate.d/samba` и объясните:

    ```
    carol@debian:~$ sudo head -n 11 /etc/logrotate.d/samba
    /var/log/samba/log.smbd {
            weekly
            missingok
            rotate 7
            postrotate
                    [ ! -f /var/run/samba/smbd.pid ] || /etc/init.d/smbd reload > /dev/null
            endscript
            compress
            delaycompress
            notifempty
    }
    ```

    | Вариант         | Значение |
    | --------------- | -------- |
    | `weekly`        |          |
    | `missingok`     |          |
    | `rotate 7`      |          |
    | `postrotate`    |          |
    | `endscript`     |          |
    | `compress`      |          |
    | `delaycompress` |          |
    | `notifyempty`   |          |

## Исследовательские упражнения <a href="#sec.108.2_01-ee" id="sec.108.2_01-ee"></a>

1.  В разделе «Шаблоны и условия фильтрации» мы использовали _фильтр на основе выражений_ в качестве условия фильтрации. _Фильтры на основе свойств_ — это ещё один тип фильтров, уникальный для `rsyslogd`. Преобразуем наш _фильтр на основе выражений_ в _фильтр на основе свойств_:

    | Фильтр на основе выражений                        | Фильтр на основе свойств |
    | ------------------------------------------------- | ------------------------ |
    | `if $FROMHOST-IP=='192.168.1.4' then ?RemoteLogs` |                          |
2. `omusrmsg` Это `rsyslog` встроенный модуль, который упрощает уведомление пользователей (он отправляет сообщения журнала на терминал пользователя). Напишите правило для отправки всех _экстренных_ сообщений со всех объектов как `root` и обычному пользователю `carol`.

## Краткие сведения <a href="#sec.108.2_01-su" id="sec.108.2_01-su"></a>

На этом уроке вы усвоили:

* Ведение журналов имеет решающее значение для системного администрирования.
* `rsyslogd` Утилита отвечает за ведение журналов в порядке и чистоте.
* Некоторые службы сами заботятся о своих журналах.
* Грубо говоря, журналы можно разделить на системные журналы и журналы служб/программ.
* Существует ряд утилит, удобных для чтения журналов: `less`, `more`, `zless`, `zmore`, `grep`, `head` и `tail`.
* Большинство файлов журналов представляют собой текстовые файлы, однако есть небольшое количество двоичных файлов журналов.
* С точки зрения журнала, `rsyslogd` получает соответствующую информацию из специальных файлов (сокетов и буферов памяти) перед ее обработкой.
* Для классификации журналов `rsyslogd` используются правила в `/etc/rsyslog.conf` или `/etc/rsyslog.d/*`.
* Любой пользователь может вручную вводить свои собственные сообщения в системный журнал с помощью `logger` утилиты.
* `rsyslog` позволяет хранить все журналы в IP-сетях на централизованном сервере журналов.
* Шаблоны удобны для динамического форматирования имен файлов журналов.
* Ротация журналов преследует двоякую цель: не допустить, чтобы старые журналы занимали слишком много места на диске, и сделать журналы консультаций управляемыми.

## Ответы на Упражнения с Руководством <a href="#sec.108.2_01-age" id="sec.108.2_01-age"></a>

1.  Какие утилиты / команды вы бы использовали в следующих сценариях:

    | Назначение и файл журнала                        | Полезность          |
    | ------------------------------------------------ | ------------------- |
    | Читать `/var/log/syslog.7.gz`                    | `zmore` или `zless` |
    | Читать `/var/log/syslog`                         | `more` или `less`   |
    | Отфильтруйте слово `renewal` в `/var/log/syslog` | `grep`              |
    | Читать `/var/log/faillog`                        | `faillog -a`        |
    | Читать `/var/log/syslog` динамически             | `tail -f`           |
2. Переставьте следующие записи в журнале таким образом, чтобы они представляли собой корректное сообщение в журнале с правильной структурой:
   * `debian-server`
   * `sshd`
   * `[515]:`
   * `Sep 13 21:47:56`
   *   `Server listening on 0.0.0.0 port 22`

       Правильный порядок таков:

       ```bash
       Sep 13 21:47:56 debian-server sshd[515]: Server listening on 0.0.0.0 port 22
       ```
3. Какие правила вы бы добавили в `/etc/rsyslog.conf` для выполнения каждого из следующих действий:
   *   Отправьте все сообщения с объекта `mail` с приоритетом/важностью `crit` (и выше) на адрес `/var/log/mail.crit`:

       ```bash
       mail.crit                 /var/log/mail.crit
       ```
   *   Отправьте все сообщения с объекта `mail` с приоритетами `alert` и `emergency` на объект `/var/log/mail.urgent`:

       ```bash
       mail.alert                        /var/log/mail.urgent
       ```
   *   За исключением сообщений, поступающих с серверов `cron` и `ntp`, отправляйте все сообщения — независимо от их сервера и приоритета — на `/var/log/allmessages`:

       ```bash
       *.*;cron.none;ntp.none                 /var/log/allmessages
       ```
   *   Сначала настройте все необходимые параметры, а затем отправьте все сообщения с устройства `mail` на удалённый хост, IP-адрес которого `192.168.1.88` , используя TCP и указав порт по умолчанию:

       ```
       mail.* @@192.168.1.88:514
       ```
   *   Независимо от их возможности, отправляйте все сообщения с `warning` приоритетом (_только с `warning` приоритетом_) в`/var/log/warnings что бы` предотвратить чрезмерную запись на диск:

       ```
       *.=warning                        -/var/log/warnings
       ```
4.  Рассмотрите следующую строфу из `/etc/logrotate.d/samba` и объясните:

    ```bash
    carol@debian:~$ sudo head -n 11 /etc/logrotate.d/samba
    /var/log/samba/log.smbd {
            weekly
            missingok
            rotate 7
            postrotate
                    [ ! -f /var/run/samba/smbd.pid ] || /etc/init.d/smbd reload > /dev/null
            endscript
            compress
            delaycompress
            notifempty
    }
    ```

    | Вариант         | Значение                                                                                          |
    | --------------- | ------------------------------------------------------------------------------------------------- |
    | `weekly`        | Меняйте файлы журналов на еженедельной основе.                                                    |
    | `missingok`     | Если журнал отсутствует, не выводите сообщение об ошибке, просто переходите к следующему журналу. |
    | `rotate 7`      | Накопите резерв на 7 недель.                                                                      |
    | `postrotate`    | Запустите скрипт в следующей строке после поворота журналов.                                      |
    | `endscript`     | Укажите конец скрипта _postrotate_.                                                               |
    | `compress`      | Сожмите журналы с помощью `gzip`.                                                                 |
    | `delaycompress` | В сочетании с `compress` отложите сжатие до следующего цикла вращения.                            |
    | `notifyempty`   | Не поворачивайте журнал, если он пустой.                                                          |

## Ответы на Исследовательские упражнения <a href="#sec.108.2_01-aee" id="sec.108.2_01-aee"></a>

1.  В разделе «Шаблоны и условия фильтрации» мы использовали _фильтр на основе выражений_ в качестве условия фильтрации. _Фильтры на основе свойств_ — это ещё один тип фильтров, уникальный для `rsyslogd`. Преобразуем наш _фильтр на основе выражений_ в _фильтр на основе свойств_:

    | Фильтр на основе выражений                        | Фильтр на основе свойств                           |
    | ------------------------------------------------- | -------------------------------------------------- |
    | `if $FROMHOST-IP=='192.168.1.4' then ?RemoteLogs` | `:fromhost-ip, isequal, "192.168.1.4" ?RemoteLogs` |
2.  `omusrmsg` Это `rsyslog` встроенный модуль, который упрощает уведомление пользователей (он отправляет сообщения журнала на терминал пользователя). Напишите правило для отправки всех _экстренных_ сообщений со всех объектов как `root` и обычному пользователю `carol`.

    ```bash
    *.emerg                        :omusrmsg:root,carol
    ```
