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
10.png



