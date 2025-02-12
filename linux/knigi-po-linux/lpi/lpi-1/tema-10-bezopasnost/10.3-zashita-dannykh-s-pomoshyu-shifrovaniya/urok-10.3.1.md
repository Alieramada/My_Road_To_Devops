# Урок 10.3.1

## Введение <a href="#sec.110.3_01-in" id="sec.110.3_01-in"></a>

Защита данных с помощью шифрования имеет первостепенное значение во многих аспектах современного системного администрирования — особенно когда речь идёт об удалённом доступе к системам. В отличие от небезопасных решений, таких как _telnet_, _rlogin_ или _FTP_, протокол _SSH_ (_Secure Shell_) был разработан с учётом требований безопасности. Используя криптографию с открытым ключом, он аутентифицирует как хосты, так и пользователей и шифрует весь последующий обмен информацией. Кроме того, SSH можно использовать для создания _туннелей через порты_, которые, помимо прочего, позволяют передавать данные по незашифрованному протоколу через зашифрованное соединение SSH. Текущая рекомендуемая версия протокола SSH — 2.0. _OpenSSH_ — это бесплатная реализация протокола SSH с открытым исходным кодом.

В этом уроке мы рассмотрим базовую конфигурацию клиента _OpenSSH_, а также роль ключей хоста _OpenSSH_ на сервере. Также мы обсудим концепцию туннелей SSH-портов. Мы будем использовать два компьютера со следующей конфигурацией:

<table data-header-hidden><thead><tr><th></th><th></th><th width="174"></th><th></th><th></th></tr></thead><tbody><tr><td>Роль хоста</td><td>Операционная система</td><td>IP - адрес</td><td>Имя хоста</td><td>Пользователь</td></tr><tr><td>Клиент</td><td>Debian GNU/Linux 10 (бастер)</td><td><code>192.168.1.55</code></td><td><code>debian</code></td><td><code>carol</code></td></tr><tr><td>Сервер</td><td>openSUSE Leap 15.1</td><td><code>192.168.1.77</code></td><td><code>halof</code></td><td><code>ina</code></td></tr></tbody></table>

## Базовая Конфигурация и использование клиента OpenSSH <a href="#basic_openssh_client_configuration_and_usage" id="basic_openssh_client_configuration_and_usage"></a>

Хотя сервер и клиент OpenSSH поставляются в отдельных пакетах, обычно можно установить метапакет, который предоставит и то, и другое одновременно. Чтобы установить удалённый сеанс с SSH-сервером, используйте команду `ssh` и укажите пользователя, от имени которого вы хотите подключиться к удалённому компьютеру, а также IP-адрес или имя удалённого компьютера. При первом подключении к удалённому хосту вы получите такое сообщение:

```bash
carol@debian:~$ ssh ina@192.168.1.77
The authenticity of host '192.168.1.77 (192.168.1.77)' can't be established.
ECDSA key fingerprint is SHA256:5JF7anupYipByCQm2BPvDHRVFJJixeslmppi2NwATYI.
Are you sure you want to continue connecting (yes/no)?
```

После ввода `yes` и нажатия клавиши Enter вам будет предложено ввести пароль удалённого пользователя. При успешном вводе вам будет показано предупреждающее сообщение, а затем вы войдёте в систему удалённого хоста:

```bash
Warning: Permanently added '192.168.1.77' (ECDSA) to the list of known hosts.
Password:
Last login: Sat Jun 20 10:52:45 2020 from 192.168.1.4
Have a lot of fun...
ina@halof:~>
```

Сообщения говорят сами за себя: поскольку вы впервые установили соединение с `192.168.1.77` удалённым сервером, его подлинность не могла быть проверена по какой-либо базе данных. Таким образом, удалённый сервер предоставил `ECDSA key fingerprint` своего открытого ключа (с помощью `SHA256` хэш-функции). Как только вы приняли соединение, открытый ключ удалённого сервера был добавлен в базу данных _известных хостов_, что позволило аутентифицировать сервер для будущих подключений. Этот список открытых ключей _известных хостов_ хранится в файле `known_hosts`, который находится в `~/.ssh`:

```bash
ina@halof:~> exit
logout
Connection to 192.168.1.77 closed.
carol@debian:~$ ls .ssh/
known_hosts
```

И `.ssh` и `known_hosts` были созданы после установления первого удалённого подключения. `~/.ssh` — это каталог по умолчанию для пользовательских настроек и информации об аутентификации.

{% hint style="info" %}
Вы также можете использовать `ssh` для выполнения одной команды на удалённом хосте, а затем вернуться в свой локальный терминал (например, запустив `ssh ina@halof ls`).
{% endhint %}

Если вы используете одного и того же пользователя на локальном и удалённом хостах, нет необходимости указывать имя пользователя при установлении SSH-соединения. Например, если вы вошли в систему как пользователь `carol` на `debian` и хотите подключиться к `halof` также как пользователь `carol`, вы можете просто ввести `ssh 192.168.1.77` или `ssh halof` (если имя можно разрешить):

```bash
carol@debian:~$ ssh halof
Password:
Last login: Wed Jul  1 23:45:02 2020 from 192.168.1.55
Have a lot of fun...
carol@halof:~>
```

Теперь предположим, что вы устанавливаете новое удалённое соединение с хостом, у которого тот же IP-адрес, что и у `halof` (это часто случается, если вы используете DHCP в своей локальной сети). Вы получите предупреждение о возможности атаки _«человек посередине»_:

<pre class="language-bash"><code class="lang-bash"><strong>carol@debian:~$ ssh john@192.168.1.77
</strong>@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ECDSA key sent by the remote host is
SHA256:KH4q3vP6C7e0SEjyG8Wlz9fVlf+jmWJ5139RBxBh3TY.
Please contact your system administrator.
Add correct host key in /home/carol/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /home/carol/.ssh/known_hosts:1
  remove with:
  ssh-keygen -f "/home/carol/.ssh/known_hosts" -R "192.168.1.77"
ECDSA host key for 192.168.1.77 has changed and you have requested strict checking.
Host key verification failed.
</code></pre>

Поскольку вы не имеете дело с атакой _«человек посередине»_, вы можете спокойно добавить отпечаток открытого ключа нового хоста в `.ssh/known_hosts`. Как указано в сообщении, вы можете сначала использовать команду `ssh-keygen -f "/home/carol/.ssh/known_hosts" -R "192.168.1.77"` для удаления _неподходящего_ ключа (в качестве альтернативы вы можете использовать `ssh-keygen -R 192.168.1.77` для удаления всех ключей, принадлежащих `192.168.1.77` из `~/.ssh/known_hosts`). Затем вы сможете установить соединение с новым хостом.

## Вход в систему по ssh ключу

Вы можете настроить свой SSH-клиент так, чтобы при входе в систему не вводить пароль, а использовать вместо него открытые ключи. Это предпочтительный способ подключения к удалённому серверу по SSH, так как он гораздо безопаснее. Первое, что вам нужно сделать, — это создать пару ключей на клиентском компьютере. Для этого вы будете использовать `ssh-keygen` с опцией `-t` для указания нужного типа шифрования (_алгоритм цифровой подписи эллиптической кривой_ в нашем случае). Затем вам будет предложено указать путь для сохранения пары ключей (`~/.ssh/` — это удобно, так как это расположение по умолчанию) и кодовую фразу. Кодовая фраза не является обязательной, но настоятельно рекомендуется использовать её.

```bash
carol@debian:~/.ssh$ ssh-keygen -t ecdsa
Generating public/private ecdsa key pair.
Enter file in which to save the key (/home/carol/.ssh/id_ecdsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/carol/.ssh/id_ecdsa.
Your public key has been saved in /home/carol/.ssh/id_ecdsa.pub.
The key fingerprint is:
SHA256:tlamD0SaTquPZYdNepwj8XN4xvqmHCbe8g5FKKUfMo8 carol@debian
The key's randomart image is:
+---[ECDSA 256]---+
|      .          |
|     o .         |
|    = o o        |
|     B *         |
|    E B S o      |
|     o & O       |
|      @ ^ =      |
|     *.@ @.      |
|    o.o+B+o      |
+----[SHA256]-----+
```

{% hint style="info" %}
При создании пары ключей вы можете передать `ssh-keygen` параметр `-b` для указания размера ключа в битах (например, `ssh-keygen -t ecdsa -b 521`).
{% endhint %}

Предыдущая команда создала еще два файла в вашем `~/.ssh` каталоге:

```bash
carol@debian:~/.ssh$ ls
id_ecdsa  id_ecdsa.pub  known_hosts
```

`id_ecdsa`

Это ваш закрытый ключ.

`id_ecdsa.pub`

Это ваш открытый ключ.

\
Следующее, что вам нужно сделать, — это добавить свой открытый ключ в файл `~/.ssh/authorized_keys` пользователя, от имени которого вы хотите войти на удалённый хост (если каталог `~/.ssh` ещё не существует, вам нужно будет создать его). Вы можете скопировать свой открытый ключ на удалённый сервер несколькими способами: с помощью USB-накопителя, с помощью команды `scp` — которая перенесёт файл с помощью SSH — или _выведя_ содержимое вашего открытого ключа и передав его в `ssh` следующим образом:

```bash
carol@debian:~/.ssh$ cat id_ecdsa.pub |ssh ina@192.168.1.77 'cat >> .ssh/authorized_keys'
Password:
```

