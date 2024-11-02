# Глава 6. Архивирование, шифрование и дешифрование файлов

В предыдущей главе мы узнали об обработке файлов, каталогов и данных. Мы также узнали о модуле tarfile. В этой главе мы узнаем об архивировании, шифровании и расшифровке файлов. Архивирование играет важную роль в управлении файлами, каталогами и данными. Но сначала разберемся, что такое архивация? Архивация - это процесс, при котором файлы и каталоги сохраняются в одном файле. В Python есть модуль tarfile для создания таких архивных файлов.

В этой главе мы рассмотрим следующие темы:

Создание и распаковка архивов&#x20;

Архивы Tar

&#x20;Создание ZIP-архивов

&#x20;Шифрование и дешифрование файлов

## Создание и распаковка архивов&#x20;

В этом разделе мы узнаем о том, как создавать и распаковывать архивы с помощью модуля shutil в Python. В модуле shutil есть функция make\_archive(), которая создает новый архивный файл. Используя make\_archive(), мы можем заархивировать весь каталог вместе с его содержимым

## Создание архивов

Теперь мы собираемся написать скрипт под названием shutil\_make\_archive.py и поместить в него следующее содержимое:

```python
import tarfile
import shutil
import sys

shutil.make_archive(
            'work_sample', 'gztar',
            root_dir='..',
            base_dir='work',
)
print('Archive contents:')
with tarfile.open('work_sample.tar.gz', 'r') as t_file:
  for names in t_file.getnames():
    print(names)
```

Запустите скрипт:

```sh
$ python3 shutil_make_archive.py
Archive contents:
work
work/bye.py
work/shutil_make_archive.py
work/welcome.py
work/hello.py
```

В предыдущем примере для создания архивного файла мы использовали модули shutil и tarfile из Python. В shutil.make\_archive() мы указали work\_sample, который будет именем архивного файла в формате gz. Мы указали имя нашего рабочего каталога в атрибуте base directory. Наконец, мы напечатали имена файлов, которые будут заархивированы.

## Распаковка архивов

Для распаковки архивов в модуле shutil есть функция unpack\_archive(). Используя эту функцию, мы можем извлечь архивные файлы. Мы передали имя файла архива и каталог, из которого мы хотим извлечь содержимое. Если имя каталога не указано, то содержимое будет извлечено в ваш текущий рабочий каталог.

Теперь создайте скрипт с именем shutil\_unpack\_archive.py и напишите в нем следующий код:

```bash
import pathlib
import shutil
import sys
import tempfile
with tempfile.TemporaryDirectory() as d:
 shutil.unpack_archive('work_sample.tar.gz', extract_dir='/home/student/work',)
 prefix_len = len(d) + 1
 for extracted in pathlib.Path(d).rglob('*'):
 print(str(extracted)[prefix_len:])
```

Запустите скрипт:

```bash
student@ubuntu:~/work$ python3 shutil_unpack_archive.py
```

Теперь проверьте свой work/ каталог, и вы найдете в нем  папку  work/, в которой будут находиться извлеченные файлы.

## Архивы Tar&#x20;

В этом разделе мы познакомимся с модулем tarfile. Мы также узнаем о проверке введенного имени файла, чтобы определить, является ли оно допустимым именем файла архива или нет. Мы рассмотрим, как добавить новый файл в уже заархивированный файл, как мы можем считывать метаданные с помощью модуля tarfile и как извлекать файлы из архива с помощью функции extractall().

Сначала мы проверим, является ли введенное имя файла допустимым файлом архива или нет. Для проверки этого в модуле tarfile есть функция is\_tar file(), которая возвращает логическое значение.

Создайте скрипт с именем check\_archive\_file.py и запишите в него следующее содержимое:

```python
import tarfile

for f_name in ['hello.py', 'work.tar.gz', 'welcome.py', 'nofile.tar', 'sample.tar.xz']:
  try:
    print('{:} {}'.format(f_name, tarfile.is_tarfile(f_name)))
  except IOError as err:
    print('{:} {}'.format(f_name, err))
```

Запустите скрипт:

```sh
hello.py          False
work.tar.gz      True
welcome.py     False
nofile.tar         [Errno 2] No such file or directory: 'nofile.tar'
sample.tar.xz   True
```

Итак, tar-файл.is\_tar-файл() проверит каждое имя файла, упомянутое в списке. Файлы hello.py, welcome.py не являются tar-файлами, поэтому мы получили логическое значение False. work.tar.gz и выберем файлы.tar.xz или tar, чтобы мы получили логическое значение True. И в нашем каталоге нет такого файла, как file.tar, поэтому мы получили исключение, как мы и написали в нашем скрипте.

Теперь мы собираемся добавить новый файл в наш уже созданный архивный файл. Создайте скрипт с именем add\_to\_archive.py и напишите в нем следующий код:

```python
import shutil
import os
import tarfile
print('creating archive')
shutil.make_archive('work', 'tar', root_dir='..', base_dir='work',)
print('\nArchive contents:')
with tarfile.open('work.tar', 'r') as t_file:
 for names in t_file.getnames():
 print(names)
os.system('touch sample.txt')
print('adding sample.txt')
with tarfile.open('work.tar', mode='a') as t:
 t.add('sample.txt')
print('contents:',)
with tarfile.open('work.tar', mode='r') as t:
 print([m.name for m in t.getmembers()])
```

Запустите скрипт:

```sh
student@ubuntu:~/work$ python3 add_to_archive.py
Output :
creating archive
Archive contents:
work
work/bye.py
work/shutil_make_archive.py
work/check_archive_file.py
work/welcome.py
work/add_to_archive.py
work/shutil_unpack_archive.py
work/hello.py
adding sample.txt
contents:
['work', 'work/bye.py', 'work/shutil_make_archive.py', 'work/check_archive_file.py', 'work/welcome.py', 'work/add_to_archive.py', 'work/shutil_unpack_archive.py', 'work/hello.py', 'sample.txt']
```

В этом примере сначала мы создали архивный файл, используя shutil.make\_archive(), а затем распечатали содержимое архивированного файла. Затем мы создали файл sample.txt в следующей инструкции. Теперь мы хотим добавить это sample.txt в уже созданный work.tar. Здесь мы использовали режим добавления, a. И далее мы снова отображаем содержимое заархивированного файла.

Теперь мы узнаем о том, как мы можем прочитать метаданные из архивного файла. Функция getmembers() загрузит метаданные из файлов. Создайте скрипт с именем read\_metadata.py и запишите в него следующее содержимое:

```python
import tarfile
import time
with tarfile.open('work.tar', 'r') as t:
            for file_info in t.getmembers():
                        print(file_info.name)
                        print("Size   :", file_info.size, 'bytes')
                        print("Type   :", file_info.type)
                        print()
```

Запустите скрипт:

```sh
student@ubuntu:~/work$ python3 read_metadata.py
Output:
work/bye.py
Size : 30 bytes
Type : b'0' 
work/shutil_make_archive.py
Size : 243 bytes
Type : b'0'
work/check_archive_file.py
Size : 233 bytes
Type : b'0'
 
work/welcome.py
Size : 48 bytes
Type : b'0'
 
work/add_to_archive.py
Size : 491 bytes
Type : b'0'
 
work/shutil_unpack_archive.py
Size : 279 bytes
Type : b'0'
```

Теперь мы извлекем содержимое из архива, используя функцию extract all(). Для этого создайте скрипт с именем extract\_contents.py и напишите в нем следующий код:

```python
import tarfile
import os
os.mkdir('work')
with tarfile.open('work.tar', 'r') as t:
            t.extractall('work')
print(os.listdir('work'))
```

Запустите скрипт:

```bash
student@ubuntu:~/work$ python3 extract_contents.py
```

Проверьте свой текущий рабочий каталог, и вы найдете work/ каталог. Перейдите в этот каталог, и вы сможете найти извлеченные файлы.

## Cоздание ZIP файлов&#x20;

В этом разделе мы будем работать с ZIP-файлами. Мы узнаем о модуле zipfile в python, о том, как создавать ZIP-файлы, как проверить, является ли введенное имя файла допустимым именем zip-файла или нет, о чтении метаданных и так далее.

Сначала мы узнаем, как создать zip-файл с помощью функции make\_archive() модуля shutil. Создайте скрипт с именем make\_zip\_file.py и напишите в нем следующий код:

```python
import shutil
shutil.make_archive('work', 'zip', 'work')
```

Запустите скрипт:

```sh
student@ubuntu:~$ python3 make_zip_file.py
```

Теперь проверьте свой текущий рабочий каталог, и вы увидите work.zip.

Теперь мы проверим, является ли введенное имя файла zip-файлом или нет. Для этой цели в модуле zipfile есть функция is\_zip file().

Создайте скрипт с именем check\_zip\_file.py и запишите в него следующее содержимое:

<pre class="language-python"><code class="lang-python"><strong>import zipfile
</strong><strong>for f_name in ['hello.py', 'work.zip', 'welcome.py', 'sample.txt', 'test.zip']:
</strong><strong>            try:
</strong><strong>                        print('{:}           {}'.format(f_name, zipfile.is_zipfile(f_name)))
</strong><strong>            except IOError as err:
</strong><strong>                        print('{:}           {}'.format(f_name, err))
</strong></code></pre>

Запустите скрипт:

```sh
student@ubuntu:~$ python3 check_zip_file.py
Output :
hello.py          False
work.zip         True
welcome.py     False
sample.txt       False
test.zip            True
```

В этом примере мы использовали цикл for, в котором мы проверяем имена файлов в списке. Функция is\_zip file() проверяет имена файлов одно за другим и выдает логические значения в качестве результата.

Теперь мы увидим, как мы можем прочитать метаданные из архивированного ZIP-файла, используя модуль zipfile в Python. Создайте скрипт с именем read\_metadata.py и запишите в него следующее содержимое:

```python
import zipfile

def meta_info(names):
            with zipfile.ZipFile(names) as zf:
                        for info in zf.infolist():
                                    print(info.filename)
                                    if info.create_system == 0:
                                                system = 'Windows'
                                    elif info.create_system == 3:
                                                system = 'Unix'
                                    else:
                                                system = 'UNKNOWN'
                                    print("System         :", system)
                                    print("Zip Version    :", info.create_version)
                                    print("Compressed     :", info.compress_size, 'bytes')
                                    print("Uncompressed   :", info.file_size, 'bytes')
                                    print()

if __name__ == '__main__':
    meta_info('work.zip')
```

Запустите скрипт:

```sh
student@ubuntu:~$ python3 read_metadata.py
Output:
sample.txt
System         : Unix
Zip Version    : 20
Compressed     : 2 bytes
Uncompressed   : 0 bytes
 
bye.py
System         : Unix
Zip Version    : 20
Compressed     : 32 bytes
Uncompressed   : 30 bytes
 
extract_contents.py
System         : Unix
Zip Version    : 20
Compressed     : 95 bytes
Uncompressed   : 132 bytes
 
shutil_make_archive.py
```

Чтобы получить информацию о метаданных zip-файла, мы использовали метод infolist() класса ZipFile.

## Шифрование и дешифрование файлов

В этом разделе мы узнаем о модуле pyAesCrypt в Python. pyAesCrypt - это модуль шифрования файлов, который использует AES256-CBC для шифрования/дешифрования файлов и двоичных потоков.

Установите pyAesCrypt следующим образом:

```sh
pip3 install pyAesCrypt
```

Создайте скрипт с именем file\_encrypt.py и напишите в нем следующий код:

```python
import pyAesCrypt

from os import stat, remove
# encryption/decryption buffer size - 64K
bufferSize = 64 * 1024
password = "#Training"
with open("sample.txt", "rb") as fIn:
 with open("sample.txt.aes", "wb") as fOut:
 pyAesCrypt.encryptStream(fIn, fOut, password, bufferSize)
# get encrypted file size
encFileSize = stat("sample.txt.aes").st_size 
```

Запустите скрипт:

```sh
student@ubuntu:~/work$ python3 file_encrypt.py
Output :
```

Проверьте ваш текущий рабочий каталог. В нем вы найдете зашифрованный файл sample.txt.aes.

В этом примере мы уже упоминали размер буфера и пароль. Далее мы упомянули имя нашего файла, который будет зашифрован. В  секции encryptStream мы упомянули fIn, который является нашим файлом для шифрования, и fOut, который является нашим именем файла после шифрования. Мы сохранили наш зашифрованный файл как sample.txt.aes.

Теперь мы расшифруем файл sample.txt.aes, чтобы получить содержимое файла. Создайте скрипт с именем file\_decrypt.py и запишите в него следующее содержимое:

```python
import pyAesCrypt
from os import stat, remove
bufferSize = 64 * 1024
password = "#Training"
encFileSize = stat("sample.txt.aes").st_size
with open("sample.txt.aes", "rb") as fIn:
 with open("sampleout.txt", "wb") as fOut:
 try:
 pyAesCrypt.decryptStream(fIn, fOut, password, bufferSize, encFileSize)
 except ValueError:
 remove("sampleout.txt")
```

Запустите скрипт:

```sh
student@ubuntu:~/work$ python3 file_decrypt.py
```

Теперь проверьте свой текущий рабочий каталог. Будет создан файл с именем sampleout.txt. Это ваш расшифрованный файл.

В этом примере мы указали имя файла для расшифровки, которое является sample.text.aes. Далее наш расшифрованный файл будет выглядеть как sampleout.txt. В decryptStream() мы упомянули fIn, который является нашим файлом для расшифровки, и fOut, который является именем расшифрованного файла.

## Итоги

В этой главе мы узнали о создании и извлечении архивных файлов. Архивирование играет важную роль в управлении файлами, каталогами и данными. Оно также позволяет сохранять файлы и каталоги в одном файле.

Мы подробно ознакомились с модулями tarfile и zipfile на Python, которые позволяют создавать, извлекать и тестировать архивные файлы. Вы сможете добавлять новый файл в уже заархивированный файл, считывать метаданные, извлекать файлы из архива. Вы также узнали о шифровании и дешифровании файлов с помощью модуля pyAescrypt.

В следующей главе вы узнаете об обработке текста и регулярных выражениях в python. В Python есть очень мощная библиотека под названием regular expressions, которая выполняет такие задачи, как поиск и извлечение данных.

## Вопросы

1. Можем ли мы сжать данные, используя защиту паролем? если да, то как?

Для сжатия данных с использованием пароля в Python вы можете использовать модуль pyAesCrypt для шифрования, а затем применить сжатие с помощью модуля tarfile. Вот пример того, как это можно сделать:

```python
import os
from pyAesCrypt import encrypt, decrypt
from tarfile import TarFile

# Функция для сжатия и шифрования файла
def compress_and_encrypt(input_file, output_file):
    # Шифрование файла
    encrypted_file = encrypt(input_file)

    # Создание архива с зашифрованным файлом
    with TarFile.open(output_file, "w") as tar:
        tar.add(encrypted_file)
        os.remove(encrypted_file)  # Удаление зашифрованного файла после добавления в архив

    return output_file

# Пример использования функции
input_file = "original_data.txt"
output_file = "compressed_encrypted.tar"
compress_and_encrypt(input_file, output_file)

# Распаковка и расшифровка данных
with TarFile.open(output_file) as tar:
    member = tar.getmember("original_data.txt")
    decrypted_file = decrypt(member.name)
    with open(decrypted_file, 'wb') as outfile:
        outfile.write(tar.extractfile(member).read())

# Удаление сжатого зашифрованного файла
os.remove(output_file)
```

В этом примере мы используем модуль pyAesCrypt для симметричного шифрования данных, а затем применяем сжатие с помощью tar-архивации. Это позволяет не только защитить данные паролем, но и уменьшить их размер для удобства хранения и передачи.

2. &#x20;Что такое контекстный менеджер в python?

Контекстный менеджер в Python — это конструкция, которая позволяет автоматически управлять ресурсами, такими как файлы, соединения с базой данных или блокировки. Он используется для обеспечения правильного освобождения ресурсов после завершения блока кода.

Контекстные менеджеры реализованы через два ключевых слова: `with` и `as`. Они позволяют определить блок кода, который будет выполняться внутри контекста менеджера. Когда выполнение блока завершается, контекстный менеджер гарантирует освобождение ресурсов.

Пример использования контекстного менеджера:

```python
python
with open("file.txt", "r") as file:
    content = file.read()

# После выполнения блока файл автоматически закрывается
```

В этом примере файл открывается для чтения и закрывается после выполнения блока кода внутри `with`. Это предотвращает ситуации, когда файл остаётся открытым и не освобождает ресурсы, что может привести к ошибкам.

3. Что такое pickling и unpickling?

Pickling в Python — это процесс сериализации объекта, то есть его преобразования в последовательность байтов, которая может быть сохранена или передана. Это позволяет сохранить состояние объекта и восстановить его позже.

Unpickling в Python — обратный процесс, который восстанавливает объект из последовательности байтов. Этот процесс позволяет получить доступ к данным объекта после их сохранения или передачи.

Для работы с pickling и unpickling используются модули pickle и dill. Они позволяют преобразовывать объекты в байтовые последовательности и обратно.

5. Какие существуют различные типы функций в python?

В Python существуют различные типы функций:

1. **Встроенные функции** — это функции, которые уже определены в Python и доступны для использования. Они включают в себя такие функции, как print(), len(), max() и другие.
2. **Пользовательские функции** — создаются программистом для выполнения определённых задач. Программист определяет имя функции, параметры и тело функции.
3. **Анонимные функции (лямбда-функции)** — это небольшие функции без имени, которые могут быть созданы с использованием ключевого слова lambda. Лямбда-функции часто используются в сочетании с функциями высшего порядка.
4. **Рекурсивные функции** — функции, которые вызывают сами себя для решения задачи. Рекурсия может быть мощным инструментом для работы с данными, имеющими древовидную структуру.
5. **Функции высшего порядка** — принимают другие функции в качестве аргументов или возвращают функции в качестве результатов. Примерами функций высшего порядка являются map(), filter() и reduce().
6. **Декораторы** — специальные функции, которые позволяют добавлять новое поведение к функциям без изменения их кода. Декораторы могут использоваться для добавления логирования, проверки прав доступа и других функций.
7. **Статические методы** — методы, определённые внутри класса, но не привязанные к конкретному экземпляру класса. Статические методы вызываются через класс, а не через экземпляр класса.
8. **Методы класса** — похожи на статические методы, но они могут получить доступ к состоянию класса через self. Методы класса вызываются через класс.
9. **Конструкторы** — специальная функция, которая вызывается при создании нового экземпляра класса. Конструкторы обычно используются для инициализации состояния объекта.

## Читать далее

Сжатие и архивирование данных: [https://docs.python.org/3/library/archiving.html](https://docs.python.org/3/library/archiving.html)

&#x20;Документация tempfile: [https://docs.python.org/2/library/tempfile.html ](https://docs.python.org/2/library/tempfile.html)

Документация по криптографии на Python: [https://docs.python.org/3/library/crypto.html](https://docs.python.org/3/library/crypto.html)

Документация по shutil: [https://docs.python.org/3](https://docs.python.org/3/library/shutil.html)
