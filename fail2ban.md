## Как пользоваться


<br>

```ruby
# Смотрим какие есть jail для того, чтобы в них добавлять правила
fail2ban-client status

# Synopsis
fail2ban-client set <JAIL> addignoreip <IP>

# Добавляем
fail2ban-client set sshd addignoreip 185.63.60.81
fail2ban-client set sshd addignoreip 185.63.61.33
fail2ban-client set exim-isp addignoreip 185.63.61.33
fail2ban-client set exim-isp addignoreip 185.63.60.81

# Смотрим
fail2ban-client get sshd ignoreip
fail2ban-client get exim-isp ignoreip

# Удаляем
fail2ban-client set sshd delignoreip 185.63.61.33
fail2ban-client set exim-isp delignoreip 185.63.61.33
```

```ruby
# Linux man page
https://linux.die.net/man/1/fail2ban-client
```

```ruby
# sudo nano /etc/fail2ban/jail.local

[DEFAULT]
bantime  = 1h
maxretry = 5
findtime = 10m
ignoreip = 10.0.15.0/24


[sshd]
enabled = true
port    = ssh
logpath = /var/log/auth.log
backend = auto

[nginx-http-auth]
enabled  = true
port     = http,https
logpath  = /var/log/nginx/error.log
maxretry = 3

[nginx-limit-req]
enabled = true
port    = http,https
logpath = /var/log/nginx/error.log

[nginx-botsearch]
enabled  = true
port     = http,https
logpath  = /var/log/nginx/error.log
maxretry = 2



[apache-auth]
enabled  = true
port     = http,https
logpath  = /var/log/apache2/error.log
maxretry = 5

[apache-badbots]
port     = http,https
logpath  = /var/log/apache2/access.log
bantime  = 48h
maxretry = 1

[apache-noscript]
port     = http,https
logpath  = /var/log/apache2/error.log

[apache-overflows]
port     = http,https
logpath  = /var/log/apache2/error.log
maxretry = 2

[apache-nohome]
port     = http,https
logpath  = /var/log/apache2/error.log
maxretry = 2

[apache-botsearch]
port     = http,https
logpath  = /var/log/apache2/error.log
maxretry = 2

[apache-modsecurity]
port     = http,https
logpath  = /var/log/apache2/error.log
maxretry = 2

[apache-shellshock]
port     = http,https
logpath  = /var/log/apache2/error.log
maxretry = 1
```


### Пользовательский фильтр для Nginx и защита WordPress на Apache

```ruby
# Пользовательский фильтр для Nginx
nano /etc/fail2ban/filter.d/nginx-404.conf

[Definition]
failregex = ^<HOST> - .* "(GET|POST|HEAD) .*HTTP/.*" 404 .*
ignoreregex =

Тюрьма в jail.local

[nginx-404]
enabled = true
port = http, https
filter = nginx-404
logpath = /var/log/nginx/access.log
maxretry = 5
findtime = 10m
bantime = 24h
```

```ruby
# Фильтр: защита WordPress на Apache
sudo nano /etc/fail2ban/filter.d/wordpress.conf
[Definition]
failregex = ^<HOST> .* "POST /wp-login.php
            ^<HOST> .* "POST /wordpress/wp-login.php
            ^<HOST> .* "POST /xmlrpc\.php
ignoreregex =

# Тюрьма в jail.local
[wordpress]
enabled  = true
filter   = wordpress
port     = http,https
logpath  = /var/log/apache2/access.log   # для Debian/Ubuntu
# logpath = /var/log/httpd/access_log    # для RHEL
maxretry = 3
findtime = 5m
bantime  = 1d

# Тестирование перед запуском
sudo fail2ban-regex /var/log/nginx/access.log /etc/fail2ban/filter.d/nginx-404.conf
sudo fail2ban-regex /var/log/apache2/access.log /etc/fail2ban/filter.d/wordpress.conf
```



```ruby
sudo fail2ban-client status
sudo fail2ban-client status sshd

sudo fail2ban-client set sshd unbanip 192.168.234.128

https://firstvds.ru/technology/ustanovka-i-nastroyka-fail2ban
https://wiki.astralinux.ru/pages/viewpage.action?pageId=48760206
https://redos.red-soft.ru/base/redos-7_3/7_3-security/7_3-fail2ban/
https://adminvps.ru/blog/nastrojka-fail2ban-na-ubuntu-24-04-lts-i-zashhita-ot-brutforsa/
```
