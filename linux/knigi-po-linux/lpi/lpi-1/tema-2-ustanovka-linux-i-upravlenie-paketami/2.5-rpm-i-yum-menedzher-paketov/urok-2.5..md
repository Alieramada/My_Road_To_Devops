# Урок 2.5.

## Введение <a href="#sec.102.5_01-in" id="sec.102.5_01-in"></a>

Давным-давно, когда Linux только зарождался, самым распространённым способом распространения программного обеспечения был сжатый файл (обычно в виде архива `.tar.gz` с исходным кодом), который нужно было распаковать и скомпилировать самостоятельно.

Однако по мере роста количества и сложности программного обеспечения стала очевидной необходимость в способе распространения предварительно скомпилированного программного обеспечения. В конце концов, не у всех были ресурсы, как временные, так и вычислительные, для компиляции крупных проектов, таких как ядро Linux или X-сервер.

Вскоре усилия по стандартизации способов распространения этих программных «пакетов» усилились, и появились первые менеджеры пакетов. Эти инструменты значительно упростили установку, настройку и удаление программного обеспечения из системы.

Одним из них был _менеджер пакетов RPM_ и соответствующий инструмент (`rpm`), разработанный компанией Red Hat. Сегодня они широко используются не только в Red Hat Enterprise Linux (RHEL), но и в его производных, таких как Fedora, CentOS и Oracle Linux, в других дистрибутивах, таких как openSUSE, и даже в других операционных системах, таких как IBM AIX.

Другие инструменты управления пакетами, популярные в дистрибутивах, совместимых с Red Hat, — это `yum` (YellowDog Updater Modified), `dnf` (Dandified YUM) и `zypper`, которые могут упростить многие аспекты установки, обслуживания и удаления пакетов, значительно облегчая управление пакетами.

В этом уроке мы узнаем, как использовать `rpm`, `yum`, `dnf` и `zypper` для получения, установки, управления и удаления программного обеспечения в системе Linux.

{% hint style="info" %}
Несмотря на использование одного и того же формата пакетов, между дистрибутивами существуют внутренние различия, поэтому пакет, созданный специально для openSUSE, может не работать в системе RHEL, и наоборот. При поиске пакетов всегда проверяйте их совместимость и по возможности старайтесь найти пакет, созданный специально для вашего дистрибутива.
{% endhint %}

## Менеджер пакетов RPM (rpm) <a href="#the_rpm_package_manager_rpm" id="the_rpm_package_manager_rpm"></a>

Менеджер пакетов RPM (`rpm`) — это важный инструмент для управления пакетами программного обеспечения в системах на базе Red Hat (или производных от неё).

## **Установка, обновление и удаление пакетов**

Самая простая операция — установка пакета, которую можно выполнить с помощью:

```bash
# rpm -i PACKAGENAME
```

Где `PACKAGENAME` - это название `.rpm` пакета, который вы хотите установить.

Если в системе есть предыдущая версия пакета, вы можете обновить её до более новой версии с помощью параметра `-U`:

```bash
# rpm -U PACKAGENAME
```

Если предыдущая версия `PACKAGENAME` не установлена, будет установлена новая копия. Чтобы избежать этого и _только_ обновить _установленный_ пакет, используйте опцию `-F`.

В обеих операциях вы можете добавить параметр `-v` для получения подробного вывода (во время установки отображается больше информации) и `-h` для получения знаков решетки (`#`) в качестве визуального индикатора хода установки. Несколько параметров можно объединить в один, поэтому `rpm -i -v -h` равно `rpm -ivh`.

Чтобы удалить установленный пакет, передайте параметр `-e` (например, «erase») в `rpm` и укажите имя пакета, который вы хотите удалить:

```bash
# rpm -e wget
```

Если установленный пакет зависит от удаляемого пакета, вы получите сообщение об ошибке:

```bash
# rpm -e unzip
error: Failed dependencies:
 /usr/bin/unzip is needed by (installed) file-roller-3.28.1-2.el7.x86_64
```

Чтобы завершить операцию, сначала вам нужно будет удалить пакеты, которые зависят от того, который вы хотите удалить (в приведённом выше примере — `file-roller`). Вы можете передать `rpm -e` несколько имён пакетов, чтобы удалить сразу несколько пакетов.

## **Работа с зависимостями**

Чаще всего работа пакета по назначению зависит от других пакетов. Например, для открытия файлов JPG редактору изображений могут понадобиться библиотеки, а для пользовательского интерфейса утилиты может понадобиться набор виджетов, такой как Qt или GTK.

`rpm` программа проверит, установлены ли эти зависимости в вашей системе, и не сможет установить пакет, если они не установлены. В этом случае `rpm` будет указан список недостающих зависимостей. Однако программа _не может_ самостоятельно решить проблему зависимостей.

В приведённом ниже примере пользователь попытался установить пакет для редактора изображений GIMP, но ему не хватало некоторых зависимостей:

```bash
# rpm -i gimp-2.8.22-1.el7.x86_64.rpm
error: Failed dependencies:
	babl(x86-64) >= 0.1.10 is needed by gimp-2:2.8.22-1.el7.x86_64
	gegl(x86-64) >= 0.2.0 is needed by gimp-2:2.8.22-1.el7.x86_64
	gimp-libs(x86-64) = 2:2.8.22-1.el7 is needed by gimp-2:2.8.22-1.el7.x86_64
	libbabl-0.1.so.0()(64bit) is needed by gimp-2:2.8.22-1.el7.x86_64
	libgegl-0.2.so.0()(64bit) is needed by gimp-2:2.8.22-1.el7.x86_64
	libgimp-2.0.so.0()(64bit) is needed by gimp-2:2.8.22-1.el7.x86_64
	libgimpbase-2.0.so.0()(64bit) is needed by gimp-2:2.8.22-1.el7.x86_64
	libgimpcolor-2.0.so.0()(64bit) is needed by gimp-2:2.8.22-1.el7.x86_64
	libgimpconfig-2.0.so.0()(64bit) is needed by gimp-2:2.8.22-1.el7.x86_64
	libgimpmath-2.0.so.0()(64bit) is needed by gimp-2:2.8.22-1.el7.x86_64
	libgimpmodule-2.0.so.0()(64bit) is needed by gimp-2:2.8.22-1.el7.x86_64
	libgimpthumb-2.0.so.0()(64bit) is needed by gimp-2:2.8.22-1.el7.x86_64
	libgimpui-2.0.so.0()(64bit) is needed by gimp-2:2.8.22-1.el7.x86_64
	libgimpwidgets-2.0.so.0()(64bit) is needed by gimp-2:2.8.22-1.el7.x86_64
	libmng.so.1()(64bit) is needed by gimp-2:2.8.22-1.el7.x86_64
	libwmf-0.2.so.7()(64bit) is needed by gimp-2:2.8.22-1.el7.x86_64
	libwmflite-0.2.so.7()(64bit) is needed by gimp-2:2.8.22-1.el7.x86_64
```

Пользователь должен найти пакеты `.rpm` с соответствующими зависимостями и установить их. В менеджерах пакетов, таких как `yum`, `zypper` и `dnf`, есть инструменты, которые могут определить, какой пакет предоставляет конкретный файл. Они будут рассмотрены далее в этом уроке.

## **Список установленных пакетов**

Чтобы получить список всех установленных в вашей системе пакетов, используйте `rpm -qa` (подумайте о «запросе всех»).

```bash
# rpm -qa
selinux-policy-3.13.1-229.el7.noarch
pciutils-libs-3.5.1-3.el7.x86_64
redhat-menus-12.0.2-8.el7.noarch
grubby-8.28-25.el7.x86_64
hunspell-en-0.20121024-6.el7.noarch
dejavu-fonts-common-2.33-6.el7.noarch
xorg-x11-drv-dummy-0.3.7-1.el7.1.x86_64
libevdev-1.5.6-1.el7.x86_64
[...]
```

## **Получение информации о пакете**

Чтобы получить информацию об _установленном_ пакете, такую как номер его версии, архитектура, дата установки, разработчик, сводка и т. д., используйте `rpm` с параметрами `-qi` (подумайте о «запросе информации»), а затем укажите имя пакета. Например:

```bash
# rpm -qi unzip
Name        : unzip
Version     : 6.0
Release     : 19.el7
Architecture: x86_64
Install Date: Sun 25 Aug 2019 05:14:39 PM EDT
Group       : Applications/Archiving
Size        : 373986
License     : BSD
Signature   : RSA/SHA256, Wed 25 Apr 2018 07:50:02 AM EDT, Key ID 24c6a8a7f4a80eb5
Source RPM  : unzip-6.0-19.el7.src.rpm
Build Date  : Wed 11 Apr 2018 01:24:53 AM EDT
Build Host  : x86-01.bsys.centos.org
Relocations : (not relocatable)
Packager    : CentOS BuildSystem <http://bugs.centos.org>
Vendor      : CentOS
URL         : http://www.info-zip.org/UnZip.html
Summary     : A utility for unpacking zip files
Description :
The unzip utility is used to list, test, or extract files from a zip
archive. Zip archives are commonly found on MS-DOS systems. The zip
utility, included in the zip package, creates zip archives. Zip and
unzip are both compatible with archives created by PKWARE(R)'s PKZIP
for MS-DOS, but the programs' options and default behaviors do differ
in some respects.

Install the unzip package if you need to list, test or extract files from
a zip archive.
```

Чтобы получить список файлов, входящих в _установленный_ пакет, используйте параметры `-ql` (подумайте о «списке запросов»), а затем укажите название пакета:

```bash
# rpm -ql unzip
/usr/bin/funzip
/usr/bin/unzip
/usr/bin/unzipsfx
/usr/bin/zipgrep
/usr/bin/zipinfo
/usr/share/doc/unzip-6.0
/usr/share/doc/unzip-6.0/BUGS
/usr/share/doc/unzip-6.0/LICENSE
/usr/share/doc/unzip-6.0/README
/usr/share/man/man1/funzip.1.gz
/usr/share/man/man1/unzip.1.gz
/usr/share/man/man1/unzipsfx.1.gz
/usr/share/man/man1/zipgrep.1.gz
/usr/share/man/man1/zipinfo.1.gz
```

Если вы хотите получить информацию или список файлов из пакета, который ещё _не_ установлен, просто добавьте параметр `-p` к приведённым выше командам, а затем укажите имя файла RPM (`FILENAME`). Таким образом, `rpm -qi PACKAGENAME` становится `rpm -qip FILENAME`, а `rpm -ql PACKAGENAME` становится `rpm -qlp FILENAME`, как показано ниже.

```bash
# rpm -qip atom.x86_64.rpm
Name        : atom
Version     : 1.40.0
Release     : 0.1
Architecture: x86_64
Install Date: (not installed)
Group       : Unspecified
Size        : 570783704
License     : MIT
Signature   : (none)
Source RPM  : atom-1.40.0-0.1.src.rpm
Build Date  : sex 09 ago 2019 12:36:31 -03
Build Host  : b01bbeaf3a88
Relocations : /usr
URL         : https://atom.io/
Summary     : A hackable text editor for the 21st Century.
Description :
A hackable text editor for the 21st Century.
```

```bash
# rpm -qlp atom.x86_64.rpm
/usr/bin/apm
/usr/bin/atom
/usr/share/applications/atom.desktop
/usr/share/atom
/usr/share/atom/LICENSE
/usr/share/atom/LICENSES.chromium.html
/usr/share/atom/atom
/usr/share/atom/atom.png
/usr/share/atom/blink_image_resources_200_percent.pak
/usr/share/atom/content_resources_200_percent.pak
/usr/share/atom/content_shell.pak

(listing goes on)
```

## **Выяснение, какому пакету принадлежит определенный файл**

Чтобы узнать, какому установленному пакету принадлежит файл, используйте `-qf` (подумайте о «запросе файла»), а затем укажите полный путь к файлу:

```bash
# rpm -qf /usr/bin/unzip
unzip-6.0-19.el7.x86_64
```

В приведенном выше примере файл `/usr/bin/unzip` принадлежит `unzip-6.0-19.el7.x86_64` пакету.

## YellowDog Updater Modified (YUM) <a href="#yellowdog_updater_modified_yum" id="yellowdog_updater_modified_yum"></a>

`yum` Изначально он был разработан как _Yellow Dog Updater_ (YUP) — инструмент для управления пакетами в дистрибутиве Yellow Dog Linux. Со временем он стал использоваться для управления пакетами в других системах на основе RPM, таких как Fedora, CentOS, Red Hat Enterprise Linux и Oracle Linux.

По функциональным возможностям он похож на утилиту `apt` в системах на базе Debian, позволяя искать, устанавливать, обновлять и удалять пакеты, а также автоматически обрабатывать зависимости. `yum` можно использовать для установки одного пакета или для обновления всей системы сразу.

## **Поиск пакетов**

Чтобы установить пакет, вам нужно знать его название. Для этого вы можете выполнить поиск с помощью `yum search PATTERN`, где `PATTERN` — название пакета, который вы ищете. Результатом будет список пакетов, название или описание которых содержат указанный шаблон поиска. Например, если вам нужна утилита для работы со сжатыми файлами 7Zip (с расширением `.7z`), вы можете использовать:

```bash
# yum search 7zip
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.ufscar.br
 * epel: mirror.globo.com
 * extras: mirror.ufscar.br
 * updates: mirror.ufscar.br
=========================== N/S matchyutr54ed: 7zip ============================
p7zip-plugins.x86_64 : Additional plugins for p7zip
p7zip.x86_64 : Very high compression ratio file archiver
p7zip-doc.noarch : Manual documentation and contrib directory
p7zip-gui.x86_64 : 7zG - 7-Zip GUI version

  Name and summary matches only, use "search all" for everything.
```

## **Установка, обновление и удаление пакетов**

Чтобы установить пакет с помощью `yum`, используйте команду `yum install PACKAGENAME`, где `PACKAGENAME` — это название пакета. `yum` загрузит пакет и соответствующие зависимости из онлайн-репозитория и установит всё в вашей системе.

```bash
# yum install p7zip
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.ufscar.br
 * epel: mirror.globo.com
 * extras: mirror.ufscar.br
 * updates: mirror.ufscar.br
Resolving Dependencies
--> Running transaction check
---> Package p7zip.x86_64 0:16.02-10.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==========================================================================
 Package        Arch            Version               Repository     Size
==========================================================================
Installing:
 p7zip          x86_64          16.02-10.el7          epel          604 k

Transaction Summary
==========================================================================
Install  1 Package

Total download size: 604 k
Installed size: 1.7 M
Is this ok [y/d/N]:
```

Чтобы обновить установленный пакет, используйте `yum update PACKAGENAME`, где `PACKAGENAME` — это имя пакета, который вы хотите обновить. Например:

```bash
# yum update wget
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.ufscar.br
 * epel: mirror.globo.com
 * extras: mirror.ufscar.br
 * updates: mirror.ufscar.br
Resolving Dependencies
--> Running transaction check
---> Package wget.x86_64 0:1.14-18.el7 will be updated
---> Package wget.x86_64 0:1.14-18.el7_6.1 will be an update
--> Finished Dependency Resolution

Dependencies Resolved

==========================================================================
 Package     Arch          Version                   Repository      Size
==========================================================================
Updating:
 wget        x86_64        1.14-18.el7_6.1           updates        547 k

Transaction Summary
==========================================================================
Upgrade  1 Package

Total download size: 547 k
Is this ok [y/d/N]:
```

Если вы не укажете название пакета, вы сможете обновить все пакеты в системе, для которых доступно обновление.

Чтобы проверить, доступно ли обновление для конкретного пакета, используйте `yum check-update PACKAGENAME`. Как и раньше, если вы не укажете название пакета, `yum` проверит наличие обновлений для всех установленных пакетов в системе.

Чтобы удалить установленный пакет, используйте `yum remove PACKAGENAME`, где `PACKAGENAME` — это название пакета, который вы хотите удалить.

## **Определение того, какой пакет предоставляет конкретный файл**

В предыдущем примере мы показали попытку установить редактор изображений `gimp`, которая не удалась из-за неудовлетворённых зависимостей. Однако `rpm` показывает, каких файлов не хватает, но не перечисляет названия пакетов, которые их предоставляют.

Например, одной из отсутствующих зависимостей была `libgimpui-2.0.so.0`. Чтобы узнать, какой пакет предоставляет её, вы можете использовать `yum whatprovides`, а затем указать имя файла, который вы ищете:

```bash
# yum whatprovides libgimpui-2.0.so.0
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.ufscar.br
 * epel: mirror.globo.com
 * extras: mirror.ufscar.br
 * updates: mirror.ufscar.br
2:gimp-libs-2.8.22-1.el7.i686 : GIMP libraries
Repo        : base
Matched from:
Provides    : libgimpui-2.0.so.0
```

Ответ — `gimp-libs-2.8.22-1.el7.i686`. Затем вы можете установить пакет с помощью команды `yum install gimp-libs`.

Это также работает для файлов, уже имеющихся в вашей системе. Например, если вы хотите узнать, откуда взялся файл `/etc/hosts`, вы можете использовать:

```bash
# yum whatprovides /etc/hosts
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.ufscar.br
 * epel: mirror.globo.com
 * extras: mirror.ufscar.br
 * updates: mirror.ufscar.br
setup-2.8.71-10.el7.noarch : A set of system configuration and setup files
Repo        : base
Matched from:
Filename    : /etc/hosts
```

Ответ  `setup-2.8.71-10.el7.noarch`.

## **Получение информации о посылке**

Чтобы получить информацию о пакете, например о его версии, архитектуре, описании, размере и т. д., используйте `yum info PACKAGENAME` где `PACKAGENAME` — это название пакета, информацию о котором вы хотите получить:

```bash
# yum info firefox
Last metadata expiration check: 0:24:16 ago on Sat 21 Sep 2019 02:39:43 PM -03.
Installed Packages
Name         : firefox
Version      : 69.0.1
Release      : 3.fc30
Architecture : x86_64
Size         : 268 M
Source       : firefox-69.0.1-3.fc30.src.rpm
Repository   : @System
From repo    : updates
Summary      : Mozilla Firefox Web browser
URL          : https://www.mozilla.org/firefox/
License      : MPLv1.1 or GPLv2+ or LGPLv2+
Description  : Mozilla Firefox is an open-source web browser, designed
             : for standards compliance, performance and portability.
```

## **Управление репозиториями программного обеспечения**

Для `yum` «репозитории» перечислены в каталоге `/etc/yum.repos.d/`. Каждый репозиторий представлен файлом `.repo` типа `CentOS-Base.repo`.

Дополнительные репозитории могут быть добавлены пользователем путём добавления файла `.repo` в каталог, упомянутый выше, или в конец `/etc/yum.conf`. Однако рекомендуется добавлять репозитории или управлять ими с помощью инструмента `yum-config-manager`.

Чтобы добавить репозиторий, используйте параметр `--add-repo`, за которым следует URL-адрес файла `.repo` .

```bash
# yum-config-manager --add-repo https://rpms.remirepo.net/enterprise/remi.repo
Loaded plugins: fastestmirror, langpacks
adding repo from: https://rpms.remirepo.net/enterprise/remi.repo
grabbing file https://rpms.remirepo.net/enterprise/remi.repo to /etc/yum.repos.d/remi.repo
repo saved to /etc/yum.repos.d/remi.repo
```

Чтобы получить список всех доступных репозиториев, используйте `yum repolist all`. Вы получите примерно такой результат:

```bash
# yum repolist all
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.ufscar.br
 * epel: mirror.globo.com
 * extras: mirror.ufscar.br
 * updates: mirror.ufscar.br
repo id                       repo name                    status
updates/7/x86_64              CentOS-7 - Updates           enabled:  2,500
updates-source/7              CentOS-7 - Updates Sources   disabled
```

`disabled` Репозитории будут игнорироваться при установке или обновлении программного обеспечения. Чтобы включить или отключить репозиторий, используйте утилиту `yum-config-manager` с указанием идентификатора репозитория.

В приведенном выше выводе идентификатор репозитория отображается в первом столбце (`repo id`) каждой строки. Используйте только часть до первого `/`, поэтому идентификатор репозитория `CentOS-7 - Updates` — `updates`, а не `updates/7/x86_64`.

```bash
# yum-config-manager --disable updates
```

Приведённая выше команда отключит репозиторий `updates` . Чтобы снова включить его, используйте:

```bash
# yum-config-manager --enable updates
```

{% hint style="info" %}
Yum хранит загруженные пакеты и связанные с ними метаданные в каталоге кэша (обычно `/var/cache/yum`). По мере обновления системы и установки новых пакетов этот кэш может стать довольно большим. Чтобы очистить кэш и освободить место на диске, вы можете использовать команду `yum clean` с указанием того, что нужно очистить. Наиболее полезными параметрами являются `packages` (`yum clean packages`) для удаления загруженных пакетов и `metadata` (`yum clean metadata`) для удаления связанных метаданных. Дополнительные сведения см. в справочной странице `yum` (введите `man yum`).
{% endhint %}

## DNF <a href="#dnf" id="dnf"></a>

`dnf` Это инструмент управления пакетами, используемый в Fedora, и он является ответвлением `yum`. Таким образом, многие команды и параметры схожи. В этом разделе вы получите краткий обзор `dnf`.

Поиск пакетов

`dnf search PATTERN`, где `PATTERN` — это то, что вы ищете. Например, `dnf search unzip` покажет все пакеты, в названии или описании которых есть слово `unzip`.

Получение информации о пакете

`dnf info PACKAGENAME`

Установка пакетов

`dnf install PACKAGENAME`, где `PACKAGENAME` — это название пакета, который вы хотите установить. Вы можете найти название, выполнив поиск.

Удаление пакетов

`dnf remove PACKAGENAME`

Обновление пакетов

`dnf upgrade PACKAGENAME` чтобы обновить только один пакет. Чтобы обновить все пакеты в системе, не указывайте название пакета.

Выяснение, какой пакет предоставляет конкретный файл

`dnf provides FILENAME`

Получение списка всех пакетов, установленных в системе

`dnf list --installed`

Перечисление содержимого пакета

`dnf repoquery -l PACKAGENAME`\


{% hint style="info" %}
`dnf` Встроена система помощи, которая показывает дополнительную информацию (например, дополнительные параметры) для каждой команды. Чтобы воспользоваться ею, введите `dnf help` и команду, например `dnf help install`.
{% endhint %}

## **Управление репозиториями программного обеспечения**

Как и в случае с `yum` и `zypper`, `dnf` работает с репозиториями программного обеспечения (repos). В каждом дистрибутиве есть список репозиториев по умолчанию, и администраторы могут добавлять или удалять репозитории по мере необходимости.

Чтобы получить список всех доступных репозиториев, используйте `dnf repolist`. Чтобы отобразить только активные репозитории, добавьте параметр `--enabled`, а чтобы отобразить только неактивные репозитории, добавьте параметр `--disabled`.

```bash
# dnf repolist
Last metadata expiration check: 0:20:09 ago on Sat 21 Sep 2019 02:39:43 PM -03.
repo id                    repo name                                      status
*fedora                    Fedora 30 - x86_64                             56,582
*fedora-modular            Fedora Modular 30 - x86_64                        135
*updates                   Fedora 30 - x86_64 - Updates                   12,774
*updates-modular           Fedora Modular 30 - x86_64 - Updates              145
```

Чтобы добавить репозиторий, используйте `dnf config-manager --add_repo URL`, где `URL` — это полный URL-адрес репозитория. Чтобы включить репозиторий, используйте `dnf config-manager --set-enabled REPO_ID`.

Аналогичным образом, чтобы отключить репозиторий, используйте `dnf config-manager --set-disabled REPO_ID`. В обоих случаях `REPO_ID` — это уникальный идентификатор репозитория, который можно получить с помощью `dnf repolist`. Добавленные репозитории включены по умолчанию.

Репозитории хранятся в файлах `.repo` в каталоге `/etc/yum.repos.d/` с использованием того же синтаксиса, что и для `yum`.

## Zypper <a href="#zypper" id="zypper"></a>

`zypper` Это инструмент управления пакетами, используемый в SUSE Linux и OpenSUSE. По своим функциям он похож на `apt` и `yum`, позволяя устанавливать, обновлять и удалять пакеты из системы с автоматическим разрешением зависимостей.

## **Обновление индекса пакета**

Как и другие инструменты управления пакетами, `zypper` работает с репозиториями, содержащими пакеты и метаданные. Эти метаданные необходимо время от времени обновлять, чтобы утилита знала о доступных последних пакетах. Чтобы выполнить обновление, просто введите:

```bash
# zypper refresh
Repository 'Non-OSS Repository' is up to date.
Repository 'Main Repository' is up to date.
Repository 'Main Update Repository' is up to date.
Repository 'Update Repository (Non-Oss)' is up to date.
All repositories have been refreshed.
```

В нём есть функция автоматического обновления, которую можно включить для каждого репозитория. Это означает, что некоторые репозитории могут обновляться автоматически перед выполнением запроса или установкой пакета, а другие могут нуждаться в обновлении вручную. Вскоре вы узнаете, как управлять этой функцией.

## **Поиск пакетов**

Чтобы найти пакет, используйте оператор `search` (или `se`) и укажите название пакета:

```bash
# zypper se gnumeric
Loading repository data...
Reading installed packages...

S | Name           | Summary                           | Type
--+----------------+-----------------------------------+--------
  | gnumeric       | Spreadsheet Application           | package
  | gnumeric-devel | Spreadsheet Application           | package
  | gnumeric-doc   | Documentation files for Gnumeric  | package
  | gnumeric-lang  | Translations for package gnumeric | package
```

Оператор поиска также можно использовать для получения списка всех установленных в системе пакетов. Для этого используйте параметр `-i` без имени пакета, например `zypper se -i`.

Чтобы узнать, установлен ли конкретный пакет, добавьте его название в приведённую выше команду. Например, следующая команда выполнит поиск среди установленных пакетов всех, в названии которых есть «firefox»:

```bash
# zypper se -i firefox
Loading repository data...
Reading installed packages...

S | Name                               | Summary                 | Type
--+------------------------------------+-------------------------+--------
i | MozillaFirefox                     | Mozilla Firefox Web B-> | package
i | MozillaFirefox-branding-openSUSE   | openSUSE branding of -> | package
i | MozillaFirefox-translations-common | Common translations f-> | package
```

Чтобы искать только среди _неустановленных_ пакетов, добавьте параметр `-u` к оператору `se`.

## **Установка, обновление и удаление пакетов**

```bash
# zypper in unrar
zypper in unrar
Loading repository data...
Reading installed packages...
Resolving package dependencies...

The following NEW package is going to be installed:
  unrar

1 new package to install.
Overall download size: 141.2 KiB. Already cached: 0 B. After the operation, additional 301.6 KiB will be used.
Continue? [y/n/v/...? shows all options] (y): y
Retrieving package unrar-5.7.5-lp151.1.1.x86_64
                                     (1/1), 141.2 KiB (301.6 KiB unpacked)
Retrieving: unrar-5.7.5-lp151.1.1.x86_64.rpm .......................[done]
Checking for file conflicts: .......................................[done]
(1/1) Installing: unrar-5.7.5-lp151.1.1.x86_64 .....................[done]
```

`zypper` Также можно использовать для установки пакета RPM на диск, пытаясь удовлетворить его зависимости с помощью пакетов из репозиториев. Для этого просто укажите полный путь к пакету вместо его имени, например `zypper in /home/john/newpackage.rpm`.

Чтобы обновить пакеты, установленные в системе, используйте `zypper update`. Как и в процессе установки, будет показан список пакетов, которые необходимо установить/обновить, прежде чем вы сможете продолжить.

Если вы хотите только просмотреть доступные обновления, ничего не устанавливая, вы можете использовать `zypper list-updates`.

Чтобы удалить пакет, используйте оператор `remove` (или `rm`) и укажите имя пакета:

```bash
# zypper rm unrar
Loading repository data...
Reading installed packages...
Resolving package dependencies...

The following package is going to be REMOVED:
  unrar

1 package to remove.
After the operation, 301.6 KiB will be freed.
Continue? [y/n/v/...? shows all options] (y): y
(1/1) Removing unrar-5.7.5-lp151.1.1.x86_64 ........................[done]
```

Имейте в виду, что при удалении пакета удаляются и все другие пакеты, от которых он зависит. Например:

```bash
# zypper rm libgimp-2_0-0
Loading repository data...
Warning: No repositories defined. Operating only with the installed resolvables. Nothing can be installed.
Reading installed packages...
Resolving package dependencies...

The following 6 packages are going to be REMOVED:
  gimp gimp-help gimp-lang gimp-plugins-python libgimp-2_0-0
  libgimpui-2_0-0

6 packages to remove.
After the operation, 98.0 MiB will be freed.
Continue? [y/n/v/...? shows all options] (y):
```

## **Определение того, какие пакеты содержат определенный файл**

Чтобы узнать, какие пакеты содержат определённый файл, используйте оператор поиска, за которым следует параметр `--provides` и имя файла (или полный путь к нему). Например, если вы хотите узнать, какие пакеты содержат файл `libgimpmodule-2.0.so.0` в `/usr/lib64/`, вы можете использовать:

```bash
# zypper se --provides /usr/lib64/libgimpmodule-2.0.so.0
Loading repository data...
Reading installed packages...

S | Name          | Summary                                      | Type
--+---------------+----------------------------------------------+--------
i | libgimp-2_0-0 | The GNU Image Manipulation Program - Libra-> | package
```

## **Получение информации о пакете**

Чтобы просмотреть метаданные, связанные с пакетом, используйте оператор `info` и укажите имя пакета. Это позволит вам узнать исходный репозиторий, имя пакета, версию, архитектуру, производителя, размер после установки, если пакет установлен, статус (если он обновлен), исходный пакет и описание.

```bash
# zypper info gimp
Loading repository data...
Reading installed packages...

Information for package gimp:
 -----------------------------
Repository     : Main Repository
Name           : gimp
Version        : 2.8.22-lp151.4.6
Arch           : x86_64
Vendor         : openSUSE
Installed Size : 29.1 MiB
Installed      : Yes (automatically)
Status         : up-to-date
Source package : gimp-2.8.22-lp151.4.6.src
Summary        : The GNU Image Manipulation Program
Description    :
    The GIMP is an image composition and editing program, which can be
    used for creating logos and other graphics for Web pages. The GIMP
    offers many tools and filters, and provides a large image
    manipulation toolbox, including channel operations and layers,
    effects, subpixel imaging and antialiasing, and conversions, together
    with multilevel undo. The GIMP offers a scripting facility, but many
    of the included scripts rely on fonts that we cannot distribute.
```

## **Управление репозиториями программного обеспечения**

`zypper` Также можно использовать для управления репозиториями программного обеспечения. Чтобы просмотреть список всех репозиториев, зарегистрированных в вашей системе, используйте `zypper repos`:

```bash
# zypper repos
Repository priorities are without effect. All enabled repositories share the same priority.

#  | Alias                     | Name                               | Enabled | GPG Check | Refresh
---+---------------------------+------------------------------------+---------+-----------+--------
 1 | openSUSE-Leap-15.1-1      | openSUSE-Leap-15.1-1               | No      | ----      | ----
 2 | repo-debug                | Debug Repository                   | No      | ----      | ----
 3 | repo-debug-non-oss        | Debug Repository (Non-OSS)         | No      | ----      | ----
 4 | repo-debug-update         | Update Repository (Debug)          | No      | ----      | ----
 5 | repo-debug-update-non-oss | Update Repository (Debug, Non-OSS) | No      | ----      | ----
 6 | repo-non-oss              | Non-OSS Repository                 | Yes     | (r ) Yes  | Yes
 7 | repo-oss                  | Main Repository                    | Yes     | (r ) Yes  | Yes
 8 | repo-source               | Source Repository                  | No      | ----      | ----
 9 | repo-source-non-oss       | Source Repository (Non-OSS)        | No      | ----      | ----
10 | repo-update               | Main Update Repository             | Yes     | (r ) Yes  | Yes
11 | repo-update-non-oss       | Update Repository (Non-Oss)        | Yes     | (r ) Yes  | Yes
```

В столбце `Enabled` видно, что некоторые репозитории включены, а другие — нет. Вы можете изменить это с помощью оператора `modifyrepo`, за которым следует параметр `-e` (включить) или `-d` (отключить) и псевдоним репозитория (второй столбец в выводе выше).

```bash
# zypper modifyrepo -F repo-non-oss
Autorefresh has been disabled for repository 'repo-non-oss'.

# zypper modifyrepo -f repo-non-oss
Autorefresh has been enabled for repository 'repo-non-oss'.
```

Ранее мы упоминали, что `zypper` имеет функцию _автоматического обновления_, которую можно включить для каждого репозитория. Если этот флаг включен, `zypper` выполнит операцию обновления (аналогичную выполнению `zypper refresh`) перед работой с указанным репозиторием. Это можно контролировать с помощью параметров `-f` и `-F` оператора `modifyrepo`:

```bash
# zypper modifyrepo -F repo-non-oss
Autorefresh has been disabled for repository 'repo-non-oss'.

# zypper modifyrepo -f repo-non-oss
Autorefresh has been enabled for repository 'repo-non-oss'.
```

## **Добавление и удаление репозиториев**

Чтобы добавить новый репозиторий программного обеспечения для `zypper`, используйте оператор `addrepo` с указанием URL-адреса репозитория и его названия, как показано ниже:

```bash
# zypper addrepo http://packman.inode.at/suse/openSUSE_Leap_15.1/ packman
Adding repository 'packman' ........................................[done]
Repository 'packman' successfully added

URI         : http://packman.inode.at/suse/openSUSE_Leap_15.1/
Enabled     : Yes
GPG Check   : Yes
Autorefresh : No
Priority    : 99 (default priority)

Repository priorities are without effect. All enabled repositories share the same priority.
```

При добавлении репозитория вы можете включить автоматическое обновление с помощью параметра `-f` . Добавленные репозитории включены по умолчанию, но вы можете добавить и отключить репозиторий одновременно с помощью параметра `-d` .

Чтобы удалить репозиторий, используйте оператор `removerepo` и укажите имя репозитория (псевдоним). Чтобы удалить репозиторий, добавленный в примере выше, используйте следующую команду:

```bash
# zypper removerepo packman
Removing repository 'packman' ......................................[done]
Repository 'packman' has been removed.
```

## Упражнения с руководством <a href="#sec.102.5_01-ge" id="sec.102.5_01-ge"></a>

1. Как установить пакет `rpm` в системе Red Hat Enterprise Linux, используя `file-roller-3.28.1-2.el7.x86_64.rpm` и отображая индикатор выполнения во время установки?
2. Используя `rpm`, выясните, какой пакет содержит файл `/etc/redhat-release`.
3. Как бы вы использовали `yum` для проверки обновлений всех пакетов в системе?
4. Используя `zypper`, как бы вы отключили репозиторий под названием repo-extras?
5. Если у вас есть файл `.repo` с описанием нового репозитория, куда следует поместить этот файл, чтобы он распознавался DNF?

## Исследовательские упражнения <a href="#sec.102.5_01-ee" id="sec.102.5_01-ee"></a>

1. Как бы вы использовали `zypper`, чтобы узнать, какому пакету принадлежит файл `/usr/sbin/swapon`?
2. Как получить список всех установленных в системе пакетов с помощью `dnf`?
3. С помощью `dnf` какая команда используется для добавления репозитория, расположенного по адресу `https://www.example.url/home:reponame.repo`, в систему?
4. Как вы можете использовать `zypper`, чтобы проверить, установлен ли пакет `unzip`?
5. Используя `yum`, выясните, какой пакет предоставляет файл `/bin/wget`.

## Краткие сведения <a href="#sec.102.5_01-su" id="sec.102.5_01-su"></a>

На этом уроке вы узнали:

* Как использовать `rpm` для установки, обновления и удаления пакетов.
* Как использовать `yum`, `zypper` и `dnf`.
* Как получить информацию о пакете.
* Как получить список содержимого упаковки.
* Как узнать, из какого пакета пришел файл.
* Как составлять список, добавлять, удалять, включать или отключать репозитории программного обеспечения.

Были обсуждены следующие команды:

* `rpm`
* `yum`
* `dnf`
* `zypper`

### Ответы на упражнения с руководством <a href="#sec.102.5_01-age" id="sec.102.5_01-age"></a>

1.  Как установить пакет `rpm` в системе Red Hat Enterprise Linux, используя `file-roller-3.28.1-2.el7.x86_64.rpm` и отображая индикатор выполнения во время установки?

    Используйте параметр `-i` для установки пакета и опцию `-h` для включения «хэш-меток», показывающих ход установки. Итак, ответ: `rpm -ih file-roller-3.28.1-2.el7.x86_64.rpm`.
2.  Используя `rpm`, выясните, какой пакет содержит файл `/etc/redhat-release`.

    Вы запрашиваете информацию о файле, поэтому используйте параметр `-qf`: `rpm -qf /etc/redhat-release`.
3.  Как бы вы использовали `yum` для проверки обновлений всех пакетов в системе?

    Используйте `check-update` операцию _без_ имени пакета: `yum check-update`.
4.  Используя `zypper`, как бы вы отключили репозиторий под названием repo-extras?

    Используйте операцию `modifyrepo` для изменения параметров репозитория и параметр `-d` для его отключения: `zypper modifyrepo -d repo-extras`.
5.  Если у вас есть файл `.repo` с описанием нового репозитория, куда следует поместить этот файл, чтобы он распознавался DNF?

    `.repo` файлы для DNF должны быть помещены в то же место, что и файлы YUM, внутри `/etc/yum.repos.d/`.

### Ответы на исследовательские упражнения <a href="#sec.102.5_01-aee" id="sec.102.5_01-aee"></a>

1.  Как бы вы использовали `zypper`, чтобы узнать, какому пакету принадлежит файл `/usr/sbin/swapon`?

    Используйте оператор `se` (search) и `--provides` параметр: `zypper se --provides /usr/sbin/swapon`.
2.  Как получить список всех установленных в системе пакетов с помощью `dnf`?

    Используйте `list` оператор, за которым следует `--installed` параметр: `dnf list --installed`.
3.  &#x20;Какая команда dnf используется для добавления репозитория, расположенного по адресу `https://www.example.url/home:reponame.repo`, в систему?

    Работа с репозиториями — это «изменение конфигурации», поэтому используйте `config-manager` и параметр `--add_repo`: `dnf config-manager --add_repo https://www.example.url/home:reponame.repo`.
4.  Как вы можете использовать `zypper`, чтобы проверить, установлен ли пакет `unzip`?

    Вам нужно выполнить поиск (`se`) по установленным (`-i`) пакетам: `zypper se -i unzip`.
5.  Используя `yum`, выясните, какой пакет предоставляет файл `/bin/wget`.

    Чтобы узнать, что содержит файл, используйте `whatprovides` и имя файла: `yum whatprovides /bin/wget`.
