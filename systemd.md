### [:diamond_shape_with_a_dot_inside:](#toc) <a name='3'>systemd</a>

`systemd` в Linux — подсистема инициализации и управления службами. Основная особенность `systemd` — интенсивное распараллеливание запуска служб в процессе загрузки системы, что позволяет существенно ускорить запуск операционной системы.

```ruby
# Проверка использования systemd дистрибутиве Linux
sudo stat /sbin/init
sudo stat /proc/1/exe
cat /proc/1/comm
```

![image](https://github.com/user-attachments/assets/b87b502c-6bc2-4aaa-b4fd-94c2c0937935)
