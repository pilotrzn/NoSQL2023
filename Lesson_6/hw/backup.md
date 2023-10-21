# Резервное копирование

так же я сделал скрипт для выполнения резевного копирования на ноде бекапов

```
#!/bin/bash

PORT_ARR=(27000 27010 27011 27012)

DAYS=1;
log() {
        TS=`date '+%D %T'`
        echo "$TS $1"
}

DATE=$(date '+%Y-%m-%d')

mkdir -p /var/lib/mongo/backup/${DATE}

time for i in ${PORT_ARR[@]}; do
    PORT=$i
    echo "Locking shard on port ${PORT}"

        mongo  --authenticationDatabase admin --username admin -p admin --port=${PORT} --eval "db.fsyncLock()"

done;

time for j in $(seq 1 1 3); do
    PORT=$j
    echo "Rsyncing shard on port ${PORT}"
    rsync -a /var/lib/mongo/shard0${PORT} /var/lib/mongo/backup/${DATE}
done

echo "Rsyncing config"
rsync -a /var/lib/mongo/config /var/lib/mongo/backup/${DATE}

time for i in ${PORT_ARR[@]};do
    PORT=$i
    echo "Unlocking shard on port ${PORT}"
    mongo --authenticationDatabase admin --username admin -p admin --port=${PORT} --eval "db.fsyncUnlock()"
done;
```

Показал это просто приятным бонусом))

- [Назад](README.md)