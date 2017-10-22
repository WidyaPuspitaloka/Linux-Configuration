# Linux Server Configuration

Baseline installation of a Linux distribution on a virtual machine and prepare it to host web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database server

- Public IP: 18.194.51.171 
- SSH port: 2200 
- URL: http://18.194.51.171/

# Configuration Steps

## Create an instance with Amazon Lightsail

1. Sign in to Amazon Lightsail using an Amazon Web Services account
2. Follow the Create an instance link
3. Choose the OS Only and Ubuntu 16.04 LTS options
4. Choose a payment plan - the lowest one should be enough
5. Name the instance (wap-lightsail) and click 'Create'
6. Wait for the instance to start up
7. Connect to the instance on a local machine

Note: While Amazon Lightsail provides a broswer-based connection method, this will no longer work once the SSH port is changed (see below). The following steps outline how to connect to the instance via the Terminal program on Mac OS machines.

8. Download the instance's private key from Amazon Lightsail 'Account page'

9. Click on SSH keys -> Download default key

10. A file called LightsailDefaultPrivateKey-eu-central-1.pem will be downloaded; open this in a text editor

11. Create a file named lightrail_key touch ~/lightrail_key.rsa

12. Copy the text from LightsailDefaultPrivateKey-eu-central-1.pem and put it in lightrail_key.rsa

13. Move the file to ~/.ssh/ directory using mv ~/lightrail_key.rsa ~/.ssh

14. Run chmod 600 ~/.ssh/lightrail_key.rsa

15. Log in with the following command: ssh -i ~/.ssh/lightrail_key.rsa ubuntu@18.194.228.221 , after @ is the public IP address of the instance (note that Lightsail will not allow someone to log in as root; ubuntu is the default user for Lightsail instances)

## Create a new user named grader

in server ubuntu directory

1. Run `sudo adduser grader`
2. Enter in a new UNIX password (twice) when prompted
3. Fill out information for the new grader user. ( name: grader)
4. To switch to the grader user, run su - grader, and enter the password

## Give grader user sudo permissions

In ubuntu directory

1. Run sudo visudo

2. Search for a line that looks like this: `root ALL=(ALL:ALL) ALL`

3. Add the following line below this one: `grader ALL=(ALL:ALL) ALL`

4. Save and close the visudo file

5. To verify that grader has sudo permissions, su as grader (run su - grader), enter the password, and run sudo -l; after entering in the password (again), a line like the following should appear, meaning grader has sudo permissions:

```
Matching Defaults entries for grader on
    ip-XX-XX-XX-XX.ec2.internal:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User grader may run the following commands on
	ip-XX-XX-XX-XX.ec2.internal:
    (ALL : ALL) ALL'
Allow grader to log in to the virtual machine
```

## Run ssh-keygen on the local terminal machine

1. Choose a file name for the key pair (such as grader_key)

2. Enter in a passphrase twice (two files will be generated; the second one will end in .pub) -mv grader_key.pub ~/.ssh -mv grader_key ~/.ssh

3. Log in to the virtual machine (logged in into the server - ubuntu@ip)
```
su - grader
mkdir .ssh
touch .ssh/authorized_keys
```

4. On the local machine, run cat ~/.ssh/grader_key.pub, make sure the text match in local machine and authorized_keys, otherwise you can not log in as grader.

5. Copy the contents of the file, and paste them in the `.ssh/authorized_keys` file on the virtual machine

```
vim .ssh/authorized_keys
copy paste
chmod 700 .ssh
chmod 644 .ssh/authorized_keys
sudo service ssh restart
```

6. Exit grader to ubuntu user

7. `chmod 600 ~/.ssh/grader_key` in local machine

8. Log in as the grader using the following command:

`ssh -i ~/.ssh/grader_key grader@18.194.51.171`

Note that a pop-up window will ask for grader's password.

## Configure SSH Daemon

Now that we have our new account, we can secure our server a little bit by modifying its SSH daemon configuration (the program that allows us to log in remotely) to disallow remote SSH access to the root account.

1. Begin by opening the configuration file with your text editor as root:

`sudo nano /etc/ssh/sshd_config` Next, we need to find the line that looks like this:
/etc/ssh/sshd_config (before) PermitRootLogin yes Here, we have the option to disable root login through SSH. This is generally a more secure setting since we can now access our server through our normal user account and escalate privileges when necessary.

2. Modify this line to "no" like this to disable root login:
```
/etc/ssh/sshd_config (after) 
PermitRootLogin no
```

## Upgrade currently installed packages

1. After log in to the server (ubuntu@ip) from the terminal in your computer,

2. Notify the system of what package updates are available by running sudo apt-get update

3. Download available package updates by running `sudo apt-get upgrade`

4. Install package maintainer

5. Changing the SSH port from 22 to 2200 (open up the sudo vim/etc/ssh/sshd_config file, change the port number on line 5 to 2200, then restart SSH by running sudo service ssh restart)

## Configure the Uncomplicated Firewall (UFW)

1. Check to see if the ufw (the preinstalled ubuntu firewall) is active by running sudo ufw status - it is inactive at this point

2. Run sudo ufw allow 2200/tcp to allow all tcp connections for port 2200 so that SSH will work

3. Run sudo ufw allow 80/tcp to set the ufw firewall to allow a basic HTTP server

4. Run sudo ufw allow 123/udp to set the ufw firewall to allow NTP

5. Run sudo ufw enable to enable the ufw firewall

6. Run sudo ufw status to check which ports are open and to see if the ufw is active; if done correctly, it should look like this:

```
To                         Action      From
--                         ------      ----

2200/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
123/udp                    ALLOW       Anywhere

2200/tcp (v6)              ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
123/udp (v6)               ALLOW       Anywhere (v6)

```

7. Update the external (Amazon Lightsail) firewall on the browser by clicking on the 'Manage' option, then the 'Networking' tab, and then changing the firewall configuration to match the internal firewall settings above (only ports 80(TCP), 123(UDP), and 2200(TCP) should be allowed; make sure to deny the default port 22)

## Configure the local timezone to UTC

1. Configure the time zone `sudo dpkg-reconfigure tzdata`

2. It is already set to UTC.

## Install and configure Apache to serve a Python mod_wsgi application

1. Install Apache `sudo apt-get install apache2` Check to make sure it worked by using the public IP of the Amazon Lightsail instance as as a URL in a browser; if Apache is working correctly, a page with the title 'Apache2 Ubuntu Default Page' should load

2. Mod_wsgi is an Apache HTTP server mod that enables Apache to serve Flask applications. Install mod_wsgi with the following command: `Install mod_wsgi sudo apt-get install libapache2-mod-wsgi python-dev `

3. Enable mod_wsgi: `sudo a2enmod wsgi`

```

perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
	LANGUAGE = (unset),
	LC_ALL = (unset),
	LC_CTYPE = "UTF-8",
	LANG = "en_US.UTF-8"
    are supported and installed on your system.
perl: warning: Falling back to a fallback locale ("en_US.UTF-8").
Module wsgi already enabled
```

- run this to fix error:
```
$ sudo locale-gen "en_US.UTF-8"
Generating locales...
  en_US.UTF-8... done
Generation complete.

$ sudo dpkg-reconfigure locales
Generating locales...
  en_US.UTF-8... up-to-date
Generation complete.
```

- and run this:
```
export LC_ALL=en_US.UTF-8
export LANGUAGE=en_US.UTF-8
export LC_TYPE=en_US.UTF-8
```

-- result:
```
LANG=en_US.UTF-8
LANGUAGE=en_US.UTF-8
LC_CTYPE="en_US.UTF-8"
LC_NUMERIC="en_US.UTF-8"
LC_TIME="en_US.UTF-8"
LC_COLLATE="en_US.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_PAPER="en_US.UTF-8"
LC_NAME="en_US.UTF-8"
LC_ADDRESS="en_US.UTF-8"
LC_TELEPHONE="en_US.UTF-8"
LC_MEASUREMENT="en_US.UTF-8"
LC_IDENTIFICATION="en_US.UTF-8"
LC_ALL=en_US.UTF-8
```

4. Restart Apache `sudo service apache2 restart`

## Install and configure PostgreSQL

1. Install PostgreSQL `sudo apt-get install postgresql`

2. Check if no remote connections are allowed `sudo vim /etc/postgresql/9.5/main/pg_hba.conf` Make sure it looks like this:

```
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```

## Make sure Python is installed

Python should already be installed on a machine running Ubuntu 16.04. To verify, simply run `python`. Something like the following should appear:

```
Python 2.7.12 (default, Nov 19 2016, 06:48:10) 
[GCC 5.4.0 20160609] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> 
```

## Create a new PostgreSQL user named catalog with limited permissions


1. PostgreSQL creates a Linux user with the name postgres during installation; switch to this user by running `sudo su - postgres` (for security reasons, it is important to only use the postgres user for accessing the PostgreSQL software)

2. Login as user "postgres" `sudo su - postgres`

3. Connect to psql (the terminal for interacting with PostgreSQL) by running `psql`

4. Create the catalog user by running `CREATE USER catalog WITH PASSWORD 'sillypassword'`;

5. Next, give the catalog user the ability to create databases: `ALTER USER catalog CREATEDB`;

6. Create the 'catalog' database owned by catalog user: `CREATE DATABASE catalog WITH OWNER catalog`;.

7. Connect to the database: # \c catalog

8. Check to make sure the catalog user was created by running \du; a table of sorts will be returned, and it should look like this:

```
				   List of roles
 Role name |                         Attributes                         | Member of 
-----------+------------------------------------------------------------+-----------
 catalog   | Create DB                                                  | {}
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 ```
 
9. Exit psql by running \q

10. Switch back to the ubuntu user by running exit

## Install git and clone the catalog project

1. Run `sudo apt-get install git`

2. Use `cd /var/www` to move to the /var/www directory

3. Create the application directory `sudo mkdir CatalogApp`

4. Cd inside this directory using `cd CatalogApp`

5. Clone the Catalog App to the virtual machine `sudo git clone https://github.com/WidyaPuspitaloka/Catalog.git CatalogApp`

6. Move to the inner CatalogApp directory using cd CatalogApp

7. Rename application.py to init.py using `sudo mv finalproject.py __init__.py`

8. Edit 
- database_setup.py
- init.py 
- coldplaydiscography.py

and change engine = create_engine('sqlite:///toyshop.db') to engine = create_engine('postgresql://catalog:sillypassword@localhost/catalog')

## Set up a vitual environment and install dependencies

Install Flask

Setting up a virtual environment will keep the application and its dependencies isolated from the main system. Changes to it will not affect the cloud server's system configurations.

In this step, we will create a virtual environment for our flask application.

We will use pip to install virtualenv and Flask. If pip is not installed, install it on Ubuntu through apt-get.

1. Start by installing pip (if it isn't installed already) with the following command:
`sudo apt-get install python-pip`

2. Install virtualenv with apt-get by running `sudo apt-get install python-virtualenv`

3. Change to the /var/www/CatalogApp/CatalogApp directory; choose a name for a temporary environment ('venv' is used in this example), and create this environment by running virtualenv venv (make sure to not use sudo here as it can cause problems later on)

`sudo virtualenv venv`

4. Activate the virtual environment: `$ source venv/bin/activate`.

5. Change permissions to the virtual environment folder: `$ sudo chmod -R 777 venv`.

6. Install:

```
pip install Flask
pip install httplib2
pip install --upgrade oauth2client
pip install sqlalchemy
pip install psycopg2
pip install requests
```

7. In order to make sure everything was installed correctly, run `python __init__.py`; the following (among other things) should be returned:
```
Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```

8. Create database schema python database_setup.py

## Configure and enable a new virtual host

1. Create a virtual host conifg file: `sudo nano /etc/apache2/sites-available/CatalogApp.conf`

2. Paste in the following lines of code:

```
<VirtualHost *:80>
        ServerName mywebsite.com
        ServerAdmin widyapuspitaloka11@yahoo.com
        WSGIDaemonProcess CatalogApp user=www-data group=www-data threads=5
        WSGIScriptAlias / /var/www/CatalogApp/catalogapp.wsgi
        <Directory /var/www/CatalogApp/CatalogApp/>
                WSGIProcessGroup CatalogApp
                WSGIApplicationGroup %{GLOBAL}
                Order allow,deny
                Allow from all
        </Directory>
        Alias /static /var/www/CatalogApp/CatalogApp/static
        <Directory /var/www/CatalogApp/CatalogApp/static/>
                Order allow,deny
                Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

3. Run `sudo a2ensite CatalogApp` and get this message
```
Enabling site CatalogApp.
To activate the new configuration, you need to run:
  service apache2 reload
 ```
 
4. Then `sudo service apache2 reload`

## Disable the default Apache site

1. At some point during the configuration, the default Apache site will likely need to be disabled; to do this, run sudo `a2dissite 000-default.conf`

2. The following prompt will be returned:
```
Site 000-default disabled.
To activate the new configuration, you need to run:
  service apache2 reload
  ```
  
3. Run `sudo service apache2 reload`

## Write a .wsgi file

1. Apache serves Flask applications by using a .wsgi file; create a file called catalogapp.wsgi in /var/www/CatalogApp

2. Add the following to the file:

```
#!/usr/bin/python
import os
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/CatalogApp/")

activate_this = '/var/www/CatalogApp/CatalogApp/venv/bin/activate_this.py'
execfile(activate_this, dict(__file__=activate_this))

from CatalogApp import app as application
application.secret_key = 'super_secret_key'
```

3. Run `sudo service apache2 restart`

4. Check if there's error using `sudo tail /var/log/apache2/error.log`

## Add client_secrets.json files

1. On the Google API Console: make sure to add http://18.194.51.171
 and http://ec2-18.194.51.171
.compute-1.amazonaws.com as authorized JavaScript origins

2. Add http://ec2-18.194.51.171
.compute-1.amazonaws.com/login, http://ec2-18.194.51.171
.compute-1.amazonaws.com/gconnect, and http://ec2-18.194.51.171
.compute-1.amazonaws.com/oauth2callback as authorized redirect URIs

4. Google will provide a client ID and client secret for the project; download the JSON file, and copy and paste the contents into the client_secrets.json file

5. Add the client ID to templates/login.html file in the project directory

6. Add the complete file path for the client_secrets.json file in the __init__.py file; change it from client_secrets.json to /var/www/CatalogApp/CatalogApp/client_secrets.json

## Update packages to the latest versions

1. Sudo apt install unattended-upgrades

2. Sudo apt autoremove

3. Sudo apt-get update && sudo apt-get upgrade

## Resources

1. [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
2. [How To Serve Flask Applications with uWSGI and Nginx on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-uwsgi-and-nginx-on-ubuntu-16-04)
3. [Initial Server Setup with Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04)
4.[Starting web projects using Flask, virtualenv, pip](http://modwsgi.readthedocs.io/en/develop/user-guides/virtual-environments.html)
5.[Connecting to Your Linux Instance Using SSH](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)
6. [How do I fix my locale issue?](https://askubuntu.com/questions/162391/how-do-i-fix-my-locale-issue)
7. 1-1 Appointment with Udacity's Mentor

## License

MIT License
