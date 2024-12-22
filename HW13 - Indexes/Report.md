# Отчет о выполнении домашнего задания
## Виды индексов. Работа с индексами и оптимизация запросов

1. Создал 5 таблиц (данные таблицы взяты из практики вебинара): 
```SQL
create table hwindex."SimpleIndex" (
    id int,
    user_id int,
    order_date date,
    status text,
    some_text text
);

create table hwindex."FullTextSearchingIndexTable" (
    id int,
    user_id int,
    order_date date,
    status text,
    some_text text
);

create table hwindex."FullTextSearchingIndexTable2" (
    id int,
    user_id int,
    order_date date,
    status text,
    some_text text
);

create table hwindex."PartialIndex" (
    id int,
    user_id int,
    order_date date,
    status text,
    some_text text
);

create table hwindex."MulticolumnIndexes" (
    id int,
    user_id int,
    order_date date,
    status text,
    some_text text
);
```

| Таблица                      | Для чего                                                                |
 |------------------------------|-------------------------------------------------------------------------|
| SimpleIndex                  | Для самого простого индекса                                             |
| FullTextSearchingIndexTable  | Для индекса полнотекстового поиска с добавлением столбца типа tsvector  |
| FullTextSearchingIndexTable2 | Для индекса полнотекстового поиска с добавлением столбца типа tsvector  |
| PartialIndex                 | Для частичного индекс                                                   |
| MulticolumnIndexes           | Для составного индекса                                                  |

2. Добавил к таблице FullTextSearchingIndexTable новый столбец и заполнил все таблицы данными (данные также из практики вебинара):

```SQL
insert into hwindex."SimpleIndex"(id, user_id, order_date, status, some_text)
select generate_series, (random() * 70), date'2019-01-01' + (random() * 300)::int as order_date
        , (array['returned', 'completed', 'placed', 'shipped'])[(random() * 4)::int]
        , concat_ws(' ', (array['go', 'space', 'sun', 'London'])[(random() * 5)::int]
            , (array['the', 'capital', 'of', 'Great', 'Britain'])[(random() * 6)::int]
            , (array['some', 'another', 'example', 'with', 'words'])[(random() * 6)::int]
            )
from generate_series(1, 1000000);



insert into hwindex."FullTextSearchingIndexTable"(id, user_id, order_date, status, some_text)
select id, user_id, order_date, status, some_text from  hwindex."SimpleIndex"


alter table hwindex."FullTextSearchingIndexTable" add column some_text_lexeme tsvector;
update hwindex."FullTextSearchingIndexTable"
set some_text_lexeme = to_tsvector(some_text);



insert into hwindex."FullTextSearchingIndexTable2"(id, user_id, order_date, status, some_text)
select id, user_id, order_date, status, some_text from  hwindex."SimpleIndex"



insert into hwindex."PartialIndex"(id, user_id, order_date, status, some_text)
select id, user_id, order_date, status, some_text from  hwindex."SimpleIndex"


insert into hwindex."MulticolumnIndexes"(id, user_id, order_date, status, some_text)
select id, user_id, order_date, status, some_text from  hwindex."SimpleIndex"
```

### Обычный индекс
3. Запустил скрипт:
```SQL
explain analyze
select * from hwindex."SimpleIndex" where id = 10;
```
**Результат**

|id| user_id  | order_date  | status    | some_text          |
|--|----------|-------------|-----------|--------------------|
|1| 28       | 2019-06-04  | placed    | London the example |
|2| 53       | 2019-05-23  | completed | space Britain      |

| QUERY PLAN                                                                                                                   |
|------------------------------------------------------------------------------------------------------------------------------|
| Gather  (cost=1000.00..14221.43 rows=1 width=34) (actual time=0.714..67.955 rows=1 loops=1)                                  |
| Workers Planned: 2                                                                                                           |
| Workers Launched: 2                                                                                                          |
| ->  Parallel Seq Scan on "SimpleIndex" si  (cost=0.00..13221.33 rows=1 width=34) (actual time=14.766..35.844 rows=0 loops=3) |
| Filter: (id = 10)                                                                                                            |
| Rows Removed by Filter: 333333                                                                                               |
| Planning Time: 0.369 ms                                                                                                      |
| Execution Time: 67.977 ms                                                                                                    |
                                                                                          |

4. Создал индекс на таблицу, на столбец id:
```SQL
create index idx_SimpleIndex_id on hwindex."SimpleIndex"(id);
```
5. Запустил скрипт:
```SQL
explain analyze
select * from hwindex."SimpleIndex" where id = 10;
```
**Результат**

| id  | user_id  | order_date  | status    | some_text          |
|-----|----------|-------------|-----------|--------------------|
| 1   | 28       | 2019-06-04  | placed    | London the example |
| 2   | 53       | 2019-05-23  | completed | space Britain      |

| QUERY PLAN                                                                                                                           |
|--------------------------------------------------------------------------------------------------------------------------------------|
| Index Scan using idx_simpleindex_id on "SimpleIndex" si  (cost=0.42..8.44 rows=1 width=34) (actual time=0.097..0.098 rows=1 loops=1) |
| Index Cond: (id = 10)                                                                                                                |
| Planning Time: 0.665 ms                                                                                                              |
| Execution Time: 0.146 ms                                                                                                             |
                                                                                                              |

Видно явное уменьшение времени выполнения запроса и использование индекса. 

### Индекс для полнотекстового поиска
6. Запустил скрипт:
```SQL
explain analyze
select * from hwindex."FullTextSearchingIndexTable" 
where some_text_lexeme @@ to_tsquery('london');
```
**Результат (только две строки и план запроса)**

| id   | user_id  | order_date  | status   | some_text            | some_text_lexeme                |
|------|----------|-------------|----------|----------------------|---------------------------------|
| 1658 | 5        | 2019-06-20  | returned | London Great example | 'exampl':3 'great':2 'london':1 |
| 1661 | 69       | 2019-05-09  | placed   | London the with      | 'london':1                      |

| QUERY PLAN                                                                                                                                         |
|----------------------------------------------------------------------------------------------------------------------------------------------------|
| Gather  (cost=1000.00..149844.30 rows=198733 width=62) (actual time=10.991..558.905 rows=200352 loops=1)                                           |
| Workers Planned: 2                                                                                                                                 |
| Workers Launched: 2                                                                                                                                |
| ->  Parallel Seq Scan on "FullTextSearchingIndexTable"  (cost=0.00..128971.00 rows=82805 width=62) (actual time=5.154..408.532 rows=66784 loops=3) |
| Filter: (some_text_lexeme @@ to_tsquery('london'::text))                                                                                           |
| Rows Removed by Filter: 266549                                                                                                                     |
| Planning Time: 0.262 ms                                                                                                                            |
| JIT:                                                                                                                                               |
| Functions: 6                                                                                                                                       |
| Options: Inlining false, Optimization false, Expressions true, Deforming true                                                                      |
| Timing: Generation 1.015 ms, Inlining 0.000 ms, Optimization 1.081 ms, Emission 11.107 ms, Total 13.203 ms                                         |
| Execution Time: 565.426 ms                                                                                                                         |
                                                                                             |

7. Создал индек на таблицу, на столбец some_text_lexeme:
```SQL
CREATE INDEX search_index_ftsit ON hwindex."FullTextSearchingIndexTable"  USING GIN (some_text_lexeme);
```
8. Запустил скрипт:
```SQL
explain analyze
select * from hwindex."FullTextSearchingIndexTable" 
where some_text_lexeme @@ to_tsquery('london');
```
**Результат (только две строки и план запроса)**

| id   | user_id  | order_date  | status   | some_text            | some_text_lexeme                |
|------|----------|-------------|----------|----------------------|---------------------------------|
| 1658 | 5        | 2019-06-20  | returned | London Great example | 'exampl':3 'great':2 'london':1 |
| 1661 | 69       | 2019-05-09  | placed   | London the with      | 'london':1                      |

| QUERY PLAN                                                                                                                                                   |
|--------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Gather  (cost=2836.43..64042.16 rows=198733 width=62) (actual time=37.633..86.928 rows=200352 loops=1)                                                       |
| Workers Planned: 2                                                                                                                                           |
| Workers Launched: 2                                                                                                                                          |
| ->  Parallel Bitmap Heap Scan on "FullTextSearchingIndexTable"  (cost=1836.43..43168.86 rows=82805 width=62) (actual time=12.295..33.063 rows=66784 loops=3) |
| Recheck Cond: (some_text_lexeme @@ to_tsquery('london'::text))                                                                                               |
| Heap Blocks: exact=5407                                                                                                                                      |
| ->  Bitmap Index Scan on search_index_ftsit  (cost=0.00..1786.75 rows=198733 width=0) (actual time=35.081..35.081 rows=200352 loops=1)                       |
| Index Cond: (some_text_lexeme @@ to_tsquery('london'::text))                                                                                                 |
| Planning Time: 0.679 ms                                                                                                                                      |
| Execution Time: 93.759 ms                                                                                                                                    |
                                                                                                          |

9. Далее решил проверить полнотекстовый поиск не создавая отдельного столбца для этого:

```SQL
explain analyze
select * from hwindex."FullTextSearchingIndexTable" 
where some_text_lexeme @@ to_tsquery('london');
```
**Результат (только две строки и план запроса)**

| id  | user_id  | order_date  | status  | some_text           |
|-----|----------|-------------|---------|---------------------|
| 1   | 28       | 2019-06-04  | placed  | London the example  |
| 6   | 50       | 2019-06-06  | placed  | London Britain with |

| QUERY PLAN                                                                                                                                          |
|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| Gather  (cost=1000.00..223054.67 rows=5000 width=34) (actual time=6.982..1315.155 rows=200352 loops=1)                                              |
| Workers Planned: 2                                                                                                                                  |
| Workers Launched: 2                                                                                                                                 |
| ->  Parallel Seq Scan on "FullTextSearchingIndexTable2"  (cost=0.00..221554.67 rows=2083 width=34) (actual time=5.372..1275.220 rows=66784 loops=3) |
| Filter: (to_tsvector(some_text) @@ to_tsquery('london'::text))                                                                                      |
| Rows Removed by Filter: 266549                                                                                                                      |
| Planning Time: 0.606 ms                                                                                                                             |
| JIT:                                                                                                                                                |
| Functions: 6                                                                                                                                        |
| Options: Inlining false, Optimization false, Expressions true, Deforming true                                                                       |
| Timing: Generation 1.111 ms, Inlining 0.000 ms, Optimization 0.756 ms, Emission 10.563 ms, Total 12.429 ms                                          |
| Execution Time: 1321.681 ms                                                                                                                         |
                                                                                                          |

10. Создал индекс на таблицу, на столбец some_text:
```SQL
CREATE INDEX search_index_ftsit ON hwindex."FullTextSearchingIndexTable"  USING GIN (some_text_lexeme);
```
11. Запустил скрипт:
```SQL
explain analyze
select *
from hwindex."FullTextSearchingIndexTable2" 
where to_tsvector(some_text) @@ to_tsquery('london');
```
**Результат (только две строки и план запроса)**

| id   | user_id  | order_date  | status   | some_text            | some_text_lexeme                |
|------|----------|-------------|----------|----------------------|---------------------------------|
| 1658 | 5        | 2019-06-20  | returned | London Great example | 'exampl':3 'great':2 'london':1 |
| 1661 | 69       | 2019-05-09  | placed   | London the with      | 'london':1                      |

| QUERY PLAN                                                                                                                                          |
|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| Gather  (cost=1000.00..223054.67 rows=5000 width=34) (actual time=5.820..1294.741 rows=200352 loops=1)                                              |
| Workers Planned: 2                                                                                                                                  |
| Workers Launched: 2                                                                                                                                 |
| ->  Parallel Seq Scan on "FullTextSearchingIndexTable2"  (cost=0.00..221554.67 rows=2083 width=34) (actual time=3.659..1251.419 rows=66784 loops=3) |
| Filter: (to_tsvector(some_text) @@ to_tsquery('london'::text))                                                                                      |
| Rows Removed by Filter: 266549                                                                                                                      |
| Planning Time: 0.468 ms                                                                                                                             |
| JIT:                                                                                                                                                |
| Functions: 6                                                                                                                                        |
| Options: Inlining false, Optimization false, Expressions true, Deforming true                                                                       |
| Timing: Generation 8.825 ms, Inlining 0.000 ms, Optimization 0.666 ms, Emission 9.634 ms, Total 19.126 ms                                           |
| Execution Time: 1300.920 ms                                                                                                                         |
                                                                                                          |
Не особо помогло...

### Частичный индекс

12. Запустил скрипт:
```SQL
explain analyze
select *
from hwindex."PartialIndex" 
where id = 100;
```
**Результат (Далее только план запроса)**

| QUERY PLAN                                                                                                                |
|---------------------------------------------------------------------------------------------------------------------------|
| Gather  (cost=1000.00..14221.43 rows=1 width=34) (actual time=0.642..53.548 rows=1 loops=1)                               |
| Workers Planned: 2                                                                                                        |
| Workers Launched: 2                                                                                                       |
| ->  Parallel Seq Scan on "PartialIndex"  (cost=0.00..13221.33 rows=1 width=34) (actual time=5.537..21.674 rows=0 loops=3) |
| Filter: (id = 100)                                                                                                        |
| Rows Removed by Filter: 333333                                                                                            |
| Planning Time: 0.095 ms                                                                                                   |
| Execution Time: 53.584 ms                                                                                                 |
                                                                                           |
                                                                                        |

13. Создал индекс на таблицу, на столбец id:
```SQL
create index idx_PartialIndex_id_100 on hwindex."PartialIndex" (id) where id > 10;
```
14. Запустил скрипт:
```SQL
explain analyze
select *
from hwindex."PartialIndex" 
where id = 100;
```
**Результат (план запроса)**

| QUERY PLAN                                                                                                                                  |
|---------------------------------------------------------------------------------------------------------------------------------------------|
| Limit  (cost=0.42..8.44 rows=1 width=34) (actual time=0.070..0.072 rows=1 loops=1)                                                          |
| ->  Index Scan using idx_partialindex_id_100 on "PartialIndex"  (cost=0.42..8.44 rows=1 width=34) (actual time=0.066..0.066 rows=1 loops=1) |
| Index Cond: (id = 100)                                                                                                                      |
| Planning Time: 0.910 ms                                                                                                                     |
| Execution Time: 0.108 ms                                                                                                                    |
                                                                                         |

Результат лучше, но не значительно.

### Составной индекс
15. Запустил скрипт:
```SQL
explain analyze
select *
from hwindex."MulticolumnIndexes" 
where id = 10 AND user_id = 40
```
**Результат (план запроса)**

| QUERY PLAN                                                                                                                       |
|----------------------------------------------------------------------------------------------------------------------------------|
| Gather  (cost=1000.00..15263.10 rows=1 width=34) (actual time=38.928..40.597 rows=0 loops=1)                                     |
| Workers Planned: 2                                                                                                               |
| Workers Launched: 2                                                                                                              |
| ->  Parallel Seq Scan on "MulticolumnIndexes"  (cost=0.00..14263.00 rows=1 width=34) (actual time=31.062..31.063 rows=0 loops=3) |
| Filter: ((id = 10) AND (user_id = 40))                                                                                           |
| Rows Removed by Filter: 333333                                                                                                   |
| Planning Time: 0.134 ms                                                                                                          |
| Execution Time: 40.627 ms                                                                                                        |

16. Создал индекс на таблицу, на столбец id:
```SQL
create index idx_SimpleIndex_id on hwindex."SimpleIndex"(id);
```
17. Запустил скрипт:
```SQL
explain analyze
select *
from hwindex."MulticolumnIndexes" 
where id = 10 AND user_id = 40
```
**Результат (план запроса)**

| QUERY PLAN                                                                                                                                                  |
|-------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Limit  (cost=0.42..8.45 rows=1 width=34) (actual time=0.150..0.152 rows=0 loops=1)                                                                          |
| ->  Index Scan using idx_multicolumnindexes_id_user_id on "MulticolumnIndexes"  (cost=0.42..8.45 rows=1 width=34) (actual time=0.148..0.148 rows=0 loops=1) |
| Index Cond: ((id = 10) AND (user_id = 40))                                                                                                                  |
| Planning Time: 0.465 ms                                                                                                                                     |
| Execution Time: 0.186 ms                                                                                                                                    |

