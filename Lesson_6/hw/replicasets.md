# Настройка репликасетов

После того, как будут запущены экземпляры, выполняем конфигурирование на первой ноде. Предполагаем что она будет являться мастером.

---
1. Настройка конфигсета

подключаемся в консоль монго 

```
$ mongo --port 27000
```

В консоли выполняем команды

```
> rsconf = {
    _id: "config",
    configsvr: true,
    members: [
        {_id: 0, host: "mongo01:27000"},
        {_id: 1, host: "mongo02:27000"},
        {_id: 2, host: "mongo03:27000"},
        {_id: 3, host: "mongobkp:27000", "hidden": true, "priority" : 0}
    ]
}
> rs.initiate(rsconf)
```

После инициализации получаем репликасет конфиг из 3 нод.

---
2. Настройка репликасетов

На первой же ноде подклчаемся в консоль монго

```
$ mongo --port 27010
```

В консоли ыполняем команды

```
> rsconf = {
    _id: "shard01",
    members: [
        {_id: 0, host: "mongo01:27010"},
        {_id: 1, host: "mongo02:27010"},
        {_id: 2, host: "mongo03:27010", "arbiterOnly": true},
        {_id: 3, host: "mongobkp:27010", "hidden": true,"priority" :0 }
    ]
}
> rs.initiate(rsconf)
```
---

Далее 

```
$ mongo --port 27011
> rsconf = {
    _id: "shard02",
    members: [
        {_id: 0, host:  "mongo01:27011"},
        {_id: 1, host:  "mongo02:27011"},
        {_id: 2, host:  "mongo03:27011", "arbiterOnly": true},
        {_id: 3, host: "mongobkp:27011", "hidden": true,"priority" :0 }
    ]
}
> rs.initiate(rsconf)
```

---
И 3 репликасет

```
mongo --port 27012
> rsconf = {
    _id: "shard03",
    members: [
        {_id: 0, host:  "mongo01:27012"},
        {_id: 1, host:  "mongo02:27012"},
        {_id: 2, host:  "mongo03:27012", "arbiterOnly": true},
        {_id: 3, host: "mongobkp:27012", "hidden": true,"priority" :0 }
    ]
}
> rs.initiate(rsconf)
```

в Итоге получаем 4 независимых репликасета, из которых один является конфигсетом.

Далее собираем все в общий кластер.

---
3. Кластер

Чтобы собрать всё воедино подключаемся к mongos консоли

```
mongo --port 27017
mongos>sh.addShard( "shard01/mongo01:27010,mongo02:27010")
mongos>sh.addShard( "shard02/mongo01:27011,mongo02:27011")
mongos>sh.addShard( "shard03/mongo01:27012,mongo02:27012")
```
Внимание! добавляем только видимые шарды. скрытые и арбитры добавлять не нужно.

---
4. ПОльзователи

Возможно я сделал некорректно, но в общем то получилось так. Поэтому опишу. после добавления шард в mongos я создал пользователя

```
mongos> db.createUser(
    {
      user: "admin",
      pwd: "admin",
      roles: [ { role: "root", db: "admin" } ]
    }
  )
```

тоже самое проделал на каждом репликасете. После яего подклчение в консоль стало возможным в следующем виде:

```
$ mongo --port 27000 --authenticationDatabase admin --username admin -p admin
```

---
5. Статус

весь листинг команды выводить не буду, покажу часть про шарды

```
mongos> sh.status()
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
  active mongoses:
        "3.6.13" : 4
  autosplit:
        Currently enabled: yes
  balancer:
        Currently enabled:  yes
```
---

- [Назад](README.md)