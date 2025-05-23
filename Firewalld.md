## Firewalld

### Что такое зона (zone) в firewalld?

Firewalld использует зоны для управления уровнем доверия к сетевым интерфейсам. Каждая зона определяет:

- Какие порты открыты.
- Какие сервисы разрешены.
- Какие правила (rules) применяются.



### Вывод списка всех зон и всех служб, управляемых каждой зоной

```ruby
# Список всех зон
sudo firewall-cmd --get-zones

# Зона по умолчанию
sudo firewall-cmd --get-default-zone

# Список всех активных зон — зон, используемых в текущий момент
sudo firewall-cmd --get-active-zones

# Настройки одной зоны
sudo firewall-cmd --zone=public --list-all

# Настройки всех зон
sudo firewall-cmd --list-all-zones
```



### Ввод списка поддерживаемых служб

```ruby
sudo firewall-cmd --get-services
sudo firewall-cmd --get-services| xargs -n1

sudo firewall-cmd --info-service bittorrent-lsd
sudo firewall-cmd --info-service ceph-mon
```



### Выбор и настройка зоны

```ruby
# Настройки зоны work
sudo firewall-cmd --zone=work --list-all

# rich rules — собственные правила
sudo firewall-cmd --zone=block --permanent --add-rich-rule='rule family="ipv4" source address="192.168.11.0/24" port port="22" protocol="tcp" accept'
sudo firewall-cmd --zone=block --permanent --remove-rich-rule='rule family="ipv4" source address="192.168.11.0/24" port port="22" protocol="tcp" accept'

# Добавить/Удалить сервис
sudo firewall-cmd --permanent --zone=block --add-service=ssh
sudo firewall-cmd --permanent --zone=block --remove-service=ssh

# Привязка зоны к сетевому интерфейсу. После привязки зоны к сетевому интерфейсу нет необходимости перезагружать конфигурацию или перезапускать firewalld
sudo firewall-cmd --zone=work --permanent --change-interface=eth0

# Проверяем как прошла привязка интерфейса
sudo firewall-cmd --zone=work --list-interfaces

# Преобразуйте временные изменения в постоянные при необходимости
sudo firewall-cmd --runtime-to-permanent
```


### Изменение зоны по умолчанию

```ruby
# Проверьте, какая зона используется по умолчанию
firewall-cmd --get-default-zone

# Допустим, вы решили использовать зону drop, как наиболее ограничивающую
sudo firewall-cmd --set-default-zone drop

# После выполнения этой команды не требуется перезагружать или перезапускать firewalld.
```

### Настройка зоны

```ruby
# Смотрим текущие настройки зоны
firewall-cmd --zone=internal --list-all

# Убрать поддержку samba-client
sudo firewall-cmd --permanent --remove-service=samba-client --zone=public

# Добавить поддержку службы LDAPS
sudo firewall-cmd --permanent --zone=internal --add-service=ldaps

# rich rules — собственные правила
sudo firewall-cmd --zone=block --permanent --add-rich-rule='rule family="ipv4" source address="192.168.11.0/24" port port="22" protocol="tcp" accept'
sudo firewall-cmd --zone=block --permanent --remove-rich-rule='rule family="ipv4" source address="192.168.11.0/24" port port="22" protocol="tcp" accept'

# Сохранение временных правил
sudo firewall-cmd --runtime-to-permanent

sudo firewall-cmd --reload
```

**Комментарий**

Параметр `--reload` не разрывает уже имеющихся активных соединений. Параметр `--complete-reload` перезапускает `firewalld` и перезагружает модули ядра, вследствие чего имеющиеся активные соединения разрываются. **_Это хороший вариант для случаев, когда временные изменения настолько запутанны, что было бы желательно начать все сначала_**.


### Создание новой зоны

Создайте XML-файл с настройками зоны, затем перезагрузите брандмауэр `firewalld`, и он будет готов к использованию новой зоны.

Следующий пример создает зону для локальных служб имен с серверами DNS и DHCP на одном компьютере с доступом по SSH. Настройки сохраняются в файле `/etc/firewalld/zones/names.xml`:

```ruby
<?xml version="1.0" encoding="utf-8"?>
<zone>
 <short>Name Services</short>
 <description>
 DNS and DHCP servers for the local network, IPv4 only.
 </description>
 <service name="dns"/>
 <service name="dhcp"/>
 <service name="ssh"/>
</zone>
```

```ruby
# Проверяем появление новой зоны
sudo firewall-cmd --permanent --get-zones

# Перезагрузите firewalld
sudo firewall-cmd --reload

# Теперь новая зона присутствует в общем списке
sudo firewall-cmd --get-zones

# Вывести
sudo firewall-cmd --zone=names --list-all
```

Чтобы удалить зону, удалите ее файл `.xml`, а затем перезагрузите `firewalld`.



### Команды для управления интерфейсами

```ruby
# 1. Просмотр зоны, к которой принадлежит интерфейс
sudo firewall-cmd --get-zone-of-interface=eth0

# 2. Удаление интерфейса из зоны
sudo firewall-cmd --zone=public --remove-interface=eth0

# 3. Изменение зоны интерфейса
sudo firewall-cmd --zone=work --change-interface=eth0

# 4. Просмотр интерфейсов в зоне
sudo firewall-cmd --zone=public --list-interfaces

# 5. Добавление интерфейса в зону
sudo firewall-cmd --zone=public --add-interface=eth0

# 6. Установка интерфейса в доверенную зону (permanent)
sudo firewall-cmd --permanent --zone=trusted --add-interface=eth0
sudo firewall-cmd --reload

# 7. Полный список интерфейсов и их зон
sudo firewall-cmd --list-all-zones | grep -E "zone|interfaces"

# 8. Сброс привязки интерфейсов (очистка)
sudo firewall-cmd --zone=public --remove-interface=eth0 --permanent
sudo firewall-cmd --reload

# 9. Временное отключение firewalld для интерфейса
sudo firewall-cmd --zone=drop --add-interface=eth0

# 10. Постоянное назначение интерфейса (сохраняется после перезагрузки)
sudo firewall-cmd --permanent --zone=internal --add-interface=eth1
sudo firewall-cmd --reload
```

**Примечания:**

- `--permanent` — применяет настройки постоянно (требует `--reload`).
- `firewall-cmd --reload` — перезагружает правила без разрыва текущих соединений.
- Имена интерфейсов можно проверить через `ip a` или `nmcli device status`.



### Блокировка и разблокировка конкретных портов

```ruby
sudo firewall-cmd --zone=work --add-port=2022/tcp
sudo firewall-cmd --zone=work --remove-port=22/tcp
```

<br><br><br>

























```ruby
# Список всех доступных зон
firewall-cmd --get-zones	

# Показать, какие зоны сейчас активны и к каким интерфейсам привязаны
firewall-cmd --get-active-zones

#  Показывает текущую зону по умолчанию
firewall-cmd --get-default-zone

# Изменить зону по умолчанию (напр., на home)
firewall-cmd --set-default-zone=home

# Показать правила конкретной зоны (public)
firewall-cmd --zone=public --list-all	

# Изменение зоны по умолчанию
sudo firewall-cmd --permanent --set-default-zone=home
sudo firewall-cmd --reload
```



### Как изменить зону для сетевого интерфейса в firewalld

```ruby
# Проверить текущую зону интерфейса
firewall-cmd --get-active-zones

# Изменить зону интерфейса
sudo firewall-cmd --permanent --zone=НоваяЗона --change-interface=Интерфейс
sudo firewall-cmd --permanent --zone=trusted --change-interface=wlp4s0
sudo firewall-cmd --reload

# Проверить, что зона изменилась
firewall-cmd --get-active-zones
firewall-cmd --get-zone-of-interface=wlp4s0
```


### Как назначить политику по умолчанию для зоны в firewalld

В `firewalld` можно настроить действие по умолчанию для входящего (`input`), исходящего (`output`) и пересылаемого (`forward`) трафика в конкретной зоне. Это полезно, если нужно:

- **Блокировать все входящие подключения**, кроме разрешённых правил (`DROP/REJECT`).
- **Разрешить весь трафик** в доверенной зоне (`ACCEPT`).

```ruby
# Проверить текущие политики зоны
sudo firewall-cmd --zone=Зона --list-all
```

**Доступные политики (target) для зоны**

```ruby
# Стандартное поведение (зависит от зоны: REJECT для public, ACCEPT для trusted)
default	

# Разрешает весь трафик, даже если нет явных правил (опасно для public!)
ACCEPT	

# Тихий сброс пакетов без уведомления (аналог iptables -P INPUT DROP)
DROP	

# Отклоняет пакеты с отправкой ICMP error (клиент видит "Connection refused")
REJECT	
```

**Изменить политику зоны**

```ruby
# sudo firewall-cmd --zone=Зона --set-target=Политика

# Запретить все входящие подключения в зоне public (кроме разрешённых в services)
sudo firewall-cmd --permanent --zone=public --set-target=DROP

# Разрешить весь трафик в зоне trusted
sudo firewall-cmd --permanent --zone=trusted --set-target=ACCEPT

# Запретить ВСЕ кроме явно разрешенного
sudo firewall-cmd --permanent --zone=public --set-target=REJECT
sudo firewall-cmd --reload
```

**Проверить изменения**

```ruby
sudo firewall-cmd --zone=Зона --list-all
```

**Сбросить политику на стандартную**

```ruby
# Чтобы вернуть поведение по умолчанию
sudo firewall-cmd --permanent --zone=Зона --set-target=default
sudo firewall-cmd --reload
```

**Настройка и удаление правил**

```ruby
# Назначить правило
firewall-cmd --permanent --direct --add-rule ipv4 filter INPUT 0 -s 172.16.1.100 -j DROP
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.11.124" port port="22" protocol="tcp" drop'

# Удалить правило
firewall-cmd --permanent --direct --remove-rule ipv4 filter INPUT 0 -s 172.16.1.100 -j DROP
sudo firewall-cmd --permanent --remove-rich-rule='rule family="ipv4" source address="192.168.11.124" port port="22" protocol="tcp" drop'
```

















<br><br><br><br><br><br><br>

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
 
 
#### :earth_americas: Как с помощью firewall-cmd открыть доступ конкретному адресу по конкрентому порту
 
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
sudo firewall-cmd --list-rich-rules
sudo firewall-cmd --reload
```


#### Запретить любой трафик

```ruby
sudo firewall-cmd --set-default-zone=drop
sudo firewall-cmd --reload
```


<br>


```ruby
# 1. Проверить текущую активную зону
sudo firewall-cmd --get-active-zones

# 2. Удаляем все сервисы из зоны если они есть
sudo firewall-cmd --zone=public --remove-service=http --permanent
sudo firewall-cmd --zone=public --remove-service=https --permanent
# и другие сервисы, если нужно

# 3. Открываем для одного IP
sudo firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.1.100" port protocol="tcp" port="80" accept' --permanent
sudo firewall-cmd --reload

# 4. Запретить все входящие в зону public
sudo firewall-cmd --zone=public --set-target=DROP --permanent

# 5. Перезагрузка
sudo firewall-cmd --reload

# 6. Проверка
sudo firewall-cmd --zone=public --list-all
```



```ruby
# 1. Просмотр всех интерфейсов и их зон
sudo firewall-cmd --get-active-zones

# 2. Узнать зону конкретного интерфейса
sudo firewall-cmd --get-zone-of-interface=eth0

# 3. Просмотр всех зон и их правил
sudo firewall-cmd --list-all-zones

# 4. Список все зон
sudo firewall-cmd --get-zones

# 5. Изменение зоны интерфейса
sudo firewall-cmd --zone=<новая_зона> --change-interface=<интерфейс> --permanent

# Пример: перемещаем eth0 в зону  internal
sudo firewall-cmd --zone=internal --change-interface=eth0 --permanent

# 6. Применяем
sudo firewall-cmd --reload

# 7. Проверяем
sudo firewall-cmd --reload
```


```ruby

# 1. Какие интерфейсы привязаны к зоне
sudo firewall-cmd --zone=<зона> --list-interfaces

# 2. Удалить интерфейс из зоны
sudo firewall-cmd --zone=<зона> --remove-interface=<интерфейс> --permanent

# 3. Добавить интерфейс в зону
sudo firewall-cmd --zone=<зона> --add-interface=<интерфейс> --permanent
```


```ruby

# 1. Смотрим зоны
sudo firewall-cmd --get-zones

# 2. Смотрим правила зон
sudo firewall-cmd --list-all-zones

# 3. Смотрим активную зону
sudo firewall-cmd --get-active-zones

# 4. Смотрим в какой зоне интерфейс
sudo firewall-cmd --get-zone-of-interface=eth0

# 5. Меняем зону интерфейса
sudo firewall-cmd --zone=<новая_зона> --change-interface=<интерфейс> --permanent

# 6. Удаляем интерфейс из зоны
sudo firewall-cmd --zone=<зона> --remove-interface=<интерфейс> --permanent

# 7. Политика зоны по умолчанию. Запрещенно все, что явно не разрешено: 
sudo firewall-cmd --zone=public --set-target=DROP --permanent

# 8. Добавить интерфейс в зону
sudo firewall-cmd --zone=<зона> --add-interface=<интерфейс> --permanent

# 9. Проверка
sudo firewall-cmd --zone=public --list-all
```

[Установка и настройка Firewalld](https://blog.sedicomm.com/2017/04/17/ustanovka-i-nastrojka-firewalld/)
