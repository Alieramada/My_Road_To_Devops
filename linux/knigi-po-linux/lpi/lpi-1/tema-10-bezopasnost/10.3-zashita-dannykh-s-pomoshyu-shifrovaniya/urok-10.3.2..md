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

