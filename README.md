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

### Only authorize this user to connect ?
TODO

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
* Install necessary librairy
```bash
$ sudo apt-get install -y gcc g++ python clang make
```

* Go to node download page, and find the download link to the version you want. 
```bash
$ wget https://nodejs.org/dist/v4.2.4/node-v4.2.4.tar.gz
```
* Unzip the files and go to directory
```bash
$ tar -xzvf node-v4.2.4.tar.gz
$ cd ~/node-v4.2.4
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
* Add some symbolic for node modules
```bash
$ ln -s ~/.local/lib/node_modules ~/.node_modules
```
* Add node bin to your PATH
Add this to the .profile
```bash
export PATH=$HOME/.local/bin:$PATH
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
* Log as your node user
```bash
$ sudo su nodejs
```
* Install pm2
```bash
$ npm install -g nodejs
```

## Install and configure mongo 
### Install and start
It really depend of the version on your server. These instructions are specific for Ubuntu 15.04.

There is some compatibily problem with Ubuntu 15.04, here is a [link to a solution](https://jira.mongodb.org/browse/SERVER-17742), which consist
of (install the debian version)[https://docs.mongodb.org/manual/tutorial/install-mongodb-on-debian/].

```bash
$ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927
$ echo "deb http://repo.mongodb.org/apt/debian wheezy/mongodb-org/3.2 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list
$ sudo apt-get update
$ sudo apt-get install -y mongodb-org
$ sudo service mongod start
```

### Enable authentication on your database
* Create an admin user
```bash
$ mongo
>
use admin
db.createUser(
  {
    user: "myUserAdmin",
    pwd: "abc123",
    roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
  }
)
```
* Change the configuration for use authentication
```bash
$ sudo vi /etc/mongod.conf
```
Add these lines
```
security:
  authorization: enabled
```
* Restart mongo
```bash
$ sudo service mongod restart
```
* Check if it's ok.
```bash
$ mongo
> db.getUsers()
```
You will see an error.
```bash
$ mongo -u "myUserAdmin" --authenticationDatabase "admin" -p
> db.getUsers()
```
Everything ok

### Enable authentication on your database
* Create a database for your application
```bash
$ mongo -u "myUserAdmin" --authenticationDatabase "admin" -p
> use myApplicationDB
```
* Create a user with the good right (read, readWrite, or write)
```bash
>
db.createUser(
    {
      user: "myAppUser",
      pwd: "secretPWD",
      roles: [
         { role: "readWrite", db: "myApplicationDB" }
      ]
    }
)
```

## Install and configure nginx for proxy your node app
Nginx is a good solution to redirect all http port 
### Add a proxy to redirect 80 to your app
to do
### Https consideration
to do




