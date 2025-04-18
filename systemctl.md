### [:diamond_shape_with_a_dot_inside:](#toc) <a name='3'>systemctl</a>

_**Задача:** Перечислить все службы, и вывести их состояние: запущены они или находятся в состоянии ошибки._

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

_**Задача:** Узнать состояние одной или нескольких конкретных служб._

```ruby
systemctl status cups.service
systemctl status mariadb.service bluetooth.service lm-sensors.service
```
![image](https://github.com/user-attachments/assets/0d73bacf-cd62-411e-bae9-cf9179cf1f73)


Круглый маркер рядом — это индикатор состояния. Он отображается разными цветами. Белый — неактивное состояние. Красный — состояние сбоя или ошибки. Зеленый — служба активна, запускается или перезапускается.

`Loaded` сообщает, загружен ли файл модуля в память и полный путь к нему, текущий статус службы и статус, предопределенный производителем (vendor preset:), который сообщает, должна ли, по мнению производителя, запускаться эта служба во время загрузки. Когда служба выключена, это означает, что по умолчанию производитель не предполагал запускать ее при загрузке. Этот параметр лишь отражает предпочтения производителя и не сообщает, включена ли служба в настоящее время.

`Active` сообщает, активна ли служба и как долго она находится в этом состоянии.

`Process` сообщает идентификаторы PID процессов, команды и имена демонов.

`Main PID` — это номер процесса для сегмента контрольной группы.

`Tasks` сообщает количество задач, запущенных службой. Задачи — это идентификаторы процессов PID.

`CGroup` показывает, к какому сегменту модуля принадлежит служба и его PID. Существует три сегмента модулей по умолчанию: `user.slice`, `system.slice` и `machine.slice`.

Контрольные группы Linux (cgroups) — это наборы связанных процессов и всех их потомков. Сегмент в `systemd` — это часть контрольной группы, и каждый сегмент управляет определенной группой процессов. Запустите `systemctl status`, чтобы увидеть иерархию контрольных групп.  По умолчанию службы и области видимости модулей сгруппированы в `/lib/systemd/system/system.slice`. Пользовательские сеансы сгруппированы в `/lib/systemd/system/user.slice`. Виртуальные машины и контейнеры, зарегистрированные в systemd, сгруппированы в `/lib/systemd/system/machine.slice`.

![image](https://github.com/user-attachments/assets/7eb9765b-e269-41d5-a713-90b24ffdc19f)

```ruby
# Дополнительная информация
man 1 systemctl
man 5 systemd.slice
man 1 journalctl
```

<br>

_**Задача:** Запуск и остановка служб._

```ruby
# Запуск службы:
sudo systemctl start sshd.service

# Остановка службы:
sudo systemctl stop sshd.service

# Остановка и перезапуск службы:
sudo systemctl restart sshd.service

# Перезагрузка службы. Например, внесли изменения в sshd_config, чтобы эти изменения вступили в силу без перезапуска службы:
sudo systemctl reload sshd.service

# Запуск нескольких служб
sudo systemctl start sshd.service mariadb.service firewalld.service
```

```ruby
# Дополнительная информация
man 1 systemctl
```


<br>

_**Задача:** Включение и выключение служб._

```ruby

# включение службы сводится к созданию в каталоге /etc/systemd/system/ символической ссылки на файл службы в каталоге /lib/ systemd/system/.
sudo systemctl enable sshd.service

# включение и немедленный запуск службы
sudo systemctl enable --now sshd.service

# выключиние службы sshd
sudo systemctl disable sshd.service

# выключиние и немедленная остановка службы sshd
sudo systemctl disable --now sshd.service

# остановка службы sshd
sudo systemctl stop sshd.service

# Перевключение службы
sudo systemctl reenable mariadb.service

# команда полностью выключает службу bluetooth, маскируя ее
sudo systemctl mask bluetooth.service

# Размаскирование службы не включает ее, поэтому ее нужно запускать вручную
sudo systemctl unmask bluetooth.service
sudo systemctl start bluetooth.service
```

```ruby
# Дополнительная информация
man 1 systemctl
```


<br>

_**Задача:** Остановка неисправных процессов._

```ruby

sudo systemctl kill mariadb
sudo systemctl kill -9 mariadb

sudo systemctl kill 1234
sudo systemctl kill -9 1234
```

```ruby
# Дополнительная информация
man 5 systemd.kill
man 1 systemctl
man 1 kill
man 7 signal
```


<br>

_**Задача:** Управление уровнями запуска._

```ruby
# проверяет, работает ли система
systemctl is-system-running

# Сообщает текущую цель по умолчанию:
systemctl get-default

# Сообщает текущий уровень запуска:
runlevel

# Перезагрузить систему в режиме восстановления:
sudo systemctl rescue

# Перезагрузить систему в аварийном режиме:
sudo systemctl emergency

# Перезагрузить систему в режиме по умолчанию:
sudo systemctl reboot

# Перезагрузить в другом режиме без изменения режима по умолчанию:
sudo systemctl isolate multi-user.target

# Установить уровень запуска по умолчанию:
sudo systemctl set-default multi-user.target

# Вывести список имеющихся файлов, определяющих уровни запуска:
ls -l /lib/systemd/system/runlevel*

# Вывести список зависимостей для выбранного уровня запуска:
systemctl list-dependencies graphical.target

```

<br>

### [:diamond_shape_with_a_dot_inside:](#toc) <a name='3'>runlevel</a>

Уровни запуска в SysV — это разные состояния, в которые может загружаться система, например, с графическим рабочим столом, без графического рабочего стола, а также в аварийном и восстановительном режиме, если система не загружается с уровнем запуска по умолчанию. Цели в systemd примерно соответствуют устаревшим уровням запуска в SysV:

```ruby
# остановка системы
runlevel0.target, poweroff.target

# однопользовательский режим без графической среды, все локальные файловые системы монтируются, вход может выполнить только пользователь root, сеть неактивна
runlevel1.target, rescue.target

# многопользовательский режим без графической среды
runlevel3.target, multi-user.target

# многопользовательский режим с графической средой
runlevel5.target, graphical.target

# перезагрузка
runlevel6.target, reboot.target
```

Команда `systemctl emergency` — это особая аварийная цель, более ограниченная, чем режим восстановления `rescue`: в этом режиме не запускаются службы, немонтируются файловые системы, кроме корневой, нет сети и вход может выполнить только пользователь `root`. Это самая минимальная работающая конфигурация системы, предназначенная для устранения проблем. Варианты загрузки в аварийном и восстановительном режимах доступны в меню загрузчика GRUB2.

Команда systemctl is-system-running сообщает текущее состояние системы, которое может быть одним из следующих:

```ruby
# система еще не завершила запуск
initializing

# система на заключительном этапе запуска
starting

# система полностью работоспособна и все процессы запущены
running

# система работоспособна, но один или несколько модулей systemd потерпели неудачу. Выполните systemctl | grep failed, чтобы увидеть, какие это модули
degraded

# система загружена в аварийном (emergency) или восстановительном (rescue) режиме
maintenance

# systemd останавливается
stopping

# systemd не запущена
offline

# существует проблема, не позволяющая systemd определить текущее состояние
unknown
```














