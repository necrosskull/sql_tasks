# Pr 5

1. **Создание базы данных и таблицы:**
```sql
-- Создание базы данных "Library"
CREATE DATABASE Library;

-- Подключение к базе данных "Library"
\c Library

-- Создание таблицы "Books"
CREATE TABLE Books (
    ID SERIAL PRIMARY KEY,
    Title VARCHAR(255),
    Author VARCHAR(255),
    Year INTEGER
);
```

2. **Заполнение данными:**
```sql
-- Вставка нескольких записей в таблицу "Books"
INSERT INTO Books (Title, Author, Year) VALUES
('Книга 1', 'Автор 1', 1998),
('Книга 2', 'Автор 2', 2005),
('Книга 3', 'Автор 3', 2010);
```

3. **SQL-запросы:**
```sql
-- Получение всех книг из таблицы "Books"
SELECT * FROM Books;

-- Получение книг, опубликованных после 2000 года
SELECT * FROM Books WHERE Year > 2000;

-- Получение книг определенного автора
SELECT * FROM Books WHERE Author = 'Автор 1';

-- Обновление года издания для определенной книги
UPDATE Books SET Year = 2022 WHERE Title = 'Книга 1';

-- Удаление книги из таблицы
DELETE FROM Books WHERE Title = 'Книга 2';
```

# Pr 6

### Более сложные примеры можно найти [тут](https://github.com/necrosskull/sql_tasks/tree/main/task1)

Давайте создадим пример базы данных с таблицей "employees" и выполним некоторые из перечисленных вами запросов:

1. **Создание таблицы с числовыми значениями:**
```sql
CREATE TABLE employees (
    employee_id serial PRIMARY KEY,
    first_name varchar(50),
    last_name varchar(50),
    salary numeric,
    sales_amount numeric
);

-- Вставим несколько записей для примера
INSERT INTO employees (first_name, last_name, salary, sales_amount) VALUES
('John', 'Doe', 60000, 120000),
('Jane', 'Smith', 75000, 150000),
('Bob', 'Johnson', 50000, 100000),
('Alice', 'Williams', 90000, 180000);
```

2. **Примеры запросов:**
```sql
-- MIN - нахождение минимального значения в столбце
SELECT MIN(salary) AS min_salary FROM employees;

-- MAX - нахождение максимального значения в столбце
SELECT MAX(salary) AS max_salary FROM employees;

-- AVG - нахождение среднего значения в столбце
SELECT AVG(salary) AS avg_salary FROM employees;

-- ROUND - округление числового значения
SELECT ROUND(AVG(salary), 2) AS rounded_avg_salary FROM employees;

-- SUM - нахождение суммы значений в столбце
SELECT SUM(sales_amount) AS total_sales FROM employees;

-- Арифметические операции
SELECT 10^2 AS power_result, salary * 1.1 AS increased_salary FROM employees;

-- NOW() - текущая дата и время
SELECT NOW(), NOW()::DATE, NOW()::TIME, NOW() - INTERVAL '1 year', EXTRACT(year FROM NOW());

-- Unique - создание ограничения уникальности
CREATE TABLE employees_unique (
    employee_id serial PRIMARY KEY,
    email varchar(255) UNIQUE
);

-- Check - создание ограничения на условие
CREATE TABLE students (
    student_id serial PRIMARY KEY,
    age smallint CHECK (age >= 18 AND age <= 30)
);
```

```sql
-- Добавление ограничения уникальности в существующую таблицу
ALTER TABLE employees ADD CONSTRAINT unique_email UNIQUE (email);
```

```sql
-- Добавление ограничения на условие в существующую таблицу
ALTER TABLE students ADD CONSTRAINT check_age_range CHECK (age >= 18 AND age <= 30);
```

# Pr 7

### Пункты с 5 находятся [тут](https://github.com/necrosskull/sql_tasks/tree/main/task2)

1. **Создание роли:**
```sql
CREATE ROLE read_only_role NOLOGIN;
```

2. **Создание пользователя и предоставление ему привилегий:**
```sql
CREATE USER demo_user WITH PASSWORD 'your_password';
GRANT read_only_role TO demo_user;

-- Предоставление прав для чтения данных из демонстрационной базы данных
GRANT CONNECT ON DATABASE demo_database TO read_only_role;
GRANT USAGE ON SCHEMA public TO read_only_role;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO read_only_role;
```

3. **Отзыв привилегий у пользователя:**
```sql
-- Отзыв прав для пользователя
REVOKE read_only_role FROM demo_user;

-- Проверка, что пользователь не может выбирать данные
-- (попытка выполнения SELECT вызовет ошибку)
SELECT * FROM some_table; -- Примерная команда, необходимо заменить на реальный запрос
```

4. **Пример с использованием схемы:**
```sql
-- Создание схемы и таблицы
CREATE SCHEMA my_schema;
CREATE TABLE my_schema.my_table (id serial PRIMARY KEY, name varchar(255));

-- Предоставление прав для чтения данных из таблицы в схеме
GRANT USAGE ON SCHEMA my_schema TO read_only_role;
GRANT SELECT ON TABLE my_schema.my_table TO read_only_role;

-- Попытка выбора данных
-- (попытка выполнения SELECT вызовет ошибку)
SELECT * FROM my_schema.my_table;
```

# Pr 9

### [выполнена тут](https://github.com/necrosskull/sql_tasks/tree/main/task3)




