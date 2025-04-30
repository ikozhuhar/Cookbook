### firewall-cmd

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
 
 
#### Как с помощью firewall-cmd открыть доступ конкретному адресу по конкрентому порту
 
```ruby
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.100" port port="22" protocol="tcp" accept'
sudo firewall-cmd --reload
sudo firewall-cmd --list-rich-rules

# Разрешить несколько портов
Если нужно открыть несколько портов для одного IP, добавьте каждое правило отдельно или используйте --add-service (если порты описаны в сервисе).
```

```ruby
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="185.66.85.0/24" port port="80,443" protocol="tcp" accept'
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="185.66.86.0/24" port port="80,443" protocol="tcp" accept'
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="185.35.5.0/24" port port="80,443" protocol="tcp" accept'
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="185.35.6.0/24" port port="80,443" protocol="tcp" accept'
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="109.238.89.0/24" port port="80,443" protocol="tcp" accept'
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="89.20.63.0/24" port port="80,443" protocol="tcp" accept'
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="185.63.60.0/22" port port="22" protocol="tcp" accept'
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="185.63.60.0/22" accept'
sudo firewall-cmd --set-default-zone=drop
sudo firewall-cmd --reload
```


#### Запретить любой трафик

```ruby
sudo firewall-cmd --set-default-zone=drop
sudo firewall-cmd --reload
```

[Установка и настройка Firewalld](https://blog.sedicomm.com/2017/04/17/ustanovka-i-nastrojka-firewalld/)
