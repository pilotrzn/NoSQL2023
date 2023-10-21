# Установка Монго

Мой тестовый кластер будет выполнен на версии монго 3.6(связано с работой)

---
1. Для установки добавил репозиторий

```
cat /etc/yum.repos.d/mongodb.repo

[mongodb-org-3.6]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.6/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.6.asc
```

---
2. Устанавливаем Монго

```
$ sudo yum install -y mongodb-org-3.6.13 mongodb-org-server-3.6.13 mongodb-org-shell-3.6.13 mongodb-org-mongos-3.6.13 mongodb-org-tools-3.6.13
```

Установка выполняется на каждой машине.
После установки будут созданы каталоги, в случае с /var/lib/mongo - переназначены права, так как у нас уже примаплены отдельные диски. НА машине с арбитром дополнительный диск под данные не требуется, так как арбитр не хранит данных.

---
3. Каталоги 

На каждом сервере создадим каталоги под бд для каждого репликасета

```
$ ls -lstr /var/lib/mongo/
total 16
4 drwxr-xr-x  4 mongod mongod 4096 Oct 17 20:21 shard01
4 drwxr-xr-x. 4 mongod mongod 4096 Oct 17 20:21 shard02
4 drwxr-xr-x. 4 mongod mongod 4096 Oct 17 20:21 shard03
4 drwxr-xr-x. 4 mongod mongod 4096 Oct 17 20:21 config
```

---
4. Для использования аутентификации репликасетов по ключу создаем ключи

```
$ sudo mkdir /etc/mongod.d/ #если не существует
$ sudo chown mongod:mongod /etc/mongod.d
$ sudo openssl rand -base64 756 > /etc/mongod.d/mongod.key
$ sudo chmod 400 /etc/mongod.d/mongod.key
$ sudo chown mongod:mongod /etc/mongod.d/mongod.key
```

Далее создаем такой же каталог на каждой машине и копируем туда созданный файл ключа. Не забываем назначить права

Возможно я что-то некорректно проделывал, но для нормальной работы созданный ключ пришлось раскопировать для каждого репликасета. Поэтому в каталоге созданы следующие ключи:

```
$ ls -lstr *.key
4 -rw------- 1 mongod mongod 90 Sep 25 15:48 config.key
4 -rw------- 1 mongod mongod 90 Sep 25 15:49 sh1.key
4 -rw------- 1 mongod mongod 90 Sep 25 15:49 sh2.key
4 -rw------- 1 mongod mongod 90 Sep 25 15:49 sh3.key
4 -rw------- 1 mongod mongod 90 Sep 25 15:49 mongos.key
```

---
5. Конфигурирование

Каждый член репликасета изначально будет запущен в standalone режиме. Для каждого члена на каждом сервере создаем конфигурационные файлы:

```
$ ls -lstr *.conf
4 -rw-r--r-- 1 root   root   758 Sep 25 15:53 mongod_sh2.conf
4 -rw-r--r-- 1 root   root   728 Sep 25 15:53 mongod_sh1.conf
4 -rw-r--r-- 1 root   root   729 Oct  7 22:26 mongod_sh3.conf
4 -rw-r--r-- 1 root   root   459 Oct 10 18:15 mongos.conf
4 -rw-r--r-- 1 mongod mongod 548 Oct 10 22:47 mongod_config.conf
```

Содержимое файла конфигсета:

```
$ cat mongod_config.conf
systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongod_config.log

storage:
  dbPath: /var/lib/mongo/config
  journal:
    enabled: true

processManagement:
  fork: true  
  pidFilePath: /var/run/mongodb/mongod.pid 

net:
  port: 27000
  bindIp: 0.0.0.0 

security:
  authorization: enabled
  keyFile:  /etc/mongod.d/config.key

replication:
  replSetName: config

sharding:
  clusterRole: configsvr
```

Содержимое файла одного из репликасетов:

```
$ cat mongod_sh1.conf
systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongod_sh1.log

storage:
  dbPath: /var/lib/mongo/shard01
  journal:
    enabled: true

processManagement:
  fork: true 
  pidFilePath: /var/run/mongodb/mongod_sh1.pid

net:
  port: 27010
  bindIp: 0.0.0.0

security:
  authorization: enabled
  keyFile:  /etc/mongod.d/sh1.key

replication:
  replSetName: shard01

sharding:
  clusterRole: shardsvr
```

Каждый репликасет расписывать не буду. для остальных репликасетов нужно менять порт, путь к бд, путь к логу, путь к ключу и имя replSetName.



---
6. Сервисы

После всех конфигураций так же создаю копии файлов сервисов mongo, в которых меняю пути в Environment и PIDFile.

```
# /usr/lib/systemd/system/mongod.service
[Unit]
Description=High-performance, schema-free document-oriented database
After=network.target

[Service]
User=mongod
Group=mongod
Environment="OPTIONS=-f /etc/mongod.d/mongod_config.conf"
ExecStart=/usr/bin/mongod $OPTIONS
ExecStartPre=/usr/bin/mkdir -p /var/run/mongodb
ExecStartPre=/usr/bin/chown mongod:mongod /var/run/mongodb
ExecStartPre=/usr/bin/chmod 0755 /var/run/mongodb
PermissionsStartOnly=true
PIDFile=/var/run/mongodb/mongod.pid
Type=forking
# file size
LimitFSIZE=infinity
# cpu time
LimitCPU=infinity
# virtual memory size
LimitAS=infinity
# open files
LimitNOFILE=64000
# processes/threads
LimitNPROC=64000
# locked memory
LimitMEMLOCK=infinity
# total threads (user+kernel)
TasksMax=infinity
TasksAccounting=false

[Install]
WantedBy=multi-user.target
```

После всех настроек запускаю все экземпляры на каждой ноде -  конфиг и 3 будущих репликасета. После запуска экземпляры не знают ничего друг о друге, нужно провести конфигурирование.

- [Назад](README.md)