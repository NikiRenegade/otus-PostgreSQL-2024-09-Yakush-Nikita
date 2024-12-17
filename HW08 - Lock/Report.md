# Отчет о выполнении домашнего задания
## Тема: Блокировки

### 1. Воспроизведите ситуацию, при которой в журнале появятся сообщения о блокировках.
1. Установил deadlock_timeout 200 мс, включил запись в журнал информации о блокировках и перезагрузил кофигурацию:
```SQL
alter system set
deadlock_timeout to 200;

alter system set
log_lock_waits = on;

select pg_reload_conf();
```
2. Начал две сессии, посмотрел id данных сессий: 
```SQL
SELECT pg_backend_pid();
```
| № сессии | pg_backend_pid |
|----------|----------------|
| 1        | 1121           |
| 2        | 1383           |


Для своей работы использовал таблицу из первого ДЗ:

| id | first_name | second_name |
|----|------------|-------------|
| 1  | ivan       | ivanov      |
| 2  | petr       | petrov      |
| 3  | sergey     | sergeev     |
| 4  | sveta      | svetova     |

3. В первой сессии выполнил команду:
```SQL
begin;
alter table public.persons
    add Address text null;

select pg_sleep(10)

    commit
```
4. Не дожидаясь выполнения первой сессии во второй сессии выполнил комманду:
```SQL
begin;

select * from persons p ;

commit;
```
и как следствие получил блокировку между ACCESS SHARE (select во второй сессии)
и ACCESS EXCLUSIVE (alter table в первой сесии). 

5. Посмотрел журнал:
```CMD
sudo tail -n 20 /var/log/postgresql/postgresql-15-main.log
```
**Результат:**
```CMD
2024-10-31 12:36:11.319 +07 [1383] postgres@OTUS СООБЩЕНИЕ:  процесс 1383 продолжает ожидать в режиме AccessShareLock блокировку "отношение 16421 базы данных 16391" в течение 205.513 мс (символ 16)
2024-10-31 12:36:11.319 +07 [1383] postgres@OTUS ПОДРОБНОСТИ:  Process holding the lock: 1121. Wait queue: 1383.
2024-10-31 12:36:11.319 +07 [1383] postgres@OTUS ОПЕРАТОР:  
	select * from persons p 
2024-10-31 12:36:43.818 +07 [1383] postgres@OTUS СООБЩЕНИЕ:  процесс 1383 получил в режиме AccessShareLock блокировку "отношение 16421 базы данных 16391" через 32704.802 мс (символ 16)
```
На второй строке видно, что транзакция первой сессии (1121) блокирует транзакцию второй сессии (1383)

### 2. Смоделировать ситуацию обновления одной и той же строки тремя командами UPDATE в разных сеансах. 

1. Закрыл старые две сессии и создал три новых:

| № сессии | pg_backend_pid |
|----------|----------------|
| 1        | 1641           |
| 2        | 1670           |
| 3        | 1678           |

2. В первой транзакции выполнил комманду: 
```SQL
begin;
update public.persons
set second_name = 'Sergeeva'
where id = 4;
```
Во второй:
```SQL
begin;
update public.persons
set second_name = 'Ivanova'
where id = 4;
```
В третьей:
```SQL
begin;
update public.persons
set second_name = 'Petrova'
where id = 4;
```
3. Далее выполнил комманду:
```SQL
SELECT locktype, relation::REGCLASS, virtualxid AS virtxid, transactionid AS xid, mode, granted
FROM pg_locks WHERE pid = 1641;
```
Результат:

| locktype      | relation | virtxid | xid      | mode             | granted |
|---------------|----------|---------|----------|------------------|---------|
| relation      | pg_locks |         |          | AccessShareLock  | true    |
| relation      | persons  |         |          | RowExclusiveLock | true    |
| virtualxid    |          | 9/512   |          | ExclusiveLock    | true    |
| transactionid |          |         | 13979290 | ExclusiveLock    | true    |

Первая строка сообщает, что таблица pg_locks занята блокировкой AccessShared (Тк был выполнен select)

Вторая строка сообщает, что таблица persons занята блокировкой RowExclusiveLock (построчная блокировка при update)

Тертья строка сообщает, что присутвует эксклюзивная блокировка самой себя

Четвертая строка сообщает, что присутсвует эксклюзивная блокировка для транзакции выполняющей комманду update

4. Далее выполнил комманду уже для второй сессии:
```SQL
SELECT locktype, relation::REGCLASS, virtualxid AS virtxid, transactionid AS xid, mode, granted
FROM pg_locks WHERE pid = 1670;
```
Результат:

| locktype      | relation  | virtxid  | xid      | mode             | granted  |
|---------------|-----------|----------|----------|------------------|----------|
| relation      | persons   |          |          | RowExclusiveLock | true     |
| virtualxid    |           | 10/83    |          | ExclusiveLock    | true     |
| tuple         | persons   |          |          | ExclusiveLock    | true     |
| transactionid |           |          | 13979290 | ShareLock        | false    |
| transactionid |           |          | 13979291 | ExclusiveLock    | true     |

Первая строка сообщает, что таблица persons занята блокировкой RowExclusiveLock (построчная блокировка при update)

Вторая строка сообщает, что присутвует эксклюзивная блокировка самой себя

Тертья и четвертая строка сообщает, что транзакция пытается изменить таблицу person и блокирует остальнцые изменения и так же, что сама блокируется транзакцией 13979290

Пятая строка сообщает, что присутсвует эксклюзивная блокировка для транзакции выполняющей комманду update

5. Далее выполнил комманду уже для третьей сессии:
```SQL
SELECT locktype, relation::REGCLASS, virtualxid AS virtxid, transactionid AS xid, mode, granted
FROM pg_locks WHERE pid = 1678;
```
| locktype      | relation  | virtxid  | xid      | mode             | granted  |
|---------------|-----------|----------|----------|------------------|----------|
| relation      | persons   |          |          | RowExclusiveLock | true     |
| virtualxid    |           | 8/211    |          | ExclusiveLock    | true     |
| transactionid |           |          | 13979292 | ExclusiveLock    | true     |
| tuple         | persons   |          |          | ExclusiveLock    | false    |

Первая строка сообщает, что таблица persons занята блокировкой RowExclusiveLock (построчная блокировка при update)

Вторая строка сообщает, что присутвует эксклюзивная блокировка самой себя

Третья строка сообщает, что присутсвует эксклюзивная блокировка для транзакции выполняющей комманду update

Четвертая строка сообщает, что транзакция пытается изменить таблицу person, но сама блокируется

### 3. Воспроизвести взаимоблокировку трех транзакций.
Номера ссесий:
| № сессии | pg_backend_pid |
|----------|----------------|
| 1        | 1641           |
| 2        | 1670           |
| 3        | 1678           |

1. В первой сессии выполнил команду:
```SQL
begin;
update public.persons
set address = '1'
where id = 1;
```
2. Во второй сессии выполнил команду:
```SQL
begin;
update public.persons
set address = '2'
where id = 2;
```
3. В третьей сессии выполнил команду:
```SQL
begin;
update public.persons
set address = '3'
where id = 3;
```
4. В первой сессии выполнил команду:
```SQL
update public.persons
set address = '3'
where id = 2;
```
5. Во второй сессии выполнил команду:
```SQL
update public.persons
set address = '1'
where id = 3;
```
6. В третьей сессии выполнил команду:
```SQL
update public.persons
set address = '2'
where id = 1;
```
После этого DBeaver выдал ошибку и postgreSQL откинул третью сессию с транзакцией тем самым освободив вторую. Первая транзакция была по прежнему заблокирована второй.
По завершению второй транзакции первая освободилась. 

Посмотрев журнал коммандой;
```CMD
sudo tail -n 20 /var/log/postgresql/postgresql-15-main.log
```
Можно обноружить информацию о взаимоблокировках:
```CMD
2024-10-31 15:54:21.318 +07 [1678] postgres@OTUS ПОДРОБНОСТИ:  
        Процесс 1678 ожидает в режиме ShareLock блокировку "транзакция 13979294"; заблокирован процессом 1641.
	Процесс 1641 ожидает в режиме ShareLock блокировку "транзакция 13979295"; заблокирован процессом 1670.
	Процесс 1670 ожидает в режиме ShareLock блокировку "транзакция 13979296"; заблокирован процессом 1678.
```
### 4. Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга?
На сколько я понял две транзакции не могу заблокировать друг друга. Вторая транзакция будет заблокирована первой. Но возможно я что-то упускаю
### *. Воспроизвести такую ситуацию (Если в пунке выше ничего не упустил). 
1. В первой сессии выполнил команду:
```SQL
begin;
update public.persons
set address = '1'
```
2. Во второй сессии выполнил команду:
```SQL
begin;
update public.persons
set address = '2'
```

Эти действия привели к блокировки второй транзакции.