## Резервное копирование с  помощью rsync и  cp


<br>

**Важные замечания:**
**Для сохранения владельца** (`-o`) и группы (`-g`) нужно запускать rsync от root (`sudo`), иначе операция завершится ошибкой (если у вас нет прав на смену владельца).

Безопасно протестировать команду `rsync` без копирования файлов можно, добавив параметр `--dry-run`:

```ruby
duchess@pc:~$ rsync -av --dry-run \
~/Music/scores ~/Music/woodwinds /media/duchess/2tbdisk/duchess/
```

<br>

_**Задача**: rsync для создания локальной резервной копии._

Следующий пример показывает, как создать резервную копию домашнего каталога пользователя Duchess на USB-накопитель с именем `2tbdisk`:

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


<br>

## Автоматизация резервного копирования с помощью rsync, cron и SSH

_**Задача**: Создать задание cron для автоматического и безопасного резервного копирования с помощью rsync._

_Для решения поставленной задачи необходимо настроить на целевом компьютере SSH с аутентификацией без пароля и сетевой доступ к целевому компьютеру для клиентов._

Затем можно использовать `/etc/crontab` для передачи, требующей привилегий `root`. Следующий пример копирует файлы из `/home/padmin/atlassian-jira-software-9.12.13-x64.bin` каждый час в 59 минут на сервер с именем `padmin@192.168.11.124`, находящийся в локальной сети:

```ruby
# мин часы день_месяца месяц день_недели user command
59 * * * * padmin /usr/bin/rsync -a /home/padmin/atlassian-jira-software-9.12.13-x64.bin padmin@192.168.11.124:~/dump
```


<br>

## Включение и исключение с помощью файла со списком для исключения 

```ruby
# Linux книга рецептов. Карла Шрёдер, стр. 224
```

<br>

_**Задача**: Ограничение скорости передачи в командеrsync_

```ruby
sudo rsync --bwlimit=512 -ave ssh ~/Music/arias empress@laptop:songs/
```

<br>

_**Задача**: Создание сервера резервного копирования с помощью rsyncd_

```ruby
# Linux книга рецептов. Карла Шрёдер, стр. 228
```
