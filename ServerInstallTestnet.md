## Preparing your server for testnet

These instructions will help you in preparing a server for the Oxycoin testnet network. 

### Prerequisites

To complete this tutorial, you will need:

A linux server with preferably Ubuntu 16.04 with at least 2 GB memory and 20 GB storage. 

To check your linux distribution you can use (on the command line):

```
cat /etc/issue
```

This will output something like:

Ubuntu 16.04.3 LTS \n \l

To check the amount of memory type:

```
free -h
```

This will output something like:

              total        used        free      shared  buff/cache   available
Mem:           3.8G        291M        188M        160M        3.3G        3.1G
Swap:          3.7G        2.0M        3.7G


In which the row 'Mem:' and column 'total' displays your total memory

To check the amount of storage type:

```
df -h
```

This will output something like:

Filesystem      Size  Used Avail Use% Mounted on
udev            1.9G     0  1.9G   0% /dev
tmpfs           391M   24M  368M   6% /run
/dev/md2        911G  5.2G  859G   1% /
tmpfs           2.0G  4.0K  2.0G   1% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/md0        472M   55M  393M  13% /boot
/dev/md1        3.7G  7.5M  3.4G   1% /tmp

Look at the column 'Mounted on' for the single / which indicates your used and available storage in GB (unless you have a seperate home directory mountpoint e.g. /home)



### Installation

A step by step series of examples that tell you have to get a development env running

Say what the step will be

```
Give the example
```

And repeat

```
until finished
```

End with an example of getting some data out of the system or using it for a little demo


## Built With

* [Dropwizard](http://www.dropwizard.io/1.0.2/docs/) - The web framework used
* [Maven](https://maven.apache.org/) - Dependency Management
* [ROME](https://rometools.github.io/rome/) - Used to generate RSS Feeds


## Acknowledgments

* Hat tip to anyone who's code was used
* Inspiration
* etc


01. To start login to your server with the root user:

ssh root@[your_server_ip-addres]
02. Now update and upgrade your system to the latest repositories:
apt-get update
apt-get upgrade
apt-get dist-upgrade
03. After this is completed it's time to setup your main user with sudo privileges. Add the user chosing your username:
useradd -d /home/[your_username] -m -s /bin/bash [your_username]
04. Set the password for the username (make a note of this password):
passwd [your_username]
05. Add the user to the sudo group:
usermod -aG sudo [your_username]
06. Now it's time to verify this user can login to your server. Exit the login session with the root user and login with the new user:
ssh [your_username]@[your_server_ip-addres]
07. Check if the user has root privileges (password of user is required):
sudo -l
This will ask for your password and if you do have sudo rights, will return with something similar to:

User [your_username] may run the following commands on [your hostname]:
    (ALL : ALL) ALL

If this doesn't happen your user doesn't have the required sudo privileges (return to step 5).
08. Now install the packages required for the remainder of the installation process.
sudo apt-get install vim iptables git
09. Configure your editing preferences. Assuming you will continu using vi(m), create a vimrc file:
cd
vi .vimrc
Add the following line to the empty .vimrc file:
set nocompatible
Exit vi by hitting ESC and then typing (and hitting ENTER afterwards):
:wq
Activate the changes to the .vimrc file by typing:
source .vimrc
10. Change the bash preferences file to always use vim instead of vi (read here why).
cd
vi .bashrc 
Add the following line to the .bashrc file:
alias vi='vim'
Exit vi by hitting ESC and then typing (and hitting ENTER afterwards):
:wq
Activate the changes to the .bashrc file by typing:
source .bashrc
11. Running the SSH daemon on your server on the default port (22) will generate quite some brute force login attempts. To run the SSH daemon on a different port follow the steps below:
sudo vi /etc/ssh/sshd_config
Find the line stating the port number most likely looking like this:
#Port 22
Uncomment the line (remove the #) and change 22 in a number somewhere between 10000 and 65000 e.g.:
Port 12322
12. Save and exit vi by hitting ESC and typing (and hitting ENTER afterwards):
:wq
13. Restart the SSH daemon:
sudo /etc/init.d/ssh restart
14. Verify you can still login by opening a new terminal and typing:

ssh -p[your_sshd_port] root@[your_server_ip-addres]
15. We are now ready to configure the firewall rules allowing you to manage your server using SSH and your server to communicate with the rest of the OXY network. The easiest way is to put the rules in a script by typing the following commands:

cd
vi firewall_rules.sh
Paste the following lines in the firewall_rules.sh script file (replace the [your_sshd_port] variable with the port number chosen in step 11):

#!/bin/bash
iptables -F

iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -p tcp --dport [your_sshd_port] -j ACCEPT
iptables -A INPUT -p tcp --dport 9998 -j ACCEPT
iptables -A INPUT -p tcp --dport 9999 -j ACCEPT
iptables -A INPUT -j DROP
iptables -I INPUT 1 -i lo -j ACCEPT

iptables -L -v -n
sh -c "iptables-save > /etc/iptables.rules"
Exit vi by hitting ESC and then typing (and hitting ENTER afterwards) and then make the script executable:

chmod +x firewall_rules.sh
Note that these rules will block all other traffic to your server other than management over SSH and communication with the OXY network. If you have any other services running, make sure these will still function by adding their incoming TCP/UDP ports to the firewall rules. 
16. Next is the activation of the firewall:  

cd
sudo ./firewall_rules.sh
After the script has executed it will display the firewall rules activated and this rules will be persistant. It's important to verify you can still login to your server with the firewall active so open a new terminal and login again:

ssh -p[your_sshd_port] root@[your_server_ip-addres]
