# Домашка к уроку "Логический уровень PostgreSQL "
1 создайте новый кластер PostgresSQL 14
```
apt install postgresql
```
2 зайдите в созданный кластер под пользователем postgres
```
sudo postgres
psql
```
3 создайте новую базу данных testdb
```
create database testdb;
```
4 зайдите в созданную базу данных под пользователем postgres
```
\c testdb
```
5 создайте новую схему testnm
```
create schema testnm;
```
6 создайте новую таблицу t1 с одной колонкой c1 типа integer
```
create table t1 (c1 int);
```
7 вставьте строку со значением c1=1
```
insert into t1 values (1);
```
8 создайте новую роль readonly
```
create role readonly;
```
9 дайте новой роли право на подключение к базе данных testdb
```
grant connect on database testdb to readonly;
```
10 дайте новой роли право на использование схемы testnm
```
GRANT USAGE ON SCHEMA testnm TO readonly;
```
11 дайте новой роли право на select для всех таблиц схемы testnm
```
GRANT SELECT ON ALL TABLES IN SCHEMA testnm TO readonly;
```
12 создайте пользователя testread с паролем test123
```
CREATE USER testread WITH PASSWORD 'test123';
```
13 дайте роль readonly пользователю testread
```
GRANT readonly TO testread;
```
14 зайдите под пользователем testread в базу данных testdb
```
psql -h localhost -U testread testdb
```
15 сделайте select * from t1;
```
testdb=> select * from c1;
```
16 получилось? (могло если вы делали сами не по шпаргалке и не упустили один существенный момент про который позже)
```
ERROR:  permission denied for table t1
testdb=> select * from testnm.t1;
ERROR:  relation "testnm.t1" does not exist
LINE 1: select * from testnm.t1;
```
17 напишите что именно произошло в тексте домашнего задания
18 у вас есть идеи почему? ведь права то дали?
```
таблица t1 создана в схеме public, так как по умолчанию search path указывает на public
```
19 посмотрите на список таблиц
```
\dt
testdb=> \dt
        List of relations
 Schema | Name | Type  |  Owner   
--------+------+-------+----------
 public | t1   | table | postgres
(1 row)
```
20 подсказка в шпаргалке под пунктом 20
21 а почему так получилось с таблицей (если делали сами и без шпаргалки то может у вас все нормально)
```
по умолчанию search path указывает на public
```
22 вернитесь в базу данных testdb под пользователем postgres
```
\q
sudo postgres
psql
```
23 удалите таблицу t1
```
drop table t1;
```
24 создайте ее заново но уже с явным указанием имени схемы testnm
```
create table testnm.t1 (c1 int);
```
25 вставьте строку со значением c1=1
```
insert into testnm.t1 (c1) values (1);
```
26 зайдите под пользователем testread в базу данных testdb
```
psql -h localhost -U testread testdb
```
27 сделайте select * from testnm.t1;
```
testdb=> select * from testnm.t1;
```
28 получилось?
```
ERROR:  permission denied for table t1
```
29 есть идеи почему? если нет - смотрите шпаргалку
```
Права выдали на существующие таблицы. Таблицу пересоздали - права заного не назначали.
```
30 как сделать так чтобы такое больше не повторялось? если нет идей - смотрите шпаргалку
```
назначить права для таблиц, включая вновь созаваемые
ALTER default privileges in SCHEMA testnm grant SELECT on TABLES to readonly; 
```
31 сделайте select * from testnm.t1;
```
testdb=> select * from testnm.t1;
 c1 
----
  1
(1 row)
```
32 получилось?
```
теперь получилось
```
33 есть идеи почему? если нет - смотрите шпаргалку
31 сделайте select * from testnm.t1;
32 получилось?
33 ура!
34 теперь попробуйте выполнить команду create table t2(c1 integer); insert into t2 values (2);
```
testdb=> create table t2(c1 integer); insert into t2 values (2);
CREATE TABLE
INSERT 0 1
```
35 а как так? нам же никто прав на создание таблиц и insert в них под ролью readonly?
```
Таблица создалась в public, где create по дефолту есть в PG 14
testdb=> \dt
        List of relations
 Schema | Name | Type  |  Owner   
--------+------+-------+----------
 public | t2   | table | testread
(1 row)

```
36 есть идеи как убрать эти права? если нет - смотрите шпаргалку
```
Отозвать права на создание таблиц в схеме public
REVOKE CREATE ON SCHEMA public FROM PUBLIC;
Можно отозвать все права через revoce all database 'database_name' from public; 
```
37 если вы справились сами то расскажите что сделали и почему, если смотрели шпаргалку - объясните что сделали и почему выполнив указанные в ней команды
38 теперь попробуйте выполнить команду create table t3(c1 integer); insert into t2 values (2);
```
Теперь нет прав на создание таблиц
ERROR:  permission denied for schema public
LINE 1: create table t3(c1 integer);
                     ^
INSERT 0 1
```
