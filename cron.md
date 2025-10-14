

```ruby
# Создание файла
nano backup.sh

#!/bin/bash
cp /home/CORP.COSMOSGROUP.RU/ikozhuhar/EMCSetup.exe /home/CORP.COSMOSGROUP.RU/ikozhuhar/test-cron/
cp /home/CORP.COSMOSGROUP.RU/ikozhuhar/EMCSetup.exe /run/media/ikozhuhar/Ventoy/

# Право на исполнение
chmod +x backup.sh
```

```ruby
# Файл /etc/crontab
45 14 * * * ikozhuhar /home/CORP.COSMOSGROUP.RU/ikozhuhar/backup.sh
50 13 * * * root /sbin/shutdown -h +20
```


```ruby
# Linux книга рецептов. Карла Шрёдер, стр. 101

```


### Настройка скрипта для переключения сетевых служб

```ruby
sudo nano /usr/local/bin/network_switch_advanced.sh
```

```ruby
#!/bin/bash

LOG_FILE="/var/log/network_switch.log"

log_message() {
    echo "$(date): $1" >> "$LOG_FILE"
}

log_message "Starting network service switch..."

# Выполняем первую команду
if systemctl disable --now networking; then
    log_message "Successfully disabled networking service"
else
    log_message "ERROR: Failed to disable networking service"
    exit 1
fi

# Ждем 10 секунд
log_message "Waiting 10 seconds..."
sleep 10


# Выполняем вторую команду
if systemctl enable --now systemd-networkd; then
    log_message "Successfully enabled systemd-networkd service"
else
    log_message "ERROR: Failed to enable systemd-networkd service"
    exit 1
fi

log_message "Network service switch completed successfully"
```

```ruby
sudo crontab -e
```

```ruby
0 0 * * * root /usr/local/bin/network_switch_advanced.sh
```


