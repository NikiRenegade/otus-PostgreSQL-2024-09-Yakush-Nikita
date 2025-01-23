# Развернутый отчет о проектной работе
## Реализация высоконагруженных и отказоустойчивых кластеров PostgreSQL на базе Patroni
### <a id="con">Содержание</a>

1. [Настройка DCS](#dcs)  
2. [Настройка PostgreSQL](#pos)
3. [Настройка Potroni](#pat)
4. [Настройка HAProxy](#hap)

Для данной работы были использованы 5 ВМ созданных в ПО "Parallels Desctop"

**Описание ВМ**

| Название ОС   | Версия ОС       | ip ОС        | Пользователь для демонстрации |
|---------------|-----------------|--------------|-------------------------------|
| ubuntu1       | Ubuntu 20.04.5  | 10.211.55.47 | administrator                 |
| ubuntu2       | Ubuntu 20.04.5  | 10.211.55.48 | administrator                 |
| ubuntu3       | Ubuntu 20.04.5  | 10.211.55.49 | administrator                 |
| ubuntuhaproxy | Ubuntu 20.04.5  | 10.211.55.50 | administrator                 |
### <a id="dcs">Настройка DCS</a>
[Вернуться к содержанию](#con)  
DCS – распределенная система хранения конфигурации,
в которой реализован протокол консенсуса для достижения консистентности
хранимых данных на всех узлах таковой системы.
Примеры DCS:
- [ ] Etcd
- [ ] ZooKeeper
- [x] Consul

Мною был выбран consul тк он без лишних проблем может находится вместе с postgre на одной ВМ.

1. Загрузка бинарного файла consul на три ВМ (DebianMaster, DebianReplicaFirst, DebianReplicaSecond).
```CMD
wget https://releases.hashicorp.com/consul/1.14.3/consul_1.14.3_linux_arm64.zip
unzip consul_1.14.3_linux_arm64.zip
```
2. Перенос consul из папки tmp в /usr/bin/ и предоставление для файла права на исполнение на всех трех ВМ:
```CMD
sudo mv consul /usr/bin && sudo chmod +x /usr/bin/consul
```
3. Создание каталогов для consul, назначение владельца и выдача прав на всех трех ВМ:
```CMD
sudo mkdir -p /var/lib/consul /etc/consul.d && sudo chown administrator:administrator /var/lib/consul /etc/consul.d && sudo chmod 775 /var/lib/consul /etc/consul.d
```
4. Создание конфигурационного файла (json) и service файла Consul для всех трех ВМ:
```CMD
consul keygen
sudo nano  /etc/consul.d/config.json
sudo nano /etc/systemd/system/consul.service
```
P.S. Все файлы конфигурации [можно найти здесь](https://github.com/NikiRenegade/otus-PostgreSQL-2024-09-Yakush-Nikita/blob/main/Project/Configuration.md)

5. Запуск Consul
```CMD
sudo systemctl daemon-reload && sudo systemctl enable consul && sudo systemctl start consul
```
6. Проверка работоспособности Consul
```CMD
sudo systemctl status consul        --Active и без ошибок
consul members                      --Должно быть все 3 ноды
journalctl -u consul -f             --Просмотр журнала
```

P.S.
Во время настройки Consul были проблемы
(скорее всего из-за неправильного времени у одной из node),
а также в consul.service одинаковое имя нод реплик.
В связи с чем у меня было всего 2 ноды, что приводило к тому,
что лидер не мог быть определен, а комманда consul operator raft list-peers
приводила к ошибке.

### <a id="pos">Настройка PostgreSQL 16</a>
[Вернуться к содержанию](#con) 

1. Добавление репозитория PostgreSQL на 3 ВМ:
```CMD
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
```
2. Добавление ключа GPG:
```CMD
curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
```
3. Обновление список пакетов:
```CMD
sudo apt update
```
4. Установка PostgresSQL 16:
```CMD
sudo apt install postgresql-16 postgresql-contrib-16
```
5. Запуск PostgresSQL 16:
```CMD
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

### <a id="pat">Настройка Potroni</a>
[Вернуться к содержанию](#con) 
1. Удаление кластер postgres:
```CMD
sudo systemctl stop postgresql@16-main && sudo -u postgres pg_dropcluster 16 main
```
2. Установка Python 3, pip и Git:
```CMD
sudo apt install -y python3 python3-pip git 
```
3. Установка psycopg2-binary:
```CMD
sudo pip3 install psycopg2-binary
```
4. Установка Patroni с поддержкой Consul:
```CMD
sudo pip3 install patroni[consul]
```
5. Создание ссылки для Patroni:
```CMD
sudo ln -s /usr/local/bin/patroni /bin/patroni
```
6. Создание конфигурационного файла (yml) и service файла patroni для всех трех ВМ:
```CMD
sudo nano /etc/systemd/system/patroni.service
sudo nano /etc/patroni.yml
```
P.S. Все файлы конфигурации [можно найти здесь](https://github.com/NikiRenegade/otus-PostgreSQL-2024-09-Yakush-Nikita/blob/main/Project/Configuration.md)
7. Заупуск Patroni
```CMD
sudo systemctl enable patroni && sudo systemctl start patroni 
```

### <a id="hap">Настройка HAProxy</a>
[Вернуться к содержанию](#con) 
1. Установка HAProxy
```CMD
sudo systemctl enable patroni && sudo systemctl start patroni 
```
2. Открытие конфигурационного файла:
```CMD
sudo nano /etc/haproxy/haproxy.cfg
```
Суть была сделать так чтобы один порт - чтение, другой - запись
P.S. Все файлы конфигурации [можно найти здесь](https://github.com/NikiRenegade/otus-PostgreSQL-2024-09-Yakush-Nikita/blob/main/Project/Configuration.md)
3. Заупуск HAProxy
```CMD
sudo systemctl start haproxy
sudo systemctl enable haproxy
```
