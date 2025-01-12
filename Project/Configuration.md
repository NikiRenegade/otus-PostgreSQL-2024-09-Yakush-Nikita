# Все файлы конфигурации для проектной работы

## Consul

### /etc/consul.d/config.json

#### DebianMaster
```CMD
{
  "bind_addr": "10.211.55.29",
  "bootstrap_expect": 3,
  "client_addr": "0.0.0.0",
  "data_dir": "/var/lib/consul",
  "enable_script_checks": true,
  "dns_config": {
    "enable_truncate": true,
    "only_passing": true
  },
  "enable_syslog": true,
  "encrypt": "oneS8h79WporNo4iJOkOSnY+7N36xsL6AOmOVE+xlO8=",
  "leave_on_terminate": true,
  "log_level": "INFO",
  "rejoin_after_leave": true,
  "retry_join": [
    "10.211.55.30",
    "10.211.55.31"
  ],
  "server": true,
  "start_join": [
    "10.211.55.29",
    "10.211.55.30",
    "10.211.55.31"
  ],
  "ui_config": { "enabled": true }
}
```
#### DebianReplica1
```CMD
{
  "bind_addr": "10.211.55.30",
  "client_addr": "0.0.0.0",
  "data_dir": "/var/lib/consul",
  "enable_script_checks": true,
  "dns_config": {
    "enable_truncate": true,
    "only_passing": true
  },
  "enable_syslog": true,
  "encrypt": "oneS8h79WporNo4iJOkOSnY+7N36xsL6AOmOVE+xlO8=",
  "leave_on_terminate": true,
  "log_level": "INFO",
  "rejoin_after_leave": true,
  "retry_join": [
    "10.211.55.29",
    "10.211.55.31"
  ],
  "server": true,
  "start_join": [
    "10.211.55.29",
    "10.211.55.30",     
    "10.211.55.31"
  ],
  "ui_config": { "enabled": true }
}
```
#### DebianReplica2
```CMD
{
  "bind_addr": "10.211.55.31",
  "client_addr": "0.0.0.0",
  "data_dir": "/var/lib/consul",
  "enable_script_checks": true,
  "dns_config": {
    "enable_truncate": true,
    "only_passing": true
  },
  "enable_syslog": true,
  "encrypt": "oneS8h79WporNo4iJOkOSnY+7N36xsL6AOmOVE+xlO8=",
  "leave_on_terminate": true,
  "log_level": "INFO",
  "rejoin_after_leave": true,
  "retry_join": [
    "10.211.55.29",
    "10.211.55.30"
  ],
  "server": true,
  "start_join": [
    "10.211.55.29",
    "10.211.55.30",     
    "10.211.55.31"
  ],
  "ui_config": { "enabled": true }
}
```

### /etc/systemd/system/consul.service

#### DebianMaster
```CMD
[Unit]
Description=Consul Service Discovery Agent
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=admin
ExecStart=/usr/bin/consul agent \
    -node=consul_master.dmosk.local \
    -config-dir=/etc/consul.d
ExecReload=/bin/kill -HUP $MAINPID
KillSignal=SIGINT
TimeoutStopSec=5
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
#### DebianReplica1
```CMD
[Unit]
Description=Consul Service Discovery Agent
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=admin
ExecStart=/usr/bin/consul agent \
    -node=consul_replica1.dmosk.local \
    -config-dir=/etc/consul.d
ExecReload=/bin/kill -HUP $MAINPID
KillSignal=SIGINT
TimeoutStopSec=5
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
#### DebianReplica2
```CMD
[Unit]
Description=Consul Service Discovery Agent
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=admin
ExecStart=/usr/bin/consul agent \
    -node=consul_replica2.dmosk.local \
    -config-dir=/etc/consul.d
ExecReload=/bin/kill -HUP $MAINPID
KillSignal=SIGINT
TimeoutStopSec=5
Restart=on-failure

[Install]
WantedBy=multi-user.target
```