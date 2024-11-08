# Урок 2.4.

## Введение <a href="#sec.102.4_01-in" id="sec.102.4_01-in"></a>

Давным-давно, когда Linux только зарождался, самым распространённым способом распространения программного обеспечения был сжатый файл (обычно `.tar.gz`-архив) с исходным кодом, который нужно было распаковать и скомпилировать самостоятельно.

Однако по мере роста количества и сложности программного обеспечения стала очевидной необходимость в способе распространения предварительно скомпилированного программного обеспечения. В конце концов, не у всех были ресурсы, как временные, так и вычислительные, для компиляции крупных проектов, таких как ядро Linux или X-сервер.

Вскоре усилия по стандартизации способов распространения этих программных «пакетов» усилились, и появились первые менеджеры пакетов. Эти инструменты значительно упростили установку, настройку и удаление программного обеспечения из системы.

Одним из них был формат пакетов Debian (`.deb`) и инструмент для работы с пакетами (`dpkg`). Сегодня они широко используются не только в самом Debian, но и в его производных, таких как Ubuntu и производные от неё.

Ещё один популярный инструмент управления пакетами в системах на базе Debian — _Advanced Package Tool_ (`apt`), который упрощает многие аспекты установки, обслуживания и удаления пакетов.

В этом уроке мы узнаем, как использовать `dpkg` и `apt` для получения, установки, обслуживания и удаления программного обеспечения в системе Linux на базе Debian.

## Debian Package Tool (dpkg) <a href="#the_debian_package_tool_dpkg" id="the_debian_package_tool_dpkg"></a>

_Debian Package tool_ (`dpkg`) — это основная утилита для установки, настройки, обслуживания и удаления программных пакетов в системах на базе Debian. Самая простая операция — установка `.deb` пакета, которую можно выполнить с помощью:

```bash
# dpkg -i PACKAGENAME
```

Где `PACKAGENAME` - это имя `.deb` файла, который вы хотите установить.

Обновление пакетов выполняется аналогичным образом. Перед установкой пакета `dpkg` проверяется, существует ли в системе предыдущая версия. Если да, то пакет будет обновлен до новой версии. Если нет, то будет установлена новая копия.

## **Работа с зависимостями**

Чаще всего работа пакета зависит от других пакетов. Например, для работы редактора изображений могут потребоваться библиотеки для открытия файлов JPEG, а для работы другой утилиты может потребоваться набор виджетов, например Qt или GTK, для пользовательского интерфейса.

`dpkg` система проверит, установлены ли эти зависимости в вашей системе, и не сможет установить пакет, если они не установлены. В этом случае `dpkg` будет указан список отсутствующих пакетов. Однако система _не может_ самостоятельно решить проблему зависимостей. Пользователь должен найти `.deb` пакеты с соответствующими зависимостями и установить их.

В приведённом ниже примере пользователь пытается установить пакет видеоредактора OpenShot, но ему не хватает некоторых зависимостей:

```bash
# dpkg -i openshot-qt_2.4.3+dfsg1-1_all.deb
(Reading database ... 269630 files and directories currently installed.)
Preparing to unpack openshot-qt_2.4.3+dfsg1-1_all.deb ...
Unpacking openshot-qt (2.4.3+dfsg1-1) over (2.4.3+dfsg1-1) ...
dpkg: dependency problems prevent configuration of openshot-qt:
 openshot-qt depends on fonts-cantarell; however:
  Package fonts-cantarell is not installed.
 openshot-qt depends on python3-openshot; however:
  Package python3-openshot is not installed.
 openshot-qt depends on python3-pyqt5; however:
  Package python3-pyqt5 is not installed.
 openshot-qt depends on python3-pyqt5.qtsvg; however:
  Package python3-pyqt5.qtsvg is not installed.
 openshot-qt depends on python3-pyqt5.qtwebkit; however:
  Package python3-pyqt5.qtwebkit is not installed.
 openshot-qt depends on python3-zmq; however:
  Package python3-zmq is not installed.

dpkg: error processing package openshot-qt (--install):
 dependency problems - leaving unconfigured
Processing triggers for mime-support (3.60ubuntu1) ...
Processing triggers for gnome-menus (3.32.0-1ubuntu1) ...
Processing triggers for desktop-file-utils (0.23-4ubuntu1) ...
Processing triggers for hicolor-icon-theme (0.17-2) ...
Processing triggers for man-db (2.8.5-2) ...
Errors were encountered while processing:
 openshot-qt
```

Как показано выше, OpenShot зависит от пакетов `fonts-cantarell`, `python3-openshot`, `python3-pyqt5`, `python3-pyqt5.qtsvg`, `python3-pyqt5.qtwebkit` и `python3-zmq`. Все они должны быть установлены до начала установки OpenShot.

## **Удаление пакетов**

Чтобы удалить пакет, передайте параметр `-r` в `dpkg`, а затем укажите имя пакета. Например, следующая команда удалит пакет `unrar` из системы:\


```bash
# dpkg -r unrar
(Reading database ... 269630 files and directories currently installed.)
Removing unrar (1:5.6.6-2) ...
Processing triggers for man-db (2.8.5-2) ...
```

При удалении также выполняется проверка зависимостей, и пакет не может быть удалён, если не будут удалены все другие пакеты, от которых он зависит. Если вы попытаетесь это сделать, вы получите сообщение об ошибке, подобное приведенному ниже:

```bash
# dpkg -r p7zip
dpkg: dependency problems prevent removal of p7zip:
 winetricks depends on p7zip; however:
  Package p7zip is to be removed.
 p7zip-full depends on p7zip (= 16.02+dfsg-6).

dpkg: error processing package p7zip (--remove):
 dependency problems - not removing
Errors were encountered while processing:
 p7zip
```

Вы можете передать в `dpkg -r` несколько названий пакетов, чтобы удалить их все сразу.

При удалении пакета соответствующие файлы конфигурации остаются в системе. Если вы хотите удалить _все_, связанное с пакетом, используйте опцию `-P` (очистка) вместо `-r`.

{% hint style="info" %}
Вы можете принудительно `dpkg` установить или удалить пакет, даже если не соблюдены зависимости, добавив параметр `--force` как в `dpkg -i --force PACKAGENAME`. Однако это, скорее всего, приведёт к сбою установленного пакета или даже вашей системы. _Не используйте_ `--force`, если вы не уверены в своих действиях.
{% endhint %}

## **Получение информации о пакете**

Чтобы получить информацию о `.deb` пакете, такую как его версия, архитектура, сопровождающий, зависимости и многое другое, используйте `dpkg` команду с `-I` параметром, за которым следует имя файла пакета, который вы хотите проверить:

```bash
# dpkg -I google-chrome-stable_current_amd64.deb
 new Debian package, version 2.0.
 size 59477810 bytes: control archive=10394 bytes.
    1222 bytes,    13 lines      control
   16906 bytes,   457 lines   *  postinst             #!/bin/sh
   12983 bytes,   344 lines   *  postrm               #!/bin/sh
    1385 bytes,    42 lines   *  prerm                #!/bin/sh
 Package: google-chrome-stable
 Version: 76.0.3809.100-1
 Architecture: amd64
 Maintainer: Chrome Linux Team <chromium-dev@chromium.org>
 Installed-Size: 205436
 Pre-Depends: dpkg (>= 1.14.0)
 Depends: ca-certificates, fonts-liberation, libappindicator3-1, libasound2 (>= 1.0.16), libatk-bridge2.0-0 (>= 2.5.3), libatk1.0-0 (>= 2.2.0), libatspi2.0-0 (>= 2.9.90), libc6 (>= 2.16), libcairo2 (>= 1.6.0), libcups2 (>= 1.4.0), libdbus-1-3 (>= 1.5.12), libexpat1 (>= 2.0.1), libgcc1 (>= 1:3.0), libgdk-pixbuf2.0-0 (>= 2.22.0), libglib2.0-0 (>= 2.31.8), libgtk-3-0 (>= 3.9.10), libnspr4 (>= 2:4.9-2~), libnss3 (>= 2:3.22), libpango-1.0-0 (>= 1.14.0), libpangocairo-1.0-0 (>= 1.14.0), libuuid1 (>= 2.16), libx11-6 (>= 2:1.4.99.1), libx11-xcb1, libxcb1 (>= 1.6), libxcomposite1 (>= 1:0.3-1), libxcursor1 (>> 1.1.2), libxdamage1 (>= 1:1.1), libxext6, libxfixes3, libxi6 (>= 2:1.2.99.4), libxrandr2 (>= 2:1.2.99.3), libxrender1, libxss1, libxtst6, lsb-release, wget, xdg-utils (>= 1.0.2)
 Recommends: libu2f-udev
 Provides: www-browser
 Section: web
 Priority: optional
 Description: The web browser from Google
  Google Chrome is a browser that combines a minimal design with sophisticated technology to make the web faster, safer, and easier.
```

## **Список установленных пакетов и их содержимого**

Чтобы получить список всех пакетов, установленных в вашей системе, используйте опцию `--get-selections`, например `dpkg --get-selections`. Вы также можете получить список всех файлов, установленных конкретным пакетом, передав параметр `-L PACKAGENAME` в `dpkg`, как показано ниже:

```bash
# dpkg -L unrar
/.
/usr
/usr/bin
/usr/bin/unrar-nonfree
/usr/share
/usr/share/doc
/usr/share/doc/unrar
/usr/share/doc/unrar/changelog.Debian.gz
/usr/share/doc/unrar/copyright
/usr/share/man
/usr/share/man/man1
/usr/share/man/man1/unrar-nonfree.1.gz
```

## Принадлежность файла пакету

Иногда вам может понадобиться узнать, какому пакету принадлежит конкретный файл в вашей системе. Это можно сделать с помощью утилиты `dpkg-query` с параметром `-S` и путём к нужному файлу:

```bash
# dpkg-query -S /usr/bin/unrar-nonfree
unrar: /usr/bin/unrar-nonfree
```

## **Перенастройка установленных пакетов**

После установки пакета выполняется этап настройки, называемый _пост-установкой_, в ходе которого запускается скрипт для настройки всего необходимого для работы программного обеспечения, например разрешений, размещения файлов конфигурации и т. д. Также могут быть заданы некоторые вопросы пользователю для настройки параметров работы программного обеспечения.

Иногда из-за повреждённого или некорректного файла конфигурации вам может потребоваться восстановить настройки пакета до «исходного» состояния. Или вы можете захотеть изменить ответы, которые вы дали на первоначальные вопросы о конфигурации. Для этого запустите утилиту `dpkg-reconfigure` и укажите имя пакета.

Эта программа создаст резервную копию старых файлов конфигурации, распакует новые файлы в нужные каталоги и запустит скрипт _после установки_, предоставленный пакетом, как если бы пакет был установлен впервые. Попробуйте перенастроить пакет `tzdata` с помощью следующего примера:

```bash
# dpkg-reconfigure tzdata
```

#### Расширенный инструмент для создания пакетов  _Advanced Package Tool_  (apt) <a href="#advanced_package_tool_apt" id="advanced_package_tool_apt"></a>

_Advanced Package Tool_ (APT) — это система управления пакетами, включающая набор инструментов, которые значительно упрощают установку, обновление, удаление пакетов и управление ими. APT предоставляет такие функции, как расширенный поиск и автоматическое разрешение зависимостей.

APT не является “заменой” `dpkg`. Вы можете думать о нем как о “интерфейсе”, оптимизирующем операции и заполняющем пробелы в `dpkg` функциональности, такие как разрешение зависимостей.

APT работает совместно с репозиториями программного обеспечения, которые содержат доступные для установки пакеты. Такими репозиториями могут быть локальный или удалённый сервер или (что реже) даже компакт-диск.

Дистрибутивы Linux, такие как Debian и Ubuntu, поддерживают собственные репозитории, а другие репозитории могут поддерживаться разработчиками или группами пользователей для предоставления программного обеспечения, недоступного в основных репозиториях дистрибутивов.

Существует множество утилит, взаимодействующих с APT, основными из которых являются:

`apt-get`

используется для загрузки, установки, обновления или удаления пакетов из системы.

`apt-cache`

используется для выполнения операций, таких как поиск, в индексе пакета.

`apt-file`

используется для поиска файлов внутри пакетов.

Существует также более «дружественная» утилита под названием просто `apt`, объединяющая наиболее часто используемые функции `apt-get` и `apt-cache` в одной утилите. Многие команды для `apt` совпадают с командами для `apt-get`, поэтому во многих случаях их можно использовать взаимозаменяемо. Однако, поскольку `apt` может быть не установлена в системе, рекомендуется изучить, как использовать `apt-get` и `apt-cache`.

{% hint style="info" %}
`apt` и `apt-get` может потребоваться подключение к сети, поскольку пакеты и индексы пакетов могут быть загружены с удалённого сервера.
{% endhint %}

## **Обновление индекса пакета**

Перед установкой или обновлением программного обеспечения с помощью APT рекомендуется сначала обновить индекс пакетов, чтобы получить информацию о новых и обновлённых пакетах. Это делается с помощью команды `apt-get` с параметром `update`:

```bash
# apt-get update
Ign:1 http://dl.google.com/linux/chrome/deb stable InRelease
Hit:2 https://repo.skype.com/deb stable InRelease
Hit:3 http://us.archive.ubuntu.com/ubuntu disco InRelease
Hit:4 http://repository.spotify.com stable InRelease
Hit:5 http://dl.google.com/linux/chrome/deb stable Release
Hit:6 http://apt.pop-os.org/proprietary disco InRelease
Hit:7 http://ppa.launchpad.net/system76/pop/ubuntu disco InRelease
Hit:8 http://us.archive.ubuntu.com/ubuntu disco-security InRelease
Hit:9 http://us.archive.ubuntu.com/ubuntu disco-updates InRelease
Hit:10 http://us.archive.ubuntu.com/ubuntu disco-backports InRelease
Reading package lists... Done
```

{% hint style="info" %}
Вместо `apt-get update` вы также можете использовать `apt update`.
{% endhint %}

## **Установка и удаление пакетов**

После обновления индекса пакетов вы можете установить пакет. Для этого введите `apt-get install`, а затем название пакета, который вы хотите установить:

```bash
# apt-get install xournal
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following NEW packages will be installed:
  xournal
0 upgraded, 1 newly installed, 0 to remove and 75 not upgraded.
Need to get 285 kB of archives.
After this operation, 1041 kB of additional disk space will be used.
```

Аналогично, для удаления пакета используйте `apt-get remove`, за которым следует название пакета:

```bash
# apt-get remove xournal
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages will be REMOVED:
  xournal
0 upgraded, 0 newly installed, 1 to remove and 75 not upgraded.
After this operation, 1041 kB disk space will be freed.
Do you want to continue? [Y/n]
```

Имейте в виду, что при установке или удалении пакетов APT автоматически разрешит зависимости. Это означает, что любые дополнительные пакеты, необходимые для устанавливаемого вами пакета, _также будут установлены_, а пакеты, зависящие от удаляемого вами пакета, _также будут удалены_. APT всегда показывает, что будет установлено или удалено, прежде чем спросить, хотите ли вы продолжить:

```bash
# apt-get remove p7zip
Reading package lists... Done
Building dependency tree
The following packages will be REMOVED:
  android-libbacktrace android-libunwind android-libutils
  android-libziparchive android-sdk-platform-tools fastboot p7zip p7zip-full
0 upgraded, 0 newly installed, 8 to remove and 75 not upgraded.
After this operation, 6545 kB disk space will be freed.
Do you want to continue? [Y/n]
```

Обратите внимание, что при удалении пакета соответствующие файлы конфигурации остаются в системе. Чтобы удалить пакет _и_ все файлы конфигурации, используйте параметр `purge` вместо `remove` или параметр `remove` с опцией `--purge`:

```bash
# apt-get purge p7zip
```

или

```bash
# apt-get remove --purge p7zip
```

{% hint style="info" %}
Вы также можете использовать `apt install` и `apt remove`.
{% endhint %}

## **Исправление нарушенных зависимостей**

В системе могут быть «нарушенные зависимости». Это означает, что один или несколько установленных пакетов зависят от других пакетов, которые не были установлены или больше не существуют. Это может произойти из-за ошибки APT или из-за установки пакета вручную.

Чтобы решить эту проблему, используйте команду `apt-get install -f` . Она попытается «исправить» повреждённые пакеты, установив недостающие зависимости и обеспечив согласованность всех пакетов.

{% hint style="info" %}
Вы также можете использовать `apt install -f`.
{% endhint %}

## **Обновление пакетов**

APT можно использовать для автоматического обновления любых установленных пакетов до последних версий, доступных в репозиториях. Это делается с помощью команды `apt-get upgrade` . Перед её выполнением сначала обновите индекс пакетов с помощью `apt-get update`:

```bash
# apt-get update
Hit:1 http://us.archive.ubuntu.com/ubuntu disco InRelease
Hit:2 http://us.archive.ubuntu.com/ubuntu disco-security InRelease
Hit:3 http://us.archive.ubuntu.com/ubuntu disco-updates InRelease
Hit:4 http://us.archive.ubuntu.com/ubuntu disco-backports InRelease
Reading package lists... Done

# apt-get upgrade
Reading package lists... Done
Building dependency tree
Reading state information... Done
Calculating upgrade... Done
The following packages have been kept back:
  gnome-control-center
The following packages will be upgraded:
  cups cups-bsd cups-client cups-common cups-core-drivers cups-daemon
  cups-ipp-utils cups-ppdc cups-server-common firefox-locale-ar (...)

74 upgraded, 0 newly installed, 0 to remove and 1 not upgraded.
Need to get 243 MB of archives.
After this operation, 30.7 kB of additional disk space will be used.
Do you want to continue? [Y/n]
```

Сводка внизу выходных данных показывает, сколько пакетов будет обновлено, сколько будет установлено, удалено или сохранено обратно, общий размер загрузки и сколько дополнительного места на диске потребуется для завершения операции. Чтобы завершить обновление, просто ответьте `Y` и дождитесь `apt-get` завершения задания.

Чтобы обновить один пакет, просто выполните команду `apt-get upgrade` с указанием имени пакета. Как и в случае с `dpkg`, `apt-get` сначала проверит, установлена ли предыдущая версия пакета. Если да, то пакет будет обновлен до последней версии, доступной в репозитории. Если нет, то будет установлена новая копия.

{% hint style="info" %}
Вы также можете использовать `apt upgrade` и `apt update`.
{% endhint %}

## **Локальный кэш**

При установке или обновлении пакета соответствующий файл `.deb` загружается в локальный каталог кэша перед установкой пакета. По умолчанию этот каталог называется `/var/cache/apt/archives`. Частично загруженные файлы копируются в `/var/cache/apt/archives/partial/`.

По мере установки и обновления пакетов каталог кэша может стать довольно большим. Чтобы освободить место, вы можете очистить кэш с помощью команды `apt-get clean` . Это удалит содержимое каталогов `/var/cache/apt/archives` и `/var/cache/apt/archives/partial/` .

{% hint style="info" %}
Вы также можете использовать `apt clean`.
{% endhint %}

## **Поиск пакетов**

Утилиту `apt-cache` можно использовать для выполнения операций с индексом пакетов, таких как поиск определённого пакета или перечисление пакетов, содержащих определённый файл.

Чтобы выполнить поиск, используйте `apt-cache search` и введите шаблон поиска. В результате будет получен список всех пакетов, содержащих шаблон в названии, описании или файлах.

```bash
# apt-cache search p7zip
liblzma-dev - XZ-format compression library - development files
liblzma5 - XZ-format compression library
forensics-extra - Forensics Environment - extra console components (metapackage)
p7zip - 7zr file archiver with high compression ratio
p7zip-full - 7z and 7za file archivers with high compression ratio
p7zip-rar - non-free rar module for p7zip
```

\
В приведённом выше примере запись `liblzma5 - XZ-format compression library` не соответствует шаблону. Однако если мы отобразим полную информацию, включая описание, для пакета с помощью параметра `show`, то найдём там шаблон:

```bash
# apt-cache show liblzma5
Package: liblzma5
Architecture: amd64
Version: 5.2.4-1
Multi-Arch: same
Priority: required
Section: libs
Source: xz-utils
Origin: Ubuntu
Maintainer: Ubuntu Developers <ubuntu-devel-discuss@lists.ubuntu.com>
Original-Maintainer: Jonathan Nieder <jrnieder@gmail.com>
Bugs: https://bugs.launchpad.net/ubuntu/+filebug
Installed-Size: 259
Depends: libc6 (>= 2.17)
Breaks: liblzma2 (<< 5.1.1alpha+20110809-3~)
Filename: pool/main/x/xz-utils/liblzma5_5.2.4-1_amd64.deb
Size: 92352
MD5sum: 223533a347dc76a8cc9445cfc6146ec3
SHA1: 8ed14092fb1caecfebc556fda0745e1e74ba5a67
SHA256: 01020b5a0515dbc9a7c00b464a65450f788b0258c3fbb733ecad0438f5124800
Homepage: https://tukaani.org/xz/
Description-en: XZ-format compression library
 XZ is the successor to the Lempel-Ziv/Markov-chain Algorithm
 compression format, which provides memory-hungry but powerful
 compression (often better than bzip2) and fast, easy decompression.
 .
 The native format of liblzma is XZ; it also supports raw (headerless)
 streams and the older LZMA format used by lzma. (For 7-Zip's related
 format, use the p7zip package instead.)
```

Вы можете использовать _регулярные выражения_ в шаблоне поиска, что позволяет выполнять очень сложные (и точные) поиски. Однако эта тема выходит за рамки данного урока.

{% hint style="info" %}
Вы также можете использовать `apt search` вместо `apt-cache search` и `apt show` вместо `apt-cache show`.
{% endhint %}

## **Список источников**

APT использует список источников, чтобы знать, откуда брать пакеты. Этот список хранится в файле `sources.list`, расположенном в каталоге `/etc/apt`. Этот файл можно редактировать напрямую с помощью текстового редактора, например `vi`, `pico` или `nano`, или с помощью графических утилит, таких как `aptitude` или `synaptic`.

Типичная строка внутри `sources.list` выглядит следующим образом:

```bash
deb http://us.archive.ubuntu.com/ubuntu/ disco main restricted universe multiverse
```

Синтаксис: тип архива, URL, Distribution и один или несколько компонентов, где:

Тип архива

Репозиторий может содержать пакеты с готовым к запуску программным обеспечением (двоичные пакеты, тип `deb`) или с исходным кодом этого программного обеспечения (исходные пакеты, тип `deb-src`). В приведённом выше примере представлены двоичные пакеты.

URL

URL-адрес репозитория.

Distribution

Название (или кодовое имя) дистрибутива, для которого предоставляются пакеты. В одном репозитории могут храниться пакеты для нескольких дистрибутивов. В приведенном выше примере `disco` — это кодовое имя Ubuntu 19.04 _Disco Dingo_.

Компоненты

Каждый компонент представляет собой набор пакетов. Эти компоненты могут отличаться в разных дистрибутивах Linux. Например, в Ubuntu и её производных они следующие:

`main`

содержит официально поддерживаемые пакеты с открытым исходным кодом.

`restricted`

содержит официально поддерживаемое программное обеспечение с закрытым исходным кодом, например, драйверы устройств для графических карт.

`universe`

содержит программное обеспечение с открытым исходным кодом, поддерживаемое сообществом.

`multiverse`

содержит неподдерживаемое программное обеспечение с закрытым исходным кодом или обремененное патентами.

В Debian основными компонентами являются:

`main`

состоит из пакетов, соответствующих _Руководству по свободному программному обеспечению Debian Debian Free Software Guidelines_ (DFSG), которые не используют для работы программное обеспечение за пределами этой области. Включённые сюда пакеты считаются частью дистрибутива Debian.

`contrib`

содержит пакеты, совместимые с DFSG, но зависящие от других пакетов, которых нет в `main`.

`non-free`

содержит пакеты, которые не совместимы с DFSG.

`security`

содержит обновления для системы безопасности.

`backports`

содержит более новые версии пакетов, которые есть в `main`. Цикл разработки стабильных версий Debian довольно долгий (около двух лет), и это гарантирует, что пользователи смогут получать самые актуальные пакеты без необходимости изменять `main` основной репозиторий.

{% hint style="info" %}
Вы можете узнать больше о _Руководстве по свободному программному обеспечению Debian_ по адресу: [https://www.debian.org/social\_contract#guidelines](https://www.debian.org/social\_contract#guidelines)
{% endhint %}

Чтобы добавить новый репозиторий для получения пакетов, вы можете просто добавить соответствующую строку (обычно предоставляемую разработчиком репозитория) в конец `sources.list`, сохранить файл и перезагрузить индекс пакетов с помощью `apt-get update`. После этого пакеты из нового репозитория будут доступны для установки с помощью `apt-get install`.

Имейте в виду, что строки, начинающиеся с символа `#`, считаются комментариями и игнорируются.

**The `/etc/apt/sources.list.d` Directory**

Внутри каталога `/etc/apt/sources.list.d` вы можете добавлять файлы с дополнительными репозиториями, которые будут использоваться APT, без необходимости изменять основной файл `/etc/apt/sources.list` . Это простые текстовые файлы с тем же синтаксисом, что описан выше, и расширением `.list` .

Ниже вы видите содержимое файла под названием `/etc/apt/sources.list.d/buster-backports.list`:

```bash
deb http://deb.debian.org/debian buster-backports main contrib non-free
deb-src http://deb.debian.org/debian buster-backports main contrib non-free
```

## **Перечисление содержимого пакета и поиск файлов**

Утилиту под названием `apt-file` можно использовать для выполнения дополнительных операций в индексе пакетов, например для просмотра содержимого пакета или поиска пакета, содержащего определённый файл. Эта утилита может быть не установлена в вашей системе по умолчанию. В этом случае вы обычно можете установить её с помощью `apt-get`:

```bash
# apt-get install apt-file
```

После установки вам нужно будет обновить кэш пакета, используемый для `apt-file`:

```bash
# apt-file update
```

Обычно это занимает всего несколько секунд. После этого вы готовы к использованию `apt-file`.

Чтобы вывести список содержимого пакета, используйте параметр `list` и укажите название пакета:

```bash
# apt-file list unrar
unrar: /usr/bin/unrar-nonfree
unrar: /usr/share/doc/unrar/changelog.Debian.gz
unrar: /usr/share/doc/unrar/copyright
unrar: /usr/share/man/man1/unrar-nonfree.1.gz
```

{% hint style="info" %}
Вы также можете использовать `apt list` вместо `apt-file list`.
{% endhint %}

Вы можете найти файл во всех пакетах с помощью параметра `search` и имени файла. Например, если вы хотите узнать, в каком пакете находится файл с именем `libSDL2.so`, вы можете использовать:

```basic
# apt-file search libSDL2.so
libsdl2-dev: /usr/lib/x86_64-linux-gnu/libSDL2.so
```

Ответ заключается в пакете `libsdl2-dev`, который предоставляет файл `/usr/lib/x86_64-linux-gnu/libSDL2.so`.

Разница между `apt-file search` и `dpkg-query` заключается в том, что `apt-file search` также выполняет поиск в удалённых пакетах, а `dpkg-query` может показывать только файлы, принадлежащие установленному пакету.

## Упражнения с руководством <a href="#sec.102.4_01-ge" id="sec.102.4_01-ge"></a>

1. Какая команда используется для установки пакета с именем `package.deb` используя `dpkg`?
2. Используя `dpkg-query`, найдите, какой пакет содержит файл с именем `7zr.1.gz`.
3. Можно ли удалить пакет под названием `unzip` из системы с помощью `dpkg -r unzip` если пакет `file-roller` зависит от него? Если нет, то как это правильно сделать?
4. Используя `apt-file`, как вы можете узнать, какой пакет содержит файл `unrar`?
5. Используя `apt-cache`, какую команду нужно использовать для отображения информации о пакете `gimp`?

## Исследовательские упражнения <a href="#sec.102.4_01-ee" id="sec.102.4_01-ee"></a>

1. Рассмотрим репозиторий с исходными пакетами Debian для дистрибутива `xenial` на сервере `http://us.archive.ubuntu.com/ubuntu/` и с пакетами для компонента `universe` . Какую строку нужно добавить в `/etc/apt/sources.list`?
2. При компиляции программы вы столкнулись с сообщением об ошибке, в котором говорится, что в вашей системе отсутствует заголовочный файл `zzip-io.h`. Как узнать, какой пакет предоставляет этот файл?
3. Как можно проигнорировать предупреждение о зависимости и удалить пакет с помощью `dpkg`, даже если в системе есть пакеты, от которых он зависит?
4. Как вы можете получить дополнительную информацию о пакете под названием `midori` используя `apt`?
5. Перед установкой или обновлением пакетов с помощью `apt` какую команду следует использовать, чтобы убедиться, что индекс пакетов обновлен?

## Краткие сведения <a href="#sec.102.4_01-su" id="sec.102.4_01-su"></a>

На этом уроке вы узнали:

* Как использовать `dpkg` для установки и удаления пакетов.
* Как составить список установленных пакетов и их содержимого.
* Как перенастроить установленный пакет.
* Что такое `apt` и как устанавливать, обновлять и удалять пакеты с его помощью.
* Как использовать `apt-cache` для поиска пакетов.
* Как работает `/etc/apt/sources.list` .
* Как использовать `apt-file` для отображения содержимого пакета или как узнать, в каком пакете находится конкретный файл.

Были обсуждены следующие команды:

`dpkg -i`

Устанавливает один пакет или список пакетов, разделенных пробелами.

`dpkg -r`

Удаляет пакет или список пакетов, разделенных пробелом.

`dpkg -I`

Проверяет пакет, предоставляя подробную информацию о программном обеспечении, которое он устанавливает, и о необходимых зависимостях.

`dpkg --get-selections`

Перечисляются все пакеты, которые `dpkg` были установлены в системе.

`dpkg -L`

Выводит список всех файлов, которые устанавливает конкретный пакет.

`dpkg-query`

With a specified file name, this command will print out the package that installed the file.

`dpkg-reconfigure`

Эта команда повторно запустит сценарий _после установки_ пакетов, чтобы администратор мог внести изменения в конфигурацию установки пакета.

`apt-get update`

Эта команда обновит локальный индекс пакетов в соответствии с тем, что доступно в настроенных репозиториях в каталоге `/etc/apt/` .

`apt-get install`

Эта команда загрузит пакет из удалённого репозитория и установит его вместе с зависимостями. Её также можно использовать для установки уже загруженного пакета Debian.

`apt-get remove`

Эта команда удалит указанные пакеты из системы.

`apt-cache show`

Как и команда `dpkg -I`, эта команда может использоваться для отображения подробной информации о конкретном пакете.

`apt-cache search`

Эта команда выполнит поиск определенного пакета в вашей локальной кэшированной базе данных APT.

`apt-file update`

Эта команда обновит кэш пакетов, чтобы команда `apt-file` могла запрашивать его содержимое.

`apt-file search`

Эта команда ищет, в какой пакет включён файл. Возвращается список всех пакетов, содержащих шаблон.

`apt-file list`

Эта команда используется для вывода списка содержимого пакета, как и команда `dpkg -L` .

### Ответы на упражнения с Руководством <a href="#sec.102.4_01-age" id="sec.102.4_01-age"></a>

1.  Какая команда используется для установки пакета с именем `package.deb` используя `dpkg`?

    Передайте `-i` параметр в `dpkg`:

    ```bash
    # dpkg -i package.deb
    ```
2.  Используя `dpkg-query`, найдите, какой пакет содержит файл с именем `7zr.1.gz`.

    Добавьте `-S` параметр в `dpkg-query`:

    <pre class="language-bash"><code class="lang-bash"><strong># dpkg-query -S 7zr.1.gz
    </strong></code></pre>
3.  Можно ли удалить пакет под названием `unzip` из системы с помощью `dpkg -r unzip` если пакет `file-roller` зависит от него? Если нет, то как это правильно сделать?

    Нет. `dpkg` не устранит зависимости и не позволит вам удалить пакет, если от него зависит другой установленный пакет. В этом примере вы можете сначала удалить `file-roller` (при условии, что от него ничего не зависит), а затем удалить `unzip` или удалить оба пакета одновременно с помощью:

    ```bash
    # dpkg -r unzip file-roller
    ```
4.  Как узнать, в каком пакете находится файл `/usr/bin/unrar` с помощью утилиты `apt-file`?

    Используйте параметр `search`, за которым следует путь (или имя файла).:

    ```bash
    # apt-file search /usr/bin/unrar
    ```
5.  Используя `apt-cache`, какую команду нужно использовать для отображения информации о пакете `gimp`?

    Используйте параметр `show` , за которым следует название пакета:

    ```bash
    # apt-cache show gimp
    ```

### Ответы на исследовательские упражнения <a href="#sec.102.4_01-aee" id="sec.102.4_01-aee"></a>

1.  Рассмотрим репозиторий с исходными пакетами Debian для дистрибутива `xenial` на сервере `http://us.archive.ubuntu.com/ubuntu/` и с пакетами для компонента `universe` . Какую строку нужно добавить в `/etc/apt/sources.list`?

    Исходные пакеты имеют тип `deb-src`, поэтому строка должна быть:

    ```bash
    deb-src http://us.archive.ubuntu.com/ubuntu/ xenial universe
    ```

    Эту строку также можно добавить в файл `.list` в `/etc/apt/sources.list.d/`. Название на ваше усмотрение, но оно должно быть информативным, например `xenial_sources.list`.
2.  При компиляции программы вы столкнулись с сообщением об ошибке, в котором говорится, что в вашей системе отсутствует заголовочный файл `zzip-io.h`. Как узнать, какой пакет предоставляет этот файл?

    Используйте `apt-file search`, чтобы определить, какой пакет содержит файл, отсутствующий в системе:

    <pre class="language-basic"><code class="lang-basic"><strong># apt-file search zzip-io.h
    </strong></code></pre>
3.  Как можно проигнорировать предупреждение о зависимости и удалить пакет с помощью `dpkg`, даже если в системе есть пакеты, от которых он зависит?

    Параметр `--force` можно использовать, но _никогда_ не делайте этого, если не знаете точно, что делаете, так как существует большой риск того, что ваша система останется в несогласованном или «сломанном» состоянии.
4.  Как вы можете получить дополнительную информацию о пакете под названием `midori` using `apt-cache`?

    Используйте `apt-cache show` , за которым следует название пакета:

    ```bash
    # apt-cache show midori
    ```
5.  Перед установкой или обновлением пакетов с помощью `apt-get` какую команду следует использовать, чтобы убедиться, что индекс пакетов обновлен?

    `apt-get update` Следует использовать для загрузки последних индексов пакетов из репозиториев, описанных в файле `/etc/apt/sources.list` или в каталоге `/etc/apt/sources.list.d/`.
