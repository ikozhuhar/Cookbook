## Мониторинг

<img width="878" height="510" alt="image" src="https://github.com/user-attachments/assets/52c3175f-dce5-405c-8c09-fe35c35cb892" />

### Установка Prometheus

```ruby
$ wget https://github.com/.../prometheus-2.45.3.linux-amd64.tar.gz
$ tar xf prometheus-*.tar.gz

$ cd prometheus-*
$ cp prometheus promtool /usr/local/bin
$ mkdir /etc/prometheus /var/lib/prometheus
$ cp prometheus.yml /etc/prometheus
$ useradd --no-create-home --home-dir / --shell /bin/false prometheus
$ chown -R prometheus:prometheus /var/lib/prometheus

# В архиве будут два ненужных каталога: `console_libraries/` и `consoles/`. Их не трогаем.
```

_Создаём systemd-юнит: `/etc/systemd/system/prometheus.service`_

```ruby
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Restart=on-failure
ExecStart=/usr/local/bin/prometheus \
  --config.file /etc/prometheus/prometheus.yml \
  --storage.tsdb.path /var/lib/prometheus/
ExecReload=/bin/kill -HUP $MAINPID
ProtectHome=true
ProtectSystem=full

[Install]
WantedBy=default.target
```

_Запускаем Prometheus server:_

```ruby
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl status prometheus
```


_Файл `/etc/prometheus/prometheus.yml`_

```ruby
global:
  scrape_interval:     15s
  evaluation_interval: 15s
  scrape_timeout: 8s

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
       - localhost:9093

rule_files:
  - "first_rules.yml"
  - "second_rules.yml"

scrape_configs:
  - job_name: 'node'
    static_configs:
    - targets:
       - localhost:9100
     # - anotherhost:9100
```

_Теперь можно сходить на `http://localhost:9090`_



<br>

### Установка node exporter

```ruby
sudo wget https://github.com/.../node_exporter-0.18.1.linux-amd64.tar.gz
sudo tar xf node_exporter-*.tar.gz
sudo cd node_exporter-*
sudo cp node_exporter /usr/local/bin
sudo useradd --no-create-home --home-dir / --shell /bin/false node_exporter

_Создаём systemd-юнит: `/etc/systemd/system/node_exporter.service`_

```ruby
[Unit]
Description=Prometheus Node Exporter
After=network.target

[Service]
Type=simple
User=node_exporter
Group=node_exporter
ExecStart=/usr/local/bin/node_exporter

SyslogIdentifier=node_exporter
Restart=always

PrivateTmp=yes
ProtectHome=yes
NoNewPrivileges=yes

ProtectSystem=strict
ProtectControlGroups=true
ProtectKernelModules=true
ProtectKernelTunables=yes

[Install]
WantedBy=multi-user.target
```

**Важно!** Если у вас `/home` вынесен на отдельный раздел, директиву `ProtectHome=yes` надо убрать, иначе `node exporter` будет неправильно показывать оставшееся место на разделе `/home`.

_Запускаем node exporter:_

```ruby
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl status node_exporter
```

_Теперь можно сходить на `http://localhost:9100/metrics`_

