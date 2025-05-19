Настройка кэширующего DNS с помощью BIND9

### Установка BIND9

```ruby
sudo apt update
sudo apt install bind9 -y
```

### Настройка BIND9 как кэширующего DNS

```ruby
# Отредактируйте конфигурационный файл named.conf.options
sudo nano /etc/bind/named.conf.options
```

### Добавьте или измените следующие параметры

```ruby
options {
    directory "/var/cache/bind";

    // Отключение рекурсии для внешних запросов (опционально)
    recursion yes;
    allow-recursion { 127.0.0.1; 192.168.1.0/24; }; // Разрешить рекурсию только для локальных клиентов
    allow-query { 127.0.0.1; 192.168.1.0/24; };    // Разрешить запросы только от доверенных сетей

    forwarders {
        8.8.8.8;       // Google DNS
        8.8.4.4;       // или другие публичные DNS
        77.88.8.8;      // Яндекс DNS
    };

    dnssec-validation auto; // Включить DNSSEC (рекомендуется)

    listen-on { 127.0.0.1; 192.168.1.100; }; // На каких интерфейсах слушать
    listen-on-v6 { none; };                  // Отключить IPv6, если не используется
};
```

### Проверка конфигурации и перезапуск BIND

```ruby
sudo named-checkconf  # Проверка синтаксиса
sudo systemctl restart bind9
sudo systemctl enable bind9
```

### Настройка клиентов

На клиентских машинах укажите IP-адрес этого сервера в `/etc/resolv.conf` или через DHCP.
