# roger-skyline-1
42 Roger-Skyline1

apt install -y vim sudo net-tools iptables-persistent fail2ban sendmail apache2 crontab

nano /etc/ssh/sshd_config and change lines to:
        Port 2222
        
        PasswordAuthentification yes
        
        PermitRootLogin no
        
        PubkeyAuthentication yes
        
nano  / etc / network / interfaces

//delete everything and copy-paste:

    source /etc/network/interfaces.d/*
    
    The loopback network interface
    
    auto lo iface lo inet loopback

    allow-hotplug enp0s3 iface enp0s3 inet dhcp
    
    allow-hotplug enp0s8 iface enp0s8 inet static address 192.168.56.3 netmask 255.255.255.252

su root

adduser username

adduser username sudo

#restart and login as username you jsut created

ssh-keygen

cat ~ / .ssh / id_rsa.pub and copy the entire contents.

#connect to you machine:

ssh root@debian -p 2222

mkdir .ssh

Then paste the key into .ssh / authorized_keys

sudo nano /etc/ssh/sshd_config

//change line to:

PasswordAuthentification no.

sudo service ssh restart

#Exit the ssh with exit and retry a connection with the previous command.

#The connection is now password-free with PublicKeys.

#You will find your VM in the .ssh / known_hosts file.



sudo nano /etc/network/if-pre-up.d/iptables

//enter this:

#! / Bin / bash

iptables-restore </etc/iptables.test.rules

iptables -F iptables -X iptables -t nat -F iptables -t nat -X iptables -t mangle -F iptables -t mangle -X

iptables -P INPUT DROP

iptables -P OUTPUT DROP

iptables -P FORWARD DROP

iptables -A INPUT -m conntrack -ctstate ESTABLISHED, RELATED -j ACCEPT

iptables -A INPUT -p tcp -i enp0s8 -dport 2222 -j ACCEPT

iptables -A INPUT -p tcp -i enp0s8 -dport 80 -j ACCEPT

iptables -A INPUT -p tcp -i enp0s8 -dport 443 -j ACCEPT

iptables -A OUTPUT -m conntrack! --ctstate INVALID -j ACCEPT

iptables -I INPUT -i lo -j ACCEPT

iptables -A INPUT -j LOG

iptables -A FORWARD -j LOG

iptables -I INPUT -p tcp -dport 80 -m connlimit -connlimit-above 10 -connlimit-mask 20 -j DROP



sudo chmod +x /etc/network/if-pre-up.d/iptables

The iptables rules are reset at each reboot. This file will allow the iptables-persistent package to load your rules every 
time you reboot. Modify port 2222 by the port of your ssh.



sudo touch /var/log/apache2/server.log

sudo vim /etc/fail2ban/jail.local

[DEFAULT] destemail = USER@student.42.us.org sender = root@debian

