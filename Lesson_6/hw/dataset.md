# Данные

Для наглядной работы и примеров использовал известный датасет чикагского такси, который был сохранен с курса по Postgres и вполне подошел для тестовых данных

загружал файлы csv с помощью скрипта в созданную бд taxi_trip, коллекция taxi.

```
#!/bin/bash

for i in {1..9}; do
        file="taxi.csv.00000000000"$i;

        mongoimport --authenticationDatabase=admin --db=taxi_trip --collection=taxi --port=27017 --username=admin -p admin --type=csv --headerline --file=$file;
done;
```
  в общем было загружено 14 файлов, в результате бд занимает около 6,5 GB

Пример документа


```{
  "_id": {
    "$oid": "6527cdc01e368c287fe9bd49"
  },
  "unique_key": "50781ea8239ff499fc0d582b2769ff2ff4f73e89",
  "taxi_id": "d22d4e2b6b75447d9b7a64a4b680cacfab600f8ee95887bd1107add1516a1ab17933921257292001a299bcc403c0754ec50937d518bddd8e3cf26563967fa4ce",
  "trip_start_timestamp": "2016-10-02 09:30:00",
  "trip_end_timestamp": "2016-10-02 09:45:00",
  "trip_seconds": 472,
  "trip_miles": 1.9,
  "pickup_census_tract": "",
  "dropoff_census_tract": "",
  "pickup_community_area": "",
  "dropoff_community_area": "",
  "fare": 7,
  "tips": 0,
  "tolls": "",
  "extras": 1,
  "trip_total": 8,
  "payment_type": "Cash",
  "company": "303 Taxi",
  "pickup_latitude": "",
  "pickup_longitude": "",
  "pickup_location": "",
  "dropoff_latitude": "",
  "dropoff_longitude": "",
  "dropoff_location": ""
}
```

- [Назад](README.md)