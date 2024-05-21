# MVCC, vacuum и autovacuum. 

Занятие от 25.04.2024

## Выполнение домашнего задания:

 - Создать инстанс ВМ с 2 ядрами и 4 Гб ОЗУ и SSD 10GB
 - Установить на него PostgreSQL 15 с дефолтными настройками
 - Создать БД для тестов: выполнить pgbench -i postgres

Вывод команды:\
***dropping old tables...***
***NOTICE:  table "pgbench_accounts" does not exist, skipping***\
***NOTICE:  table "pgbench_branches" does not exist, skipping***\
***NOTICE:  table "pgbench_history" does not exist, skipping***\
***NOTICE:  table "pgbench_tellers" does not exist, skipping***\
***creating tables...***\
***generating data (client-side)...***\
***100000 of 100000 tuples (100%) done (elapsed 0.07 s, remaining 0.00 s)***\
***vacuuming...***\
***creating primary keys...***\
***done in 0.34 s (drop tables 0.00 s, create tables 0.01 s, client-side generate 0.24 s, vacuum 0.04 s, primary keys 0.05 s).***

 - Запустить pgbench -c8 -P 6 -T 60 -U postgres postgres

Вывод команды:\
***pgbench (15.7 (Ubuntu 15.7-1.pgdg22.04+1))***\
***starting vacuum...end.***\
***progress: 6.0 s, 749.3 tps, lat 10.404 ms stddev 2.786, 0 failed***\
***progress: 12.0 s, 755.2 tps, lat 10.403 ms stddev 2.659, 0 failed***\
***progress: 18.0 s, 752.5 tps, lat 10.443 ms stddev 2.670, 0 failed***\
***progress: 24.0 s, 752.5 tps, lat 10.440 ms stddev 2.720, 0 failed***\
***progress: 30.0 s, 747.6 tps, lat 10.503 ms stddev 2.604, 0 failed***\
***progress: 36.0 s, 755.0 tps, lat 10.409 ms stddev 2.692, 0 failed***\
***progress: 42.0 s, 755.1 tps, lat 10.406 ms stddev 2.607, 0 failed***\
***progress: 48.0 s, 755.7 tps, lat 10.399 ms stddev 2.618, 0 failed***\
***progress: 54.0 s, 754.3 tps, lat 10.417 ms stddev 2.736, 0 failed***\
***progress: 60.0 s, 739.8 tps, lat 10.618 ms stddev 3.314, 0 failed***\
***transaction type: <builtin: TPC-B (sort of)>***\
***scaling factor: 1***\
***query mode: simple***\
***number of clients: 8***\
***number of threads: 1***\
***maximum number of tries: 1***\
***duration: 60 s***\
***number of transactions actually processed: 45110***\
***number of failed transactions: 0 (0.000%)***\
***latency average = 10.447 ms***\
***latency stddev = 2.767 ms***\
***initial connection time = 39.896 ms***\
***tps = 751.667941 (without initial connection time)***

 - Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла
 - Протестировать заново

Вывод команды:\
***pgbench (15.7 (Ubuntu 15.7-1.pgdg22.04+1))***\
***starting vacuum...end.***\
***progress: 6.0 s, 717.1 tps, lat 10.873 ms stddev 2.673, 0 failed***\
***progress: 12.0 s, 747.9 tps, lat 10.505 ms stddev 2.539, 0 failed***\
***progress: 18.0 s, 750.4 tps, lat 10.467 ms stddev 2.509, 0 failed***\
***progress: 24.0 s, 748.6 tps, lat 10.495 ms stddev 2.510, 0 failed***\
***progress: 30.0 s, 751.1 tps, lat 10.465 ms stddev 2.490, 0 failed***\
***progress: 36.0 s, 750.3 tps, lat 10.470 ms stddev 2.494, 0 failed***\
***progress: 42.0 s, 749.5 tps, lat 10.481 ms stddev 2.589, 0 failed***\
***progress: 48.0 s, 751.3 tps, lat 10.454 ms stddev 2.507, 0 failed***\
***progress: 54.0 s, 714.6 tps, lat 10.988 ms stddev 3.628, 0 failed***\
***progress: 60.0 s, 722.5 tps, lat 10.873 ms stddev 3.200, 0 failed***\
***transaction type: <builtin: TPC-B (sort of)>***\
***scaling factor: 1***\
***query mode: simple***\
***number of clients: 8***\
***number of threads: 1***\
***maximum number of tries: 1***\
***duration: 60 s***\
***number of transactions actually processed: 44428***\
***number of failed transactions: 0 (0.000%)***\
***latency average = 10.607 ms***\
***latency stddev = 2.762 ms***\
***initial connection time = 41.043 ms***\
***tps = 740.355120 (without initial connection time)***

 - Что изменилось и почему?

Ответ: в моём случае заметных изменений нет, но судя по изменяемым настройкам предположу, что должно измениться среднее время задержки (*average latency*). Затрагиваемые настройки в общей массе уменьшают работу с диском за счёт увеличение объема памяти, что должно повысить скорость выполнения запросов.

 - Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1млн строк
 - Посмотреть размер файла с таблицей

Вывод команды:\
***pg_size_pretty***\
***----------------***\
***35 MB***

 - 5 раз обновить все строчки и добавить к каждой строчке любой символ
 - Посмотреть количество мертвых строчек в таблице и когда последний раз приходил автовакуум

Вывод команды:\
***relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum***\
***---------+------------+------------+--------+-------------------------------***\
***t1      |    1000000 |    4999852 |    499 | 2024-05-11 13:57:44.840609+03***

 - Подождать некоторое время, проверяя, пришел ли автовакуум

Вывод команды:\
 ***relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum***\
***---------+------------+------------+--------+-------------------------------***\
 ***t1      |    1000000 |          0 |      0 | 2024-05-11 14:00:45.814508+03***

 - 5 раз обновить все строчки и добавить к каждой строчке любой символ
 - Посмотреть размер файла с таблицей

***pg_size_pretty***\
***----------------***\
***261 MB***

 - Отключить Автовакуум на конкретной таблице

Запрос:\
***alter table t1 set (autovacuum_enabled = false);***

 - 10 раз обновить все строчки и добавить к каждой строчке любой символ
 - Посмотреть размер файла с таблицей

Вывод команды:\
***pg_size_pretty***\
***----------------***\
***570 MB***

 - Объясните полученный результат

Ответ: autovacuum удаляет мертвые строки, но не уменьшает размер таблицы. Зарезервированное место таблицей будет повторно использоваться таблицей. Поэтому когда обновили таблицу 5 раз, то занимаемое место таблицей было 261МВ, и когда затем обновили таблицу ещё 10 раз, то стала 570МВ, а не 261МВ + 570МВ.

 - Не забудьте включить автовакуум)

Задание со *:

 - Написать анонимную процедуру, в которой в цикле 10 раз обновятся все строчки в искомой таблице.

Текст запроса:
```
DO
$do$
BEGIN
   FOR i IN 1..10 LOOP
      update t1
      set c1 = concat(c1, 'f');
      raise notice 'STEP: %', i;
   END LOOP;
END
$do$;
```
 - Не забыть вывести номер шага цикла.

Вывод команды:
```
NOTICE:  STEP: 1
NOTICE:  STEP: 2
NOTICE:  STEP: 3
NOTICE:  STEP: 4
NOTICE:  STEP: 5
DO
```