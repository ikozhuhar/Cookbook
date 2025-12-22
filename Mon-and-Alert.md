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
