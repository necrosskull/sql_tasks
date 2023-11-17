**Первая часть:**

1. **Создание нежурналируемой таблицы и удаление табличного пространства:**
```sql
-- Создание табличного пространства
CREATE TABLESPACE my_tablespace LOCATION '/path/to/my_tablespace';

-- Создание нежурналируемой таблицы в пользовательском табличном пространстве
CREATE UNLOGGED TABLE my_unlogged_table (id serial, data text) TABLESPACE my_tablespace;

-- Проверка существования слоя init для таблицы
SELECT relname, relpersistence FROM pg_class WHERE relname = 'my_unlogged_table';

-- Удаление табличного пространства
DROP TABLESPACE my_tablespace;
```

2. **Создание таблицы с типом данных text и изменение стратегии хранения:**
```sql
-- Создание таблицы с типом данных text
CREATE TABLE text_table (id serial, content text);

-- Проверка текущей стратегии хранения (Storage)
SELECT column_name, storage FROM information_schema.columns WHERE table_name = 'text_table';

-- Изменение стратегии хранения на external
ALTER TABLE text_table ALTER COLUMN content SET STORAGE EXTERNAL;

-- Вставка короткой строки
INSERT INTO text_table (content) VALUES ('Short text');

-- Вставка длинной строки
INSERT INTO text_table (content) VALUES (repeat('A', 10000));

-- Проверка toast-таблицы
SELECT * FROM pg_toast.pg_toast_1;
```

**Вторая часть:**

1. **Создание нового табличного пространства:**
```sql
CREATE TABLESPACE new_tablespace LOCATION '/path/to/new_tablespace';
```

2. **Изменение табличного пространства по умолчанию для template1:**
```sql
-- Сначала подключитесь к базе данных template1
\c template1

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
