# Linux Server Configuration

Baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database server

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
2. Click on `Download default key`
3. A file called LightsailDefaultPrivateKey-eu-central-1.pem will be downloaded; open this in a text editor
4. Move the private key file into the folder ~/.ssh. If there is no directory, make one `mkdir .ssh` in home directory
5. Create a file named lightrail_key 'touch ~/lightrail.key.rsa
6. Copy the text from LightsailDefaultPrivateKey-eu-central-1.pem and put it in lightrail_key.rsa 
7. Move the file to `~/.ssh/ directory`
8. Run `chmod 600 ~/.ssh/lightrail_key.rsa`
9. Log in with the following command: `ssh -i ~/.ssh/lightrail_key.rsa ubuntu@18.194.164.182 -p 2200
`, after @ is the public IP address of the instance (note that Lightsail will not allow someone to log in as root; ubuntu is the default user for Lightsail instances)

#### Upgrade currently installed packages

After log in to the server from the terminal in your computer,
1. Notify the system of what package updates are available by running sudo apt-get update
2. Download available package updates by running sudo apt-get upgrade

#### Configure the firewall

1. Start by changing the SSH port from 22 to 2200 (open up the `sudo vim/etc/ssh/sshd_config file`, change the port number on line 5 to 2200, then restart SSH by running sudo service ssh restart)
2. Check to see if the ufw (the preinstalled ubuntu firewall) is active by running sudo ufw status - it is inactive at this point
3. Run sudo ufw default deny incoming to set the ufw firewall to block everything coming in
4. Run sudo ufw default allow outgoing to set the ufw firewall to allow everything outgoing
5. Run sudo ufw allow ssh to set the ufw firewall to allow SSH
6. Run sudo ufw allow 2200/tcp to allow all tcp connections for port 2200 so that SSH will work
7. Run sudo ufw allow www to set the ufw firewall to allow a basic HTTP server
8. Run sudo ufw allow 123/udp to set the ufw firewall to allow NTP
9. Run sudo ufw deny 22 to deny port 22 (deny this port since it is not being used for anything; it is the default port for SSH, but this virtual machine has now been configured so that SSH uses port 2200)
10. Run sudo ufw enable to enable the ufw firewall
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

#### Create a new user named grader

1. Run `sudo adduser grader`
2. Enter in a new UNIX password (twice) when prompted
3. Fill out information for the new grader user. ( name: Grader)
4. To switch to the grader user, run `su - grader`, and enter the password

#### Give grader user sudo permissions

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

1. Run `ssh-keygen` on the local machine (not in vagrant)
2. Choose a file name for the key pair (such as grader_key)
3. Enter in a passphrase twice (two files will be generated; the second one will end in .pub)
4. Log in to the virtual machine (I logged in into the server - ubuntu@ip)
 - made grader's directory under home directory (mkdir grader), has ubuntu as another directory
- Switch to grader's home directory, and create a new directory called .ssh (run `mkdir .ssh`)
6. under home/grader directory:
 - Run `chmod 700 ~/.ssh`
 - run `touch ~/.ssh/authorized_keys`
 - run `chmod 600 ~/.ssh/authorized_keys`
 
7. On the local machine terminal (not in vagrant), run `cat ~/.ssh/insert-name-of-file.pub`

8. Copy the contents of the file, and paste them in the .ssh/authorized_keys file on the virtual machine - has not succesful yet

...
