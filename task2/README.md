# JOIN, LEFT JOIN, смена порядка столбцов, заполнение нового столбца, триггеры
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
## Добавим новый столбец и заполним его данными на основе существующих данных
```sql
ALTER TABLE users
ADD COLUMN total_quantity INT DEFAULT 0;

UPDATE users AS u
SET total_quantity = (
  SELECT SUM(o.quantity)
  FROM orders AS o
  WHERE o.user_id = u.user_id
);
```
## Напишем триггер, который будет это делать сам за нас в дальнейшем
```sql
CREATE OR REPLACE FUNCTION update_total_quantity()
RETURNS TRIGGER AS '
BEGIN
  UPDATE users
  SET total_quantity = (
    SELECT SUM(quantity)
    FROM orders
    WHERE user_id = NEW.user_id
  )
  WHERE user_id = NEW.user_id;
  RETURN NEW;
END;
' LANGUAGE plpgsql;

CREATE TRIGGER update_product_quantity
AFTER INSERT OR UPDATE ON orders
FOR EACH ROW
EXECUTE FUNCTION update_total_quantity();
```
