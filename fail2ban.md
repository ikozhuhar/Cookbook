## Как пользоваться


<br>

```ruby
# Смотрим какие есть jail для того, чтобы в них добавлять правила
fail2ban-client status

# Добавляем
# fail2ban-client set <JAIL> addignoreip <IP>
fail2ban-client set sshd addignoreip 185.63.60.81
fail2ban-client set sshd addignoreip 185.63.61.33
fail2ban-client set exim-isp addignoreip 185.63.61.33
fail2ban-client set exim-isp addignoreip 185.63.60.81

# Смотрим
fail2ban-client get sshd ignoreip
fail2ban-client get exim-isp ignoreip

# Удаляем
fail2ban-client set sshd delignoreip 185.63.61.33
fail2ban-client set exim-isp delignoreip 185.63.61.33
```

```ruby
# Linux man page
https://linux.die.net/man/1/fail2ban-client
```
