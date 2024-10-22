# Глава 4. Автоматизация  деятельности системного администратора

Системные администраторы выполняют различные действия. Эти действия могут включать обработку файлов, ведение журнала, администрирование процессора и памяти, обработку паролей и, что наиболее важно, создание резервных копий. Эти действия требуют автоматизации. В этой главе мы рассмотрим автоматизацию этих действий с помощью Python.

В этой главе мы рассмотрим следующие темы:

* Прием входных данных с помощью перенаправления, канала и входных файлов
* Обработка паролей во время выполнения в скриптах
* Выполнение внешних команд и получение их выходных данных
* Запрос пароля во время выполнения и проверки
* Чтение конфигурационных файлов
* Добавление кода ведения журнала и предупреждения в скрипты
* Установление ограничений на использование процессора и памяти
* Запуск веб-браузера
* Использование os модуля для работы с каталогами и файлами
* Создание резервных копий (с помощью rsync)

## Прием входных данных с помощью перенаправления, канала и входных файлов

В этом разделе мы узнаем о том, как пользователи могут принимать ввод с помощью перенаправления, конвейера и внешних входных файлов.

Для приема ввода с помощью перенаправления мы используем стандартный идентификатор. Конвейер - это еще одна форма перенаправления. Эта концепция означает предоставление выходных данных одной программы в качестве входных данных для другой программы. Мы можем принимать входные данные как из внешних файлов, так и с помощью Python.

## Ввод с помощью перенаправления

stdin и stdout - это объекты, созданные модулем os. Мы собираемся написать скрипт, в котором будем использовать stdin и stdout.

```python
import sys

class Redirection(object):
    def __init__(self, in_obj, out_obj):
        self.input = in_obj
        self.output = out_obj
    def read_line(self):
        res = self.input.readline()
        self.output.write(res)
        return res

 if __name__ == '__main__':
    if not sys.stdin.isatty():
        sys.stdin = Redirection(in_obj=sys.stdin, out_obj=sys.stdout)
        
    a = input('Enter a string: ')
    b = input('Enter another string: ')
    print ('Entered strings are: ', repr(a), 'and', repr(b)) 
```

Запустите предыдущую программу следующим образом:

```sh
$ python3 redirection.py
```

Мы получим следующий результат:

```sh
Output:
Enter a string: hello
Enter another string: python
Entered strings are:  'hello' and 'python'
```

Всякий раз, когда программа запускается в интерактивном сеансе, stdin используется для ввода с клавиатуры, а stdout - вывода на  пользовательский терминал. Функция input() используется для получения входных данных от пользователя, а print() - это способ ввода на терминале (стандартный вывод).

## Ввод через pipe

Pipe - это еще одна форма перенаправления. Этот метод используется для передачи информации из одной программы в другую. Символ | обозначает канал. Используя метод канала, мы можем использовать более двух команд таким образом, что выходные данные одной команды используются как входные данные для следующей команды.

Теперь мы посмотрим, как мы можем принимать входные данные с помощью pipe. Для этого сначала мы напишем простой скрипт, который возвращает результат целочисленного деления. Создайте скрипт с именем accept\_by\_pipe.py и напишите в нем следующий код:

```python
import sys

for n in sys.stdin:
    print ( int(n.strip())//2 ) 
```

Запустите скрипт, и вы получите следующий результат:

```bash
$ echo 15 | python3 accept_by_pipe.py
Output:
7 
```

В предыдущем сценарии stdin вводился с клавиатуры. Мы выполняем целочисленное деление числа, которое вводим во время выполнения. Целочисленное деление возвращает только целую часть частного. Когда мы запускаем программу, мы передаем 15, за которым следует | символ, а затем название нашего скрипта. Итак, мы вводим 15 в качестве входных данных для нашего скрипта. Таким образом, целочисленное деление, и мы получаем результат в виде 7.

Мы можем передать несколько входных данных в наш скрипт. Итак, в следующем выполнении мы передали несколько входных значений как 15, 45 и 20. Для обработки нескольких входных значений мы написали цикл for в нашем скрипте. Таким образом, сначала он примет значение 15, затем 45, а затем 20. Выходные данные будут напечатаны в новой строке для каждого входного значения, поскольку мы записали \n между входными значениями. Чтобы включить такую интерпретацию обратной косой черты, мы установили флаг -e:

```sh
$ echo -e '15\n45\n20' | python3 accept_by_pipe.py
Output:
7
22
10
```

После этого мы получили целочисленное деление 15, 45 и 20 как 7, 22 и 10, соответственно, на новых строках.

## Ввод с помощью входного файла

В этом разделе мы узнаем о том, как мы можем использовать входные данные из входного файла. В Python проще использовать входные данные из входного файла. Мы рассмотрим пример для этого. Но сначала мы создадим простой текстовый файл с именем sample.txt и запишем в него следующий код:

Sample.txt:

```
Hello World
Hello Python
```

Теперь создайте скрипт с именем accept\_by\_input\_file.py и напишите в нем следующий код:

```python
i = open('sample.txt','r')
o = open('sample_output.txt','w')

a = i.read()
o.write(a)
```

Запустите программу, и вы получите следующий результат:

```bash
$ python3 accept_by_input_file.py
$ cat sample_output.txt
Hello World
Hello Python
```

## Обработка паролей во время выполнения в скриптах

В этом разделе мы рассмотрим простой пример работы с паролями в скрипте. Мы создадим скрипт с именем handling\_password.py и запишем в него следующее содержимое:

```python
import sys
import paramiko
import time

ip_address = "192.168.2.106"
username = "student"
password = "training"
ssh_client = paramiko.SSHClient()
ssh_client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
ssh_client.load_system_host_keys()
ssh_client.connect(hostname=ip_address,\
                                    username=username, password=password)
print ("Successful connection", ip_address)
ssh_client.invoke_shell()
remote_connection = ssh_client.exec_command('cd Desktop; mkdir work\n')
remote_connection = ssh_client.exec_command('mkdir test_folder\n')
#print( remote_connection.read() )
ssh_client.close
```

Запустите предыдущий скрипт, и вы получите следующий результат:

```sh
$ python3 handling_password.py

Output:
Successful connection 192.168.2.106
```

В предыдущем скрипте мы использовали модуль paramiko. Модуль paramiko - это реализация ssh на языке Python, которая обеспечивает функциональность клиент-сервер.

```sh
pip3 install paramiko
```

В предыдущем сценарии мы удаленно подключались к хосту 192.168.2.106. В нашем сценарии мы указали имя пользователя и пароль хоста.

После запуска этого скрипта на рабочем столе 192.168.2.106 вы найдете папку work, и папку test\_folder  в каталоге home.

## Выполнение внешних команд и получение их выходных данных

В этом разделе мы познакомимся с модулем subprocess в Python. Используя subprocess, легко создавать новые процессы и получать их возвращаемый код, выполнять внешние команды и запускать новые приложения.

```python
import subprocess
subprocess.call(["touch", "sample.txt"])
subprocess.call(["ls"])
print("Sample file created")
subprocess.call(["rm", "sample.txt"])
subprocess.call(["ls"])
print("Sample file deleted")
```

Запустите программу, и вы получите следующий результат:

```bash
$ python3 execute_external_commands.py
Output:
1.py     accept_by_pipe.py      sample_output.txt       sample.txt
accept_by_input_file.py         execute_external_commands.py         output.txt        sample.py
Sample.txt file created
1.py     accept_by_input_file.py         accept_by_pipe.py execute_external_commands.py  output.txt            sample_output.txt       sample.py
Sample.txt file deleted

```

## Захват выходных данных с помощью модуля subprocess

В этом разделе мы узнаем о том, как мы можем перехватывать выходные данные. Мы передадим PIPE в качестве аргумента stdout для перехвата выходных данных. Напишите сценарий с именем capture\_output.py и напишите в нем следующий код:

```python
import subprocess
res = subprocess.run(['ls', '-1'], stdout=subprocess.PIPE,)
print('returncode:', res.returncode)
print(' {} bytes in stdout:\n{}'.format(len(res.stdout), res.stdout.decode('utf-8')))
```

Выполните сценарий следующим образом:

```bash
student@ubuntu:~$ python3 capture_output.py

```

При выполнении мы получим следующий результат:

```bash
Output:
returncode: 0
191 bytes in stdout:
1.py
accept_by_input_file.py
accept_by_pipe.py
execute_external_commands.py
getpass_example.py
ouput.txt
output.txt
password_prompt_again.py
sample_output.txt
sample.py
capture_output.py
```

В предыдущем скрипте мы импортировали модуль subprocess из Python, который помогает получать выходные данные. Модуль subprocess используется для создания новых процессов. Он также помогает подключать каналы ввода/вывода и получать код возврата. subprocess.run() выполнит команду, переданную в качестве аргумента. Возвращаемый код будет означать статус завершения вашего дочернего процесса. Если в выходных данных вы получите код возврата в виде 0, это означает, что он был успешно выполнен.

## **Запрос паролей во время выполнения и проверка**

В этом разделе мы познакомимся с модулем getpass для обработки паролей во время выполнения. Модуль getpass() в Python предлагает пользователю ввести пароль без повторения. Модуль getpass используется для обработки запроса пароля всякий раз, когда программы взаимодействуют с пользователем через терминал.

Мы рассмотрим несколько примеров использования модуля getpass:

1. Создайте скрипт с именем no\_prompt.py и напишите в нем следующий код:



<pre class="language-python"><code class="lang-python"><strong>import getpass
</strong><strong>try:
</strong><strong>            p = getpass.getpass()
</strong><strong>except Exception as error:
</strong><strong>            print('ERROR', error)
</strong><strong>else:
</strong><strong>            print('Password entered:', p)
</strong></code></pre>

В этом скрипте запрос для пользователя не предусмотрен. По умолчанию установлено требование ввести пароль.

Запустите скрипт следующим образом:

```bash
$ python3 no_prompt.py
Output :
Password:
Password entered: abcd
```

2. Мы предусмотрим запрос на ввод пароля. Для этого создайте скрипт под названием with\_prompt.py и напишите в нём следующий код:

```python
import getpass
try:
            p = getpass.getpass("Enter your password: ")
except Exception as error:
            print('ERROR', error)
else:
            print('Password entered:', p)
```

Теперь мы написали скрипт, который выдает запрос на ввод пароля. Запустите программу следующим образом:

<pre class="language-sh"><code class="lang-sh"><strong>$ python3 with_prompt.py
</strong><strong>Output:
</strong><strong>Enter your password:
</strong><strong>Password entered: abcd
</strong><strong>
</strong></code></pre>

Мы предоставили пользователю запрос на ввод пароля.

Теперь мы напишем скрипт, в котором, если мы введем неправильный пароль, он просто напечатает простое сообщение, но больше не будет предлагать ввести правильный пароль.

3. Напишите скрипт с именем getpass\_example.py и напишите в нем следующий код:

```python
import getpass
passwd = getpass.getpass(prompt='Enter your password: ')
if passwd.lower() == '#pythonworld':
            print('Welcome!!')
else:
            print('The password entered is incorrect!!')
```

Запустите программу следующим образом (здесь мы вводим правильный пароль, то есть #pythonworld).:

<pre class="language-sh"><code class="lang-sh"><strong>$ python3 getpass_example.py
</strong><strong>Output:
</strong><strong>Enter your password:
</strong><strong>Welcome!!
</strong></code></pre>

Теперь мы введем неправильный пароль и проверим, какое сообщение мы получим:

<pre class="language-sh"><code class="lang-sh"><strong>$ python3 getpass_example.py
</strong><strong>Output:
</strong><strong>Enter your password:
</strong><strong>The password entered is incorrect!!
</strong></code></pre>

Здесь мы написали скрипт, который никогда больше не попросит ввести пароль, если мы введем неправильный пароль.

Теперь мы напишем скрипт, который снова попросит ввести правильный пароль, когда мы введем неправильный пароль. Чтобы получить логин пользователя, используется функция getuser(). Функция getuser() вернет пользователя, вошедшего в систему. Создайте скрипт с именем password\_prompt\_again.py и напишите в нем следующий код:

<pre class="language-python"><code class="lang-python"><strong>import getpass
</strong><strong>user_name = getpass.getuser()
</strong><strong>print ("User Name : %s" % user_name)
</strong><strong>while True:
</strong><strong>            passwd = getpass.getpass("Enter your Password : ")
</strong><strong>            if passwd == '#pythonworld':
</strong><strong>                        print ("Welcome!!!")
</strong><strong>                        break
</strong><strong>            else:
</strong><strong>                        print ("The password you entered is incorrect.")
</strong></code></pre>

Запустите программу, и вы получите следующий результат:

<pre class="language-sh"><code class="lang-sh"><strong>student@ubuntu:~$ python3 password_prompt_again.py
</strong><strong>User Name : student
</strong><strong>Enter your Password :
</strong><strong>The password you entered is incorrect.
</strong><strong>Enter your Password :
</strong><strong>Welcome!!!
</strong></code></pre>

## Чтение конфигурационных файлов

В этом разделе мы познакомимся с модулем configparser в Python. Используя модуль configparser, вы можете управлять редактируемыми пользователем файлами конфигурации приложения.&#x20;

Обычное использование этих конфигурационных файлов заключается в том, что пользователи или системные администраторы могут редактировать файлы с помощью простого текстового редактора, чтобы установить значения приложения по умолчанию, а затем приложение будет считывать, анализировать их и действовать в соответствии с записанным в них содержимым.&#x20;

Для чтения конфигурационного файла в configparser есть метод read(). Теперь мы напишем простой скрипт с именем read\_config\_file.py. Перед этим создадим ini-файл с именем read\_simple.ini и запишем в него следующее содержимое: read\_simple.ini

```
[bug_tracker]
url =https://timesofindia.indiatimes.com/
```

Создайте файл read\_config\_file.py и введите в него следующее содержимое:

```python
from configparser import ConfigParser
p = ConfigParser()
p.read('read_simple.ini')
print(p.get('bug_tracker', 'url'))
```

Запустите read\_config\_file.py, и вы получите следующий результат:

```bash
$ python3 read_config_file.py

Output:
https://timesofindia.indiatimes.com/
```

Метод read() принимает более одного имени файла. Всякий раз, когда проверяется каждое имя файла, и если этот файл существует, он будет открыт и прочитан. Теперь мы напишем скрипт для чтения нескольких имен файлов. Создайте скрипт с именем read\_many\_config\_file.py и напишите в нем следующий код:

```python
from configparser import ConfigParser
import glob

p = ConfigParser()
files = ['hello.ini', 'bye.ini', 'read_simple.ini', 'welcome.ini']
files_found = p.read(files)
files_missing = set(files) - set(files_found)
print('Files found: ', sorted(files_found))
print('Files missing: ', sorted(files_missing))
```

Запустите предыдущий скрипт, и вы получите следующий результат:

```bash
$ python3 read_many_config_file.py

Output
Files found: ['read_simple.ini']
Files missing: ['bye.ini', 'hello.ini', 'welcome.ini']
```

В предыдущем примере мы использовали модуль configparser на Python, который помогает управлять файлами конфигурации. Сначала мы создали список файлов с именами. Функция read() будет считывать файлы конфигурации. В примере мы создали переменную files\_found, в которой будут храниться имена файлов конфигурации, присутствующих в вашем каталоге. Затем мы создали другую переменную files\_missing, которая будет возвращать имена файлов, отсутствующих в вашем каталоге. И, наконец, мы печатаем имена файлов, которые присутствуют и отсутствующие.

