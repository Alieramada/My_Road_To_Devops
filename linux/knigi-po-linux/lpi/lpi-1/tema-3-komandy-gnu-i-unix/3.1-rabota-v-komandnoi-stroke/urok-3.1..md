# Урок 3.1.

## Введение <a href="#sec.103.1_01-in" id="sec.103.1_01-in"></a>

Новички в мире администрирования Linux и оболочки Bash часто чувствуют себя немного потерянными без успокаивающего комфорта графического интерфейса. Они привыкли к тому, что щелчок правой кнопкой мыши открывает доступ к визуальным подсказкам и контекстной информации, которые предоставляют графические файловые менеджеры. Поэтому важно быстро изучить и освоить относительно небольшой набор инструментов командной строки, с помощью которых можно мгновенно получить доступ ко всем данным, которые предоставляет ваш старый графический интерфейс, и даже больше.

## Получение системной информации <a href="#getting_system_information" id="getting_system_information"></a>

Глядя на мигающий прямоугольник командной строки, вы, скорее всего, зададитесь вопросом: «Где я?» Или, точнее, «Где я сейчас в файловой системе Linux, и если, скажем, я создам новый файл, где он будет находиться?» То, что вам нужно, — это ваш _текущий рабочий каталог_, и команда `pwd` подскажет вам, что вы хотите узнать:

```bash
$ pwd
/home/frank
```

Предположим, что Фрэнк в данный момент вошёл в систему и находится в своём домашнем каталоге: `/home/frank/`. Если Фрэнк создаст пустой файл с помощью команды `touch` без указания другого местоположения в файловой системе, файл будет создан в `/home/frank/`. Если мы перечислим содержимое каталога с помощью `ls`, то увидим новый файл:

```bash
$ touch newfile
$ ls
newfile
```

Помимо вашего местоположения в файловой системе, вам часто понадобится информация о системе Linux, на которой вы работаете. Это может включать точный номер выпуска вашего дистрибутива или версию ядра Linux, которая загружена в данный момент. Здесь вам нужен `uname` инструмент. И, в частности, `uname` использование `-a` опции (“все”).

```bash
$ uname -a
 Linux base 4.18.0-18-generic #19~18.04.1-Ubuntu SMP Fri Apr 5 10:22:13 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```

Здесь `uname` показывает, что на компьютере Фрэнка установлена версия ядра Linux 4.18.0 и работает Ubuntu 18.04 на 64-битном (`x86_64`) процессоре.

## Получение информации о командах <a href="#getting_command_information" id="getting_command_information"></a>

Вы часто будете сталкиваться с документацией, в которой говорится о командах Linux, с которыми вы ещё не знакомы. Сама командная строка предлагает всевозможную полезную информацию о том, что делают команды и как их эффективно использовать. Возможно, самую полезную информацию можно найти в многочисленных файлах `man` системы.

Как правило, разработчики Linux пишут `man` файлы и распространяют их вместе с создаваемыми ими утилитами. `man` файлы представляют собой высокоструктурированные документы, содержимое которых интуитивно разделено стандартными заголовками разделов. Набрав `man`, за которым следует название команды, вы получите информацию, которая включает название команды, краткий обзор использования, более подробное описание и некоторые важные исторические сведения и сведения о лицензировании. Вот пример.:

```bash
$ man uname
UNAME(1) User Commands UNAME(1)
NAME
 uname - print system information
SYNOPSIS
 uname [OPTION]...
DESCRIPTION
 Print certain system information. With no OPTION, same as -s.
 -a, --all
 print all information, in the following order, except omit -p
 and -i if unknown:
 -s, --kernel-name
 print the kernel name
 -n, --nodename
 print the network node hostname
 -r, --kernel-release
 print the kernel release
 -v, --kernel-version
 print the kernel version
 -m, --machine
 print the machine hardware name
 -p, --processor
 print the processor type (non-portable)
 -i, --hardware-platform
 print the hardware platform (non-portable)
 -o, --operating-system
 print the operating system
 --help display this help and exit
 --version
 output version information and exit
AUTHOR
 Written by David MacKenzie.
REPORTING BUGS
 GNU coreutils online help: <http://www.gnu.org/software/coreutils/>
 Report uname translation bugs to
 <http://translationproject.org/team/>
COPYRIGHT
 Copyright©2017 Free Software Foundation, Inc. License GPLv3+: GNU
 GPL version 3 or later <http://gnu.org/licenses/gpl.html>.
 This is free software: you are free to change and redistribute it.
 There is NO WARRANTY, to the extent permitted by law.
SEE ALSO
 arch(1), uname(2)
 Full documentation at: <http://www.gnu.org/software/coreutils/uname>
 or available locally via: info '(coreutils) uname invocation'
GNU coreutils 8.28 January 2018 UNAME(1)
```

`man` работает только в том случае, если вы вводите точное имя команды. Если, однако, вы не уверены в названии нужной вам команды, вы можете использовать команду `apropos` для поиска по названиям и описаниям страниц `man`. Например, если вы не можете вспомнить, что именно `uname` показывает текущую версию ядра Linux, вы можете передать слово `kernel` в `apropros`. Вы, вероятно, получите много строк вывода, но среди них должны быть следующие:

```bash
$ apropos kernel
systemd-udevd-kernel.socket (8) - Device event managing daemon
uname (2)            - get name and information about current kernel
urandom (4)          - kernel random number source devices
```

Если вам не нужна полная документация по команде, вы можете быстро получить основные сведения о команде с помощью `type`. В этом примере `type` используется для одновременного запроса четырёх отдельных команд. Результаты показывают, что `cp` («копирование») — это программа, которая находится в `/bin/cp` и что `kill` (изменение состояния запущенного процесса) — это встроенная команда оболочки, то есть она фактически является частью самой оболочки Bash:

```bash
$ type uname cp kill which
uname is hashed (/bin/uname)
cp is /bin/cp
kill is a shell builtin
which is /usr/bin/which
```

Обратите внимание, что `cp` — это не только обычная двоичная команда, как `uname`, но и «хешированная». Это связано с тем, что Фрэнк недавно использовал `uname` и для повышения эффективности системы она была добавлена в хеш-таблицу, чтобы сделать её более доступной при следующем запуске. Если бы Фрэнк запустил `type uname` после загрузки системы, он бы обнаружил, что `type` снова описывает `uname` как обычную двоичную команду.

{% hint style="info" %}
Более быстрый способ очистить хеш-таблицу — выполнить команду `hash -d`.
{% endhint %}

Иногда — особенно при работе с автоматизированными сценариями — вам понадобится более простой источник информации о команде. Команда `which` , которую мы использовали в предыдущей команде `type` , не возвращает ничего, кроме абсолютного местоположения команды. В этом примере находятся команды `uname` и `which` .

```bash
$ which uname which
/bin/uname/
/usr/bin/which

```

{% hint style="info" %}
Если вы хотите отобразить информацию о «встроенных» командах, вы можете использовать команду `help` .
{% endhint %}

