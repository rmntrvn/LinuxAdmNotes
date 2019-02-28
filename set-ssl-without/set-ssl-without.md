*В данной статье рассмотрены методы установки SSL-сертификата на сервер без панели управления сервером (ISPmanager и др.). В статье рассматривается установка сертификата на web-сервер Nginx и Apache.*

Чтобы узнать какой web-сервер установлен на хостинге можно использовать команду с удаленной машины:
```
curl --insecure --silent --show-error --connect-timeout 1 -I domain.ru | grep Server
```
Результатом команды будет вывод `Server: nginx`. Однако, не всегда данный способ поможет точно определить какой web-сервер используется. Если на сервере используются два web-сервера (Nginx - proxy-web-сервер, Apache-backend-web-сервер), то данный способ покажет только Nginx (хотя в данном случае необходимо устанавливать именно на Nginx - подробнее далее). Также, подключившись к серверу можно выполнить команду:
```
netstat -tnlp
```
Данная команда покажет прослушиваемые порты сервера. Однако, данная утилита установлена не на всех серверах. Также, возможно, на сервере нестандартно настроен сетевой экран. В этом случае прослушиваемые порты можно посмотреть командой:
```
lsof -i -P
```
Можно выполнить команду:
```
ls -la /var/log
```
Данная команда покажет все лог-файлы на сервере. Среди них можно найти лог-файлы web-сервера/web-серверов.
Существует множество способов определить, какой web-сервер используется.

Для установки SSL-сертификата на сервер с операционной системой Linux без панели управления сервером ISPmanager потребуется:
1. Данные SSL-сертификата:

| №     |         Наименование          | Имя поля в Sd услуги сертификата  |
|:-:    |:----------------------------: |:--------------------------------: |
| 1     | SSL-сертификат формата x509   |       certificate_x509cert        |
| 2     | Промежуточный SSL-сертификат  |       certificate_ca_inter        |
| 3     | Корневой сертификат           |        certificate_ca_root        |
| 4     | Приватный ключ                |            private_key            |

>? Данные сертификатов также отправляются на контактный e-mail после выпуска сертификата.

2. Доступ на сервер для пользователя root.

## Установка SSL-сертификата на Nginx
Выполняем следующий алгоритм для установки SSL-сертификата.
1. Подключаемся к серверу по SSH.
2. Далее необходимо создать два файла в директории `/etc/ssl` **(!)**:
 - Файл-связка трёх сертификатов - SSL-сертификат формата x509, промежуточный сертификат и корневой сертификат,
 - Файл с приватным ключем.
 >? [(!)] Директория c расположением файлов сертификата, на самом деле, может быть любой - например `/etc/nginx/ssl`, но стандартно все SSL-сертификаты должны располагаться именно в этой директории.
 
 1. Переходим в директорию `/etc/ssl`:
 ```
 cd /etc/ssl
 ```
 2. Создаём пустой файл-связку с расширением `.crt`. Имя созданного файла может быть любым, например, имя может содержать доменное имя и год окончания действия сертификата:
 ```
 touch domain.ru_2019.crt
 ```
 3. Открываем созданный файл в текстовом редакторе (vi, vim, nano):
 ```sh
 vim domain.ru_2019.crt
 ```
 >? Для CentOS вероятнее всего редактор nano не установлен на сервере. Если это необходимо, то его возможно установить следующей командой:
 >? ```yum install nano```

         Далее заполняем файл сертификатами. Сертификаты должны быть вставлены в файл в строгом порядке: SSL-сертификат формата x509, промежуточный сертификат, корневой сертификат. Пробелы и пустые строки между сертификатами не допускаются.
         ```
         -----BEGIN CERTIFICATE-----
         $ SSL-сертификат формата x509 / certificate_x509cert $
         -----END CERTIFICATE-----
         -----BEGIN CERTIFICATE-----
         $ Промежуточный SSL-сертификат / certificate_ca_inter $
         -----END CERTIFICATE-----
         -----BEGIN CERTIFICATE-----
         $ Корневой сертификат / certificate_ca_root $
         -----END CERTIFICATE-----
         ```
         >? Если данных промежуточного сертификата не хватает, то данные можно взять в GlobalSign: [AlphaSSL Intermediate Certificates](https://support.globalsign.com/customer/portal/articles/1223298-alphassl-intermediate-certificates) (для случая установки AlfaSSL-сертификата). Использовать данные в кодировке Base64.
         >?
 
         После того, как данные сертификата перенесены в данный файл, сохраняем его и выходим в директорию `/etc/ssl`.

     4. Создаём пустой файл приватного ключа с расширением `.key`. Имя созданного файла может быть любым, например, имя может содержать доменное имя и год окончания действия сертификата:
     ```sh
     touch domain.ru_2019.key
     ```
     5. Открываем созданный файл в текстовом редакторе (vi, vim, nano):
     ```sh
     vim domain.ru_2019.key
     ```
     Переносим данные приватного ключа в файл.
     ```
     -----BEGIN RSA PRIVATE KEY-----
     $ Приватный ключ / private_key $
     -----END RSA PRIVATE KEY-----
     ```
     После того, как данные приватного ключа перенесены в данный файл, сохраняем его и выходим в директорию `/etc/ssl`.

3. Откроем файл конфигурации Nginx. Общий файл конфигурации находится по следующему пути `/etc/nginx/nginx.conf`.
 ```sh
 vim /etc/nginx/sites-enable/domain.ru
 ```
 >На Debian-based ОС файлы параметров сайтов web-сервера Nginx, как правило, находятся в директории `/etc/nginx/sites-enabled/`. На CentOS стандартное расположение в `/etc/nginx/conf.d/`.
 >>Для того, чтобы определить какой дистрибутив установлен на сервер необходимо выполнить команду:
 >> ```sh
 >>cat /etc/*-release
 >> ```
 >
 >Для поиска нужной конфигурации можно использовать команду `ls /директория/конфигурация`. Например:  
 >```sh
 >ls /etc/nginx/sites-enabled
 >```
 >Данная команда отображает полный список файлов в указанной директории. Затем с помощью текстового редактора необходимо открыть определенный файл. Например:
 >```sh
 >nano /etc/nginx/sites-enabled/domain.ru)
 >```
 >Проверить, что открытый файл действительно является конфигурацией сайта возможно, найдя в нем строку `server_name`. Её значение должно соответствовать доменному имени, для которого устанавливаем SSL-сертификат (например. `www.mydomain.ru`)

4. Далее необходимо добавить в файл конфигурации сайта строки:
 - `ssl on;` - включаем функцию SSL,
 - `ssl_certificate /etc/ssl/domain.ru_2019.crt;` - указываем путь до файла-связки,
 - `ssl_certificate_key /etc/ssl/domain.ru_2019.key;` - указываем путь до приватного ключа.
 Итоговая запись будет выглядеть примерно следующим образом:
 ```
 server{
 listen 443;
 ssl on;
 ssl_certificate /etc/ssl/domain.ru_2019.crt;
 ssl_certificate_key /etc/ssl/domain.ru_2019.key;
 server_name domain.ru;
 ```
 >?Если необходимо, чтобы сайт работал как с защищенным соединением по протоколу HTTPS (https://), так и с незащищенным по протоколу HTTP (http://), необходимо иметь две секции `server{}` в файле конфигурации виртуального хоста для каждого типа соединения: можно создать копию секции `server{}` и оставить её без изменения, а вторую отредактировать, как указано выше в этой статье;
5. Чтобы изменения вступили в силу необходимо перезагрузить Nginx:
 Для Debian-Base:
 ```sh
 /etc/init.d/nginx restart
 ```
 Для CentOS-base:
 ```
 service nginx restart
 ```
6. После всех вышеописанных действий проверяем установку SSL-сертификата с помощью сервиса [SSLShopper](https://www.sslshopper.com/ssl-checker.html).

## Установка SSL-сертификата на Apache
Выполняем следующий алгоритм для установки SSL-сертификата.
1. Подключаемся к серверу по SSH.
2. Активируем использование опции SSL web-сервером Apache:
 - Для Debian-base
 ```sh
 a2enmod ssl
 ```
 - Для CentOS-base
 ```sh
 yum install mod_ssl
 ```
3. Далее необходимо создать три файла в директории `/etc/ssl` **(!)**:
 - SSL-сертификат формата x509,
 - Цепочка сертификатов (содержит промежуточный сертификат, затем корневой),
 - Файл с приватным ключем.
 >? [(!)] Директория c расположением файлов сертификата, на самом деле, может быть любой, но стандартно все SSL-сертификаты должны располагаться именно в этой директории.
 1. Переходим в директорию `/etc/ssl`:
 ```
 cd /etc/ssl
 ```
 2. Создаём пустой файл SSL-сертификата формата x509 с расширением `.crt`. Имя созданного файла может быть любым, например, имя может содержать доменное имя и год окончания действия сертификата:
 ```
 touch domain.ru_2019.crt
 ```
 3. Открываем созданный файл в текстовом редакторе (vi, vim, nano):
 ```sh
 vim domain.ru_2019.crt
 ```
>? Для CentOS вероятнее всего редактор nano не установлен на сервере. Если это необходимо, то его возможно установить следующей командой:
>?```yum install nano```

         Далее заполняем файл данными SSL-сертификата формата x509.
         ```
         -----BEGIN CERTIFICATE-----
         $ SSL-сертификат формата x509 / certificate_x509cert $
         -----END CERTIFICATE-----
         ```
         Сохраняем данные, выходим из редактирования файла.
 4. Создаём пустой файл цепочки сертификатов с расширением `.crt`. Имя созданного файла может быть любым, например, имя может содержать доменное имя и год окончания действия сертификата:
 ```sh
 touch domain.ru_chain_2019.crt
 ```
 5. Открываем созданный файл в текстовом редакторе (vi, vim, nano):
 ```sh
 vim domain.ru_chain_2019.key
 ```
 Переносим данные цепочки сертификатов в файл.
 Цепочка сертификатов, содержит сначала промежуточный сертификат, и следом за ним корневой (с новой строки без пробелов и пустых строк).
 ```
 -----BEGIN CERTIFICATE-----
 $ Промежуточный SSL-сертификат / certificate_ca_inter $
 -----END CERTIFICATE-----
 -----BEGIN CERTIFICATE-----
 $ Корневой сертификат / certificate_ca_root $
 -----END CERTIFICATE-----
 ```  
 >? Если данных промежуточного сертификата не хватает, то данные можно взять у GlobalSign: [AlphaSSL Intermediate Certificates](https://support.globalsign.com/customer/portal/articles/1223298-alphassl-intermediate-certificates) (для случая использования AlfaSSL-сертификата). Использовать данные в кодировке Base64.

         После того, как данные сертификата перенесены в данный файл, сохраняем его и выходим в директорию `/etc/ssl`.

     6. Создаём пустой файл приватного ключа с расширением `.key`. Имя созданного файла может быть любым, например, имя может содержать доменное имя и год окончания действия сертификата:
     ```sh
     touch domain.ru_2019.key
     ```
     7. Открываем созданный файл в текстовом редакторе (vi, vim, nano):
     ```sh
     vim domain.ru_2019.key
     ```
     Переносим данные приватного ключа в файл.
     ```
     -----BEGIN RSA PRIVATE KEY-----
     $ Приватный ключ / private_key $
     -----END RSA PRIVATE KEY-----
     ```
     После того, как данные приватного ключа перенесены в данный файл, сохраняем его и выходим в директорию `/etc/ssl`.
4. Откроем файл конфигурации Apache или файл конфигурации виртуального хоста. В зависимости от операционной системы, данный файл может отличаться своим расположением.

 >На Debian-based ОС файлы параметров сайтов web-сервера Apache, как правило, находятся в директории `/etc/apache2/sites-enabled/`. На CentOS стандартное расположение в `/etc/httpd/conf.d/`.
 >>Для того, чтобы определить какой дистрибутив установлен на сервер необходимо выполнить команду:
 >> ```sh
 >>cat /etc/*-release
 >> ```
 >
 >Для поиска нужной конфигурации можно использовать команду `ls /директория/конфигурация`. Например:  
 >```sh
 >ls /etc/apache2/sites-enabled
 >```
 >Данная команда отображает полный список файлов в указанной директории. Затем с помощью текстового редактора необходимо открыть определенный файл. Например:
 >```sh
 >nano /etc/apache2/sites-enabled/domain.ru)
 >```
 >Проверить, что открытый файл действительно является конфигурацией сайта возможно, найдя в нем строку `ServerName`. Её значение должно соответствовать доменному имени, для которого устанавливаем SSL-сертификат (например. `www.mydomain.ru`)

5. Добавим в открытый файл конфигурации cледующие строки:
 ```
 SSLEngine on
 SSLCertificateFile /etc/ssl/mydomain.ru_crt.crt
 SSLCertificateChainFile /etc/ssl/mydomain.ru_ca.crt
 SSLCertificateKeyFile /etc/ssl/mydomain.ru_key.key
 ```
где
 - `SSLEngine on` - подключение директивы SSL **(!)**,
 - `SSLCertificateFile /etc/ssl/domain.ru.crt` - путь до файла сертификата сайта,
 - `SSLCertificateChainFile /etc/ssl/domain.ru_chain.crt` - путь до файла цепочки сертификатов,
 - `SSLCertificateKeyFile /etc/ssl/domain.ru.key` - путь к файлу приватчного ключа
 >? [(!)] После директивы `SSLEngine on` рекомендуется добавить строку `SSLProtocol all -SSLv2`.

     Если необходимо, чтобы после установки SSL-сертификата сайт был доступен только по безопасному протоколу https (порт 443), отредактируйте файл его конфигурации по аналогии с приведенным ниже **Примером 1**. Если необходимо, чтобы сайт также оставался доступен по незащищенному протоколу http (порт 80), отредактируйте файл его конфигурации по аналогии с приведенным ниже **Примером 2**.


     >? [Пример 1 (только HTTPS):]
     >?```
     >?<VirtualHost *:443>
     >?        # The ServerName directive sets the request scheme, hostname and port that
     >?        # the server uses to identify itself. This is used when creating
     >?        # redirection URLs. In the context of virtual hosts, the ServerName
     >?        # specifies what hostname must appear in the request's Host: header to
     >?        # match this virtual host. For the default virtual host (this file) this
     >?        # value is not decisive as it is used as a last resort host regardless.
     >?        # However, you must set it for any further virtual host explicitly.
     >?        #ServerName www.mydomain.ru
     >?        ServerAdmin webmaster@localhost
     >?        DocumentRoot /var/www/html
     >?        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
     >?        # error, crit, alert, emerg.
     >?        # It is also possible to configure the loglevel for particular
     >?        # modules, e.g.
     >?        #LogLevel info ssl:warn
     >?        ErrorLog ${APACHE_LOG_DIR}/error.log
     >?        CustomLog ${APACHE_LOG_DIR}/access.log combined
     >?        # For most configuration files from conf-available/, which are
     >?        # enabled or disabled at a global level, it is possible to
     >?        # include a line for only one particular virtual host. For example the
     >?        # following line enables the CGI configuration for this host only
     >?        # after it has been globally disabled with "a2disconf".
     >?        #Include conf-available/serve-cgi-bin.conf
     >?        SSLEngine on
     >?        SSLCertificateFile /etc/ssl/domain.ru.crt
     >?        SSLCertificateChainFile /etc/ssl/domain.ru_chain.crt
     >?        SSLCertificateKeyFile /etc/ssl/domain.ru.key
     >?</VirtualHost>
     >?```
   
     >? [Пример 2 (HTTPS и HTTP):]
     >?```
     >?<VirtualHost *:443>
     >?        # The ServerName directive sets the request scheme, hostname and port that
     >?        # the server uses to identify itself. This is used when creating
     >?        # redirection URLs. In the context of virtual hosts, the ServerName
     >?        # specifies what hostname must appear in the request's Host: header to
     >?        # match this virtual host. For the default virtual host (this file) this
     >?        # value is not decisive as it is used as a last resort host regardless.
     >?        # However, you must set it for any further virtual host explicitly.
     >?        #ServerName www.mydomain.ru
     >?        ServerAdmin webmaster@localhost
     >?        DocumentRoot /var/www/html
     >?        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
     >?        # error, crit, alert, emerg.
     >?        # It is also possible to configure the loglevel for particular
     >?        # modules, e.g.
     >?        #LogLevel info ssl:warn
     >?        ErrorLog ${APACHE_LOG_DIR}/error.log
     >?        CustomLog ${APACHE_LOG_DIR}/access.log combined
     >?        # For most configuration files from conf-available/, which are
     >?        # enabled or disabled at a global level, it is possible to
     >?        # include a line for only one particular virtual host. For example the
     >?        # following line enables the CGI configuration for this host only
     >?        # after it has been globally disabled with "a2disconf".
     >?        #Include conf-available/serve-cgi-bin.conf
     >?        SSLEngine on
     >?        SSLCertificateFile /etc/ssl/domain.ru.crt
     >?        SSLCertificateChainFile /etc/ssl/domain.ru_chain.crt
     >?        SSLCertificateKeyFile /etc/ssl/domain.ru.key
     >?</VirtualHost>
     >?<VirtualHost *:80>
     >?        # The ServerName directive sets the request scheme, hostname and port that
     >?        # the server uses to identify itself. This is used when creating
     >?        # redirection URLs. In the context of virtual hosts, the ServerName
     >?        # specifies what hostname must appear in the request's Host: header to
     >?        # match this virtual host. For the default virtual host (this file) this
     >?        # value is not decisive as it is used as a last resort host regardless.
     >?        # However, you must set it for any further virtual host explicitly.
     >?        #ServerName www.mydomain.ru
     >?        ServerAdmin webmaster@localhost
     >?        DocumentRoot /var/www/html
     >?        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
     >?        # error, crit, alert, emerg.
     >?        # It is also possible to configure the loglevel for particular
     >?        # modules, e.g.
     >?        #LogLevel info ssl:warn
     >?        ErrorLog ${APACHE_LOG_DIR}/error.log
     >?        CustomLog ${APACHE_LOG_DIR}/access.log combined
     >?        # For most configuration files from conf-available/, which are
     >?        # enabled or disabled at a global level, it is possible to
     >?        # include a line for only one particular virtual host. For example the
     >?        # following line enables the CGI configuration for this host only
     >?        # after it has been globally disabled with "a2disconf".
     >?        #Include conf-available/serve-cgi-bin.conf
     >?</VirtualHost>
     >?```

6. Чтобы изменения вступили в силу необходимо перезагрузить Nginx:
 Для Debian-Base:
 ```sh
 /etc/init.d/nginx restart
 ```
 Для CentOS-base:
 ```
 service nginx restart
 ```
7. Если на сервере настроен файрвол iptables, необходимо разрешить входящие подключения по протоколу https (порт 443) для файрвола. Для этого воспользуйтесь документацией к ОС, т.к. в разных дистрибутивах Linux работа с iptables осуществляется по разному. Ниже приведено несколько примеров:
 Ubuntu 16.04:
 ```sh
 ufw allow 443/tcp
 ```
 Debian:
 ```
 iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 443 -j ACCEPT
 ```
 CentOS:
 ```
 iptables -A INPUT -p tcp -m tcp --dport 443 -j ACCEPT
 ```
8. После всех вышеописанных действий проверяем установку SSL-сертификата с помощью сервиса [SSLShopper](https://www.sslshopper.com/ssl-checker.html).

## Установка SSL-сертификата в случае использовании Proxy web-сервера
В случае, если на сервере установлено два web-сервера на один IP-адрес: proxy web-сервер, например - Nginx, и backend web-сервер, например - Apache, то SSL-сертификат уставливается на proxy web-сервер. Подобная схема установки двух серверов потребует дополнительной настройки proxy и backend серверов и позволит распределять нагрузку статического и динамического web-трафика между web-серверами.

## Источники
1. [Установка SSL-сертификата на Nginx (Linux)](https://1cloud.ru/help/ssl/installsslnginx)
2. [Установка SSL-сертификата на Nginx](https://www.reg.ru/support/ssl-sertifikaty/ustanovka-ssl-sertifikata/ustanovka-ssl-sertifikata-na-nginx)
3. [Установка SSL-сертификата на Apache (Linux)](https://1cloud.ru/help/ssl/installsslapache)
4. [Установка SSL-сертификата на Apache](https://www.reg.ru/support/ssl-sertifikaty/ustanovka-ssl-sertifikata/ustanovka-SSL-sertifikata-na-apache)
