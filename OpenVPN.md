### [:diamond_shape_with_a_dot_inside:](#toc) 1. Настройка простого VPN подключения для тестирования

```ruby
# На сервере
sudo openvpn --remote 192.168.50.147 --dev tun0 --ifconfig 10.0.0.1 10.0.0.2

# На клиенте
sudo openvpn --remote 192.168.50.133 --dev tun0 --ifconfig 10.0.0.2 10.0.0.1
```


<br>

### [:diamond_shape_with_a_dot_inside:](#toc) 2. Настройка простого VPN шифрования со статическими ключами

_**Задача:** Добавить шифрование для OpenVPN максимально простым способом._

Самый простой способ настроить шифрование — применить общие статические ключи. Они удобны для тестирования, но не подходят для использования в промышленном окружении. 

В следующих примерах сервер `OpenVPN` находится на `server1`, клиент — на `client1`, а новый ключ — в файле `myvpn.key`. Но, вообще говоря, ключи можно хранить в файлах с любыми именами.

Выполните эти шаги:

**1)** создайте общий статический ключ и передайте на два хоста;  
**2)** создайте конфигурационные файлы для сервера и клиента;  
**3)** запустите OpenVPN на обоих хостах, использовав ссылки на их конфигурационные файлы.  

Создайте новый каталог на `сервере` OpenVPN для хранения ключей, затем создайте новый статический ключ:

_**На сервере**_

```ruby
sudo mkdir /etc/openvpn/keys
sudo openvpn --genkey secret /etc/openvpn/keys/myvpn.key
```
![image](https://github.com/user-attachments/assets/6817fd6d-23df-43a2-b496-c55b814c1f6e)


```ruby
# server1.conf
dev tun
ifconfig 10.0.0.1 10.0.0.2
secret /etc/openvpn/keys/myvpn.key
local 192.168.50.133
```

![image](https://github.com/user-attachments/assets/de36d250-4da1-41bc-9450-065cd17f8344)


_**На клиенте**_

```ruby
# Нужно создать папку и скопировать ключ myvpn.key с сервера
sudo mkdir /etc/openvpn/keys
scp ikozhuhar@192.168.50.133:/etc/openvpn/keys/myvpn.key .
```
![image](https://github.com/user-attachments/assets/6817fd6d-23df-43a2-b496-c55b814c1f6e)


```ruby
# client1.conf
dev tun
ifconfig 10.0.0.2 10.0.0.1
secret /etc/openvpn/keys/myvpn.key
remote 192.168.50.133
```

![image](https://github.com/user-attachments/assets/392e6c0c-504f-47a2-89b8-623575fa85fb)

```ruby
sudo openvpn /etc/openvpn/server1.conf
```

![image](https://github.com/user-attachments/assets/9dd65fef-bcfa-4331-8768-480cd525d8f5)

![image](https://github.com/user-attachments/assets/7df219c7-4b04-4348-a9a4-7b1e15a54559)

```ruby
ping -I tun0 10.0.0.1
ping -I tun0 10.0.0.2
```
![image](https://github.com/user-attachments/assets/abafe2ba-5a7e-41e3-9740-84b31d3b7e09)



<br>

### [:diamond_shape_with_a_dot_inside:](#toc) 3. Настройка EasyRSA. Установка для управления инфраструктурой PKI

Инфраструктура открытых ключей (PKI) может располагаться где угодно, причем не обязательно на сервере OpenVPN. Вы должны создать сертификаты **_сервера_** и **_клиента_** в PKI, а затем скопировать их на соответствующие хосты.

Fedora и Ubuntu помещают все файлы EasyRSA в рабочий каталог /usr/share/. Он не является самым лучшим, поскольку перезаписывается системными обновлениями. Создайте новый каталог, которым управляете только вы, и в месте, где не нужны права root, как в нашем примере с пользователем Duchess, который создал каталог /home/duchess/mypki:

```ruby
# Вы можете установить пакет easy-rsa
apt install easy-rsa
```
![image](https://github.com/user-attachments/assets/e7e089cb-9e04-4caf-bf06-f1c4d8a6c7b5)
![image](https://github.com/user-attachments/assets/dd94ea75-d94c-4115-add6-a3f403be0b37)
![image](https://github.com/user-attachments/assets/5a812b04-ecc1-42c0-8af7-ce3b553bcf66)

```ruby
# Каталог для хранения инфраструктуры открытых ключей
mkdir mypki

# скопируйте каталог /usr/share/easy-rsa в созданный вами каталог:
sudo cp -r /usr/share/easy-rsa mypki
```
![image](https://github.com/user-attachments/assets/9307b8b3-79d4-47dc-b61a-f47cd5619cac)

![image](https://github.com/user-attachments/assets/f9e8e1eb-d91a-41d1-ae2d-08d567f65992)

В результате будет создан каталог `mypki/easy-rsa`. Проверьте его разрешения; **_владельцами всего_**, что находится в этом каталоге, **_должны быть вы и ваша группа_**.

![image](https://github.com/user-attachments/assets/27ff6e11-7889-4f98-8aa3-48e60a81cef7)

**_Комментарий_**

Для создания инфраструктуры PKI и управления ею не нужны права root. Вы можете разместить ее где угодно, и она должна быть отделена от вашей конфигурации OpenVPN, то есть находиться в отдельном каталоге или на отдельном компьютере. Те, кто давно работает с OpenVPN, рекомендуют устанавливать инфраструктуру на хорошо защищенную машину, не подключенную к Интернету.


### Создание инфраструктуры PKI

После установки EasyRSA нужно правильно настроить инфраструктуру открытых ключей (Public Key Infrastructure, PKI)

Создание инфраструктуры PKI включает следующие шаги:

**1)** Создать собственный сертификат центра сертификации (CA) для подписи сертификатов сервера и клиента. Он должен храниться отдельно от конфигурации сервера OpenVPN — в отдельном каталоге или на отдельном компьютере;
**2)** Создать и подписать сертификат сервера OpenVPN;
**3)** Создать и подписать клиентские сертификаты;
**4)** Скопировать сертификаты сервера и клиентов в /etc/openvpn/keys на соответствующие компьютеры. (Имя каталога может быть другим, отличным от keys.)

Перейдите в каталог инфраструктуры и выполните команду ее инициализации:

```ruby
# Она создаст пустую структуру каталогов для новой PKI.
cd ~/mypki/easy-rsa
./easyrsa init-pki

# Затем создайте новый центр сертификации. ЦС создает и подписывает сертификаты сервера и клиентов
./easyrsa build-ca
```

![image](https://github.com/user-attachments/assets/fa7079a2-8dc6-4005-a060-3ca13a6496d7)
![image](https://github.com/user-attachments/assets/259b95a4-ee1f-4b6c-b97f-3b0f10cf6b9f)
![image](https://github.com/user-attachments/assets/cb6e021d-9b08-4d03-aa12-b0d93b6c5d49)



Сгенерируйте запрос на СОЗДАНИЕ пары ключей и подписание сертификата для вашего СЕРВЕРА OpenVPN. Обычно не принято добавлять парольную фразу в закрытый ключ сервера, но вы можете защитить его паролем, если хотите, опустив параметр nopass. Парольная фраза обеспечивает надежную защиту, но тогда вам придется вводить ее каждый раз при перезапуске сервера:

```ruby
easyrsa gen-req vpnserver1 nopass
```

![image](https://github.com/user-attachments/assets/f9c25f02-f3fc-4764-ac73-569ca3f27c59)


Сгенерируйте запрос на создание пары ключей и подписание сертификата для клиента.

```ruby
easyrsa gen-req vpnclient1
```

![image](https://github.com/user-attachments/assets/dcf2adfc-9fa8-40c3-b7cc-6b3cfb43ac9a)


Подпишите запросы, используя общие имена (Common Name) сервера и клиента. Используйте только имена; если вы введете их пути, то это приведет к ошибке:

```ruby
easyrsa sign-req server vpnserver1
easyrsa sign-req client vpnclient1
```

Создайте на сервере ключ HMAC (Hash-based Message Authentication Code — код аутентификации сообщения на основе хеш-функции):

```ruby
openvpn --genkey --secret ta.key
```

- Скопируйте vpnclient1.key, vpnclient1.crt, ca.crt и ta.key в каталог `/etc/openvpn/keys` на машине `client1`.
- Скопируйте vpnserver1.key, vpnserver1.crt, ca.crt, dh.pem и ta.key в каталог `/etc/openvpn/keys` на машине `server1`.

После подписания запроса на подпись сертификата можно удалить все файлы `*.req`.


### Создание и тестирование конфигураций сервера и клиента

В данном рецепте мы настроим простую тестовую конфигурацию с двумя хостами в одной подсети, server1 и client1. Это хороший простой способ проверить настройки сервера, не беспокоясь о маршрутизации и пересечении вашего интернет-шлюза.

В следующем примере представлены настройки СЕРВЕРА OpenVPN. Обратите внимание, что серверные ключи можно хранить где угодно на сервере, важно только правильно указать их местоположение в файле с настройками:

_Пример настроек клиента:_

```ruby
# vpnserver1.conf
port 1194
proto udp
dev tun
user nobody
group nobody
ca /etc/openvpn/keys/ca.crt
cert /etc/openvpn/keys/vpnserver1.crt
key /etc/openvpn/keys/vpnserver1.key
dh /etc/openvpn/keys/dh.pem
tls-auth /etc/openvpn/keys/ta.key 0

server 10.10.0.0 255.255.255.0
ifconfig-pool-persist ipp.txt
keepalive 10 120
persist-key
persist-tun

tls-server
remote-cert-tls client

status openvpn-status.log
verb 4
mute 20
explicit-exit-notify 1
```

_Пример настроек клиента:_

```ruby
# vpnclient1.conf
client
dev tun
proto udp
remote server1 1194

persist-key
persist-tun
resolv-retry infinite
nobind

user nobody
group nobody
tls-client
remote-cert-tls server
verb 4

ca /etc/openvpn/keys/ca.crt
cert /etc/openvpn/keys/vpnclient1.crt
key /etc/openvpn/keys/vpnclient1.key
tls-auth /etc/openvpn/keys/ta.key 1
```

Запустите OpenVPN на обоих хостах с помощью команды openvpn:

```ruby
sudo openvpn /etc/openvpn/vpnserver1.conf
sudo openvpn /etc/openvpn/vpnclient1.conf
```

Вот и все! Вывод команд подтверждает, что настройки верны и между хостами успешно установлено соединение. Для остановки нажмите Ctrl+C на обоих хостах.

