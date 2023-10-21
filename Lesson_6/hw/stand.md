# Тестовый стенд

1. Для равертывания шардированного отказоустойчивого кластера созданы 4 ВМ

|server |CPU|RAM|  OS   |   Storage    |target|
|-------|---|---|-------|--------------|------|
|mongo01| 4 | 4 |Oracle Linux 7| 15GB |master|
|mongo02| 4 | 4 |Oracle Linux 7| 15GB |replica|
|mongo03| 4 | 4 |Oracle Linux 7| 10GB |arbiter|
|mongobkp| 4 | 4 |Oracle Linux 7| 25GB |backup node|


1 и 2 машины будут выполнять роли мастера и реплики, т.е. переключение будет между ними. 3 нода арбитр, 4 нода для бекапов, скрытая. На каждой машине будет по 1 мастеру и 2 реплики, то есть каждый сервер будет содержать 3 экземпляра монго, плюс 1 конфиг и 1 роутер.

В итоге должны будем получить следующую конфигурацию


| type |mongo01| mongo02 | mongo03 | mongobkp |port|
|------|-------|---------|---------|----------|----|
|config|primary|secondary|secondary|secondary hidden |27000
|rs01|primary|secondary|arbiter|secondary hidden |27010
|rs02|primary|secondary|arbiter|secondary hidden |27011
|rs03|primary|secondary|arbiter|secondary hidden |27012

Роутер mongos так же будет на каждой машине на порту 27017

---

2. На машинах настроим hosts

```
cat /etc/hosts
192.168.122.10 mongo01.domain.local mongo01
192.168.122.11 mongo02.domain.local mongo02
192.168.122.12 mongo03.domain.local mongo03
192.168.122.13 mongobkp.domain.local mongobkp
```

3. Отключаем selinux

```
$ sudo sed -i 's/^SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config
```
4. Настраиваем firewall

```
$ sudo firewall-cmd --permanent --add-port=27000-27017/tcp   
$ sudo firewall-cmd --reload
```

5. смонтированы диски в соданные ранее каталоги

```
/dev/sdb1       /var/lib/mongo  xfs     noatime,nodiratime,noexec 0     0
```

- [Назад](README.md)