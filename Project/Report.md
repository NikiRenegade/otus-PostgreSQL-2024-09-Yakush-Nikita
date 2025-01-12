# Развернутый отчет о проектной работе
## Реализация высоконагруженных и отказоустойчивых кластеров PostgreSQL на базе Patroni

Для данной работы были использованы 5 ВМ созданных в ПО "Parallels Desctop"

**Описание ВМ**

| Название ОС         | Версия ОС            | ip ОС        | Пользователь для демонстрации |
|---------------------|----------------------|--------------|-------------------------------|
| DebianMaster        | Debian GNU/Linux 12  | 10.211.55.29 | admin                         |
| DebianReplicaFirst  | Debian GNU/Linux 12  | 10.211.55.30 | admin                         |
| DebianReplicaSecond | Debian GNU/Linux 12  | 10.211.55.31 | admin                         |
| DebianHaProxy       | Debian GNU/Linux 12  | 10.211.55.27 | admin                         |
| DebianGrafana       | Debian GNU/Linux 12  | 10.211.55.28 | admin                         |

### Настройка DCS
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
sudo mv /tmp/consul /usr/bin
sudo chmod +x /usr/bin/consul
```
3. Создание каталогов для consul, назначение владельца и выдача прав на всех трех ВМ:
```CMD
sudo mkdir -p /var/lib/consul /etc/consul.d
sudo chown admin:admin /var/lib/consul /etc/consul.d
sudo chmod 775 /var/lib/consul /etc/consul.d
```
4. Создание конфигурационного файла  (json) и файла сервиса (service) consul для всех трех ВМ:
```CMD
sudo nano  /etc/consul.d/config.json
sudo nano /etc/systemd/system/consul.service
```
P.S. Все файлы конфигурации

5. Запуск Consul
```CMD
sudo systemctl daemon-reload
sudo systemctl start consul
sudo systemctl enable consul
```
6. Проверка работоспособности Consul
```CMD
sudo systemctl status consul        --Active и без ошибок
consul members                      --Должно быть все 3 ноды
consul operator raft list-peers     --Посмотреть лидера
journalctl -u consul -f             --Просмотр журнала
```

P.S.
Во время настройки Consul были проблемы
(скорее всего из-за неправильного времени у одной из node),
а также в consul.service одинаковое имя нод реплик.
В связи с чем у меня было всего 2 ноды, что приводило к тому,
что лидер не мог быть определен, а комманда consul operator raft list-peers
приводила к ошибке.

