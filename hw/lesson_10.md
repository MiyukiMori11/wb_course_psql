# Оконные функции и полнотекстовый поиск

___

### Задание:

| Done                            | Задание                                                                           | Результат                    |
|---------------------------------|-----------------------------------------------------------------------------------|------------------------------|
| <input type="checkbox" checked> | Проанализировать данные о зарплатах сотрудников с использованием оконных функций. | Done [(Подробнее)](#Подробности) |

# Подробности

___

### Запрос

```sql
SELECT e.first_name,
       e.last_name,
       g.value,
       s.amount,
       COALESCE(s.amount - LAG(s.amount) OVER (PARTITION BY e.id ORDER BY s.from_date), 0) AS salary_diff
FROM salary s
         JOIN
     employee e ON s.fk_employee = e.id
         JOIN grade g ON g.id = s.fk_grade
ORDER BY e.id, s.from_date;
```

### Результат

| first_name | last_name | value  | amount | salary_diff |
|------------|-----------|--------|--------|-------------|
| Eugene     | Aristov   | junior | 100000 | 0           |
| Eugene     | Aristov   | middle | 200000 | 100000      |
| Eugene     | Aristov   | senoir | 300000 | 100000      |
| Ivan       | Ivanov    | middle | 200000 | 0           |
| Petr       | Petrov    | middle | 200000 | 0           |

---
