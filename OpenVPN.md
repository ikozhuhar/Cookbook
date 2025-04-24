### Настройка простого подключения для тестирования

```ruby
# На сервере
sudo openvpn --remote 192.168.50.147 --dev tun0 --ifconfig 10.0.0.1 10.0.0.2

# На клиенте
sudo openvpn --remote 192.168.50.133 --dev tun0 --ifconfig 10.0.0.2 10.0.0.1
```

### Настройка простого шифрования со статическими ключами

_Задача: Добавить шифрование для OpenVPN максимально простым способом._

Самый простой способ настроить шифрование — применить общие статические ключи. Они удобны для тестирования, но не подходят для использования в промышленном окружении. 

В следующих примерах сервер `OpenVPN` находится на `server1`, клиент — на `client1`, а новый ключ — в файле `myvpn.key`. Но, вообще говоря, ключи можно хранить в файлах с любыми именами.

Выполните эти шаги:

1) создайте общий статический ключ и передайте на два хоста;
2) создайте конфигурационные файлы для сервера и клиента;
3) запустите OpenVPN на обоих хостах, использовав ссылки на их конфигурационные файлы.

Создайте новый каталог на сервере OpenVPN для хранения ключей, затем создайте новый статический ключ:

```ruby
$ sudo mkdir /etc/openvpn/keys
$ sudo openvpn --genkey secret /etc/openvpn/keys/myvpn.key
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

```ruby
ping -I tun0 10.0.0.1
ping -I tun0 10.0.0.2
```

![image](https://github.com/user-attachments/assets/9dd65fef-bcfa-4331-8768-480cd525d8f5)


![image](https://github.com/user-attachments/assets/a8d1cd17-73b1-4a71-b07b-a472e247f51f)




