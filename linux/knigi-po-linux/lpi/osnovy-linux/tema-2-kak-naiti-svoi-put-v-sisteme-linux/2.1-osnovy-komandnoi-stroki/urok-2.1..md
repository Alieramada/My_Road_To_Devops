# Урок 2.1.

## &#x20;Введение

Современные дистрибутивы Linux имеют широкий спектр графических пользовательских интерфейсов, но администратору всегда нужно будет знать, как работать с командной строкой, или, как ее еще называют, _оболочкой_. Оболочка - это программа, которая обеспечивает обмен текстовыми данными между операционной системой и пользователем. Обычно это программа в текстовом режиме, которая считывает вводимые пользователем данные и интерпретирует их как команды для системы.

В Linux существует несколько различных оболочек, вот лишь некоторые из них:

* Bourne-again shell (Bash)
* Оболочка C (csh или tcsh, усовершенствованный csh)
* Оболочка Korn (ksh)
* Оболочка Z (zsh)

В Linux наиболее распространённой является оболочка Bash. Она же будет использоваться в примерах и упражнениях, приведённых здесь.

При использовании интерактивной оболочки пользователь вводит команды в так называемую строку приглашения. В каждом дистрибутиве Linux строка приглашения по умолчанию может выглядеть немного по-разному, но обычно она имеет следующую структуру:

```
username@hostname current_directory shell_type
```

В Ubuntu или Debian GNU/Linux приглашение для обычного пользователя, скорее всего, будет выглядеть так:

```
carol@mycomputer:~$
```

Приглашение суперпользователя будет выглядеть следующим образом:

```
root@mycomputer:~#
```

В CentOS или Red Hat Linux приглашение для обычного пользователя вместо этого будет выглядеть следующим образом:

```
[dave@mycomputer ~]$
```

И приглашение суперпользователя будет выглядеть следующим образом:

```
[root@mycomputer ~]#
```

Давайте объясним каждый компонент структуры:

`username`

Имя пользователя, запускающего оболочку

`hostname`

Имя хоста, на котором запускается оболочка. Существует также команда `hostname`, с помощью которой вы можете отобразить или задать имя хоста системы.

`current_directory`

Каталог, в котором в данный момент находится оболочка. А `~` означает, что оболочка находится в домашнем каталоге текущего пользователя.

`shell_type`

`$` указывает, что оболочка запущена обычным пользователем.

`#` указывает, что оболочка запускается суперпользователем `root`.

Поскольку нам не нужны никакие особые привилегии, в следующих примерах мы будем использовать непривилегированное приглашение `$`.

## &#x20;Структура командной строки

Большинство команд в командной строке имеют одну и ту же базовую структуру:

```
command  [option(s)/parameter(s)...]  [argument(s)...]
```

Возьмем в качестве примера следующую команду:

```
$ ls -l /home
```

Давайте объясним назначение каждого компонента:

**Command**

Программа, которую будет запускать пользователь – `ls` в приведенном выше примере.

**Option(s)/Parameter(s)**

«Переключатель», который каким-либо образом изменяет поведение команды, например `-l` в приведённом выше примере. Опции могут быть доступны в короткой и длинной форме. Например, `-l` идентично `--format=long`.

Также можно комбинировать несколько вариантов, и для краткой формы буквы обычно можно вводить вместе. Например, все следующие команды выполняют то же самое:

```
$ ls -al
$ ls -a -l
$ ls --all --format=long
```

**Argument(s)**

Дополнительные данные, которые требуются программе, такие как имя файла или путь, например, `/home` в приведенном выше примере.

Единственной обязательной частью этой структуры является сама команда. Как правило, все остальные элементы необязательны, но программе может потребоваться указать определенные опции, параметры или аргументы.

{% hint style="info" %}
Большинство команд отображают краткий обзор доступных параметров при запуске с параметром `--help` . В ближайшее время мы рассмотрим дополнительные способы получения информации о командах Linux.
{% endhint %}

## &#x20;Типы поведения команд

Оболочка поддерживает два типа команд:

**Внутренний**

Эти команды являются частью самой командной оболочки и не являются отдельными программами. Существует около 30 таких команд. Их основное назначение - выполнение задач внутри оболочки (например, `cd`, `set`, `export`).

**Внешний**

Эти команды хранятся в отдельных файлах. Обычно эти файлы представляют собой двоичные программы или скрипты. Когда выполняется команда, которая не является встроенной в оболочку, оболочка использует `PATH` переменную для поиска исполняемого файла с тем же именем, что и у команды. В дополнение к программам, которые устанавливаются с менеджером пакетов дистрибутива, пользователи также могут создавать свои собственные внешние команды.

Команда **`type`** показывает, к какому типу относится конкретная команда:

```
$ type echo
echo is a shell builtin
$ type man
man is /usr/bin/man
```

## &#x20;Цитирование

Как пользователю Linux, вам придётся создавать файлы или переменные и управлять ими различными способами. Это легко сделать при работе с короткими именами файлов и отдельными значениями, но становится сложнее, когда, например, используются пробелы, специальные символы и переменные. Оболочки предоставляют функцию, называемую цитированием, которая заключает такие данные в различные виды кавычек ("" и '). В Bash есть три типа кавычек:

* Двойные кавычки
* Одинарные кавычки
* Экранирующие символы

Например, следующие команды не работают одинаково из-за кавычек:

```
$ TWOWORDS="two words"
$ touch $TWOWORDS
$ ls -l
-rw-r--r-- 1 carol carol     0 Mar 10 14:56 two
-rw-r--r-- 1 carol carol     0 Mar 10 14:56 words
$ touch "$TWOWORDS"
$ ls -l
-rw-r--r-- 1 carol carol     0 Mar 10 14:56  two
-rw-r--r-- 1 carol carol     0 Mar 10 14:58 'two words'
-rw-r--r-- 1 carol carol     0 Mar 10 14:56  words
$ touch '$TWOWORDS'
$ ls -l
-rw-r--r-- 1 carol carol     0 Mar 10 15:00 '$TWOWORDS'
-rw-r--r-- 1 carol carol     0 Mar 10 14:56  two
-rw-r--r-- 1 carol carol     0 Mar 10 14:58 'two words'
-rw-r--r-- 1 carol carol     0 Mar 10 14:56  words
```

{% hint style="info" %}
Строка с `TWOWORDS=` - это переменная Bash, которую мы создали сами. Мы представим переменные позже. Это просто для того, чтобы показать вам, как кавычки влияют на вывод переменных.
{% endhint %}

## &#x20;**Двойные кавычки**

Двойные кавычки позволяют командной оболочке воспринимать текст между кавычками ("...") как обычные символы. Все специальные символы теряют свое значение, за исключением `$` (знак доллара), `\` (обратная косая черта) и `` ` `` (кавычка). Это означает, что переменные, подстановка команд и арифметические функции по-прежнему могут использоваться.

Например, двойные кавычки не влияют на подстановку переменной `$USER`:

```
$ echo I am $USER
I am tom
$ echo "I am $USER"
I am tom
```

С другой стороны, пробел теряет своё значение в качестве разделителя аргументов:

```
$ touch new file
$ ls -l
-rw-rw-r-- 1 tom students 0 Oct 8 15:18 file
-rw-rw-r-- 1 tom students 0 Oct 8 15:18 new
$ touch "new file"
$ ls -l
-rw-rw-r-- 1 tom students 0 Oct 8 15:19 new file
```

Как вы можете видеть, в первом примере команда `touch` создает два отдельных файла, команда интерпретирует две строки как отдельные аргументы. Во втором примере команда интерпретирует обе строки как один аргумент, поэтому создает только один файл. Однако рекомендуется избегать пробела в именах файлов. Вместо этого можно было бы использовать символ подчеркивания (`_`) или точку (`.`).

## &#x20;**Одинарные кавычки**

Одинарные кавычки не имеют исключений в виде двойных кавычек. Они лишают каждый символ какого-либо особого значения. Давайте возьмем один из первых примеров из приведенных выше.:

```
$ echo I am $USER
I am tom
```

При применении одинарных кавычек вы видите другой результат:

```
$ echo 'I am $USER'
I am $USER
```

Теперь команда отображает точную строку без подстановки переменной.

## &#x20;**Экранирующие символы**

Мы можем использовать _экранирующие символы_ для удаления специальных значений символов в Bash. Вернёмся к переменной среды `$USER`:

```
$ echo $USER
carol
```

Мы видим, что по умолчанию содержимое переменной отображается в терминале. Однако, если перед знаком доллара поставить обратную косую черту (`\`), то особое значение знака доллара будет сведено на нет. Это, в свою очередь, не позволит Bash расширить значение переменной до имени пользователя, выполняющего команду, но вместо этого интерпретирует имя переменной буквально:

```
$ echo \$USER
$USER
```

Если вы помните, мы можем получить аналогичные результаты, используя одинарную кавычку, которая выводит буквальное содержимое всего, что находится между одинарными кавычками. Однако экранирующий символ работает по-другому, указывая Bash игнорировать любое особое значение, которым может обладать символ, которому он предшествует.

## &#x20;Упражнения с руководством

Разделите приведенные ниже строки на компоненты command, option(s)/parameter(s), argument(s):

Пример: `cat -n /etc/passwd`

| Команда:  | `cat`         |
| --------- | ------------- |
| Вариант:  | `-n`          |
| Аргумент: | `/etc/passwd` |

`ls -l /etc`

\


| Команда:  |   |
| --------- | - |
| Вариант:  |   |
| Аргумент: |   |

`ls -l -a`

| Команда:  |   |
| --------- | - |
| Вариант:  |   |
| Аргумент: |   |

`cd /home/user`

| Команда:  |   |
| --------- | - |
| Вариант:  |   |
| Аргумент: |   |

Определите, к какому типу относятся следующие команды:

Пример:

| `pwd` | Встроенная оболочка |
| ----- | ------------------- |
| `mv`  | Внешняя команда     |

\


| `cd`   |   |
| ------ | - |
| `cat`  |   |
| `exit` |   |

Разрешите следующие команды, использующие кавычки:

Пример:

| `echo "$HOME is my home directory"` | `echo /home/user is my home directory` |
| ----------------------------------- | -------------------------------------- |

| `touch "$USER"` |   |
| --------------- | - |
| `touch 'touch'` |   |

## &#x20;Исследовательские упражнения

1. С помощью одной команды и расширения фигурных скобок в Bash (ознакомьтесь со справочной страницей Bash) создайте 5 файлов с номерами от 1 до 5 с префиксом `game` (`game1`, `game2`, …​).
2. Удалите все 5 файлов, которые вы только что создали, с помощью одной команды, используя другой специальный символ (см. _Расширение пути_ в справочных страницах Bash).
3. Есть ли другой способ заставить две команды взаимодействовать друг с другом? Что это за команды?

### Краткие сведения <a href="#sec.2.1_01-su" id="sec.2.1_01-su"></a>

В этой лабораторной работе вы узнали:

* Концепции оболочки Linux
* Что такое оболочка Bash
* Структура командной строки
* Введение в цитирование

**Команды, используемые в упражнениях:**

`bash`

Самая популярная оболочка на компьютерах с Linux.

`echo`

Вывод текста на терминал.

`ls`

Составьте список содержимого каталога.

`type`

Покажите, как выполняется конкретная команда.

`touch`

Создайте пустой файл или обновите дату модификации существующего файла.

`hostname`

Отображение или изменение имени хоста системы.

### Ответы на упражнения с Руководством <a href="#sec.2.1_01-age" id="sec.2.1_01-age"></a>

1. Разделите приведенные ниже строки на компоненты
2.
   *   `ls -l /etc`

       | Команда:  | `ls`   |
       | --------- | ------ |
       | Вариант:  | `-l`   |
       | Аргумент: | `/etc` |
   *   `ls -l -a`

       | Команда:  | `ls`    |
       | --------- | ------- |
       | Вариант:  | `-l -a` |
       | Аргумент: |         |
   *   `cd /home/user`

       | Команда:  | `cd`         |
       | --------- | ------------ |
       | Вариант:  |              |
       | Аргумент: | `/home/user` |
3.  Определите, к какому типу относятся следующие команды:

    | `cd`   | Встроенная оболочка |
    | ------ | ------------------- |
    | `cat`  | Внешняя команда     |
    | `exit` | Встроенная оболочка |
4.  Разрешите следующие команды, использующие кавычки:

    | `touch "$USER"` | `tom`                         |
    | --------------- | ----------------------------- |
    | `touch 'touch'` | Создает файл с именем `touch` |

## &#x20;Ответы на исследовательские упражнения

С помощью одной команды и расширения фигурных скобок в Bash (ознакомьтесь со справочной страницей Bash) создайте 5 файлов с номерами от 1 до 5 с префиксом `game` (`game1`, `game2`, …​).

```
$ touch game{1..5}
$ ls
game1  game2  game3  game4  game5
```

Удалите все 5 файлов, которые вы только что создали, всего одной командой, используя разные специальные символы (просмотрите _расширение пути_ на страницах руководства Bash).

Поскольку все файлы начинаются с `game` и заканчиваются одним символом (в данном случае числом от 1 до 5), `?` может использоваться как специальный символ для последнего символа в имени файла:

```
$ rm game?
```

Есть ли какой-либо другой способ заставить две команды взаимодействовать друг с другом? Что это?

Да, одна команда может, например, записать данные в файл, который затем обрабатывается другой командой. Linux также может собирать выходные данные одной команды и использовать их в качестве входных данных для другой команды. Это называется _piping_, и мы узнаем о нем больше в следующем уроке.
