# Блокировки. 

Занятие от 6.05.2024

## Выполнение домашнего задания:

 - Настройте сервер так, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 миллисекунд. Воспроизведите ситуацию, при которой в журнале появятся такие сообщения.

Запрос включения вывода сообщений в журнал сервера:
```
alter system set log_lock_waits = on;
```
Запрос установки вывода сообщений об ожидании дольше 200ms:
```
 alter system set deadlock_timeout = 200;
```
Вывод файла ***/var/log/postgresql/postgresql-15-main.log***:
```
2024-05-14 17:35:18.990 MSK [4049] postgres@accounts LOG:  process 4049 still waiting for ShareLock on transaction 2611240 after 200.217 ms

2024-05-14 17:35:43.221 MSK [4049] postgres@accounts LOG:  process 4049 acquired ShareLock on transaction 2611240 after 24431.224 ms
```

 - Смоделируйте ситуацию обновления одной и той же строки тремя командами UPDATE в разных сеансах. Изучите возникшие блокировки в представлении pg_locks и убедитесь, что все они понятны. Пришлите список блокировок и объясните, что значит каждая.

```
   locktype    | database | relation | page | tuple | virtualxid | transactionid | classid | objid | objsubid | virtualtransaction | pid  |       mode       | granted | fastpath |           waitstart
---------------+----------+----------+------+-------+------------+---------------+---------+-------+----------+--------------------+------+------------------+---------+----------+-------------------------------
 relation      |    16468 |    16474 |      |       |            |               |         |       |          | 6/2                | 4086 | RowExclusiveLock | t       | t        |
 relation      |    16468 |    16469 |      |       |            |               |         |       |          | 6/2                | 4086 | RowExclusiveLock | t       | t        |
 virtualxid    |          |          |      |       | 6/2        |               |         |       |          | 6/2                | 4086 | ExclusiveLock    | t       | t        |
 relation      |    16468 |    16474 |      |       |            |               |         |       |          | 5/10               | 4044 | RowExclusiveLock | t       | t        |
 relation      |    16468 |    16469 |      |       |            |               |         |       |          | 5/10               | 4044 | RowExclusiveLock | t       | t        |
 virtualxid    |          |          |      |       | 5/10       |               |         |       |          | 5/10               | 4044 | ExclusiveLock    | t       | t        |
 relation      |        5 |    12073 |      |       |            |               |         |       |          | 4/66               | 4147 | AccessShareLock  | t       | t        |
 virtualxid    |          |          |      |       | 4/66       |               |         |       |          | 4/66               | 4147 | ExclusiveLock    | t       | t        |
 relation      |    16468 |    16474 |      |       |            |               |         |       |          | 3/14               | 4049 | RowExclusiveLock | t       | t        |
 relation      |    16468 |    16469 |      |       |            |               |         |       |          | 3/14               | 4049 | RowExclusiveLock | t       | t        |
 virtualxid    |          |          |      |       | 3/14       |               |         |       |          | 3/14               | 4049 | ExclusiveLock    | t       | t        |
 transactionid |          |          |      |       |            |       2611242 |         |       |          | 5/10               | 4044 | ExclusiveLock    | t       | f        |
 tuple         |    16468 |    16469 |    0 |    10 |            |               |         |       |          | 3/14               | 4049 | ExclusiveLock    | t       | f        |
 transactionid |          |          |      |       |            |       2611244 |         |       |          | 6/2                | 4086 | ExclusiveLock    | t       | f        |
 tuple         |    16468 |    16469 |    0 |    10 |            |               |         |       |          | 6/2                | 4086 | ExclusiveLock    | f       | f        | 2024-05-14 17:45:19.188518+03
 transactionid |          |          |      |       |            |       2611242 |         |       |          | 3/14               | 4049 | ShareLock        | f       | f        | 2024-05-14 17:45:09.756249+03
 transactionid |          |          |      |       |            |       2611243 |         |       |          | 3/14               | 4049 | ExclusiveLock    | t       | f        |
```


 - Воспроизведите взаимоблокировку трех транзакций. Можно ли разобраться в ситуации постфактум, изучая журнал сообщений?
 - Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга?
 - Задание со звездочкой*\
 Попробуйте воспроизвести такую ситуацию.