# Блокировки

___

### Задание:

| Done                            | Задание                                                                                                     | Результат                     |
|---------------------------------|-------------------------------------------------------------------------------------------------------------|-------------------------------|
| <input type="checkbox" checked> | Создать таблицу accounts(id integer, amount numeric);                                                       | -                             |
| <input type="checkbox" checked> | Добавить несколько записей и подключившись через 2 терминала добиться ситуации взаимоблокировки (deadlock). | Done [(Подробнее)](#deadlock) |
| <input type="checkbox" checked> | Посмотреть логи и убедиться, что информация о дедлоке туда попала.                                          | Done [(Подробнее)](#лог)      |

# Подробности

___

### Deadlock

Терминал 1:
```postgresql
wb=# BEGIN;
BEGIN
wb=*# UPDATE accounts SET amount = 30 WHERE id = 1;
UPDATE 1
```
Терминал 2:
```postgresql
wb=# BEGIN;
BEGIN
wb=*# UPDATE accounts SET amount = 66 WHERE id in (1,3,4);
```
Терминал 1:
```postgresql
wb=*# UPDATE accounts SET amount = 30 WHERE id = 3;
ERROR:  deadlock detected
DETAIL:  Process 316505 waits for ShareLock on transaction 976; blocked by process 316092.
Process 316092 waits for ShareLock on transaction 975; blocked by process 316505.
HINT:  See server log for query details.
CONTEXT:  while updating tuple (0,19) in relation "accounts"
```

### Лог
```postgresql
2024-10-27 20:42:07.603 MSK [316505] postgres@wb ERROR:  deadlock detected
2024-10-27 20:42:07.603 MSK [316505] postgres@wb DETAIL:  Process 316505 waits for ShareLock on transaction 976; blocked by process 316092.
	Process 316092 waits for ShareLock on transaction 975; blocked by process 316505.
	Process 316505: UPDATE accounts SET amount = 30 WHERE id = 3;
	Process 316092: UPDATE accounts SET amount = 66 WHERE id in (1,3,4);
2024-10-27 20:42:07.603 MSK [316505] postgres@wb HINT:  See server log for query details.
2024-10-27 20:42:07.603 MSK [316505] postgres@wb CONTEXT:  while updating tuple (0,19) in relation "accounts"
2024-10-27 20:42:07.603 MSK [316505] postgres@wb STATEMENT:  UPDATE accounts SET amount = 30 WHERE id = 3;
```