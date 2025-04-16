## Резервное копирование с  помощью rsync и  cp



<br>

_**Задача**: rsync для создания локальной резервной копии._

Следующий пример показывает, как создать резервную копию домашнего каталога пользователя Duchess на USB-накопитель с именем 2tbdisk:

```ruby
sudo rsync -av ~ /media/duchess/2tbdisk/
sudo rsync -av ~/arias ~/overtures /media/duchess/2tbdisk/duchess/
```

При добавлении косой черты в конце `~/ (/home/duchess/)` будет копироваться только содержимое каталога `duchess/`, но не сам он, в результате получится набор файлов `/media/duchess/2tbdisk/[файлы]`.  Если опустить косую черту в конце, то файлы из `/home/duchess` будут скопированы вместе с каталогом `duchess` и получится: `/media/duchess/2tbdisk/duchess/[файлы]`.  Завершающая косая черта имеет значение только для исходного каталога и не имеет значения для целевого.




<br>

_**Задача**: Безопасная передача файлов с помощью rsync по сети через SSH._

При передаче файлов на другой компьютер rsync по умолчанию использует SSH. На удаленной машине должен быть запущен SSH-сервер, а на исходной машине — настроен SSH-клиент.

Ниже представлен пример копирования каталога `/woodwinds` со всем его содержимым с удаленного хоста в домашний каталог Duchess:

```ruby
sudo rsync -av empress@remote.example.com:/backups/woodwinds /home/duchess/Music/
sudo rsync -av admin@10.104.100.156:/var/www/html .

sudo rsync -hav --progress --log-file=/home/kozhukhar/mysite.ru/rsynclog.txt padmin@10.104.100.155:~/04.04.2025.dump.sql .
```
