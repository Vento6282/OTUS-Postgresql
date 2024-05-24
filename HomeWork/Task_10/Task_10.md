# Виды и устройство репликации в PostgreSQL. Практика применения. 

Занятие от 20.05.2024

## Выполнение домашнего задания:

 - На 1 ВМ создаем таблицы test для записи, test2 для запросов на чтение.

Текст запросов:
```
create table test (number int);
create table test2 (number int);
```

 - Создаем публикацию таблицы test и подписываемся на публикацию таблицы test2 с ВМ №2.

Текст запросов:
```
alter system set wal_level = logical;
SELECT pg_reload_conf();
create publication test_pub for table test;
\password
create subscription test_sub
connection 'host=192.168.10.204 port=5432 user=postgres password=postgres dbname=postgres'
publication test_pub with (copy_data = true);
```

 - На 2 ВМ создаем таблицы test2 для записи, test для запросов на чтение.

Текст запросов:
```
create table test (number int);
create table test2 (number int);
```
 - Создаем публикацию таблицы test2 и подписываемся на публикацию таблицы test с ВМ №1.

Текст запросов:
```
create publication test_pub for table test2;
\password
create subscription test_sub\
connection 'host=192.168.10.195 port=5432 user=postgres password=postgres dbname=postgres'\
publication test_pub with (copy_data = true);
```

Комментарий: на первой ВМ пришлось пересоздать подписку. Если сначала создать подписку на таблицу на ВМ1, и только потом создать эту таблицу на ВМ2, на которую была создана подписка, то таблица с ВМ1 не видит изменений в таблице на ВМ2.

 - 3 ВМ использовать как реплику для чтения и бэкапов (подписаться на таблицы из ВМ №1 и №2 ).

Текст запросов:
```
create table test (number int);
create table test2 (number int);
create subscription test_sub
connection 'host=192.168.10.195 port=5432 user=postgres password=postgres dbname=postgres'
publication test_pub with (copy_data = true);
create subscription test_sub2
connection 'host=192.168.10.204 port=5432 user=postgres password=postgres dbname=postgres'
publication test_pub with (copy_data = true);
```

 - Задание со * : реализовать горячее реплицирование для высокой доступности на 4ВМ. Источником должна выступать ВМ №3. Написать с какими проблемами столкнулись.

Добавил параментры в файл ***/etc/postgresql/15/main/postgresql.conf*** на ВМ3:
```
archive_mode = on
archive_command = 'cp /var/replication/%f'
wal_level = hot_standby
max_wal_senders = 8
hot_standby = on
```

Выключаем кластер на ВМ4 и очищаем директорию ***/var/lib/postgresql/15/main/***.

Восстанавливаем кластер на ВМ4 и включаем его:
```
sudo pg_basebackup -R -h 192.168.10.207 -U postgres -D /var/lib/postgresql/15/main
```

Комментарий: основной проблемой было настроить видимость кластеров друг друга.

