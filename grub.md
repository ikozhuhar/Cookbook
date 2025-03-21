### [:diamond_shape_with_a_dot_inside:](#toc) <a name='3'>Управление загрузчиком GRUB</a>

_Задача: Собрать конфигурацию GRUB после изменения настроек._

```ruby
# Команды сборки конфигурации GRUB в разных дистрибутивах могут различаться. В Fedora и openSUSE эта команда выглядит так:
$ sudo grub2-mkconfig -o /boot/grub2/grub.cfg

# В некоторых дистрибутивах, таких как Ubuntu, она имеет следующий вид:
$ sudo grub-mkconfig -o /boot/grub/grub.cfg

# В Ubuntu Linux имеется также сценарий update-grub, который запускает grubmkconfig:
$ sudo update-grub

# Страницы справочного руководства: man -k grub, info grub или info grub2
```
