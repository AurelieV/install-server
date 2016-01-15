# install-server
Instruction for prepare a private Ubuntu 15.04 server for handle node app with a mongo base

## Security install
ssh with root account on your server

### Install sudo and update apt-get
```bash
$ apt-get update
$ apt-get install sudo
```

### Create a user with sudo right
```bash
$ adduser YOUR_USER_NAME
$ adduser YOUR_USER_NAME sudo
```

### Configure ssh
It's more secure to change port number for ssh (it avoid you most of auto attack).

We also deny root access by ssh
```bash
$ vi /etc/ssh/sshd_config
```
* Change `Port` value (for example 2222) 
* Pass `PermitRootLogin` to `no` /!\ Be sure to have an other user !
* Restart ssh
```bash
$ /etc/init.d/ssh restart
```

### Simple firewall configuration
We will use iptable for this.
```bash
$ apt-get install iptables
$ vi /etc/init.d/firewall
```
Copy the `firewall.sh` content in the file

* By default we close all ports, and reopen only necessary ones.
* /!\/!\ Think about changing the ssh port number in the file from the one you choose in previous step (line 33-34)
* Make the file executable

```bash
$ chmod +x /etc/init.d/firewall
```
* Launch it
```bash
$ /etc/init.d/firewall
```
* Add it to boot script (Ubuntu 14 version)
```bash
$ update-rc.d firewall defaults
```
* Add it to boot script (Ubuntu 15 version)
Add the line in `/etc/rc.local` 
```bash
$ sudo /etc/init.d/firewall
```

### Block ports scanning with portsentry (DO NOT USE IF CONNECTING WITH VARIOUS IP)
```bash
$ apt-get install portsentry
```
* Edit `/etc/portsentry/portsentry.ignore` and add your local IP (if no, you will be blocked !)
* Edit `/etc/default/portsentry` and put this content
```
TCP_MODE="atcp"
UDP_MODE="audp"
```
* Edit `/etc/portsentry/portsentry.conf` file
    * Modify ``IGNORE OPTION` by (default value is 0)
    ```
    BLOCK_UDP="1"
    BLOCK_TCP="1"
    ```
    * Comment `KILL_HOSTS_DENY` lines
    * Because we use iptables, comment all lines begining with `KILL_ROUTE”` EXCEPT `KILL_ROUTE="/sbin/iptables -I INPUT -s $TARGET$ -j DROP"`
* Restart service
```bash
$ service portsentry restart
```

### Install fail2ban
TO DO

### For the next connection
Now, always connect with the user you create

## Install node
Connect via ssh to your server, using the user you create (and not root). Care about precising the ssh port you define
```bash
$ ssh my_user@mon_ip -p 2222
```

### Create a user for node (better for isolate node process) and connect with it
```bash
$ sudo useradd nodejs
$ sudo su nodejs
```

### Create directory and npm config
* Create a local directory
```bash
$ mkdir ~/.local
```
* Create a `.npmrc` file and put the content of `.npmrc` of this repo
```bash
$ vi ~/.npmrc
```

### Download and install node
* Go to node download page, and find the download link to the version you want. 
```bash
$ wget https://github.com/joyent/node/archive/v0.10.29.tar.gz
```
* Unzip the files and go to directory
```bash
$ tar -xzvf ~/v0.10.29.tar.gz
$ cd ~/node-0.10.29
```
* Configure the build
```bash
$ ./configure --prefix=~/.local
```
* Build and install (may be long)
```bash
$ make
$ make install
```
* Add some symbolic link and path
```bash
$ ln -s ~/.local/lib/node_modules ~/.node_modules
$ export PATH=$HOME/.local/bin:$PATH
```
* Check if everything is ok
```bash
$ node -v
$ npm -v
```
* Clean
```bash
$ rm -R /home/nodejs/node-0.10.29
$ rm /home/nodejs/v0.10.29.tar.gz
```
* Now for using node, connect as nodejs user !

## Install pm2, to run your node app

## Install and configure mongo 
### Install
It really depend of the version on your server. These instructions are specific for Ubuntu 15.04




