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

## Добавление кода ведения журнала и предупреждений в скрипты

В этом разделе мы познакомимся с модулями ведения журнала и предупреждений в Python. Модуль ведения журнала будет отслеживать события, происходящие в программе. Модуль предупреждений уведомляет программистов об изменениях, внесенных в язык, а также в библиотеки.

Теперь мы рассмотрим простой пример ведения журнала. Мы напишем скрипт с именем logging\_example.py и введем в него следующий код:

```python
import logging
LOG_FILENAME = 'log.txt'
logging.basicConfig(filename=LOG_FILENAME, level=logging.DEBUG,)
logging.debug('This message should go to the log file')
with open(LOG_FILENAME, 'rt') as f:
prg = f.read()
print('FILE:')
print(prg)
```

Запустите программу следующим образом:

```sh
$ python3 logging_example.py

Output:
FILE:
DEBUG:root:This message should go to the log file
```

Проверьте hello.py и вы увидите сообщение об отладке, напечатанное в этом скрипте:

```bash
$ cat log.txt
Output:
DEBUG:root:This message should go to the log file
```

Теперь мы напишем скрипт под названием logging\_warnings\_codes.py и введем в него следующий код:

```sh
import logging
import warnings
logging.basicConfig(level=logging.INFO,)
warnings.warn('This warning is not sent to the logs')
logging.captureWarnings(True)
warnings.warn('This warning is sent to the logs')
```

Запустите скрипт следующим образом:

```sh
$ python3 logging_warnings_codes.py

Output:
logging_warnings_codes.py:6: UserWarning: This warning is not sent to the logs
warnings.warn('This warning is not sent to the logs')
WARNING:py.warnings:logging_warnings_codes.py:10: UserWarning: This warning is sent to the logs
warnings.warn('This warning is sent to the logs')
```

## Генерирование предупреждений

Для генерации предупреждений используется функция warn(). Сейчас мы рассмотрим простой пример генерации предупреждений. Напишите скрипт с именем generate\_warnings.py и напишите в нем следующий код:

```python
import warnings
warnings.simplefilter('error', UserWarning)
print('Before')
warnings.warn('Write your warning message here')
print('After')
```

Запустите скрипт следующим образом:

```bash
$ python3 generate_warnings.py

Output:
Before:
Traceback (most recent call last):
File "generate_warnings.py", line 6, in <module>
warnings.warn('Write your warning message here')
UserWarning: Write your warning message here
```

В предыдущем скрипте мы передавали предупреждающее сообщение через функцию warn(). Мы использовали простой фильтр, чтобы ваше предупреждение было расценено как ошибка, и программист соответствующим образом устранил эту ошибку.

## Установка ограничений на использование процессора и памяти

В этом разделе мы узнаем о том, как мы можем ограничить использование процессора и памяти. Сначала мы напишем скрипт для ограничения использования процессора. Создайте скрипт с именем put\_cpu\_limit.py и напишите в нем следующий код:

```python
import resource
import sys
import signal
import time
def time_expired(n, stack):
print('EXPIRED :', time.ctime())
raise SystemExit('(time ran out)')
signal.signal(signal.SIGXCPU, time_expired)
# Adjust the CPU time limit
soft, hard = resource.getrlimit(resource.RLIMIT_CPU)
print('Soft limit starts as :', soft)
resource.setrlimit(resource.RLIMIT_CPU, (10, hard))
soft, hard = resource.getrlimit(resource.RLIMIT_CPU)
print('Soft limit changed to :', soft)
print()
# Consume some CPU time in a pointless exercise
print('Starting:', time.ctime())
for i in range(200000):
for i in range(200000):
v = i * i
# We should never make it this far
print('Exiting :', time.ctime())
```

Запустите предыдущий скрипт следующим образом:

```sh
$ python3 put_cpu_limit.py

Output:
Soft limit starts as : -1
Soft limit changed to : 10
Starting: Thu Sep 6 16:13:20 2018
EXPIRED : Thu Sep 6 16:13:31 2018
(time ran out)
```

В предыдущем скрипте мы использовали функцию setrlimit() для ограничения загрузки процессора. Итак, в нашем скрипте мы установили ограничение на 10 секунд.

## Запуск веб-браузера

В этом разделе мы познакомимся с модулем webbrowser на Python. В этом модуле есть функции для открытия URL-адресов в браузерных приложениях. Мы рассмотрим простой пример. Создайте скрипт с именем open\_web.py и напишите в нем следующий код:

```python
import webbrowser
webbrowser.open('https://timesofindia.indiatimes.com/world')
```

```sh
$ python3 open_web.py

Output:
Url mentioned in open() will be opened in your browser.
webbrowser – Command line interface
```

Вы также можете использовать модуль webbrowser на Python через командную строку и можете использовать его полностью. Чтобы использовать веб-браузер через командную строку, выполните следующую команду:

```sh
$ python3 -m webbrowser -n https://www.google.com/
```

https://www.google.com / будет открыт в окне браузера. Вы можете использовать следующие два варианта:

• -n: Открыть новое окно&#x20;

• -t: Открыть новую вкладку

## Использование модуля os для работы с каталогами и файлами

В этом разделе мы познакомимся с модулем os Python. Модуль os Python помогает в решении задач операционной системы. Нам необходимо импортировать модуль os, если мы хотим выполнять задачи операционной системы. Мы рассмотрим несколько примеров, связанных с обработкой файлов и каталогов.

## Создание и удаление каталога

В этом разделе мы собираемся создать скрипт, в котором мы увидим, какие функции мы можем использовать для работы с каталогами в вашей файловой системе, включая создание, перечисление и удаление содержимого. Создайте скрипт с именем os\_dir\_example.py и напишите в нем следующий код:

```python
import os
directory_name = 'abcd'
print('Creating', directory_name)
os.makedirs(directory_name)
file_name = os.path.join(directory_name, 'sample_example.txt')
print('Creating', file_name)
with open(file_name, 'wt') as f:
f.write('sample example file')
print('Cleaning up')
os.unlink(file_name)
os.rmdir(directory_name) # Will delete the directory
```

Запустите скрипт:

```sh
$ python3 os_dir_example.py

Output:
Creating abcd
Creating abcd/sample_example.txt
Cleaning up
```



Когда вы создаете каталог с помощью mkdir(), все родительские каталоги должны быть уже созданы. Но когда вы создаете каталог с помощью makedirs(), он создаст любой каталог, который указан в пути, который не существует. функция unlink() удалит путь к файлу, а функция rmdir() удалит путь к каталогу.

## Проверка содержимого файловой системы

В этом разделе мы перечислим все содержимое каталога с помощью функции listdir(). Создайте скрипт с именем list\_dir.py и напишите в нем следующий код:

```python
import os
import sys
print(sorted(os.listdir(sys.argv[1])))
```

Запустите скрипт:

```sh
$ python3 list_dir.py /home/student/

['.ICEauthority', '.bash_history', '.bash_logout', '.bashrc', '.cache', '.config', '.gnupg', '.local', '.mozilla', '.pam_environment', '.profile', '.python_history', '.ssh', '.sudo_as_admin_successful', '.viminfo', '1.sh', '1.sh.x', '1.sh.x.c', 'Desktop', 'Documents', 'Downloads', 'Music', 'Pictures', 'Public', 'Templates', 'Videos', 'examples.desktop', 'execute_external_commands.py', 'log.txt', 'numbers.txt', 'python_learning', 'work']
```

Итак, используя listdir(), вы можете просмотреть список всего содержимого папки.

## Создание резервных копий (с помощью rsync)

Это самая важная работа, которую приходится выполнять системным администраторам. В этом разделе мы узнаем о создании резервных копий с помощью rsync. Команда rsync используется для локального и удаленного копирования файлов и каталогов, а также для выполнения резервного копирования данных с помощью rsync. Для этого мы собираемся написать скрипт под названием take\_backup.py и вписать в него следующий код:

```python
import os
import shutil
import time
from sh import rsync
def check_dir(os_dir):
if not os.path.exists(os_dir):
print (os_dir, "does not exist.")
exit(1)
def ask_for_confirm():
ans = input("Do you want to Continue? yes/no\n")
global con_exit
if ans == 'yes':
con_exit = 0
return con_exit
elif ans == "no":
con_exit = 1
return con_exit
else:1
print ("Answer with yes or no.")
ask_for_confirm()
def delete_files(ending):
for r, d, f in os.walk(backup_dir):
for files in f:
if files.endswith("." + ending):
os.remove(os.path.join(r, files))

backup_dir = input("Enter directory to backup\n") # Enter directory name
check_dir(backup_dir)
print (backup_dir, "saved.")
time.sleep(3)
backup_to_dir= input("Where to backup?\n")
check_dir(backup_to_dir)
print ("Doing the backup now!")
ask_for_confirm()
if con_exit == 1:
print ("Aborting the backup process!")
exit(1)
rsync("-auhv", "--delete", "--exclude=lost+found", "--exclude=/sys", "--exclude=/tmp", "--exclude=/proc",
"--exclude=/mnt", "--exclude=/dev", "--exclude=/backup", backup_dir, backup_to_dir)
```

Запустите скрипт:

```
student@ubuntu:~/work$ python3 take_backup.py

Output :
Enter directory to backup
/home/student/work
/home/student/work saved.
Where to backup?
/home/student/Desktop
Doing the backup now!
Do you want to Continue? yes/no
yes
```

Теперь проверьте папку Desktop , и вы увидите свою рабочую папку в этом каталоге. С помощью команды rsync можно использовать несколько параметров, а именно:

• -a: Архивировать

• -u: Обновить&#x20;

• -h: Удобочитаемый формат

• -v: Подробнее

• --delete: Удалить посторонние файлы с принимающей стороны

• --exclude: Исключить

## **Итоги**

В этой главе мы узнали о том, как можно автоматизировать обычные административные задачи. Мы узнали о том, как принимать вводимые данные различными способами, запрашивать пароли во время выполнения, выполнять внешние команды, читать файлы конфигурации, добавлять предупреждения в свой скрипт, запускать веббраузер как через скрипт, так и из командной строки, использовать модуль os для обработки файлов и каталогов и создавать резервные копии. В следующей главе вы узнаете больше о модуле os и обработке данных. Кроме того, вы узнаете о модуле tarfile и о том, как его можно использовать.

## Вопросы&#x20;

1.Как использовать модуль readline?&#x20;

Для использования модуля `readline` в Python, вам нужно сначала импортировать его в свой код. Это можно сделать с помощью команды:

```
import readline
```

После импорта вы можете использовать функции модуля для работы с командной строкой. Вот несколько примеров того, как это можно сделать:

**Использование функции `input()` с `readline`:**\
Модуль `readline` предоставляет возможность использовать функцию `input()`, которая позволяет пользователю вводить данные с клавиатуры. Вы можете использовать её для получения ввода от пользователя.

Пример:

```
import readline

name = input("Введите ваше имя: ")
print("Здравствуйте,", name + "!")
```

В этом примере функция `input` использует возможности модуля `readline`, чтобы предоставить пользователю возможность ввести своё имя.

**Настройка истории команд:**\
Вы можете настроить историю команд, чтобы иметь возможность просматривать и редактировать предыдущие команды. Для этого используйте функцию `set_history()` из модуля `readline`.

Пример:

```
import readline

# Устанавливаем размер истории команд
readline.set_history_size(100)

# Получаем ввод от пользователя
user_input = input("Введите команду: ")

# Отображаем историю команд
print("История команд:")
for command in readline.get_history_file().splitlines():
    print(command)
```

Этот пример устанавливает размер истории команд равным 100 и позволяет просматривать историю команд после каждого ввода.

**Редактирование команд:**\
Модуль `readline` также предоставляет возможность редактирования команд перед их выполнением. Для этого вы можете использовать функцию `read_history_and_edit()`.

Пример:

```python
import readline

def edit_command(command):
    # Получаем редактируемую команду
    edited_command = readline.read_history_and_edit(command)
    return edited_command

# Используем функцию для редактирования команды
user_input = input("Введите или отредактируйте команду: ")
edited_command = edit_command(user_input)
print("Отредактированная команда:", edited_command)
```

Здесь функция `edit_command` позволяет редактировать команду перед её выполнением.

Это лишь некоторые примеры использования модуля `readline` в Python. Он предоставляет множество функций для работы с командной строкой и может быть очень полезным при разработке интерактивных приложений.

2.Какие команды Linux используются для чтения, создания нового файла, удаления файла, отображения файла в текущем каталоге?&#x20;

Для чтения, создания нового файла, удаления файла и отображения файлов в текущем каталоге в Linux используются следующие команды:

**Чтение файла.** Чтобы прочитать содержимое файла, используется команда `cat` (сокращение от «concatenate» — объединять). Например, чтобы вывести содержимое файла `file.txt`, нужно ввести команду `cat file.txt`.

**Создание нового файла.** Для создания нового файла используется команда `touch`. Например, чтобы создать файл `new_file.txt`, введите команду `touch new_file.txt`. Если файл уже существует, команда `touch` обновит его время модификации.

**Удаление файла.** Для удаления файла используется команда `rm` (сокращение от «remove» — удалять). Например, чтобы удалить файл `old_file.txt`, введите команду `rm old_file.txt`. Будьте осторожны при использовании этой команды, так как она удаляет файлы без возможности восстановления.

**Отображение файлов в текущем каталоге.** Для отображения списка файлов и каталогов в текущем рабочем каталоге используется команда `ls` (сокращение от «list» — список). Например, чтобы увидеть список всех файлов и каталогов в вашем текущем рабочем каталоге, введите команду `ls`. Вы также можете использовать различные опции команды `ls`, такие как `-l` для длинного формата вывода или `-a` для отображения всех файлов, включая скрытые.

3.Какие пакеты доступны для запуска команд Linux/Windows на python?

Для запуска команд Linux и Windows на Python существует несколько пакетов. Вот некоторые из них:

**os** — это встроенный модуль в Python, который предоставляет функции для взаимодействия с операционной системой. Он позволяет выполнять команды, связанные с файловой системой, процессами и другими системными функциями.

**subprocess** — этот модуль позволяет запускать новые процессы и получать их вывод. Он может использоваться для выполнения команд Linux и Windows.

**shutil** — модуль shutil предоставляет функции для работы с файлами и папками. Хотя он не предназначен специально для запуска команд, его можно использовать для копирования, перемещения и удаления файлов и папок.

**paramiko** — это библиотека Python, которая позволяет выполнять команды на удалённых серверах через SSH. Она может быть полезна для запуска команд на Linux-серверах.

**plumbum** — эта библиотека предоставляет простой способ выполнения команд в командной строке. Она поддерживает как локальные, так и удалённые команды.

**Pexpect** — ещё одна библиотека, которая позволяет взаимодействовать с программами через командную строку. Она может использоваться для автоматизации задач и выполнения команд.

**WMI (Windows Management Instrumentation)** — это интерфейс для управления и мониторинга компьютеров под управлением Windows. В Python есть несколько библиотек, которые позволяют работать с WMI, например, wmi или pywin32. Они могут использоваться для выполнения команд на Windows-компьютерах.

4.Как прочитать или задать новые значения в ini-файле&#x20;

Для работы с ini-файлами в Python существует несколько библиотек. Одна из них — ConfigParser, которая входит в стандартную библиотеку Python.

Чтобы прочитать или задать новые значения в ini-файле с помощью Python на Linux, выполните следующие шаги:

**Установите библиотеку ConfigParser**, если она ещё не установлена. Это можно сделать с помощью команды pip install configparser.

**Откройте файл**. Используйте функцию open() для открытия файла. Если файл находится в другом каталоге, укажите полный путь к файлу.

**Создайте объект ConfigParser** и передайте ему имя файла.

**Прочитайте файл**. Вызовите метод read\_file() объекта ConfigParser и передайте ему объект файла, который вы открыли на предыдущем шаге.

**Получите доступ к разделам и параметрам**. Используйте методы get() или items() объекта ConfigParser для получения доступа к разделам и параметрам.

**Измените значения параметров**. Если вы хотите изменить значения параметров, используйте метод set() объекта ConfigParser.

**Сохраните изменения**. После внесения изменений вызовите метод write() объекта ConfigParser, чтобы сохранить изменения в файле.

Пример кода для чтения и изменения значений в ini-файле:

```sh
import configparser

# Открываем файл
with open('config.ini', 'r+') as configfile:
    # Создаём объект ConfigParser
    config = configparser.ConfigParser()
    # Читаем файл
    config.read_file(configfile)

    # Получаем доступ к разделу и параметру
    section = config['Section1']
    parameter = section['Parameter1']

    # Изменяем значение параметра
    new_value = 'New Value'
    section['Parameter1'] = new_value

    # Сохраняем изменения
    with open('config.ini', 'w') as configfile:
        config.write(configfile)
```

Этот код открывает файл config.ini, читает его содержимое, изменяет значение параметра Parameter1 в разделе Section1 и сохраняет изменения в файле.

5.Перечислите библиотеки, доступные для определения загрузки процессора?&#x20;

Для определения загрузки процессора в Python существует несколько библиотек. Вот некоторые из них:

**psutil** — кроссплатформенная библиотека для получения информации о системе и процессах. Она предоставляет информацию о загрузке процессора, использовании памяти, сетевом трафике и других системных параметрах.

**os** — стандартная библиотека Python, которая предоставляет функции для взаимодействия с операционной системой. С её помощью можно получить информацию о загрузке процессора через системные вызовы.

**platform** — ещё одна стандартная библиотека, которая позволяет получать информацию о платформе и версии Python. Хотя она не предоставляет прямой доступ к загрузке процессора, её можно использовать для получения общей информации о системе.

**statsmodels** — библиотека для статистического моделирования и анализа данных. Хотя она в первую очередь предназначена для статистических задач, некоторые функции могут быть полезны для анализа загрузки процессора.

**Guppy** — инструмент для профилирования и анализа использования памяти и процессора. Он может быть полезен для мониторинга загрузки процессора и выявления узких мест в производительности.

**multiprocessing** — библиотека для параллельного выполнения задач. Хотя она больше ориентирована на многопоточность, она также может предоставить информацию о загрузке процессора при выполнении задач.

Выбор библиотеки зависит от ваших конкретных потребностей и требований проекта.

6.Перечислите различные методы получения входных данных от пользователя?

В Python существует несколько методов получения входных данных от пользователя:

**Функция input()**. Это самый простой и наиболее часто используемый метод. Он приостанавливает выполнение программы и ждёт, пока пользователь введёт данные с клавиатуры.

**Чтение из файла**. Можно попросить пользователя ввести данные заранее и сохранить их в файле. Затем программа может прочитать этот файл и использовать его содержимое в качестве входных данных.

**Аргументы командной строки**. При запуске программы можно передать ей аргументы через командную строку. Эти аргументы могут быть использованы в программе как входные данные.

**Использование библиотеки getpass**. Если нужно получить пароль от пользователя, можно использовать библиотеку getpass. Она позволяет скрыть вводимые символы.

**Клавиатурные прерывания (keyboard interrupts)**. Хотя это не совсем стандартный метод получения ввода, он может быть использован для обработки пользовательского ввода при возникновении определённого события (например, нажатия определённой клавиши).

**Консольные команды с библиотекой cmd**. Библиотека cmd позволяет создавать интерактивные интерфейсы командной строки, где пользователь может вводить команды и получать результаты

**Форматированный ввод с использованием библиотеки argparse**. Эта библиотека позволяет анализировать и обрабатывать аргументы командной строки в структурированном формате.

**Ввод через графический интерфейс (GUI)**. Для программ с графическим интерфейсом можно использовать виджеты, такие как поля ввода или текстовые поля, для получения данных от пользователя.
