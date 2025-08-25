# 42_Born2beRoot
You can do anything you want to do. Virtual Machine, this is your world!
This server was made using Debian 13.

## Download Image
The image used is the net install. Click to [here](https://www.debian.org/) and hit Download.

## Instalation through VirtualBox
After choosing language, your local, locale options and keyboard layout the installation starts.
- Insert hostname: mmaquine42
- Insira a domain name
- Insert root password
- Retype root passowrd
Insert a name for a User, this user account will be crated for you to use instead of the root account fot non-administrative activities
Insert you 42 login as username
Insert a password for login and retype
Choose clock
On [!] Partition disks
Select method "Guided - use entire disk an set up LVM"  
X -> On partition disks, select Guided - use entire disk and set up LVM
X -> Select separet /home partition
Cancel the erasing data
Choose Yes on Write changes to disks and configure LVM
Just hit continue when choosing amount of volume group to use guided partitioning
Write changes to disk - YES
Left blank htt proxy information. It will start configuring apt

[IMPORTANT STEP]
On [!] Software selection unmark all Debian desktop environment
Mark WEB SERVER and SSH SERVER

Not necessary install GRUB. On [!] Configuring grup-pc choose /dev/sda

## Installing packages
These packages must be installed has root user. Use su root and type root password:
apt-get install sudo
apt-get install ufw
apt-get install libpam-pwquality
apt-get install net-tools
apt-get install apparmor apparmor-profiles apparmor-utils

## Configure AppArmor

## Configuring SSH
Edit file /etc/ssh/sshd_config with nano
- On line #Port 22, remove # and change 22 to 4242
- On the line PermitRootLogin edit option to no (remove #)

## Configuring UFW
Uncomplicated Firewall (ufw) provides a framework for managing netfilter, as well a command-line interface for manipulating the firewall.
Enable SSH:
ufw allow OpenSSH
Testar em casa o allow OpenSSH
Allow 4242 to ssh:
ufw allow 4242/tcp

## Create group
addgroup user42

## Configuring SUDO rules

## Configuring password rules

### Configure VirtualBox to accepet SSH for your virtual machine
Recommended for Host-to_Guest
- In VirtualBox, select guest VM, go to Settings > Network > Adapter 1.
- Set "Attached to:" to NAT.
- Click Advanced > Port Forwarding.
- Add a new rule:
	- Name: SSH (or any descriptive name)
	- Protocol: TCP
	- Host Port: Choose an unused port on your host machine (e.g., 4242).
	- Guest port: 4242 (the port configured on SSH service)
	- Leave Host IP and Guest IP blank.
After that, to login via ssh on a terminal:
ssh <user>@127.0.0.1 -p 4242

## Adding a user
useradd LOGIN -m -c "NAME" -G grp1,grp2 -s /bin/bash -p "$(openssl passwd -6 "SENHA")"