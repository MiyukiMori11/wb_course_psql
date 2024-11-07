# Блокировки

___

### Задание:

| Done                            | Задание                                                                                                                  | Результат                      |
|---------------------------------|--------------------------------------------------------------------------------------------------------------------------|--------------------------------|
| <input type="checkbox" checked> | Развернуть асинхронную реплику (можно использовать 1 ВМ, просто рядом кластер развернуть и подключиться через localhost) | -                              |
| <input type="checkbox" checked> | Тестируем производительность по сравнению с сингл инстансом                                                              | Done [(Подробнее)](#Бенчмарки) |

# Подробности

___

## Бенчмарки

### Без репликации

###### Запись:

TPS: 4223.517576

```
postgres@test1:~$ /usr/lib/postgresql/15/bin/pgbench -c 8 -j 4 -T 10 -f ~/workload2.sql -n -U postgres -p 5432 thai
pgbench (15.8 (Debian 15.8-0+deb12u1))
transaction type: /var/lib/postgresql/workload2.sql
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 42228
number of failed transactions: 0 (0.000%)
latency average = 1.894 ms
initial connection time = 9.139 ms
tps = 4223.517576 (without initial connection time)
```

###### Чтение:

TPS: 27179.848487

```
postgres@test1:~$ /usr/lib/postgresql/15/bin/pgbench -c 8 -j 4 -T 10 -f ~/workload.sql -n -U postgres -p 5432 thai
pgbench (15.8 (Debian 15.8-0+deb12u1))
transaction type: /var/lib/postgresql/workload.sql
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 271818
number of failed transactions: 0 (0.000%)
latency average = 0.294 ms
initial connection time = 9.443 ms
tps = 27179.848487 (without initial connection time)
```

###### Вывод:

Производительность по умолчанию, зависит от железа, настроек базы, навешанных индексов и тд
___

### Физическая репликация

#### Мастер:

###### Запись:

TPS: 3299.810182

```
postgres@test1:~$ /usr/lib/postgresql/15/bin/pgbench -c 8 -j 4 -T 10 -f ~/workload2.sql -n -U postgres -p 5432 thai
pgbench (15.8 (Debian 15.8-0+deb12u1))
transaction type: /var/lib/postgresql/workload2.sql
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 32995
number of failed transactions: 0 (0.000%)
latency average = 2.424 ms
initial connection time = 9.804 ms
tps = 3299.810182 (without initial connection time)
```

###### Чтение:

TPS: 28918.312266

```
postgres@test1:~$ /usr/lib/postgresql/15/bin/pgbench -c 8 -j 4 -T 10 -f ~/workload.sql -n -U postgres -p 5432 thai
pgbench (15.8 (Debian 15.8-0+deb12u1))
transaction type: /var/lib/postgresql/workload.sql
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 288949
number of failed transactions: 0 (0.000%)
latency average = 0.277 ms
initial connection time = 10.515 ms
tps = 28918.312266 (without initial connection time)
```

#### Реплика:

###### Чтение:

TPS: 28303.558146

```
postgres@test1:~$ /usr/lib/postgresql/15/bin/pgbench -c 8 -j 4 -T 10 -f ~/workload.sql -n -U postgres -p 5433 thai
pgbench (15.8 (Debian 15.8-0+deb12u1))
transaction type: /var/lib/postgresql/workload.sql
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 283094
number of failed transactions: 0 (0.000%)
latency average = 0.283 ms
initial connection time = 8.787 ms
tps = 28303.558146 (without initial connection time)
```

###### Вывод:

Запись просела ~ на 1000tps, чтение увеличилось (скорее всего прогрев).
Производительность в рамках записи снизилась, но на реальных данных важнее будет снижение нагрузки на мастер за счет 
использования реплики в качестве источника данных.
С реплики чтение аналогично мастеру,
что позволяет ее использовать в качестве производительного источника данных, когда мастер будет нагружаться только изменениями.

___

### Логическая + физическая репликация

#### Мастер:

###### Запись:

TPS: 3314.623149

```
postgres@test1:~$ /usr/lib/postgresql/15/bin/pgbench -c 8 -j 4 -T 10 -f ~/workload2.sql -n -U postgres -p 5432 thai
pgbench (15.8 (Debian 15.8-0+deb12u1))
transaction type: /var/lib/postgresql/workload2.sql
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 33140
number of failed transactions: 0 (0.000%)
latency average = 2.414 ms
initial connection time = 9.492 ms
tps = 3314.623149 (without initial connection time)
```

###### Чтение:

TPS: 28201.281950

```
postgres@test1:/home/suzdaleva.karina$ /usr/lib/postgresql/15/bin/pgbench -c 8 -j 4 -T 10 -f ~/workload.sql -n -U postgres -p 5432 thai
pgbench (15.8 (Debian 15.8-0+deb12u1))
transaction type: /var/lib/postgresql/workload.sql
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 281949
number of failed transactions: 0 (0.000%)
latency average = 0.284 ms
initial connection time = 10.713 ms
tps = 28201.281950 (without initial connection time)
```

##### Реплика:

###### Чтение:

TPS: 1234.083794

```
postgres@test1:/home/suzdaleva.karina$ /usr/lib/postgresql/15/bin/pgbench -c 8 -j 4 -T 10 -f ~/workload.sql -n -U postgres -p 5434 thai
pgbench (15.8 (Debian 15.8-0+deb12u1))
transaction type: /var/lib/postgresql/workload.sql
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 12340
number of failed transactions: 0 (0.000%)
latency average = 6.483 ms
initial connection time = 9.975 ms
tps = 1234.083794 (without initial connection time)
```

###### Вывод:

На логической реплике сильно просело чтение, что обусловлено необходимостью создавать не только таблицу,
но и навешивать индексы. Из плюсов - можно модифицировать таблицу и индексы как угодно, не затрагивая мастер.
Из минусов - необходимо следить, что все необходимые для корректной работы таблицы изменения были применены.

___