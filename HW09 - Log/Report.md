# Отчет о выполнении домашнего задания
## Тема: Журналы
1. Открыл файл postgressql.conf и выставил параметр checkpoint_timeout = 30s. Далее для вступления изменений в силу выполнил комманду:
```CMD
 pg_ctlcluster 15 main restart
```
2. В связи с ошибкой о которой ранее писал пришлось выставить следущие параметры:

| Параметр       | Значение |
 |----------------|----------|
| max_wal_size   | 4GB      |
| min_wal_size   | 1GB      |
| wal_keep_size  | 3GB      |
Значения можно было поставить и меньше, но чтобы точно все заработало было принято решение выставить эти значения

3. Проверил текущий lsn:
```SQL
select pg_current_wal_lsn();
```
**Результат**

| pg_current_wal_lsn |
|--------------------|
| 1/3D463F48         |

4. Проверил wal файл по текущему lsn:
```SQL
select pg_walfile_name('1/3D463F48'); 
```
**Результат**

| pg_walfile_name          |
|--------------------------|
| 00000001000000010000003D |

5. Проверил наличие данного wal файла:
```SQL
select * from pg_ls_waldir()
where name ='00000001000000010000003D'
```
**Результат**

| name                     | size     | modification                  |
|--------------------------|----------|-------------------------------|
| 00000001000000010000003D | 16777216 | 2024-11-08 10:12:47.000 +0700 |

5. Проверил наличие данного wal файла:
```SQL
select * from pg_ls_waldir()
where name ='00000001000000010000003D'
```
**Результат**

| name                     | size     | modification                  |
|--------------------------|----------|-------------------------------|
| 00000001000000010000003D | 16777216 | 2024-11-08 10:12:47.000 +0700 |

6. Проверил текущий журнал:
```SQL
select * from pg_stat_bgwriter;
```
**Результат**

| checkpoints_timed | checkpoints_req | checkpoint_write_time | checkpoint_sync_time | buffers_checkpoint | buffers_clean | maxwritten_clean | buffers_backend | buffers_backend_fsync | buffers_alloc | stats_reset                    |
|-------------------|-----------------|-----------------------|----------------------|--------------------|---------------|------------------|-----------------|-----------------------|---------------|--------------------------------|
| 964               | 21              | 7982876.0             | 706.0                | 383013             | 0             | 0                | 54145           | 0                     | 77153         | 2024-11-06 07:35:38.943 +0700  |

Важными будут параметры *checkpoints_timed* и *checkpoints_req* тк они показывают количество конторольных точек по рассписанию и нет.

***PS*** Количество большое из-за большого количества экспериментов связанных с данной дз.

7. Инициализировал pgbench на БД postgres, запустил pgbecnh на 10 минут
```CMD
 sudo -u postgres pgbench -i postgres
 sudo -u postgres pgbench -P 1 -T 600 postgres
```
### Описание параметров команды

| Параметр       | Значение                                                    |
|----------------|-------------------------------------------------------------|
| -U  (username) | Имя пользователя для подключения                            |
| -P  (progress) | Время вывода отчёта о прогрессе через заданное число секунд |
| -T  (time)     | Ограничение выполнения теста по заданному времени           |

8. По завершению pgbench проверил текущий lsn:
```SQL
select pg_current_wal_lsn();
```
**Результат**

| pg_current_wal_lsn |
|--------------------|
| 1/6DB73618         |

9. Проверил wal файл по текущему lsn:
```SQL
select pg_walfile_name('1/6DB73618'); 
```
**Результат**

| pg_walfile_name          |
|--------------------------|
| 00000001000000010000006D |

10. Проверил наличие данного wal файла и первого wal файла:
```SQL
select * from pg_ls_waldir()
where name in ('00000001000000010000006D', '00000001000000010000003D')
```
**Результат**

| name                     | size     | modification                  |
|--------------------------|----------|-------------------------------|
| 00000001000000010000006D | 16777216 | 2024-11-08 11:34:16.000 +0700 |
| 00000001000000010000003D | 16777216 | 2024-11-08 11:23:31.000 +0700 |

11. Проверил текущий журнал:
```SQL
select * from pg_stat_bgwriter;
```
**Результат**

| checkpoints_timed | checkpoints_req |
|-------------------|-----------------|
| 985               | 21              | 

Из журнала видно, что по расписанию спистя 10 минут выполнилась 21 контрольная точка.
Должно было быть 20, скорее всего данная проблема вызвана поздним вызовом журнала
тк перед этим были вызваны комманды указанные выше.
Не по расписанию было вызвано 0 контрольных точек, что удивительно.
Как я понимаю контрольные точки вызванные не по рассписанию вызываются при слишком большой нагрузке, когда одна контрольная точка не успела записаться.

12. Проверил общий объем между двумя lsn:
```SQL
select '1/6DB73618'::pg_lsn - '1/3D463F48'::pg_lsn as bytes;
```
**Результат**

| bytes     |
|-----------|
| 812709584 |
Итого 812 МБ. мне известно, что контрольных точек между этими lsn = 20.
По простой формуле можно вычеслить среднее значение на каждую контрольную точку = 812/20 = 40,6 МБ

13. Проверил tps при синхронном режиме. tps = 2018.054197
14. В файле postgres.conf установил synchronous_commit = off.
15. Перезапустил postgres и запустил pgbecnh на 10 минут
```CMD
sudo pg_ctlcluster 15 main restart
sudo -u postgres pgbench -P 1 -T 600 postgres
```
16. Проверил tps при асинхронном режиме. tps = 2500.540323. tps вырос. 