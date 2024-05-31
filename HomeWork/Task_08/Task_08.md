# Настройка PostgreSQL. 

Занятие от 13.05.2024

## Выполнение домашнего задания:

 - развернуть виртуальную машину любым удобным способом
 - поставить на неё PostgreSQL 15 любым способом
 - настроить кластер PostgreSQL 15 на максимальную производительность не обращая внимание на возможные проблемы с надежностью в случае аварийной перезагрузки виртуальной машины
 - нагрузить кластер через утилиту через утилиту pgbench (https://postgrespro.ru/docs/postgrespro/14/pgbench)
 - написать какого значения tps удалось достичь, показать какие параметры в какие значения устанавливали и почему
 - Задание со *: аналогично протестировать через утилиту https://github.com/Percona-Lab/sysbench-tpcc (требует установки
https://github.com/akopytov/sysbench)



https://pgconfigurator.cybertec.at/

Параметры ВМ: 2 CPU, 4 RAM, SSD.

Для тестирования использовал команду с параметрами - ***pgbench -c 10 -T 300 -U postgres postgres***

Изменяемые параметры:

При значениях по умолчанию - tps = 997.551954

После внесения изменений - tps = 2821.606357

***shared_buffers = 1Gb***\
Параметр **shared_buffers** определяет , сколько памяти выделено серверу для кэширования данных. Рекомендуемое значение в пределах от 15% до 25% от общего объема оперативной памяти компьютера.

***work_mem = 32MB***\
Параметр **work_mem** в основном определяет объем памяти, который будет использоваться внутренними операциями сортировки и хеш-таблицами перед записью во временные файлы на диске. Операции сортировки используются для операций упорядочивания, разделения и объединения слиянием. Хэш-таблицы используются в хэш-соединениях и агрегации на основе хеша. Установка правильного значения параметра **work_mem** может привести к меньшему количеству подкачки дисков и, следовательно, к более быстрому выполнению запросов. 

***maintenance_work_mem = 320MB***\
Устанавливает базовый максимальный объем памяти, который будет использоваться операцией запроса (например, сортировкой или хэш-таблицей) перед записью во временные файлы на диске.

***effective_cache_size = 3GB***\
Этот параметр позволяет PostgreSQL лучше использовать доступную память, что уменьшает количество необходимых операций чтения с диска и повышает производительность запросов.


***effective_io_concurrency = 100***\
Задаёт допустимое число параллельных операций ввода/вывода, которое говорит PostgreSQL о том, сколько операций ввода/вывода могут быть выполнены одновременно. Чем больше это число, тем больше операций ввода/вывода будет пытаться выполнить параллельно PostgreSQL в отдельном сеансе. 

***random_page_cost = 1.25***\
Данный параметр влияет на план запроса. Значение random_page_cost по умолчанию - 4.0. Уменьшение параметра приведёт к тому, что при выполении запроса будет сканироваться не таблица, а подходящий индекс.

# Checkpointing:
checkpoint_timeout = '15 min'
checkpoint_completion_target = 0.9
max_wal_size = '1024 MB'
min_wal_size = '512 MB'


# WAL writing
wal_compression = on
wal_buffers = -1 # auto-tuned by Postgres till maximum of segment size (16MB by default)


# Background writer
bgwriter_delay = 200ms
bgwriter_lru_maxpages = 100
bgwriter_lru_multiplier = 2.0
bgwriter_flush_after = 0

# Parallel queries:
max_worker_processes = 2
max_parallel_workers_per_gather = 1
max_parallel_maintenance_workers = 1
max_parallel_workers = 2
parallel_leader_participation = on

# Advanced features
enable_partitionwise_join = on
enable_partitionwise_aggregate = on
jit = on
max_slot_wal_keep_size = '1000 MB'
track_wal_io_timing = on
maintenance_io_concurrency = 100
wal_recycle = on

***synchronous_commit = off***\