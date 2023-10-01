# Домашка "Журналы"
1) Настройте выполнение контрольной точки раз в 30 секунд.
```sql
ALTER SYSTEM SET checkpoint_timeout = '30s';
```
```sql
SELECT pg_reload_conf();
```
Сброс статистики
```
SELECT pg_stat_reset_shared('bgwriter');
```
2) 10 минут c помощью утилиты pgbench подавайте нагрузку.
```
pgbench -c8 -P 6 -T 600 -U postgres postgres
```
3) Измерьте, какой объем журнальных файлов был сгенерирован за это время. 
```
SELECT * FROM pg_stat_bgwriter \gx
-[ RECORD 1 ]---------+------------------------------
checkpoints_timed     | 20
checkpoints_req       | 0
checkpoint_write_time | 510744
checkpoint_sync_time  | 40
buffers_checkpoint    | 36573
buffers_clean         | 0
maxwritten_clean      | 0
buffers_backend       | 2360
buffers_backend_fsync | 0
buffers_alloc         | 4505
stats_reset           | 2023-10-01 20:04:28.576928+00
```

```
SELECT buffers_backend * current_setting('block_size')::numeric AS bytes_written
FROM pg_stat_bgwriter;

 bytes_written 
---------------
      19791872
(1 row)
18 мб
```
Оцените, какой объем приходится в среднем на одну контрольную точку.
```
SELECT (buffers_backend * current_setting('block_size')::numeric) / checkpoints_timed AS avg_bytes_per_checkpoint
FROM pg_stat_bgwriter;
 avg_bytes_per_checkpoint 
--------------------------
      899630.545454545455
(1 row)
878 килобайт
```
4) Проверьте данные статистики: все ли контрольные точки выполнялись точно по расписанию. Почему так произошло?
```
по расписанию
max_wal_size не достигался
```
5) Сравните tps в синхронном/асинхронном режиме утилитой pgbench. Объясните полученный результат.
```
ALTER SYSTEM SET synchronous_commit = on;
SELECT pg_reload_conf();
pgbench -c8 -P 6 -T 60 -U postgres postgres

tps = 624.245232 
```
```
ALTER SYSTEM SET synchronous_commit = off;
SELECT pg_reload_conf();
pgbench -c8 -P 6 -T 60 -U postgres postgres

tps = 1017.188432
```
В синхронном режиме TPS ниже из-за ожидания подтверждения каждой транзакции перед началом следующей, в то время как в асинхронном режиме TPS выше из-за возможности начала новых транзакций до подтверждения предыдущих.

6) Создайте новый кластер с включенной контрольной суммой страниц. Создайте таблицу. 

- После создания таблицы получим путь к файлу
```
SELECT pg_relation_filepath('test_table');
 pg_relation_filepath 
----------------------
 base/16397/16399
(1 row)
```
Вставьте несколько значений. Выключите кластер. Измените пару байт в таблице. Включите кластер и сделайте выборку из таблицы.  Что и почему произошло? как проигнорировать ошибку и продолжить работу?

- Страницы данных защищены контрольными суммами, при изменении вручную данных получаем ошибку:
```
WARNING:  page verification failed, calculated checksum 34566 but expected 38146
ERROR:  invalid page in block 0 of relation base/16397/16399
```
- Для игнорирования контрольной суммы включаем:
```
SET ignore_checksum_failure = on;
```
После этого SELECT работает
