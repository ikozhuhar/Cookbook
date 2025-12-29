### ✅ Автоматическая аутентификация через .pgpass

Если в команде `sudo -i -u postgres /opt/pgpro/ent-14/bin/pg_dump --data-only --table="$DB_NAME" MIP_UPSD > "$BACKUP_FILE"` не указан пароль, то он ищется в `.pgpass`

Когда в команде `sudo -i -u postgres /opt/pgpro/ent-14/bin/pg_dump --data-only --table="$DB_NAME" MIP_UPSD > "$BACKUP_FILE"` не указаны явные параметры аутентификации (`--username`, `--password` или `-W`), PostgreSQL клиентские утилиты используют цепочку автоматической аутентификации:

#### Порядок поиска параметров подключения:

1. Параметры командной строки (если явно указаны)
2. Переменные окружения:
```ruby
PGPASSWORD - пароль
PGUSER - имя пользователя
PGDATABASE - имя базы данных
PGHOST - хост
PGPORT - порт
```
3. Файл `~/.pgpass` - если другие способы не заданы
4. Файл `pg_service.conf` - для сервисных файлов
5. Аутентификация через `socket` (peer auth) - для локальных подключений



### ✅ Tantor

```ruby
Как принудительно переинициализировать:

# Остановить службу
sudo systemctl stop tantor-be-server-17.service

# Удалить данные
sudo rm -rf /var/lib/postgresql/tantor-be-17/data/*

# Или удалить всю директорию
sudo rm -rf /var/lib/postgresql/tantor-be-17/data

# Создать заново
# Создать директорию в домашней папке
mkdir -p /var/lib/postgresql/tantor-be-17/data
chmod 700 ~/tantor-data/data
sudo chown postgres:postgres /var/lib/postgresql/tantor-be-17/data
sudo chown postgres:postgres ~/tantor-data/data

# Инициализировать БД
sudo -u postgres /opt/tantor/db/17/bin/initdb -D /var/lib/postgresql/tantor-be-17/data
```
