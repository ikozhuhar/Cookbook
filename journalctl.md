:white_check_mark: _**Задача:** <a name='1'>Поиск полезной информации в файлах журналов</a>._

```ruby
# Для начала просмотрите самые новые записи
sudo journalctl -r
```

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
sudo journalctl -u httpd.service -S '2021-03-05' -u '2021-03-09'
sudo journalctl -u nginx.service -S '2 hours ago'
sudo journalctl -u nginx.service -S '5 min ago'
```


```ruby
# Просмотреть события, имевшие место от часа до пяти минут тому назад, и имена файлов модулей, ответственных за эти события
sudo journalctl -S '1h ago' -U '5 min ago' -o with-unit
```

```ruby
# Просмотреть события сервера HTTP, произошедшие с момента последней загрузки, и вывести при этом только 50 самых последних записей
journalctl -b -n 50 -u httpd.service
```


<br>




```ruby
# Удаляет системные журналы старше 7 дней
sudo journalctl --vacuum-time=7d
```
