---
created: 2023-12-10T13:53
updated: 2023-12-17T12:26
---
##### ROOT - IPTABLES
```
apt update
apt dist-upgrade
```
##### ROOT - SSH
```
apt install sudo
useradd -s /bin/bash -m administrateur
passwd administrateur
usermod -aG sudo administrateur
umask 077; mkdir /home/administrateur/.ssh
cat /root/.ssh/authorized_keys > /home/administrateur/.ssh/authorized_keys
chown -R administrateur:administrateur /home/administrateur/.ssh
echo "PasswordAuthentication no" > /etc/ssh/sshd_config.d/rule.conf
service ssh restart
```
##### ROOT - IPTABLES
```
apt install iptables -y
iptables -F INPUT
iptables -P INPUT DROP
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp -m comment --comment "SSH" --dport 22 -j ACCEPT
iptables-save > .iptables
echo -e "[Unit]\nDescription=Execute iptables-restore command\n\n[Service]\nExecStart=iptables-restore /root/.iptables\nType=simple\nUser=root\nGroup=root\nWorkingDirectory=/root\nRestart=on-failure\n\n[Install]\nWantedBy=multi-user.target" > /etc/systemd/system/iptables-restore.service
systemctl daemon-reload
systemctl enable iptables-restore
echo done
```