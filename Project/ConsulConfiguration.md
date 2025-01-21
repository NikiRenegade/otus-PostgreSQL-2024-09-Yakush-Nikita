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















```CFG
global
	log /dev/log	local0
	log /dev/log	local1 notice
	chroot /var/lib/haproxy
	stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
	stats timeout 30s
	user haproxy
	group haproxy
	daemon

	# Default SSL material locations
	ca-base /etc/ssl/certs
	crt-base /etc/ssl/private

	# See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&config=intermediate
        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
        ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
        ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

defaults
	log	global
	mode	http
	option	httplog
	option	dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
	errorfile 400 /etc/haproxy/errors/400.http
	errorfile 403 /etc/haproxy/errors/403.http
	errorfile 408 /etc/haproxy/errors/408.http
	errorfile 500 /etc/haproxy/errors/500.http
	errorfile 502 /etc/haproxy/errors/502.http
	errorfile 503 /etc/haproxy/errors/503.http
	errorfile 504 /etc/haproxy/errors/504.http
	
frontend pgsql_front
    bind *:5432         
    mode tcp           
    default_backend pgsql_back 

backend pgsql_back
    mode tcp            
    balance roundrobin 
    option tcp-check   

    server ubuntu1 10.211.55.47:5432 check  
    server ubuntu2 10.211.55.48:5432 check 
    server ubuntu3 10.211.55.49:5432 check
    
listen stats
    bind *:8404
    stats enable
    stats uri /
    stats refresh 10s
```


```CMD
global
    maxconn 100

defaults
    log    global
    mode    tcp
    retries 2
    timeout client 30m
    timeout connect 4s
    timeout server 30m
    timeout check 5s

listen stats
    mode http
    bind *:7000
    stats enable
    stats uri /

listen primary
    bind *:5000
    option httpchk OPTIONS /master
    http-check expect status 200
    default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions
    server ubuntu1 10.211.55.47:5432 maxconn 100 check port 8008
    server ubuntu2 10.211.55.48:5432 maxconn 100 check port 8008
    server ubuntu3 10.211.55.49:5432 maxconn 100 check port 8008

listen standbys
    balance roundrobin
    bind *:5001
    option httpchk OPTIONS /replica
    http-check expect status 200
    default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions
    server ubuntu1 10.211.55.47:5432 maxconn 100 check port 8008
    server ubuntu2 10.211.55.48:5432 maxconn 100 check port 8008
    server ubuntu3 10.211.55.49:5432 maxconn 100 check port 8008
```




```CMD
global
        log /dev/log    local0
        log /dev/log    local1 notice
        chroot /var/lib/haproxy
        stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
        stats timeout 30s
        user haproxy
        group haproxy
        daemon

        # Default SSL material locations
        ca-base /etc/ssl/certs
        crt-base /etc/ssl/private

        # See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&config=intermediate
        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DH>
        ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
        ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

defaults
        log     global
        mode    http
        option  httplog
        option  dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http

listen primary
    bind *:5000
    option httpchk OPTIONS /master
    http-check expect status 200
    default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions
    server node1 node1:5432 maxconn 100 check port 8008
    server node2 node2:5432 maxconn 100 check port 8008
    server node3 node3:5432 maxconn 100 check port 8008

listen standbys
    balance roundrobin
    bind *:5001
    option httpchk OPTIONS /replica
    http-check expect status 200
    default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions
    server node1 node1:5432 maxconn 100 check port 8008
    server node2 node2:5432 maxconn 100 check port 8008
    server node3 node3:5432 maxconn 100 check port 8008

listen stats
    bind *:8404
    stats enable
    stats uri /
    stats refresh 10s


```