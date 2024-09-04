
# Домашняя работа: Systemd - создание init-файла

Цель работы: Научиться редактировать существующие и создавать новые init-файлы.

Что нужно сделать?

- Написать service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова 
 (файл лога и ключевое слово должны задаваться в /etc/default);
 
- Установить spawn-fcgi и создать unit-файл (spawn-fcgi.sevice) с помощью переделки init-скрипта (https://gist.github.com/cea2k/1318020);

- Доработать unit-файл Nginx (nginx.service) для запуска нескольких инстансов сервера с разными конфигурационными файлами одновременно.

- Выполнить следующие задания и подготовить развёртывание результата выполнения с использованием Vagrant и Vagrant shell provisioner.


# Пишем service

Создаём фаил с переменными для service в директории /etc/default:
```
vagrant@sysd:~$ sudo -i
root@sysd:~# cat /etc/default/watchlog
# Configuration file for my watchlog service
# Place it to /etc/default

# File and word in that file that we will be monit
WORD="ALERT"
LOG=/var/log/watchlog.log
```
В директории /var/log/ создаём фаил "watchlog.log" c ключевым словом "ALERT":
```
root@sysd:~# cat /var/log/watchlog.log
+%b %d %T" ALERT
``` 
В директории opt создадим скрипт:

```
root@sysd:~# cat /opt/watchlog.sh
#!/bin/bash

WORD=$1
LOG=$2
DATE=`date`

if grep $WORD $LOG &> /dev/null
then
logger "$DATE: I found word, Master!"
else
exit 0
fi

````
Добавляем права на запуск файла:
````
root@sysd:~# chmod +x /opt/watchlog.sh
````
Cоздаём юниты для service и для timer:

````
root@sysd:~# cat /etc/systemd/system/watchlog.service
[Unit]
Description=My watchlog service

[Service]
Type=oneshot
EnvironmentFile=/etc/default/watchlog
ExecStart=/opt/watchlog.sh $WORD $LOG
````
````
root@sysd:~# cat /etc/systemd/system/watchlog.timer
[Unit]
Description=Run watchlog script every 30 second

[Timer]
# Run every 30 second
OnUnitActiveSec=30
Unit=watchlog.service

[Install]
WantedBy=multi-user.target
````
Запускаем юниты:
````
root@sysd:~# systemctl start watchlog.service
root@sysd:~# systemctl start watchlog.timer
````
Проверяем работу service:
````
root@sysd:~# tail -n 1000 /var/log/syslog | grep word
Sep  4 17:16:27 sysd kernel: [   14.590254] systemd[1]: Started Forward Password Requests to Wall Directory Watch.
Sep  4 17:50:05 sysd root: Wed Sep  4 05:50:05 PM UTC 2024: I found word, Master!
root@sysd:~# tail -n 1000 /var/log/syslog | grep word
Sep  4 17:16:27 sysd kernel: [   14.590254] systemd[1]: Started Forward Password Requests to Wall Directory Watch.
Sep  4 17:50:05 sysd root: Wed Sep  4 05:50:05 PM UTC 2024: I found word, Master!

````

## Установить spawn-fcgi и создать unit-файл (spawn-fcgi.sevice) с помощью переделки init-скрипта (https://gist.github.com/cea2k/1318020)

Устанавливаем spawn-fcgi и необходимые пакеты:

root@sysd:~# apt install spawn-fcgi php php-cgi php-cli apache2 libapache2-mod-fcgid -y

Создадим фаил с настройками для сервиса и юнит сервиса:
````
root@sysd:/etc/spawn-fcgi# cat /etc/spawn-fcgi/fcgi.conf
# You must set some working options before the "spawn-fcgi" service will work.
# If SOCKET points to a file, then this file is cleaned up by the init script.
#
# See spawn-fcgi(1) for all possible options.
#
# Example :
SOCKET=/var/run/php-fcgi.sock
OPTIONS="-u www-data -g www-data -s $SOCKET -S -M 0600 -C 32 -F 1 -- /usr/bin/php-cgi"
````
````
root@sysd:/etc/spawn-fcgi# cat /etc/systemd/system/spawn-fcgi.service
[Unit]
Description=Spawn-fcgi startup service by Otus
After=network.target

[Service]
Type=simple
PIDFile=/var/run/spawn-fcgi.pid
EnvironmentFile=/etc/spawn-fcgi/fcgi.conf
ExecStart=/usr/bin/spawn-fcgi -n $OPTIONS
KillMode=process

[Install]

````
Проверяем работу сервиса:
````
root@sysd:/etc/spawn-fcgi# systemctl start spawn-fcgi
root@sysd:/etc/spawn-fcgi# systemctl status spawn-fcgi
● spawn-fcgi.service - Spawn-fcgi startup service by Otus
     Loaded: loaded (/etc/systemd/system/spawn-fcgi.service; disabled; vendor preset: enabled)
     Active: active (running) since Wed 2024-09-04 18:21:06 UTC; 12s ago
   Main PID: 2787 (php-cgi)
      Tasks: 33 (limit: 1011)
     Memory: 18.8M
        CPU: 94ms
     CGroup: /system.slice/spawn-fcgi.service
             ├─2787 /usr/bin/php-cgi
             ├─2788 /usr/bin/php-cgi
             ├─2789 /usr/bin/php-cgi
             ├─2790 /usr/bin/php-cgi
             ├─2791 /usr/bin/php-cgi
             ├─2792 /usr/bin/php-cgi
             ├─2793 /usr/bin/php-cgi
             ├─2794 /usr/bin/php-cgi
             ├─2795 /usr/bin/php-cgi
             ├─2796 /usr/bin/php-cgi
             ├─2797 /usr/bin/php-cgi
             ├─2798 /usr/bin/php-cgi
             ├─2799 /usr/bin/php-cgi
             ├─2800 /usr/bin/php-cgi
             ├─2801 /usr/bin/php-cgi
             ├─2802 /usr/bin/php-cgi
             ├─2803 /usr/bin/php-cgi
             ├─2804 /usr/bin/php-cgi
             ├─2805 /usr/bin/php-cgi
             ├─2806 /usr/bin/php-cgi
             ├─2807 /usr/bin/php-cgi
             ├─2808 /usr/bin/php-cgi
             ├─2809 /usr/bin/php-cgi
             ├─2810 /usr/bin/php-cgi
             ├─2811 /usr/bin/php-cgi
             ├─2812 /usr/bin/php-cgi
`````

## Доработать unit-файл Nginx (nginx.service) для запуска нескольких инстансов сервера с разными конфигурационными файлами одновременно.

Обновим список реп и установим nginx:

root@sysd:/# apt update
root@sysd:/#  apt install nginx -y

Создадим юнит для работы с шаблонами:
````
root@sysd:/# cat /etc/systemd/system/nginx@.service
# Stop dance for nginx
# =======================
#
# ExecStop sends SIGSTOP (graceful stop) to the nginx process.
# If, after 5s (--retry QUIT/5) nginx is still running, systemd
#takes control
# and sends SIGTERM (fast shutdown) to the main process.
# After another 5s (TimeoutStopSec=5), and if nginx is alive,
systemd sends
# SIGKILL to all the remaining processes in the process group
#(KillMode=mixed).
#
# nginx signals reference doc:
# http://nginx.org/en/docs/control.html
#
[Unit]
Description=A high performance web server and a reverse proxy server
Documentation=man:nginx(8)
After=network.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx-%I.pid
ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx-%I.conf -q -g 'daemon on; master_process on;'
ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx-%I.conf -g 'daemon on; master_process on;'
ExecReload=/usr/sbin/nginx -c /etc/nginx/nginx-%I.conf -g 'daemon on; master_process on;' -s reload
ExecStop=-/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx-%I.pid
TimeoutStopSec=5
KillMode=mixed

[Install]
WantedBy=multi-user.target
````
Создаем два файла конфигурации (nginx-first.conf и nginx-second.conf) на основе стандартного конфига nginx c разделением по портам и модификацией путей 
до pid - файлов, по пути: etc/nginx. 

Запускаем оба сервиса и проверяем:

````
root@sysd:/# systemctl start nginx@first
root@sysd:/# systemctl start nginx@second

root@sysd:/# ss -tnulp | grep nginx
tcp   LISTEN 0      511           0.0.0.0:9002      0.0.0.0:*    users:(("nginx",pid=3640,fd=6),("nginx",pid=3639,fd=6),("nginx",pid=3638,fd=6))
tcp   LISTEN 0      511           0.0.0.0:9001      0.0.0.0:*    users:(("nginx",pid=3631,fd=6),("nginx",pid=3630,fd=6),("nginx",pid=3629,fd=6))
tcp   LISTEN 0      511           0.0.0.0:80        0.0.0.0:*    users:(("nginx",pid=870,fd=6),("nginx",pid=869,fd=6),("nginx",pid=868,fd=6))   
tcp   LISTEN 0      511              [::]:80           [::]:*    users:(("nginx",pid=870,fd=7),("nginx",pid=869,fd=7),("nginx",pid=868,fd=7)) 

````
## Выполнить следующие задания и подготовить развёртывание результата выполнения с использованием Vagrant и Vagrant shell provisioner.

Реализация представлена в блоке "config.vm.provision" vagrant-файла. 



