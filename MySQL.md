
```ruby
mysql -v -h 127.0.0.1 -u ikozhuhar -pZEwht62M5j otus < /home/ansible/otus_05-12-2025.sql

CREATE DATABASE IF NOT EXISTS otus;
CREATE USER IF NOT EXISTS 'ikozhuhar'@'%' IDENTIFIED BY 'ZEwht62M5j';
GRANT ALL PRIVILEGES ON *.* TO `ikozhuhar`@`%`;
FLUSH PRIVILEGES;

SELECT User, Host FROM mysql.user;
```

### Правильный способ полного удаления MySQL:

```ruby
# 1. Остановить сервис
sudo systemctl stop mysql

# 2. Удалить ВСЕ пакеты MySQL (без подстановки звёздочкой в apt!)
sudo apt-get purge mysql-server mysql-server-8.0 mysql-client mysql-common -y

# 3. Удалить зависимости
sudo apt-get autoremove -y
sudo apt-get autoclean

# 4. Только после этого удалить оставшиеся файлы
sudo rm -rf /etc/mysql /var/lib/mysql /var/log/mysql
```
