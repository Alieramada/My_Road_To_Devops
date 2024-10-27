# Глава 5. Обработка файлов, каталогов и данных

Системный администратор выполняет такие задачи, как обработка различных файлов, каталогов и данных. В этой главе мы познакомимся с модулем os. Модуль os предоставляет функциональные возможности для взаимодействия с операционной системой. Программисты на Python могут легко использовать модуль os для выполнения операций с файлами и каталогами. Модуль os предоставляет программистам инструменты, которые работают с файлами, путями к ним, каталогами и данными. В этой главе вы узнаете о следующем:&#x20;

• Использование модуля os для работы с каталогами&#x20;

• Копирование, перемещение, переименование и удаление данных&#x20;

• Работа с путями, каталогами и файлами&#x20;

• Сравнение данных&#x20;

• Объединение данных&#x20;

• Сопоставление файлов и каталогов с шаблонами&#x20;

• Метаданные: данные о данных&#x20;

• Сжатие и восстановление

• Использование модуля tarfile для создания TAR-архивов&#x20;

• Использование модуля tarfile для проверки содержимого файлов TAR

## Использование модуля os для работы с каталогами&#x20;

Каталог или папка - это набор файлов и подкаталогов. Модуль os предоставляет различные функции, которые позволяют нам взаимодействовать с операционной системой. В этом разделе мы узнаем о некоторых функциях, которые можно использовать при работе с каталогами.

## Получить рабочий каталог&#x20;

Чтобы начать работу с каталогами, сначала мы получим название нашего текущего рабочего каталога. В модуле os есть функция getcwd(), с помощью которой мы можем получить текущий рабочий каталог. Запустите консоль python3 и введите следующие команды, чтобы получить имя каталога:

```python
$ python3
Python 3.6.5 (default, Apr  1 2018, 05:46:30) 
[GCC 7.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import os
>>> os.getcwd()
'/home/student'
>> 
```

## Изменение каталога

Используя модуль os, мы можем изменить текущий рабочий каталог. Для этого в модуле os есть функция chdir(), например:

```bash
>>> os.chdir('/home/student/work')
>>> print(os.getcwd())
/home/student/work
>>> 
```

## Список файлов и каталогов

Вывод содержимого каталога на Python прост. Мы собираемся использовать модуль операционной системы, в котором есть функция с именем listdir(), которая будет возвращать имена файлов и каталогов из вашего рабочего каталога:

```bash
>>> os.listdir()
['Public', 'python_learning', '.ICEauthority', '.python_history', 'work', '.bashrc', 'Pictures', '.gnupg', '.cache', '.bash_logout', '.sudo_as_admin_successful', '.bash_history', '.config', '.viminfo', 'Desktop', 'Documents', 'examples.desktop', 'Videos', '.ssh', 'Templates', '.profile', 'dir', '.pam_environment', 'Downloads', '.local', '.dbus', 'Music', '.mozilla']
>>> 
```

## Переименование каталога

В модуле os на Python есть функция rename(), которая помогает изменить название каталога:

```bash
>>> os.rename('work', 'work1')
>>> os.listdir()
['Public', 'work1', 'python_learning', '.ICEauthority', '.python_history', '.bashrc', 'Pictures', '.gnupg', '.cache', '.bash_logout', '.sudo_as_admin_successful', '.bash_history', '.config', '.viminfo', 'Desktop', 'Documents', 'examples.desktop', 'Videos', '.ssh', 'Templates', '.profile', 'dir', '.pam_environment', 'Downloads', '.local', '.dbus', 'Music', '.mozilla']
>> 
```

## Копирование, перемещение, переименование и удаление данных

Мы познакомимся с четырьмя основными операциями, которые системные администраторы выполняют с данными: копированием, перемещением, переименованием и удалением. В Python есть встроенный модуль под названием shutil, который может выполнять эти задачи. Используя модуль shutil, мы также можем выполнять высокоуровневые операции с данными. Чтобы использовать модуль shutil в вашей программе, просто напишите инструкцию import shutil. Модуль shutil предлагает несколько функций, которые поддерживают операции копирования и удаления файлов. Давайте рассмотрим эти операции одну за другой.

## Копирование данных

В этом разделе мы увидим, как мы можем копировать файлы с помощью модуля shutil. Для этого сначала мы создадим файл hello.py и введем в него некоторый текст.

```python
hello.py:

print ("")
print ("Hello World\n")
print ("Hello Python\n")
```

Теперь мы напишем код для копирования в скрипт shutil\_copy\_example.py. Запишем в него следующее содержимое:

```python
import shutil
import os
shutil.copy('hello.py', 'welcome.py')
print("Copy Successful")
```

Запустите скрипт следующим образом:

```sh
$ python3 shutil_copy_example.py

Output:
Copy Successful
```

Проверьте наличие скрипта welcome.py, и вы увидите, что содержимое hello.py успешно скопировано в welcome.py.

## Перемещение данных

Мы увидим, как мы можем переместить данные. Для этой цели мы будем использовать shutil.move(). shutil.move(источник, пункт назначения) переместит файл из источника в пункт назначения. Теперь мы создадим shutil\_move\_example.py скрипт и запишем в него следующее содержимое:

```python
import shutil
shutil.move('/home/student/sample.txt', '/home/student/Desktop/.')
```

Запустите скрипт следующим образом:

```sh
$ python3 shutil_move_example.py
```

В этом скрипте нам нужно переместить файл sample.txt, который находится в каталоге /home/student. /home/student - это наша исходная папка, а /home/student/Desktop - наша целевая папка. Итак, после запуска скрипта sample.txt будет перемещен из каталога /home/student в каталог /home/student/Desktop.

## Переименование данных

В предыдущем разделе мы узнали, как можно использовать shutil.move() для перемещения файлов из источника в место назначения. С помощью shutil.move() файлы можно переименовывать. Создайте скрипт shutil\_rename\_example.py и запишите в него следующее содержимое:

```python
import shutil
shutil.move('hello.py', 'hello_renamed.py')
```

Запустите скрипт следующим образом:

```sh
$ python3 shutil_rename_example.py
```

Теперь убедитесь, что ваше имя файла будет переименовано hello\_renamed.py.

## Удаление данных

Мы узнаем, как удалять файлы и папки с помощью модуля os на Python. Метод remove() модуля os удалит файл. Если вы попытаетесь удалить каталог с помощью этого метода, это выдаст ошибку OSError. Чтобы удалить каталоги, используйте rmdir().

Теперь создайте скрипт os\_remove\_file\_directory.py и запишите в него следующее содержимое:

```python
import os
os.remove('sample.txt')
print("File removed successfully")
os.rmdir('work1')
print("Directory removed successfully")
```

Запустите скрипт следующим образом:

```sh
$ python3 os_remove_file_directory.py

Output:
File removed successfully
Directory removed successfully
```

## Работа с путями

Теперь мы познакомимся с методом os.path(). Он используется для манипулирования путями. В этом разделе мы рассмотрим некоторые функции, которые модуль os предлагает для путей.

Запустите консоль python3:

```sh
student@ubuntu:~$ python3
Python 3.6.6 (default, Sep 12 2018, 18:26:19)
[GCC 8.0.1 20180414 (experimental) [trunk revision 259383]] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>
```

os.path.abspath(путь): возвращает абсолютный путь для заданного пути.

```bash
>>> import os
>>> os.path.abspath('sample.txt')
'/home/student/work/sample.txt'
```

os.path.dirname(путь): используется для получения пути к родительскому каталогу по указанному пути до файла или каталога.

```bash
>>> os.path.dirname('/home/student/work/sample.txt')
'/home/student/work'
```

os.path.basename(путь) в Python используется для получения базового имени файла или пути. Она возвращает последний компонент пути, исключая любые директории.

```python
>>> os.path.basename('/home/student/work/sample.txt')
'sample.txt'
```

os.path.exists(путь): Возвращает значение True, если путь ссылается на существующий путь

```python
>>> os.path.exists('/home/student/work/sample.txt')
True
```

* os.path.getsize(путь): Возвращает размер введенного пути в байтах.

```python
>>> os.path.getsize('/home/student/work/sample.txt')
39
```

os.path.isfile(путь): Проверяет, является ли введенный путь существующим файлом или нет. Возвращает значение True, если это файл.

```python
>>> os.path.isfile('/home/student/work/sample.txt')
True
```

os.path.isdir(путь): проверяет, является ли введенный путь существующим каталогом или нет. Возвращает значение True, если это каталог.

```python
>>> os.path.isdir('/home/student/work/sample.txt')
False
```

## Сравнение данных

Мы узнаем о том, как сравнивать данные в Python. Для этой цели мы будем использовать модуль pandas.

Pandas - это библиотека анализа данных с открытым исходным кодом, которая предоставляет структуры данных и простые в использовании инструменты анализа данных. Это упрощает импорт и анализ данных.

Прежде чем приступить к рассмотрению примера, убедитесь, что в вашей системе установлен pandas. Вы можете установить pandas следующим образом

```bash
pip3 install pandas     --- For Python3

or

pip install pandas       --- For python2
```

Мы рассмотрим пример сравнения данных с помощью pandas. Изначально мы создадим два csv-файла: student 1.csv и student 2.csv. Мы сравним данные из этих двух csv-файлов, и на выходе он должен вернуть результаты сравнения. Создайте два csv-файла следующим образом:

Создайте содержимое файла student1.csv следующим образом:

```
Id,Name,Gender,Age,Address
101,John,Male,20,New York
102,Mary,Female,18,London
103,Aditya,Male,22,Mumbai
104,Leo,Male,22,Chicago
105,Sam,Male,21,Paris
106,Tina,Female,23,Sydney
```

Создайте содержимое файла student2.csv следующим образом:

```
Id,Name,Gender,Age,Address
101,John,Male,21,New York
102,Mary,Female,20,London
103,Aditya,Male,22,Mumbai
104,Leo,Male,23,Chicago
105,Sam,Male,21,Paris
106,Tina,Female,23,Sydney
```

Теперь мы создадим compare\_data.py скрипт и запишем в него следующее содержимое:

```python
import pandas as pd
df1 = pd.read_csv("student1.csv")
df2 = pd.read_csv("student2.csv")
s1 = set([ tuple(values) for values in df1.values.tolist()])
s2 = set([ tuple(values) for values in df2.values.tolist()])
s1.symmetric_difference(s2)
print (pd.DataFrame(list(s1.difference(s2))), '\n')
print (pd.DataFrame(list(s2.difference(s1))), '\n')
```

Запустите скрипт следующим образом:

```bash
$ python3 compare_data.py

Output:
     0     1       2   3         4
0  102  Mary  Female  18    London
1  104   Leo    Male  22   Chicago
2  101  John    Male  20  New York


     0     1       2   3         4
0  101  John    Male  21  New York
1  104   Leo    Male  23   Chicago
2  102  Mary  Female  20    London
```

В предыдущем примере мы сравниваем данные из двух csv-файлов: student1.csv и student2.csv. Сначала мы преобразовали наши фреймы данных (df1, df2) в наборы (s1, s2). Затем мы использовали набор symmetric\_difference(). Таким образом, он проверит симметричную разницу между s1 и s2, а затем мы напечатаем результат.

## Объединение данных

Мы собираемся узнать о том, как объединять данные в Python. Для этого мы будем использовать библиотеку pandas в Python. Чтобы объединить данные, мы собираемся использовать два csv-файла, которые уже были созданы в предыдущем разделе, student1.csv и student2.csv.

Теперь создайте скрипт merge\_data.py и напишите в нем следующий код:

```
import pandas as pd
df1 = pd.read_csv("student1.csv")
df2 = pd.read_csv("student2.csv")
result = pd.concat([df1, df2])
print(result)
```

Запустите скрипт:

```bash
$ python3 merge_data.py

Output:
    Id    Name  Gender  Age   Address
0  101    John    Male   20  New York
1  102    Mary  Female   18    London
2  103  Aditya    Male   22    Mumbai
3  104     Leo    Male   22   Chicago
4  105     Sam    Male   21     Paris
5  106    Tina  Female   23    Sydney
0  101    John    Male   21  New York
1  102    Mary  Female   20    London
2  103  Aditya    Male   22    Mumbai
3  104     Leo    Male   23   Chicago
4  105     Sam    Male   21     Paris
5  106    Tina  Female   23    Sydney
```

## Файлы и каталоги, соответствующие шаблону

В этом разделе мы узнаем о сопоставлении шаблонов для файлов и каталогов. В Python есть модуль glob, который используется для поиска имен файлов и каталогов, соответствующих определенным шаблонам.

Теперь мы рассмотрим пример. Сначала создайте скрипт pattern\_match.py и напишите в нем следующее содержимое:

```python
import glob
file_match = glob.glob('*.txt')
print(file_match)
file_match = glob.glob('[0-9].txt')
print(file_match)
file_match = glob.glob('**/*.txt', recursive=True)
print(file_match)
file_match = glob.glob('**/', recursive=True)
print(file_match)
```

Запустите скрипт:

```
$ python3 pattern_match.py

Output:
['file1.txt', 'filea.txt', 'fileb.txt', 'file2.txt', '2.txt', '1.txt', 'file.txt']
['2.txt', '1.txt']
['file1.txt', 'filea.txt', 'fileb.txt', 'file2.txt', '2.txt', '1.txt', 'file.txt', 'dir1/3.txt', 'dir1/4.txt']
['dir1/']
```

В предыдущем примере мы использовали модуль Python glob для сопоставления с шаблоном. glob (pathname) вернет список имен, совпадающих с именем пути. В нашем скрипте мы передали три имени путей в трех разных функциях glob(). В первом glob() мы передали путь к файлу как \*.txt; это вернет все имена файлов с расширением .txt.

Во втором globe() мы передали \[0-9].txt; это вернет имена файлов, начинающиеся с цифры. В третьем globe() мы передали \*\*/\*.txt, который вернет имена файлов, а также имена каталогов. Он также вернет имена файлов из этих каталогов. В четвертом globe() мы передали \*\*/, который будет возвращать только имена каталогов.

## Метаданные: данные о данных

В этом разделе мы познакомимся с модулем pyPdf, который помогает извлекать метаданные из pdf-файла. Но сначала разберемся, что такое метаданные? Метаданные - это данные о данных. Метаданные - это структурированная информация, которая описывает первичные данные. Метаданные - это краткое изложение этих данных. Он содержит основную информацию о ваших фактических данных. Он помогает найти конкретный экземпляр ваших данных.

{% hint style="info" %}
Убедитесь, что в вашем каталоге есть pdf-файл, из которого вы хотите извлечь информацию.
{% endhint %}

Во-первых, мы должны установить модуль pyPdf следующим образом:

```bash
pip install pyPdf
```

Теперь мы напишем скрипт metadata\_example.py и посмотрим, как мы получим из него информацию о метаданных. Мы собираемся написать этот скрипт на Python 2:

```python
import pyPdf
def main():
            file_name = '/home/student/sample_pdf.pdf'
            pdfFile = pyPdf.PdfFileReader(file(file_name,'rb'))
            pdf_data = pdfFile.getDocumentInfo()
            print ("----Metadata of the file----")
            for md in pdf_data:
                        print (md+ ":" +pdf_data[md])
if __name__ == '__main__':
            main()
```

Запустите скрипт:

```bash
student@ubuntu:~$ python metadata_example.py
----Metadata of the file----
/Producer:Acrobat Distiller Command 3.0 for SunOS 4.1.3 and later (SPARC)
/CreationDate:D:19980930143358
```

В предыдущем сценарии мы использовали модуль pyPdf в Python 2. Сначала мы создали переменную file\_name, в которой хранится путь к нашему pdf-файлу. Данные считываются с помощью функции чтения PdfFileReader(). Переменная pdf\_data будет содержать информацию о вашем pdf-файле. Наконец, мы написали цикл for для получения информации о метаданных.

