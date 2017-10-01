## Preparing your server for testnet or mainnet

These instructions will guide you through the steps preparing a server for the OXY testnet or mainnet network. After you're done with these preparations, please continue with the testnet/mainnet installation for an OXY node

### Prerequisites

To complete this tutorial, you will need:

* A Linux server with preferably Ubuntu 16.04 with at least 2 GB memory and 20 GB storage. 

* Some knowledge of how to work with command line tools in a terminal session such as Putty on Windows or Terminal on Linux/MacOSX. 

### Prerequisite verification (proceed to 'Installation' if you know the prerequisites are met)

0. Login to your server:

```
ssh username@[your_server_ip_address]
```

To check your linux distribution you can use:

```
cat /etc/issue
```

This will output something like:

```
Ubuntu 16.04.3 LTS \n \l
```

To check the amount of memory type:

```
free -h
```

This will output something like:
```
              total        used        free      shared  buff/cache   available
Mem:           3.8G        291M        188M        160M        3.3G        3.1G
Swap:          3.7G        2.0M        3.7G
```

In which the row 'Mem:' and column 'total' displays your total memory which is in this example 3.8GB (~4GB)

To check the amount of storage type:

```
df -h
```

This will output something like:
```
Filesystem      Size  Used Avail Use% Mounted on
udev            1.9G     0  1.9G   0% /dev
tmpfs           391M   24M  368M   6% /run
/dev/md2        911G  5.2G  859G   1% /
tmpfs           2.0G  4.0K  2.0G   1% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/md0        472M   55M  393M  13% /boot
/dev/md1        3.7G  7.5M  3.4G   1% /tmp
```
Look at the column 'Mounted on' for the single / which indicates your used and available storage in GB (unless you have a seperate home directory partition mounted on e.g. /home in which case you need to also check this mountpoint for available space)


### Installation

The installation document assumes you're installing a new server without (sudo) users already configured. If you do already have a sudo user configured on your server, you can login with this user and replace all commands executed by root below with: sudo [command]

1. To start login to your server with the root user:

```
ssh root@[your_server_ip_address]
```

2. Update and upgrade your system to the latest repositories:

```
apt-get update
apt-get upgrade
apt-get dist-upgrade
```

3. After this has completed it's time to setup your main user with sudo privileges. Add the user choosing your username:

```
useradd -d /home/[your_username] -m -s /bin/bash [your_username]
```

4. Set the password for the username (and make a note of this password):

```
passwd [your_username]
```

5. Add the user to the sudo group:

```
usermod -aG sudo [your_username]
```

6. Now it's time to verify this user can login to your server. Exit the login session with the root user:

```
exit
```

and login with the new user:

```
ssh [your_username]@[your_server_ip_address]
```

7. Check if the user has sudo privileges (password of user is required):

```
sudo -l
```

This will ask for your password and if you do have sudo rights, will return with something similar to:

```
User [your_username] may run the following commands on [your hostname]:
    (ALL : ALL) ALL
```

If this doesn't happen your user doesn't have the required sudo privileges (return to step 5)

8. Now install the packages required for the remainder of the installation process.

```
sudo apt-get install vim iptables git
```

9. Configure your editing preferences. Assuming you will continu using vi/vim, create a vimrc file (please refer to 'External reference i' for a vi/vim quick reference card):
```
cd
vi .vimrc
```
Add the following line to the empty .vimrc file (type i for insert mode and then type the following):
```
set nocompatible
```
Exit vi by hitting ESC and then typing (and hitting ENTER afterwards):
```
:wq
```
Activate the changes to the .vimrc file by typing:
```
source .vimrc
```

10. Change the bash preferences file to always use vim instead of vi (please refer to 'External reference ii' for advantages).

```
cd
vi .bashrc 
```
Add the following line to the .bashrc file (by default there's an aliases section but anywhere will work fine):
```
alias vi='vim'
```
Exit vi by hitting ESC and then typing (and hitting ENTER afterwards):
```
:wq
```
Activate the changes to the .bashrc file by typing:
```
source .bashrc
```

11. Running the SSH daemon on your server on the default port (22) will generate quite some brute force login attempts. To run the SSH daemon on a different port follow the steps below:

```
sudo vi /etc/ssh/sshd_config
```
Find the line stating the port number looking like this:
```
#Port 22
```

Uncomment the line (remove the #) and change 22 in a number somewhere between 11000 and 65000 e.g.:

```
Port 12322
```

12. Save and exit vi by hitting ESC and typing (and hitting ENTER afterwards):
```
:wq
```

13. Restart the SSH daemon:

```
sudo /etc/init.d/ssh restart
```

14. Verify you can still login by opening a new terminal and typing:

```
ssh -p[your_sshd_port] [your_username]@[your_server_ip_address]
```

15. We are now ready to configure the firewall rules allowing you to manage your server using SSH and your server to communicate with the rest of the OXY network. The easiest way is to put the rules in a script by typing the following commands:

```
cd
vi firewall_rules.sh
```
15.1 For *testnet* paste the following lines in the firewall_rules.sh script file (replace the [your_sshd_port] variable with the port number chosen in step 11):
```
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
```

15.2 For *mainnet* paste the following lines in the firewall_rules.sh script file (replace the [your_sshd_port] variable with the port number chosen in step 11):
```
#!/bin/bash
iptables -F

iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -p tcp --dport [your_sshd_port] -j ACCEPT
iptables -A INPUT -p tcp --dport 10000 -j ACCEPT
iptables -A INPUT -p tcp --dport 10001 -j ACCEPT
iptables -A INPUT -j DROP
iptables -I INPUT 1 -i lo -j ACCEPT

iptables -L -v -n
sh -c "iptables-save > /etc/iptables.rules"
```

Exit vi by hitting ESC and then typing :wq (and hitting ENTER afterwards) and then make the script executable:
```
chmod +x firewall_rules.sh
```

Note that these rules will block all other traffic to your server other than management over SSH and communication with the OXY network. If you have any other services running, make sure these will still function by adding their incoming TCP/UDP ports to the firewall rules. 

16. Next is the activation of the firewall:  

```
cd
sudo ./firewall_rules.sh
```

After the script has executed it will display the firewall rules activated and these rules will be persistant. It's important to verify you can still login to your server with the firewall active so open a new terminal and login again:

```
ssh -p[your_sshd_port] [your_username]@[your_server_ip_address]
```

Exit all your login sessions:

```
exit
```

You are all set up and you can proceed to the guide for installing the node


## Advanced options

If you don't want to type your password every time you login to your server you can configure PublicKey Authentication for SSH. Besides the fact it's more secure, it's very convenient as you don't have to type your password anymore

17. Start by generating a SSH keypair (if you don't already have a keypair) on your local machine (for Windows/Putty use the PuTTYgen utility):

```
ssh-keygen
```

This will generate, by default, a RSA keypair named id_rsa and id_rsa.pub. The id_rsa file is your private key and should never be distributed to machines you don't own. The id_rsa.pub is your public key and can freely be distributed for your needs

Your public key needs to copied onto your server in the users home SSH directory e.g. /home/user/.ssh/authorized_keys


18.1 On Linux distributions there's a tool available for copying your public key to the server, adding it to the users authorized_keys file and setting the correct permissions:

```
ssh-copy-id -i ~/.ssh/id_rsa -p[your_sshd_port] [your_username]@[your_server_ip_address]
```

18.2 On MacOSX/Windows this tool is not available by default. Open the id_rsa.pub you created in a text-editor and copy the contents to the clipboard on your machine. To copy the key to the authorized_keys file, login to your server and type the following commands:

```
ssh -p[your_sshd_port] [your_username]@[your_server_ip_address]

mkdir .ssh
chmod 0700 .ssh/
chown [your_username]:[your_username] .ssh/
vi .ssh/authorized_keys
```

Now paste the contents of your clipboard holding the id_rsa.pub to this file and exit vi by hitting ESC and then typing :wq (and hitting ENTER afterwards)

19. By default SSH allows PublicKey Authentication but let's make sure it does:

```
sudo vi /etc/ssh/sshd_config
```

And make sure the following line is present and not commented:

```
PubkeyAuthentication yes
```

20. If you want to further secure SSH by only allowing [your_username] to login usign a public key only make sure the following lines are also present in the sshd_config file:

```
PermitRootLogin no
PasswordAuthentication no
AllowUsers [your_username]
```

Make sure you replace [your_username] with the username you want to allow logging in. Exit vi and restart the SSH Daemon:

```
sudo /etc/init.d/ssh restart
```

21. It's very important to verify you can still login to your server (now using PublicKey Authentication) so open a new terminal and login again:

```
ssh -i ~/.ssh/id_rsa -p[your_sshd_port] [your_username]@[your_server_ip_address]
```

22. If you can login successfully using PublicKey Authentication, exit all login sessions and continue installing the OXY node

```
exit
```


## External references

* 1. <a href="http://silverwraith.com/papers/vi-quickref.txt" target="_blank">A vi quick reference card</a>
* 2. <a href="https://vimhelp.appspot.com/vim_faq.txt.html#faq-1.4" target="_blank">Some advantages in using vim over vi</a>


## Acknowledgments

* <a href="https://github.com/seatrips/" target="_blank">Seatrips</a>
* <a href="https://github.com/lepetitjan/" target="_blank">Lepetitjan</a>
