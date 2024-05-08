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
***starting vacuum...end.***\
***progress: 6.0 s, 1005.7 tps, lat 7.892 ms stddev 4.913, 0 failed***\
***progress: 12.0 s, 1004.3 tps, lat 7.932 ms stddev 5.070, 0 failed***\
***progress: 18.0 s, 1026.0 tps, lat 7.769 ms stddev 5.170, 0 failed***\
***progress: 24.0 s, 1030.2 tps, lat 7.747 ms stddev 5.604, 0 failed***\
***progress: 30.0 s, 126.3 tps, lat 61.373 ms stddev 144.146, 0 failed***\
***progress: 36.0 s, 21.8 tps, lat 368.332 ms stddev 235.422, 0 failed***\
***progress: 42.0 s, 370.1 tps, lat 22.134 ms stddev 77.985, 0 failed***\
***progress: 48.0 s, 997.1 tps, lat 7.999 ms stddev 5.212, 0 failed***\
***progress: 54.0 s, 946.4 tps, lat 8.417 ms stddev 6.535, 0 failed***\
***progress: 60.0 s, 995.5 tps, lat 8.009 ms stddev 5.253, 0 failed***\
***transaction type: <builtin: TPC-B (sort of)>***\
***scaling factor: 1***\
***query mode: simple***\
***number of clients: 8***\
***number of threads: 1***\
***maximum number of tries: 1***\
***duration: 60 s***\
***number of transactions actually processed: 45148***\
***number of failed transactions: 0 (0.000%)***\
***latency average = 10.602 ms***\
***latency stddev = 35.547 ms***\
***initial connection time = 18.620 ms***\
***tps = 752.425258 (without initial connection time)***

 - Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла
 - Протестировать заново

Вывод команды:\
***starting vacuum...end.***
***progress: 6.0 s, 970.1 tps, lat 8.184 ms stddev 5.734, 0 failed***\
***progress: 12.0 s, 987.5 tps, lat 8.060 ms stddev 5.482, 0 failed***\
***progress: 18.0 s, 1001.1 tps, lat 7.961 ms stddev 5.892, 0 failed***\
***progress: 24.0 s, 1061.2 tps, lat 7.512 ms stddev 5.649, 0 failed***\
***progress: 30.0 s, 1007.2 tps, lat 7.914 ms stddev 5.933, 0 failed***\
***progress: 36.0 s, 1024.8 tps, lat 7.775 ms stddev 5.690, 0 failed***\
***progress: 42.0 s, 1010.2 tps, lat 7.894 ms stddev 5.818, 0 failed***\
***progress: 48.0 s, 1027.3 tps, lat 7.746 ms stddev 6.176, 0 failed***\
***progress: 54.0 s, 925.8 tps, lat 8.626 ms stddev 7.618, 0 failed***\
***progress: 60.0 s, 1018.8 tps, lat 7.828 ms stddev 6.128, 0 failed***\
***transaction type: <builtin: TPC-B (sort of)>***\
***scaling factor: 1***\
***query mode: simple***\
***number of clients: 8***\
***number of threads: 1***\
***maximum number of tries: 1***\
***duration: 60 s***\
***number of transactions actually processed: 60213***\
***number of failed transactions: 0 (0.000%)***\
***latency average = 7.941 ms***\
***latency stddev = 6.031 ms***\
***initial connection time = 19.528 ms***\
***tps = 1003.577565 (without initial connection time)***

 - Что изменилось и почему?
 - Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1млн строк
 - Посмотреть размер файла с таблицей
 - 5 раз обновить все строчки и добавить к каждой строчке любой символ
 - Посмотреть количество мертвых строчек в таблице и когда последний раз приходил автовакуум
 - Подождать некоторое время, проверяя, пришел ли автовакуум
 - 5 раз обновить все строчки и добавить к каждой строчке любой символ
 - Посмотреть размер файла с таблицей
 - Отключить Автовакуум на конкретной таблице
 - 10 раз обновить все строчки и добавить к каждой строчке любой символ
 - Посмотреть размер файла с таблицей
 - Объясните полученный результат
 - Не забудьте включить автовакуум)

Задание со *:

 - Написать анонимную процедуру, в которой в цикле 10 раз обновятся все строчки в искомой таблице.
 - Не забыть вывести номер шага цикла.