# Индексы и их слабые стороны

___

### Задание:

| Done                            | Задание                                                                                 | Результат                                                                                           |
|---------------------------------|-----------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------|
| <input type="checkbox" checked> | Развернуть ВМ (Linux) с PostgreSQL                                                      | -                                                                                                   |
| <input type="checkbox" checked> | Залить Тайские перевозки     https://github.com/aeuge/postgres16book/tree/main/database | -                                                                                                   |
| <input type="checkbox" checked> | Проверить скорость выполнения сложного запроса (приложен в конце файла скриптов)        | cost=3350597.07..3350597.09 ([подробнее](#Скорость-без-индексов))                                   |
| <input type="checkbox" checked> | Навесить индексы на внешние ключи                                                       | -                                                                                                   |
| <input type="checkbox" checked> | Проверить, помогли ли индексы на внешние ключи ускориться                               | cost=3350571.24..3350571.27 rows=10 width=56 ([подробнее](#Скорость-с-индексами-на-внешних-ключах)) |

# Подробности

___

### Скорость без индексов

```sql
                                                                                           QUERY PLAN                                                                                            
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 Limit  (cost=3350597.07..3350597.09 rows=10 width=56) (actual time=234417.068..236546.758 rows=10 loops=1)
   ->  Sort  (cost=3350597.07..3353967.76 rows=1348276 width=56) (actual time=234104.358..236234.047 rows=10 loops=1)
         Sort Key: r.startdate
         Sort Method: top-N heapsort  Memory: 26kB
         ->  HashAggregate  (cost=3274903.65..3321461.31 rows=1348276 width=56) (actual time=233330.667..235985.208 rows=1500000 loops=1)
               Group Key: r.id, ((bs.city || ', '::text) || bs.name), (count(t.id)), (count(s_1.id))
               Planned Partitions: 32  Batches: 33  Memory Usage: 8465kB  Disk Usage: 96144kB
               ->  Hash Join  (cost=2674533.38..3129121.31 rows=1348276 width=56) (actual time=230306.780..234796.378 rows=1500000 loops=1)
                     Hash Cond: (r.fkbus = s_1.fkbus)
                     ->  Hash Join  (cost=2674528.22..3115835.63 rows=1348276 width=84) (actual time=230306.681..234460.858 rows=1500000 loops=1)
                           Hash Cond: (r.fkschedule = s.id)
                           ->  Merge Join  (cost=2674469.07..3097237.69 rows=1348276 width=24) (actual time=230305.767..234226.216 rows=1500000 loops=1)
                                 Merge Cond: (r.id = t.fkride)
                                 ->  Index Scan using ride_pkey on ride r  (cost=0.43..47095.30 rows=1500925 width=16) (actual time=0.013..255.591 rows=1500000 loops=1)
                                 ->  Finalize GroupAggregate  (cost=2674468.65..3016053.87 rows=1348276 width=12) (actual time=230305.739..233661.754 rows=1500000 loops=1)
                                       Group Key: t.fkride
                                       ->  Gather Merge  (cost=2674468.65..2989088.35 rows=2696552 width=12) (actual time=230305.694..233194.090 rows=4500000 loops=1)
                                             Workers Planned: 2
                                             Workers Launched: 2
                                             ->  Sort  (cost=2673468.62..2676839.31 rows=1348276 width=12) (actual time=229113.128..229254.407 rows=1500000 loops=3)
                                                   Sort Key: t.fkride
                                                   Sort Method: external merge  Disk: 38224kB
                                                   Worker 0:  Sort Method: external merge  Disk: 38224kB
                                                   Worker 1:  Sort Method: external merge  Disk: 38224kB
                                                   ->  Partial HashAggregate  (cost=2279951.86..2513152.03 rows=1348276 width=12) (actual time=189186.985..224723.283 rows=1500000 loops=3)
                                                         Group Key: t.fkride
                                                         Planned Partitions: 32  Batches: 33  Memory Usage: 8209kB  Disk Usage: 557064kB
                                                         Worker 0:  Batches: 33  Memory Usage: 8209kB  Disk Usage: 554056kB
                                                         Worker 1:  Batches: 33  Memory Usage: 8209kB  Disk Usage: 554432kB
                                                         ->  Parallel Seq Scan on tickets t  (cost=0.00..838605.63 rows=22499063 width=12) (actual time=4.806..127483.368 rows=17999158 loops=3)
                           ->  Hash  (cost=40.40..40.40 rows=1500 width=68) (actual time=0.902..0.906 rows=1500 loops=1)
                                 Buckets: 2048  Batches: 1  Memory Usage: 94kB
                                 ->  Hash Join  (cost=3.58..40.40 rows=1500 width=68) (actual time=0.055..0.604 rows=1500 loops=1)
                                       Hash Cond: (br.fkbusstationfrom = bs.id)
                                       ->  Hash Join  (cost=2.35..33.57 rows=1500 width=8) (actual time=0.033..0.373 rows=1500 loops=1)
                                             Hash Cond: (s.fkroute = br.id)
                                             ->  Seq Scan on schedule s  (cost=0.00..27.00 rows=1500 width=8) (actual time=0.004..0.125 rows=1500 loops=1)
                                             ->  Hash  (cost=1.60..1.60 rows=60 width=8) (actual time=0.020..0.021 rows=60 loops=1)
                                                   Buckets: 1024  Batches: 1  Memory Usage: 11kB
                                                   ->  Seq Scan on busroute br  (cost=0.00..1.60 rows=60 width=8) (actual time=0.007..0.011 rows=60 loops=1)
                                       ->  Hash  (cost=1.10..1.10 rows=10 width=68) (actual time=0.014..0.015 rows=10 loops=1)
                                             Buckets: 1024  Batches: 1  Memory Usage: 9kB
                                             ->  Seq Scan on busstation bs  (cost=0.00..1.10 rows=10 width=68) (actual time=0.008..0.009 rows=10 loops=1)
                     ->  Hash  (cost=5.10..5.10 rows=5 width=12) (actual time=0.084..0.085 rows=5 loops=1)
                           Buckets: 1024  Batches: 1  Memory Usage: 9kB
                           ->  HashAggregate  (cost=5.00..5.05 rows=5 width=12) (actual time=0.079..0.080 rows=5 loops=1)
                                 Group Key: s_1.fkbus
                                 Batches: 1  Memory Usage: 24kB
                                 ->  Seq Scan on seat s_1  (cost=0.00..4.00 rows=200 width=8) (actual time=0.025..0.036 rows=200 loops=1)
 Planning Time: 0.703 ms
 JIT:
   Functions: 87
   Options: Inlining true, Optimization true, Expressions true, Deforming true
   Timing: Generation 3.817 ms, Inlining 177.378 ms, Optimization 310.980 ms, Emission 190.706 ms, Total 682.882 ms
 Execution Time: 236606.300 ms
(55 rows)
```

### Скорость с индексами на внешних ключах

```sql
                                                                                           QUERY PLAN                                                                                           
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 Limit  (cost=3350571.24..3350571.27 rows=10 width=56) (actual time=189166.437..195183.195 rows=10 loops=1)
   ->  Sort  (cost=3350571.24..3353941.93 rows=1348276 width=56) (actual time=188818.015..194834.771 rows=10 loops=1)
         Sort Key: r.startdate
         Sort Method: top-N heapsort  Memory: 26kB
         ->  HashAggregate  (cost=3274877.83..3321435.48 rows=1348276 width=56) (actual time=188042.223..194587.178 rows=1500000 loops=1)
               Group Key: r.id, ((bs.city || ', '::text) || bs.name), (count(t.id)), (count(s_1.id))
               Planned Partitions: 32  Batches: 33  Memory Usage: 8473kB  Disk Usage: 96144kB
               ->  Hash Join  (cost=2674523.74..3129095.48 rows=1348276 width=56) (actual time=185021.832..193404.527 rows=1500000 loops=1)
                     Hash Cond: (r.fkbus = s_1.fkbus)
                     ->  Hash Join  (cost=2674518.58..3115809.80 rows=1348276 width=84) (actual time=185021.741..193069.989 rows=1500000 loops=1)
                           Hash Cond: (r.fkschedule = s.id)
                           ->  Merge Join  (cost=2674459.43..3097211.86 rows=1348276 width=24) (actual time=185020.939..192831.646 rows=1500000 loops=1)
                                 Merge Cond: (r.id = t.fkride)
                                 ->  Index Scan using ride_pkey on ride r  (cost=0.43..47081.43 rows=1500000 width=16) (actual time=0.012..260.317 rows=1500000 loops=1)
                                 ->  Finalize GroupAggregate  (cost=2674459.01..3016044.22 rows=1348276 width=12) (actual time=185020.916..192265.065 rows=1500000 loops=1)
                                       Group Key: t.fkride
                                       ->  Gather Merge  (cost=2674459.01..2989078.70 rows=2696552 width=12) (actual time=185020.883..191800.226 rows=4500000 loops=1)
                                             Workers Planned: 2
                                             Workers Launched: 2
                                             ->  Sort  (cost=2673458.98..2676829.67 rows=1348276 width=12) (actual time=184691.399..184836.078 rows=1500000 loops=3)
                                                   Sort Key: t.fkride
                                                   Sort Method: external merge  Disk: 38224kB
                                                   Worker 0:  Sort Method: external merge  Disk: 38224kB
                                                   Worker 1:  Sort Method: external merge  Disk: 38224kB
                                                   ->  Partial HashAggregate  (cost=2279943.34..2513142.39 rows=1348276 width=12) (actual time=150166.820..181311.059 rows=1500000 loops=3)
                                                         Group Key: t.fkride
                                                         Planned Partitions: 32  Batches: 33  Memory Usage: 8209kB  Disk Usage: 554272kB
                                                         Worker 0:  Batches: 33  Memory Usage: 8209kB  Disk Usage: 554232kB
                                                         Worker 1:  Batches: 33  Memory Usage: 8209kB  Disk Usage: 578568kB
                                                         ->  Parallel Seq Scan on tickets t  (cost=0.00..838604.48 rows=22498948 width=12) (actual time=7.011..98865.772 rows=17999158 loops=3)
                           ->  Hash  (cost=40.40..40.40 rows=1500 width=68) (actual time=0.790..0.794 rows=1500 loops=1)
                                 Buckets: 2048  Batches: 1  Memory Usage: 94kB
                                 ->  Hash Join  (cost=3.58..40.40 rows=1500 width=68) (actual time=0.073..0.535 rows=1500 loops=1)
                                       Hash Cond: (br.fkbusstationfrom = bs.id)
                                       ->  Hash Join  (cost=2.35..33.57 rows=1500 width=8) (actual time=0.033..0.312 rows=1500 loops=1)
                                             Hash Cond: (s.fkroute = br.id)
                                             ->  Seq Scan on schedule s  (cost=0.00..27.00 rows=1500 width=8) (actual time=0.004..0.090 rows=1500 loops=1)
                                             ->  Hash  (cost=1.60..1.60 rows=60 width=8) (actual time=0.019..0.021 rows=60 loops=1)
                                                   Buckets: 1024  Batches: 1  Memory Usage: 11kB
                                                   ->  Seq Scan on busroute br  (cost=0.00..1.60 rows=60 width=8) (actual time=0.006..0.010 rows=60 loops=1)
                                       ->  Hash  (cost=1.10..1.10 rows=10 width=68) (actual time=0.031..0.032 rows=10 loops=1)
                                             Buckets: 1024  Batches: 1  Memory Usage: 9kB
                                             ->  Seq Scan on busstation bs  (cost=0.00..1.10 rows=10 width=68) (actual time=0.025..0.026 rows=10 loops=1)
                     ->  Hash  (cost=5.10..5.10 rows=5 width=12) (actual time=0.075..0.076 rows=5 loops=1)
                           Buckets: 1024  Batches: 1  Memory Usage: 9kB
                           ->  HashAggregate  (cost=5.00..5.05 rows=5 width=12) (actual time=0.069..0.070 rows=5 loops=1)
                                 Group Key: s_1.fkbus
                                 Batches: 1  Memory Usage: 24kB
                                 ->  Seq Scan on seat s_1  (cost=0.00..4.00 rows=200 width=8) (actual time=0.014..0.026 rows=200 loops=1)
 Planning Time: 2.177 ms
 JIT:
   Functions: 87
   Options: Inlining true, Optimization true, Expressions true, Deforming true
   Timing: Generation 3.420 ms, Inlining 223.999 ms, Optimization 288.697 ms, Emission 182.830 ms, Total 698.946 ms
 Execution Time: 195446.252 ms
(55 rows)
```

### Вывод:
Скорость выполнения ускорилась незначительно, так как самые тяжелые джоины идут из CTE, где индексов нет