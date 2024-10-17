# Отладка

**Отладка** - это процесс, который устраняет проблемы, возникающие в вашем коде и препятствующие правильной работе вашего программного обеспечения. В Python отладка очень проста. Отладчик Python устанавливает условные точки останова и выполняет отладку исходного кода по одной строке за раз. Мы будем отлаживать наши скрипты на Python с помощью модуля pdb, который присутствует в стандартной библиотеке Python.

## &#x20;Методы отладки Python

Для улучшения отладки программ на Python доступны различные методы. Мы рассмотрим четыре метода отладки на Python:

* оператор print(): Это самый простой способ узнать, что именно происходит, чтобы вы могли проверить, что было выполнено.
* ведение журнала logging(): Это похоже на print(), но с большим количеством контекстной информации, чтобы вы могли полностью ее понять.
* отладчик pdb: это широко используемый метод отладки. Преимущество использования pdb заключается в том, что вы можете использовать pdb из командной строки, в интерпретаторе и в программе.
* Отладчик IDE: IDE имеет встроенный отладчик. Это позволяет разработчикам выполнять свой код, а затем разработчик может проверять его во время выполнения программы.

## &#x20;Обработка ошибок (exception handling)

В этом разделе мы узнаем, как Python обрабатывает исключения. Но сначала разберемся, что такое exception? Исключение - это ошибка, возникающая во время выполнения программы. Всякий раз, когда возникает какая-либо ошибка, Python генерирует исключение, которое будет обрабатываться с помощью блока try...except. Некоторые исключения не могут быть обработаны программами, поэтому они приводят к появлению сообщений об ошибках. Сейчас мы рассмотрим несколько примеров исключений.

В вашем терминале запустите интерактивную консоль python3, и мы увидим несколько примеров исключений:

```
student@ubuntu:~$ python3
Python 3.5.2 (default, Nov 23 2017, 16:37:01)
[GCC 5.4.0 20160609] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>>
>>> 50 / 0


Traceback (most recent call last):
 File "<stdin>", line 1, in <module>
ZeroDivisionError: division by zero
>>>
>>> 6 + abc*5
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'abc' is not defined
>>>
>>> 'abc' + 2
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: Can't convert 'int' object to str implicitly
>>>
>>> import abcd
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  ImportError: No module named 'abcd'
>>> 
```

Вот несколько примеров исключений. Теперь мы посмотрим, как мы можем обрабатывать исключения.

Всякий раз, когда в вашей программе на Python возникают ошибки, вызываются исключения. Мы также можем принудительно вызвать исключение, используя ключевое слово raise.

Теперь мы увидим блок try...except, который обрабатывает исключение. В блоке try мы напишем код, который может генерировать исключение. В блоке except мы напишем решение для этого исключения.

Синтаксис для try...except выглядит следующим образом:

```
try:
            statement(s)
except:
            statement(s)
```

Блок try может содержать несколько инструкций except. Мы также можем обрабатывать конкретные исключения, введя имя исключения после ключевого слова except. Синтаксис для обработки конкретного исключения следующий:

```
try:
            statement(s)
except exception_name:
            statement(s)
```

Мы собираемся создать exception\_example.py скрипт для перехвата ошибки ZeroDivisionError. Напишите в своем скрипте следующий код:

```
a = 35
b = 57
try:
            c = a + b
            print("The value of c is: ", c)
            d = b / 0
            print("The value of d is: ", d)
 
except:
            print("Division by zero is not possible")
 
print("Out of try...except block")
```

Запустите скрипт следующим образом, и вы получите следующий результат:

```
student@ubuntu:~$ python3 exception_example.py
The value of c is:  92
Division by zero is not possible
Out of try...except block
```

## &#x20;Инструменты для отладки

В Python поддерживается множество инструментов отладки:

* winpdb
* pydev
* pydb
* pdb
* gdb
* pyDebug

В этом разделе мы познакомимся с отладчиком pdb Python. модуль pdb является частью стандартной библиотеки Python и всегда доступен для использования.

## &#x20;Отладчик pdb

Модуль pdb используется для отладки программ на Python. Программы на Python используют pdb interactive source code debugger для отладки программ. pdb устанавливает точки останова, проверяет фреймы стека и выводит список исходного кода.

Теперь мы узнаем о том, как мы можем использовать pdb-отладчик. Существует три способа использования этого отладчика:

* В интерпретаторе
* &#x20;Из командной строки&#x20;
* В скрипте на Python

Мы собираемся создать pdb\_example.py скрипт и добавить в него следующий контент:

```
class Student:
            def __init__(self, std):
                        self.count = std
 
            def print_std(self):
                        for i in range(self.count):
                                    print(i)
                        return
if __name__ == '__main__':
            Student(5).print_std()
```

Используя этот скрипт в качестве примера для изучения отладки Python, мы подробно рассмотрим, как мы можем запустить отладчик.

## &#x20;Внутри интерпретатора

Чтобы запустить отладчик из интерактивной консоли Python, мы используем run() или runeval().

Запустите интерактивную консоль python3. Для запуска консоли выполните следующую команду:

```
$ python3
```

Импортируем имя нашего сценария pdb\_example и модуль pdb. Теперь мы собираемся использовать функцию run() и передаем в качестве аргумента функции run() строковое выражение, которое будет вычислено самим интерпретатором Python:

```
student@ubuntu:~$ python3
Python 3.5.2 (default, Nov 23 2017, 16:37:01)
[GCC 5.4.0 20160609] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>>
>>> import pdb_example
>>> import pdb
>>> pdb.run('pdb_example.Student(5).print_std()')
> <string>(1)<module>()
(Pdb)
```

Чтобы продолжить отладку, введите continue после запроса (Pdb) и нажмите Enter. Если вы хотите узнать, какие опции мы можем использовать при этом, то после запроса (Pdb) дважды нажмите клавишу Tab.

Теперь, после ввода continue, мы получим следующий результат:

```
student@ubuntu:~$ python3
Python 3.5.2 (default, Nov 23 2017, 16:37:01)
[GCC 5.4.0 20160609] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>>
>>> import pdb_example
>>> import pdb
>>> pdb.run('pdb_example.Student(5).print_std()')
> <string>(1)<module>()
(Pdb) continue
0
1
2
3
4
>>> 
```

## &#x20;Из командной строки

Самый простой и понятный способ запустить отладчик - это из командной строки. Наша программа будет использоваться в качестве входных данных для отладчика. Вы можете использовать отладчик из командной строки следующим образом:

```
$ python3 -m pdb pdb_example.py
```

Когда вы запустите отладчик из командной строки, будет загружен исходный код, и он остановит выполнение в первой найденной строке. Введите continue, чтобы продолжить отладку. Вот результат:

```
student@ubuntu:~$ python3 -m pdb pdb_example.py
> /home/student/pdb_example.py(1)<module>()
-> class Student:
(Pdb) continue
0
1
2
3
4
The program finished and will be restarted
> /home/student/pdb_example.py(1)<module>()
-> class Student:
(Pdb)
```

## &#x20;Внутри скрипта на Python

Предыдущие два метода позволяют запустить отладчик в начале работы программы на Python. Но третий метод лучше всего подходит для длительных процессов. Чтобы запустить отладчик в скрипте, используйте set\_trace().

Теперь измените свой файл pdb\_example.py следующим образом:

```
import pdb
class Student:
            def __init__(self, std):
                        self.count = std
 
            def print_std(self):
                        for i in range(self.count):
                                    pdb.set_trace()
                                    print(i)
                        return
 
if __name__ == '__main__':
            Student(5).print_std()
```

Теперь запустите программу следующим образом:

```
student@ubuntu:~$ python3 pdb_example.py
> /home/student/pdb_example.py(10)print_std()
-> print(i)
(Pdb) continue
0
> /home/student/pdb_example.py(9)print_std()
-> pdb.set_trace()
(Pdb)
```

set\_trace() - это функция Python, поэтому вы можете вызвать ее в любой момент вашей программы.

Итак, вот три способа, с помощью которых вы можете запустить отладчик.

## &#x20;Отладка сбоев в основной программе

&#x20;В этом разделе мы познакомимся с модулем трассировки. Модуль трассировки помогает отслеживать выполнение программы. Таким образом, всякий раз, когда ваша программа на Python выходит из строя, мы можем понять, где она выходит из строя. Мы можем использовать модуль трассировки, импортировав его в ваш скрипт, а также из командной строки.

Теперь мы создадим скрипт с именем trace\_example.py и запишем в скрипт следующее содержимое:

```
class Student:
            def __init__(self, std):
                        self.count = std
 
            def go(self):
                        for i in range(self.count):
                                    print(i)
                        return
if __name__ == '__main__':
            Student(5).go()
```

Результат будет выглядеть следующим образом:

```
student@ubuntu:~$ python3 -m trace --trace trace_example.py
 --- modulename: trace_example, funcname: <module>
trace_example.py(1): class Student:
 --- modulename: trace_example, funcname: Student
trace_example.py(1): class Student:
trace_example.py(2):   def __init__(self, std):
trace_example.py(5):   def go(self):
trace_example.py(10): if __name__ == '__main__':
trace_example.py(11):             Student(5).go()
 --- modulename: trace_example, funcname: init
trace_example.py(3):               self.count = std
 --- modulename: trace_example, funcname: go
trace_example.py(6):               for i in range(self.count):
trace_example.py(7):                           print(i)
0
trace_example.py(6):               for i in range(self.count):
trace_example.py(7):                           print(i)
1
trace_example.py(6):               for i in range(self.count):
trace_example.py(7):                           print(i)
2
trace_example.py(6):               for i in range(self.count):
trace_example.py(7):                           print(i)
3
trace_example.py(6):               for i in range(self.count):
trace_example.py(7):                           print(i)
4
```

Таким образом, используя trace -трассировку в командной строке, разработчик может отслеживать программу построчно. Таким образом, всякий раз, когда программа выходит из строя, разработчик будет знать, в каком месте произошел сбой.















