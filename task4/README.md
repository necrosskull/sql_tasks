**Первая часть:**

1. **Создание нежурналируемой таблицы и удаление табличного пространства:**
```bash
cd home/
mkdir /home/tablespace
chown postgres:postgres /home/tablespace
chmod g+rwx /home/tablespace
```

```sql
-- Создание табличного пространства
CREATE TABLESPACE new_tablespace
OWNER postgres
LOCATION '/home/tablespace';

-- Создание нежурналируемой таблицы в пользовательском табличном пространстве
CREATE UNLOGGED TABLE my_unlogged_table (id serial, data text) TABLESPACE new_tablespace;

-- Проверка существования слоя init для таблицы
SELECT relname, relpersistence FROM pg_class WHERE relname = 'my_unlogged_table';

-- Удаление табличного пространства
DROP TABLE my_unlogged_table;
DROP TABLESPACE new_tablespace;
```

2. **Создание таблицы с типом данных text и изменение стратегии хранения:**
```sql
-- Создание таблицы с типом данных text
CREATE TABLE text_table (content text);

-- Проверка текущей стратегии хранения (Storage)
\d+ text_table;

-- Изменение стратегии хранения на external
ALTER TABLE text_table ALTER COLUMN content SET STORAGE EXTERNAL;

-- Вставка короткой строки
INSERT INTO text_table (content) VALUES ('Short text');

-- Вставка длинной строки
INSERT INTO text_table (content) VALUES (repeat('A', 10000));

-- Нахождение отношения
SELECT relname, relfilenode
FROM pg_class
WHERE oid = (
    SELECT reltoastrelid
    FROM pg_class
    WHERE oid = 'text_table'::regclass
);

-- Проверка toast-таблицы
select * from pg_toast.pg_toast_24603;
```

**Вторая часть:**

1. **Создание нового табличного пространства:**
```sql
CREATE TABLESPACE new_tablespace
OWNER postgres
LOCATION '/home/tablespace';
```

2. **Изменение табличного пространства по умолчанию для template1:**
```sql
-- Изменение табличного пространства по умолчанию
ALTER DATABASE template1 SET TABLESPACE new_tablespace;
```

3. **Создание новой базы данных и проверка табличного пространства по умолчанию:**
```sql
-- Создание новой базы данных
CREATE DATABASE new_database;

-- Проверка табличного пространства по умолчанию для новой базы данных
SELECT datname, pg_tablespace_location(dattablespace) FROM pg_database WHERE datname = 'new_database';
```

4. **Просмотр символической ссылки на каталог табличного пространства:**
```sql
-- Проверка символической ссылки
SHOW data_directory;
```

5. **Удаление созданного табличного пространства:**
```sql
-- Переключитесь на базу данных template1 перед удалением
\c template1

-- Удаление табличного пространства
DROP TABLESPACE new_tablespace;
```
