### [:diamond_shape_with_a_dot_inside:](#toc) <a name='3'>systemctl</a>

_**Задача:** Перечислить все службы, установленные в системе, и вывести их состояние: запущены они или находятся в состоянии ошибки._

```ruby

# Перечисление всех модулей, активные и неактивные:
sudo systemctl --all

# Сколько всего файлов модулей?
sudo systemctl list-unit-files

# Службы
sudo systemctl list-unit-files --type=service
sudo systemctl list-unit-files --type=service --state=enabled
sudo systemctl list-unit-files --type=service --state=disabled
sudo systemctl list-unit-files --type=service --state=static
sudo systemctl list-unit-files --type=service --state=masked
```


Статус  `enabled` показывает, что служба доступна и управляется системой `systemd`. Когда служба включена, systemd создает символическую ссылку в `/etc/systemd/system/` на файл модуля в `/lib/systemd/system/`. 

Статус `disabled` означает, что в `/etc/systemd/system/` нет символической ссылки, и эта служба не запускается автоматически при загрузке (но ее можно запустить и остановить вручную).

Статус `masked` говорит о том, что служба ссылается на `/dev/null/`. Она полностью отключена, и ее нельзя запустить никаким способом.

Статус `static` означает, что файл модуля является зависимостью для других файлов модуля и не может быть запущен или остановлен пользователем.

Статус `indirect` присваивается службам, которые не предназначены для управления пользователями, но могут управляться другими службами; generated сообщает, что служба была преобразована из конфигурационного файла системы инициализации, отличной от systemd, — SysV init или LSB init.


<br>

_**Задача:** Определение состояния выбранных служб._
