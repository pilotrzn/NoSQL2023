# Домашнее задание

Стенд: Ноутбук с установленной Fedora 36 KDE.

## Установка MongoDB 

Установка в docker:

```code
$ docker run --name mongodb -d \
-v ~/docker/mongodb:/data/db \
-p 27017:27017 \
-e MONGO_INITDB_ROOT_USERNAME=alex \
-e MONGO_INITDB_ROOT_PASSWORD=alexpass \
mongo
```

Чтобы была возможность подключаться и работать с установленной MongoBD используем 2 решения:
- MongoBD Compass
- MongoDB mongosh

Для проверки работоспособности выполним подключение к докеру:

```
$ mongosh --port 27017 -u "alex" -p "alexpass"
> test
```

В результате попадаем в консоль mongodb. В консоли создадим базу learndb и в ней коллекцию sample:

```code
test> use learndb
switched to db learndb
learndb> db.createCollection('sample')
{ ok: 1 }
```

Аналогично подключаемся с помощью Compass. Проверяем что наша база видна:

![compass][1]

[1]: ../img/compass.png


## Заполнение данными

Перед выполнением процедуры импорта данных я скопировал csv файл в каталог ~/docker/mongodb/importfiles. 
Его я предварительно создал в консоли mongo, подключившись через команду:

```bash
~$ docker exec -it mongodb bash
```

Чтобы скопированный файл csv можно было импортировать, сначала ему меняю владельца на mongodb

После подключения к консоли mongo выполняем команду:

```bash
~$ mongoimport --host=127.0.0.1 --authenticationDatabase=admin -d citibikes -c tripdata --type csv --file /data/db/importfiles/201912-citibike-tripdata-subset.csv --headerline --username 'alex' --password 'alexpass'
```

Налиие данных можем проверить в Compass.
![compass_bikes][2]

[2]: ../img/compass_citibikes.png
