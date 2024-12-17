# Отчет о выполнении домашнего задания
## Тема: Логический уровень PostgreSQL
1. Создал виртуальную машину, установил PostgreSQL 15, выдал доступ из вне
2. Проверил работоспособность 
3. Зашел в psql под пользователем postgres
```CMD
sudo -u postgres psql
```
4. Создал новую базу данных testdb
```SQL
create database testdb
```
5. Зашел в БД testdb (\c testdb)
6. создайте новую схему testnm
```SQL
create schema testnm
```
7. Создал новую таблицу t1 и заполнил данными:
```SQL
create table testnm.t1(c1 integer);
insert
into
    testnm.t1
values(1);
```
8. Cоздаk новую роль readonly, дал роли readonly право на подключение к БД testdb, 
дал роли readonly право на использование схемы testnm, дал роли readonly
право на select для всех таблиц схемы testnm
```SQL
create role readonly;

grant connect on
database testdb to readonly;

grant usage on
schema testnm to readonly;

grant
select
on
    all tables in schema testnm to readonly;
```
9. Создал пользователя testread с паролем test123, дал роль readonly пользователю testread:
```SQL
create user testread with password 'test123';

grant readonly to testread;
```
10. Переподключился под пользователя testread к БД testdb
```CMD
psql -U testread -h 127.0.0.1 -d testdb -W
```
11. Выполнил select таблицы t1
```SQL
select
    *
from
    testnm.t1 
```
12. Select прошел успешно
13. Читая задние дальше понятно, что расчет был на то, что таблица создана в схеме public
14. Выполнил команду создания таблицы t2
```SQL
create table testnm.t2(c1 integer);
```
Выполнить create не вышло из-за отсутсвия прав: 
```CMD
ОШИБКА:  нет доступа к схеме testnm
СТРОКА 1: create table testnm.t2(c1 integer);
```
попробовал выполнить в схеме public:
```SQL
create table public.t2(c1 integer);
```
Так же не вышло:
```CMD
ОШИБКА:  нет доступа к схеме public
СТРОКА 1: create table public.t2(c1 integer);
```
15. **Итог:** добавить таблицу в схему public не вышло из-за отсутвия дефолтных прав у роли readonly.
Не заметил, что нужно было использовать PostgreSQL 14... в котором данные права были. Использовал PostgreSQL 15.
Но суть уловил, поэтому ДЗ отдаю под сдачу. 
