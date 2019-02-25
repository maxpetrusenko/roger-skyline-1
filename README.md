# roger-skyline-1
42 Roger-Skyline1

Before launching the virtual machine. In VirtualBox, go to the host's network manager (files-> Host Network Manager or CMD + W). Click Create, and then uncheck the DHCP server. You should have a vboxnet0 host with the ip 192.168.56.1/24.

Go to the settings of the previously created Debian machine: Settings-> network. For card 1, leave access mode NAT, then enable card 2 in "Host-Only Adapter " mode. It will default to vboxnet0.

You can start the VM.

At the installation:

Create a 4.2Gb partition (in creation, create one of 4.501gb) with / in mountpoint.

Then a partition of 1gb type swap and finally with the rest of the hard disk, make a partition / home.

/! \ Install only ssh and usual services (not web or desktop environment)

PART 2: Up to ssh
Install the necessary packages for Roger-Skyline:

apt install -y sudo net-tools iptables-persist fail2ban sendmail apache2

Then modify the ssh parameters:

vim / etc / ssh / sshd_config

Change the port to 2222 (or whatever, but stay consistent in the rest of the project) then uncomment the PasswordAuthentification yes line. Uncomment #PermitRootLogin prohibit-password and replace with No. Uncomment #PubkeyAuthentication yes

Now create a new network interface that will link your machine to your host.

vim / etc / network / interfaces

And add the following content:

# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces (5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug enp0s3
iface enp0s3 inet dhcp

allow-hotplug enp0s8
iface enp0s8 inet static
address 192.168.56.3
netmask 255.255.255.252
Create a new user now:

adduser username

Enter the info and add it to the sudo group:

adduser username sudo

Restart the machine now: reboot

In a terminal of your host, generate a public ssh key:

ssh-keygen

Then cat ~ / .ssh / id_rsa.pub and copy the entire contents.

Connect now to your machine:

ssh user @ IPMACHINE -p 2222

Enter the password (required the first time) and create the .ssh folder in your user's home:

mkdir .ssh

Then paste the key into .ssh / authorized_keys

Then edit the ssh configuration file again:

sudo vim / etc / ssh / sshd_config and pass "PasswordAuthentification yes" to no.

Restart the ssh service: sudo service ssh restart

Exit the ssh with exit and retry a connection with the previous command. The connection is now password-free with PublicKeys. You will find your VM in the .ssh / known_hosts file.

PART 3: Firewall
The Firewall part is via iptables rules. You can list existing rules with:

sudo iptables -L

There are currently no rules. So add the file sudo vim /etc/network/if-pre-up.d/iptables

In this file add the following lines:

#! / Bin / bash

iptables-restore </etc/iptables.test.rules

iptables -F
iptables -X
iptables -t nat -F
iptables -t nat -X
iptables -t mangle -F
iptables -t mangle -X

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

exit 0
Then make this file executable.

sudo chmod + x /etc/network/if-pre-up.d/iptables

The iptables rules are reset at each reboot. This file will allow the iptables-persistent package to load your rules every time you reboot. Modify port 2222 by the port of your ssh.

PART 4: DOS
Create a log file for your Apache server.

sudo touch /var/log/apache2/server.log

The fail2ban package includes protections against common attacks. Just activate them by creating a local configuration file. This makes it possible not to modify directly the default jail.conf jails.

sudo vim /etc/fail2ban/jail.local

Then add the following jails:

[DEFAULT]
destemail = USER@student.le-101.fr
sender = root@roger-skyline.fr
