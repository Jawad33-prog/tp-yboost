# TP YBOOST : Sécuriser un serveur Debian (Jawad Aniymar)
## 1/ Mettre les paquets dans leur version la plus à jour possible :
````powershell
apt-get update
apt-get upgrade 
````
## 2/ Installer et configurer un pare-feu pour bloquer tout sauf les services nécessaires :
````powershell
sudo apt install ufw -y
sudo ufw allow 'Nginx FULL'
sudo ufw allow OpenSSH   
sudo ufw enable  
sudo ufw status    
````
## 3/ Faire en sorte que les répertoires ne soient lisible que par leur propriétaire respectif :
````powershell
dpkg-reconfigure adduser
````
## 4/ Configurer l'authentification par clé SSH :
````powershell
openssl req -new -newkey rsa:4096 -days 365 -nodes -x509 -keyout server.key -out server.crt
sudo mv server.key /etc/ssl/private/server.key
sudo mv server.crt /etc/ssl/certs/server.crt
sudo chown root:root /etc/ssl/private/server.key
sudo chown root:root /etc/ssl/certs/server.crt
sudo nano /etc/ssh/sshd_config
    PasswordAuthentication no
    PermitRootLogin yes
````
## 5/ Installer Fail2Ban pour protèger contre les attaques par force brute :
````powershell
sudo apt install fail2ban -y
sudo nano /etc/fail2ban/jail.local
    [sshd]
    enabled = true
    port    = ssh
    logpath = /var/log/auth.log
    maxretry = 3
    bantime  = 3600
systemctl restart fail2ban
````
## 6/ Installer un IDS/IPS pour détecter et prévenir les attaques réseau :
````powershell
apt install suricata -y
nano /etc/suricata/suricata.yaml
    af-packet:
    - interface: ens18
        threads: auto
        cluster-id: 99
        cluster-type: cluster_flow
        defrag: yes
systemctl restart suricata
````
## 7/ Configurer les permissions sur /boot et autres répertoires critiques :
````powershell
sudo chmod 700 /boot
sudo chmod -R o-rwx /etc
````
## 8/ Configurer les limites des connexions simultanées pour se protéger contre les attaques DoS :
````powershell
sudo nano /etc/security/limits.conf
    * hard nofile 4096
sudo nano /etc/sysctl.conf
    net.ipv4.tcp_syncookies = 1
    net.ipv4.tcp_max_syn_backlog = 2048
    net.ipv4.icmp_echo_ignore_broadcasts = 1
    net.ipv4.conf.all.log_martians = 1
````