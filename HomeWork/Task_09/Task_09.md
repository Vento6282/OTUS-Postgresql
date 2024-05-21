# Журналы. 

Занятие от 16.05.2024

## Выполнение домашнего задания:

 - Создаем ВМ/докер c ПГ.
 - Создаем БД, схему и в ней таблицу.
 - Заполним таблицы автосгенерированными 100 записями.

Текст запросов:
```
CREATE DATABASE
postgres=# \c otus
otus=# create schema humans;
otus=# create table humans.students as select generate_series(1, 100) as id, md5(random()::text)::char(10) as fio;
```
 - Под линукс пользователем Postgres создадим каталог для бэкапов
 
Команды линукс:
```
sudo mkdir /tmp/backups
```
 - Сделаем логический бэкап используя утилиту COPY

Текст запроса:
```
\copy humans.students to '/tmp/backups/backup_copy_students.sql';
```
 - Восстановим в 2 таблицу данные из бэкапа.

Текст запросов:
```
create table humans.students_copy (id int, fio char(10));
\copy humans.students_copy from '/tmp/backups/backup_copy_students.sql';
```
 - Используя утилиту pg_dump создадим бэкап в кастомном сжатом формате двух таблиц

Команда линукс:
```
pg_dump -Fc otus > /tmp/backups/backup_students_dump.gz
```
 - Используя утилиту pg_restore восстановим в новую БД только вторую таблицу!

Текст запросов:
```
create database otus_copy;
\c otus_copy
create schema humans;
```

Команда линукс:
```
pg_restore -d otus_copy -t students_copy /tmp/backups/backup_students_dump.gz
```