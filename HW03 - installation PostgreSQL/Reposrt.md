# Отчет о выполнении домашнего заданичя
## Тема: Установка PostgreSQL

1.  На личном ПК был установлен Docker согласно инструкции 
[Установка Docker на Mac](https://docs.docker.com/desktop/install/mac-install/)
2. Чтобы убедится в том, что контейнуры создаются успешно была выполнена команда: 
```CMD
docker run --name some-postgres -e POSTGRES_PASSWORD=mysecretpassword -d postgres
```
3. Так как корректно смонтировать директорию /var/lib/postgresql не вышло была смонтирована /var/lib/postgresql/data.
Команда с помощью которой указана ниже
```CMD
docker run -p 5433:5432 -v /Users/yakushnikita/Documents/var/lib/postgresql/data:/var/lib/postgresql/data --name PGOTUS -e POSTGRES_PASSWORD=TestPass -d postgres
```
4. Через psql подключился к серверу и запустил скрипт создания таблицы и скрипт заполнения таблиццы данными:

```SQL
create table public.testtable (
                                  id serial not null,
                                  value varchar not null
);

insert
into
    public.testtable
    (value)
values('test1');

insert
into
    public.testtable
    (value)
values('test2');
```
5. Подключился через DBeaver и проверил текущее состояние БД. Таблица создана успешно. 
6. Удалил контейнер с сервером docker коммандами:
```CMD
sudo docker stop PGOTUS 
sudo docker rm PGOTUS  
```
7. Создал контейнер заново коммандой:
```CMD
docker run -p 5433:5432 -v /Users/yakushnikita/Documents/var/lib/postgresql/data:/var/lib/postgresql/data --name PGOTUS -e POSTGRES_PASSWORD=TestPass -d postgres
```
8. Проверил наличие данных. Данные не пропали.