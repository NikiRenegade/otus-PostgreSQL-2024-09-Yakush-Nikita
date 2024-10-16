# Отчет о выполнении домашнего заданичя
## Тема: SQL и реляционные СУБД. Введение в PostgreSQL

1.  Cоздана таблица "persons" и заполненена первоначальными данными:

```SQL
create table persons(id serial,
                     first_name text,
                     second_name text);

insert
into
    persons(first_name,
            second_name)
values('ivan',
       'ivanov');

insert
into
    persons(first_name,
            second_name)
values('petr',
       'petrov');

commit;
```
2. Открыто две новые ссесия и проверен текущий уровень изоляции:
```SQL
show transaction isolation level
```
**Результат:**

| transaction_isolation | 
|-----------------------| 
| read committed        |

3. В первой сессии добавлена новая запись: 

```SQL
insert
	into
	persons(first_name,
	second_name)
values('sergey',
'sergeev')
```
4. Во второй сессии выполнена комманда:
```SQL
select * from persons
```
**Результат:**

|id|first_name|second_name|
|--|----------|-----------|
|1|ivan|ivanov|
|2|petr|petrov|


Новая запись во второй сессии не отобразилась тк не был выполнен commit первой трнзакции.
При уровне изоляции транзакции "read uncommited" 
присутствовала бы аномалия "грязного чтения" и запись была бы видна до коммита.
5. Выполнен commit первой транзакции.
6. Во второй сессии выполнена комманда:
```SQL
select * from persons
```
**Результат:**

| id |first_name|second_name|
|----|----------|-----------|
| 1  |ivan|ivanov|
| 2  |petr|petrov|
| 3  |sergey|sergeev|

Новая запись во второй сессии отобразилась тк был выполнен commit первой трнзакции.
7. Завершены две старые сессии.
8. открыто две новые ссесия и изменен текущий уровень изоляции на repeatable read:
```SQL
set transaction isolation level repeatable read;
```
9. В первой сессии добавлена новая запись:

```SQL
insert
into
    persons(first_name,
            second_name)
values('sveta',
       'svetova');
```
10. Во второй сессии выполнена комманда:
```SQL
select * from persons
```
**Результат:**

| id |first_name|second_name|
|----|----------|-----------|
| 1  |ivan|ivanov|
| 2  |petr|petrov|
| 3  |sergey|sergeev|

Новая запись во второй сессии не отобразилась тк не был выполнено commit,
а уровень изоляции repeatable read более строгий чем read commited.
11. Выполнен commit первой транзакции.
12. Во второй сессии выполнена комманда:
```SQL
select * from persons
```
**Результат:**

| id |first_name|second_name|
|----|----------|-----------|
| 1  |ivan|ivanov|
| 2  |petr|petrov|
| 3  |sergey|sergeev|

Новая запись во второй сессии не отобразилась тк уровень изоляции repeatable read более строгий чем read commited 
и не допускает аномалии неповторяющегося чтения.
В новых сессиях, которые будут созданы после успешного коммита второй транзакции, новая запись будет отображатся
13. Закрты обе сессии