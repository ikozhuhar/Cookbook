## Мониторинг

В этом кратком руководстве описан процесс установки и настройки мониторинга на основе `Prometheus`, `Node exporter` и `Alertmanager` с последующей отправкой алертов в телеграмм. В руководстве рассматриваются два вида установки и настройки - ручная установка и с помощью `Docker`.

#### <a name='toc'>Содержание</a>

1. [Установка Prometheus](#prometheus)
2. [Установка Node Exporter](#node_exporter)
3. [Установка Alertmanager](#alertmanager)



<img width="878" height="510" alt="image" src="https://github.com/user-attachments/assets/52c3175f-dce5-405c-8c09-fe35c35cb892" />

### ✅ <a name='prometheus'>Установка Prometheus</a>

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


_Файл `/etc/prometheus/prometheus.yml`_

```ruby
global:
  scrape_interval:     15s
  evaluation_interval: 15s
  scrape_timeout: 8s

alerting:
  alertmanagers:
  - static_configs:
    - targets:
       - localhost:9093

rule_files:
  - "rules_alerts.yml"
  - "second_rules.yml"

scrape_configs:
  - job_name: 'node'
    static_configs:
    - targets:
       - localhost:9100
       - anotherhost:9100
```


_Файл `rules_alerts.yml`_

```ruby
groups:
- name: vm-availability
  rules:
    - alert: VMDown
      expr: up == 0
      for: 0s
      labels:
        severity: 'critical'
      annotations:
        summary: "Виртуальная машина {{ $labels.hostname }} ({{ $labels.instаncе }}) недоступна!"
        description: "Виртуальная машина {{ $labels.instance }} недоступна!"
        value: "{{ $value }}"
```



_Запускаем Prometheus server:_

```ruby
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl status prometheus
```

_Теперь можно сходить на `http://localhost:9090`_




<br>

### ✅ <a name='node_exporter'>Установка Node Exporter</a>

```ruby
sudo wget https://github.com/.../node_exporter-0.18.1.linux-amd64.tar.gz
sudo tar xf node_exporter-*.tar.gz
sudo cd node_exporter-*
sudo cp node_exporter /usr/local/bin
sudo useradd --no-create-home --home-dir / --shell /bin/false node_exporter
```

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



<br>

### ✅ <a name='alertmanager'>Установка Alertmanager</a>

```ruby
sudo wget https://github.com/.../alertmanager-0.18.0.linux-amd64.tar.gz
sudo tar xf alertmanager-*.tar.gz
sudo cd alertmanager-*
sudo cp alertmanager /usr/local/bin
sudo mkdir /etc/alertmanager /var/lib/alertmanager
sudo cp alertmanager.yml /etc/alertmanager
sudo useradd --no-create-home --home-dir / --shell /bin/false alertmanager
sudo chown -R alertmanager:alertmanager /var/lib/alertmanager
```

_Создаём systemd-юнит: `/etc/systemd/system/alertmanager.service`_

```ruby
[Unit]
Description=Alertmanager for prometheus
After=network.target

[Service]
User=alertmanager
ExecStart=/usr/local/bin/alertmanager \
  --config.file=/etc/alertmanager/alertmanager.yml \
  --storage.path=/var/lib/alertmanager/
ExecReload=/bin/kill -HUP $MAINPID

NoNewPrivileges=true
ProtectHome=true
ProtectSystem=full

[Install]
WantedBy=multi-user.target
```

```ruby
global:
  resolve_timeout: 1s
  telegram_api_url: "https://api.telegram.org"

templates:
- '/etc/alertmanager/templates/*.tmpl'

route:
  receiver: telegram
  group_by: ['alertname']
  group_wait: 0s
  group_interval: 5s
  #repeat_interval: 1h

receivers:
- name: telegram
  telegram_configs:
    - api_url: 'https://api.telegram.org'
      bot_token: 9840143014:BEEkxseb78IYeZsow0GTz82LD4qLtcdxiw4
      chat_id: -1764485087758
      send_resolved: true
      parse_mode: HTML
      message: '{{ template "telegram.message" . }}'
```


_Файл `telegram.tmpl`_

```ruby
{{ define "telegram.message" }}
<b>{{ if eq .Status "firing" }}⚠️ Возникла проблема⚠️{{ else }}✅ Проблема решена ✅{{ end }}</b>

<b>Событие:</b> {{ .CommonLabels.alertname }}
<b>Описание:</b> {{ (index .Alerts 0).Annotations.description }}
<b>Статус:</b> {{ .Status }}
<b>Критичность:</b> {{ .CommonLabels.severity }}
<b>Виртуальная машина:</b> {{ .CommonLabels.instance }}

<b>Время начала:</b> {{ (index .Alerts 0).StartsAt.Local.Format "02-01-2006 15:04:05 MST" }}
{{ end }}
```


_Запускаем `Alertmanager`:_

```ruby
sudo systemctl daemon-reload
sudo systemctl start alertmanager
sudo systemctl status alertmanager
```

_Теперь можно сходить на http://localhost:9093/metrics_

На этом установка и настройка завершина. Для теста, можно "погасить" VM указанную в файле `/etc/prometheus/prometheus.yml`, после чего алерт должен прилететь в телеграмм.








