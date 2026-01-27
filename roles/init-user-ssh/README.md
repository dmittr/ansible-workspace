Before use make sure you able to login as root with ssh-key on port 22

Steps:
- Create user from inventory with root's authorized_keys
- firewalld hardening - zone drop as default
- sshd hardening - custom port, no passwords, no root logins
