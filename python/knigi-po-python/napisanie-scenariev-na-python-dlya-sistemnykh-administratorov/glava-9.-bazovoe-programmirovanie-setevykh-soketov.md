# Глава 9. Базовое программирование сетевых сокетов

В этой главе вы узнаете о сокетах и трех интернет-протоколах: http, ftplib и urllib. Вы также узнаете о модуле socket в Python, который используется для работы в сети. http - это пакет, который используется для работы с протоколом передачи гипертекста (HTTP). Модуль ftplib используется для выполнения автоматизированной работы, связанной с FTP. urllib - это пакет, который обрабатывает работу, связанную с URL.

В этой главе вы узнаете о следующем:

* Сокеты
* &#x20;Пакет http&#x20;
* Модуль ftplib&#x20;
* Пакет urllib

## Сокеты

В этом разделе мы познакомимся с сокетами. Мы будем использовать модуль сокетов Python. Сокеты - это конечные точки для обмена данными между компьютерами, будь то локально или через Интернет. Модуль сокетов имеет класс socket, который используется для обработки канала передачи данных. В нем также есть функции для сетевых задач. Чтобы использовать функциональность модуля socket, нам сначала нужно импортировать модуль socket.

Давайте посмотрим, как создать сокет. Класс socket имеет функцию socket с двумя аргументами: address\_family и socket type.

Ниже приведен синтаксис:

```python
import socket
s = socket.socket(address_family, socket type)
```

Параметр address\_family управляет протоколом сетевого уровня OSI.

Тип сокета управляет протоколом транспортного уровня.

Python поддерживает три семейства адресов: AF\_INET, AF\_INET6 и AF\_UNIX. Наиболее часто используется AF\_INET, который используется для интернет-адресации. AF\_INET6 используется для интернет-адресации IPv6. AF\_UNIX используется для доменных сокетов Unix (UDS), которые представляют собой протокол межпроцессного взаимодействия.

Существует два типа сокетов: SOCK\_DGRAM и SOCK\_STREAM. Тип сокета SOCK\_DGRAM используется для передачи дейтаграмм, ориентированных на сообщения; они связаны с протоколом UDP. Сокеты с дейтаграммами доставляют отдельные сообщения. SOCK\_STREAM используется для потоковой передачи данных; они связаны с TCP. Потоковые сокеты обеспечивают передачу байтовых потоков между клиентом и сервером.

Сокеты могут быть сконфигурированы как серверные, так и клиентские. Когда оба сокета TCP/IP подключены, связь будет двусторонней. Теперь мы рассмотрим пример взаимодействия клиент-сервер. Мы создадим два сценария: server.py и client.py.

Сценарий server.py выглядит следующим образом:

```bash
import socket

host_name = socket.gethostname()
port = 5000
s_socket = socket.socket()
s_socket.bind((host_name, port))
s_socket.listen(2)

conn, address = s_socket.accept()
print("Connection from: " + str(address))

while True:
            recv_data = conn.recv(1024).decode()
            if not recv_data:
                        break
            print("from connected user: " + str(recv_data))
            recv_data = input(' -> ')
            conn.send(recv_data.encode())

conn.close()
```

Теперь мы напишем скрипт для клиента.

client.py Скрипт выглядит следующим образом:

```bash
import socket

host_name = socket.gethostname()
port = 5000

c_socket = socket.socket()
c_socket.connect((host_name, port))
msg = input(" -> ")

while msg.lower().strip() != 'bye':
            c_socket.send(msg.encode())
            recv_data = c_socket.recv(1024).decode()
            print('Received from server: ' + recv_data)
            msg = input(" -> ")

c_socket.close()
```

Теперь мы напишем скрипт для клиента.

client.py Скрипт выглядит следующим образом:

| **Terminal 1:** python3 server.py                                                                                                                                                                                                  | **Terminal 2:** python3 client.py                                                                                                                                                        |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <pre class="language-bash"><code class="lang-bash"><strong>student@ubuntu:~/work$ python3 server.py
</strong>
Connection from: ('127.0.0.1', 35120)

from connected user: Hello from client

 -> Hello from server !
</code></pre> | <p><code>student@ubuntu:~/work$ python3 client.py</code></p><p><code>-> Hello from client</code></p><p><code>Received from server: Hello from server !</code></p><p> <code>-></code></p> |



```bash
student@ubuntu:~/work$ python3 server.py

Connection from: ('127.0.0.1', 35120)

from connected user: Hello from client

 -> Hello from server !
```

```bash
student@ubuntu:~/work$ python3 client.py

-> Hello from client

Received from server: Hello from server !


```



\
