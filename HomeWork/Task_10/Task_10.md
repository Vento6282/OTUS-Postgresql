# Виды и устройство репликации в PostgreSQL. Практика применения. 

Занятие от 20.05.2024

## Выполнение домашнего задания:

 - На 1 ВМ создаем таблицы test для записи, test2 для запросов на чтение.

create table test (number int);
create table test2 (number int);

 - Создаем публикацию таблицы test и подписываемся на публикацию таблицы test2 с ВМ №2.

На первой ВМ:\
alter system set wal_level = logical;\
SELECT pg_reload_conf();\
create publication test_pub for table test;\
create subscription test2_sub\
connection 'host=192.168.10.204 port=5432 user=postgres password=postgres dbname=postgres'\
publication test_pub with (copy_data = true);\
\password -- postgres

На второй ВМ\
\password -- postgres\
create table test (number int);\
create table test2 (number int);\


create subscription test2_sub
connection 'host=192.168.10.204 port=5432 user=postgres password=postgres dbname=postgres'
publication test_pub with (copy_data = true);

 - На 2 ВМ создаем таблицы test2 для записи, test для запросов на чтение.

create table test (number int);
create table test2 (number int);

 - Создаем публикацию таблицы test2 и подписываемся на публикацию таблицы test1 с ВМ №1.

create publication test_pub for table test2;


 - 3 ВМ использовать как реплику для чтения и бэкапов (подписаться на таблицы из ВМ №1 и №2 ).
 - Задание со * : реализовать горячее реплицирование для высокой доступности на 4ВМ. Источником должна выступать ВМ №3. Написать с какими проблемами столкнулись.



