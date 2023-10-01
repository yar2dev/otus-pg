# Домашка к уроку "MVCC, vacuum и autovacuum."


1) Создать инстанс ВМ с 2 ядрами и 4 Гб ОЗУ и SSD 10GB

2) Установить на него PostgreSQL 15 с дефолтными настройками

3) Создать БД для тестов: выполнить pgbench -i postgres
4) Запустить pgbench -c8 -P 6 -T 60 -U postgres postgres
```
с настройками по умолчанию:
tps = 657.265847 (without initial connection time)
```
5) Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла
Протестировать заново
```
с настройками из файла:
tps = 673.095332 (without initial connection time)
```
Что изменилось и почему?
```
бенчмарк не показал прироста производительности, хотя в реальных запросах прирост есть. увеличены рабочая память, кеши, работа с диском.
```
6) Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1млн строк
```
CREATE TABLE test_table (
    id SERIAL PRIMARY KEY,
    text_field TEXT
);
INSERT INTO test_table (text_field)
SELECT md5(random()::text)
FROM generate_series(1, 1000000);
```
Посмотреть размер файла с таблицей
```
\df+
                                             List of relations
 Schema |       Name        |   Type   |  Owner   | Persistence | Access method |    Size    | Description 
--------+-------------------+----------+----------+-------------+---------------+------------+-------------
 public | test_table        | table    | postgres | permanent   | heap          | 65 MB      | 

```
7) 5 раз обновить все строчки и добавить к каждой строчке любой символ
Посмотреть количество мертвых строчек в таблице и когда последний раз приходил автовакуум
```
UPDATE test_table SET text_field = text_field || 'A';
UPDATE test_table SET text_field = text_field || 'B';
UPDATE test_table SET text_field = text_field || 'C';
UPDATE test_table SET text_field = text_field || 'D';
UPDATE test_table SET text_field = text_field || 'E';
```
```
SELECT schemaname,relname,n_live_tup,n_dead_tup,last_vacuum,last_autovacuum FROM pg_stat_user_tables ORDER BY n_dead_tup;
 schemaname |     relname      | n_live_tup | n_dead_tup |          last_vacuum          |        last_autovacuum        
------------+------------------+------------+------------+-------------------------------+-------------------------------
 public     | test_table       |    1000000 |     999855 |                               | 2023-10-01 16:43:25.255273+00
```
Подождать некоторое время, проверяя, пришел ли автовакуум
```
 schemaname |     relname      | n_live_tup | n_dead_tup |          last_vacuum          |        last_autovacuum        
------------+------------------+------------+------------+-------------------------------+-------------------------------
 public     | test_table       |    1526160 |          0 |                               | 2023-10-01 16:44:26.251935+00
```

8) 5 раз обновить все строчки и добавить к каждой строчке любой символ
Посмотреть размер файла с таблицей
```
 Schema |       Name        |   Type   |  Owner   | Persistence | Access method |    Size    | Description 
--------+-------------------+----------+----------+-------------+---------------+------------+-------------
 public | test_table        | table    | postgres | permanent   | heap          | 308 MB     | 

```
9) Отключить Автовакуум на конкретной таблице
```

ALTER TABLE test_table SET (autovacuum_enabled = false);
```
10) 10 раз обновить все строчки и добавить к каждой строчке любой символ
Посмотреть размер файла с таблицей
```
 Schema |       Name        |   Type   |  Owner   | Persistence | Access method |    Size    | Description 
--------+-------------------+----------+----------+-------------+---------------+------------+-------------
 public | test_table        | table    | postgres | permanent   | heap          | 990 MB     | 
```
Объясните полученный результат
```
При аддейте старые данные на диске ну удаляются, только добавляются.
Если выполнить  vacuul full test_table; размер таблицы на диске становится 104 MB.

```
