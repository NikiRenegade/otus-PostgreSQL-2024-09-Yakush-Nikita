1. Демострация презентации
2. Демонстрация в консоле 4ех ВМ
3. Демонстрация консула в web интерфейсе 
4. Рассказ про HAProxy

5. Привести пример
create database otus
create schema otus
https://github.com/NikiRenegade/otus-PostgreSQL-2024-09-Yakush-Nikita/blob/main/HW23%20-%20statistics%20(Join)/Report.md
drop database otus



curl -i http://10.211.55.48:8008/leader
patronictl -c /etc/patroni.yml switchover patroni

psql -h 10.211.55.50 -p 5432 -U postgres -d postgres

consul members