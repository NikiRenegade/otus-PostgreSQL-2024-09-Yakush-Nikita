#  Файлы конфигурации для HAProxy


| Название ОС   | ip ОС        |
|---------------|--------------|
| ubuntuhaproxy | 10.211.55.50 | 

## /etc/haproxy/haproxy.cfg
```CFG
 global
    maxconn 200

defaults
    log    global
    mode    tcp
    retries 2
    timeout client 30m
    timeout connect 4s
    timeout server 30m
    timeout check 5s

listen primary
    bind *:5432
    option httpchk OPTIONS /leader
    http-check expect status 200
    default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions
    server ubuntu1 10.211.55.47:5432 maxconn 100 check port 8008
    server ubuntu2 10.211.55.48:5432 maxconn 100 check port 8008
    server ubuntu3 10.211.55.49:5432 maxconn 100 check port 8008

listen replica
    balance roundrobin
    bind *:5433
    option httpchk OPTIONS /replica
    http-check expect status 200
    default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions
    server ubuntu1 10.211.55.47:5432 maxconn 100 check port 8008
    server ubuntu2 10.211.55.48:5432 maxconn 100 check port 8008
    server ubuntu3 10.211.55.49:5432 maxconn 100 check port 8008

listen stats
    mode http
    bind *:5000
    stats enable
    stats uri /
```
### Описание конфигурации
- global - Содержит глобальные параметры для HAProxy, которые применяются ко всем настройкам.
  - maxconn — Указывает максимальное количество соединений, которые HAProxy может обрабатывать одновременно.
- defaults - Содержит параметры по умолчанию, применяемые ко всем секциям listen, если для них не указаны индивидуальные настройки.
  - log — Логи для секций
  - mode — Указывает режим работы HAProxy.
  - retries — Количество попыток подключения к серверу перед тем, как будет зафиксирована ошибка.
  - timeout client — Максимальное время ожидания активности от клиента, после чего соединение разрывается.
  - timeout connect — Максимальное время ожидания подключения к серверу.
  - timeout server — Максимальное время ожидания ответа от сервера.
  - timeout check — Максимальное время ожидания проверки работоспособности узлов.
- listen primary - Секция для управления подключениями к основному узлу кластера PostgreSQL (primary).
  - bind — Указывает, что HAProxy будет слушать входящие соединения на порту 5432.
  - option httpchk OPTIONS — Проверяет состояние узлов с помощью HTTP-запроса OPTIONS на путь /leader.
  - http-check expect status  — Узел считается работоспособным, если ответ HTTP-запроса имеет код 200.
  - default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions:
      - inter — Интервал проверки состояния узла (каждые 3 секунды).
      - fall — Узел помечается как недоступный, если три проверки подряд завершаются неуспешно.
      - rise — Узел помечается как доступный, если две проверки подряд успешны.
      - on-marked-down shutdown-sessions — Отключает все существующие сессии при пометке узла как недоступного.
  - server ubuntu1 10.211.55.47:5432 maxconn 100 check port 8008 — Указывает сервер PostgreSQL с адресом 10.211.55.47 и портом 5432:
      - maxconn — Максимальное количество одновременных соединений.
      - check port — Использует порт 8008 для проверки состояния узла.
- listen stats - Секция для настройки веб-интерфейса HAProxy.
  - mode — Указывает в каком формате работает сессия (В данном конфиге HTTP).
  - *bind — Интерфейс доступен на порту 5000.
  - stats — Включает отображение статистики.
  - stats  / — Указывает путь для доступа к веб-интерфейсу.