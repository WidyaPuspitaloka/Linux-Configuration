# Linux Server Configuration

Baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database server

Public IP: 18.194.228.221

### Configuration Steps

#### Create an instance with Amazon Lightsail

1. Sign in to Amazon Lightsail using an Amazon Web Services account
2. Follow the `Create an instance` link
3. Choose the `OS Only` and `Ubuntu 16.04 LTS` options
4. Choose a payment plan - the lowest one should be enough
5. Name the instance (wap-lightsail) and click 'Create'
6. Wait for the instance to start up

#### Connect to the instance on a local machine

Note: While Amazon Lightsail provides a broswer-based connection method, this will no longer work once the SSH port is changed (see below). The following steps outline how to connect to the instance via the Terminal program on Mac OS machines.

1. Download the instance's private key from Amazon Lightsail 'Account page'
2. Click on SSH keys -> `Download default key`
3. A file called LightsailDefaultPrivateKey-eu-central-1.pem will be downloaded; open this in a text editor

5. Create a file named lightrail_key `touch ~/lightrail_key.rsa`
6. Copy the text from LightsailDefaultPrivateKey-eu-central-1.pem and put it in lightrail_key.rsa 
7. Move the file to `~/.ssh/ directory` using `mv ~/lightrail_key.rsa ~/.ssh`
8. Run `chmod 600 ~/.ssh/lightrail_key.rsa`
9. Log in with the following command: `ssh -i ~/.ssh/lightrail_key.rsa ubuntu@18.194.228.221`
`, after @ is the public IP address of the instance (note that Lightsail will not allow someone to log in as root; ubuntu is the default user for Lightsail instances)`

#### Create a new user named grader

in server ubuntu directory
1. Run `sudo adduser grader`
2. Enter in a new UNIX password (twice) when prompted
3. Fill out information for the new grader user. ( name: grader)
4. To switch to the grader user, run `su - grader`, and enter the password


#### Give grader user sudo permissions
in ubuntu directory

1. Run `sudo visudo`
2. Search for a line that looks like this:
`root ALL=(ALL:ALL) ALL`

3. Add the following line below this one:
`grader ALL=(ALL:ALL) ALL`

4. Save and close the visudo file
5. To verify that grader has sudo permissions, su as grader (run `su - grader`), enter the password, and run sudo -l; after entering in the password (again), a line like the following should appear, meaning grader has sudo permissions:
```
Matching Defaults entries for grader on
    ip-XX-XX-XX-XX.ec2.internal:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User grader may run the following commands on
	ip-XX-XX-XX-XX.ec2.internal:
    (ALL : ALL) ALL'
 ```
 
 
    
#### Allow grader to log in to the virtual machine

1. Run `ssh-keygen` on the local terminal machine
2. Choose a file name for the key pair (such as grader_key)
3. Enter in a passphrase twice (two files will be generated; the second one will end in .pub)
-mv grader_key.pub ~/.ssh
-mv grader_key ~/.ssh
4. Log in to the virtual machine (logged in into the server - ubuntu@ip)
 - su - grader
 - mkdir .ssh
 - touch .ssh/authorized_keys
 
 5. On the local machine, run cat ~/.ssh/grader_key.pub:ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC4OnXLOWFQ32VXuGufCrBFlt3Z+afuE1aU0NcJYMeBgi8F3wpvMa59egMgB6FP57ff0fDcWPARylxYaf4FaU3qRFfbxw+8esIvKh+5dIBXtwoKlu/1YI5H6X8LX6skVCZaRjC/cFQwFNdmv1G9sAHHwtW75i4X8aJH3UGvP+EzBfH4u4W4ImaHUA2Vtkdxjqe/zR2MP6wNOl1s6j5XaJWuhYanpeLHqtIvEGdHSXnaRQvdqmNfsjc5PwYwX5qsiEbdj22QP1UpXfTeAYwCXopi6yqxY9iySGD7vLxY6mOSkpv048khDn1QYiqiTzS2I+/uahYPU66o7JV584/dDccd widyapuspitaloka@Widyas-MacBook-Air.local

make sure the text match in local machine and authorized_keys, otherwise you can not log in as grader.

6. Copy the contents of the file, and paste them in the .ssh/authorized_keys file on the virtual machine 
- vim .ssh/authorized_keys
- chmod 700 .ssh
- chmod 644 .ssh/authorized_keys
-Make sure key-based authentication is forced (log in as grader, open the /etc/ssh/sshd_config file, and find the line that says, '# Change to no to disable tunnelled clear text passwords'; if the next line says, 'PasswordAuthentication yes', change the 'yes' to 'no' - already no

7. sudo service ssh restart
exit grader to ubuntu user

8. chmod 600 ~/.ssh/grader_key in local machine

9. Log in as the grader using the following command:

ssh -i ~/.ssh/grader_key grader@35.157.35.166

Note that a pop-up window will ask for grader's password.
 
#### Upgrade currently installed packages

After log in to the server (ubuntu@ip) from the terminal in your computer,
1. Notify the system of what package updates are available by running `sudo apt-get update`
2. Download available package updates by running `sudo apt-get upgrade`
- keep the local file

#### Change the SSH port from 22 to 2200

After log in to the server (ubuntu@ip) from the terminal in your computer,
1. Start by changing the SSH port from 22 to 2200 (open up the `sudo vim/etc/ssh/sshd_config file`, change the port number on line 5 to 2200, then restart SSH by running sudo service ssh restart)

### Configure the Uncomplicated Firewall (UFW)
2. Check to see if the ufw (the preinstalled ubuntu firewall) is active by running sudo ufw status - it is inactive at this point

6. Run sudo ufw allow 2200/tcp to allow all tcp connections for port 2200 so that SSH will work
7. Run sudo ufw allow 80/tcp to set the ufw firewall to allow a basic HTTP server
8. Run sudo ufw allow 123/udp to set the ufw firewall to allow NTP

not 9. Run sudo ufw deny 22 to deny port 22 (deny this port since it is not being used for anything; it is the default port for SSH, but this virtual machine has now been configured so that SSH uses port 2200)

10. Run `sudo ufw enable` to enable the ufw firewall
11. Run sudo ufw status to check which ports are open and to see if the ufw is active; if done correctly, it should look like this:
```
To                         Action      From
--                         ------      ----
22                         DENY        Anywhere
2200/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
123/udp                    ALLOW       Anywhere
22 (v6)                    DENY        Anywhere (v6)
2200/tcp (v6)              ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
123/udp (v6)               ALLOW       Anywhere (v6)
```

12. Update the external (Amazon Lightsail) firewall on the browser by clicking on the 'Manage' option, then the 'Networking' tab, and then changing the firewall configuration to match the internal firewall settings above (only ports 80(TCP), 123(UDP), and 2200(TCP) should be allowed; make sure to deny the default port 22)


### Configure the local timezone to UTC

1. Configure the time zone `sudo dpkg-reconfigure tzdata`
2. It is already set to UTC.

### Install and configure Apache to serve a Python mod_wsgi application

1. Install Apache `sudo apt-get install apache2`
Check to make sure it worked by using the public IP of the Amazon Lightsail instance as as a URL in a browser; if Apache is working correctly, a page with the title 'Apache2 Ubuntu Default Page' should load

2. Mod_wsgi is an Apache HTTP server mod that enables Apache to serve Flask applications. Install mod_wsgi with the following command:
Install mod_wsgi `sudo apt-get install python-setuptools libapache2-mod-wsgi`
Enable mod_wsgi: `sudo a2enmod wsgi`

'''
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
	LANGUAGE = (unset),
	LC_ALL = (unset),
	LC_CTYPE = "UTF-8",
	LANG = "en_US.UTF-8"
    are supported and installed on your system.
perl: warning: Falling back to a fallback locale ("en_US.UTF-8").
Module wsgi already enabled
'''

3. Restart Apache `sudo service apache2 restart`

### Install and configure PostgreSQL
1. Install PostgreSQL `sudo apt-get install postgresql`

2. Check if no remote connections are allowed sudo vim /etc/postgresql/9.3/main/pg_hba.conf
Make sure it looks like this (comments have been removed here for easier reading):

'''
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
'''



### Make sure Python is installed

Python should already be installed on a machine running Ubuntu 16.04. To verify, simply run `python`. Something like the following should appear:

'''
Python 2.7.12 (default, Nov 19 2016, 06:48:10) 
[GCC 5.4.0 20160609] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> 
'''

### Create a new PostgreSQL user named catalog with limited permissions

PostgreSQL creates a Linux user with the name postgres during installation; switch to this user by running sudo su - postgres (for security reasons, it is important to only use the postgres user for accessing the PostgreSQL software)

1. Login as user "postgres" `sudo su - postgres`
2. Connect to psql (the terminal for interacting with PostgreSQL) by running `psql`

Create the catalog user by running CREATE ROLE catalog WITH LOGIN;

Next, give the catalog user the ability to create databases: ALTER ROLE catalog CREATEDB;

Finally, give the catalog user a password by running \password catalog

Check to make sure the catalog user was created by running \du; a table of sorts will be returned, and it should look like this:

				   List of roles
 Role name |                         Attributes                         | Member of 
-----------+------------------------------------------------------------+-----------
 catalog   | Create DB                                                  | {}
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
Exit psql by running \q

Switch back to the ubuntu user by running `exit`

### Install git and clone the catalog project

1. Run `sudo apt-get install git`

2. Use cd /var/www to move to the /var/www directory
3. Create the application directory sudo mkdir CatalogApp
4. Move inside this directory using cd FlaskApp
Clone the Catalog App to the virtual machine `sudo git clone https://github.com/WidyaPuspitaloka/Catalog.git`
5. rename the project name : sudo mv ./Catalog ./CatalogApp
6. Move to the inner CatalogApp directory using `cd CatalogApp`
7. rename application.py to __init__.py using sudo mv finalproject.py __init__.py
8. Edit database_setup.py, finalproject.py and change engine = create_engine('sqlite:///toyshop.db') to engine = create_engine('postgresql://catalog:password@localhost/catalog')


### Set up a vitual environment and install dependencies

1. Start by installing pip (if it isn't installed already) with the following command:

sudo apt-get install python-pip

2. Install virtualenv with apt-get by running `sudo apt-get install python-virtualenv`

3. Change to the /var/www/catalogApp/catalogApp/ directory; choose a name for a temporary environment ('venv' is used in this example), and create this environment by running virtualenv venv (make sure to not use sudo here as it can cause problems later on)

sudo apt install virtualenv
sudo virtualenv venv
Running virtualenv with interpreter /usr/bin/python2
New python executable in /var/www/CatalogApp/CatalogApp/venv/bin/python2
Also creating executable in /var/www/CatalogApp/CatalogApp/venv/bin/python
Installing setuptools, pkg_resources, pip, wheel...
  Complete output from command /var/www/CatalogApp/...App/venv/bin/python2 - setuptools pkg_resources pip wheel:
  Traceback (most recent call last):
  File "<stdin>", line 24, in <module>
  File "/usr/share/python-wheels/pip-8.1.1-py2.py3-none-any.whl/pip/__init__.py", line 215, in main

  File "/var/www/CatalogApp/CatalogApp/venv/lib/python2.7/locale.py", line 581, in setlocale
    return _setlocale(category, locale)
locale.Error: unsupported locale setting
----------------------------------------
...Installing setuptools, pkg_resources, pip, wheel...done.
Traceback (most recent call last):
  File "/usr/lib/python3/dist-packages/virtualenv.py", line 2363, in <module>
    main()
  File "/usr/lib/python3/dist-packages/virtualenv.py", line 719, in main
    symlink=options.symlink)
  File "/usr/lib/python3/dist-packages/virtualenv.py", line 988, in create_environment
    download=download,
  File "/usr/lib/python3/dist-packages/virtualenv.py", line 918, in install_wheel
    call_subprocess(cmd, show_stdout=False, extra_env=env, stdin=SCRIPT)
  File "/usr/lib/python3/dist-packages/virtualenv.py", line 812, in call_subprocess
    % (cmd_desc, proc.returncode))
OSError: Command /var/www/CatalogApp/...App/venv/bin/python2 - setuptools pkg_resources pip wheel failed with error code 1
	
	sudo apt-get update
	sudo apt-get install python-pip python-dev nginx
