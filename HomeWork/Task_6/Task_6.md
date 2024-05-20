# Журналы. 

Занятие от 2.05.2024

## Выполнение домашнего задания:

 - Настройте выполнение контрольной точки раз в 30 секунд.
 - 10 минут c помощью утилиты pgbench подавайте нагрузку.

Команда:
```
pgbench -U postgres -T 600 postgres
```

 - Измерьте, какой объем журнальных файлов был сгенерирован за это время. Оцените, какой объем приходится в среднем на одну контрольную точку.

Ответ: было сгенерировано 510524168 байтов журнальных файлов. На одну контрольную точку в среднем приходилось по 27КБ.

Запрос и  запроса:
```
SELECT '0/20926640'::pg_lsn - '0/2246B38'::pg_lsn;
```

Вывод результата:
```
 ?column?
-----------
 510524168
```

 - Проверьте данные статистики: все ли контрольные точки выполнялись точно по расписанию. Почему так произошло?

Ответ: точки выполнялись по расписанию.

Вывод результата запроса - ***SELECT * FROM pg_stat_bgwriter \gx***:
```
checkpoints_timed     | 121
checkpoints_req       | 5
checkpoint_write_time | 571840
checkpoint_sync_time  | 4579
buffers_checkpoint    | 43886
buffers_clean         | 0
maxwritten_clean      | 0
buffers_backend       | 5150
buffers_backend_fsync | 0
buffers_alloc         | 6093
stats_reset           | 2024-05-13 17:50:55.44739+03
```

Вывод файла ***/var/log/postgresql/postgresql-15-main.log***:
```
2024-05-14 10:21:46.854 MSK [1508] LOG:  checkpoint starting: time
2024-05-14 10:22:16.305 MSK [1508] LOG:  checkpoint starting: time
2024-05-14 10:22:46.136 MSK [1508] LOG:  checkpoint starting: time
2024-05-14 10:23:16.061 MSK [1508] LOG:  checkpoint starting: time
2024-05-14 10:23:46.073 MSK [1508] LOG:  checkpoint starting: time
2024-05-14 10:24:16.521 MSK [1508] LOG:  checkpoint starting: time
2024-05-14 10:24:46.033 MSK [1508] LOG:  checkpoint starting: time
2024-05-14 10:25:16.041 MSK [1508] LOG:  checkpoint starting: time
2024-05-14 10:25:46.069 MSK [1508] LOG:  checkpoint starting: time
2024-05-14 10:26:16.308 MSK [1508] LOG:  checkpoint starting: time
2024-05-14 10:26:46.121 MSK [1508] LOG:  checkpoint starting: time
2024-05-14 10:27:16.061 MSK [1508] LOG:  checkpoint starting: time
2024-05-14 10:27:46.053 MSK [1508] LOG:  checkpoint starting: time
2024-05-14 10:28:16.068 MSK [1508] LOG:  checkpoint starting: time
2024-05-14 10:28:46.073 MSK [1508] LOG:  checkpoint starting: time
2024-05-14 10:29:16.025 MSK [1508] LOG:  checkpoint starting: time
2024-05-14 10:29:46.581 MSK [1508] LOG:  checkpoint starting: time
2024-05-14 10:30:16.109 MSK [1508] LOG:  checkpoint starting: time
2024-05-14 10:30:46.113 MSK [1508] LOG:  checkpoint starting: time
2024-05-14 10:31:16.033 MSK [1508] LOG:  checkpoint starting: time
2024-05-14 10:31:46.762 MSK [1508] LOG:  checkpoint starting: time
```

 - Сравните tps в синхронном/асинхронном режиме утилитой pgbench. Объясните полученный результат.

Ответ: в синхронном режиме tps = 883.302233, в асинхронном tps = 3467.357161. Количество запросов в секунду в асинхронном режиме больше, так как сервер не ожидает, пока записи WAL будут сброшены на дис. Асинхронный режим позволяет транзакциям выполняться быстрее, но за счет того, что самые последние транзакции могут быть потеряны в случае сбоя базы данных. 

 - Создайте новый кластер с включенной контрольной суммой страниц. Создайте таблицу. Вставьте несколько значений. Выключите кластер. Измените пару байт в таблице. Включите кластер и сделайте выборку из таблицы. Что и почему произошло? как проигнорировать ошибку и продолжить работу?

Команда линукса для включения контрольных сумм:
```
su - postgres -c '/usr/lib/postgresql/15/bin/pg_checksums --enable -D "/var/lib/postgresql/15/main"'
```

Проверка включения контрольных сумм:
```
postgres=# show data_checksums;
 data_checksums 
----------------
 on
```

Запрос создания и заполнения таблицы:
```
create table humans as select generate_series(1, 10) as id, md5(random()::text)::char(10) as fio;
```

Запрос чтобы определить, где физически расположена таблица:
```
select pg_relation_filepath('humans');
 pg_relation_filepath 
----------------------
 base/5/16388
```

Вывод ошибки, при попытке сделать выборку:
```
select * from humans;
WARNING:  page verification failed, calculated checksum 63334 but expected 341
ERROR:  invalid page in block 0 of relation base/5/16388
```

Ответ: Контрольные суммы страниц — функция, помогающая проверить целостность данных, хранящихся на диске. То есть контрольные суммы определяют, были испорчены данные на физически или нет. Чтобы попробовать игнорировать данную ошибку, необходимо включить ignore_checksum_failure. Будут прочитаны те данные, которые возможно. Если будет изменён заголовок (как я сначала и сделал), то данные не удастся прочитать даже с включенным параметром ignore_checksum_failure.