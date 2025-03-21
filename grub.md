### [:diamond_shape_with_a_dot_inside:](#toc) <a name='3'>Управление загрузчиком GRUB</a>


_**Задача:** Устройство конфигурационных файлов GRUB._

```ruby
Конфигурационные файлы GRUB находятся в:

# Главный конфигурационный файл GRUB. Cоздается в процессе сборки конфигурации после внесения изменений
# в файлы из /etc/grub.d/ и /etc/default/grub.
1. /boot/grub/grub.cfg

# Хранит файлы изображений и оформлений для настройки внешнего вида меню GRUB
2. /boot/grub/

# Настройки внешнего вида меню GRUB
3. /etc/default/grub

# Поддерживают более сложные конфигурации
4. /etc/grub.d/

# Каталог /etc/grub.d/ содержит целый комплекс конфигурационных файлов. Вместо одного конф файла
# настройки GRUB хранятся в нескольких отдельных файлах, и каждый отвечает за конкретную задачу.
# Файлы пронумерованы в том порядке, в котором GRUB должен их читать: чем меньше число, тем приоритетнее.
# Каждый из этих файлов является сценарием и должен иметь разрешение на выполнение.
# Вы можете запретить использовать любой из них, сбросив бит разрешения на выполнение.
```

<br>

_**Задача:** Отображение скрытого меню GRUB._

```ruby
$ sudo nano /etc/default/grub
GRUB_TIMEOUT="10"
GRUB_TIMEOUT_STYLE=menu

# Закоментировать
GRUB_HIDDEN_TIMEOUT=0 и GRUB_HIDDEN_TIMEOUT_QUIET=true

# Собрать конфигурацию
$ sudo update-grub или sudo grub2-mkconfig -o /boot/grub2/grub.cfg, grub-mkconfig -o /boot/grub/grub.cfg


# Параметр GRUB_HIDDEN_TIMEOUT=0 запрещает отображение меню GRUB, а параметр
# GRUB_HIDDEN_TIMEOUT_QUIET=true запрещает отображение таймера обратного отсчета. Если установить
# на компьютер еще одну операционную систему в режиме мультизагрузки,
# то меню GRUB должно стать видимым автоматически.

# Страницы справочного руководства: man -k grub, info grub или info grub2
```

<br>

_**Задача:** Собрать конфигурацию GRUB после изменения настроек._

```ruby
# Команды сборки конфигурации GRUB в разных дистрибутивах могут различаться.
# В Fedora и openSUSE эта команда выглядит так:
$ sudo grub2-mkconfig -o /boot/grub2/grub.cfg

# В некоторых дистрибутивах, таких как Ubuntu, она имеет следующий вид:
$ sudo grub-mkconfig -o /boot/grub/grub.cfg

# В Ubuntu Linux имеется также сценарий update-grub, который запускает grubmkconfig:
$ sudo update-grub

# Страницы справочного руководства: man -k grub, info grub или info grub2
```

<br>	

_**Задача:** Создание минимального конфигурационного файла GRUB._

```ruby
$ nano /etc/default/grub
GRUB_DEFAULT=0
GRUB_TIMEOUT=10
GRUB_TIMEOUT_STYLE=menu

$ sudo awk -F\' '/menuentry / {print i++,$2}' /boot/grub/grub.cfg

# Изменив этот файл, выполните команду: sudo update-grub или sudo grub2-mkconfig -o /boot/grub2/grub.cfg,
# sudo grub-mkconfig -o /boot/grub/grub.cfg для обновления /boot/grub2/grub.cfg

# Linux книга рецептов. Карла Шрёдер, стр. 73
```

<br>

_**Задача:** Настройка фонового изображения для меню GRUB._

```ruby
1. Скопируйте изображение в /boot/grub/
2. nano /etc/default/grub или /etc/sysconfig/grub2 
   GRUB_BACKGROUND="/boot/grub/duchess-books.jpg"
3. grub-mkconfig -o /boot/grub/grub.cfg

# Linux книга рецептов. Карла Шрёдер, стр. 73
```

<br>

_**Задача:** Восстановление незагружающейся системы из приглашения grub>_

```ruby
# Когда процесс загрузки останавливается на приглашении grub>, это
# означает, что загрузчик нашел каталог /boot/grub/, 
# но не может найти корневую файловую систему.

# Для решения нужно найти корневую файловую систему, ядро Linux и соответствующий файл initrd.

grub> set pager=1

# GRUB поддерживает свой способ идентификации жестких дисков и разделов. 
# Он нумерует диски, начиная с 0, а разделы — с 1, и маркирует все жесткие диски как hd. 
# В работающей системе Linux жесткие диски обозначаются как /dev/sda, /dev/sdb и т. д

# Команда grub> ls обнаружила два жестких диска, hd0 и hd1, 
# которые соответствуют устройствам /dev/sda и /dev/sdb. 
# Раздел hd0,gpt5 соответствует разделу /dev/sda5, а hd1,msdos1 — разделу /dev/sdb1

grub> ls
grub> ls (hd0,3)

# Слеш означает «список всех файлов и каталогов в разделе»
grub> ls (hd0,2)/

# Все загрузочные файлы находятся в каталоге /boot
grub> ls (hd0,2)/boot

grub> set root=(hd0,2)
grub> linux /boot/vmlinuz-5.3.18-lp152.57-default root=/dev/sda2
grub> initrd /boot/initrd-5.3.18-lp152.57-default
grub> boot

# Ядро Linux
vmlinuz-5.3.18-lp152.57-default
# Временная корневая файловая система, используемая только в начале запуска системы.
initrd-5.3.18-lp152.57-default

# Linux книга рецептов. Карла Шрёдер, стр. 82
```

<br>

_**Задача:** Восстановление системы из приглашения grub rescue>_

```ruby

# Если процесс загрузки останавливается на приглашении grub rescue>
# аварийной командной оболочки, это означает, что загрузчик не нашел каталог /boot.

# Вывести список разделов
grub rescue> ls

# Вывести типы файловой системы, UUID и другую информацию
grub rescue> ls (hd0,3)

# Слеш означает «список всех файлов и каталогов в разделе»
grub rescue> ls (hd0,2)/

# Вывести список файлов в /boot
grub rescue> ls (hd0,2)/boot

# В приглашении grub rescue> необходимо выполнить несколько дополнительных
# команд: вы должны указать, где находится каталог /boot/grub, а затем загрузить
# модули ядра normal и linux, которые находятся в /boot/grub/i386-pc (вместе
# со многими другими модулями ядра, которые GRUB использует при запуске).
# Модуль normal изменяет режим загрузки с аварийного на нормальный, а linux
# запускает системный загрузчик:

grub rescue> set prefix=(hd0,2)/boot/grub
grub rescue> set root=(hd0,2)
grub rescue> insmod normal
grub rescue> insmod linux

# После этого нужно сообщить загрузчику GRUB, где ядро и файл initrd:

grub> linux /boot/vmlinuz-5.3.18-lp152.57-default root=/dev/sda2
grub> initrd /boot/initrd-5.3.18-lp152.57-default
grub> boot

# Linux книга рецептов. Карла Шрёдер, стр. 85

```


<br>

_**Задача:** Переустановка конфигурации GRUB_

```ruby

$ sudo grub-mkconfig -o /boot/grub/grub.cfg
$ sudo grub-install /dev/sda

# Прежде чем выполнять манипуляции с загрузчиком, создайте резервные копии
# своих данных. Если восстановить загрузку системы не удалось, то попробуйте
# переустановить GRUB с диска SystemRescue

# Linux книга рецептов. Карла Шрёдер, стр. 87

```

