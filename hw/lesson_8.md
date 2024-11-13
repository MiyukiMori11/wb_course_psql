# Языки программирования и пользовательские типы данных

___

### Задание:

| Done                            | Задание                                                                            | Результат                      |
|---------------------------------|------------------------------------------------------------------------------------|--------------------------------|
| <input type="checkbox" checked> | Создать таблицу с продажами.                                                       | Done [(Подробнее)](#Таблица)   |
| <input type="checkbox" checked> | Реализовать функцию выбор трети года (1-4 месяц - первая треть, 5-8 - вторая и тд) | Done [(Подробнее)](#Функция)   |
| <input type="checkbox" checked> | Вызвать эту функцию в SELECT из таблицы с продажами, уведиться, что всё отработало | Done [(Подробнее)](#Результат) |

# Подробности

___

### Таблица

```sql
CREATE TABLE sales
(
    order_id integer not null,
    sale_dt  timestamptz,
    meta     text
);

INSERT INTO sales (order_id, sale_dt)
VALUES (1, '2010-01-10'),
       (2, '2010-05-10'),
       (3, '2010-09-10'),
       (4, '2010-04-10'),
       (5, '2010-08-10'),
       (6, '2010-12-10'),
       (7, null);
```

### Функция

```sql
CREATE OR REPLACE FUNCTION get_year_third(dt timestamptz, OUT third integer) AS
$$
DECLARE
    month INTEGER := date_part('month', dt);
BEGIN
    third = CASE
                WHEN month BETWEEN 1 AND 4 THEN 1
                WHEN month BETWEEN 5 AND 8 THEN 2
                WHEN month BETWEEN 9 AND 12 THEN 3
        END;
END;
$$ LANGUAGE plpgsql
    STRICT;
```

### Результат

```sql
test=#
SELECT order_id, sale_dt, get_year_third(sale_dt)
FROM sales;
order_id |        sale_dt         | get_year_third
----------+------------------------+----------------
1 | 2010-01-10 00:00:00+03 |              1
2 | 2010-05-10 00:00:00+04 |              2
3 | 2010-09-10 00:00:00+04 |              3
4 | 2010-04-10 00:00:00+04 |              1
5 | 2010-08-10 00:00:00+04 |              2
6 | 2010-12-10 00:00:00+03 |              3
7 |                        |               
(7 rows)
```