# Linux-Server-Configuration

Given a baseline installation of a Linux server, the project is to prepare it to host your web applications. 

This README explains  set up a Linux distribution on a virtual machine, install and configure a web and database server to host a web application.



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
1. Install PostgreSQL `sudo apt-get install postgresql`
2. Check if no remote connections are allowed `sudo vim /etc/postgresql/10/main/pg_hba.conf`
3. Login as user "postgres" `sudo su - postgres`
4. Get into postgreSQL shell `psql`
5. Create a new database named catalog  and create a new user named catalog in postgreSQL shell
	
	```
	postgres=# CREATE DATABASE catalog;
	postgres=# CREATE USER catalog;
	```
5. Set a password for user catalog
	
	```
	postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
	```
6. Give user "catalog" permission to "catalog" application database
	
	```
	postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
	```
7. Quit postgreSQL `postgres=# \q`
8. Exit from user "postgres" 
	
	```
	exit
	```
 
## Install git, clone and setup your Catalog App project.
While logged in as `grader`:
1. Install Git using `sudo apt-get install git`
2. `cd /var/www` 
3. Clone the Grocery App `git clone git@github.com:vidhya0406/grocery_list.git`
4. Rename the project's name `sudo mv .grocery_list ./catalog`
5. Rename `server.py` to `__init__.py` 
6. Edit `db_setup.py`, `__init__.py` to use the postgres db. 
8. Install pip `sudo apt-get install python-pip`
9. Use pip to install dependencies `sudo pip install -r requirements.txt`
10. Install psycopg2 `sudo apt-get -qqy install postgresql python-psycopg2`

## Install the virtual environment and dependencies

- While logged in as `grader`, install pip: `sudo apt-get install python3-pip`.
- Install the virtual environment: `sudo apt-get install python-virtualenv`
- Change to the `/var/www/catalog/` directory.
- Create the virtual environment: `sudo virtualenv -p python3 venv3`.
- Change the ownership to `grader` with: `sudo chown -R grader:grader venv3/`.
- Activate the new environment: `. venv3/bin/activate`.
- Install psycopg2 `sudo apt-get -qqy install postgresql python-psycopg2`
- Use pip to install dependencies `sudo pip install -r requirements.txt`

- Run `python3 __init__.py` and you should see:
  ```
  * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
  ```

- Deactivate the virtual environment: `deactivate`.

**References**
- Flask documentation, [virtualenv](http://flask.pocoo.org/docs/0.12/installation/).
- [Create a Python 3 virtual environment](https://superuser.com/questions/1039369/create-a-python-3-virtual-environment).

## Configure and Enable a New Virtual Host
1. Create catalog.conf to edit: `sudo nano /etc/apache2/sites-available/catalog.conf`
2. Add the following lines of code to the file to configure the virtual host. 
	
	```
	<VirtualHost *:80>
		ServerName 54.188.52.163
		ServerAdmin vidhya0406@gmail.com
		WSGIScriptAlias / /var/www/catalog/catalog.wsgi
		<Directory /var/www/catalog/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/catlog/static
		<Directory /var/www/catalog/static/>
			Order allow,deny
			Allow from all
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
	
	```
	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/catalog/")

	from FlaskApp import app as application
	application.secret_key = 'Add your secret key'
	```

## Restart Apache
1. Restart Apache `sudo service apache2 restart `

## References:
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
