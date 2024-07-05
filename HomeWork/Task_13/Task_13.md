# Секционирование.

Занятие от 24.06.2024

## Выполнение домашнего задания:

 - Секционировать большую таблицу из демо базы flights.

### Установка демонстрационной базы.

Скачиваем архив демонстрационной базы:
```
wget https://edu.postgrespro.ru/demo-big.zip
```
Распаковываем архив:
```
unzip demo-small.zip
```
В postgres запускаем скрипт:
```
\i demo-small-20170815.sql
```
### Секционирование большой таблицы.

С помощью запроса определим самые большие таблицы в базе:
```
SELECT tablename, pg_size_pretty(pg_total_relation_size(CAST(tablename as text))) as size
from pg_tables
where schemaname = 'bookings'
```
Самой большой таблицей является boarding_passes:
```
        tablename        |  size
-------------------------+---------
 boarding_passes         | 1102 MB
 seats                   | 144 kB
 bookings                | 150 MB
 boarding_passes_part_08 | 151 MB
 boarding_passes_part_02 | 151 MB
 aircrafts_data          | 32 kB
 flights                 | 32 MB
 tickets                 | 475 MB
 airports_data           | 72 kB
 ticket_flights          | 871 MB
 ```
Секционирование выбрал по hash по полю flight_id, чтобы информацию по одному полёту попадало в одну секцию.\
Создаём секционированную по hash таблицу bookings.boarding_passes_part:
 ```
CREATE TABLE bookings.boarding_passes_part (
	ticket_no bpchar(13) NOT NULL,
	flight_id int4 NOT NULL,
	boarding_no int4 NOT NULL,
	seat_no varchar(4) NOT NULL,
	CONSTRAINT boarding_passes_part_flight_id_boarding_no_key UNIQUE (flight_id, boarding_no),
	CONSTRAINT boarding_passes_part_flight_id_seat_no_key UNIQUE (flight_id, seat_no),
	CONSTRAINT boarding_passes_part_pkey PRIMARY KEY (ticket_no, flight_id)
) partition by hash(flight_id);

ALTER TABLE bookings.boarding_passes_part ADD CONSTRAINT boarding_passes_part_ticket_no_fkey FOREIGN KEY (ticket_no,flight_id) REFERENCES bookings.ticket_flights(ticket_no,flight_id);
 ```
 Создаём секции для секционированной таблицы:
 ```
 CREATE TABLE bookings.boarding_passes_part_01 PARTITION OF bookings.boarding_passes_part
    FOR VALUES WITH (MODULUS 10, REMAINDER 0);

CREATE TABLE bookings.boarding_passes_part_02 PARTITION OF bookings.boarding_passes_part
    FOR VALUES WITH (MODULUS 10, REMAINDER 1);

CREATE TABLE bookings.boarding_passes_part_03 PARTITION OF bookings.boarding_passes_part
    FOR VALUES WITH (MODULUS 10, REMAINDER 2);

CREATE TABLE bookings.boarding_passes_part_04 PARTITION OF bookings.boarding_passes_part
    FOR VALUES WITH (MODULUS 10, REMAINDER 3);

CREATE TABLE bookings.boarding_passes_part_05 PARTITION OF bookings.boarding_passes_part
    FOR VALUES WITH (MODULUS 10, REMAINDER 4);

CREATE TABLE bookings.boarding_passes_part_06 PARTITION OF bookings.boarding_passes_part
    FOR VALUES WITH (MODULUS 10, REMAINDER 5);

CREATE TABLE bookings.boarding_passes_part_07 PARTITION OF bookings.boarding_passes_part
    FOR VALUES WITH (MODULUS 10, REMAINDER 6);
   
CREATE TABLE bookings.boarding_passes_part_08 PARTITION OF bookings.boarding_passes_part
    FOR VALUES WITH (MODULUS 10, REMAINDER 7);

CREATE TABLE bookings.boarding_passes_part_09 PARTITION OF bookings.boarding_passes_part
    FOR VALUES WITH (MODULUS 10, REMAINDER 8);

CREATE TABLE bookings.boarding_passes_part_10 PARTITION OF bookings.boarding_passes_part
    FOR VALUES WITH (MODULUS 10, REMAINDER 9);
 ```
Заполняем секционированную таблицу bookings.boarding_passes_part данными из таблицы bookings.boarding_passes:
```
INSERT INTO bookings.boarding_passes_part (ticket_no, flight_id, boarding_no, seat_no)
SELECT * FROM bookings.boarding_passes;
```
Проверяем размеры заполненных секций и самой таблицы:
```
        tablename        |  size
-------------------------+---------
 boarding_passes_part    | 0 bytes
 boarding_passes_part_01 | 149 MB
 boarding_passes_part_02 | 151 MB
 boarding_passes_part_03 | 148 MB
 boarding_passes_part_04 | 150 MB
 boarding_passes_part_05 | 150 MB
 boarding_passes_part_06 | 147 MB
 boarding_passes_part_07 | 149 MB
 boarding_passes_part_08 | 151 MB
 boarding_passes_part_09 | 147 MB
 boarding_passes_part_10 | 148 MB
```