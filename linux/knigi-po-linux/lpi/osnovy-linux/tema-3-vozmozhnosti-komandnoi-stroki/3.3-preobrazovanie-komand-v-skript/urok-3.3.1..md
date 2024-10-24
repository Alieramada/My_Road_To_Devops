# Урок 3.3.1.

## Введение <a href="#sec.3.3_01-in" id="sec.3.3_01-in"></a>

До сих пор мы учились выполнять команды из оболочки, но мы также можем вводить команды в файл, а затем сделать этот файл исполняемым. При выполнении файла эти команды запускаются одна за другой. Такие исполняемые файлы называются _скриптами_, и они являются важнейшим инструментом для любого системного администратора Linux. По сути, мы можем рассматривать Bash как язык программирования, а также как оболочку.

## Вывод на печать <a href="#printing_output" id="printing_output"></a>

Давайте начнём с демонстрации команды, которую вы, возможно, видели в предыдущих уроках: `echo` выводит аргумент в стандартный вывод.

```sh
$ echo "Hello World!"
Hello World
```

Теперь мы воспользуемся перенаправлением файлов, чтобы отправить эту команду в новый файл под названием `new_script`.

```sh
$ echo 'echo "Hello World!"' > new_script
$ cat new_script
echo "Hello World!"
```

Файл `new_script` теперь содержит ту же команду, что и раньше.

## Создание исполняемого скрипта <a href="#making_a_script_executable" id="making_a_script_executable"></a>

Давайте продемонстрируем некоторые шаги, необходимые для того, чтобы этот файл работал так, как мы ожидаем. Первой мыслью пользователя может быть просто ввести название скрипта, как он ввёл бы название любой другой команды:

```sh
$ new_script
/bin/bash: new_script: command not found
```

Мы можем с уверенностью предположить, что `new_script` существует в нашем текущем местоположении, но обратите внимание, что сообщение об ошибке говорит нам не о том, что _**файл**_ не существует, а о том, что _**команда**_ не существует. Было бы полезно обсудить, как Linux обрабатывает команды и исполняемые файлы.

## Команды и `PATH` <a href="#commands_and_path" id="commands_and_path"></a>

Например, когда мы вводим в оболочку команду `ls` мы запускаем файл с именем `ls`, который существует в нашей файловой системе. Вы можете убедиться в этом, используя `which`:

```
$ which ls
/bin/ls
```

Было бы утомительно каждый раз вводить абсолютный путь к `ls` при просмотре содержимого каталога, поэтому в Bash есть _**переменная среды**_, которая содержит все каталоги, в которых мы можем найти команды, которые хотим запустить. Вы можете просмотреть содержимое этой переменной с помощью `echo`.

```sh
$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```

В каждом из этих мест оболочка ожидает найти команду, каталоги разделённы двоеточиями (`:`). Вы заметите, что каталог `/bin` присутствует, но можно с уверенностью предположить, что нашего текущего каталога в этой переменной нет. Оболочка будет искать `new_script` в каждом из этих каталогов, но не найдёт его и поэтому выдаст ошибку, которую мы видели выше.

Есть три способа решить эту проблему: мы можем переместить `new_script` в один из `PATH` каталогов, мы можем добавить текущий каталог в `PATH`  переменную или изменить способ вызова скрипта. Последний способ самый простой, он требует лишь указать _**текущее местоположение**_ при вызове скрипта с помощью косой черты (`./`).

```sh
$ ./new_script
/bin/bash: ./new_script: Permission denied
```

Сообщение об ошибке изменилось, что указывает на то, что мы добились некоторого прогресса.

## Разрешение на выполнение <a href="#execute_permissions" id="execute_permissions"></a>

Первое, что должен сделать пользователь в этом случае, — это использовать `ls -l` для просмотра файла:

```sh
$ ls -l new_script
-rw-rw-r-- 1 user user 20 Apr 30 12:12 new_script
```

Мы видим, что права доступа к этому файлу по умолчанию установлены на `664`. Мы ещё не установили для этого файла _права на выполнение_.

```sh
$ chmod +x new_script
$ ls -l new_script
-rwxrwxr-x 1 user user 20 Apr 30 12:12 new_script
```

Эта команда предоставила _**всем**_ пользователям права на выполнение. Имейте в виду, что это может представлять угрозу безопасности, но на данный момент это приемлемый уровень разрешений.

```sh
$ ./new_script
Hello World!
```

Теперь мы можем выполнить наш скрипт.

## Определение интерпретатора <a href="#defining_the_interpreter" id="defining_the_interpreter"></a>

Как мы продемонстрировали, мы смогли просто ввести текст в файл, сделать его исполняемым и запустить. `new_script` Функционально это по-прежнему обычный текстовый файл, но нам удалось интерпретировать его с помощью Bash. Но что, если он написан на Perl или Python?

Очень хорошая практика — **указывать тип интерпретатора**, который мы хотим использовать, в первой строке скрипта. Эта строка называется _строкой-разделителем bang line_ или, чаще, _шебангом shebang_. Она указывает системе, как мы хотим, чтобы выполнялся этот файл. Поскольку мы изучаем Bash, мы будем использовать абсолютный путь к исполняемому файлу Bash, снова с помощью `which`:

```sh
$ which bash
/bin/bash
```

Наш шебанг начинается со знака решётки и восклицательного знака, за которыми следует абсолютный путь, указанный выше. Давайте откроем `new_script` в текстовом редакторе и вставим шебанг. Давайте также воспользуемся возможностью и добавим в наш скрипт _комментарий_. Комментарии игнорируются интерпретатором. Они пишутся для других пользователей, которые хотят понять ваш скрипт.

```
#!/bin/bash

# This is our first comment. It is also good practice to document all scripts.

echo "Hello World!"
```

Мы также внесём одно дополнительное изменение в имя файла: сохраним этот файл как `new_script.sh`. Суффикс файла «.sh» никак не влияет на выполнение файла. Обычно скрипты bash помечаются `.sh` или `.bash` для более удобного распознавания, так же как скрипты Python обычно помечаются суффиксом `.py`.

## **Распространенные текстовые редакторы**

Пользователям Linux часто приходится работать в среде, где графические текстовые редакторы недоступны. Поэтому настоятельно рекомендуется хотя бы немного ознакомиться с редактированием текстовых файлов из командной строки. Два наиболее распространённых текстовых редактора — `vi` и `nano`.

**vi**

`vi` Это почтенный текстовый редактор, который по умолчанию установлен почти в каждой существующей системе Linux. `vi` породил клон под названием _vi improved_ или `vim`, который добавляет некоторые функции, но сохраняет интерфейс `vi`. Хотя работа с `vi` может показаться сложной для новичка, этот редактор популярен и любим пользователями, которые изучают его многочисленные функции.

Самое важное различие между `vi` и такими приложениями, как «Блокнот», заключается в том, что `vi` имеет три разных режима. При запуске клавиши H, J, K и L используются для навигации, а не для ввода текста. В этом _режиме навигации_ вы можете нажать I для перехода в _режим редактирования_. В этот момент вы можете вводить текст обычным способом. Чтобы выйти из _режима редактирования_, нажмите Esc для возврата в _режим навигации_. В _режиме навигации_ вы можете нажать : для перехода в _командный режим_. В этом режиме вы можете сохранять, удалять, завершать работу или изменять параметры.

Несмотря на то, что `vi` требует времени на освоение, различные режимы со временем могут позволить опытному пользователю работать более эффективно, чем с другими редакторами.

nano

`nano` Это более новый инструмент, созданный для того, чтобы быть простым и удобным в использовании по сравнению с `vi`. В `nano`  нет разных режимов. Вместо этого пользователь при запуске может начать печатать и использовать Ctrl для доступа к инструментам, расположенным в нижней части экрана.

```
                         [ Welcome to nano.  For basic help, type Ctrl+G. ]
^G Get Help   ^O Write Out  ^W Where Is   ^K Cut Text   ^J Justify    ^C Cur Pos    M-U Undo
^X Exit       ^R Read File  ^\ Replace    ^U Uncut Text ^T To Spell   ^_ Go To Line M-E Redo
```

Текстовые редакторы — это вопрос личных предпочтений, и редактор, который вы выберете, не повлияет на этот урок. Но знакомство с одним или несколькими текстовыми редакторами и привыкание к ним пригодятся вам в будущем.

## Переменные <a href="#variables" id="variables"></a>

Переменные — важная часть любого языка программирования, и Bash не исключение. Когда вы запускаете новый сеанс в терминале, оболочка уже устанавливает для вас некоторые переменные. Переменная `PATH` — пример такой переменной. Мы называем их _**переменными среды**_, потому что они обычно определяют характеристики нашей оболочки. Вы можете изменять и добавлять переменные среды, но пока давайте сосредоточимся на установке переменных внутри нашего скрипта.

Мы изменим наш скрипт, чтобы он выглядел следующим образом:

```sh
#!/bin/bash

# This is our first comment. It is also good practice to comment all scripts.

username=Carol

echo "Hello $username!"
```

В этом случае мы создали _переменную_ под названием `username` и присвоили ей _значение_`Carol`. Обратите внимание, что между именем переменной, знаком равенства и присвоенным значением **нет пробелов**.

В следующей строке мы использовали команду `echo` с переменной, но перед именем переменной **стоит знак доллара (`$`)**. Это важно, так как это указывает оболочке, что мы хотим рассматривать `username` как переменную, а не просто как обычное слово. Вводя `$username` в нашу команду, мы указываем, что хотим выполнить _подстановку_, заменив _имя_ переменной _значением_, присвоенным этой переменной.

Выполняя новый скрипт, мы получаем следующий результат:

```sh
$ ./new_script.sh
Hello Carol!
```

* Имена переменных должны содержать только буквенно-цифровые символы или подчёркивания и чувствительны к регистру. `Username` и `username` будут рассматриваться как отдельные переменные.
* Подстановка переменной также может иметь формат **`${username}`** с добавлением `{ }`. Это тоже допустимо.
* Переменные в Bash имеют _**неявный тип**_ и считаются строками. Это означает, что выполнение математических функций в Bash сложнее, чем в других языках программирования, таких как C/C++.

```sh
#!/bin/bash

# This is our first comment. It is also good practice to comment all scripts.

username=Carol
x=2
y=4
z=$x+$y
echo "Hello $username!"
echo "$x + $y"
echo "$z"
```

```sh
$ ./new_script.sh
Hello Carol!
2 + 4
2+4
```

## Использование кавычек с переменными <a href="#using_quotes_with_variables" id="using_quotes_with_variables"></a>

Давайте внесем следующее изменение в значение нашей переменной `username`:

```sh
#!/bin/bash

# This is our first comment. It is also good practice to comment all scripts.

username=Carol Smith

echo "Hello $username!"
```

Запуск этого скрипта выдаст нам сообщение об ошибке:

```sh
$ ./new_script.sh
./new_script.sh: line 5: Smith: command not found
Hello !
```

Имейте в виду, что Bash — это интерпретатор, и как таковой он _интерпретирует_ наш скрипт построчно. В данном случае он правильно интерпретирует `username=Carol` как присваивание переменной `username` значения `Carol`. Но затем он интерпретирует пробел как указание на конец этого присваивания, а `Smith` как имя команды. Чтобы пробел и имя `Smith` были включены в новое значение нашей переменной, мы заключим имя в двойные кавычки (`"`).

```sh
#!/bin/bash

# This is our first comment. It is also good practice to comment all scripts.

username="Carol Smith"

echo "Hello $username!"
```

```sh
$ ./new_script.sh
Hello Carol Smith!
```

В Bash важно отметить, что двойные и одинарные кавычки (`'`) ведут себя по-разному. Двойные кавычки считаются «слабыми», потому что они позволяют интерпретатору выполнять подстановку внутри кавычек. Одинарные кавычки считаются «сильными», потому что они предотвращают подстановку. Рассмотрим следующий пример:

```sh
#!/bin/bash

# This is our first comment. It is also good practice to comment all scripts.

username="Carol Smith"

echo "Hello $username!"
echo 'Hello $username!'
```

```sh
$ ./new_script.sh
Hello Carol Smith!
Hello $username!
```

Во второй команде `echo` интерпретатор не может заменить `$username` на `Carol Smith`, поэтому вывод воспринимается буквально.

{% hint style="info" %}
Возможно сюда следует добавить информацию об обратных кавычках **\`**
{% endhint %}

## Аргументы <a href="#arguments" id="arguments"></a>

Вы уже знакомы с использованием аргументов в основных утилитах Linux. Например, `rm testfile` содержит как исполняемый файл `rm`, так и один аргумент `testfile`. Аргументы могут передаваться скрипту при выполнении и изменять его поведение. Их легко реализовать.

```bash
#!/bin/bash

# This is our first comment. It is also good practice to comment all scripts.

username=$1

echo "Hello $username!"
```

Вместо того чтобы присваивать значение `username` непосредственно внутри скрипта, мы присваиваем ему значение новой переменной `$1`. Это относится к значению _первого аргумента_.

```bash
$ ./new_script.sh Carol
Hello Carol!
```

Первые **девять** аргументов обрабатываются таким образом. Есть способы обрабатывать более девяти аргументов, но это выходит за рамки данного урока. Мы покажем пример с использованием всего двух аргументов:

```bash
#!/bin/bash

# This is our first comment. It is also good practice to comment all scripts.

username1=$1
username2=$2
echo "Hello $username1 and $username2!"
```

```sh
$ ./new_script.sh Carol Dave
Hello Carol and Dave!
```

При использовании аргументов следует учитывать важный момент: в приведённом выше примере есть два аргумента `Carol` и `Dave`, присвоенные `$1` и `$2` соответственно. Если, например, второй аргумент отсутствует, оболочка не выдаст ошибку. Значение `$2` будет просто _null_, то есть ничем.

```sh
$ ./new_script.sh Carol
Hello Carol and !
```

В нашем случае было бы неплохо добавить в скрипт некоторую логику, чтобы различные _условия_ влияли на _вывод_, который мы хотим получить. Начнём с добавления ещё одной полезной переменной, а затем перейдём к созданию _операторов if_.

## Возврат количества аргументов <a href="#returning_the_number_of_arguments" id="returning_the_number_of_arguments"></a>

В то время как такие переменные, как `$1` и `$2`, содержат значения позиционных аргументов, другая переменная `$#` содержит _количество аргументов_.

```sh
#!/bin/bash

# This is our first comment. It is also good practice to comment all scripts.

username=$1

echo "Hello $username!"
echo "Number of arguments: $#."
```

```sh
$ ./new_script.sh Carol Dave
Hello Carol!
Number of arguments: 2.
```

## Условная логика <a href="#conditional_logic" id="conditional_logic"></a>

Использование условной логики в программировании — обширная тема, которая не будет подробно рассматриваться в этом уроке. Мы сосредоточимся на _синтаксисе_ условных конструкций в Bash, который отличается от большинства других языков программирования.

Давайте начнём с того, чего мы надеемся достичь. У нас есть простой скрипт, который должен выводить приветствие для одного пользователя. Если пользователей больше одного, мы должны выводить сообщение об ошибке.

* _Условие_, которое мы проверяем, — это количество пользователей, содержащееся в переменной `$#`. Мы хотели бы узнать, равно ли значение `$#1`.
* Если условие _выполняется_, мы _выполним действие_, чтобы поприветствовать пользователя.
* Если условие равно _false_, мы выведем сообщение об ошибке.

Теперь, когда логика ясна, мы сосредоточимся на _синтаксисе_, необходимом для реализации этой логики.

```bash
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

Условная логика содержится между `if` и `fi`. Проверяемое условие находится в квадратных скобках `[ ]`, а действие, которое необходимо выполнить, если условие истинно, указывается после `then`. Обратите внимание **на пробелы** между квадратными скобками и содержащейся в них логикой. Если не оставить эти пробелы, возникнут ошибки.

Этот скрипт выведет либо наше приветствие, _либо_ сообщение об ошибке. Но он всегда выведет строку `Number of arguments`.

```bash
$ ./new_script.sh
Please enter only one argument.
Number of arguments: 0.
$ ./new_script.sh Carol
Hello Carol!
Number of arguments: 1.
```

Обратите внимание на оператор `if`. Мы использовали `-eq` для _числового сравнения_. В данном случае мы проверяем, что значение `$#`_равно_ единице. Другие сравнения, которые мы можем выполнить:

`-ne`

Не равно

`-gt`

Больше, чем

`-ge`

Больше или равно

`-lt`

Меньше, чем

`-le`

Меньше или равно

### Упражнения с руководством <a href="#sec.3.3_01-ge" id="sec.3.3_01-ge"></a>

1. Пользователь вводит в свою оболочку следующее:

```bash
$ PATH=~/scripts
$ ls
Command 'ls' is available in '/bin/ls'
The command could not be located because '/bin' is not included in the PATH environment variable.
ls: command not found
```

* Что сделал пользователь?
* Какая команда объединит текущее значение `PATH` с новым каталогом `~/scripts`?

2. Рассмотрим следующий скрипт. Обратите внимание, что он использует `elif` для проверки второго условия:

```bash
>  /!bin/bash

> fruit1 = Apples
> fruit2 = Oranges

  if [ $1 -lt $# ]
  then
    echo "This is like comparing $fruit1 and $fruit2!"
> elif [$1 -gt $2 ]
  then
>   echo '$fruit1 win!'
  else
>   echo "Fruit2 win!"
> done
```

* Строки, отмеченные знаком `>`, содержат ошибки. Исправьте ошибки.

3. Каким будет результат в следующих ситуациях?

<pre class="language-bash"><code class="lang-bash"><strong>$ ./guided1.sh 3 0
</strong></code></pre>

<pre class="language-bash"><code class="lang-bash"><strong>$ ./guided1.sh 2 4
</strong></code></pre>

<pre class="language-bash"><code class="lang-bash"><strong>$ ./guided1.sh 0 1
</strong></code></pre>

## Исследовательские упражнения <a href="#sec.3.3_01-ee" id="sec.3.3_01-ee"></a>

1. Напишите простой скрипт, который будет проверять, переданы ли ровно два аргумента. Если да, выведите аргументы в обратном порядке. Рассмотрим этот пример (примечание: ваш код может выглядеть иначе, но должен приводить к такому же результату):

```bash
if [ $1 == $number ]
then
  echo "True!"
fi
```

2. Этот код правильный, но это не сравнение чисел. Поищите в интернете, чем этот код отличается от использования `-eq`.
3. Существует переменная среды, которая выводит текущий каталог. Используйте `env` для определения имени этой переменной.
4. Используя то, что вы узнали из вопросов 2 и 3, напишите короткий скрипт, который принимает аргумент. Если аргумент передан, проверьте, совпадает ли он с названием текущего каталога. Если да, выведите `yes`. В противном случае выведите `no`.

## Краткие сведения <a href="#sec.3.3_01-su" id="sec.3.3_01-su"></a>

В этом разделе вы узнали:

* Как создавать и выполнять простые скрипты
* Как использовать shebang для указания интерпретатора
* Как устанавливать и использовать переменные внутри скриптов
* Как обрабатывать аргументы в скриптах
* Как создавать `if` операторы
* Как сравнивать числа с помощью числовых операторов

Команды, используемые в упражнениях:

`echo`

Выведите строку в стандартный вывод.

`env`

Выводит все переменные среды в стандартный вывод.

`which`

Выводит абсолютный путь к команде.

`chmod`

Изменяет права доступа к файлу.

Специальные переменные, используемые в упражнениях:

`$1, $2, …​ $9`

Содержит позиционные аргументы, передаваемые скрипту.

`$#`

Содержит количество аргументов, переданных скрипту.

`$PATH`

Содержит каталоги с исполняемыми файлами, используемыми системой.

Операторы, используемые в упражнениях:

`-ne`

Не равно

`-gt`

Больше, чем

`-ge`

Больше или равно

`-lt`

Меньше, чем

`-le`

Меньше или равно

## Ответы на упражнения с руководством <a href="#sec.3.3_01-age" id="sec.3.3_01-age"></a>

1. Пользователь вводит в свою оболочку следующее:

```sh
$ PATH=~/scripts
$ ls
Command 'ls' is available in '/bin/ls'
The command could not be located because '/bin' is not included in the PATH environment variable.
ls: command not found
```

*   Что сделал пользователь?

    Пользователь перезаписал содержимое PATH каталогом `~/scripts`. Команда `ls` больше не может быть найдена, так как она не содержится в PATH. Обратите внимание, что это изменение влияет только на текущий сеанс. Выход из системы и повторный вход вернут изменения.
*   Какая команда объединит текущее значение `PATH` с новым каталогом `~/scripts`?

    `PATH=$PATH:~/scripts`

2. Рассмотрим следующий скрипт. Обратите внимание, что он использует `elif` для проверки второго условия:

```bash
>  /!bin/bash

> fruit1 = Apples
> fruit2 = Oranges

  if [ $1 -lt $# ]
  then
    echo "This is like comparing $fruit1 and $fruit2!"
> elif [$1 -gt $2 ]
  then
>   echo '$fruit1 win!'
  else
>   echo "Fruit2 win!"
> done
```

* Строки, отмеченные знаком `>`, содержат ошибки. Исправьте ошибки.

```bash
#!/bin/bash

fruit1=Apples
fruit2=Oranges

if [ $1 -lt $# ]
then
  echo "This is like comparing $fruit1 and $fruit2!"
elif [ $1 -gt $2 ]
then
  echo "$fruit1 win!"
else
  echo "$fruit2 win!"
fi
```

3. Каким будет результат в следующих ситуациях?

<pre class="language-bash"><code class="lang-bash"><strong>$ ./guided1.sh 3 0
</strong></code></pre>

`Apples win!`

<pre class="language-bash"><code class="lang-bash"><strong>$ ./guided1.sh 2 4
</strong></code></pre>

`Oranges win!`

<pre class="language-bash"><code class="lang-bash"><strong>$ ./guided1.sh 0 1
</strong></code></pre>

`This is like comparing Apples and Oranges!`

## Ответы на исследовательские упражнения <a href="#sec.3.3_01-aee" id="sec.3.3_01-aee"></a>

1.  Напишите простой скрипт, который будет проверять, переданы ли ровно два аргумента. Если да, выведите аргументы в обратном порядке. Рассмотрим этот пример (примечание: ваш код может выглядеть иначе, но должен приводить к такому же результату):

    ```bash
    if [ $1 == $number ]
    then
      echo "True!"
    fi
    ```

```bash
#!/bin/bash

if [ $# -ne 2 ]
then
  echo "Error"
else
  echo "$2 $1"
fi
```

2. Этот код правильный, но это не сравнение чисел. Поищите в интернете, чтобы узнать, чем этот код отличается от использования `-eq`.

Использование `==` позволяет сравнивать _строки_. То есть, если символы обеих переменных совпадают, то условие выполняется.

| `abc == abc` | _верно_ |
| ------------ | ------- |
| `abc == ABC` | _ложь_  |
| `1 == 1`     | _верно_ |
| `1+1 == 2`   | _ложь_  |

Сравнение строк приводит к неожиданному поведению, если вы тестируете числа.

3. Существует переменная среды, которая выводит текущий каталог. Используйте `env` для определения имени этой переменной.

`PWD`

4. Используя то, что вы узнали из вопросов 2 и 3, напишите короткий скрипт, который принимает аргумент. Если аргумент передан, проверьте, совпадает ли он с названием текущего каталога. Если да, выведите `yes`. В противном случае выведите `no`.

```bash
#!/bin/bash

if [ "$1" == "$PWD" ]
then
  echo "yes"
else
  echo "no"
fi
```
