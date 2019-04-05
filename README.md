# Linux-Server-Configuration

Given a baseline installation of a Linux server, the project is to prepare it to host your web applications. 

This README explains  set up a Linux distribution on a virtual machine, install and configure a web and database server to host a web application.

Hosted app can be reached: 
http://ec2-54-188-52-163.us-west-2.compute.amazonaws.com/


## Amazon Light Sail

### Setup a new Ubuntu Linux server instance on Amazon Lightsail 

- Login into [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/resources) using an Amazon Web Services account.
- Once you are login into the site, click `Create instance`. 
- Choose `Ubuntu Server 18.04 LTS (HVM) - 64-bit x86` 
- Choose the instance with 1vCPU and 1 GB memory and keep the rest of the settings to default. 


## SSH Access
1. Download Private Key below
2. Move the private key file into the folder `~/.ssh`. 
3. Open your terminal and type in
	```chmod 600 ~/.ssh/aws_key.rsa```
4. Create an alias in `~/.alias` to avoid typing the long ssh login command. 
```
udacityaws=ssh  -i ~/Documents/udacity/aws_udacity.pem ubuntu@ec2-54-188-52-163.us-west-2.compute.amazonaws.com 
```
5. Source the alias file using the command: `source ~/.alias`
6. Login using: `udacityaws` command in the terminal.


## Create a new user named grader
1. `sudo adduser grader`
2. `vim /etc/sudoers`
3. `touch /etc/sudoers.d/grader`
4. `vim /etc/sudoers.d/grader`, type in `grader ALL=(ALL:ALL) ALL`, save and quit

## Set ssh login using keys
1. generate keys on local machine using`ssh-keygen` ; then save the private key in `~/.ssh` on local machine
2. deploy public key on developement enviroment

	On you virtual machine:
	```
	$ su - grader
	$ mkdir .ssh
	$ touch .ssh/authorized_keys
	$ vim .ssh/authorized_keys
	```
	Copy the public key generated on your local machine to this file and save
	```
	$ chmod 700 .ssh
	$ chmod 644 .ssh/authorized_keys
	```
	
3. reload SSH using `service ssh restart`
4. Create an alias in `~/.alias` to avoid typing the long ssh login command. 
```
graderaws=ssh  -i ~/Documents/udacity/grader_key.pem ubuntu@ec2-54-188-52-163.us-west-2.compute.amazonaws.com
```
5. Source the alias file using the command: `source ~/.alias`
6. Login using `graderaws`

## Update all currently installed packages
While logged in as `ubuntu`:
	sudo apt-get update
	sudo apt-get upgrade

## Change the SSH port from 22 to 2200
While logged in as `ubuntu`:
1. Use `sudo vim /etc/ssh/sshd_config` and then change Port 22 to Port 2200 , save & quit.
2. Reload SSH using `sudo service ssh restart`

## Configure the Uncomplicated Firewall (UFW)
While logged in as `ubuntu`:
- Configure the default firewall for Ubuntu to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
  ```
  sudo ufw status                  # The UFW should be inactive.
  sudo ufw default deny incoming   # Deny any incoming traffic.
  sudo ufw default allow outgoing  # Enable outgoing traffic.
  sudo ufw allow 2200/tcp          # Allow incoming tcp packets on port 2200.
  sudo ufw allow www               # Allow HTTP traffic in.
  sudo ufw allow 123/udp           # Allow incoming udp packets on port 123.
  sudo ufw deny 22                 # Deny tcp and udp packets on port 53.
  ```

- Turn UFW on: `sudo ufw enable`. The output should be like this:
  ```
  Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
  Firewall is active and enabled on system startup
  ```

- Check the status of UFW to list current roles: `sudo ufw status`. The output should be like this:

  ```
  Status: active
  
  To                         Action      From
  --                         ------      ----
  2200/tcp                   ALLOW       Anywhere                  
  80/tcp                     ALLOW       Anywhere                  
  123/udp                    ALLOW       Anywhere                  
  22                         DENY        Anywhere                  
  2200/tcp (v6)              ALLOW       Anywhere (v6)             
  80/tcp (v6)                ALLOW       Anywhere (v6)             
  123/udp (v6)               ALLOW       Anywhere (v6)             
  22 (v6)                    DENY        Anywhere (v6)
  ```

- Exit the SSH connection: `exit`.

## Attack Vector
While logged in as `grader`:
- Install Fail2Ban: `sudo apt-get install fail2ban`.
- Install sendmail for email notice: `sudo apt-get install sendmail iptables-persistent`.
- Create a copy of a file: `sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`.
- Change the settings in `/etc/fail2ban/jail.local` file:
```
[sshd]
enabled = true
port = 2200
filter = sshd
# the length of time between login attempts for maxretry. 
findtime = 600
# attempts from a single ip before a ban is imposed.
maxretry = 5
# the number of seconds that a host is banned for.
bantime = 3600
```
- Enable and fail2ban: 
```
sudo systemctl enable fail2ban
sudo systemctl status fail2ban
```
- Verify the status of sshd using 
```
sudo fail2ban-client status sshd
Status for the jail: sshd
|- Filter
|  |- Currently failed:	0
|  |- Total failed:	0
|  `- File list:	/var/log/auth.log
`- Actions
   |- Currently banned:	0
   |- Total banned:	0
   `- Banned IP list:	

```
** Reference:
https://www.tricksofthetrades.net/2018/05/18/fail2ban-installing-bionic/

## Enable Automatic Updates
- Install the unattended-upgrades package:
	`sudo apt install unattended-upgrades`
This package may already be installed on your server.

- Edit the config file: `sudo vi /etc/apt/apt.conf.d/50unattended-upgrades`
Uncomment the “updates” line:
`"${distro_id}:${distro_codename}-updates";`

- Enable automatic updates and set up update intervals by running:
`sudo vi /etc/apt/apt.conf.d/20auto-upgrades`
Copy and paste the following lines:

```

APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::AutocleanInterval "7";
APT::Periodic::Unattended-Upgrade "1";

```

- Do a test run: `sudo unattended-upgrades --dry-run --debug`

This will make sure the automatic updates run everyday.

## Configure the local timezone to UTC
While logged in as `grader`:
1. Configure the time zone `sudo dpkg-reconfigure tzdata`
2. It is already set to UTC.

## Install and configure Apache to serve a Python mod_wsgi application
While logged in as `grader`:
1. Install Apache `sudo apt-get install apache2`
2. Install mod_wsgi `sudo apt-get install python-setuptools libapache2-mod-wsgi`
3. Restart Apache `sudo service apache2 restart`

## Install and configure PostgreSQL
While logged in as `grader`:
- Install PostgreSQL:
 `sudo apt-get install postgresql`.
- PostgreSQL should not allow remote connections. In the  `/etc/postgresql/10/main/pg_hba.conf` file, you should see:
  ```
  local   all             postgres                                peer
  local   all             all                                     peer
  host    all             all             127.0.0.1/32            md5
  host    all             all             ::1/128                 md5
  ```

- Switch to the `postgres` user: `sudo su - postgres`.
- Open PostgreSQL interactive terminal with `psql`.
- Create the `catalog` user with a password and give them the ability to create databases:
  ```
  postgres=# CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';
  postgres=# ALTER ROLE catalog CREATEDB;
  ```

- List the existing roles: `\du`. The output should be like this:
  ```
                                     List of roles
   Role name |                         Attributes                         | Member of 
  -----------+------------------------------------------------------------+-----------
   catalog   | Create DB                                                  | {}
   postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
  ```

- Exit psql: `\q`.
- Switch back to the `grader` user: `exit`.
- Create a new Linux user called `catalog`: `sudo adduser catalog`. Enter password and fill out information.
- Give to `catalog` user the permission to sudo. Run: `sudo visudo`.
- Search for the lines that looks like this:
  ```
  root    ALL=(ALL:ALL) ALL
  grader  ALL=(ALL:ALL) ALL
  ```

- Below this line, add a new line to give sudo privileges to `catalog` user.
  ```
  root    ALL=(ALL:ALL) ALL
  grader  ALL=(ALL:ALL) ALL
  catalog  ALL=(ALL:ALL) ALL
  ```

- Save and exit using CTRL+X and confirm with Y.
- Verify that `catalog` has sudo permissions. Run `su - catalog`
- While logged in as `catalog`, create a database: `createdb catalog`.
- Run `psql` and then run `\l` to see that the new database has been created. The output should be like this:
  ```
                                    List of databases
     Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
  -----------+----------+----------+-------------+-------------+-----------------------
   catalog   | catalog  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
   postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
   template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
             |          |          |             |             | postgres=CTc/postgres
   template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
             |          |          |             |             | postgres=CTc/postgres
  (4 rows)
  ```
- Exit psql: `\q`.
- Switch back to the `grader` user: `exit`.

**Reference**
- DigitalOcean, [How To Secure PostgreSQL on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps).


## Install git, clone and setup your Catalog App project.
While logged in as `grader`:
1. Install Git using `sudo apt-get install git`
2. `cd /var/www` 
3. Clone the Grocery App `git clone git@github.com:vidhya0406/grocery_list.git`
4. Rename the project's name `sudo mv .grocery_list ./catalog`
5. Rename `server.py` to `catalog.py` 
6. Edit `db_setup.py`, `catalog.py` to use the postgres db. 
7. Create an empty `__init__.py` file.
8. Replace server port details in `catalog.py` from:
```
if __name__ == "__main__":
    app.run(host='0.0.0.0', port=env.get('PORT', 3000))
```
to
```
if __name__ == "__main__":
    app.run()
```
8. Install pip `sudo apt-get install python-pip`
9. Use pip to install dependencies `sudo pip install -r requirements.txt`
10. Install psycopg2 `sudo apt-get -qqy install postgresql python-psycopg2`

## Install the virtual environment and dependencies

- While logged in as `grader`, install pip: `sudo apt-get install python3-pip`.
- Install the virtual environment: `sudo apt-get install python-virtualenv`
- Change to the `/var/www/catalog/` directory.
- Create the virtual environment: `sudo virtualenv -p python3 venv3`.
- Change the ownership to `grader` with: `sudo chown -R grader:grader venv3/`.
- Install psycopg2 `sudo apt-get -qqy install postgresql python-psycopg2`
- Use pip to install dependencies `sudo pip install -r requirements.txt`


## Configure and Enable a New Virtual Host
1. Create catalog.conf to edit: `sudo nano /etc/apache2/sites-available/catalog.conf`
2. Add the following lines of code to the file to configure the virtual host. 
	
```
	<VirtualHost *:80>
  	ServerName 54.188.52.163
  	ServerAlias ec2-54-188-52-163.us-west-2.compute.amazonaws.com

	  # Give an alias to to start your website url with
	  WSGIScriptAlias / /var/www/catalog/catalog.wgsi
	     <Directory /home/username/ExampleFlask/ExampleFlask/>
			# set permissions as per apache2.conf file
		    Options FollowSymLinks
		    AllowOverride None
		    Require all granted
	     </Directory>
	     ErrorLog ${APACHE_LOG_DIR}/error.log
	     LogLevel warn
	     CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>
```
3. Enable the virtual host with the following command: `sudo a2ensite catalog`

## Create the .wsgi File
1. Create the .wsgi File under /var/www/catalog: 
	
	```
	cd /var/www/catalog
	sudo nano catalog.wsgi 
	```
2. Add the following lines of code to the catalog.wsgi file:
Values for AUTH0 environment variables can be found: https://jsonblob.com/5b08bac1-51e5-11e9-9fb6-3777fb6f71ab
	
```
	activate_this = '/var/www/catalog/venv3/bin/activate_this.py'
	with open(activate_this) as file_:
   	 exec(file_.read(), dict(__file__=activate_this))
	#!/usr/bin/python
	import sys
	import logging
	import os
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/catalog")
	os.environ['AUTH0_CLIENT_ID']="<CLIENT_ID>"
	os.environ['AUTH0_CLIENT_SECRET']="<CLIENT_SECRET>"
	os.environ['AUTH0_CALLBACK_URL']="<CALLBACK>"
	os.environ['AUTH0_DOMAIN']="<DOMAIN>"
	os.environ['AUTH0_AUDIENCE']="<AUDIENCE>"

	from catalog import app as application
	application.secret_key = 'secret'

```

## Restart Apache
- Restart Apache `sudo systemctl reload apache2 `

## References:
https://www.codementor.io/abhishake/minimal-apache-configuration-for-deploying-a-flask-app-ubuntu-18-04-phu50a7ft


