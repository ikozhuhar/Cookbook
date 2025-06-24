## Как очистить сервер от мусора

### Как это работает?

**Таймер** `clean-server.timer` **ищет сервис с таким же именем**, но с расширением `.service` вместо `.timer`. `clean-server.timer` → автоматически связывается с `clean-server.service`. **Если имена разные**, **можно явно указать**, какой **сервис** запускать, добавив в секцию `[Timer]`: `Unit=my-custom-service.service`


:white_check_mark: Скрипт: `nano /usr/local/bin/clean-server.sh`

```bash
#!/bin/bash

CURRENT_DATE=$(date +"%Y-%m-%d")
LOG_FILE="/var/log/clean-server/server-cleanup_${CURRENT_DATE}.log"
echo "[$(date)] Очистка начата" >> "$LOG_FILE"
echo "" >> "$LOG_FILE"

echo "Удаления .log-файлов старше 7 дней" >> "$LOG_FILE"
find /var/log -type f -name "*.log" -mtime +7 -delete >> "$LOG_FILE" 2>&1
echo "" >> "$LOG_FILE"

echo "Очистить systemd-журналы" >> "$LOG_FILE"
journalctl --vacuum-time=7d >> "$LOG_FILE" 2>&1
echo "" >> "$LOG_FILE"

echo "Очистить временные файлы" >> "$LOG_FILE"
rm -rf /tmp/* /var/tmp/* >> "$LOG_FILE" 2>&1
echo "" >> "$LOG_FILE"

echo "Очистить кэш и мусор после установки пакетов" >> "$LOG_FILE"
apt clean && apt autoremove -y >> "$LOG_FILE" 2>&1
echo "" >> "$LOG_FILE"

echo "[$(date)] Очистка завершена" >> "$LOG_FILE"
rm -rf /var/log/server-cleanup.log
```


:white_check_mark: Создадим юнит для сервиса: `nano /etc/systemd/system/clean-server.service`

```ruby
[Unit]
Description=Автоматическая еженедельная очистка сервера

[Service]
Type=oneshot
ExecStart=/usr/local/bin/clean-server.sh
```


:white_check_mark: Создаем таймер: `nano /etc/systemd/system/clean-server.timer`

```ruby
[Unit]
Description=Таймер для еженедельной очистки сервера

[Timer]
OnCalendar=Wed 09:00
Persistent=true

[Install]
WantedBy=timers.target
```


:white_check_mark: Активируем

```ruby
sudo systemctl daemon-reexec # перезапускает демон systemd, но расписание таймера сохраняется
sudo systemctl daemon-reload # перечитывает конфиги, но не влияет на уже запланированные таймеры
sudo systemctl enable --now clean-server.timer
```


:white_check_mark: Можем проверить статус

```ruby
systemctl list-timers | grep clean-server
sudo systemctl is-enabled clean-server.timer
systemctl status cleanup-server.timer
```


:white_check_mark: Логи

```ruby
journalctl -u clean-server.timer 
journalctl -u clean-server.service
```
