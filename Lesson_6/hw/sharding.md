# Шардирование бд

Чтобы выполнить разделение базы на шарды, нужно включить шардирование базы, создать индекс, который можно использвать в качестве клчюа шардирования. Без индекса выполнить шардирование не получится

1. Активируем шардинг

```
mongos> sh.enableSharding("taxi_trip")
```

2. Создание индекса

индекс я создал по полю unique_key

```
mongos> use taxi_trip
mongos> db.taxi.createIndex({ "unique_key" : 1})
{
        "raw" : {
                "shard03/mongo01:27012,mongo02:27012" : {
                        "createdCollectionAutomatically" : false,
                        "numIndexesBefore" : 1,
                        "numIndexesAfter" : 2,
                        "ok" : 1
                },
                "shard02/mongo01:27011,mongo02:27011" : {
                        "createdCollectionAutomatically" : false,
                        "numIndexesBefore" : 1,
                        "numIndexesAfter" : 2,
                        "ok" : 1
                },
                "shard01/mongo01:27010,mongo02:27010" : {
                        "createdCollectionAutomatically" : false,
                        "numIndexesBefore" : 1,
                        "numIndexesAfter" : 2,
                        "ok" : 1
                }
        },
        "ok" : 1,
        "operationTime" : Timestamp(1697914492, 1),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1697914492, 1),
                "signature" : {
                        "hash" : BinData(0,"0uGOtcs7b4VrcoobNNrnwBvWib8="),
                        "keyId" : NumberLong("7288421229134872602")
                }
        }
}
```
 Необходимо получить ответ о создании индекса("ok" : 1)


3. Разбиение коллекции

```
mongos> sh.shardCollection("taxi_trip.taxi", {"unique_key" : 1})
```

После этого будет запущен процесс разбиения коллекции на шарды. Процесс не быстрый. Изначально вся база была на первой шарде.

4. Результат

Вывод с детализацией

```
sh.status(true)
--- Sharding Status ---
  sharding version: {
        "_id" : 1,
        "minCompatibleVersion" : 5,
        "currentVersion" : 6,
        "clusterId" : ObjectId("6525ac7edf01e8fbe3aad65b")
  }
  shards:
        {  "_id" : "shard01",  "host" : "shard01/mongo01:27010,mongo02:27010",  "state" : 1 }
        {  "_id" : "shard02",  "host" : "shard02/mongo01:27011,mongo02:27011",  "state" : 1 }
        {  "_id" : "shard03",  "host" : "shard03/mongo01:27012,mongo02:27012",  "state" : 1 }

  autosplit:
        Currently enabled: yes
  balancer:
        Currently enabled:  yes
        Currently running:  no


        {  "_id" : "taxi_trip",  "primary" : "shard01",  "partitioned" : true }
                taxi_trip.taxi
                        shard key: { "unique_key" : 1 }
                        unique: false
                        balancing: true
                        chunks:
                                shard01 74
                                shard02 74
                                shard03 75
                        { "unique_key" : { "$minKey" : 1 } } -->> { "unique_key" : "00005bb306e8284a31755b125d35234d9b6ab266" } on : shard02 Timestamp(13, 0)
                        { "unique_key" : "00005bb306e8284a31755b125d35234d9b6ab266" } -->> { "unique_key" : "012f70fedc3dbb948d38da484d401145df78b1fb" } on : shard02 Timestamp(12, 0)
                        { "unique_key" : "012f70fedc3dbb948d38da484d401145df78b1fb" } -->> { "unique_key" : "0260095b3df1016aaf37360d4999970ad23ffd85" } on : shard02 Timestamp(14, 0)
                        { "unique_key" : "0260095b3df1016aaf37360d4999970ad23ffd85" } -->> { "unique_key" : "028b05b602a44532c93d4d3bb04117bbe7885782" } on : shard02 Timestamp(15, 0)
```

весь вывод не стал расписывать, показал важдную информацию. Как видим, по завершении имеем количество чанков = 74+74+75, распределенное по 3 шардам.

можнон посмотреть распределение данных по шардам

```
mongos> db.taxi.getShardDistribution()

Shard shard01 at shard01/mongo01:27010,mongo02:27010
 data : 3.38GiB docs : 4637869 chunks : 74
 estimated data per chunk : 46.9MiB
 estimated docs per chunk : 62673

Shard shard02 at shard02/mongo01:27011,mongo02:27011
 data : 3.84GiB docs : 5264707 chunks : 74
 estimated data per chunk : 53.24MiB
 estimated docs per chunk : 71144

Shard shard03 at shard03/mongo01:27012,mongo02:27012
 data : 3.54GiB docs : 4851021 chunks : 75
 estimated data per chunk : 48.39MiB
 estimated docs per chunk : 64680

Totals
 data : 10.78GiB docs : 14753597 chunks : 223
 Shard shard01 contains 31.43% data, 31.43% docs in cluster, avg obj size on shard : 784B
 Shard shard02 contains 35.68% data, 35.68% docs in cluster, avg obj size on shard : 784B
 Shard shard03 contains 32.87% data, 32.88% docs in cluster, avg obj size on shard : 784B
```


- [Назад](README.md)