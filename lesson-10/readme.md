# Домашка "Блокировки"

1) Настройте сервер так, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 миллисекунд. Воспроизведите ситуацию, при которой в журнале появятся такие сообщения.

- Подготовка:
```
ALTER SYSTEM SET log_lock_waits = on;
ALTER SYSTEM SET deadlock_timeout = 200;
SELECT pg_reload_conf();
CREATE DATABASE loks;
CREATE TABLE test_locks (num int);
INSERT into test_locks VALUES (10);

```
В первой сессии выполним блокировку
```
BEGIN
LOCK TABLE test_locks IN ACCESS EXCLUSIVE MODE;
```
в второй сделаем выборку
```
SELECT * FROM test_locks;
```
в логах видим:
```
LOG:  process 4538 still waiting for AccessShareLock on relation 16385 of database 16384 after 200.121 ms at character 15
```

2) Смоделируйте ситуацию обновления одной и той же строки тремя командами UPDATE в разных сеансах. Изучите возникшие блокировки в представлении pg_locks и убедитесь, что все они понятны. Пришлите список блокировок и объясните, что значит каждая.

```
сеанс 1
begin;
UPDATE test_locks SET num=11;
```
```
сеанс 2
begin;
UPDATE test_locks SET num=12;
```
```
сеанс 3
begin;
UPDATE test_locks SET num=13;
```
```
В логах видем
на втором апдейте: ... waiting for ShareLock
на третем апдейте: ...waiting for ExclusiveLock
```
Сообщается о ожидании установки ShareLock, так как запись заблокирована первым апдейтом. 
ShareLock допускает только операцию чтения для других процессов
ExclusiveLock не позволяет выполнять операции чтения и записи другим процессам  


3) Воспроизведите взаимоблокировку трех транзакций. Можно ли разобраться в ситуации постфактум, изучая журнал сообщений?

```
 сеанс 1
BEGIN;
UPDATE test_locks SET num = 1 WHERE no= 1;

 сеанс 2
BEGIN;
UPDATE test_locks SET num = 2 WHERE no= 2;

 сеанс 1
UPDATE test_locks SET num = 3 WHERE no= 2;

 сеанс 1
 UPDATE test_locks SET num = 4 WHERE no= 1;
```
```
В логах  есть сообщение о дедлоке
ERROR:  deadlock detected
2023-10-17 20:23:18.047 UTC [2133] postgres@loks DETAIL:  Process 2133 waits for ShareLock on transaction 773; blocked by process 2132.
	Process 2132 waits for ShareLock on transaction 774; blocked by process 2133.
	Process 2133: UPDATE test_locks SET num = 4 WHERE no= 1;
	Process 2132: UPDATE test_locks SET num = 3 WHERE no= 2;
Соостветсвенно по логам можно найти причину дедлока
```
4) Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга?
```
Может возникнуть ситуация, когда два апдейта пересекутся на одной строке то случится взаимоблокировка
```
