# Урок 2.4.

### Введение <a href="#sec.2.4_01-in" id="sec.2.4_01-in"></a>

В этом уроке рассказывается об управлении файлами и каталогами в Linux с помощью инструментов командной строки.

Файл - это набор данных с именем и набором атрибутов. Если бы, например, вы перенесли несколько фотографий со своего телефона на компьютер и дали им описательные названия, то теперь у вас на компьютере была бы куча файлов изображений. У этих файлов есть такие атрибуты, как время последнего обращения к файлу или его изменения.

Каталог - это особый вид файла, используемый для упорядочивания файлов. Хороший способ представить каталоги как папки, используемые для упорядочивания документов в картотеке. В отличие от бумажных папок с файлами, вы можете легко помещать каталоги внутри других каталогов.

Командная строка - наиболее эффективный способ управления файлами в системе Linux. Средства командной оболочки и командной строки обладают функциями, которые делают использование командной строки более быстрым и простым, чем графический файловый менеджер.

В этом разделе вы будете использовать команды `ls`, `mv`, `cp`, `pwd`, `find`, `touch`, `rm`, `rmdir`, `echo`, `cat и` `mkdir` для управления файлами и каталогами и их упорядочивания.

## &#x20;Чувствительность к регистру

В отличие от Microsoft Windows, в системах Linux имена файлов и каталогов чувствительны к регистру. Это означает, что имена `/etc/` и `/ETC/` относятся к разным каталогам. Попробуйте выполнить следующие команды:

```
$ cd /
$ ls
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr
$ cd ETC
bash: cd: ETC: No such file or directory
$ pwd
/
$ cd etc
$ pwd
/etc
```

`pwd` отображает каталог, в котором вы находитесь в данный момент. Как вы можете видеть, переход в каталог `/ETC` не сработало, поскольку такого каталога нет. Переход в каталог `/etc` который существует, прошло успешно.

## &#x20;Создание каталогов

Команда `mkdir` используется для создания каталогов.

Давайте создадим новый каталог в нашем домашнем каталоге:

```
$ cd ~
$ pwd
/home/user
$ ls
Desktop  Documents  Downloads
$ mkdir linux_essentials-2.4
$ ls
Desktop  Documents  Downloads  linux_essentials-2.4
$ cd linux_essentials-2.4
$ pwd
/home/emma/linux_essentials-2.4
```

На протяжении этого урока все команды будут выполняться в этом каталоге или в одном из его подкаталогов.

Чтобы легко вернуться в каталог уроков из любой другой позиции в вашей файловой системе, вы можете использовать команду:

`$ cd ~/linux_essentials-2.4`

Когда вы окажетесь в каталоге с уроками, создайте ещё несколько каталогов, которые мы будем использовать для упражнений. Вы можете добавить все названия каталогов, разделённые пробелами  с командой `mkdir`:

```
$ mkdir creating moving copying/files copying/directories deleting/directories deleting/files globs
mkdir: cannot create directory ‘copying/files’: No such file or directory
mkdir: cannot create directory ‘copying/directories’: No such file or directory
mkdir: cannot create directory ‘deleting/directories’: No such file or directory
mkdir: cannot create directory ‘deleting/files’: No such file or directory
$ ls
creating  globs  moving
```

Обратите внимание на сообщение об ошибке и на то, что были созданы только `moving`, `globs` и `creating`. Каталоги `copying` и `deleting` еще не существуют. `mkdir` по умолчанию не создает каталог внутри каталога, который не существует. Параметр `-p` или `--parents` указывает `mkdir` создать родительские каталоги, если они не существуют. Попробуйте ту же `mkdir` команду с `-p` параметром:

```
$ mkdir -p creating moving copying/files copying/directories deleting/directories deleting/files globs
```

Теперь вы не получаете сообщений об ошибках. Давайте посмотрим, какие каталоги существуют сейчас:

```
$ find
.
./creating
./moving
./globs
./copying
./copying/files
./copying/directories
./deleting
./deleting/directories
./deleting/files
```

Программа `find` обычно используется для поиска файлов и каталогов, но без каких-либо параметров она покажет вам список всех файлов, каталогов и подкаталогов в текущем каталоге.

{% hint style="info" %}
При перечислении содержимого каталога с помощью `ls` особенно удобны параметры `-t` и `-r`. Они сортируют вывод по времени (`-t`) и меняют порядок сортировки на противоположный (`-r`). В этом случае самые новые файлы будут в нижней части вывода.
{% endhint %}

## &#x20;Создание файлов

Обычно файлы создаются программами, которые работают с данными, хранящимися в файлах. Пустой файл можно создать с помощью команды `touch`. Если вы запустите `touch` для существующего файла, содержимое файла изменено не будет, но будет обновлена временная метка изменения файла.

Выполните следующую команду, чтобы создать несколько файлов для урока по глобированию:

```
$ touch globs/question1 globs/question2012 globs/question23 globs/question13 globs/question14
$ touch globs/star10 globs/star1100 globs/star2002 globs/star2013
```

Теперь давайте убедимся, что все файлы существуют в `globs` каталоге:

```
$ cd globs
$ ls
question1   question14    question23  star1100  star2013
question13  question2012  star10      star2002
```

Заметили, как `touch` создал файлы? Вы можете просмотреть содержимое текстового файла с помощью `cat` команды. Попробуйте применить это к одному из только что созданных файлов.:

```
$ cat question14
```

Поскольку `touch` создает пустые файлы, вы не должны получать выходных данных. Вы можете использовать `echo` с `>` для создания простых текстовых файлов. Попробуйте:

```
$ echo hello > question15
$ cat question15
hello
```

`echo` отображает текст в командной строке. Символ `>` указывает командной оболочке записать выходные данные команды в указанный файл вместо вашего терминала. Это приводит к тому, что выходные данные `echo`, `hello` в данном случае, записываются в файл `question15`. Это не относится конкретно к `echo`, его можно использовать с любой командой.

{% hint style="info" %}
Будьте осторожны при использовании `>`! Если указанный файл уже существует, он будет перезаписан!
{% endhint %}

## &#x20;Переименование файлов

Файлы перемещаются и переименовываются с помощью команды `mv`.

Установите для своего рабочего каталога значение `moving` .

```
cd ~/linux_essentials-2.4/moving
```

Создайте несколько файлов для практики. К этому моменту вы уже должны быть знакомы с этими командами:

```
$ touch file1 file22
$ echo file3 > file3
$ echo file4 > file4
$ ls
file1 file22 file3 file4
```

Предположим, что `file22` — это опечатка, и должно быть `file2`. Исправьте это с помощью команды `mv`. При переименовании файла первый аргумент — это текущее имя, второй — новое имя:

```
$ mv file22 file2
$ ls
file1 file2 file3 file4
```

Будьте осторожны с командой `mv`: если вы переименуете файл в имя существующего файла, он будет перезаписан. Давайте проверим это с помощью `file3` и `file4`:

```
$ cat file3
file3
$ cat file4
file4
$ mv file4 file3
$ cat file3
file4
$ ls
file1  file2  file3
```

Обратите внимание, каким стало содержимое `file3` теперь он `file4`. Используйте `-i` опцию, чтобы сделать `mv` запрос, собираетесь ли вы перезаписать существующий файл. Попробуйте:\\

```
$ touch file4 file5
$ mv -i file4 file3
mv: overwrite ‘file3’? y
```

## &#x20;Перемещение файлов

Файлы перемещаются из одного каталога в другой с помощью команды `mv`.

Создайте несколько каталогов для перемещения файлов в них:

```
$ cd ~/linux_essentials-2.4/moving
$ mkdir dir1 dir2
$ ls
dir1  dir2  file1  file2  file3  file
```

Переместите `file1` в `dir1`:

```
$ mv file1 dir1
$ ls
dir1  dir2  file2  file3  file5
$ ls dir1
file1
```

Обратите внимание, что последним аргументом `mv` является целевой каталог. Если последним аргументом `mv` является каталог, файлы перемещаются в него. В одной команде `mv` можно указать несколько файлов:

```
$ mv file2 file3 dir2
$ ls
dir1  dir2  file5
$ ls dir2
file2  file3
```

`mv` также можно использовать для перемещения и переименования каталогов. Переименовать `dir1` в `dir3`:

```
$ ls
dir1  dir2  file5
$ ls dir1
file1
$ mv dir1 dir3
$ ls
dir2  dir3  file5
$ ls dir3
file1
```

## &#x20;Удаление файлов и каталогов

Команда `rm` может удалять файлы и каталоги, в то время как команда `rmdir` может удалять только каталоги. Давайте очистим `moving` каталог, удалив `file5`:

```
$ cd ~/linux_essentials-2.4/moving
$ ls
dir2  dir3  file5
$ rmdir file5
rmdir: failed to remove ‘file5’: Not a directory
$ rm file5
$ ls
dir2  dir3
```

По умолчанию `rmdir` можно удалять только пустые каталоги, поэтому нам пришлось использовать `rm` для удаления обычного файла. Попробуйте удалить `deleting` каталог:

```
$ cd ~/linux_essentials-2.4/
$ ls
copying  creating  deleting  globs  moving
$ rmdir deleting
rmdir: failed to remove ‘deleting’: Directory not empty
$ ls -l deleting
total 0
drwxrwxr-x. 2 emma emma 6 Mar 26 14:58 directories
drwxrwxr-x. 2 emma emma 6 Mar 26 14:58 files
```

По умолчанию `rmdir` не позволяет удалить непустой каталог. Используйте `rmdir` для удаления одного из пустых подкаталогов каталога `deleting`:

```
$ ls -a deleting/files
.  ..
$ rmdir deleting/files
$ ls -l deleting
directories
```

Удаление большого количества файлов или глубоких структур каталогов со множеством подкаталогов может показаться утомительным, но на самом деле это просто. По умолчанию `rm` работает только с обычными файлами. Опция `-r` используется для переопределения этого поведения. Будьте осторожны, `rm -r` это отличный ножной пистолет! При использовании `-r` опции `rm` будут удалены не только любые каталоги, но и все, что находится внутри этого каталога, включая подкаталоги и их содержимое. Посмотрите сами, как это работает `rm -r`

```
$ ls
copying  creating  deleting  globs  moving
$ rm deleting
rm: cannot remove ‘deleting’: Is a directory
$ ls -l deleting
total 0
drwxrwxr-x. 2 emma emma 6 Mar 26 14:58 directories
$ rm -r deleting
$ ls
copying  creating  globs  moving
```

Обратите внимание, что `deleting` исчезло, хотя оно не было пустым? Как и в случае с `mv`, `rm` имеет `-i` опцию, которая запрашивает ваше разрешение перед выполнением каких-либо действий. Используйте `rm -ri` для удаления каталогов из `moving` раздела, которые больше не нужны:

```
$ find
.
./creating
./moving
./moving/dir2
./moving/dir2/file2
./moving/dir2/file3
./moving/dir3
./moving/dir3/file1
./globs
./globs/question1
./globs/question2012
./globs/question23
./globs/question13
./globs/question14
./globs/star10
./globs/star1100
./globs/star2002
./globs/star2013
./globs/question15
./copying
./copying/files
./copying/directories
$ rm -ri moving
rm: descend into directory ‘moving’? y
rm: descend into directory ‘moving/dir2’? y
rm: remove regular empty file ‘moving/dir2/file2’? y
rm: remove regular empty file ‘moving/dir2/file3’? y
rm: remove directory ‘moving/dir2’? y
rm: descend into directory ‘moving/dir3’? y
rm: remove regular empty file ‘moving/dir3/file1’? y
rm: remove directory ‘moving/dir3’? y
rm: remove directory ‘moving’? y
```

