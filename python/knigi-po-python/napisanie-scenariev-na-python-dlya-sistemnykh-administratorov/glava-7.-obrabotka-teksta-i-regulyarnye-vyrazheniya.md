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

