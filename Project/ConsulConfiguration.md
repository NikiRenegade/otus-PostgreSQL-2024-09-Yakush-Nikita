#  Файлы конфигурации для Consul

| Название ОС   | ip ОС         |
|---------------|---------------|
| ubuntu1       | 10.211.55.47  | 
## /etc/consul.d/config.json
### Моя конфигурация
```JSON
{
  "bind_addr": "10.211.55.47",
  "bootstrap_expect": 3,
  "client_addr": "0.0.0.0",
  "data_dir": "/var/lib/consul",
  "enable_script_checks": true,
  "dns_config": {
    "enable_truncate": true,
    "only_passing": true
  },
  "enable_syslog": true,
  "encrypt": "consul keygen",
  "leave_on_terminate": true,
  "log_level": "INFO",
  "rejoin_after_leave": true,
  "retry_join": [
    "10.211.55.48",
    "10.211.55.49"
  ],
  "server": true,
  "start_join": [
    "10.211.55.47",
    "10.211.55.48",
    "10.211.55.49"
  ],
  "ui_config": { "enabled": true }
}
```
### Описание конфигурации
- bind_addr — IP-адрес, на котором Consul будет слушать входящие соединения.
- bootstrap_expect — Указывает количество серверов, которые должны быть доступны кластере.
- client_addr — IP-адрес, на котором Consul будет принимать клиентские соединения.
- data_dir — Каталог, где Consul хранит данные.
- enable_script_checks — Разрешение выполнения проверки работоспособности.
- dns_config - Конфигурация DNS.
    - enable_truncate — Разрешение сокращения ответов DNS.
    - only_passing — Указание возвращать только узлы, которые проходят проверки работоспособности.
- enable_syslog — Разрешение отправки логов Consul в системный лог.
- encrypt — Ключ шифрования для безопасного взаимодействия между узлами Consul.
- leave_on_terminate — Указание Consul при получении сигнала на остановку процесса, корректно отключать ноду от кластера.
- log_level — Уровень детализации логов.
- rejoin_after_leave — Позволение узлу повторно присоединяться к кластеру после временного выхода.
- retry_join — Список серверов, к которым Consul будет пытаться подключаться в случае потери связи.
- server — Указывает, что узел является сервером (а не клиентом) в кластере.
- start_join — Список серверов, к которым узел пытается подключиться при запуске.
- ui_config:
    - enabled — Включает или отключает веб-интерфейс Consul.

### Минимаьная конфигурация
```JSON
{
  "bind_addr": "10.211.55.47",
  "bootstrap_expect": 3,
  "data_dir": "/var/lib/consul",
  "encrypt": "consul keygen",
  "log_level": "INFO",
  "retry_join": [
    "10.211.55.48",
    "10.211.55.49"
  ],
  "server": true,
  "ui_config": { "enabled": true }
}
}
```
### /etc/systemd/system/consul.service
```Service
[Unit]
Description=Consul Service Discovery Agent
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=administrator
ExecStart=/usr/bin/consul agent \
    -node=consul_server1.dmosk.local \
    -config-dir=/etc/consul.d
ExecReload=/bin/kill -HUP $MAINPID
KillSignal=SIGINT
TimeoutStopSec=5
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
