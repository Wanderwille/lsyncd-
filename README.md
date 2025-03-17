# Настройка синхронизации файлов с помощью lsyncd

Утилита для синхронизиции файлой с одного хоста, на другой

## Установка: 
```bash
apt install lsyncd -y   # Debian/Ubuntu
yum install lsyncd -y   # CentOS
```

После установки создаем конфигурационный файл ```/etc/lsyncd.conf.lua```

```bash
settings {
    statusFile = "/tmp/lsyncd.status",
    logfile = "/var/log/lsyncd.log",
    nodaemon = false,
    maxProcesses = 1
}

sync {
    default.rsyncssh,
    source = "/home/bitrix/www",
    host = "bitrix@192.168.90.68",
    targetdir = "/home/bitrix/www",
    rsync = {
        archive = true,
        compress = true,
        verbose = true
    }
}
```
В данном случае мы переносим папку синхронизируем папку с сайтом, с Master машины 192.168.90.69 на 192.168.90.68

Делалось из под BitrixVM, утилита должна работать из под пользователя Bitrix

1. Генерируем SSH ключи на мастере(ОЧЕНЬ ВАЖНО ПОЛЬЗОВАТЕЛЬ ДОЛЖЕН БЫТЬ BITRIX):

```bash 
ssh-keygen -t rsa -b 4096 -C ""
```
2. Копируем ключ в (ВНИМАНИЕ:) в файл /home/bitrix/.ssh/authorized_keys

3. Уставнавливаем права: 
```bash
chmod 600 /home/bitrix/.ssh/authorized_keys
chmod 700 /home/bitrix/.ssh/
```

4. Проверяем ssh bitrix@192.168.90.68 (Если подключение есть, значит все отлично)

5. Далее настраиваем саму утилиту на работу из под пользователя Bitrix

Создаем файл ```nano /etc/systemd/system/lsyncd.service``` со следующим содержимым:
```bash
[Unit]
Description=Live Syncing (Mirror) Daemon
After=network.target

[Service]
ExecStart=/usr/bin/lsyncd -nodaemon /etc/lsyncd.conf.lua
Restart=always
User=bitrix
Group=bitrix

[Install]
WantedBy=multi-user.target
```
6. Выполняем следующие команды:
```bash
chown bitrix:bitrix /etc/lsyncd.conf.lua
chown bitrix:bitrix /var/log/lsyncd.log
systemctl daemon-reload
systemctl restart lsyncd
systemctl status lsyncd
```
7. Если все настроено правильно вывод будет примерно таким:
```bash
[root@localhost ~]# systemctl status lsyncd
● lsyncd.service - Live Syncing (Mirror) Daemon
     Loaded: loaded (/etc/systemd/system/lsyncd.service; enabled; preset: disabled)
     Active: active (running) since Mon 2025-03-17 16:49:33 MSK; 6s ago
   Main PID: 302657 (lsyncd)
      Tasks: 3 (limit: 22798)
     Memory: 143.4M
        CPU: 3.759s
     CGroup: /system.slice/lsyncd.service
             ├─302657 /usr/bin/lsyncd -nodaemon /etc/lsyncd.conf.lua
             ├─302658 /usr/bin/rsync --delete --ignore-errors -svzptlogD -r /home/bitrix/www/ bitrix@192.168.90.68:/home/bitrix/www/
             └─302659 ssh -l bitrix 192.168.90.68 rsync --server -svlogDtprze.iLsfxCIvu

Mar 17 16:49:38 localhost.localdomain lsyncd[302658]: bitrix/components/bitrix/forum.topic.active/lang/en/
Mar 17 16:49:38 localhost.localdomain lsyncd[302658]: bitrix/components/bitrix/forum.topic.active/lang/en/.description.php
Mar 17 16:49:38 localhost.localdomain lsyncd[302658]: bitrix/components/bitrix/forum.topic.active/lang/en/.parameters.php
Mar 17 16:49:38 localhost.localdomain lsyncd[302658]: bitrix/components/bitrix/forum.topic.active/lang/en/component.php
Mar 17 16:49:38 localhost.localdomain lsyncd[302658]: bitrix/components/bitrix/forum.topic.active/lang/ru/
Mar 17 16:49:38 localhost.localdomain lsyncd[302658]: bitrix/components/bitrix/forum.topic.active/lang/ru/.description.php
Mar 17 16:49:38 localhost.localdomain lsyncd[302658]: bitrix/components/bitrix/forum.topic.active/lang/ru/.parameters.php
Mar 17 16:49:38 localhost.localdomain lsyncd[302658]: bitrix/components/bitrix/forum.topic.active/lang/ru/component.php
Mar 17 16:49:38 localhost.localdomain lsyncd[302658]: bitrix/components/bitrix/forum.topic.active/templates/
Mar 17 16:49:38 localhost.localdomain lsyncd[302658]: bitrix/components/bitrix/forum.topic.active/templates/.default/
```


