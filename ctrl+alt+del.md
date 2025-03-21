_**Задача:** Перезагрузка с помощью Ctrl+Alt+Delete_

```ruby

# В системах Linux без systemd комбинация Ctrl+Alt+Delete настраивается в файле /etc/inittab.

# Что делать по нажатии CTRL+ALT+DEL.
ca:12345:ctrlaltdel:/sbin/shutdown -t1 -r now

# Последовательность цифр 12345 активирует комбинацию на уровнях запуска 1, 2, 3, 4 и 5.
# Параметр -t1 означает «ждать одну секунду», а -r означает reboot — «перезагрузка». Н

# Поведением Ctrl+Alt+Delete в systemd управляет /etc/systemd/system/ctrl-alt-del.target.
# На самом деле ctrl-alt-del.target это ссылка на /lib/systemd/system/reboot.target


# Linux книга рецептов. Карла Шрёдер, стр. 99

```
