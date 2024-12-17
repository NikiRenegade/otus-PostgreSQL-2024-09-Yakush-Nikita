# Отчет о выполнении домашнего задания
## Тема: Физический уровень PostgreSQL
### Основное задание
1. Создал виртуальную машину, установил PostgreSQL 15, выдал доступ из вне
2. Проверил работоспособность 
3. Выполнил комманду:
```SQL
create table test(c1 text);

insert
into
    test
values('1');
```
4. Вышел из psql (\q) и остановил postgres коммандой: 
```CMD
sudo -u postgres pg_ctlcluster 15 main stop
```
5. создайте новый диск для ВМ размером 10GB через Control center Parallels
6. Проинициализировал диск  и подмонтировал файловую систему согласно инструкции 
[Как разделить и отформатировать устройства хранения данных в Linux](https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux)
7. Перезагрузил ВМ - диск остался примонтирован
8. Сделал пользователя postgres владельцем /mnt/data коммандой:
```CMD
chown -R postgres:postgres /mnt/data/
```
9. Перенес директорию /var/lib/postgres/15 в /mnt/data коммандой: 
```CMD
mv /var/lib/postgresql/15 /mnt/data
```
10. Запустить кластер очевидно не вышло тк при запуске он ссылается на: 

который находится в:
```CMD
/etc/postgresql/15/main/postgresql.conf
```
Поэтому в данном файле подменил путь на корректный и запустил кластер командой:
```CMD
sudo -u postgres pg_ctlcluster 15 main start
```
11. Проверил содержимое данных. Все сохранилось
12. Вернул все дефолтное состояние
### Задание *
1. На перавой ВМ (которая была в основном задании) скопировал директорию /var/lib/postgres/15 в /mnt/data коммандой:
```CMD
sudo cp -r /var/lib/postgresql/15 /mnt/data
```
2. Создал виртуальную машину, установил PostgreSQL 15, выдал доступ из вне
3. Проверил работоспособность 
4. Удалил директорию /var/lib/postgresql/15 коммандой
```CMD
rm -r /var/lib/postgresql/15
```
5. Перемонтировал внешний диск от первой ВМ ко второй
6. Из инструкции [Как разделить и отформатировать устройства хранения данных в Linux](https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux)
проделал шаги 4 и 5, чтобы не запутаться каталог назвал mnt1/data1
7. Cкопировал директорию /mnt1/data1/15 в /var/lib/postgres в  коммандой:
```CMD
sudo cp -r /mnt1/data1/postgresql/15  /var/lib/postgresql
```
8. Сделал пользователя postgres владельцем новой /var/lib/postgresql/15/main коммандой:
```CMD
chown -R postgres:postgres /var/lib/postgresql/15/main
```
9. Запустил кластер коммандой:
```CMD
sudo -u postgres pg_ctlcluster 15 main start
```
10. В итоге информация которую вводили на первой ВМ передалась и во вторую

**P.S.** Вместо копирования директории в "/var/lib/postgresql" (пункт 7) можно было 
подменить информацию в файле "postgresql.conf" как это было сделано в основном 
задании и итоговый результат остался бы не изменным.
