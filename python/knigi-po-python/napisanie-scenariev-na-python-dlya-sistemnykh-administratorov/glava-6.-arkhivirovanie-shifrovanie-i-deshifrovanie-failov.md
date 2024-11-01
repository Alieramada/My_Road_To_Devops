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

<pre class="language-python"><code class="lang-python"><strong>import shutil
</strong><strong>import os
</strong><strong>import tarfile
</strong><strong>print('creating archive')
</strong><strong>shutil.make_archive('work', 'tar', root_dir='..', base_dir='work',)
</strong><strong>print('\nArchive contents:')
</strong><strong>with tarfile.open('work.tar', 'r') as t_file:
</strong><strong> for names in t_file.getnames():
</strong><strong> print(names)
</strong><strong>os.system('touch sample.txt')
</strong><strong>print('adding sample.txt')
</strong><strong>with tarfile.open('work.tar', mode='a') as t:
</strong><strong> t.add('sample.txt')
</strong><strong>print('contents:',)
</strong><strong>with tarfile.open('work.tar', mode='r') as t:
</strong><strong> print([m.name for m in t.getmembers()])
</strong></code></pre>

Запустите скрипт:

<pre class="language-sh"><code class="lang-sh"><strong>student@ubuntu:~/work$ python3 add_to_archive.py
</strong><strong>Output :
</strong><strong>creating archive
</strong><strong>Archive contents:
</strong><strong>work
</strong><strong>work/bye.py
</strong><strong>work/shutil_make_archive.py
</strong><strong>work/check_archive_file.py
</strong><strong>work/welcome.py
</strong><strong>work/add_to_archive.py
</strong><strong>work/shutil_unpack_archive.py
</strong><strong>work/hello.py
</strong><strong>adding sample.txt
</strong><strong>contents:
</strong><strong>['work', 'work/bye.py', 'work/shutil_make_archive.py', 'work/check_archive_file.py', 'work/welcome.py', 'work/add_to_archive.py', 'work/shutil_unpack_archive.py', 'work/hello.py', 'sample.txt']
</strong></code></pre>

В этом примере сначала мы создали архивный файл, используя shutil.make\_archive(), а затем распечатали содержимое архивированного файла. Затем мы создали файл sample.txt в следующей инструкции. Теперь мы хотим добавить это sample.txt в уже созданный work.tar. Здесь мы использовали режим добавления, a. И далее мы снова отображаем содержимое заархивированного файла.

Теперь мы узнаем о том, как мы можем прочитать метаданные из архивного файла. Функция getmembers() загрузит метаданные из файлов. Создайте скрипт с именем read\_metadata.py и запишите в него следующее содержимое:

<pre class="language-python"><code class="lang-python"><strong>import tarfile
</strong><strong>import time
</strong><strong>with tarfile.open('work.tar', 'r') as t:
</strong><strong>            for file_info in t.getmembers():
</strong><strong>                        print(file_info.name)
</strong><strong>                        print("Size   :", file_info.size, 'bytes')
</strong><strong>                        print("Type   :", file_info.type)
</strong><strong>                        print()
</strong></code></pre>

Запустите скрипт:

<pre class="language-sh"><code class="lang-sh"><strong>student@ubuntu:~/work$ python3 read_metadata.py
</strong><strong>Output:
</strong><strong>work/bye.py
</strong><strong>Size : 30 bytes
</strong><strong>Type : b'0' 
</strong><strong>work/shutil_make_archive.py
</strong><strong>Size : 243 bytes
</strong><strong>Type : b'0'
</strong><strong>work/check_archive_file.py
</strong><strong>Size : 233 bytes
</strong><strong>Type : b'0'
</strong> 
<strong>work/welcome.py
</strong><strong>Size : 48 bytes
</strong><strong>Type : b'0'
</strong> 
<strong>work/add_to_archive.py
</strong><strong>Size : 491 bytes
</strong><strong>Type : b'0'
</strong> 
<strong>work/shutil_unpack_archive.py
</strong><strong>Size : 279 bytes
</strong><strong>Type : b'0'
</strong></code></pre>

Теперь мы извлекем содержимое из архива, используя функцию extract all(). Для этого создайте скрипт с именем extract\_contents.py и напишите в нем следующий код:

<pre class="language-python"><code class="lang-python"><strong>import tarfile
</strong><strong>import os
</strong><strong>os.mkdir('work')
</strong><strong>with tarfile.open('work.tar', 'r') as t:
</strong><strong>            t.extractall('work')
</strong><strong>print(os.listdir('work'))
</strong></code></pre>

Запустите скрипт:

<pre class="language-bash"><code class="lang-bash"><strong>student@ubuntu:~/work$ python3 extract_contents.py
</strong></code></pre>

Проверьте свой текущий рабочий каталог, и вы найдете work/ каталог. Перейдите в этот каталог, и вы сможете найти извлеченные файлы.
