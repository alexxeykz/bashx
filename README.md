# bashx
```
# bash_scripts
Создадим Vagrantfile c папками для скриптов scripts.
Подключим папку для переноса скриптов в директории указанные ниже.
```
```
Далее устанавливаем почтовый сервис, чтобы отправлять почту на внешний адрес.
Для того чтобы отчеты уходили на почту устанавливаем утилиту mail (mailx):
```
```
dnf install mailx
```
Для того, чтобы отрабатывали данные утилиты на хосте обязательно должен быть установлен какой-нибудь почтовый сервер.
```
Я выбрал sendmail
```
Почтовые системы очень чувствительны к hostname.hostdomain
```
Поэтому правим файл hosts перед установкой:
[root@bashx nginx]# vi /etc/hosts
127.0.0.1     localhost localhost.localdomain
127.0.0.1     bashx.lan
```
Далее устанавливаем:
```
dnf install sendmail sendmail-cf cyrus-sasl-plain
```
Затем создаем файл для ретрансляции:
```
touch /etc/mail/authinfo
```
Отредактируем файл, укажите полное доменное имя SMTP Relay и учетные данные для аутентификации:
```
[root@bashx nginx]# vi /etc/mail/authinfo
AuthInfo:mail.uk*****ie.ru "U:root" "I:a.kaz*****ko@uk*****ie.ru" "P:Mono******!" "M:LOGIN PLAIN"
```

Создаем базу данных аутентификации на основе текстового файла authinfo:
```
[root@bashx nginx]# cd /etc/mail/
[root@bashx nginx]# makemap hash authinfo < authinfo
```
Изменяем права доступа к файлам authinfo и authinfo.db:
```
[root@bashx nginx]#chmod 600 /etc/mail/authinfo*
```
Изменяем исходный файл конфигурации sendmail:
```
[root@bashx nginx]# vi /etc/mail/sendmail.mc
```
```
Изменяем данные строчки
dnl #
define(`SMART_HOST', `mail.uk*****ie.ru')dnl
FEATURE(`authinfo', `hash -o /etc/mail/authinfo.db')dnl
dnl #
```
Восстанавливаем файл /etc/mail/sendmail.cf :
```
[root@bashx nginx]# /etc/mail/make
```
Запустите службу sendmail:
```
[root@bashx nginx]# systemctl start sendmail
[root@bashx nginx]# systemctl enable sendmail
```
Проверяем
[root@bashx nginx]# systemctl status sendmail
```
[root@bashx nginx]# systemctl status sendmail
● sendmail.service - Sendmail Mail Transport Agent
   Loaded: loaded (/usr/lib/systemd/system/sendmail.service; enabled; vendor preset: disabled)
   Active: active (running) since Tue 2024-05-14 17:04:25 UTC; 44min ago
  Process: 966 ExecStart=/usr/sbin/sendmail -bd $SENDMAIL_OPTS $SENDMAIL_OPTARG (code=exited, status=0/SUCCESS)
  Process: 954 ExecStartPre=/etc/mail/make aliases (code=exited, status=0/SUCCESS)
  Process: 910 ExecStartPre=/etc/mail/make (code=exited, status=0/SUCCESS)
 Main PID: 1217 (sendmail)
    Tasks: 1 (limit: 11402)
   Memory: 7.5M
   CGroup: /system.slice/sendmail.service
           └─1217 sendmail: accepting connections
```
Проверяем отправку
```
[root@bashx nginx]# sendmail -v a.*****@uk*****ie.ru < /var/log/otchet.txt
```
```
Проверить логи можно командой journalctl -f -u sendmail
```
Проверка прошла, переходим к скрипту.


Пишем скрипт
```
написать скрипт для крона, который раз в час присылает на заданную почту
Список IP адресов (с наибольшим кол-вом запросов) с указанием кол-ва запросов c момента последнего запуска скрипта;
Список запрашиваемых URL (с наибольшим кол-вом запросов) с указанием кол-ва запросов c момента последнего запуска скрипта;
- Ошибки веб-сервера/приложения c момента последнего запуска;
- список всех кодов возврата с указанием их кол-ва с момента последнего запуска
Скрипт должен предотвращать одновременный запуск нескольких копий, до его завершения.
```
```
Скрипт анализирует файл access.log веб-сервера, который находится в /var/log/nginx
```
Для работы скриптов требуется поместить его в директорию где находится лог файл. То есть:
```
mv full.sh mail.sh /var/log/nginx
```
```
Так же необходимо в папке /etc/cron.hourly - Данная директория запускает раз в час.
```
```
Разместить скрипт 0mail.cron который будет запускать скрипт mail.sh раз в час, и не допускать мультизапуск.
```
```
Скрипт mail.sh производит  поиск и запуск скрипта full.sh и отправляет его вывод на почту root-пользователя.
```

[root@bashx cron.hourly]# ./0mail.cron
Lockfile active, no new runs.
```
![10](https://github.com/alexxeykz/bashx/assets/163057177/54cbc7ab-c1af-45c1-bf00-0da261603d5b)
[UpВременной диапазон:
[14/May/2024:04:12:10
May 14 22:01:57 2024
Топ-10 клиентских URL запрашиваемых с этого сервера
    157 /
    120 /wp-login.php
     57 /xmlrpc.php
     26 /robots.txt
     12 /favicon.ico
     11 400
      9 /wp-includes/js/wp-embed.min.js?ver=5.0.4
      7 /wp-admin/admin-post.php?page=301bulkoptions
      7 /1
      6 /wp-content/uploads/2016/10/robo5.jpg
      6 /wp-content/uploads/2016/10/robo4.jpg
      6 /wp-content/uploads/2016/10/robo3.jpg
      6 /wp-content/uploads/2016/10/robo2.jpg
      6 /wp-content/uploads/2016/10/robo1.jpg
      6 /wp-content/uploads/2016/10/aoc-1.jpg
      6 /wp-content/uploads/2016/10/agreed.jpg
      6 /wp-content/themes/llorix-one-lite/style.css?ver=1.0.0
      6 /wp-admin/admin-ajax.php?page=301bulkoptions
      5 /wp-includes/js/wp-emoji-release.min.js?ver=5.0.4
      5 /wp-includes/css/dist/block-library/style.min.css?ver=5.0.4
------------------------------------------------------
Топ-10 клиентских IP
     12 95.108.181.93
     12 62.210.252.196
     12 185.142.236.35
     12 162.243.13.195
      8 163.179.32.118
      7 87.250.233.75
      6 167.99.14.153
      6 165.22.19.102
      5 71.6.199.23
      5 5.45.203.12
------------------------------------------------------
Все коды состояния HTTP и их количество
    498 200
     95 301
     51 404
      7 400
      3 500
      2 499
      1 405
      1 403
      1 304
------------------------------------------------------
Все коды состояния  4xx и 5xx
     14 404
     13 404
      8 404
      6 404
      3 500
      3 404
      3 404
      3 400
------------------------------------------------------
all
loading unknown (2)…]()
```


