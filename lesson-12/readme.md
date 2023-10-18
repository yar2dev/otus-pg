# Домашка "Настройка PostgreSQL "

1. развернуть виртуальную машину любым удобным способом
```
yandex cloud
VM: 8CPU/32RAM/SSD
```
2.  поставить на неё PostgreSQL 15 любым способом
```
sudo apt update && sudo apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y install postgresql-15
```
3. настроить кластер PostgreSQL 15 на максимальную производительность не
обращая внимание на возможные проблемы с надежностью в случае
аварийной перезагрузки виртуальной машины
```
shared_buffers = '8GB'
effective_cache_size = '24GB'
maintenance_work_mem = '2GB'
checkpoint_completion_target = '0.9'
wal_buffers = '16MB'
default_statistics_target = '100'
random_page_cost = '1.1'
effective_io_concurrency = '200'
work_mem = '32 MB'
huge_pages = 'try'
min_wal_size = '1GB'
max_wal_size = '4GB'
max_worker_processes = '8'
max_parallel_workers_per_gather = '4'
max_parallel_workers = '8'
max_parallel_maintenance_workers = '4'
max_wal_senders = '0'
wal_level = 'minimal'
synchronous_commit = 'off'
```
4. нагрузить кластер через утилиту через утилиту pgbench (https://postgrespro.ru/docs/postgrespro/14/pgbench)
```
pgbench -c 50 -j 2 -P 10 -T 60 
```
5. написать какого значения tps удалось достичь, показать какие параметры в
какие значения устанавливали и почему
-  с дефолтными настройками

```
pgbench (15.4 (Ubuntu 15.4-2.pgdg22.04+1))
starting vacuum...end.
progress: 10.0 s, 1496.4 tps, lat 33.167 ms stddev 39.354, 0 failed
progress: 20.0 s, 1255.2 tps, lat 39.840 ms stddev 48.325, 0 failed
progress: 30.0 s, 1530.3 tps, lat 32.663 ms stddev 35.549, 0 failed
progress: 40.0 s, 1489.2 tps, lat 33.582 ms stddev 38.622, 0 failed
progress: 50.0 s, 1492.8 tps, lat 33.490 ms stddev 41.783, 0 failed
progress: 60.0 s, 1655.0 tps, lat 30.209 ms stddev 32.585, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 50
number of threads: 2
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 89239
number of failed transactions: 0 (0.000%)
latency average = 33.603 ms
latency stddev = 39.383 ms
initial connection time = 44.530 ms
tps = 1487.430406 (without initial connection time)
```
- с оптимизированными настройками
```
pgbench (15.4 (Ubuntu 15.4-2.pgdg22.04+1))
starting vacuum...end.
progress: 10.0 s, 4814.3 tps, lat 10.324 ms stddev 13.208, 0 failed
progress: 20.0 s, 4849.6 tps, lat 10.311 ms stddev 13.619, 0 failed
progress: 30.0 s, 4858.8 tps, lat 10.290 ms stddev 13.641, 0 failed
progress: 40.0 s, 4877.6 tps, lat 10.247 ms stddev 12.933, 0 failed
progress: 50.0 s, 4810.8 tps, lat 10.389 ms stddev 13.661, 0 failed
progress: 60.0 s, 4860.6 tps, lat 10.286 ms stddev 13.201, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 50
number of threads: 2
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 290766
number of failed transactions: 0 (0.000%)
latency average = 10.311 ms
latency stddev = 13.384 ms
initial connection time = 44.742 ms
tps = 4847.164357 (without initial connection time)
```
Основные параметры 
- synchronous_commit: Уровень синхронизации фиксации транзакции. Отключена гарантированная фиксация транзакции
- random_page_cost: затраты на произвольное чтение страницы из диска по сравнению с последовательным чтением. Установлен близко к 1 (значение последовательного чтения), так как диск SDD
- work_mem: максимальное количество памяти, которое может использоваться для выполнения операций сортировки и хеширования.
- shared_buffers: количество памяти для кэширования данных.
- effective_cache_size: размер всего кэша
- effective_io_concurrency: макс количество одновременных запросов к диску, которые могут выполняться параллельно.


sysbench-tpcc из задания в основной ветке не рабочий, есть исправления в pull request https://github.com/Percona-Lab/sysbench-tpcc/pull/40/files взял оттуда,  но очень долго заполнялась таблица - не дождался.
