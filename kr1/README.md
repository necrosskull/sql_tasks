## Основные конструкции и синтаксис
№1. Выберите все модели самолетов и соответствующие им диапазоны дальности полетов.
```sql
SELECT model, range
FROM aircrafts_data;
```

№2. Найдите все самолеты c максимальной дальностью полета:
1) либо больше 10 000 км, либо меньше 4 000 км;
2) больше 6 000 км, а название не заканчивается на «100».

Обратите внимание на порядок следования предложений WHERE и FROM.
```sql
SELECT model, range
FROM aircrafts_data
WHERE (range > 10000 OR range < 4000) OR (range > 6000 AND model->>'en' NOT LIKE '%100');
```

№3. Определите номера и время отправления всех рейсов, прибывших в аэропорт назначения не вовремя. 
```sql
SELECT flight_no, scheduled_departure
FROM flights
WHERE status = 'Arrived' AND actual_arrival > scheduled_arrival;
```

№4. Подсчитайте количество отмененных рейсов из аэропорта Пулково (LED), как вылет, так и прибытие которых было назначено на четверг.
```sql
SELECT COUNT(*) AS canceled_flights_count
FROM flights
WHERE (departure_airport = 'LED' OR arrival_airport = 'LED')
  AND status = 'Cancelled'
  AND EXTRACT(DOW FROM scheduled_departure) = 4;
```

№5. Выведите имена пассажиров, купивших билеты экономкласса за сумму, превышающую 70 000 рублей.
```sql
SELECT DISTINCT t.passenger_name, tf.amount
FROM tickets t
INNER JOIN ticket_flights tf ON t.ticket_no = tf.ticket_no
WHERE tf.fare_conditions = 'Economy' AND tf.amount > 70000;
```
## Соединения 

№6. Напечатанный посадочный талон должен содержать фамилию и имя пассажира, коды аэропортов вылета и прилета, дату и время вылета и прилета по расписанию, номер места в салоне самолета. Напишите запрос, выводящий всю необходимую информацию для полученных посадочных талонов на рейсы, которые еще не вылетели.
```sql
SELECT
    b.passenger_name AS "Фамилия и имя пассажира",
    da.airport_code AS "Код аэропорта вылета",
    aa.airport_code AS "Код аэропорта прилета",
    f.scheduled_departure AS "Дата и время вылета по расписанию",
    f.scheduled_arrival AS "Дата и время прилета по расписанию",
    bp.seat_no AS "Номер места в салоне самолета"
FROM
    boarding_passes bp
    JOIN flights f ON bp.flight_id = f.flight_id
    JOIN airports_data da ON f.departure_airport = da.airport_code
    JOIN airports_data aa ON f.arrival_airport = aa.airport_code
    JOIN tickets t ON bp.ticket_no = t.ticket_no
WHERE
    f.status = 'Scheduled' OR f.status = 'On Time'
ORDER BY
    f.scheduled_departure;
```

№7. Некоторые пассажиры, вылетающие сегодняшним рейсом («сегодня» определяется функцией bookings.now), еще не прошли регистрацию, т. е. не получили посадочного талона. Выведите имена этих пассажиров и номера рейсов.
```sql
SELECT t.passenger_name, f.flight_no, f.status
FROM bookings b
JOIN tickets t ON b.book_ref = t.book_ref
JOIN ticket_flights tf ON t.ticket_no = tf.ticket_no
JOIN flights f ON tf.flight_id = f.flight_id
WHERE b.book_date <= bookings.now()
  AND f.scheduled_departure <= bookings.now()
  AND f.actual_departure IS NULL;
```

№8. Выведите номера мест, оставшихся свободными в рейсах из Анапы (AAQ) в Шереметьево (SVO), вместе с номером рейса и его датой.
```sql
SELECT DISTINCT flights.flight_no, flights.scheduled_departure, seats.seat_no
FROM flights
JOIN seats ON flights.aircraft_code = seats.aircraft_code
LEFT JOIN boarding_passes ON flights.flight_id = boarding_passes.flight_id
WHERE flights.departure_airport = 'AAQ'
AND flights.arrival_airport = 'SVO'
AND boarding_passes.ticket_no IS NULL;
```

## Агрегирование и группировка
№9. Напишите запрос, возвращающий среднюю стоимость авиабилета из Воронежа (VOZ) в Санкт-Петербург (LED). Поэкспериментируйте с другими агрегирующими функциями (sum, max). Какие еще агрегирующие функции бывают?
```sql
SELECT ROUND(AVG(tf.amount)) AS average_ticket_price
FROM tickets AS t
JOIN ticket_flights AS tf ON t.ticket_no = tf.ticket_no
JOIN flights AS f ON tf.flight_id = f.flight_id
WHERE f.departure_airport = 'VOZ' AND f.arrival_airport = 'LED';
```

№10. Напишите запрос, возвращающий список аэропортов, в которых было принято более 500 рейсов.
```sql
SELECT airports_data.airport_code, airports_data.airport_name, COUNT(flights.flight_id)
FROM airports_data
JOIN flights ON airports_data.airport_code = flights.arrival_airport
GROUP BY airports_data.airport_code
HAVING COUNT(flights.flight_id) > 500
order by COUNT(flights.flight_id) desc;
```
## Модификация данных
№14. Авиакомпания провела модернизацию салонов всех имеющихся самолетов «Сессна» (код CN1), в результате которой был добавлен седьмой ряд кресел. Измените соответствующую таблицу, чтобы отразить этот факт.
```sql
insert into seats (aircraft_code, seat_no, fare_conditions)
VALUES ('CN1', '7A', 'Economy'),
       ('CN1', '7B', 'Economy');

select * from seats
where aircraft_code = 'CN1';
```

№16. Создайте новое бронирование текущей датой. В качестве номера бронирования можно взять любую последовательность из шести символов, начинающуюся на символ подчеркивания. Общая сумма должна составлять 30 000 рублей.
Создайте электронный билет, связанный с бронированием, на ваше имя.
Назначьте электронному билету два рейса: один из Москвы (VKO) во Владивосток (VVO) через неделю, другой — обратно через две недели. Оба рейса выполняются эконом-классом, стоимость каждого должна составлять 15 000 рублей.
```sql
-- Создайте новое бронирование текущей датой
INSERT INTO bookings (book_ref, book_date, total_amount) VALUES ('_NewRe', NOW(), 30000.00);

-- Создайте электронный билет, связанный с бронированием
INSERT INTO tickets (ticket_no, book_ref, passenger_id, passenger_name, contact_data)
VALUES ('_NewTicket', '_NewRe', '_NewPassenger', 'SLAVA KURYAEV', '{"email": "kuryaev02@gmail.com", "phone": "88005553535"}');

-- Создадим два рейса (предположим, что у нас есть доступ к данным о рейсах и нужно использовать их идентификаторы)
INSERT INTO flights (flight_no, scheduled_departure, scheduled_arrival, departure_airport, arrival_airport, status, aircraft_code)
VALUES
  ('FlyOut', NOW() + INTERVAL '1 week', NOW() + INTERVAL '1 week 8 hours', 'VKO', 'VVO', 'Scheduled', '320'),
  ('FlBack', NOW() + INTERVAL '2 weeks', NOW() + INTERVAL '2 weeks 8 hours', 'VVO', 'VKO', 'Scheduled', '320');

-- Свяжем созданные рейсы с билетом
INSERT INTO ticket_flights (ticket_no, flight_id, fare_conditions, amount)
VALUES
  ('_NewTicket', (SELECT flight_id FROM flights WHERE flight_no = 'FlyOut'), 'Economy', 15000.00),
  ('_NewTicket', (SELECT flight_id FROM flights WHERE flight_no = 'FlBack'), 'Economy', 15000.00);
```
## Описание данных: отношения

№18. Авиакомпания начинает выдавать пассажирам карточки постоянных клиентов. Вместо того чтобы каждый раз вводить имя, номер документа и контактную информацию, постоянный клиент может указать номер своей карты, к которой привязана вся необходимая информация. При этом клиенту может предоставляться скидка.
Измените существующую схему данных так, чтобы иметь возможность хранить информацию о постоянных клиентах.
```sql
create table loyalty_cards
(
    card_number char(16) primary key not null,
    customer_id varchar(20) not null,
    discount numeric(5, 2) not null
);

comment on table loyalty_cards is 'Loyalty cards for frequent customers';

comment on column loyalty_cards.card_number is 'Card number';
comment on column loyalty_cards.customer_id is 'Customer ID';
comment on column loyalty_cards.discount is 'Discount rate';

-- Связываем с билетами
alter table tickets
add column loyalty_card_number char(16);

alter table tickets
add constraint tickets_loyalty_card_fkey
foreign key (loyalty_card_number) references loyalty_cards (card_number);

comment on column tickets.loyalty_card_number is 'Loyalty card number associated with the ticket';

-- Связываем с билетами к каждому рейсу
alter table ticket_flights
add column loyalty_card_number char(16);

alter table ticket_flights
add constraint ticket_flights_loyalty_card_fkey
foreign key (loyalty_card_number) references loyalty_cards (card_number);

comment on column ticket_flights.loyalty_card_number is 'Loyalty card number associated with the flight segment';
```

## Вложенные подзапросы
№20. Найдите модели самолетов «дальнего следования», максимальная продолжительность рейсов которых составила более 6 часов.
```sql
SELECT DISTINCT model, aircraft_code
FROM aircrafts_data
WHERE aircraft_code IN (
    SELECT DISTINCT f.aircraft_code
    FROM flights f
    WHERE EXTRACT(EPOCH FROM (f.scheduled_arrival - f.scheduled_departure)) / 3600 > 6
);
```

№21. Подсчитайте количество рейсов, которые хотя бы раз были задержаны более чем на 4 часа.
```sql
SELECT COUNT(*) AS delayed_flights_count
FROM (
    SELECT flight_id
    FROM flights
    WHERE actual_departure IS NOT NULL
      AND actual_arrival IS NOT NULL
      AND actual_arrival - actual_departure > interval '4 hours'
) AS delayed_flights;
```
## Псевдонимы для таблиц
№23. С целью оценки интенсивности работы обслуживающего персонала аэропорта Шереметьево (SVO) вычислите, сколько раз вылеты следовали друг за другом с перерывом менее пяти минут.
```sql
WITH FlightGaps AS (
    SELECT
        f1.flight_id AS flight1_id,
        f1.flight_no AS flight1_no,
        f1.scheduled_departure AS flight1_departure,
        f2.flight_id AS flight2_id,
        f2.flight_no AS flight2_no,
        f2.scheduled_departure AS flight2_departure
    FROM flights f1
    JOIN flights f2 ON f1.scheduled_departure < f2.scheduled_departure
        AND f1.arrival_airport = 'SVO'
        AND f2.arrival_airport = 'SVO'
)
SELECT COUNT(*) AS flights_following_each_other
FROM FlightGaps
WHERE flight2_departure - flight1_departure <= interval '5 minutes';
```
## Представления
№24. Количество рейсов, принятых конкретным аэропортом за каждый день, — довольно востребованный запрос. Напишите представление данного запроса для аэропорта города Барнаул (BAX).
```sql
CREATE VIEW flights_per_day_at_Barnaul AS
SELECT
    date_trunc('day', f.scheduled_arrival) AS flight_date,
    COUNT(*) AS flight_count
FROM flights f
WHERE f.arrival_airport = 'BAX'
GROUP BY flight_date
ORDER BY flight_date;

SELECT * FROM flights_per_day_at_Barnaul;
```
