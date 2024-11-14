# Глава 8. Документация и отчетность

В этой главе вы узнаете, как документировать информацию и создавать отчеты с помощью Python. Вы также узнаете, как принимать входные данные с помощью скриптов Python и как печатать выходные данные. Писать скрипты для получения электронных писем на Python проще. Вы узнаете, как форматировать информацию.

В этой главе вы узнаете о следующем:

* Стандартный ввод и вывод информации&#x20;
* Форматирование информации&#x20;
* Отправка электронных писем

## Стандартный ввод и вывод

В этом разделе мы познакомимся с вводом и выводом в Python. Мы узнаем о stdin и stdout, а также о функции input().

stdin и stdout являются файлоподобными объектами. Эти объекты предоставляются операционной системой. Всякий раз, когда пользователь запускает программу в интерактивном сеансе, stdin используется в качестве входных данных, а stdout - в качестве пользовательского терминала. Поскольку stdin - это объект, подобный файлу, мы должны считывать данные из stdin, а не считывать данные во время выполнения. для вывода используется стандартный вывод. Он используется в качестве вывода для выражений и функции print(), а также в качестве запроса для функции input().

Теперь мы рассмотрим пример стандартного ввода-вывода. Для этого создайте скрипт, stdin\_stdout\_example.py и запишите в него следующее содержимое:

```python
import sys

print("Enter number1: ")
a = int(sys.stdin.readline())

print("Enter number2: ")
b = int(sys.stdin.readline())

c = a + b
sys.stdout.write("Result: %d " % c)
```

Запустите скрипт и получите следующий вывод:

```bash
student@ubuntu:~/work$ python3 stdin_stdout_example.py
Enter number1:
10
Enter number2:
20
Result: 30
```

В предыдущем примере мы использовали stdin и stdout для ввода и отображения выходных данных. Функция sys.stdin.readline() будет считывать данные из stdin. Она запишет данные.

Теперь мы познакомимся с функциями input() и print(). Функция input() используется для получения входных данных от пользователя. У функции есть необязательный параметр: строка запроса.

Синтаксис:

```python
input(prompt)
```

Функция input() возвращает строковое значение. Если вам нужно числовое значение, просто введите ключевое слово 'int' перед input(). Вы можете сделать это следующим образом:

```python
int(input(prompt))
```

Аналогично, вы можете написать float для значений с плавающей точкой. Теперь мы рассмотрим пример. Создайте скрипт input\_example.py и напишите в нем следующий код:

```python
str1 = input("Enter a string: ")
print("Entered string is: ", str1)
print()

a = int(input("Enter the value of a: "))
b = int(input("Enter the value of b: "))
c = a + b
print("Value of c is: ", c)
print()

num1 = float(input("Enter num 1: "))
num2 = float(input("Enter num 2: "))
num3 = num1/num2
print("Value of num 3 is: ", num3)
```

Запустите скрипт и получите следующий вывод:

```bash
student@ubuntu:~/work$ python3 input_example.py
Output:
Enter a string: Hello
Entered string is:  Hello
Enter the value of a: 10
Enter the value of b: 20
Value of c is:  30 
Enter num 1: 10.50
Enter num 2: 2.0
Value of num 3 is:  5.25
```

В предыдущем примере мы использовали функцию input() для трех разных значений. Первое - для строки, второе - для целого значения и третье - для значения с плавающей точкой. Чтобы использовать input() для integer и float, мы должны использовать функции преобразования типов int() и float() для преобразования полученной строки в integer и float соответственно.

Теперь для вывода данных используется функция print(). Нам нужно ввести список аргументов, разделенных запятыми. В input\_example.py для получения выходных данных мы использовали функцию print(). Используя функцию print(), вы можете просто записать данные на свой экран, заключив их в " " или " '. Чтобы получить доступ только к значению, просто введите имя переменной в функцию print(). Если вы хотите записать какой-либо текст, а также получить доступ к значению в той же функции print(), то разделите эти два действия, поставив между ними запятую.

Мы рассмотрим простой пример для функции print(). Создайте скрипт print\_example.py и напишите в нем следующее содержимое:

```python
# printing a simple string on the screen.
print("Hello Python")

# Accessing only a value.
a = 80
print(a)

# printing a string on screen as well as accessing a value.
a = 50
b = 30
c = a/b
print("The value of c is: ", c)
```

Запустите скрипт и получите следующий вывод:

```bash
student@ubuntu:~/work$ python3 print_example.py
Hello Python
80
The value of c is:  1.6666666666666667
```

В предыдущем примере сначала мы просто напечатали строку на экране. Затем мы просто получили доступ к значению a и напечатали его на экране. Наконец, мы ввели значения a и b, затем добавили их и сохранили результат в переменной c, а затем напечатали инструкцию и получили доступ к значению из той же функции print().

## Форматирование информации

В этом разделе мы познакомимся с форматированием строк. Мы научимся форматировать информацию двумя способами: с помощью метода строк format() и с помощью оператора %.

Сначала мы изучим форматирование строк с помощью метода  format(). Этот метод класса string позволяет нам выполнять форматирование значений. Он также позволяет нам выполнять замену переменных. Это позволит объединить элементы с помощью позиционных аргументов.

Теперь мы узнаем, как выполнить это форматирование с помощью форматировщиков. Строка, для которой вызывается этот метод, может содержать буквенный текст или заменяющие поля, разделенные фигурными скобками {}. При форматировании строки можно использовать несколько пар {}. Это заменяющее поле содержит либо индекс аргумента, либо имя аргумента. В результате вы получите копию строки, в которой каждое заменяющее поле заменено строковым значением аргумента.

Теперь мы рассмотрим пример форматирования строки.

Создайте format\_example.py скрипт и напишите в нем следующее содержимое:

```python
# Using single formatter
print("{}, My name is John".format("Hi"))
str1 = "This is John. I am learning {} scripting language."
print(str1.format("Python"))

print("Hi, My name is Sara and I am {} years old !!".format(26))

# Using multiple formatters
str2 = "This is Mary {}. I work at {} Resource department. I am {} years old !!"
print(str2.format("Jacobs", "Human", 30))

print("Hello {}, Nice to meet you. I am {}.".format("Emily", "Jennifer"))
```

Запустите скрипт:

```bash
student@ubuntu:~/work$ python3 format_example.py
Output:
Hi, My name is John
This is John. I am learning Python scripting language.
Hi, My name is Sara and I am 26 years old !!
This is Mary Jacobs. I work at Human Resource department. I am 30 years old !!
Hello Emily, Nice to meet you. I am Jennifer.
```

В предыдущем примере мы форматировали строку с помощью метода format() класса string, используя один и несколько форматировщиков.

Теперь мы познакомимся с форматированием строки с помощью оператора %. Существуют символы формата, используемые с оператором %. Вот некоторые часто используемые символы:

* %d: Десятичное число
* %s: Строка
* %f: Число с плаваюзей запятой
* %c: Символ

Теперь мы рассмотрим пример. Создайте скрипт string\_formatting.py и напишите в нем следующее содержимое:

```python
# Basic formatting
a = 10
b = 30
print("The values of a and b are %d %d" % (a, b))
c = a + b
print("The value of c is %d" % c)

str1 = 'John'
print("My name is %s" % str1)

x = 10.5
y = 33.5
z = x * y
print("The value of z is %f" % z)
print()

# aligning
name = 'Mary'
print("Normal: Hello, I am %s !!" % name)

print("Right aligned: Hello, I am %10s !!" % name)

print("Left aligned: Hello, I am %-10s !!" % name)
print()

# truncating
print("The truncated string is %.4s" % ('Examination'))
print()

# formatting placeholders
students = {'Name' : 'John', 'Address' : 'New York'}
print("Student details: Name:%(Name)s Address:%(Address)s" % students)
```

Запустите скрипт, и вы получите следующий результат:

```bash
student@ubuntu:~/work$ python3 string_formatting.py
The values of a and b are 10 30
The value of c is 40
My name is John
The value of z is 351.750000 
Normal: Hello, I am Mary !!
Right aligned: Hello, I am       Mary !!
Left aligned: Hello, I am Mary       !!
 The truncated string is Exam
 Student details: Name:John Address:New York
```

В предыдущем примере мы использовали оператор % для форматирования строк: %d для чисел, %s для строк и %f для чисел с плавающей точкой. Затем мы выровняли строку по левому и правому краю. Мы также узнали, как обрезать строку с помощью оператора %. %.4s будет отображать только первые четыре символа. Далее мы создали словарь с именем students и ввели пары "Имя" и "Значение ключа адреса". Затем мы поместили наши ключевые имена после оператора %, чтобы получить строки.

## Отправка электронной почты

В этом разделе мы узнаем об отправке электронного письма с помощью скрипта на Python. Для этого в Python есть модуль smtplib. Модуль smtplib в Python предоставляет объект сеанса SMTP-клиента, который используется для отправки электронного письма на любой компьютер, подключенный к Интернету с помощью SMTP-прослушивателя.

Мы рассмотрим пример. В этом примере мы отправим получателям электронное письмо, содержащее простой текст из Gmail.

Создайте send\_email.py скрипт и напишите в нем следующее содержимое:

```python
import smtplib
from email.mime.text import MIMEText
import getpass

host_name = 'smtp.gmail.com'
port = 465

u_name = 'username/emailid'
password = getpass.getpass()
sender = 'sender_name'
receivers = ['receiver1_email_address', 'receiver2_email_address']

text = MIMEText('Test mail')
text['Subject'] = 'Test'
text['From'] = sender
text['To'] = ', '.join(receivers)

s_obj = smtplib.SMTP_SSL(host_name, port)
s_obj.login(u_name, password)
s_obj.sendmail(sender, receivers, text.as_string())
s_obj.quit()
print("Mail sent successfully")
```

Запустите скрипт:

```bash
student@ubuntu:~/work$ python3 send_text.py
```

Вывод:

```bash
Password:
Mail sent successfully
```

В предыдущем примере мы отправили электронное письмо с нашего идентификатора Gmail получателям. В переменной username будет сохранен ваш идентификатор электронной почты. В переменной password вы можете либо ввести свой пароль, либо запросить пароль с помощью модуля getpass. Здесь мы запрашиваем пароль. Далее в переменной sender будет указано ваше имя. Теперь мы собираемся отправить это электронное письмо нескольким получателям. Затем мы указали тему, от кого и кому для этого электронного письма. Затем в login() мы указали наши переменные имени пользователя и пароля. Далее, в sendmail() мы указали отправителя, получателей и текстовые переменные. Итак, используя этот процесс, мы успешно отправили электронное письмо.

Теперь мы рассмотрим еще один пример отправки электронного письма с вложением. В этом примере мы собираемся отправить получателю изображение. Мы собираемся отправить это письмо через Gmail. Создайте скрипт send\_email\_attachment.py и напишите в нем следующее содержимое:

```python
import os
import smtplib
from email.mime.text import MIMEText
from email.mime.image import MIMEImage
from email.mime.multipart import MIMEMultipart
import getpass

host_name = 'smtp.gmail.com'
port = 465

u_name = 'username/emailid'
password = getpass.getpass()
sender = 'sender_name'
receivers = ['receiver1_email_address', 'receiver2_email_address']

text = MIMEMultipart()
text['Subject'] = 'Test Attachment'
text['From'] = sender
text['To'] = ', '.join(receivers)

txt = MIMEText('Sending a sample image.')
text.attach(txt)

f_path = '/home/student/Desktop/mountain.jpg'
with open(f_path, 'rb') as f:
    img = MIMEImage(f.read())

img.add_header('Content-Disposition',
               'attachment',
               filename=os.path.basename(f_path))

text.attach(img)

server = smtplib.SMTP_SSL(host_name, port)
server.login(u_name, password)
server.sendmail(sender, receivers, text.as_string())
print("Email with attachment sent successfully !!")
server.quit()
```

Запустите скрипт:

```bash
student@ubuntu:~/work$ python3 send_email_attachment.py
```

Вывод:

```
Password:
Email with attachment sent successfully!!
```

В предыдущем примере мы отправили изображение в качестве вложения получателям. Мы указали идентификаторы электронной почты отправителя и получателей. Далее, в поле f\_path, мы указали путь к изображению, которое мы отправили в качестве вложения. Затем мы отправили это изображение в качестве приложения получателю.

{% hint style="info" %}
В двух предыдущих примерах – send\_text.py и send\_email\_attachment.py – мы отправляли электронные письма через Gmail. Вы можете отправлять через любых других поставщиков электронной почты. Чтобы использовать любого другого поставщика электронной почты, просто введите имя этого поставщика в поле host\_name. Не забудьте добавить smtp перед этим. В этом примере мы использовали smtp.gmail.com; для Yahoo! вы можете использовать smtp.mail.yahoo.com. Таким образом, вы можете изменить имя хоста, а также порт в соответствии с вашими поставщиками услуг электронной почты.
{% endhint %}

## Bыводы

В этой главе мы познакомились со стандартным вводом и выводом данных. Мы узнали, как stdin и stdout выполняют функции ввода с клавиатуры и пользовательского терминала соответственно. Мы также узнали о функциях input() и print(). В дополнение к этому мы узнали об отправке электронного письма из Gmail . Мы отправили электронное письмо с простым текстом, а также вложение. Также мы узнали о форматировании строк с использованием метода format() и оператора %.

В следующей главе вы узнаете о том, как работать с различными файлами, такими как PDF, Excel и csv.

## Вопросы

1. В чем разница между stdin и input?

В контексте языков программирования, таких как Python,  **input** — это функция или метод, который позволяет пользователю вводить данные с клавиатуры. Функция input() останавливает выполнение программы и ждёт, пока пользователь введёт данные.

**Stdin** (standard input) — это стандартный поток ввода в операционных системах и языках программирования. Он представляет собой поток данных, через который программа может получать входные данные. Stdin обычно связан с клавиатурой, но может быть перенаправлен из других источников, таких как файлы или другие программы.

Таким образом, основное различие между stdin и input заключается в том, что stdin — это поток ввода, а input — это функция для получения данных от пользователя.

2. Что такое SMTP?

**SMTP** (Simple Mail Transfer Protocol) — это простой протокол передачи почты, который используется для отправки электронных писем между почтовыми серверами. Он определяет правила и формат сообщений, которые передаются между отправителем и получателем.

С помощью SMTP можно отправлять письма с любого устройства, подключённого к интернету, на любой почтовый сервер. Для этого нужно знать адрес сервера, порт, имя пользователя и пароль. SMTP работает на основе клиент-серверной модели, где клиент отправляет сообщение на сервер, а сервер пересылает его на другой сервер или доставляет получателю.

3. Какой будет вывод у следующего кода?

```python
>>> name = "Eric"
>>> profession = "comedian"
>>> affiliation = "Monty Python"
>>> age = 25
>>> message = (
...     f"Hi {name}. "
...     f"You are a {profession}. "
...     f"You were in {affiliation}."
... )
>>> message
```

Output:

```
Hi Eric. You are a comedian. You were in Monty Python.
```

4. Какой будет вывод у следующего кода?

```bash
str1 = 'Hello'
str2 ='World!'
print('str1 + str2 = ', str1 + str2)
print('str1 * 3 =', str1 * 3)
```

Output:

```
str1 + str2 = Helloworld!
str1 * 3 = HelloHelloHello
```

## Читать далее

Документация по строкам: [https://docs.python.org/3.1/library/string.html ](https://docs.python.org/3.1/library/string.html)

Документация smptplib: [https://docs.python.org/3/library/smtplib.html ](https://docs.python.org/3/library/smtplib.html)
