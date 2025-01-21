#  Файлы конфигурации для Patroni

| Название ОС   | ip ОС         |
|---------------|---------------|
| ubuntu1       | 10.211.55.47  | 

## /etc/systemd/system/patroni.service
```Service
[Unit]
Description=High availability PostgreSQL Cluster
After=syslog.target network.target

[Service]
Type=simple
User=postgres
Group=postgres
ExecStart=/usr/local/bin/patroni /etc/patroni.yml
KillMode=process
TimeoutSec=30
Restart=no

[Install]
WantedBy=multi-user.target
```
## /etc/patroni.yml
```YML
scope: patroni
name: ubuntu1
restapi:
  listen: 10.211.55.47:8008
  connect_address: 10.211.55.47:8008
consul:
  host: "localhost:8500"
  register_service: true
bootstrap:
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 10
    maximum_lag_on_failover: 1048576
    postgresql:
      use_pg_rewind: true
      parameters:
  initdb: 
  - encoding: UTF8
  - data-checksums
  pg_hba: 
  - host replication replicator 0.0.0.0/0 scram-sha-256
  - host all all 0.0.0.0/0 scram-sha-256
  users:
    admin:
      password: admin123
      options:
        - createrole
        - createdb
postgresql:
  listen: 127.0.0.1, 10.211.55.47:5432
  connect_address: 10.211.55.47:5432
  data_dir: /var/lib/postgresql/16/main
  bin_dir: /usr/lib/postgresql/16/bin
  pgpass: /tmp/pgpass0
  authentication:
    replication:
      username: replicator
      password: replicator123
    superuser:
      username: postgres
      password: postgres123
    rewind:  
      username: rewind_user
      password: user123
  parameters:
    unix_socket_directories: '.'
tags:
    nofailover: false
    noloadbalance: false
    clonefrom: false
    nosync: false
```

### Описание конфигурации
- scope - Указывает имя кластера Patroni. Все узлы кластера должны иметь одинаковое значение scope.
- name - Указывает уникальное имя текущего узла.
- restapi - Этот раздел управляет настройками REST API, используемого Patroni
- consul - Этот раздел управляет взаимодействием Patroni с Consul
- bootstrap - Этот раздел отвечает за начальную инициализацию кластера.
- bootstrap.dcs - Этот раздел указывает настройки для DCS (в данном случае Consul)
- bootstrap.pg_hba - Этот раздел указывает список правил для pg_hba.conf
- postgresql - Этот раздел определяет конфигурацию PostgreSQL
- tags - Этот раздел позволяет задавать особые роли для узлов