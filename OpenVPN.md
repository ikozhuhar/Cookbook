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
./easyrsa gen-req vpnserver1 nopass
```

![image](https://github.com/user-attachments/assets/f9c25f02-f3fc-4764-ac73-569ca3f27c59)


Сгенерируйте запрос на создание пары ключей и подписание сертификата для клиента.

```ruby
./easyrsa gen-req vpnclient1
```

![image](https://github.com/user-attachments/assets/dcf2adfc-9fa8-40c3-b7cc-6b3cfb43ac9a)


Подпишите запросы, используя общие имена (Common Name) сервера и клиента. Используйте только имена; если вы введете их пути, то это приведет к ошибке:

```ruby
./easyrsa sign-req server vpnserver1
./easyrsa sign-req client vpnclient1
```

![image](https://github.com/user-attachments/assets/fb27c7df-d5c5-4e38-960a-453ccfa30b76)
![image](https://github.com/user-attachments/assets/2165a84a-e888-428e-8fe3-8da3961962ea)


Сгенерируйте параметры протокола Диффи — Хеллмана (Diffie — Hellman) для сервера; это займет минуту или две. Данная команда должна запускаться на сервере OpenVPN:

```ruby
./easyrsa gen-dh
```

![image](https://github.com/user-attachments/assets/d770cec2-c7b1-48ae-a215-3708595c3f4a)
![image](https://github.com/user-attachments/assets/446e27b8-8d2f-4b22-a1be-5f11403f7bd4)


Создайте на сервере ключ HMAC (Hash-based Message Authentication Code — код аутентификации сообщения на основе хеш-функции):

```ruby
openvpn --genkey secret ta.key
```

![image](https://github.com/user-attachments/assets/4c813a52-d0b2-41ee-abcd-f2507d298c2c)


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
remote 192.168.50.133 1194

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


#### Распространение конфигураций с помощью файлов .ovpn

```ruby
# vpnclient1.ovpn
client
dev tun
proto udp
remote server2 1194

persist-key
persist-tun
resolv-retry infinite
nobind

user nobody
group nobody

tls-client
remote-cert-tls server
verb 4
# ca.crt
<ca>
-----BEGIN CERTIFICATE-----
MIIDSDCCAjCgAwIBAgIUD2UxdEwgvhhr0zq5fAxIDIueB2EwDQYJKoZIhvcNAQEL
BQAwFTETMBEGA1UEAwwKdnBuc2VydmVyMTAeFw0yMTAyMjExODU1MjNaFw0zMTAy
MTkxODU1MjNaMBUxEzARBgNVBAMMCnZwbnNlcnZlcjEwggEiMA0GCSqGSIb3DQEB
AQUAA4IBDwAwggEKAoIBAQDpQJo+Izt8v0zriSWwrChc1tnVj3E3h3XuyEHub7hj
y4bMu2PqKByFNr+iikEF3u0d6HrCRSDKt1BcLzL3TsTJ/hJBHAlTyqEgVce1knjL
2g9NnDbekRtJSJCxS9j+RWtP43Xdg5edb5hTCZqdNFHD8oNuSMGFBbHN4oi9eDXl
rvyVHJe+UkI1Ow6mW0+ln/IoKNFPovz+l+ds3fJ5+UHe2TaQPQc7tGZ33j7wfJQd
es8baFdK+lnmGdUOrW9BQE6ReMSezkz6dKdIZdy7jEs6xoflOzyWlgydmnkAvLnx
MBQDgDUbc5MuooVMAWa4yhtz0B9ZmdJDb8jzHDpTPqdRAgMBAAGjgY8wgYwwHQYD
VR0OBBYEFF8KPhl1xxV0110JiBs5iUEPoJ1IMFAGA1UdIwRJMEeAFF8KPhl1xxV0
110JiBs5iUEPoJ1IoRmkFzAVMRMwEQYDVQQDDAp2cG5zZXJ2ZXIxghQPZTF0TCC+
GGvTOrl8DEgMi54HYTAMBgNVHRMEBTADAQH/MAsGA1UdDwQEAwIBBjANBgkqhkiG
9w0BAQsFAAOCAQEAMnRLz3CBApSrjfUKsWYioNGQGvh77Smh/1hPGIu4eEldQSmZ
Aj7qclEaORdBxmqrVtA3Z9cX1L0xFrg14nLyddmuWHG3ZChc5ZMpYtD2YpOH265B
FFjDp96vK13dpixWKrVpvakLCCA4EvnC8CEjbm0oNFiCgSwKAoJFCcUzwC33swsU
B2w5/iT6CZKuKhSmET1IDpG8krGC/Ib2GNAS0szMI94P0ajZgVznMcXOJ7gUg4rM
sEB8OzM6GBEZTqbAa9uVMZnOZvZA5jGIbBuelUo0bqGdAyx2B68zzuL//qvsHsvw
kZCyKIaXH0NBV7vexMKWcwFLLBzWizFQbbFpFA==
-----END CERTIFICATE-----
</ca>
# vpnclient1.cert
<cert>
-----BEGIN CERTIFICATE-----
MIIDVjCCAj6gAwIBAgIQLhO4FTrqN5WZiQETULAwnzANBgkqhkiG9w0BAQsFADAV
MRMwEQYDVQQDDAp2cG5zZXJ2ZXIxMB4XDTIxMDIyMTE4NTYzM1oXDTI0MDIwNjE4
NTYzM1owFTETMBEGA1UEAwwKdnBuY2xpZW50MTCCASIwDQYJKoZIhvcNAQEBBQAD
ggEPADCCAQoCggEBALUFYXwk6JW/hRtoMs0Ug5jMcWXsjMUsCz8L8CeXNOs3wQrf
YBWF1TYCLPd2/vwXsvbqCE85IZwjsJ5mEx9YgQ5M1teDkLZqBn8y7VIyDAAU8RsN
NcrnpeMDV0LgZIBeUrHi4ZTooaw4FdJ5BBYRHR1APVaaHDWx59ohJuBDpriWhvWk
lWX0rpSJltXriIOCzky/yEwfw6ah5jWaTgfe41fXq8j3lx2IbgIL7I4//jhC6JYz
N7huTdT2uB2MUbYX0XWBffMG8wcBZtMI2XryZmPvFYWP7N5nZZsBXkLz/UngAu3k
jkYJOnJy/hdOFLN/yXj7VFydmivUSeekdjjxyAECAwEAAaOBoTCBnjAJBgNVHRME
AjAAMB0GA1UdDgQWBBSnLIQoTPLyECbJHfgYBHvQpcmfgzBQBgNVHSMESTBHgBRf
Cj4ZdccVdNddCYgbOYlBD6CdSKEZpBcwFTETMBEGA1UEAwwKdnBuc2VydmVyMYIU
D2UxdEwgvhhr0zq5fAxIDIueB2EwEwYDVR0lBAwwCgYIKwYBBQUHAwIwCwYDVR0P
BAQDAgeAMA0GCSqGSIb3DQEBCwUAA4IBAQBaBpYZXVYUzOcXOVSaijmOZAIVBTeJ
meQz9xBQjqDXaRvypWlQ1gQtO8WnK9ruafc1g/h7LtvqtiALnGiJ0NbshkH8C1KE
yen46UCau5B/Xi0gA7FoPildvYdKSn/jI6KySCsplubjnJK9H/6DjAcEuqFLcsaY
5vpKQGP9Vl7H7hEVs4f1aory1T4Ma/bdXEOqgzHmIARLmxYeJm90sUT/n7e7VXfy
fILZ+8D1fMxCbeQRBkg1e8wJfgEbMRY9aGGt1qAs9gkm9RPelGB18v4iCbyebv3X
4hVHmfjcixdbWiABC7yq/gisooQ0robW/92dgemcwO0awHZX+opNBgwr
-----END CERTIFICATE-----
</cert>
# vpnclient1.key
<key>
-----BEGIN ENCRYPTED PRIVATE KEY-----
MIIFHDBOBgkqhkiG9w0BBQ0wQTApBgkqhkiG9w0BBQwwHAQInjFvz5a4mY8CAggA
MAwGCCqGSIb3DQIJBQAwFAYIKoZIhvcNAwcECNsxQXxvMpN0BIIEyEZdgFwPnGup
vyhywXR6l6ihvHK2GRczIgH0mFIiwQDgDjZj2YsEnvSA/P3MHplkU/bgv9DJ5j2T
C5wPDmGN4yG1boHx9BQKbXqxGwdz/UcHwmNKur9qnSFrSVEvMDwvum+rmzWuKykf
gkKKBCT1JZ2DWKtjjDNYG9qhBn3S2zYVq311dDuLbBcruvo1UL031sDDYWTpVuuf
zZc0ozng0Nzb35bNkG6Ib+LYLzJi4stxzw0DTFl52lKv++R6xhmqb81IJE3vBs4H
DOutkYfifO1eGqEKksPQRl8n03UVkOtB5pH8VdQeLqEBBaq3qeIfU6FkH9XrPR/E
8VOg9BNpbyuUW7bQu0MzuJ8Ofkjy9K+HHdwFtGPyOatkeaXT/qcKVMvzWcbr8bPc
VncavzXdzo0Sb8FigsKYU1lNjgo00Phd3m0AOfptrweK6ucBds5SmqNrUFXiQ2JA
Ms3LUw4CXBBgvdu5TsA2xLGysip0RPKLyTnUPGnXxbBaaHMv8Jz3XRCrWgZbtAE3
XhE9fKw+ZMEP+2jpC/1mjN/N9VuJfYZEhgA84wzYMu6pt3zPkWZqR6yGTDFEDhvh
OAZYEpqrhe++nxDpuQlpCCl4IndSg9L9oX1ydrvPNHGbRVztd3+r9wr4Ub3fJ1g/
9ckCdanohEymKbjw34HEMmdx+fn5k2T9bLnl8fsYtcESkg04ChON3yOnZFKl6chT
BQ9X2Qmeg7FoawWiUY5o+7OHNKL7QpRt4jXPbXNuXFK9EYvuRzUqubLhL5DdmjuO
Se1vvZg7fT4C8qjYsoCa18idA00EN3ePFFf9AssHCoVW92GiUTTKG+qURCjtNtG6
dnPvxiSf98OBkkjeX3ni0cKdfMGoQTSdEy5GexvfRMF5HJrGO+CWXmqSBsuIlPUe
quqCsPmpaT2Ws/0UU9cKe4qaKjTL7CghtFmUEhH7t6Cd41Ki9gKi33j3541l9w7l
J1bgca4rRUCecp2BPF3IjJc/RnTvHkbUK4mDX9s8xJhYf9WE6JYsk3NBSNNIj/9G
FMJlo71x8H3OAdFzRN5bjV797HByZ+YidZIgGAx2dSko3PQPy7RSxdmzFbxfUvzj
9jcYEu+V9unbtDK2qZ9I+LqXGE+EXjPBui40IWp8XIYNlSLn2qgroH079lXhXKBY
+DzcBzyT7GTX2QeYE+yqqPRIFWHnbnsnD6dMnAa46h+Si+f5sq33rfRsF7UpK4gV
IhzFkncCM47/Taqi0OY04Q40LuSCDjmjFL+VzZOsAtWGRNYNzIgniThEehElJwfI
ErzClcVptjhtCer8BPuO7YaMIHk1hKecHFqw3RrimWzroL1iu9Q29m2oM+bVc6mD
we6r+t8JbaAFxoHBK4i6M0rcdJPICxDTIOjPC3Fg/MeqiCi7F0DFZvXwPGRD+0Of
MBnsDplEUjK06jbE5BjGQ7n7P+dwDxyp/aVO4CfX7ZOco6h9r3b6nqlzPVNE9erw
kS7WwT/TWraw/sfIO9sNSgle7PoRh2s/w/oGVhC6ymlMdXe+mhMzHFnGEbBRh2Rd
kd/EdYNubHg0k9+RLTwbgwZ+176cIJyOpqaoJGv0bsKM8X26Pk/fkyF6xgdQYQOx
8i9Whea8OjUOQAcgc7gUyA==
-----END ENCRYPTED PRIVATE KEY-----
</key>
# ta.key
<tls-auth>
-----BEGIN OpenVPN Static key V1-----
4eb35b44d1d8a82cfa51af394d4f58f3
69bf8fe8c0a0a032f38b0ee104889628
8a5dc89486736b39d64ad3c6831bf9ba
9f3f96c3307d322a5bf055b9bc3bfa74
929faf361c14de97445f5927794264bb
e3f71c925f2236cfb0109ecfd6406cef
857dfb39783a09ecd56d3cf09ebbc853
0f43b1c787f0db99dbecabcd2090cfbb
54c86d8102a5430fd6a7f37ab5ce8ed9
f6bec8984bde4267f78913ff702dd396
a205b6be9e7ab41cf1ebad3953c27c7c
f3b435345e02aede049ef7c9f1c2704f
2ed91110ccb19d0d3bd46a00f54c73e2
07b31160cdc54c3f5a7989bb999ac5f3
89c6de7e79fc93399924a8d298eab462
231234e690c319d5cbd832788f0dbcfb
-----END OpenVPN Static key V1-----
</tls-auth>
```

```ruby
Подробнее: Linux книга рецептов. Карла Шрёдер, стр. 365
```

[Документация для OpenVPN](https://openvpn.net/community-resources/)

