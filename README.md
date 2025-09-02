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
apt-get install apparmor apparmor-profiles apparmor-utils

## Configuring SSH
Edit file /etc/ssh/sshd_config with nano
- On line #Port 22, remove # and change 22 to 4242
- On the line PermitRootLogin edit option to no (remove #)

Edit file /etc/ssh/ssh_config with nano
- On line #Port 22, remove # and change 22 to 4242

## Configuring UFW
Uncomplicated Firewall (ufw) provides a framework for managing netfilter, as well a command-line interface for manipulating the firewall.
ufw enable
Allow 4242 to ssh:
ufw allow 4242/tcp

Use ufw deny <port> to block an acess

To delete some rule:
ufw status numbered
ufw delete <number>

## Create group
addgroup user42

## Configuring SUDO rules
Create following folder:
sudo mkdir -p /var/log/sudo
sudo chmod 700 /var/log/sudo
sudo mkdir -p /var/log/sudo-io
sudo chmod 700 /var/log/sudo-io
To the file using visudo
Defaults	passwd_tries=3
Defaults	badpass_message="Wrong password, contact server manager"
Defaults	logfile="/var/log/sudo/sudo.log"
Defaults	log_output
Defaults	iolog_dir="/var/log/sudo-io"
Defaults	requiretty
Defaults	secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"

To revise logs:
less /var/log/sudo/sudo.logs
sudo sudoreplay <ID>
## Configuring password rules
On /etc/login.defs edit these variables to become like this:
PASS_MAX_DAYS	30
PASS_MIN_DAYS	2
PASS_WARN_AGE	7

On /etc/security/pwquality.conf set ands remove #:
difok = 7
minlen = 10
dcredit = -1
ucredit = -1
lcredit = -1
maxrepeat = 3
usercheck = 1
enforcing = 1
enforce_for_root

run to root changes its password
chage -M 30 -m 2 -W 7 root
passwd
and retype new password

## Adding a user
edit file /etc/adduser.conf uncomentig these lines:
DSHELL=/bin/bash
DHOME=/home
USERS_GROUP=tty

adduser LOGIN --comment "NAME"
and type a temporary password
Force password change on first login
chage -d 0 LOGIN

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

## Monitoring Script
Create the file monitoring.sh on /usr/local/bin
#!/bin/bash

arch=$(uname -srvmo)
p_cpu=$(grep "physical id" /proc/cpuinfo | sort -u | wc -l)
v_cpu=$(grep -c "^processor" /proc/cpuinfo)
mem_total=$(free -m | awk '/Mem:/ {print $2}')
mem_used=$(free -m | awk '/Mem:/ {print $3}')
mem_percent=$(awk "BEGIN {printf \"%.2f\", $mem_used/$mem_total*100}")
disk_used=$(df -BG --total | grep total | awk '{print $3}' | sed 's/G//')
disk_total=$(df -BG --total | grep total | awk '{print $2}' | sed 's/G//')
disk_percent=$(df -BG --total | grep total | awk '{print $5}' | tr -d '%')
cpu_load=$(top -bn1 | grep "Cpu(s)" | awk '{print 100 - $8}' | awk '{printf "%.1f", $1}')
last_boot=$(who -b | awk '{print $3" "$4}')
lvm_use=$(lsblk | grep -q "lvm" && echo "yes" || echo "no")
tcp_connections=$(ss -ta | grep ESTAB | wc -l)
users_logged=$(who | wc -l)
ip_addr=$(hostname -I | awk '{print $1}')
mac_addr=$(ip link show | awk '/ether/ {print $2}' | head -n 1)
sudo_cmds=$(journalctl _COMM=sudo | grep COMMAND | wc -l)

host=$(hostname)
tty=$(tty | awk -F/ '{print $NF}')
datetime=$(date +"%a %b %d %H:%M:%S %Y")

msg="
Broadcast message from root@$host ($tty) ($datetime):
	#Architecture: $arch
	#CPU physical : $p_cpu
	#vCPU : $v_cpu
	#Memory Usage: $mem_used/${mem_total}MB (${mem_percent}%)
	#Disk Usage: ${disk_used}/${disk_total}Gb (${disk_percent}%)
	#CPU load: ${cpu_load}%
	#Last boot: $last_boot
	#LVM use: $lvm_use
	#Connections TCP : $tcp_connections ESTABLISHED
	#User log: $users_logged
	#Network: IP $ip_addr ($mac_addr)
	#Sudo : $sudo_cmds cmd
"
if [ ! -f /tmp/startup-mesg-shown ]; then
	echo "$msg" | sudo tee  /etc/issue > /dev/null
        touch /tmp/startup-mesg-shown
fi

for	tty in /dev/pts/*; do
	[ -w "$tty" ] && echo "$msg" > "$tty"
done
wall "$msg"

To run every 10 minutes do this:
sudo crontab -e
*/10 * * * *	/usr/local/bin/monitoring.sh
@reboot			/usr/local/bin/monitoring.sh

on the end o file /etc/profile add the script

## set new hostname
To change the machine name do:
sudo hostnamectl set-hostname NEW_NAME
After this just logof and login to see the the new name.