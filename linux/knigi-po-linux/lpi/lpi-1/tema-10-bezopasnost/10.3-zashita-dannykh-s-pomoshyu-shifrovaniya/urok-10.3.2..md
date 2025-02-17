# Урок 10.3.2.

## Введение <a href="#sec.110.3_02-in" id="sec.110.3_02-in"></a>

В предыдущем уроке мы узнали, как использовать _OpenSSH_ для шифрования сеансов удалённого входа в систему, а также для любого другого последующего обмена информацией. Могут быть и другие сценарии, в которых вам может понадобиться шифровать файлы или электронную почту, чтобы они безопасно доставлялись получателю и были защищены от посторонних глаз. Вам также может понадобиться цифровая подпись для этих файлов или сообщений, чтобы предотвратить их подделку.

Отличным инструментом для таких целей является _GNU Privacy Guard_ (также известный как _GnuPG_ или просто _GPG_), который представляет собой бесплатную реализацию с открытым исходным кодом проприетарного _Pretty Good Privacy_ (_PGP_). _GPG_ использует стандарт _OpenPGP_, определённый _рабочей группой OpenPGP_ _рабочей группы по интернет-инженерии_ (_IETF_) в RFC 4880. В этом уроке мы рассмотрим основы _GNU Privacy Guard_.

## Базовую настройка, использование и отзыв GnuPG <a href="#perform_basic_gnupg_configuration_usage_and_revocation" id="perform_basic_gnupg_configuration_usage_and_revocation"></a>

Как и в случае с SSH, в основе GPG лежит _асимметричная криптография_ или _криптография с открытым ключом_. Пользователь генерирует пару ключей, состоящую из _закрытого ключа_ и _открытого ключа_. Ключи математически связаны таким образом, что зашифрованное одним ключом может быть расшифровано только другим. Чтобы обмен данными прошёл успешно, пользователь должен отправить свой открытый ключ получателю.

## **Настройка и использование GnuPG**

Команда для работы с GPG — `gpg`. Вы можете передать ей несколько параметров для выполнения различных задач. Давайте начнём с создания пары ключей для пользователя `carol`. Для этого вы воспользуетесь командой `gpg --gen-key`.

```bash
carol@debian:~$ gpg --gen-key
gpg (GnuPG) 2.2.12; Copyright (C) 2018 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

gpg: directory '/home/carol/.gnupg' created
gpg: keybox '/home/carol/.gnupg/pubring.kbx' created
Note: Use "gpg --full-generate-key" for a full featured key generation dialog.

GnuPG needs to construct a user ID to identify your key.

Real name:

(...)
```

После того как вы получите уведомление о том, что, помимо прочего, каталог конфигурации `~/.gnupg` и ваша связка открытых ключей `~/.gnugpg/pubring.kbx` были созданы, `gpg` попросит вас указать ваше настоящее имя и адрес электронной почты:

```bash
(...)
Real name: carol
Email address: carol@debian
You selected this USER-ID:
    "carol <carol@debian>"

Change (N)ame, (E)mail, or (O)kay/(Q)uit?
```

Если вас устраивает полученный результат `USER-ID` и вы нажмете <kbd>O</kbd>, вам будет предложено ввести парольную фразу (рекомендуется, чтобы она была достаточно сложной):

```bash
┌──────────────────────────────────────────────────────┐
│ Please enter the passphrase to                       │
│ protect your new key                                 │
│                                                      │
│ Passphrase:  │

(...)
```

Будут показаны несколько заключительных сообщений о создании других файлов, а также самих ключей, после чего процесс создания ключей завершится:

```bash
(...)
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: /home/carol/.gnupg/trustdb.gpg: trustdb created
gpg: key 19BBEFD16813034E marked as ultimately trusted
gpg: directory '/home/carol/.gnupg/openpgp-revocs.d' created
gpg: revocation certificate stored as '/home/carol/.gnupg/openpgp-revocs.d/D18FA0021F644CDAF57FD0F919BBEFD16813034E.rev'
public and secret key created and signed.

pub   rsa3072 2020-07-03 [SC] [expires: 2022-07-03]
      D18FA0021F644CDAF57FD0F919BBEFD16813034E
uid                      carol <carol@debian>
sub   rsa3072 2020-07-03 [E] [expires: 2022-07-03]
```

Теперь вы можете увидеть, что находится внутри каталога `~/.gnupg` (каталог конфигурации GPG):

```bash
carol@debian:~/.gnupg$ ls -l
total 16
drwx------ 2 carol carol 4096 Jul  3 23:34 openpgp-revocs.d
drwx------ 2 carol carol 4096 Jul  3 23:34 private-keys-v1.d
-rw-r--r-- 1 carol carol 1962 Jul  3 23:34 pubring.kbx
-rw------- 1 carol carol 1240 Jul  3 23:34 trustdb.gpg
```

Давайте объясним использование каждого файла:

`opengp-revocs.d`

Здесь хранится сертификат отзыва, который был создан вместе с парой ключей. Права доступа к этому каталогу довольно строгие, так как любой, у кого есть доступ к сертификату, может отозвать ключ (подробнее об отзыве ключей в следующем подразделе).

`private-keys-v1.d`

Это каталог, в котором хранятся ваши закрытые ключи, поэтому разрешения являются ограничительными.

`pubring.kbx`

Это ваша связка открытых ключей. В ней хранятся ваши собственные, а также любые другие импортированные открытые ключи.

`trustdb.gpg`

База данных доверия. Это связано с концепцией _Web of Trust_ (которая выходит за рамки этого урока).

{% hint style="info" %}
С выходом _GnuPG 2.1_ произошли некоторые существенные изменения, такие как исчезновение файлов `secring.gpg` и `pubring.gpg` в пользу `private-keys-v1.d` и `pubring.kbx` соответственно.
{% endhint %}

После создания пары ключей вы можете просмотреть свои открытые ключи с помощью `gpg --list-keys` — это отобразит содержимое вашей связки открытых ключей:

```bash
carol@debian:~/.gnupg$ gpg --list-keys
/home/carol/.gnupg/pubring.kbx
------------------------------
pub   rsa3072 2020-07-03 [SC] [expires: 2022-07-03]
      D18FA0021F644CDAF57FD0F919BBEFD16813034E
uid           [ultimate] carol <carol@debian>
sub   rsa3072 2020-07-03 [E] [expires: 2022-07-03]
```

Шестнадцатеричная строка `D18FA0021F644CDAF57FD0F919BBEFD16813034E` - это отпечаток вашего _открытого ключа_.

{% hint style="info" %}
Помимо `USER-ID` (`carol` в примере), есть также `KEY-ID`. `KEY-ID` состоит из последних 8 шестнадцатеричных цифр вашего открытого ключа (`6813 034E`). Вы можете проверить отпечаток своего ключа с помощью команды `gpg --fingerprint`` `_`USER-ID`_.
{% endhint %}

## **Распространение и отзыв ключей**

Теперь, когда у вас есть открытый ключ, вам следует сохранить его (т. е. _экспортировать_) в файл, чтобы вы могли предоставить к нему доступ своим будущим получателям. Тогда они смогут использовать его для шифрования файлов или сообщений, предназначенных для вас (поскольку только у вас есть закрытый ключ, только вы сможете расшифровать и прочитать их). Точно так же ваши получатели смогут использовать его для расшифровки и проверки ваших зашифрованных или подписанных сообщений/файлов. Используйте команду `gpg --export` с последующим `USER-ID` и перенаправлением на имя выходного файла по вашему выбору:

{% hint style="info" %}
Передача параметра `-a` или `--armor` в `gpg --export` (например, `gpg --export --armor carol > carol.pub.key` ) приведёт к созданию зашифрованного вывода в формате ASCII (вместо двоичного формата OpenPGP по умолчанию), который можно безопасно отправлять по электронной почте
{% endhint %}

Как уже отмечалось, теперь вы должны отправить файл открытого ключа (`carol.pub.key`) получателю, с которым вы хотите обмениваться информацией. Например, давайте отправим файл открытого ключа `ina`   на  удалённый сервер `halof` с помощью `scp`:

```bash
carol@debian:~/.gnupg$ scp carol.pub.key ina@halof:/home/ina/
Enter passphrase for key '/home/carol/.ssh/id_ecdsa':
carol.pub.key                                                                                         100% 1740   775.8KB/s   00:00
carol@debian:~/.gnupg$
```

`carol.pub.key теперь`находится у ina. Она использует его, чтобы зашифровать файл и отправить его `carol` в следующем разделе.

{% hint style="info" %}
Другой способ распространения открытых ключей — использование _серверов ключей_: вы загружаете свой открытый ключ на сервер с помощью команды `gpg --keyserver`` `_`keyserver-name`_` ``--send-keys`` `_`KEY-ID`_, а другие пользователи получают (то есть _импортируют_) его с помощью `gpg --keyserver`` `_`keyserver-name`_` ``--recv-keys`` `_`KEY-ID`_.
{% endhint %}

Давайте завершим этот раздел обсуждением отзыва ключа. Отзыв ключа следует использовать, когда ваши закрытые ключи были скомпрометированы или удалены. Первым шагом является создание сертификата отзыва, передав `gpg` опцию `--gen-revoke`, за которой следует `USER-ID`. Вы можете использовать `--gen-revoke`  с параметром`--output`, за которым следует спецификация имени файла назначения, чтобы сохранить полученный сертификат в файл (вместо того, чтобы печатать его на экране терминала). Выходные сообщения на протяжении всего процесса отзыва говорят сами за себя:

<pre class="language-bash"><code class="lang-bash"><strong>sonya@debian:~/.gnupg$ gpg --output revocation_file.asc --gen-revoke sonya
</strong>
sec  rsa3072/0989EB7E7F9F2066 2020-07-03 sonya &#x3C;sonya@debian>

Create a revocation certificate for this key? (y/N) y
Please select the reason for the revocation:
  0 = No reason specified
  1 = Key has been compromised
  2 = Key is superseded
  3 = Key is no longer used
  Q = Cancel
(Probably you want to select 1 here)
Your decision? 1
Enter an optional description; end it with an empty line:
> My laptop was stolen.
>
Reason for revocation: Key has been compromised
My laptop was stolen.
Is this okay? (y/N) y
ASCII armored output forced.
Revocation certificate created.

Please move it to a medium which you can hide away; if Mallory gets
access to this certificate he can use it to make your key unusable.
It is smart to print this certificate and store it away, just in case
your media become unreadable.  But have some caution:  The print system of
your machine might store the data and make it available to others!
</code></pre>

Сертификат отзыва был сохранён в файл `revocation_file.asc` (`asc` для формата ASCII):

```bash
sonya@debian:~/.gnupg$ ls
openpgp-revocs.d  private-keys-v1.d  pubring.kbx  revocation_file.asc  trustdb.gpg
sonya@debian:~/.gnupg$ cat revocation_file.asc
-----BEGIN PGP PUBLIC KEY BLOCK-----
Comment: This is a revocation certificate

iQHDBCABCgAtFiEEiIVjfDnnpieFi0wvnlcN6yLCeHEFAl8ASx4PHQJzdG9sZW4g
bGFwdG9wAAoJEJ5XDesiwnhxT9YMAKkjQiMpo9Uyiy9hyvukPPSrLcmtAGLk4pKS
pLZfzA5kxa+HPQwBglAEvfNRR6VMxqXUgUGYC/IAyQQM62oNAcY2PCPrxyJNgVF7
8l4mMZKvW++5ikjZwyg6WWV0+w6oroeo9qruJFjcu752p4T+9gsHVa2r+KRqcPQe
aZ65sAvsBJlcsUDZqfWUXg2kQp9mNPCdQuqvDaKRgNCHA1zbzNFzXWVd2X5RgFo5
nY+tUP8ZQA9DTQPBLPcggICmfLopMPZYB2bft5geb2mMi2oNpf9CNPdQkdccimNV
aRjqdUP9C89PwTafBQkQiONlsR/dWTFcqprG5KOWQPA7xjeMV8wretdEgsyTxqHp
v1iRzwjshiJCKBXXvz7wSmQrJ4OfiMDHeS4ipR0AYdO8QCzmOzmcFQKikGSHGMy1
z/YRlttd6NZIKjf1TD0nTrFnRvPdsZOlKYSArbfqNrHRBQkgirOD4JPI1tYKTffq
iOeZFx25K+fj2+0AJjvrbe4HDo5m+Q==
=umI8
-----END PGP PUBLIC KEY BLOCK-----
```

Чтобы эффективно отозвать свой закрытый ключ, вам нужно объединить сертификат с ключом, импортировав файл отзыва сертификата в свою связку ключей:

```
sonya@debian:~/.gnupg$ gpg --import revocation_file.asc
gpg: key 9E570DEB22C27871: "sonya <sonya@debian>" revocation certificate imported
gpg: Total number processed: 1
gpg:    new key revocations: 1
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
gpg: next trustdb check due at 2022-07-04
```

Перечислите свои ключи прямо сейчас, и вы будете проинформированы об отзыве вашего ключа:

```bash
sonya@debian:~/.gnupg$ gpg --list-keys
/home/sonya/.gnupg/pubring.kbx
pub   rsa3072 2020-07-04 [SC] [revoked: 2020-07-04]
      8885637C39E7A627858B4C2F9E570DEB22C27871
uid           [ revoked] sonya <sonya@debian>
```

И последнее, но не менее важное: убедитесь, что отозванный ключ доступен любой стороне, у которой есть связанные с ним открытые ключи (включая серверы ключей).

## Использование GPG для шифрования, дешифрования, подписи и проверки файлов <a href="#use_gpg_to_encrypt_decrypt_sign_and_verify_files" id="use_gpg_to_encrypt_decrypt_sign_and_verify_files"></a>

В предыдущем разделе `carol` отправила свой открытый ключ `ina`. Теперь мы будем использовать его, чтобы обсудить, как GPG может шифровать, расшифровывать, подписывать и проверять файлы.

## **Шифрование и дешифрование файлов**

Сначала `ina` должна импортировать открытый ключ `carol` (`carol.pub.key`) в свою связку ключей, чтобы начать с ним работать:

```bash
ina@halof:~> gpg --import carol.pub.key
gpg: /home/ina/.gnupg/trustdb.gpg: trustdb created
gpg: key 19BBEFD16813034E: public key "carol <carol@debian>" imported
gpg: Total number processed: 1
gpg:               imported: 1
ina@halof:~> gpg --list-keys
/home/ina/.gnupg/pubring.kbx
----------------------------
pub   rsa3072 2020-07-03 [SC] [expires: 2022-07-03]
      D18FA0021F644CDAF57FD0F919BBEFD16813034E
uid           [ unknown] carol <carol@debian>
sub   rsa3072 2020-07-03 [E] [expires: 2022-07-03]
```

Далее вы создадите файл, вписав в него текст, а затем зашифруете его с помощью `gpg` (поскольку вы не подписали ключ `carol`', вас спросят, хотите ли вы использовать этот ключ):

<pre class="language-bash"><code class="lang-bash">ina@halof:~> echo "This is the message ..." > unencrypted-message
ina@halof:~> gpg --output encrypted-message --recipient carol --armor --encrypt unencrypted-message
gpg: 0227347CC92A5CB1: There is no assurance this key belongs to the named user
sub  rsa3072/0227347CC92A5CB1 2020-07-03 carol &#x3C;carol@debian>
 Primary key fingerprint: D18F A002 1F64 4CDA F57F  D0F9 19BB EFD1 6813 034E
      Subkey fingerprint: 9D89 1BF9 39A4 C130 E44B  1135 0227 347C C92A 5CB1

It is NOT certain that the key belongs to the person named
<strong>in the user ID.  If you really know what you are doing,
</strong>you may answer the next question with yes.

Use this key anyway? (y/N) y
</code></pre>

Давайте разберем `gpg` команду:

`--output encrypted-message`

Указание имени файла для зашифрованной версии исходного файла (`encrypted-message` в примере).

`--recipient carol`

Спецификация получателя `USER-ID` (`carol` в нашем примере). Если она не указана, GnuPG запросит её (если `--default-recipient` не указано).

`--armor`

Эта опция позволяет получить защищённый от ASCII вывод, который можно скопировать в электронное письмо.

`--encrypt unencrypted-message`

Спецификация имени файла исходного файла для шифрования.

Теперь вы можете отправить `encrypted-message` кому `carol` на `debian` с помощью `scp`:

```bash
ina@halof:~> scp encrypted-message carol@debian:/home/carol/
carol@debian's password:
encrypted-message                                                             100%  736     1.8MB/s   00:00
```

Если вы войдёте в систему под именем `carol` и попытаетесь прочитать `encrypted-message`, вы увидите, что оно действительно зашифровано и, следовательно, не читается:

```bash
carol@debian:~$ cat encrypted-message
-----BEGIN PGP MESSAGE-----

hQGMAwInNHzJKlyxAQv/brJ8Ubs/xya35sbv6kdRKm1C7ONLxL3OueWA4mCs0Y/P
GBna6ZEUCrMEgl/rCyByj3Yq74kuiTmzxAIRUDdvHfj0TtrOWjVAqIn/fPSfMkjk
dTxKo1i55tLJ+sj17dGMZDcNBinBTP4U1atuN71A5w7vH+XpcesRcFQLKiSOmYTt
F7SN3/5x5J6io4ISn+b0KbJgiJNNx+Ne/ub4Uzk4NlK7tmBklyC1VRualtxcG7R9
1klBPYSld6fTdDwT1Y4MofpyILAiGMZvUR1RXauEKf7OIzwC5gWU+UQPSgeCdKQu
X7QL0ZIBS0Ug2XKrO1k93lmDjf8PWsRIml6n/hNelaOBA3HMP0b6Ozv1gFeEsFvC
IxhUYPb+rfuNFTMEB7xIO94AAmWB9N4qknMxdDqNE8WhA728Plw6y8L2ngsplY15
MR4lIFDpljA/CcVh4BXVe9j0TdFWDUkrFMfaIfcPQwKLXEYJp19XYIaaEazkOs5D
W4pENN0YOcX0KWyAYX6r0l8BF0rq/HMenQwqAVXMG3s8ATuUOeqjBbR1x1qCvRQP
CR/3V73aQwc2j5ioQmhWYpqxiro0yKX2Ar/E6rZyJtJYrq+CUk8O3JoBaudknNFj
pwuRwF1amwnSZ/MZ/9kMKQ==
=g1jw
-----END PGP MESSAGE-----
```

Однако, поскольку у вас есть закрытый ключ, вы можете легко расшифровать сообщение, передав параметр `gpg` с опцией `--decrypt` и указав путь к зашифрованному файлу (потребуется парольная фраза закрытого ключа):

```bash
carol@debian:~$ gpg --decrypt encrypted-message
gpg: encrypted with 3072-bit RSA key, ID 0227347CC92A5CB1, created 2020-07-03
      "carol <carol@debian>"
This is the message ...
```

Вы также можете указать параметр `--output` для сохранения сообщения в новом незашифрованном файле:

```bash
carol@debian:~$ gpg --output unencrypted-message --decrypt encrypted-message
gpg: encrypted with 3072-bit RSA key, ID 0227347CC92A5CB1, created 2020-07-03
      "carol <carol@debian>"
carol@debian:~$ cat unencrypted-message
This is the message ...
```

## **Подписание и проверка Файлов**

Помимо шифрования, GPG можно использовать для подписи файлов. Здесь актуальна опция `--sign`. Давайте начнём с создания нового сообщения (`message`) и его подписи с помощью опции `--sign`. (потребуется парольная фраза вашего закрытого ключа):

```bash
carol@debian:~$ echo "This is the message to sign ..." > message
carol@debian:~$ gpg --output message.sig --sign message
(...)
```

Разбивка команды `gpg`:

`--output message`

Указание имени файла подписанной версии исходного файла (`message.sig` в нашем примере).

`--sign message`

Путь к исходному файлу.

{% hint style="info" %}
С помощью `--sign` документ сжимается, а затем подписывается. Результат выводится в двоичном формате.
{% endhint %}

Далее мы перенесём файл на`ina`` ``halof` с помощью `scp message.sig ina@halof:/home/ina` . Вернувшись на `ina`  `halof`, вы можете проверить его с помощью опции `--verify`:

```bash
ina@halof:~> gpg --verify message.sig
gpg: Signature made Sat 04 jul 2020 14:34:41 CEST
gpg:                using RSA key D18FA0021F644CDAF57FD0F919BBEFD16813034E
gpg: Good signature from "carol <carol@debian>" [unknown]
(...)
```

Если вы также хотите прочитать файл, вам нужно расшифровать его в новый файл (`message` в нашем случае) с помощью опции `--output`:

```bash
ina@halof:~> gpg --output message --decrypt message.sig
gpg: Signature made Sat 04 jul 2020 14:34:41 CEST
gpg:                using RSA key D18FA0021F644CDAF57FD0F919BBEFD16813034E
gpg: Good signature from "carol <carol@debian>" [unknown]
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: D18F A002 1F64 4CDA F57F  D0F9 19BB EFD1 6813 034E
ina@halof:~> cat message
This is the message to sign ...
```

## **GPG-Агент**

Мы завершим этот урок кратким описанием `gpg-agent`. `gpg-agent` — это демон, который управляет закрытыми ключами для GPG (он запускается по требованию `gpg`). Чтобы просмотреть список наиболее полезных опций, запустите `gpg-agent --help` или `gpg-agent -h`:

```bash
carol@debian:~$ gpg-agent --help
gpg-agent (GnuPG) 2.2.4
libgcrypt 1.8.1
Copyright (C) 2017 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <https://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Syntax: gpg-agent [options] [command [args]]
Secret key management for GnuPG

Options:

     --daemon                        run in daemon mode (background)
     --server                        run in server mode (foreground)
     --supervised                    run in supervised mode
 -v, --verbose                       verbose
 -q, --quiet                         be somewhat more quiet
 -s, --sh                            sh-style command output
 -c, --csh                           csh-style command output
(...)
```

{% hint style="info" %}
Для получения дополнительной информации обратитесь к справочной странице `gpg-agent`.
{% endhint %}

## Управляемые Упражнения <a href="#sec.110.3_02-ge" id="sec.110.3_02-ge"></a>

1.  Заполните таблицу, указав правильное имя файла:

    | Описание                        | Имя файла |
    | ------------------------------- | --------- |
    | База данных доверенных хостов   |           |
    | Каталог отозванных сертификатов |           |
    | Каталог закрытых ключей         |           |
    | Каталог открытых ключей         |           |
2. Ответьте на следующие вопросы:
   * Какой тип криптографии использует _GnuPG_?
   * Каковы два основных компонента криптографии с открытым ключом?
   * `Какой KEY-ID` отпечаток  открытого ключа `07A6 5898 2D3A F3DD 43E3 DA95 1F3F 3147 FA7F 54C7`?
   * Какой метод используется для распространения открытых ключей на глобальном уровне?
3. Выполните следующие шаги в правильном порядке, касающиеся отзыва закрытого ключа:
   * Сделайте отозванный ключ доступным для ваших корреспондентов.
   * Создайте сертификат отзыва.
   *   Импортируйте сертификат отзыва в ваш keyring.

       Правильный порядок таков:

       | **Шаг 1**: |   |
       | ---------- | - |
       | **Шаг 2**: |   |
       | **Шаг 3**: |   |
4. Что касается шифрования файлов, что означает `--armor` опция в команде `gpg --output encrypted-message --recipient carol --armor --encrypt unencrypted-message`?

## Исследовательские упражнения <a href="#sec.110.3_02-ee" id="sec.110.3_02-ee"></a>

1.  У большинства вариантов `gpg` есть как длинная, так и короткая версия. Дополните таблицу соответствующей короткой версией:

    | Длинная версия | Сокращенная версия |
    | -------------- | ------------------ |
    | `--armor`      |                    |
    | `--output`     |                    |
    | `--recipient`  |                    |
    | `--decrypt`    |                    |
    | `--encrypt`    |                    |
    | `--sign`       |                    |
2. Ответьте на следующие вопросы, касающиеся экспорта ключей:
   * Какую команду вы бы использовали для экспорта всех ваших открытых ключей в файл с именем `all.key`?
   * Какую команду вы бы использовали для экспорта всех ваших закрытых ключей в файл с именем `all_private.key`?
3. Какой вариант `gpg` позволяет выполнять большинство ключевых задач, связанных с управлением, предоставляя вам меню?
4. Какая `gpg` опция позволяет вам создать подпись открытым текстом?

## Краткие сведения <a href="#sec.110.3_02-su" id="sec.110.3_02-su"></a>

В этом уроке мы рассмотрели _GNU Privacy Guard_ — отличный инструмент для шифрования/дешифрования и цифровой подписи/проверки файлов. Вы узнали:

* как сгенерировать пару ключей.
* как составить список ключей в вашей связке для ключей.
* содержимое `~/.gnupg` каталога.
* что такое `USER-ID` и `KEY-ID`.
* как распространять открытые ключи среди ваших корреспондентов.
* как глобально распространять открытые ключи через серверы ключей.
* как отозвать приватные ключи.
* как шифровать и расшифровывать файлы.
* как подписывать и проверять файлы.
* основы работы _GPG-Агента_.

В этом уроке обсуждались следующие команды:

`gpg`

_OpenPGP_ инструмент шифрования и подписи.

## Ответы на Упражнения с Руководством <a href="#sec.110.3_02-age" id="sec.110.3_02-age"></a>

1.  Заполните таблицу, указав правильное имя файла:

    | Описание                        | Имя файла           |
    | ------------------------------- | ------------------- |
    |  База данных доверенных хостов  | `trustdb.gpg`       |
    | Каталог отозванных сертификатов | `opengp-revocs.d`   |
    | Каталог закрытых ключей         | `private-keys-v1.d` |
    | Список открытых ключей          | `pubring.kbx`       |
2. Ответьте на следующие вопросы:
   *   Какой тип криптографии использует _GnuPG_?

       Криптография с открытым ключом или асимметричная криптография.
   *   Каковы два основных компонента криптографии с открытым ключом?

       Открытый и закрытый ключи.
   * `Какой KEY-ID` отпечаток  открытого ключа `07A6 5898 2D3A F3DD 43E3 DA95 1F3F 3147 FA7F 54C7`?   FA7F 54C7
   *   Какой метод используется для распространения открытых ключей на глобальном уровне?

       Серверы ключей.
3. Выполните следующие шаги в правильном порядке, касающиеся отзыва закрытого ключа:
   * Сделайте отозванный ключ доступным для ваших корреспондентов
   * Создание сертификата отзыва
   *   Импортируйте сертификат отзыва в ваш keyring

       Правильный порядок таков:

       | **Шаг 1**: | Создание сертификата отзыва                                  |
       | ---------- | ------------------------------------------------------------ |
       | **Шаг 2**: | Импортируйте сертификат отзыва в ваш keyring                 |
       | **Шаг 3**: | Сделайте отозванный ключ доступным для ваших корреспондентов |
4.  Что касается шифрования файлов, что означает `--armor` опция в команде `gpg --output encrypted-message --recipient carol --armor --encrypt unencrypted-message`?

    Он выводит защищённый от взлома ASCII-текст, что позволяет скопировать полученный зашифрованный файл в электронное письмо.

## Ответы на Исследовательские упражнения <a href="#sec.110.3_02-aee" id="sec.110.3_02-aee"></a>

1.  У большинства вариантов `gpg` есть как длинная, так и короткая версия. Дополните таблицу соответствующей короткой версией:

    | Длинная версия | Сокращенная версия |
    | -------------- | ------------------ |
    | `--armor`      | `-a`               |
    | `--output`     | `-o`               |
    | `--recipient`  | `-r`               |
    | `--decrypt`    | `-d`               |
    | `--encrypt`    | `-e`               |
    | `--sign`       | `-s`               |
2. Ответьте на следующие вопросы, касающиеся экспорта ключей:
   *   Какую команду вы бы использовали для экспорта всех ваших открытых ключей в файл с именем `all.key`?

       `gpg --export --output all.key` или `gpg --export -o all.key`
   *   Какую команду вы бы использовали для экспорта всех ваших закрытых ключей в файл с именем `all_private.key`?

       `gpg --export-secret-keys --output all_private.key` или `gpg --export-secret-keys -o all_private.key` (`--export-secret-keys` можно заменить на `--export-secret-subkeys` с немного другим результатом — см. `man pgp` для получения дополнительной информации).
3.  Какой вариант `gpg` позволяет выполнять большинство ключевых задач, связанных с управлением, предоставляя вам меню?

    `--edit-key`
4.  Какая `gpg` опция позволяет вам создать подпись открытым текстом?

    `--clearsign`
