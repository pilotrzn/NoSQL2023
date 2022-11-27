# Домашнее задание

Стенд: ПК с установленной OS Debian 11. Docker.

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

Чтобы была возможность подключаться и работать с установленной MongoBD используем 2 решения(установку описывать не буду):

- MongoBD Compass;
- MongoDB mongosh.

Для проверки работоспособности выполним подключение к докеру:

```bash
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

В качестве примера использую образец данных, бд citibike. Перед выполнением процедуры импорта данных я скопировал csv файл в каталог ~/docker/mongodb/importfiles. 
Его я предварительно создал в консоли mongo, подключившись через команду:

```bash
~$ docker exec -it mongodb bash
```

Чтобы скопированный файл csv можно было импортировать, сначала ему меняю владельца на mongodb.

После подключения к консоли mongo для загрузки данных выполняем команду :

```bash
~$ mongoimport --host=127.0.0.1 \
 --authenticationDatabase=admin \
 -d learndb -c tripdata --type csv \
 --file /data/db/importfiles/201912-citibike-tripdata-subset.csv \
 --headerline --username 'alex' --password 'alexpass'
```

Выгружаю данные в новую коллекцию tripdata базы learndb.
Наличие данных можем проверить в Compass.

![compass_bikes][2]

[2]: ../img/compass_citibikes.png

Либо выполнив команду в mongosh:

```bash
learndb> db.tripdata.find()
```

## Выборка данных

Для примера сделаем выборку всех участников 1995 г.р., выведем поля bikeid,start station name, birth year, отсортируем по id городов по возрастанию.

![compass_find_1][3]

[3]: ../img/compass_find_1.png

Аналогична команда в mongosh:

```bash
learndb> db.tripdata.find({"birth year": 1995}, 
{"_id":0,"bikeid":1,"start station name":1,"birth year": 1}).sort({"start station id":1})
```

Другой вариант запроса - найти самого молодого участника с самым старшим id байка, его байк и город, с которого стартовал. Предполагаем что bikeid - порядковый номер регистрации байка, поэтому последний зарегистрированный.

```bash
learndb> db.tripdata.find({}, {"_id":0,"bikeid":1,"start station name":1,"birth year": 1}).sort({"birth year":-1,bikeid: -1}).limit(1)
[
  {
    'start station name': 'W 37 St & 5 Ave',
    bikeid: 41709,
    'birth year': 2003
  }
]

```

То же в Compass:

![compass_find_2][4]

[4]: ../img/compass_find_2.png

## Использование индексов

Явное уменьшение времени обработки запроса на моем количестве данных не будет заметно, поэтому проверку выполним с помощью Explain Plan Compass и посмотрим показатели.

Для начала создадим индекс для поля birth date. В моих запросах к нему происхоит обращение и сортировка.

```bash
learndb> db.tripdata.createIndex({"birth year":1},{"name": "ix_birth_year"})
```

Для сравнения, до создания индекса были показаны такие результаты:

![compass_noindex_1][5]

[5]: ../img/compass_noindex_1.png

![compass_noindex_2][6]

[6]: ../img/compass_noindex_2.png

То есть выполнился обычный collscan всего поля, обрабатывались все документы в базе, сортировка выполнилась в памяти. Время выполнения - 16мс.

После создания индекса:

видим, что время выполнеия = 0, используется индекс, сортировка в памяти не выполняется, количество обработанных документов = 1

![compass_index_1][7]

[7]: ../img/compass_index_1.png


![compass_index_2][8]

[8]: ../img/compass_index_2.png

На примере показателей видно, что примененный индекс увеличивает скорость обработки данных и снижает нагрузку на сервер(не используется память для сортировки).