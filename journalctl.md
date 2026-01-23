:white_check_mark: _**Задача:** <a name='1'>Поиск полезной информации в файлах журналов</a>._

```ruby
# Для начала просмотрите самые новые записи
sudo journalctl -r
```

```ruby
# Просмотрите самые последние записи с пояснительными сообщениями
sudo journalctl -ex | less
```

```ruby
# Ищите конкретные службы
sudo journalctl -u nginx.service
sudo journalctl -b -1 -p "crit" -u nginx.service
sudo journalctl -b -3 -p "crit".."warning" -u nginx.service

# Новые события с предварительным выводом десяти предыдущих
journalctl -n 10 -u nginx.service -f
```

```ruby
# Ограничьте поиск диапазоном дат. Это можно сделать несколькими способами:
sudo journalctl -u mariadb.service -S today
sudo journalctl -u ssh.service -S '1 week ago'
sudo journalctl -u libvirtd.service -S '2021-03-05'
sudo journalctl -u httpd.service -S '2021-03-05' -U '2021-03-09'
sudo journalctl -u httpd.service --since='2021-03-05' --until='2021-03-09'
sudo journalctl -u nginx.service -S '2 hours ago'
sudo journalctl -u nginx.service -S '5 min ago'
```


```ruby
# Просмотреть события, имевшие место от часа до пяти минут тому назад, и имена файлов модулей, ответственных за эти события
sudo journalctl -S '1h ago' -U '5 min ago' -o with-unit
```

```ruby
# Cобытия сервера HTTP, с момента последней загрузки, и вывести только 50 последних
journalctl -b -n 50 -u httpd.service
```


<br>




```ruby
# Удаляет системные журналы старше 7 дней
sudo journalctl --vacuum-time=7d
```

<img width="1145" height="622" alt="image" src="https://github.com/user-attachments/assets/4f97d622-563b-4962-bf5a-0bf8af755d86" />
<img width="1148" height="610" alt="image" src="https://github.com/user-attachments/assets/9c9dca61-13e5-4f7d-9936-a9d625288f1d" />
<img width="1137" height="618" alt="image" src="https://github.com/user-attachments/assets/4217aa72-a91a-4903-bc88-38b664877afc" />
<img width="1138" height="610" alt="image" src="https://github.com/user-attachments/assets/9f3bade2-951f-4404-a2f0-657bcd4380b0" />
<img width="1134" height="602" alt="image" src="https://github.com/user-attachments/assets/0c96cc23-44d0-4728-a514-6d0ac34f7449" />
<img width="1147" height="607" alt="image" src="https://github.com/user-attachments/assets/7c4ad046-022c-4298-b730-c6c4bb3c733c" />
<img width="1126" height="610" alt="image" src="https://github.com/user-attachments/assets/23f0f758-9a18-4a5c-ad05-e1821fcf2e3e" />
<img width="1116" height="573" alt="image" src="https://github.com/user-attachments/assets/1d096b97-9a77-4bfe-841e-4dc6ff28ac27" />
<img width="1132" height="615" alt="image" src="https://github.com/user-attachments/assets/aad37bc6-6a4e-48aa-8425-1d56f91d1fcb" />
<img width="1129" height="585" alt="image" src="https://github.com/user-attachments/assets/886a2369-7237-4bed-ac10-fe8c1e9bcb03" />
<img width="1129" height="614" alt="image" src="https://github.com/user-attachments/assets/242bcf87-ca62-4a6f-8e98-c34f09632691" />
<img width="1125" height="611" alt="image" src="https://github.com/user-attachments/assets/d90b9182-ba5e-4bb7-87e2-c85f0ffeb23d" />
<img width="1132" height="602" alt="image" src="https://github.com/user-attachments/assets/ba9fdfa1-1a48-4892-8c1a-a603b2c4d8da" />
<img width="1138" height="612" alt="image" src="https://github.com/user-attachments/assets/432bf9e6-dec9-41a3-b7eb-ad3c76fe0d35" />
<img width="1137" height="619" alt="image" src="https://github.com/user-attachments/assets/b0aed13d-5013-4f68-9919-2621e65a248f" />




