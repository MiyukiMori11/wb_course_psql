# Внутренняя Архитектура PostgreSQL

___

### Задание:

| Done                            | Задание                                                                             | Результат                                |
|---------------------------------|-------------------------------------------------------------------------------------|------------------------------------------|
| <input type="checkbox" checked> | Развернуть ВМ (Linux) с PostgreSQL                                                  | Done ([еще](#Характеристики-ВМ))         |
| <input type="checkbox" checked> | Залить Тайские перевозки https://github.com/aeuge/postgres16book/tree/main/database | Done ([еще](#Заливка-тайских-перевозок)) |
| <input type="checkbox" checked> | Посчитать количество поездок - select count(*) from book.tickets;                   | 53997475 ([еще](#Количество-поездок))    |

# Подробности

___

### Характеристики ВМ

- 6 GB RAM
- 2 vCPU
- 20 GB SSD
- psql (PostgreSQL) 15.8 (Debian 15.8-0+deb12u1)

### Заливка тайских перевозок

```bash
postgres@test1:~$ wget https://storage.googleapis.com/thaibus/thai_medium.tar.gz && tar -xf thai_medium.tar.gz && psql < thai.sql
--2024-10-02 21:38:17--  https://storage.googleapis.com/thaibus/thai_medium.tar.gz
Resolving storage.googleapis.com (storage.googleapis.com)... 142.250.187.155, 142.251.140.27, 172.217.17.123, ...
Connecting to storage.googleapis.com (storage.googleapis.com)|142.250.187.155|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 884448325 (843M) [application/x-gzip]
Saving to: ‘thai_medium.tar.gz’

thai_medium.tar.gz                               100%[==========================================================================================================>] 843.48M  26.0MB/s    in 41s     

2024-10-02 21:38:59 (20.8 MB/s) - ‘thai_medium.tar.gz’ saved [884448325/884448325]
```

```bash
SET
SET
SET
SET
SET
 set_config 
------------

(1 row)

SET
SET
SET
SET
CREATE DATABASE
ALTER DATABASE
You are now connected to database "thai" as user "postgres".
SET
SET
SET
SET
SET
set_config
------------

(1 row)

SET
SET
SET
SET
CREATE SCHEMA
ALTER SCHEMA
SET
SET
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
CREATE TABLE
ALTER TABLE
CREATE TABLE
ALTER TABLE
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
COPY 5
COPY 60
COPY 10
COPY 111
COPY 109
COPY 1500000
COPY 1500
COPY 200
COPY 3
COPY 53997475
setval
--------
      1
(1 row)

setval
--------
     60
(1 row)

setval
--------
      1
(1 row)

setval
---------
1500000
(1 row)

setval
--------
1500
(1 row)

setval
--------
    200
(1 row)

setval
--------
      1
(1 row)

setval
----------
53997475
(1 row)

ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
```

### Количество поездок

```bash
postgres=# \c thai
You are now connected to database "thai" as user "postgres".
thai=# select count(*) from book.tickets;
  count   
----------
 53997475
(1 row)
```