otus_hw8 Инициализация системы. Systemd.
Разворачиваем через vagrant centos 8
1. Сервис, который мониторит лог на наличие ключевого слова.
 
 Создаем файл с конфигурацией сервиса
 /etc/sysconfig/watchlog:
WORD="ALERT"
LOG=/var/log/watchlog.log

Затем создаем /var/log/watchlog.log и добавляем всякое, включая ключевое слово "ALERT"

создаем скрипт /opt/watchlog.sh
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

Создадим юнит для сервиса /etc/systemd/system/watchlog.service:
[Unit]
Description=My watchlog service
[Service]
Type=oneshot
EnvironmentFile=/etc/sysconfig/watchlog #В методичке опечатка или специально сделано указано watchdog
ExecStart=/opt/watchlog.sh $WORD $LOG

Создадим юнит для таймера  /etc/systemd/system/watchlog.timer:
[Unit]
Description=Run watchlog script every 30 second
[Timer]
# Run every 30 second
OnUnitActiveSec=30
Unit=watchlog.service
[Install]
WantedBy=multi-user.target 
 
systemctl start watchlog.timer

 tail -f /var/log/messages 
 Jun 27 10:44:56 localhost systemd[1]: Starting My watchlog service...
Jun 27 10:44:56 localhost root[1334]: Tue Jun 27 10:44:56 UTC 2023: I found word, Master!
Jun 27 10:44:56 localhost systemd[1]: watchlog.service: Succeeded.
Jun 27 10:44:56 localhost systemd[1]: Started My watchlog service.
Jun 27 10:46:12 localhost systemd[1]: Starting My watchlog service...
Jun 27 10:46:13 localhost root[1340]: Tue Jun 27 10:46:12 UTC 2023: I found word, Master!
Jun 27 10:46:13 localhost systemd[1]: watchlog.service: Succeeded.
Jun 27 10:46:13 localhost systemd[1]: Started My watchlog service.

 
 
2.Из репозитория epel установить spawn-fcgi и переписать init-скрипт на unit-файл (имя service должно называться так же: spawn-fcgi)

yum install epel-release -y && yum install spawn-fcgi php php-cli mod_fcgid httpd -y
Раскомментируем строки в /etc/sysconfig/spawn-fcgi и приводим его к следующему виду:
SOCKET=/var/run/php-fcgi.sock
OPTIONS="-u apache -g apache -s $SOCKET -S -M 0600 -C 32 -F 1 -- /usr/bin/php-cgi"

Пишем юнит файл /etc/systemd/system/spawn-fcgi.service :
[Unit]
Description=Spawn-fcgi startup service by Otus
After=network.target
[Service]
Type=simple
PIDFile=/var/run/spawn-fcgi.pid
EnvironmentFile=/etc/sysconfig/spawn-fcgi
ExecStart=/usr/bin/spawn-fcgi -n $OPTIONS
KillMode=process
[Install]
WantedBy=multi-user.target
Стартуем:
systemctl start spawn-fcgi
Проверяем:
systemctl status spawn-fcgi

● spawn-fcgi.service - Spawn-fcgi startup service by Otus
   Loaded: loaded (/etc/systemd/system/spawn-fcgi.service; disabled; vendor preset: disabled)
   Active: active (running) since Tue 2023-06-27 10:53:59 UTC; 3s ago
 Main PID: 1969 (php-cgi)
    Tasks: 33 (limit: 2746)
   Memory: 22.7M
   CGroup: /system.slice/spawn-fcgi.service
           ├─1969 /usr/bin/php-cgi
           ├─1974 /usr/bin/php-cgi
           ├─1975 /usr/bin/php-cgi
           ├─1976 /usr/bin/php-cgi
           ├─1977 /usr/bin/php-cgi
           ├─1978 /usr/bin/php-cgi
           ├─1979 /usr/bin/php-cgi
           ├─1980 /usr/bin/php-cgi
           ├─1981 /usr/bin/php-cgi
           ├─1982 /usr/bin/php-cgi
           ├─1983 /usr/bin/php-cgi
           ├─1984 /usr/bin/php-cgi
           ├─1985 /usr/bin/php-cgi
           ├─1986 /usr/bin/php-cgi
           ├─1987 /usr/bin/php-cgi
           ├─1988 /usr/bin/php-cgi
           ├─1989 /usr/bin/php-cgi
           ├─1990 /usr/bin/php-cgi
           ├─1991 /usr/bin/php-cgi
           ├─1992 /usr/bin/php-cgi
           ├─1993 /usr/bin/php-cgi
           ├─1994 /usr/bin/php-cgi
           ├─1995 /usr/bin/php-cgi
           ├─1996 /usr/bin/php-cgi
           ├─1997 /usr/bin/php-cgi
           ├─1998 /usr/bin/php-cgi
           ├─1999 /usr/bin/php-cgi
           ├─2000 /usr/bin/php-cgi
           ├─2001 /usr/bin/php-cgi
           ├─2002 /usr/bin/php-cgi
           ├─2003 /usr/bin/php-cgi
           ├─2004 /usr/bin/php-cgi
           └─2005 /usr/bin/php-cgi

Jun 27 10:53:59 localhost.localdomain systemd[1]: Started Spawn-fcgi startup service by Otus.

3.Дополнить unit-файл httpd (он же apache) возможностью запустить несколько инстансов сервера с разными конфигурационными файлами.
Скопируем /usr/lib/systemd/system/httpd.service в /etc/systemd/system/httpd.service
Добавим "-%I" в строке в скопированном файле
EnvironmentFile=/etc/sysconfig/httpd-%I 

Добавим 2 файла окружения
/etc/sysconfig/httpd-1:
OPTIONS=-f conf/1.conf
/etc/sysconfig/httpd-2:
OPTIONS=-f conf/2.conf

Добавим 2 файла конфигов, поправить достаточно только second
/etc/httpd/conf/1.conf
/etc/httpd/conf/2.conf:
PidFile /var/run/httpd-2.pid
Listen 8080
systemctl start httpd@1
systemctl start httpd@2
 ss -tnulp | grep httpd
[root@localhost sysconfig]# ss -tnulp | grep httpd
tcp     LISTEN   0        128                    *:8080                *:*       users:(("httpd",pid=4819,fd=4),("httpd",pid=4818,fd=4),("httpd",pid=4817,fd=4),("httpd",pid=4816,fd=4),("httpd",pid=4813,fd=4))
tcp     LISTEN   0        128                    *:80                  *:*       users:(("httpd",pid=4592,fd=4),("httpd",pid=4591,fd=4),("httpd",pid=4590,fd=4),("httpd",pid=4589,fd=4),("httpd",pid=4587,fd=4))






 
 
