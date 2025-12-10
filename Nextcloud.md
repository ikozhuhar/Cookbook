:white_check_mark: _**Задача:** <a name='1'>Статус</a>._

```ruby
В контексте Nextcloud под "приложениями" (apps) имеются в виду модули/расширения,
которые добавляют функциональность к базовой системе Nextcloud.

# Проверить общий статус
sudo -u www-data php /var/www/html/nextcloud/occ status

# Список всех приложений
sudo -u www-data php /var/www/html/nextcloud/occ app:list

# Посмотреть все включенные приложения
sudo -u www-data php /var/www/html/nextcloud/occ app:list --enabled

# Список отключенных приложений
sudo -u www-data php /var/www/html/nextcloud/occ app:list --disabled

# Отлючить/Включить приложение
sudo -u www-data php /var/www/html/nextcloud/occ app:disable onlyoffice
sudo -u www-data php /var/www/html/nextcloud/occ app:enable onlyoffice

# Проверить статус
sudo -u www-data php /var/www/html/nextcloud/occ app:list | grep onlyoffice

# Информация о конкретном приложении
sudo -u www-data php /var/www/html/nextcloud/occ app:list files

# Проверка состояния приложения
sudo -u www-data php /var/www/html/nextcloud/occ app:check-code [app_name]
```


<br>

:white_check_mark: _**Задача:** <a name='1'>Логи</a>._

```ruby
1. Основной лог-файл Nextcloud

# Просмотр всего лога
sudo tail -f /var/www/html/nextcloud/data/nextcloud.log

# Просмотр с фильтрацией по приложению
sudo grep "app_name" /var/www/html/nextcloud/data/nextcloud.log

# Просмотр последних 100 строк
sudo tail -n 100 /var/www/html/nextcloud/data/nextcloud.log


2. Команды через occ

# Уровни логирования
sudo -u www-data php /var/www/html/nextcloud/occ log:manage

# Просмотр логов через occ
sudo -u www-data php /var/www/html/nextcloud/occ log:tail

# Просмотр логов с фильтром по приложению
sudo -u www-data php /var/www/html/nextcloud/occ log:tail --app=files

# Просмотр логов за определенный период
sudo -u www-data php /var/www/html/nextcloud/occ log:tail --since="1 hour ago"


3. Фильтрация по конкретному приложению

# Для приложения "files"
sudo grep "\[files\]" /var/www/html/nextcloud/data/nextcloud.log

# Для приложения "calendar"
sudo grep "\[calendar\]" /var/www/html/nextcloud/data/nextcloud.log

# С просмотром в реальном времени
sudo tail -f /var/www/html/nextcloud/data/nextcloud.log | grep "\[app_name\]"


4. По уровню ошибок

# Только ошибки
sudo grep -i "error" /var/www/html/nextcloud/data/nextcloud.log

# Предупреждения и ошибки
sudo grep -E "(error|warning)" /var/www/html/nextcloud/data/nextcloud.log

# Критические ошибки
sudo grep -i "exception" /var/www/html/nextcloud/data/nextcloud.log


5. Настройка уровня логирования

# Текущий уровень
sudo -u www-data php /var/www/html/nextcloud/occ log:manage --level

# Изменить уровень (0-4)
sudo -u www-data php /var/www/html/nextcloud/occ log:manage --level=2

# Включить отладку для конкретного приложения
sudo -u www-data php /var/www/html/nextcloud/occ log:manage --level=0 --app=files


6. Полезные комбинации

# Логи за сегодня
sudo grep "$(date +%Y-%m-%d)" /var/www/html/nextcloud/data/nextcloud.log

# Логи приложения files за последний час
sudo grep "\[files\]" /var/www/html/nextcloud/data/nextcloud.log | grep "$(date -d '1 hour ago' +'%Y-%m-%d %H')"

# Поиск по конкретной ошибке
sudo grep "Exception" /var/www/html/nextcloud/data/nextcloud.log | tail -20


7. Очистка логов

# Архивация текущего лога
sudo cp /var/www/html/nextcloud/data/nextcloud.log /var/www/html/nextcloud/data/nextcloud.log.old
sudo truncate -s 0 /var/www/html/nextcloud/data/nextcloud.log


8. Примеры использования

# Мониторинг логов в реальном времени для приложения files
sudo tail -f /var/www/html/nextcloud/data/nextcloud.log | grep "\[files\]"

# Просмотр ошибок за последние 2 часа
sudo -u www-data php /var/www/html/nextcloud/occ log:tail --since="2 hours ago" --level=2

# Поиск проблем с конкретным пользователем
sudo grep "username" /var/www/html/nextcloud/data/nextcloud.log
```


<br>

:white_check_mark: _**Задача:** <a name='1'>Передача прав на папки и файлы в Nextcloud</a>._

```ruby
Все команды выполнялись с помощью консоли occ NextCloud

1. Подробная информация о пользователе
sudo -u www-data php occ user:info ncadmin

2. Убедится в существовании папки/файла
sudo ls -la "/var/www/html/nextcloud/data/ncadmin/files/"

3. Передача папки folder_name от пользователя ncadmin пользователю user01
sudo -u www-data php occ files:transfer-ownership --path="folder_name" ncadmin user01

После выполнения команды из пункта 3, из ЛК пользователя ncadmin папка folder_name переместится в ЛК пользователя user01 под названием вида: 'Передано от ncadmin 2025-10-31 17-33-01'. Nextcloud автоматически создает папку с описательным именем в котором сохраняет информацию о происхождении файла, дату и время. Переименовать папку можно вызвав меню правой кнопкой мыши по папке/файлу, выбрав пункт "Переименовать".

Важно. Пути в командах могут отличатся в зависимости от места выполнения самой команды и директории пользователя. Директорию пользователя можно увидеть из вывода команды пункта 1. Консоль oss по умолчанию находится в папке Nextcloud.

```



<br>

:white_check_mark: _**Задача:** <a name='1'>Критически важные настройки PHP</a>._


_Архитектура конфигурации PHP_

```ruby
/etc/php/8.2/
├── php.ini                   # Основной конфиг (низкий приоритет)
├── conf.d/                   # Директория модулей (высокий приоритет)
│   ├── 10-opcache.ini        # OPcache настройки
│   ├── 20-curl.ini           # Расширение cURL
│   ├── 30-mysqli.ini         # Расширение MySQLi
│   └── ...                   # Другие расширения
├── fpm/                      # Конфиги для PHP-FPM
│   ├── php.ini               # FPM версия (отдельная!)
│   └── conf.d/               # FPM модули
└── cli/                      # Конфиги для CLI
    ├── php.ini               # CLI версия (отдельная!)
    └── conf.d/               # CLI модули

1. php.ini                    # Базовые настройки
2. conf.d/10-opcache.ini      # Переопределяет opcache из php.ini
3. conf.d/20-something.ini    # Дополнительные настройки
```


_1. Память и лимиты_

```ruby
# В /etc/php/8.2/fpm/php.ini

memory_limit = 512M              # Минимум 512М, лучше 1Г для больших установок
post_max_size = 16G              # Должен быть ≥ максимальному размеру файла
upload_max_filesize = 16G        # Аналогично post_max_size
max_execution_time = 3600        # Для больших загрузок/операций
max_input_time = 3600            # Для обработки больших запросов
```

_2. OPcache (критически важен для производительности)_

```ruby
# /etc/php/8.2/fpm/conf.d/10-opcache.ini

opcache.enable = 1
opcache.memory_consumption = 256 # Минимум 256М для Nextcloud
opcache.interned_strings_buffer = 32
opcache.max_accelerated_files = 20000
opcache.revalidate_freq = 3600
opcache.fast_shutdown = 1
opcache.save_comments = 1        # ОБЯЗАТЕЛЬНО! Nextcloud требует комментарии
opcache.enable_file_override = 1
```

_3. PHP-FPM пул процессов_

```ruby
# /etc/php/8.2/fpm/pool.d/www.conf

pm = dynamic
pm.max_children = 25             # Минимум 20-25 для средней нагрузки
pm.start_servers = 6
pm.min_spare_servers = 4
pm.max_spare_servers = 12
pm.max_requests = 5000           # Предотвращает утечки памяти

pm.max_children = 120        # ← МАКСИМАЛЬНЫЙ ЛИМИТ ВСЕХ ПРОЦЕССОВ
pm.start_servers = 20        # количество при запуске
pm.min_spare_servers = 10    # минимальное idle процессов
pm.max_spare_servers = 30    # максимальное idle процессов
pm.max_requests = 500        # перезапуск после N запросов
```

_Конфигурационные файлы_

```ruby
# Основной php.ini. Содержит глобальные настройки, переопределяемые в пуле.
/etc/php/8.2/fpm/php.ini

# Конфигурация пула FPM. Локальные настройки для Nextcloud (память, процессы).
/etc/php/8.2/fpm/pool.d/www.conf

# OPcache настройки. Критичен для производительности.
/etc/php/8.2/fpm/conf.d/10-opcache.ini
```

_Быстрые команды проверки_

```ruby
# 1. Проверить все критические настройки
php -i | grep -E "(memory_limit|post_max_size|upload_max_filesize|max_execution)"

# 2. Проверить OPcache
php -i | grep -E "opcache\.(enable|memory_consumption|save_comments)"

# 3. Проверить настройки пула
sudo systemctl status php8.2-fpm | grep -A 3 "Status:"

# 4. Проверить расширения
php -m | grep -E "(gd|intl|curl|xml|mbstring|zip|pgsql|opcache|apcu)"
```



















