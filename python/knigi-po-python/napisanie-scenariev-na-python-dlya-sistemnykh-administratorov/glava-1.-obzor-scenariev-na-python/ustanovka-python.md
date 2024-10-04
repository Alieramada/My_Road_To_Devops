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

## &#x20;Установка на Mac

Чтобы установить python3, сначала в нашей системе должен быть установлен brew. Чтобы установить brew в вашей системе, выполните следующую команду:

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Выполнив предыдущую команду. brew будет установлен. Теперь мы установим python3 с помощью brew:

```
brew install python3
```



## Установка и использование pip для установки пакетов

Для установки пакетов В Linux установите pip следующим образом:

<pre><code><strong>sudo apt install python-pip --- This will install pip for python 2.
</strong><strong>sudo apt install python3-pip --- This will install pip for python 3.
</strong><strong>
</strong></code></pre>

В Windows установите pip следующим образом:

<pre><code><strong>python -m pip install pip
</strong><strong>
</strong></code></pre>

## &#x20;Установка **Jupyter notebook**

**Jupyter-notebook** — это среда разработки, где сразу можно видеть результат выполнения кода и его отдельных фрагментов. Отличие от традиционной среды разработки в том, что код можно разбить на куски и выполнять их в произвольном порядке. Представьте, что вы можете написать кусочек кода на салфетке и сказать салфетке: «Выполнись».

{% embed url="https://habr.com/ru/articles/718736/" %}

Чтобы установить Jupyter Notebook, скачайте Anaconda

Установите загруженную версию Anaconda и следуйте инструкциям мастера.

Установите Jupyter с помощью pip:

```
pip install jupyter
```

В Linux pip install jupyter установит Jupyter для python 2. Если вы хотите установить jupyter для python 3, выполните следующую команду:

```
pip3 install jupyter
```

## &#x20;Установка и использование виртуальной среды

Теперь мы увидим, как установить виртуальную среду и как ее активировать.

Чтобы установить виртуальную среду в Linux, выполните следующие действия:

&#x20;1\. Сначала проверьте, установлен ли pip или нет. Мы собираемся установить pip для python3:

```
sudo apt install python3-pip
```

2. Установите виртуальную среду с помощью pip3:

```
sudo pip3 install virtualenv
```

&#x20;3\. Теперь мы создадим виртуальную среду. Вы можете дать ей любое имя; я назвал ее pythonenv:

```
virtualenv pythonenv
```

4. Активируйте свою виртуальную среду:

```
source venv/bin/activate
```

После завершения вашей работы вы можете деактивировать virtualenv, используя следующую команду:

```
deactivate
```

В Windows запустите команду pip install virtualenv для установки виртуальной среды. Шаги по установке virtualenv такие же, как и в Linux.

## &#x20;Установка Geany и PyCharm



Загрузите Geany с сайта [https://www.geany.org/download/releases](https://www.geany.org/download/releases) и скачайте необходимые двоичные файлы. Следуйте инструкциям при установке.

Скачайте PyCharm с сайта [https://www.jetbrains.com/pycharm/download/#section=windows](https://www.jetbrains.com/pycharm/download/#section=windows) и следуйте инструкциям.

