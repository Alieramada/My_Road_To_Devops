# Установка Python

## **Установка на платформе Linux**&#x20;



В большинстве дистрибутивов Linux по умолчанию используется Python 2. В некоторые из них также включен Python 3.

Чтобы установить Python3 в Linux на базе Debian, выполните в терминале следующую команду:

<pre><code><strong>sudo apt install python3
</strong></code></pre>

sudo apt install python3 Чтобы установить python3&#x20;

На centos, выполните следующую команду в терминале:

<pre><code><strong>sudo yum install python3
</strong></code></pre>

sudo dnf install python3 Если вам не удается установить Python с помощью предыдущих команд, загрузите Python с сайта [`https://www.python.org/downloads/`](https://www.python.org/downloads/) и следуйте инструкциям.



<mark style="color:blue;background-color:green;">**В новых дистрибутивах установлен pyhon 3**</mark>

## **Установка на платформе Windows**

Для установки Python в Microsoft Windows вам необходимо загрузить исполняемый файл с сайта python.org и установить его. Загрузите python.exe с сайта [https://www.python.org/downloads/](https://www.python.org/downloads/) и выберите версию Python, которую вы хотите установить на свой компьютер. Затем дважды щелкните по загруженному exe-файлу и установите Python. В мастере установки есть флажок с надписью "Добавить Python в путь". Установите этот флажок и следуйте инструкциям по установке python3.

## Установка и использование pip

Для установки пакетов В Linux установите pip следующим образом:

<pre><code><strong>sudo apt install python-pip --- This will install pip for python 2.
</strong><strong>sudo apt install python3-pip --- This will install pip for python 3.
</strong><strong>
</strong></code></pre>

В Windows установите pip следующим образом:

<pre><code><strong>python -m pip install pip
</strong></code></pre>
