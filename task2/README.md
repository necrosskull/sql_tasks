# JOIN, LEFT JOIN, Смена порядка столбцов
## Создание таблиц users и orders
```sql
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(50) NOT NULL
);

CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    product VARCHAR(50) NOT NULL,
    quantity INT NOT NULL,
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);
```
## Получение списка всех заказов с именем пользователя:
```sql
SELECT o.order_id, u.username, o.product, o.quantity
FROM orders o
JOIN users u ON o.user_id = u.user_id;
```

## Получение общего количества продуктов в каждом заказе
```sql
SELECT order_id, SUM(quantity) AS total_quantity
FROM orders
GROUP BY order_id;
```
## Сумма количества товаров всех заказов
```sql
SELECT SUM(quantity) AS total_sum
FROM orders;
```
## Получение списка пользователей, которые не сделали ни одного заказа
```sql
SELECT u.user_id, u.username, o.quantity
FROM users u
LEFT JOIN orders o ON u.user_id = o.user_id
WHERE o.order_id IS NULL;
```
## Получение среднего количества продуктов в заказах для каждого пользователя
```sql
SELECT u.user_id, u.username, AVG(o.quantity) AS avg_quantity
FROM users u
LEFT JOIN orders o ON u.user_id = o.user_id
GROUP BY u.user_id, u.username
ORDER BY (AVG(o.quantity) IS NULL), AVG(o.quantity) DESC;
```
## Получение пользователя с самым большим общим количеством продуктов в заказах
```sql
SELECT u.user_id, u.username, u.email, SUM(o.quantity) AS total_quantity
FROM users u
JOIN orders o ON u.user_id = o.user_id
GROUP BY u.user_id
ORDER BY total_quantity DESC
LIMIT 1;
```
## Смена порядка столбцов
```sql
ALTER TABLE orders RENAME TO orders_temp;

CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    product VARCHAR(50) NOT NULL,
    user_id INT NOT NULL,
    quantity INT NOT NULL,
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);

INSERT INTO orders (order_id, product, user_id, quantity)
SELECT order_id, product, user_id, quantity
FROM orders_temp;

DROP TABLE orders_temp;
```
