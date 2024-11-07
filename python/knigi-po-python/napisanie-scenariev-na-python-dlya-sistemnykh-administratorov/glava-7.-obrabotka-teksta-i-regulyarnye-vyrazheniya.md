# Глава 7. Обработка текста и регулярные выражения

В этой главе мы познакомимся с обработкой текста и регулярными выражениями. Обработка текста - это процесс создания или изменения текста. В Python есть очень мощная библиотека под названием regular expressions, которая выполняет такие задачи, как поиск и извлечение данных. Вы узнаете, как это делать с файлами, а также научитесь читать и записывать в файлы.

Мы познакомимся с их модулем Python для работы с регулярными выражениями и обработки текста на Python. Мы познакомимся с функциями match(), search(), find all() и sub() модуля re. Мы также познакомимся с переносом текста в Python с помощью модуля textwrap. Наконец, мы познакомимся с символами unicode.

В этой главе мы рассмотрим следующие темы:

* Перенос текста
* Регулярные выражения
* &#x20;Строки Unicode

## Перенос текста&#x20;

В этом разделе мы познакомимся с модулем textwrap. Этот модуль предоставляет класс TextWrapper, который выполняет всю работу. Модуль textwrap используется для форматирования и переноса обычного текста. Этот модуль предоставляет пять основных функций: wrap(), fill(), dedent(), indent(), и shorten(). Сейчас мы рассмотрим эти функции одну за другой.

## Функция wrap()&#x20;

Функция wrap() используется для преобразования целого абзаца в одну строку. Результатом будет список выходных строк.

Синтаксис - textwrap.wrap(text, width)):

* text: текст для переноса.
* width: Максимально допустимая длина строки с переносом. Значение по умолчанию - 70.

Теперь мы рассмотрим пример использования функции wrap(). Создайте скрипт wrap\_example.py и запишите в него следующее содержимое:

```sh
import textwrap

sample_string = '''Python is an interpreted high-level programming language for general-purpose programming. Created by Guido van Rossum and first released in 1991, Python has a design philosophy that emphasizes code readability, notably using significant whitespace.'''

w = textwrap.wrap(text=sample_string, width=30)
print(w)
```

Запустите скрипт:

```sh
student@ubuntu:~/work$ python3 wrap_example.py
['Python is an interpreted high-', 'level programming language for', 'general-purpose programming.', 'Created by Guido van Rossum', 'and first released in', '1991, Python has a design', 'philosophy that emphasizes', 'code readability,  notably', 'using significant whitespace.']
```

В предыдущем примере мы использовали модуль textwrap из Python. Сначала мы создали строку с именем sample\_string. Затем, используя класс TextWrapper, мы указали ширину. Затем, используя функцию wrap, строка была обернута до ширины 30. А затем мы напечатали эти строки.

## Функция fill()&#x20;

Функция fill() работает аналогично textwrap.wrap, за исключением того, что она возвращает данные, объединенные в одну строку, разделенную новой строкой. Эта функция преобразует входные данные в текст и возвращает одну строку, содержащую текст, в который был преобразован текст.

Синтаксис этой функции следующий:

```python
textwrap.fill(text, width)

```

* text: текст для переноса.
* width: Максимально допустимая длина строки с переносом. Значение по умолчанию - 70.

Теперь мы увидим пример использования функции fill(). Создайте скрипт fill\_example.py и напишите в нем следующее содержимое:

```python
mport textwrap

sample_string = '''Python is an interpreted high-level programming language.'''

w = textwrap.fill(text=sample_string, width=50)
print(w)
```

Запустите скрипт и получите следующий результат:

```sh
student@ubuntu:~/work$ python3 fill_example.py
Python is an interpreted high-level programming
language.
```

В предыдущем примере мы использовали функцию fill(). Процедура такая же, как и в wrap(). Сначала мы создали строковую переменную. Затем мы создали объект переноса текста. Затем мы применили функцию fill(). Наконец, мы распечатали выходные данные.

## Функция dedent()

Функция detent() - это еще одна функция модуля textwrap. Эта функция удаляет общие начальные пробелы из каждой строки вашего текста.

Синтаксис этой функции следующий:

```python
textwrap.dedent(text)
```

Теперь мы рассмотрим пример decent(). Создайте dedent\_example.py скрипт и напишите в нем следующее содержимое:

```python
import textwrap

str1 = '''
            Hello Python World \tThis is Python 101
            Scripting language\n
            Python is an interpreted high-level programming language for general-purpose programming.
            '''
print("Original: \n", str1)
print()

t = textwrap.dedent(str1)
print("Dedented: \n", t)
```

Запустите скрипт и получите следующий результат:

```sh
student@ubuntu:~/work$ python3 dedent_example.py

Hello Python World   This is Python 101
Scripting language

Python is an interpreted high-level programming language for general-purpose programming
```

В предыдущем примере мы создали строковую переменную str1. Затем мы использовали textwrap.dedent() для удаления общего начального пробела. Символы табуляции и пробелы считаются пробелами, но они не равны. Таким образом, единственный распространенный пробел, которым в нашем случае является tab, удаляется.

## Функция indent()&#x20;

Функция indent() используется для добавления указанного префикса в начало выделенных строк вашего текста.

Синтаксис этой функции следующий:

```python
          textwrap.indent(text, prefix)
```

* text: Основная строка
* prefix: Префикс который хотим добавить

Создайте indent\_example.py скрипт и напишите в нем следующее содержимое:

```python
import textwrap

str1 = "Python is an interpreted high-level programming language for general-purpose programming. Created by Guido van Rossum and first released in 1991, \n\nPython has a design philosophy that emphasizes code readability, notably using significant whitespace."

w = textwrap.fill(str1, width=30)
i = textwrap.indent(w, '*')
print(i)
```

Запустите скрипт и получите следующий результат:

<pre class="language-sh"><code class="lang-sh"><strong>student@ubuntu:~/work$ python3 indent_example.py
</strong><strong>*Python is an interpreted high-
</strong><strong>*level programming language for
</strong><strong>*general-purpose programming.
</strong><strong>*Created by Guido van Rossum
</strong><strong>*and first released in 1991,
</strong><strong>*Python has a design philosophy
</strong><strong>*that emphasizes code
</strong><strong>*readability, notably using
</strong></code></pre>

В предыдущем примере мы использовали функции fill() и indent() модуля textwrap. Сначала мы использовали метод fill для сохранения данных в переменной w. Затем мы использовали метод indent. После использования функции indent(), каждая строка в выходных данных будет иметь префикс \*. И далее мы напечатали выходные данные.

## Функция shorten()&#x20;

Эта функция модуля textwrap используется для обрезки текста до заданной ширины. Например, если вы хотите создать краткое изложение или предварительный просмотр, используйте функцию shorten(). При использовании функции shorten() все пробелы в вашем тексте будут объединены в один пробел.

Синтаксис этой функции следующий:

```
textwrap.shorten(text, width)
```

Теперь мы рассмотрим пример использования функции shorten(). Создайте скрипт shorten\_example.py и напишите в нем следующее содержимое:

```python
import textwrap

str1 = "Python is an interpreted high-level programming language for general-purpose programming. Created by Guido van Rossum and first released in 1991, \n\nPython has a design philosophy that emphasizes code readability, notably using significant whitespace."

s = textwrap.shorten(str1, width=50)
print(s)
```

Запустите скрипт и получите следующий результат:

```bash
student@ubuntu:~/work$ python3 shorten_example.py
Python is an interpreted high-level [...]
```

В предыдущем примере мы использовали функцию shorten(), чтобы сократить наш текст и подогнать его под заданную ширину. Сначала все пробелы были сокращены до одного пробела. Если результат соответствовал указанной ширине, он отображался на экране. Если нет, то на экране отображались слова указанной ширины, а остальные заменились плейсхолдером.

## Регулярные выражения&#x20;

В этом разделе мы познакомимся с регулярными выражениями в Python. Регулярные выражения - это специализированный язык программирования, который встроен в Python и доступен пользователям через модуль re. Мы можем определить правила для строк, которым мы хотим соответствовать. Используя регулярные выражения, мы можем извлекать определенную информацию из файлов, кода, документов, электронных таблиц и т.д.

В Python регулярное выражение обозначается как re и может быть импортировано через модуль re. Поддержка регулярных выражений для объектов:

* Идентификаторы&#x20;
* Модификаторы
* &#x20;Пробельные символы&#x20;
* Флаги

В следующей таблице перечислены идентификаторы, и для каждого из них есть описание:

| **Identifier** | **Description**                                                                         |
| -------------- | --------------------------------------------------------------------------------------- |
| \w             | Соответствует буквенно-цифровым символам, включая символ подчеркивания (\_).            |
| \W             | Соответствует неалфавитно-цифровым символам, за исключением символа подчеркивания (\_). |
| \d             | Соответствует цифре                                                                     |
| \D             | Соответствует нецифровому значению                                                      |
| \s             | Соответствует пробелу                                                                   |
| \S             | Соответствует чему угодно, кроме пробела                                                |
| .              | Соответствует точке (.)                                                                 |
| \b             | Соответствует любому символу, кроме новой строки                                        |

В следующей таблице перечислены модификаторы, и для каждого из них есть описание:

| **Modifier** | **Description**                           |
| ------------ | ----------------------------------------- |
| ^            | Соответствует началу строки               |
| $            | Соответствует концу строки                |
| ?            | Соответствует 0 или 1 совпадению          |
| \*           | Соответствует 0 или более совпадению      |
| +            | Соответствует 1 или более совпадению      |
| \|           | Соответствует либо либо x либо y x\|y     |
| \[ ]         | Соответствует диапазону                   |
| {x}          | Количество повторений предыдущего символа |

В следующей таблице перечислены пробельные символы, и для каждого из них есть описание:

| **Character** | **Description**  |
| ------------- | ---------------- |
| \s            | Пробел           |
| \t            | Табуляция        |
| \n            | Новая линия      |
| \e            | Escape           |
| \f            | Перевод страницы |
| \r            | Return           |

В следующей таблице перечислены флаги, и для каждого из них есть описание:

| **Flag**      | **Description**                                    |
| ------------- | -------------------------------------------------- |
| re.IGNORECASE | Сопоставление без учета регистра                   |
| re.DOTALL     | Соответствует любому символу, включая новые строки |
| re.MULTILINE  | Многострочное сопоставление                        |
| Re.ASCII      | Сопоставляет escape-код только с символами ASCII   |

Теперь мы рассмотрим несколько примеров регулярных выражений. Мы познакомимся с функциями match(), search(), find all() и sub().

{% hint style="info" %}
Чтобы использовать регулярные выражения в Python, вы должны импортировать модуль re в свои скрипты, чтобы иметь возможность использовать все функции и методы для регулярных выражений.
{% endhint %}

Теперь мы рассмотрим эти функции одну за другой в следующих разделах.

## Функция match()

Функция match() является функцией модуля re. Эта функция сопоставит указанный шаблон re со строкой. Если совпадение найдено, будет возвращен объект match. Объект match будет содержать информацию о совпадении. Если совпадение не найдено, мы получим результат как None. Объект match имеет два метода:

* group(num): Возвращает полное совпадение
* groups(): Возвращает все совпадающие подгруппы в кортеже

Синтаксис этой функции следующий:

```python
re.match(pattern, string)
```

Теперь мы рассмотрим пример re.match(). Создайте скрипт re\_match.py и напишите в нем следующее содержимое:

```python
import re

str_line = "This is python tutorial. Do you enjoy learning python ?"
obj = re.match(r'(.*) enjoy (.*?) .*', str_line)
if obj:
            print(obj.groups())
```

Запустите скрипт и получите следующий результат:

```bash
student@ubuntu:~/work$ python3 re_match.py
('This is python tutorial. Do you', 'learning'
```

В предыдущем скрипте мы импортировали модуль re для использования регулярных выражений в Python. Затем мы создали строку str\_line. Затем мы создали объект match и сохранили в нем результат сопоставления с шаблоном. В этом примере шаблон (._) enjoy (._?) .\* будет печатать все перед ключевым словом enjoy и только одно слово после ключевого слова enjoy. Далее мы использовали метод groups() для сопоставления объекта. Он выведет все совпадающие подстроки в виде кортежа. Таким образом, результат, который вы получите, будет таким: ('This is python tutorial. Do you', 'learning').

## Функция search()

Функция search() модуля re выполнит поиск по строке. Она будет искать любое местоположение для указанного шаблона re. Функция search() возьмет шаблон и текст и выполнит поиск по указанной нами строке в поисках совпадения. Она вернет объект match, когда будет найдено совпадение. Она вернет None, если совпадение не найдено. Объект match имеет два метода:

group(num): Возвращает полное совпадение groups(): Возвращает все совпадающие подгруппы в кортеже Синтаксис этой функции следующий:

```python
re.search(pattern, string)
```

Создайте re\_search.py скрипт и напишите в нем следующее содержимое:

```python
import re

pattern = ['programming', 'hello']
str_line = 'Python programming is fun'
for p in pattern:
            print("Searching for %s in %s" % (p, str_line))
            if re.search(p, str_line):
                        print("Match found")
            else:
                        print("No match found")
```

Запустите скрипт и получите следующий результат:

```bash
student@ubuntu:~/work$ python3 re_search.py
Searching for programming in Python programming is fun
Match found
Searching for hello in Python programming is fun
No match found
```

В предыдущем примере мы использовали метод поиска() объекта match, чтобы найти его шаблон. После импорта модуля re мы указали шаблон в списке. В этом списке мы написали две строки: programming и hello. Затем мы создали строку: "Python programming is fun". Мы написали цикл for, который будет проверять заданный шаблон один за другим. Если совпадение найдено, будет выполнен блок if. Если совпадение не найдено, будет выполнен блок else.

## Функция findall()

Это один из методов объекта match. Метод findall() находит все совпадения и затем возвращает их в виде списка строк. Каждый элемент списка представляет собой совпадение. Этот метод выполняет поиск шаблона без перекрытия.

Создайте re\_findall\_example.py скрипт и напишите в нем следующее содержимое:

```python
import re

pattern = 'Red'
colors = 'Red, Blue, Black, Red, Green'
p = re.findall(pattern, colors)
print(p)

str_line = 'Peter Piper picked a peck of pickled peppers. How many pickled peppers did Peter Piper pick?'
pt = re.findall('pe\w+', str_line)
pt1 = re.findall('pic\w+', str_line)
print(pt)
print(pt1)

line = 'Hello hello HELLO bye'
p = re.findall('he\w+', line, re.IGNORECASE)
print(p)
```

Запустите скрипт и получите следующий результат:

<pre class="language-bash"><code class="lang-bash"><strong>student@ubuntu:~/work$ python3 re_findall_example.py
</strong><strong>['Red', 'Red']
</strong><strong>['per', 'peck', 'peppers', 'peppers', 'per']
</strong><strong>['picked', 'pickled', 'pickled', 'pick']
</strong><strong>['Hello', 'hello', 'HELLO']
</strong></code></pre>

\
В предыдущем сценарии мы написали три примера метода findall(). В первом примере мы определили шаблон и строку. Мы нашли этот шаблон из строки с помощью метода findall() и затем распечатали его. Во втором примере мы создали строку и нашли слова, первые две буквы которых - pe, используя функцию findall(), а затем напечатали их. Мы получим список слов, первые две буквы которых - pe. Кроме того, мы нашли слова, первые три буквы которых - pic, и затем напечатали их.&#x20;

Здесь мы также получим список строк. В третьем примере мы создали строку, в которой указали hello в верхнем и нижнем регистре, а также слово: bye. Используя findall(), мы находим слова, первые две буквы которых - he. Также в findall() мы использовали re.Флаг IGNORECASE, который будет игнорировать регистр слов и выводить их.

## Функция sub()&#x20;

Это одна из наиболее важных функций модуля re. Функция sub() используется для замены их шаблона на указанную замену. Она заменит все вхождения шаблона re на строку замены. Синтаксис следующий:

```python
re.sub(pattern, repl_str, string, count=0)
```

* pattern: Шаблон
* repl\_str: Строка замены
* string: Основная строка
* count: Значение по умолчанию равно 0, что означает замену всех вхождений.

Теперь мы собираемся создать re\_sub.py скрипт и записать в него следующий контент:

```python
import re

str_line = 'Peter Piper picked a peck of pickled peppers. How many pickled peppers did Peter Piper pick?'

print("Original: ", str_line)
p = re.sub('Peter', 'Mary', str_line)
print("Replaced: ", p)

p = re.sub('Peter', 'Mary', str_line, count=1)
print("Replacing only one occurrence of Peter… ")
print("Replaced: ", p)
```

Запустите скрипт и получите следующий результат:

```bash
student@ubuntu:~/work$ python3 re_sub.py
Original:  Peter Piper picked a peck of pickled peppers. How many pickled peppers did Peter Piper pick?
Replaced:  Mary Piper picked a peck of pickled peppers. How many pickled peppers did Mary Piper pick?
Replacing only one occurrence of Peter...
Replaced:  Mary Piper picked a peck of pickled peppers. How many pickled peppers did Peter Piper pick?
```

В предыдущем примере мы использовали sub(), чтобы заменить шаблон на указанную строку замены. Мы заменили Peter на Mary. Таким образом, все вхождения Peter будут заменены на Mary. Далее мы также включили параметр count. Мы упомянули count=1: это означает, что будет заменено только одно вхождение Peter, а остальные вхождения Peter останутся прежними.

Теперь мы познакомимся с функцией subn() модуля re. Функция subn() работает так же, как и функция sub(), но с дополнительными функциями. Функция subn() вернет кортеж, содержащий новую строку и количество выполненных замен. Давайте рассмотрим пример использования функции subn(). Создайте скрипт re\_subn.py и запишите в него следующее содержимое:

```python
import re

print("str1:- ")
str1 = "Sky is blue. Sky is beautiful."

print("Original: ", str1)
p = re.subn('beautiful', 'stunning', str1)
print("Replaced: ", p)
print()

print("str_line:- ")
str_line = 'Peter Piper picked a peck of pickled peppers. How many pickled peppers did Peter Piper pick?'

print("Original: ", str_line)
p = re.subn('Peter', 'Mary', str_line)
print("Replaced: ", p)
```

Запустите скрипт и получите следующий результат:

```
student@ubuntu:~/work$ python3 re_subn.py
str1:-
Original:  Sky is blue. Sky is beautiful.
Replaced:  ('Sky is blue. Sky is stunning.', 1)
 
str_line:-
Original:  Peter Piper picked a peck of pickled peppers. How many pickled peppers did Peter Piper pick?
Replaced:  ('Mary Piper picked a peck of pickled peppers. How many pickled peppers did Mary Piper pick?', 2)
```

В предыдущем примере мы использовали функцию subn() для замены шаблона. В результате мы получили кортеж, содержащий замененную строку и количество замен.\


## Строки в Юникоде

В этом разделе мы узнаем о том, как печатать строки в формате Unicode в Python. Python очень просто обрабатывает строки в формате Unicode. Тип string на самом деле содержит строки в формате Unicode, а не последовательность байтов.

Запустите консоль python3 в вашей системе и начните писать следующее:

```python
GCC 8.0.1 20180414 (experimental) [trunk revision 259383]] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> 
>>> print ('\u2713')

>>> print ('\u2724')

>>> print ('\u2750')

>>> print ('\u2780')

>>> chinese = '\u4e16\u754c\u60a8\u597d!
>>> chinese
 ----- (Meaning “Hello world!”)
>>>
>>> s = '\u092E\u0941\u0902\u092C\u0908'
>>> s
'मुंबई'                            ------(Unicode translated in Marathi)
>>>
>>> s = '\u10d2\u10d0\u10db\u10d0\u10e0\u10ef\u10dd\u10d1\u10d0'
>>> s
'გამარჯობა'                 ------(Meaning “Hello” in Georgian)
>>>
>>> s = '\u03b3\u03b5\u03b9\u03b1\u03c3\u03b1\u03c2'
>>> s
'γειασας'                     ------(Meaning “Hello” in Greek)
>>>
```

## Кодовая точка юникод

**Кодовая точка в Юникоде** — это уникальный номер, который представляет определённый символ или знак в стандарте Юникода. Этот стандарт включает в себя огромное количество символов из разных языков и систем письма, а также специальные символы и эмодзи.

В этом разделе мы познакомимся с кодовой точкой unicode. В Python есть мощная встроенная функция с именем ord() для получения кодовой точки Unicode из заданного символа. Итак, давайте рассмотрим пример получения кодовой точки Unicode из символа, как показано в следующем коде:

```python
>>> str1 = u'Office'
>>> for char in str1:
... print('U+%04x' % ord(char))
...
U+004f
U+0066
U+0066
U+0069
U+0063
U+0065
>>> str2 = 
>>> for char in str2:
... print('U+%04x' % ord(char))
...
U+4e2d
U+6587
```

## Кодирование

Преобразование кодовой точки Unicode в байтовую строку называется кодированием. Итак, давайте рассмотрим пример кодирования кодовой точки Unicode, как показано в следующем коде:

```python
>>> str = u'Office'
>>> enc_str = type(str.encode('utf-8'))
>>> enc_str
<class 'bytes'>
```

## Декодирование&#x20;

Преобразование байтовой строки в кодовую точку Unicode называется декодированием. Итак, давайте рассмотрим пример того, как декодировать байтовую строку, чтобы получить кодовую точку Unicode, как показано в следующем коде:

```python
>>> str = bytes('Office', encoding='utf-8')
>>> dec_str = str.decode('utf-8')
>>> dec_str
'Office'
```

## Как избежать ошибки UnicodeDecodeError&#x20;

Ошибка UnicodeDecodeError возникает всякий раз, когда байтовые строки не удается расшифровать в кодовые точки Unicode. Чтобы избежать этого исключения, мы можем передать replace, backslashreplace или ignore в качестве аргумента error в decode, как показано здесь:

```python
>>> str = b"\xaf"
>>> str.decode('utf-8', 'strict')
    Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeDecodeError: 'utf-8' codec can't decode byte 0xaf in position 0: invalid start byte

>>> str.decode('utf-8', "replace")
'\ufffd'
>>> str.decode('utf-8', "backslashreplace")
'\\xaf'
>>> str.decode('utf-8', "ignore")
' '
```

## Резюме

&#x20;В этой главе мы узнали о регулярных выражениях, с помощью которых мы можем определить правила для набора строк, которым мы хотим соответствовать. Мы узнали о четырех функциях модуля re: match(), search(), findall() и sub().

Мы узнали о модуле textwrap, который используется для форматирования и переноса обычного текста. Мы также узнали о функциях wrap(), fill(), dedent(), indent() и shorten() модуля textwrap. Наконец, мы узнали о символах Unicode и о том, как печатать строки Unicode в Python.

## Вопросы

1. Что такое регулярное выражение в Python?

**Регулярное выражение в Python** — это последовательность символов, которая используется для поиска, замены или извлечения определённых фрагментов текста. Регулярные выражения позволяют задавать сложные шаблоны поиска с использованием специальных символов и операторов.

В Python регулярные выражения представлены модулем `re`, который содержит функции для работы с ними. С помощью регулярных выражений можно выполнять различные операции над текстом, такие как поиск подстрок, замена строк, проверка соответствия шаблону и т. д.
