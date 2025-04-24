### Настройка простого подключения для тестирования

sudo openvpn --remote 192.168.50.147 --dev tun0 --ifconfig 10.0.0.1 10.0.0.2

sudo openvpn --remote 192.168.50.133 --dev tun0 --ifconfig 10.0.0.2 10.0.0.1
