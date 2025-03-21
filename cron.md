

```ruby
# Создание файла
nano backup.sh

#!/bin/bash
cp /home/CORP.COSMOSGROUP.RU/ikozhuhar/EMCSetup.exe /home/CORP.COSMOSGROUP.RU/ikozhuhar/test-cron/
cp /home/CORP.COSMOSGROUP.RU/ikozhuhar/EMCSetup.exe /run/media/ikozhuhar/Ventoy/

chmod +x backup.sh
```

```ruby
# Файл /etc/crontab
45 14 * * * ikozhuhar /home/CORP.COSMOSGROUP.RU/ikozhuhar/backup.sh
50 13 * * * root /sbin/shutdown -h +20
```


```ruby
# Linux книга рецептов. Карла Шрёдер, стр. 101

```
