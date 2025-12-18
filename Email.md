### Отправка письма через команду mail

```ruby
sudo apt install mailutils
```

_Настройте /etc/mail.rc_

```ruby
sudo nano /etc/mail.rc

set from="user@mosinzhproekt.ru"
set smtp="smtp.mosinzhproekt.ru"
set smtp-port=587
set smtp-auth=login
set smtp-auth-user="user"
set smtp-auth-password="8#xUcjX~RQSz&?Vv"
```


_Отправьте тестовое письмо_

```ruby
echo "Test body" | mail -s "Test Subject" Kozhukhar.I@mosinzhproekt.ru
```

### Тестирование через openssl

```ruby
openssl s_client -connect smtp.mosinzhproekt.ru:587 -starttls smtp -crlf
EHLO localhost
AUTH LOGIN
c2MtZGV2 # echo -n "sc-dev" | base64
OHhVY2pYUlFTem45N1Z2 # echo -n "8xUcjXRQSzn97Vv" | base64
```

### Telnet

```ruby
telnet smtp.mosinzhproekt.ru 587
EHLO localhost
AUTH LOGIN
c2MtZGV2
OHhVY2pYUlFTem45N1Z2
MAIL FROM: sc-dev@mosinzhproekt.ru
RCPT TO: Kozhukhar.I@mosinzhproekt.ru
DATA
From: sc-dev@mosinzhproekt.ru
To: Kozhukhar.I@mosinzhproekt.ru
Subject: Test with Telnet
Content-Type: text/plain; charset=utf-8

This is the body of the email.
HELLO message here.

.
QUIT
```

<img width="1301" height="821" alt="image" src="https://github.com/user-attachments/assets/a93fcc53-0ebb-4cab-8dcc-5bb1a2ab2127" />

