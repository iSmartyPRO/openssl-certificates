# Самоподписанные сертификаты созданные при помощи OpenSSL

> [Источник из которого создавался этот репозиторий](https://openssl-ca.readthedocs.io/en/latest/ "Ссылка на внешний ресурс")

## Создание корневой ключевой пары Certificate Authority

| Имя издателя|Файл ключ|Файл сертификата|
|-|-|-|
| Global iSmartyPRO Certificate Authority   | [./ca/private/ca.key.pem](./ca/private/ca.key.pem) | [./ca/certs/ca.cert.pem](./ca/certs/ca.cert.pem) |

### Подготовим корневой каталог
Создадим каталог ( ./ca) для хранения ключей и сертификатов.

```sh
mkdir ./ca
```

Создадим структуру каталогов. Файлы index.txt and serial будут использоваться как база данных неструктурированных файлов для отслеживания подписанных сертификатов.

```sh
mkdir ./ca/certs ./ca/crl ./ca/newcerts ./ca/private
chmod 700 ./ca/private
touch ./ca/index.txt
echo 1000 > ./ca/serial
```

### Подготовим файл конфигурации

Файл конфигурации [./ca/openssl.cnf](./ca/openssl.cnf)

### Создадим корневой ключ

```sh
openssl genrsa -aes256 -out ./ca/private/ca.key.pem 4096
```

```sh
chmod 400 ./ca/private/ca.key.pem
```

### Создадим корневой сертификат
Используя корневой ключ (ca.key.pem) для создания корневого сертификата (ca.cert.pem). Дайте корневому сертификату длительный срок действия, например тридцать лет. По истечении срока действия корневого сертификата все сертификаты, подписанные центром сертификации, становятся недействительными.

```sh
openssl req -config ./ca/openssl.cnf -new -x509 -days 10950 -sha256 -extensions v3_ca -key ./ca/private/ca.key.pem -out ./ca/certs/ca.cert.pem
```

```sh
chmod 444 ./ca/certs/ca.cert.pem
```

### Проверка корневого сертификата

```sh
openssl x509 -noout -text -in ./ca/certs/ca.cert.pem
```


## Создание промежуточных ключевых пар

Промежуточный центр сертификации (CA) — это объект, который может подписывать сертификаты от имени корневого центра сертификации. Корневой центр сертификации подписывает промежуточный сертификат, образуя цепочку доверия.

Целью использования промежуточного центра сертификации является, прежде всего, обеспечение безопасности. Корневой ключ можно хранить в автономном режиме и использовать как можно реже. Если промежуточный ключ скомпрометирован, корневой центр сертификации может отозвать промежуточный сертификат и создать новую промежуточную криптографическую пару.

### Подготовим папку для промежуточных сертификатов и ключей

Для хранения промежуточных сертификатов и ключей будем использовать отдельный каталог ./intermediate

```sh
mkdir ./intermediate
```

#### Сформируем структуру папок и файлов

```sh
mkdir ./intermediate/certs ./intermediate/crl ./intermediate/csr ./intermediate/newcerts ./intermediate/private
chmod 700 ./intermediate/private
touch ./intermediate/index.txt
echo 1000 > ./intermediate/serial
```

Добавим crlnumber файл в дерево каталогов промежуточного центра сертификации. crlnumber используется для отслеживания списков отзыва сертификатов.

```
echo 1000 > ./intermediate/crlnumber
```

Подготовим файл конфигурации OpenSSL для каждого промежуточного центра сертификации. Все конфигурационные файлы будут зраниться в отдельной папке [./intermediate/openssl-cnf/](./intermediate/openssl-cnf/)

Каждому созданному ключу прописываем доступы:
```sh
chmod 400 ./intermediate/private/intermediate.key.pem
```

### Создадим промежуточные сертификаты
Используя промежуточные ключи для создания запроса на подпись сертификата (CSR). Сведения обычно должны соответствовать корневому ЦС. Однако общее имя должно быть другим.

```sh
openssl req -config ./intermediate/openssl-cnf/openssl.cnf -new -sha256 \
    -key ./intermediate/private/intermediate.key.pem \
    -out ./intermediate/csr/intermediate.csr.pem
```

**Logs:**
```sh
openssl req -config ./intermediate/openssl-cnf/iSmartyPro-Dev.cnf -new -sha256 \
    -key ./intermediate/private/iSmartyPro-Dev.key.pem \
    -out ./intermediate/csr/iSmartyPro-Dev.csr.pem
```

Чтобы создать промежуточный сертификат, используем корневой центр сертификации с расширением v3_intermediate_ca для подписи промежуточного CSR. Промежуточный сертификат должен быть действителен в течение более короткого периода, чем корневой сертификат. Десять лет было бы разумно.

```sh
openssl ca -config ./ca/openssl.cnf -extensions v3_intermediate_ca \
    -days 3650 -notext -md sha256 -in ./intermediate/csr/intermediate.csr.pem \
    -out ./intermediate/certs/intermediate.cert.pem
```
```sh
chmod 444 intermediate/certs/intermediate.cert.pem
```

В этом файле index.txt OpenSSL caхранит базу данных сертификатов. Не удаляйте и не редактируйте этот файл вручную. Теперь он должен содержать строку, которая относится к промежуточному сертификату.

**Проверика промежуточного сертификата**
```sh
openssl x509 -noout -text -in ./intermediate/certs/intermediate.cert.pem
```
Проверьте промежуточный сертификат на соответствие корневому сертификату. Значок OKуказывает на то, что цепочка доверия не повреждена.
```sh
openssl verify -CAfile ./ca/certs/ca.cert.pem ./intermediate/certs/intermediate.cert.pem
```


**Logs:**
```sh
openssl ca -config ./ca/openssl.cnf -extensions v3_intermediate_ca \
    -days 3650 -notext -md sha256 -in ./intermediate/csr/iSmartyPro-Dev.csr.pem \
    -out ./intermediate/certs/iSmartyPro-Dev.cert.pem
chmod 444 intermediate/certs/iSmartyPro-Dev.cert.pem
openssl x509 -noout -text -in ./intermediate/certs/iSmartyPro-Dev.cert.pem
openssl verify -CAfile ./ca/certs/ca.cert.pem ./intermediate/certs/iSmartyPro-Dev.cert.pem
```

### Создадим файл цепочки сертификатов.
Когда приложение (например, веб-браузер) пытается проверить сертификат, подписанный промежуточным центром сертификации, оно также должно сверить промежуточный сертификат с корневым сертификатом. Чтобы завершить цепочку доверия, создайте цепочку сертификатов ЦС для представления приложению.

Чтобы создать цепочку сертификатов ЦС, объедините промежуточные и корневые сертификаты. Мы будем использовать этот файл позже для проверки сертификатов, подписанных промежуточным центром сертификации.

```sh
cat ./intermediate/certs/intermediate.cert.pem ./ca/certs/ca.cert.pem > \
    ./intermediate/certs/ca-chain.cert.pem
chmod 444 ./intermediate/certs/ca-chain.cert.pem
```
Наш файл цепочки сертификатов должен включать корневой сертификат, поскольку ни одно клиентское приложение еще не знает о нем. Лучшим вариантом, особенно если вы администрируете интрасеть, является установка корневого сертификата на каждом клиенте, которому необходимо подключиться. В этом случае файл цепочки должен содержать только ваш промежуточный сертификат.

**Logs:**
```sh
cat ./intermediate/certs/iSmartyPro-Dev.cert.pem ./ca/certs/ca.cert.pem > \
    ./intermediate/certs/iSmartyPro-Dev-ca-chain.cert.pem
chmod 444 ./intermediate/certs/iSmartyPro-Dev-ca-chain.cert.pem
```

### Список промежуточных центров сертификации

| Имя издателя|OpenSSL File|Файл ключ|Файл сертификат|Примечание|
|-|-|-|-|-|
| iSmartyPRO Development CA|[iSmartyPro-Dev.cnf](./intermediate/openssl-cnf/iSmartyPro-Dev.cnf)|[iSmartyPro-Dev.key.pem](./intermediate/private/iSmartyPro-Dev.key.pem)| |Development|
| iSmartyPRO Online Resources CA| | | |Web, apps etc.|
| GES Construction CA| | | |Web, apps etc.|
| AmCham Kyrgyzstan CA| | | |Web, apps etc.|
| BGLOBAL CA| | | |Web, apps etc.|
| Anchor Safety Academy CA| | | |Web, apps etc.|
| Bishkeksuukanal CA| | | |Web, apps etc.|
| GENCO Industry CA| | | |Web, apps etc.|
| FlexGroup CA| | | |Web, apps etc.|
| SRG Trade CA| | | |Web, apps etc.|
| TradeMe CA| | | |Web, apps etc.|
| Novus CA| | | |Web, apps etc.|

## Подписывание серверных и клиентских серктификатов

Мы будем подписывать сертификаты, используя наш промежуточный центр сертификации. Вы можете использовать эти подписанные сертификаты в различных ситуациях, например для защиты подключений к веб-серверу или для аутентификации клиентов, подключающихся к службе.

### Создание ключа

Наши корневые и промежуточные пары имеют длину 4096 бит. Срок действия сертификатов сервера и клиента обычно истекает через год, поэтому вместо них мы можем безопасно использовать 2048 бит.

> Хотя 4096-битный формат немного более безопасен, чем 2048-битный, он замедляет рукопожатия TLS и значительно увеличивает нагрузку на процессор во время рукопожатий. По этой причине большинство веб-сайтов используют 2048-битные пары.

Если вы создаете криптографическую пару для использования с веб-сервером (например, Apache), вам нужно будет вводить этот пароль каждый раз при перезапуске веб-сервера. Возможно, вы захотите опустить -aes256опцию создания ключа без пароля.

```sh
mkdir ./domains
```

#### Сформируем структуру папок и файлов

```sh
mkdir ./domains/certs ./domains/crl ./domains/csr ./domains/newcerts ./domains/private
chmod 700 ./domains/private
touch ./domains/index.txt
echo 1000 > ./domains/serial
```

Добавим crlnumber файл в дерево каталогов промежуточного центра сертификации. crlnumber используется для отслеживания списков отзыва сертификатов.

```sh
echo 1000 > ./domains/crlnumber
```

```sh
openssl genrsa -aes256 -out ./domains/private/dev.ismarty.pro.key.pem 2048
chmod 400 ./domains/private/dev.ismarty.pro.key.pem
```
## Создание сертификата

Используя закрытый ключ для создания запроса для подпись сертификата (CSR). Детали CSR не обязательно должны совпадать с промежуточным центром сертификации. Для сертификатов сервера общее имя должно быть полным доменным именем (например, example.com), тогда как для сертификатов клиента это может быть любой уникальный идентификатор (например, адрес электронной почты). Обратите внимание, что общее имя не может совпадать с корневым или промежуточным сертификатом.

Для сертификатов сервера может быть полезно (и необходимо для последних браузеров) предоставить расширение subjectAltName (SAN), чтобы сертификат был действителен (помимо общего имени ) и для других имен хостов. Используйте -addextпереключатель, чтобы предоставить их (требуется openssl 1.1.1), как показано в примере ниже. Для более старых версий openssl SAN можно указать, создав новый раздел (например alt_names) в ./intermediate/openssl.cnf файле и ссылаясь на него в server_cert разделе.

```sh
openssl req -config ./domains/openssl-cnf/dev.ismarty.pro.cnf \
    -key ./domains/private/dev.ismarty.pro.key.pem \
    -new -sha256 -out ./domains/csr/dev.ismarty.pro.csr.pem \
    -addext "subjectAltName = DNS:dev.ismarty.pro"
```

Чтобы создать сертификат, используем промежуточный центр сертификации для подписи CSR. Если сертификат будет использоваться на сервере, используйте расширение server_cert. Если сертификат будет использоваться для аутентификации пользователя, используйте расширение usr_cert. Срок действия сертификатов обычно составляет один год, хотя для удобства центр сертификации обычно предоставляет еще несколько дней.

примерно на 5 лет (2000 дней с запасом)
```sh
openssl ca -config ./domains/openssl-cnf/dev.ismarty.pro.cnf -extensions server_cert \
    -days 2000 -notext -md sha256 -in ./domains/csr/dev.ismarty.pro.csr.pem \
    -out ./domains/certs/dev.ismarty.pro.cert.pem
chmod 444 ./domains/certs/dev.ismarty.pro.cert.pem
```

### Проверка сертификата
```sh
openssl x509 -noout -text -in ./domains/certs/dev.ismarty.pro.cert.pem
```

```sh
openssl verify -CAfile ./intermediate/certs/ca-chain.cert.pem \
    intermediate/certs/example.com.cert.pem

```

| Имя издателя|Name/FQDN| Примечание
|-|-|-|
| iSmartyPRO Development CA | ps.ismarty.pro |Скрипты PowerShell +подписывание кода|
| iSmartyPRO Online Resources CA | www.ismarty.pro, ismarty.pro|Базовые домены|
| iSmartyPRO Online Resources CA | pass.ismarty.pro|Менеджер паролей|
| iSmartyPRO Online Resources CA | crm.ismarty.pro|CRM система и база знаний|
| iSmartyPRO Online Resources CA | zabbix.ismarty.pro|Система мониторинга Zabbix|
| iSmartyPRO Online Resources CA | cloud.ismarty.pro|Файловый облачный сервис|

