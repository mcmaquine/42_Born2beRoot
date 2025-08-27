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

## Configure AppArmor

## Configuring SSH
Edit file /etc/ssh/sshd_config with nano
- On line #Port 22, remove # and change 22 to 4242
- On the line PermitRootLogin edit option to no (remove #)

## Configuring UFW
Uncomplicated Firewall (ufw) provides a framework for managing netfilter, as well a command-line interface for manipulating the firewall.
Allow 4242 to ssh:
ufw allow 4242/tcp

## Create group
addgroup user42

## Configuring SUDO rules
Create following folder:
sudo mkdir -p /var/log/sudo_logs
sudo chmod 700 /var/log/sudo_logs
sudo mkdir -p /var/log/sudo-io
sudo chmod 700 /var/log/sudo-io
Add these lines to /etc/sudoers
Defaults	passwd_tries=3
Defaults	badpass_message="Wrong password, contact server manager"
Defaults	logfile="/var/log/sudo.log"
Defaults	log_output
Defaults	iolog_dir="/var/log/sudo-io"
Defaults	requiretty
Defaults	secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"

To revise logs:
less /var/log/sudo
sudo sudoreplay <ID>
## Configuring password rules
On /etc/login.defs edit these variables to become like this:
PASS_MAX_DAYS	30
PASS_MIN_DAYS	2
PASS_WARN_AGE	7

run to root changes its password
chage -M 30 -m 2 -W 7 root
passwd
and retype new password

On /etc/security/pwquality.conf set:
minlen = 10
ucredit = -1
lcredit = -1
dcredit = -1
maxrepeat = 3
usercheck = 1
difok = 7
enforcing = 1

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

## Monitoring Script
#!/bin/bash

monitoring_info() {
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
sudo_cmds=$(grep -c "COMMAND=" /var/log/sudo/sudo.log 2>/dev/null || echo 0)

host=$(hostname)
tty=$(tty | awk -F/ '{print $NF}')
datetime=$(date +"%a %b %d %H:%M:%S %Y")

msg="Broadcast message from root@$host ($tty) ($datetime):
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

for 	tty in /dev/pts/*; do
	[ -w "$tty" ] && echo "$msg" > "$tty"
done
}
monitoring_info

To run every 10 minutes do this:
sudo nano /etc/systemd/system/monitoring.service
Put this:
[Unit]
Description=Monitoring script

[Service]
Type=oneshot
ExecStart=/root/monitoring.sh

And do this too:
sudo nano /etc/systemd/system/monitoring.timer
[Unit]
Description=Run monitoring recurrently even on boot

[Timer]
OnBootSec=1min
OnUnitActiveSec=10min
AccuracySec=1sec	# 1 second precision
Unit=monitoring.service
Persistent=true

[Install]
WantedBy=timers.target

After crating these two services:
sudo systemctl daemon-reload
sudo systemctl enable monitoring.timer
sudo systemctl start monitoring.timer


## set new hostname
Para mudar o hostname no Debian, use os comandos hostnamectl set-hostname <novo_nome> para uma alteração imediata e permanente, e depois edite o arquivo /etc/hosts para refletir o novo nome, substituindo o antigo nome de host na linha 127.0.1.1 ou 127.0.0.1 pelo novo nome. O uso do comando hostnamectl é o método recomendado para alterar o hostname em distribuições Linux modernas como o Debian. 
Passos detalhados:

    Abra o Terminal: Inicie um terminal no seu sistema Debian. 

Torne-se Root: Execute o comando sudo -s ou su - para obter privilégios de superusuário. 
Altere o Hostname: Use o comando hostnamectl para definir o novo nome: 

Código

    sudo hostnamectl set-hostname <nome_do_novo_host>

    Substitua <nome_do_novo_host> pelo nome que você deseja. 

Este comando altera o hostname de forma temporária e o salva para a próxima inicialização. 

    Edite o Arquivo /etc/hosts: Abra o arquivo de configuração /etc/hosts com um editor de texto, como nano: 

Código

    sudo nano /etc/hosts

    1. Atualize a Entrada:
    Encontre a linha que começa com 127.0.1.1 (ou 127.0.0.1 dependendo da configuração) e substitua o nome de host antigo pelo novo. 

    Exemplo: se o antigo nome era servidor-antigo, a linha pode ser 127.0.1.1 servidor-antigo.
    Altere-a para 127.0.1.1 <nome_do_novo_host>. 

2. Salve e Saia:
Salve o arquivo e feche o editor (no nano, pressione Ctrl + X, depois Y e Enter). 
3. Verifique a Alteração:
O novo nome de host deve aparecer após reiniciar a sessão de terminal ou reiniciar o computador. Você também pode verificar o hostname com o comando hostnamectl. 
