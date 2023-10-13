# Практическая работа, направленное на определение связи между приходом новых сотрудников в отделе и средней зарплатой в этом отделе.

## Для начала создадим таблицу с отделами и заполним её 5 отделами

```sql
CREATE TABLE departments (
    id SERIAL PRIMARY KEY,
    department_name VARCHAR(255) NOT NULL
);

INSERT INTO departments (department_name) VALUES
    ('Отдел инноваций и развития'),
    ('Отдел операционной эффективности'),
    ('Отдел маркетинга и продвижения'),
    ('Отдел кадров и управления персоналом'),
    ('Отдел исследований и разработки');
```
## Дальше создаём таблицу с сотрудниками
    
```sql
    CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    firstname VARCHAR(50) NOT NULL,
    lastname VARCHAR(50) NOT NULL,
    salary INT NOT NULL,
    hiredate DATE NOT NULL,
    departmentid INT NOT NULL,
    -- Создаем внешний ключ
    FOREIGN KEY (departmentid) REFERENCES departments(id)
);
```
## На сайте [mockaroo.com](https://www.mockaroo.com) генерируем данные для таблицы сотрудников
Выбираем такие параметры как на скриншоте ниже

![Alt text](image-4.png)

Тем самым мы получили sql скрипт с данными, который можно запустить в любой среде для работы с базами данных.

Импортируем данные из файла `Employees.sql` в созданную таблицу

Пример для CLI версии PostgreSQL

```bash
postgres=# \i path/to/file/Employees.sql
```

## Теперь необходимо написать запрос, который будет выводить среднюю зарплату по каждому отделу

Мы используем функцию `AVG()` для вычисления среднего значения зарплаты и функцию `ROUND()` для округления до целого числа.

А также используем оператор `JOIN` для объединения таблиц `employees` и `departments` по полю `departmentid` и `id` соответственно.

```sql
SELECT department_name, ROUND(AVG(salary)) AS avg_salary
FROM employees
JOIN departments ON employees.departmentid = departments.id
GROUP BY department_name;
```

## Теперь необходимо написать запрос, который будет выводить количество новичков в каждом отделе

Возьмём за новичка сотрудника, который был принят меньше 3 месяцев назад.

```sql
SELECT departments.department_name, COUNT(*) AS newbie_count
FROM employees
JOIN departments ON employees.departmentid = departments.id
WHERE employees.hiredate >= NOW() - INTERVAL '3 months'
GROUP BY departments.department_name
ORDER BY newbie_count DESC
```

## По итогу мы можем написать запрос который выведет корреляцию между новичками и средней зарплатой в отделе

```sql
SELECT
    departments.department_name,
    ROUND(AVG(employees.salary)) AS avg_salary,
    COUNT(employees.id) AS employees_count,
    COUNT(CASE WHEN employees.hiredate >= NOW() - INTERVAL '3 months' THEN 1 ELSE NULL END) AS newbie_count
FROM
    employees
JOIN
    departments ON employees.departmentid = departments.id
GROUP BY
    departments.department_name;
    
SELECT corr(avg_salary, newbie_count) AS correlation
FROM (
    SELECT
        AVG(employees.salary) AS avg_salary,
        COUNT(CASE WHEN employees.hiredate >= NOW() - INTERVAL '3 months' THEN 1 ELSE NULL END) AS newbie_count
    FROM
        employees
    GROUP BY
        employees.departmentid
) AS subquery;
```
Если корреляция между новичками и средней зарплатой в отделе больше 0, то это значит что средняя зарплата в отделе растет с приходом новых сотрудников.

![Alt text](image-2.png)

![Alt text](image-3.png)

В моём случае корреляция равняется 0.7, что говорит о том что средняя зарплата в отделе растет с приходом новых сотрудников.

## Дополнительные задания на статистические функции
- Добавить колонку пола сотрудника, заполнить ее случайными данными, и посмотреть как пол влияет на среднюю зарплату в отделе.
- В каком отделе более опытные сотрудники.
- В каком отделе больше всего низкооплачиваемых сотрудников.