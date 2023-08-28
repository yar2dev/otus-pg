
# Домашка по теме: работа с уровнями изоляции, транзакции в PostgreSQL

## Выполнение
- Установлен postgresql на ВМ в яндекс облаке
- Выполнено подключение к ВМ с двух консолей
- Выключен auto commit
- Выполнены запросы:
```
create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;
```
```
show transaction isolation level
``` 
результат:  read committed

- в первой сессии добавлены новые записи 
``` 
insert into persons(first_name, second_name) values('sergey', 'sergeev');
```
- во второй сессии выполнено 
```
select * from persons 
```
результат: во второй сесии новой записи нет, так как транзакция не завершена

- в первой сесси выполнено:
```
commit;
```
результат: во второй сессии появилась новая запись так как транзакция в первой сессии завершена и при уровне изоляции read committed возможно фантомное чтение

- начать новые но уже repeatable read транзации - set transaction isolation level repeatable read; В первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova'); сделать select * from persons во второй сессии видите ли вы новую запись и если да то почему?

результат: во второй сессии после коммита в первой новая запись не видна, так как фантомное чтение в режиме repeatable read не допускается
