

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


<br>

:white_check_mark: _Задача: <a name='1'>Настройка скрипта для переключения сетевых служб</a>._  

_network_switch_advanced.sh_

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

_crontab_

```ruby
# Для root
sudo crontab -e

# Для конкретного пользователя (например, gitlab-adm)
sudo crontab -u gitlab-adm -e
```

```ruby
0 0 * * * root /usr/local/bin/network_switch_advanced.sh
```

_sudoers_

```ruby
# Добавьте строку (замените username на вашего пользователя):
root ALL=(ALL) NOPASSWD: /bin/systemctl disable networking, /bin/systemctl enable systemd-networkd, /bin/systemctl start systemd-networkd, /bin/systemctl stop networking
# Или для всех пользователей:
%sudo ALL=(ALL) NOPASSWD: /bin/systemctl disable networking, /bin/systemctl enable systemd-networkd, /bin/systemctl start systemd-networkd, /bin/systemctl stop networking
```



<br>

:white_check_mark: _Задача: <a name='1'>Настроить резервное копирование баз данных PostgreSQL</a>._

_.pgpass_

```ruby
nano /home/admin/.pgpass
# localhost:5432:db_name:db_user:db_pass
192.168.11.141:5432:*:postgres:gX#ZTPvV4$_UnBF4~xA.
```



_sys-atl-db_backup.sh_

```ruby
#!/bin/bash

# Настройки
BACKUP_DIR="/home/administrator/Загрузки/DB/backup"
PG_USER="postgres"
PG_HOST="192.168.11.141"
PG_PORT="5432"
RETENTION_DAYS="14"
LOG_FILE="/var/log/sys-atl-db_backup.log"

# Массив баз данных на одном сервере
declare -a DATABASES=(
    "confluence_new"
    "indeed_key_log"
    "indeedam_core"
    "indeedam_key"
    "indeedam_log"
    "jira"
    "pw"
)

# Создание каталога для бэкапов
mkdir -p "$BACKUP_DIR"


# ФУНКЦИЯ ДЛЯ ЛОГИРОВАНИЯ
log_message() {
    echo "[$(date)] $1" | tee -a "$LOG_FILE"
}


# ФУНКЦИЯ ДЛЯ ПРОВЕРКИ ПОДКЛЮЧЕНИЯ К БД
check_db_connection() {
    local DB_NAME="$1"

    if psql -h "$PG_HOST" -p "$PG_PORT" -U "$PG_USER" -d "$DB_NAME" -c "SELECT 1;" >/dev/null 2>&1; then
        return 0
    else
        return 1
    fi
}


# ФУНКЦИЯ ДЛЯ БЭКАПА ОДНОЙ БАЗЫ
backup_database() {
    local DB_NAME="$1"
    local DATE=$(date +%F_%H-%M)
    local BACKUP_FILE="$BACKUP_DIR/${DB_NAME}_backup_${DATE}.sql.gz"

    log_message "=========================================="
    log_message "Starting backup of database '$DB_NAME'..."

    # Проверяем подключение к БД
    if ! check_db_connection "$DB_NAME"; then
        log_message "ERROR: Cannot connect to database '$DB_NAME'. Skipping..."
        return 1
    fi

    # Создание бэкапа (пароль берется из .pgpass)
    log_message "Creating backup for $DB_NAME..."
    pg_dump -v -h "$PG_HOST" -p "$PG_PORT" -d "$DB_NAME" -U "$PG_USER" -Z1 -f "$BACKUP_FILE"

    local backup_result=$?

    if [ $backup_result -eq 0 ]; then
        # Проверяем размер файла
        local file_size=$(du -h "$BACKUP_FILE" | cut -f1)
        log_message "Backup successful: $BACKUP_FILE (size: $file_size)"

        # Удаление старых файлов для этой БД
        log_message "Removing old backups for $DB_NAME (older than $RETENTION_DAYS days)..."
        find "$BACKUP_DIR" -type f -name "${DB_NAME}_backup_*.sql.gz" -mtime +$RETENTION_DAYS -delete

        return 0
    else
        log_message "ERROR: Backup failed for $DB_NAME! Exit code: $backup_result"
        # Удаляем пустой файл бэкапа если он создался
        [ -f "$BACKUP_FILE" ] && rm -f "$BACKUP_FILE"
        return 1
    fi
}


# ФУНКЦИЯ ДЛЯ ОТПРАВКИ БЭКАПОВ НА УДАЛЕННЫЙ СЕРВЕР И ОЧИСТКА
sync_backups() {
    local REMOTE_SERVER="iamgroot@192.168.9.242"
    local REMOTE_PATH="/mnt/bkpdata/sys-atl-db"
    local LOCAL_CLEANUP_DAYS="7"
    local SSH_OPTS="-o StrictHostKeyChecking=no -o ConnectTimeout=30 -o BatchMode=yes"

    log_message "=========================================="
    log_message "Starting backup sync to remote server..."

    # Проверяем доступность удаленного сервера
    log_message "Checking remote server connectivity..."
    if ssh $SSH_OPTS "$REMOTE_SERVER" "echo 'Connection successful'" 2>/dev/null; then
        log_message "Remote server is accessible via SSH"
    else
        log_message "ERROR: Cannot connect to remote server via SSH!"
        return 1
    fi

    # Создаем папку на удаленном сервере (если не существует)
    if ! ssh $SSH_OPTS "$REMOTE_SERVER" "mkdir -p '$REMOTE_PATH'" 2>/dev/null; then
        log_message "ERROR: Cannot create remote directory!"
        return 1
    fi

    # Отправляем файлы бэкапов
    local sync_count=0
    local sync_failed=0

    for backup_file in "$BACKUP_DIR"/*.sql.gz; do
        if [ -f "$backup_file" ]; then
            local file_name=$(basename "$backup_file")
            log_message "Syncing $file_name to remote server..."

            if scp $SSH_OPTS "$backup_file" "$REMOTE_SERVER:$REMOTE_PATH/" 2>/dev/null; then
                log_message "Successfully synced: $file_name"
                ((sync_count++))
            else
                log_message "ERROR: Failed to sync: $file_name"
                ((sync_failed++))
            fi
        fi
    done

    # Очищаем локальные бэкапы старше 14 дней
    log_message "Cleaning up local backups older than $LOCAL_CLEANUP_DAYS days..."
    local deleted_files=$(find "$BACKUP_DIR" -type f -name "*.sql.gz" -mtime +$LOCAL_CLEANUP_DAYS)
    if [ -n "$deleted_files" ]; then
        echo "$deleted_files" | while read file; do
            log_message "Deleting: $(basename "$file")"
            rm -f "$file"
        done
        local deleted_count=$(echo "$deleted_files" | wc -l)
        log_message "Deleted $deleted_count old local backup files"
    else
        log_message "No old backup files to delete"
    fi

    # Итоговый отчет
    log_message "Sync completed. Successfully synced: $sync_count, Failed: $sync_failed"

    if [ $sync_failed -gt 0 ]; then
        log_message "WARNING: Some files failed to sync!"
        return 1
    else
        log_message "Backup sync finished successfully."
        return 0
    fi
}



# Основной цикл бэкапа
main() {
    log_message "Starting database backup process..."

    # Проверяем доступность PostgreSQL сервера
    log_message "Checking PostgreSQL server connectivity..."
    if pg_isready -h "$PG_HOST" -p "$PG_PORT" -U "$PG_USER" >/dev/null 2>&1; then
        log_message "PostgreSQL server is accessible"
    else
        log_message "ERROR: PostgreSQL server is not accessible!"
        exit 1
    fi

    # Бэкап каждой базы данных
    local success_count=0
    local fail_count=0

    for DB in "${DATABASES[@]}"; do
        if backup_database "$DB"; then
            ((success_count++))
        else
            ((fail_count++))
        fi
    done

    # Общий отчет
    log_message "Backup process completed. Success: $success_count, Failed: $fail_count"
    log_message "Current backups in $BACKUP_DIR:"
    ls -la "$BACKUP_DIR"/*.sql.gz 2>/dev/null | while read line; do
        log_message "  $line"
    done

    local total_files=$(find "$BACKUP_DIR" -name "*.sql.gz" -type f 2>/dev/null | wc -l)
    log_message "Total backup files: $total_files"

    if [ $fail_count -gt 0 ]; then
        log_message "WARNING: Some backups failed! Check the logs above."
        exit 1
    else
        log_message "Backup process finished successfully."
    fi


    # ВЫЗОВ ФУНКЦИИ И ПЕРЕДАЧА БЭКАПОВ НА УДАЛЕННЫЙ СЕРВЕР И ОЧИСТКА
    if [ $fail_count -eq 0 ] && [ $total_files -gt 0 ]; then
        sync_backups
        local sync_result=$?
        if [ $sync_result -ne 0 ]; then
            log_message "WARNING: Backup sync completed with errors!"
        fi
    else
        log_message "Skipping backup sync due to backup failures or no backup files"
    fi

    if [ $fail_count -gt 0 ]; then
        log_message "WARNING: Some backups failed! Check the logs above."
        exit 1
    else
        log_message "Backup process finished successfully."
    fi

}

# Запуск основного скрипта
main
```

_crontab_

```ruby
# Для root
sudo crontab -e

# Для конкретного пользователя (например, gitlab-adm)
sudo crontab -u gitlab-adm -e
```

```ruby
0 0 * * * root /usr/local/bin/sys-atl-db_backup.sh
```


