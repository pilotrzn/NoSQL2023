# Тестируем отказоустойчивость

текущее состояние кластера

```
shard01:PRIMARY> rs.status().members
[
        {
                "_id" : 0,
                "name" : "mongo01:27010",
                "health" : 1,
                "state" : 1,
                "stateStr" : "PRIMARY",
                "uptime" : 8464,
                "optime" : {
                        "ts" : Timestamp(1697918981, 1),
                        "t" : NumberLong(14)
                },
                "optimeDate" : ISODate("2023-10-21T20:09:41Z"),
                "syncingTo" : "",
                "syncSourceHost" : "",
                "syncSourceId" : -1,
                "infoMessage" : "",
                "electionTime" : Timestamp(1697912539, 3),
                "electionDate" : ISODate("2023-10-21T18:22:19Z"),
                "configVersion" : 1,
                "self" : true,
                "lastHeartbeatMessage" : ""
        },
        {
                "_id" : 1,
                "name" : "mongo02:27010",
                "health" : 1,
                "state" : 2,
                "stateStr" : "SECONDARY",
                "uptime" : 385,
                "optime" : {
                        "ts" : Timestamp(1697918981, 1),
                        "t" : NumberLong(14)
                },
                "optimeDurable" : {
                        "ts" : Timestamp(1697918981, 1),
                        "t" : NumberLong(14)
                },
                "optimeDate" : ISODate("2023-10-21T20:09:41Z"),
                "optimeDurableDate" : ISODate("2023-10-21T20:09:41Z"),
                "lastHeartbeat" : ISODate("2023-10-21T20:09:42.323Z"),
                "lastHeartbeatRecv" : ISODate("2023-10-21T20:09:43.603Z"),
                "pingMs" : NumberLong(0),
                "lastHeartbeatMessage" : "",
                "syncingTo" : "mongo01:27010",
                "syncSourceHost" : "mongo01:27010",
                "syncSourceId" : 0,
                "infoMessage" : "",
                "configVersion" : 1
        },
        {
                "_id" : 2,
                "name" : "mongo03:27010",
                "health" : 1,
                "state" : 7,
                "stateStr" : "ARBITER",
                "uptime" : 8460,
                "lastHeartbeat" : ISODate("2023-10-21T20:09:43.689Z"),
                "lastHeartbeatRecv" : ISODate("2023-10-21T20:09:43.730Z"),
                "pingMs" : NumberLong(0),
                "lastHeartbeatMessage" : "",
                "syncingTo" : "",
                "syncSourceHost" : "",
                "syncSourceId" : -1,
                "infoMessage" : "",
                "configVersion" : 1
        },
        {
                "_id" : 3,
                "name" : "mongobkp:27010",
                "health" : 1,
                "state" : 2,
                "stateStr" : "SECONDARY",
                "uptime" : 8460,
                "optime" : {
                        "ts" : Timestamp(1697918981, 1),
                        "t" : NumberLong(14)
                },
                "optimeDurable" : {
                        "ts" : Timestamp(1697918981, 1),
                        "t" : NumberLong(14)
                },
                "optimeDate" : ISODate("2023-10-21T20:09:41Z"),
                "optimeDurableDate" : ISODate("2023-10-21T20:09:41Z"),
                "lastHeartbeat" : ISODate("2023-10-21T20:09:41.755Z"),
                "lastHeartbeatRecv" : ISODate("2023-10-21T20:09:43.688Z"),
                "pingMs" : NumberLong(0),
                "lastHeartbeatMessage" : "",
                "syncingTo" : "mongo01:27010",
                "syncSourceHost" : "mongo01:27010",
                "syncSourceId" : 0,
                "infoMessage" : "",
                "configVersion" : 1
        }
]

```

Строка приглашения в монго консоли на 2 ноде выглдит так: 

```
shard01:SECONDARY>
```

На первой ноде остановим сервис, отвечающий за порт 27010. Через несколько секунд вторая нода становится мастером

```
shard01:PRIMARY> rs.status().members
[
        {
                "_id" : 0,
                "name" : "mongo01:27010",
                "health" : 0,
                "state" : 8,
                "stateStr" : "(not reachable/healthy)",
                "uptime" : 0,
                "optime" : {
                        "ts" : Timestamp(0, 0),
                        "t" : NumberLong(-1)
                },
                "optimeDurable" : {
                        "ts" : Timestamp(0, 0),
                        "t" : NumberLong(-1)
                },
                "optimeDate" : ISODate("1970-01-01T00:00:00Z"),
                "optimeDurableDate" : ISODate("1970-01-01T00:00:00Z"),
                "lastHeartbeat" : ISODate("2023-10-21T20:33:38.660Z"),
                "lastHeartbeatRecv" : ISODate("2023-10-21T20:33:15.269Z"),
                "pingMs" : NumberLong(0),
                
                "syncingTo" : "",
                "syncSourceHost" : "",
                "syncSourceId" : -1,
                "infoMessage" : "",
                "configVersion" : -1
        },
        {
                "_id" : 1,
                "name" : "mongo02:27010",
                "health" : 1,
                "state" : 1,
                "stateStr" : "PRIMARY",
                "uptime" : 1823,
                "optime" : {
                        "ts" : Timestamp(1697920416, 1),
                        "t" : NumberLong(15)
                },
                "optimeDate" : ISODate("2023-10-21T20:33:36Z"),
                "syncingTo" : "",
                "syncSourceHost" : "",
                "syncSourceId" : -1,
                "infoMessage" : "could not find member to sync from",
                "electionTime" : Timestamp(1697920404, 1),
                "electionDate" : ISODate("2023-10-21T20:33:24Z"),
                "configVersion" : 1,
                "self" : true,
                "lastHeartbeatMessage" : ""
        },
        {
                "_id" : 2,
                "name" : "mongo03:27010",
                "health" : 1,
                "state" : 7,
                "stateStr" : "ARBITER",
                "uptime" : 1822,
                "lastHeartbeat" : ISODate("2023-10-21T20:33:38.638Z"),
                "lastHeartbeatRecv" : ISODate("2023-10-21T20:33:37.258Z"),
                "pingMs" : NumberLong(0),
                "lastHeartbeatMessage" : "",
                "syncingTo" : "",
                "syncSourceHost" : "",
                "syncSourceId" : -1,
                "infoMessage" : "",
                "configVersion" : 1
        },
        {
                "_id" : 3,
                "name" : "mongobkp:27010",
                "health" : 1,
                "state" : 2,
                "stateStr" : "SECONDARY",
                "uptime" : 1822,
                "optime" : {
                        "ts" : Timestamp(1697920416, 1),
                        "t" : NumberLong(15)
                },
                "optimeDurable" : {
                        "ts" : Timestamp(1697920416, 1),
                        "t" : NumberLong(15)
                },
                "optimeDate" : ISODate("2023-10-21T20:33:36Z"),
                "optimeDurableDate" : ISODate("2023-10-21T20:33:36Z"),
                "lastHeartbeat" : ISODate("2023-10-21T20:33:38.638Z"),
                "lastHeartbeatRecv" : ISODate("2023-10-21T20:33:37.096Z"),
                "pingMs" : NumberLong(0),
                "lastHeartbeatMessage" : "",
                "syncingTo" : "mongo02:27010",
                "syncSourceHost" : "mongo02:27010",
                "syncSourceId" : 1,
                "infoMessage" : "",
                "configVersion" : 1
        }
]
```

Статус показывает, что невозможно подлкючиться к 1 ноде. Об этом сообщает статус 8 и строка "lastHeartbeatMessage" : "Connection refused"

Теперь запустим сервис на 1 ноде. ЗАпросим статус:

```
shard01:PRIMARY> rs.status().members
[
        {
                "_id" : 0,
                "name" : "mongo01:27010",
                "health" : 1,
                "state" : 2,
                "stateStr" : "SECONDARY",
                "uptime" : 2,
                "optime" : {
                        "ts" : Timestamp(1697920516, 1),
                        "t" : NumberLong(15)
                },
                "optimeDurable" : {
                        "ts" : Timestamp(1697920516, 1),
                        "t" : NumberLong(15)
                },
                "optimeDate" : ISODate("2023-10-21T20:35:16Z"),
                "optimeDurableDate" : ISODate("2023-10-21T20:35:16Z"),
                "lastHeartbeat" : ISODate("2023-10-21T20:35:24.834Z"),
                "lastHeartbeatRecv" : ISODate("2023-10-21T20:35:25.728Z"),
                "pingMs" : NumberLong(0),
                "lastHeartbeatMessage" : "",
                "syncingTo" : "mongo02:27010",
                "syncSourceHost" : "mongo02:27010",
                "syncSourceId" : 1,
                "infoMessage" : "",
                "configVersion" : 1
        },
        {
                "_id" : 1,
                "name" : "mongo02:27010",
                "health" : 1,
                "state" : 1,
                "stateStr" : "PRIMARY",
                "uptime" : 1930,
                "optime" : {
                        "ts" : Timestamp(1697920516, 1),
                        "t" : NumberLong(15)
                },
                "optimeDate" : ISODate("2023-10-21T20:35:16Z"),
                "syncingTo" : "",
                "syncSourceHost" : "",
                "syncSourceId" : -1,
                "infoMessage" : "",
                "electionTime" : Timestamp(1697920404, 1),
                "electionDate" : ISODate("2023-10-21T20:33:24Z"),
                "configVersion" : 1,
                "self" : true,
                "lastHeartbeatMessage" : ""
        },
        {
                "_id" : 2,
                "name" : "mongo03:27010",
                "health" : 1,
                "state" : 7,
                "stateStr" : "ARBITER",
                "uptime" : 1929,
                "lastHeartbeat" : ISODate("2023-10-21T20:35:24.703Z"),
                "lastHeartbeatRecv" : ISODate("2023-10-21T20:35:25.325Z"),
                "pingMs" : NumberLong(0),
                "lastHeartbeatMessage" : "",
                "syncingTo" : "",
                "syncSourceHost" : "",
                "syncSourceId" : -1,
                "infoMessage" : "",
                "configVersion" : 1
        },
        {
                "_id" : 3,
                "name" : "mongobkp:27010",
                "health" : 1,
                "state" : 2,
                "stateStr" : "SECONDARY",
                "uptime" : 1929,
                "optime" : {
                        "ts" : Timestamp(1697920516, 1),
                        "t" : NumberLong(15)
                },
                "optimeDurable" : {
                        "ts" : Timestamp(1697920516, 1),
                        "t" : NumberLong(15)
                },
                "optimeDate" : ISODate("2023-10-21T20:35:16Z"),
                "optimeDurableDate" : ISODate("2023-10-21T20:35:16Z"),
                "lastHeartbeat" : ISODate("2023-10-21T20:35:24.703Z"),
                "lastHeartbeatRecv" : ISODate("2023-10-21T20:35:25.170Z"),
                "pingMs" : NumberLong(0),
                "lastHeartbeatMessage" : "",
                "syncingTo" : "mongo02:27010",
                "syncSourceHost" : "mongo02:27010",
                "syncSourceId" : 1,
                "infoMessage" : "",
                "configVersion" : 1
        }
]
```

Теперь видим что старый мастер  стал репликой.

Такой же схемой пробовал останавливать сервисы других репликасетов. Приводил к ситуации, когда мастер 1 репликасета был на первой ноде, мастер 2 репликасета на 2. Остановка арбитра не нарушала работы, так как есть еще скрытый член, который не может быть выбран но в выборах участвует, то есть обеспечивает кворум.


- [Назад](README.md)