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
