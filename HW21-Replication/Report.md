# Отчет о выполнении домашнего задания
## Виды и устройство репликации в PostgreSQL. Практика применения 
1. Создал 4 виртуальные машины. Сконфигурировал файлы postgresql.conf, pg_hba.conf. ЧТобы можно было подключаться из вне. 
2. Выстовил 1ой и второй машине wal_level = logical:
```SQL
alter system set wal_level = logical;
```
3. Рестартанул posgres на всех ВМ:
```CMD
sudo systemctl restart postgresql
```
4. В базе первой, второй и третьей ВМ создал таблицы test и test2:
```SQL
create table test (id int);
create table test2 (id int);
```
5. Сделал публикацию таблицы test для 1ой ВМ:
```SQL
create publication test_pub for table test;
```
6. Сделал публикацию таблицы test для 1ой ВМ:
```SQL
create publication test2_pub for table test2;
```
7. Подписался на публикации первой ВМ на второй ВМ:
```SQL
create subscription test_sub 
connection 'host=10.211.55.16 port=5432 user=postgres password=P@$$w0rd dbname=postgres' 
publication test_pub with (copy_data = true);
```
8. Подписался на публикации второй ВМ на певрой ВМ:
```SQL
create subscription test2_sub 
connection 'host=10.211.55.17 port=5432 user=postgres password=P@$$w0rd dbname=postgres' 
publication test2_pub with (copy_data = true);
```
9. Подписался на публикации первой и второй ВМ на третьей ВМ:
```SQL
create subscription test3_sub 
connection 'host=10.211.55.16 port=5432 user=postgres password=P@$$w0rd dbname=postgres' 
publication test_pub with (copy_data = true);
       
create subscription test4_sub 
connection 'host=10.211.55.17 port=5432 user=postgres password=P@$$w0rd dbname=postgres' 
publication test2_pub with (copy_data = true);       
```
10. Для задания со * у третьей ВМ необходимо было поменять конфигурацию файла pg_hba.conf и в нем разрешить репликацию в данный файл добавил следующую строку:
```CMD
host    replication     all             0.0.0.0/0               scram-sha-256
```
11. На четвертой ВМ остановил кластер, удалил папку main (возможно это делать было не нужно), и указал откуда брать бэкап:
```CMD
sudo systemctl stop postgresql
sudo rm -rf /var/lib/postgresql/15/main
sudo -u postgres pg_basebackup -h 10.211.55.18  -p 5432 -R -D /var/lib/postgresql/15/main
sudo systemctl start postgresql
```
12. По данным шагам проверил работоспособность с помощью инстртов на первой и второй ВМ и все работает. На каждой ВМ данные одинаковые