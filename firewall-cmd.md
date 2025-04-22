## firewall-cmd

```ruby
# ДОБАВЛЕНИЕ СЕРВИСА
firewall-cmd --get-services
firewall-cmd --permanent --add-service=ntp

# ДОБАВЛЕНИЕ ПОРТОВ
sudo firewall-cmd --list-ports
sudo firewall-cmd --permanent --add-port=8400/tcp

# Открыть все порты для конкретного IP (настройка зоны)
firewall-cmd --list-rich-rules
sudo firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="X.X.X.X" accept'

# Удаление правила
firewall-cmd --list-rich-rules
sudo firewall-cmd --permanent --zone=public --remove-rich-rule='rule family="ipv4" source address="X.X.X.X" accept'

# УДАЛЕНИЕ ПРАВИЛ
sudo firewall-cmd --list-ports
sudo firewall-cmd --permanent --remove-port=10000/tcp

# УПРАВЛЕНИЕ ПРАВИЛАМИ
sudo firewall-cmd --list-ports
sudo firewall-cmd --reload

# Посмотреть все правила зоны public
firewall-cmd --zone=public --list-all

# Или посмотреть активные зоны и их настройки
firewall-cmd --get-active-zones

# Всегда указывайте зону, например:
sudo firewall-cmd --zone=public --add-port=80/tcp --permanent
sudo firewall-cmd --reload

# https://ruvds.com/ru/helpcenter/upravlenie-portami-v-linux/
# https://www.dmosk.ru/miniinstruktions.php?mini=firewalld-centos
```
