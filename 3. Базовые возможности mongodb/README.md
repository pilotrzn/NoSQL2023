# Домашнее задание

Стенд: Ноутбук с установленной Fedora 36 KDE

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

Чтобы была возможность подключатсья и работать с установленной MongoBD используем 2 решения:
- MongoBD Compass
- MongoDB mongosh

Для проверки работоспособности выполним подключение к докеру:

```
$ mongosh --port 27017 -u "root" -p "otus"
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
