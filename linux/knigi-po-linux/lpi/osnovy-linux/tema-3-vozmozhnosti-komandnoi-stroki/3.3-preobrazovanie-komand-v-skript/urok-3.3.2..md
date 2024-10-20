# Урок 3.3.2.

## Введение <a href="#sec.3.3_02-in" id="sec.3.3_02-in"></a>

В последнем разделе мы использовали этот простой пример для демонстрации сценариев Bash:

```sh
#!/bin/bash

# A simple script to greet a single user.

if [ $# -eq 1 ]
then
  username=$1

  echo "Hello $username!"
else
  echo "Please enter only one argument."
fi
echo "Number of arguments: $#."
```

* Все скрипты должны начинаться с _shebang_, который определяет путь к интерпретатору.
* Все скрипты должны содержать комментарии, описывающие их использование.
* Этот конкретный скрипт работает с _аргументом_, который передаётся скрипту при его вызове.
* Этот скрипт содержит _оператор if_, который проверяет условия встроенной переменной `$#`. Эта переменная устанавливается в соответствии с количеством аргументов.
* Если количество переданных в скрипт аргументов равно 1, то значение первого аргумента передаётся в новую переменную с именем `username` и скрипт выводит приветствие для пользователя. В противном случае выводится сообщение об ошибке.
* Наконец, скрипт выводит количество аргументов. Это полезно для отладки.

Это полезный пример для начала объяснения некоторых других особенностей скриптов Bash.

## Коды завершения <a href="#exit_codes" id="exit_codes"></a>

Вы заметите, что у нашего скрипта есть два возможных состояния: либо он выводит `"Hello <user>!"`, либо выводит сообщение об ошибке. Это вполне нормально для многих наших основных утилит. Рассмотрим `cat`, с которым вы, без сомнения, уже хорошо знакомы.

Давайте сравним успешное использование `cat` с ситуацией, когда оно не работает. Напомним, что в нашем примере выше это скрипт под названием `new_script.sh`.

```bash
$ cat -n new_script.sh

     1	#!/bin/bash
     2
     3	# A simple script to greet a single user.
     4
     5	if [ $# -eq 1 ]
     6	then
     7	  username=$1
     8
     9	  echo "Hello $username!"
    10	else
    11	  echo "Please enter only one argument."
    12	fi
    13	echo "Number of arguments: $#."
```

Эта команда выполнена успешно, и вы заметите, что флаг `-n` также вывел номера строк. Это очень полезно при отладке скриптов, но обратите внимание, что они _не являются_ частью скрипта.

Теперь мы проверим значение новой встроенной переменной `$?`. Пока просто обратите внимание на результат

```bash
$ echo $?
0
```

Теперь давайте рассмотрим ситуацию, в которой `cat` выдаст ошибку. Сначала мы увидим сообщение об ошибке, а затем проверим значение `$?`.

```bash
$ cat -n dummyfile.sh
cat: dummyfile.sh: No such file or directory
$ echo $?
1
```

Объяснение такому поведению следующее: при любом выполнении утилиты `cat` возвращается _код завершения_. Код завершения сообщает нам, была ли команда выполнена успешно или с ошибкой. Код завершения _ноль_ указывает на то, что команда выполнена успешно. Это верно почти для каждой команды Linux, с которой вы работаете. Любой другой код завершения указывает на ту или иную ошибку. Код завершения _последней выполненной команды_ будет сохранён в переменной `$?`.

Коды завершения обычно не видны пользователям-людям, но они очень полезны при написании скриптов. Рассмотрим сценарий, в котором мы копируем файлы на удалённый сетевой диск. Существует множество причин, по которым задача копирования может завершиться неудачно: например, наш локальный компьютер может быть не подключён к сети или удалённый диск может быть заполнен. Проверяя код завершения нашей утилиты копирования, мы можем предупредить пользователя о проблемах при запуске скрипта.

Очень полезно использовать коды завершения, поэтому мы сделаем это сейчас. В нашем скрипте есть два пути: успешный и неудачный. Давайте использовать ноль для обозначения успешного выполнения, а единицу — для обозначения неудачного.

```bash
     1	#!/bin/bash
     2
     3	# A simple script to greet a single user.
     4
     5	if [ $# -eq 1 ]
     6	then
     7	  username=$1
     8
     9	  echo "Hello $username!"
    10	  exit 0
    11	else
    12	  echo "Please enter only one argument."
    13	  exit 1
    14	fi
    15	echo "Number of arguments: $#."
```

```bash
$ ./new_script.sh Carol
Hello Carol!
$ echo $?
0
```

Обратите внимание, что команда `echo` в строке 15 была полностью проигнорирована. Использование `exit` немедленно завершает выполнение скрипта, поэтому эта строка никогда не встречается.

## Обработка многих аргументов <a href="#handling_many_arguments" id="handling_many_arguments"></a>

На данный момент наш скрипт может обрабатывать только одно имя пользователя за раз. Любое количество аргументов, кроме одного, приведёт к ошибке. Давайте рассмотрим, как можно сделать этот скрипт более универсальным.

Первым побуждением пользователя может быть использование большего количества позиционных переменных, таких как `$2`, `$3` и так далее. К сожалению, мы не можем предвидеть количество аргументов, которые пользователь может использовать. Чтобы решить эту проблему, будет полезно ввести больше встроенных переменных.

Мы изменим логику нашего скрипта. При отсутствии аргументов должна возникать ошибка, но любое другое количество аргументов должно работать. Этот новый скрипт будет называться `friendly2.sh`.

```sh
     1	#!/bin/bash
     2
     3	# a friendly script to greet users
     4
     5	if [ $# -eq 0 ]
     6	then
     7	  echo "Please enter at least one user to greet."
     8	  exit 1
     9	else
    10	  echo "Hello $@!"
    11	  exit 0
    12	fi
```

```sh
$ ./friendly2.sh Carol Dave Henry
Hello Carol Dave Henry!
```

Есть две встроенные переменные, которые содержат все аргументы, переданные скрипту: `$@` и `$*`. По большей части они ведут себя одинаково. Bash _анализирует_ аргументы и разделяет их, когда встречает пробел между ними. По сути, содержимое `$@` выглядит так:

| `0`     | `1`    | `2`     |
| ------- | ------ | ------- |
| `Carol` | `Dave` | `Henry` |

{% hint style="info" %}
В контексте параметров командной строки в Bash, символы `$@` и `$*` имеют следующие различия:

1. `$*`: представляет все аргументы командной строки как одну строку. Это полезно, когда вы хотите передать все аргументы как один аргумент другой команде или функции.
2. `$@`: представляет каждый аргумент командной строки отдельно. Это позволяет вам обращаться к каждому аргументу индивидуально.
{% endhint %}



Если вы знакомы с другими языками программирования, то можете узнать в этом типе переменных _массив_. Массивы в Bash можно создавать, просто разделяя элементы пробелами, как в переменной `FILES` в скрипте `arraytest` ниже:

```bash
FILES="/usr/sbin/accept /usr/sbin/pwck/ usr/sbin/chroot"
```

Он содержит список из множества элементов. Пока что это не очень полезно, потому что мы ещё не представили способ обработки этих элементов по отдельности.

## Цикл for

Давайте обратимся к предыдущему примеру с `arraytest` Если вы помните, в этом примере мы определяем собственный массив под названием `FILES`. Нам нужно «распаковать» эту переменную и получить доступ к каждому отдельному значению по очереди. Для этого мы будем использовать структуру, называемую _циклом for_, которая присутствует во всех языках программирования. Мы будем использовать две переменные: одна — для диапазона, а другая — для отдельного значения, с которым мы работаем в данный момент. Это сценарий в полном объеме:

```bash
#!/bin/bash

FILES="/usr/sbin/accept /usr/sbin/pwck/ usr/sbin/chroot"

for file in $FILES
do
  ls -lh $file
done
```

```bash
$ ./arraytest
lrwxrwxrwx 1 root root 10 Apr 24 11:02 /usr/sbin/accept -> cupsaccept
-rwxr-xr-x 1 root root 54K Mar 22 14:32 /usr/sbin/pwck
-rwxr-xr-x 1 root root 43K Jan 14 07:17 /usr/sbin/chroot
```

Если вы снова обратитесь к приведённому выше примеру `friendly2.sh`, то увидите, что мы работаем с диапазоном значений, содержащихся в одной переменной `$@`. Для наглядности мы назовём последнюю переменную `username`. Теперь наш скрипт выглядит так:

```bash
     1	#!/bin/bash
     2
     3	# a friendly script to greet users
     4
     5	if [ $# -eq 0 ]
     6	then
     7	  echo "Please enter at least one user to greet."
     8	  exit 1
     9	else
    10	  for username in $@
    11	  do
    12	    echo "Hello $username!"
    13	  done
    14	  exit 0
    15	fi
```

Помните, что переменная, которую вы здесь определяете, может называться как угодно, и что все строки внутри `do…​ done` будут выполняться по одному разу для каждого элемента массива. Давайте посмотрим на результат работы нашего скрипта:

```bash
$ ./friendly2.sh Carol Dave Henry
Hello Carol!
Hello Dave!
Hello Henry!
```

Теперь предположим, что мы хотим, чтобы наш результат выглядел более человечным. Мы хотим, чтобы наше приветствие занимало одну строку.

```bash
     1	#!/bin/bash
     2
     3	# a friendly script to greet users
     4
     5	if [ $# -eq 0 ]
     6	then
     7	  echo "Please enter at least one user to greet."
     8	  exit 1
     9	else
    10	  echo -n "Hello $1"
    11	  shift
    12	  for username in $@
    13	  do
    14	    echo -n ", and $username"
    15	  done
    16	  echo "!"
    17	  exit 0
    18	fi
```

Пара замечаний:

* Использование `-n` с `echo`_подавляет перевод строки_ после печати. Это означает, что все повторы будут выводиться в одну строку, а перевод строки будет выводиться только после `` !` `` в строке 16.
* Команда `shift` удалит первый элемент нашего массива, так что получится следующее:

| `0`     | `1`    | `2`     |
| ------- | ------ | ------- |
| `Carol` | `Dave` | `Henry` |

Становится таким:

| `0`    | `1`     |
| ------ | ------- |
| `Dave` | `Henry` |

Давайте понаблюдаем за результатом:

```bash
$ ./friendly2.sh Carol
Hello Carol!
$ ./friendly2.sh Carol Dave Henry
Hello Carol, and Dave, and Henry!
```

## Использование регулярных выражений для проверки ошибок <a href="#using_regular_expressions_to_perform_error_checking" id="using_regular_expressions_to_perform_error_checking"></a>

Возможно, мы хотим проверить все аргументы, которые вводит пользователь. Например, возможно, мы хотим убедиться, что все имена, передаваемые в `friendly2.sh`, содержат _только буквы_, а любые специальные символы или цифры вызовут ошибку. Для проверки на наличие ошибок мы будем использовать `grep`.

Напомним, что мы можем использовать регулярные выражения с `grep`.

```bash
$ echo Animal | grep "^[A-Za-z]*$"
Animal
$ echo $?
0
```

```bash
$ echo 4n1ml | grep "^[A-Za-z]*$"
$ echo $?
1
```

`^` и `$` обозначают начало и конец строки соответственно.`[A-Za-z]` обозначает диапазон букв, прописных или строчных.`*` — это _квантификатор_, который изменяет диапазон букв так, чтобы мы сопоставляли от нуля до множества букв. Таким образом, `grep` будет успешным, если на входе _только_ буквы, и неудачным в противном случае.

Далее следует отметить, что `grep` возвращает коды завершения в зависимости от того, было ли совпадение или нет. При положительном совпадении возвращается `0`, а при отсутствии совпадения возвращается `1`. Мы можем использовать это для проверки аргументов внутри нашего скрипта.

```bash
     1	#!/bin/bash
     2
     3	# a friendly script to greet users
     4
     5	if [ $# -eq 0 ]
     6	then
     7	  echo "Please enter at least one user to greet."
     8	  exit 1
     9	else
    10	  for username in $@
    11	  do
    12	    echo $username | grep "^[A-Za-z]*$" > /dev/null
    13	    if [ $? -eq 1 ]
    14	    then
    15	      echo "ERROR: Names must only contains letters."
    16	      exit 2
    17	    else
    18	      echo "Hello $username!"
    19	    fi
    20	  done
    21	  exit 0
    22	fi
```

В строке 12 мы перенаправляем стандартный вывод в `/dev/null`, что является простым способом его подавления. Мы не хотим видеть вывод команды `grep`, мы хотим только проверить её код завершения, что и происходит в строке 13. Обратите внимание, что мы используем код завершения `2` для обозначения недопустимого аргумента. Как правило, рекомендуется использовать разные коды завершения для обозначения разных ошибок; таким образом, опытный пользователь может использовать эти коды завершения для устранения неполадок.

```bash
$ ./friendly2.sh Carol Dave Henry
Hello Carol!
Hello Dave!
Hello Henry!
$ ./friendly2.sh 42 Carol Dave Henry
ERROR: Names must only contains letters.
$ echo $?
2
```

## Упражнения с руководством <a href="#sec.3.3_02-ge" id="sec.3.3_02-ge"></a>

1.  Прочтите содержание `script1.sh` ниже:

    ```bash
    #!/bin/bash

    if [ $# -lt 1 ]
    then
      echo "This script requires at least 1 argument."
      exit 1
    fi

    echo $1 | grep "^[A-Z]*$" > /dev/null
    if [ $? -ne 0 ]
    then
      echo "no cake for you!"
      exit 2
    fi

    echo "here's your cake!"
    exit 0
    ```

    Каков результат выполнения этих команд?

    * `./script1.sh`
    * `echo $?`
    * `./script1.sh cake`
    * `echo $?`
    * `./script1.sh CAKE`
    * `echo $?`
2.  Прочитайте содержимое файла `script2.sh`:

    ```bash
    for filename in $1/*.txt
    do
       cp $filename $filename.bak
    done
    ```

    Опишите назначение этого скрипта так, как вы его понимаете.

## Исследовательские упражнения <a href="#sec.3.3_02-ee" id="sec.3.3_02-ee"></a>

1. Создайте скрипт, который будет принимать любое количество аргументов от пользователя и выводить только те аргументы, которые являются числами больше 10.

## Краткие сведения <a href="#sec.3.3_02-su" id="sec.3.3_02-su"></a>

В этом разделе вы узнали:

* Что такое коды завершения, что они означают и как их реализовать
* Как проверить код завершения команды
* Что такое цикл `for`  и как его использовать с массивами
* Как использовать `grep` , регулярные выражения и коды завершения для проверки пользовательского ввода в скриптах.

Команды, используемые в упражнениях:

`shift`

Будет удален первый элемент массива.

Специальные переменные:

`$?`

Содержит код завершения последней выполненной команды.

`$@`, `$*`

Содержать все аргументы, передаваемые скрипту, в виде массива.

## Ответы на упражнения с руководством <a href="#sec.3.3_02-age" id="sec.3.3_02-age"></a>

1.  Прочтите содержание `script1.sh` ниже:

    ```bash
    #!/bin/bash

    if [ $# -lt 1 ]
    then
      echo "This script requires at least 1 argument."
      exit 1
    fi

    echo $1 | grep "^[A-Z]*$" > /dev/null
    if [ $? -ne 0 ]
    then
      echo "no cake for you!"
      exit 2
    fi

    echo "here's your cake!"
    exit 0
    ```

    Каков результат выполнения этих команд?

    *   Команда: `./script1.sh`

        Результат: `This script requires at least 1 argument.`
    *   Команда: `echo $?`

        Результат: `1`
    *   Команда: `./script1.sh cake`

        Результат: `no cake for you!`
    *   Команда: `echo $?`

        Результат: `2`
    *   Команда: `./script1.sh CAKE`

        Результат: `here’s your cake!`
    *   Команда: `echo $?`

        Результат: `0`
2.  Прочитайте содержимое файла `script2.sh`:

    ```bash
    for filename in $1/*.txt
    do
       cp $filename $filename.bak
    done
    ```

    Опишите назначение этого скрипта так, как вы его понимаете.

    Этот скрипт создаст резервные копии всех файлов, заканчивающихся на `.txt` в подкаталоге, указанном в первом аргументе.

## Ответы на исследовательские упражнения <a href="#sec.3.3_02-aee" id="sec.3.3_02-aee"></a>

1.  Создайте скрипт, который будет принимать любое количество аргументов от пользователя и выводить только те аргументы, которые являются числами больше 10.

    ```bash
    #!/bin/bash

    for i in $@
    do
     echo $i | grep "^[0-9]*$" > /dev/null
     if [ $? -eq 0 ]
     then
     if [ $i -gt 10 ]
     then
     echo -n "$i "
     fi
     fi
    done
    echo ""
    ```
