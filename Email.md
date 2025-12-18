### Отправка письма через команду mail

```ruby
sudo apt install mailutils
```

### Настройте /etc/mail.rc

```ruby
sudo nano /etc/mail.rc

set from="user@mosinzhproekt.ru"
set smtp="smtp.mosinzhproekt.ru"
set smtp-port=587
set smtp-auth=login
set smtp-auth-user="user"
set smtp-auth-password="8#xUcjX~RQSz&?Vv"
```


### Отправьте тестовое письмо

```ruby
echo "Test body" | mail -s "Test Subject" Kozhukhar.I@mosinzhproekt.ru
```

```ruby
openssl s_client -connect smtp.mosinzhproekt.ru:25 -starttls smtp -crlf
```
