# Уровень изоляции Read Committed:

В сессии 1:

```sql
-- Сессия 1
CREATE TABLE my_table (id serial PRIMARY KEY, data text);
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

В сессии 2:

```sql
-- Сессия 2
BEGIN TRANSACTION;
DELETE FROM my_table WHERE id = 1;
COMMIT;
```

В сессии 1:

```sql
-- Сессия 1
SELECT * FROM my_table;
```

Завершите сессию 1:

```sql
-- Сессия 1
COMMIT;
```
# Уровень изоляции Repeatable Read:

В сессии 1:

```sql
-- Сессия 1
CREATE TABLE my_table (id serial PRIMARY KEY, data text);
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

В сессии 2:

```sql
-- Сессия 2
BEGIN TRANSACTION;
DELETE FROM my_table WHERE id = 1;
COMMIT;
```

В сессии 1:

```sql
-- Сессия 1
SELECT * FROM my_table;
```

Завершите сессию 1:

```sql
-- Сессия 1
COMMIT;
```

# Очистка


`SELECT pg_stat_statements_reset();`

`ALTER SYSTEM SET autovacuum = off;`

```sql
CREATE TABLE my_table (
    random_number numeric
);

CREATE INDEX my_index ON my_table (random_number);

INSERT INTO my_table (random_number) SELECT floor(random() * 100000)::integer
FROM generate_series(1, 100000);
```

```sql
SELECT pg_size_pretty(pg_total_relation_size('my_table')) AS table_size,
       pg_size_pretty(pg_indexes_size('my_table')) AS indexes_size;

-- Обновите половину строк на новые случайные числа
UPDATE my_table
SET random_number = floor(random() * 100000)::integer
WHERE random() < 0.5;


-- Запустите полную очистку
VACUUM FULL my_table;
```

`ALTER SYSTEM SET autovacuum = on;`
