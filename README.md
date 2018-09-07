# Item Application On Linux Web Server

## Description
The goals from the Udacity program state that:

You will take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

The application referred to is our item catalog app, which we created [here](https://github.com/hackinhukk/ItemCatalogUDC/tree/master/app)


## Step by step walkthrough

1.  Log into the remote VM as root user through ssh

  1. First download the private key for the specific Lightsail instance from the SSH Keys section in the Account profile of Amazon Lightsail.

  2. Move the private key into the folder **~/.ssh** If you downloaded the file and saved it to the Downloads file, run the command ```mv ~/Downloads/Lightsail-key.pem ~/.ssh/``` **note: Lightsail-key is not meant to be literal, it is meant to represent the file name you downloaded from the Amazon Lightsail site.**

  3. Type in ```chmod 400 ~/.ssh/Lightsail-key.pem``` to the terminal.

  4. Run the command ```ssh -i ~/.ssh/Lightsail-key.pem ubuntu@18.191.210.24```.


2. Update all currently installed packages

  1. ```sudo apt-get update```

  2. ```sudo apt-get upgrade```

  3. Install *finger*, by running ```sudo apt-get install finger```.

3. Create a new user named *grader*: ```sudo adduser grader```

  1. Create a new file in the sudoers directory with ```sudo nano /etc/sudoers.d/grader```
    2. In the new file, add the text ```grader ALL=(ALL:ALL) ALL``` and then save the file (```control + O then press enter``` to save in nano, ```control + X``` to exit nano).

  3. Run the command ```sudo nano /etc/hosts```

  4. In order to prevent the error message ```sudo: unable to resolve host``` by adding the host```127.0.1.1 ip-10-20-52-12```


4. Change SSH Port from 22 to 2200.  

  1. Run the command ```sudo nano /etc/ssh/sshd_config```.  Find the *Port* line and edit it to **2200**.

  2. Run ```sudo service ssh restart```.

  3. Now the way you can remotely login into the VM through ssh is by typing the command ```ssh -i ~/.ssh/LightsailDefaultPrivateKey-us-east-2.pem -p 2200 ubuntu@18.191.210.24```.  

  **Note:  You have to add and save port 2200** with *Application* as *Custom* and *Protocol* as *TCP*.  This is found within the Networking section of the specific Amazon Lightsail instance.

5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123). - Source https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-14-04

  1. Set the defaults for UFW, first by typing in ```sudo ufw default deny incoming``` to block all incoming connections on all ports.

  2. Also set the UFW default to allow outgoing connections on all ports with the command ```sudo ufw default allow outgoing```.

  3.  Now add exceptions to the default, by allowing an incoming ssh connections on port 2200.  ```sudo ufw allow 2200/tcp```.

  4. Allow incoming HTTP connections on port 80.  ```sudo ufw allow 80/tcp```

  5. Allow the NTP connections on port 123. ```sudo ufw allow 123/udp```

  6. To check the rules that have been added before enabling the firewall, type in ```sudo ufw show added```.

  7.  If everything looks good, enable the firewall using the command ```sudo ufw enable```.

  8. To check the status of the firewall, use the command ```sudo ufw status```.

6. Configure key-based authentication for *grader* user

  1.  Generate keys on **your local machine** with the command ```ssh-keygen -f ~/.ssh/udacity_grader_key.rsa```

  2. On your VM, Type ```sudo mkdir /home/grader/.ssh```

  3. Type ```sudo touch home/grader/.ssh/authorized_keys```

  4. Type ```sudo nano home/grader/.ssh/authorized_keys```.  Copy the public key (**.pub extension**) generated on your local machine to this file and save (with control + O to save, control + X to exit).

  5. Change the file permissions of the ssh directory and the authorized_keys file.  Type in ```sudo chmod 700 /home/grader/.ssh``` to make the right permission changes for the directory.  Then type in ```sudo chmod 644 /home/grader/.ssh/authorized_keys``` to make the right permission changes for the file.

  6. Change the ownership of this ssh directory and the authorized_keys file to the *grader* user.  ```sudo chown grader:grader /home/grader/.ssh``` for the directory and ```sudo chown grader:grader .ssh/authorized_keys``` for the file.

  7. Type in ```sudo service ssh restart``` to reload SSH

  8.  You can now login as the *grader* user using the command: ```ssh -i ~/.ssh/udacity_grader_key.rsa -p 2200 grader@18.223.125.137```

7. Disable ssh login for root user and enforce key based authentication.

  1. Type in ```sudo nano /etc/ssh/sshd_config``` and change the **PermitRootLogin prohibit-password** to **PermitRootLogin no**.

  2. Make sure the following line exists and is uncommented in the same file ```PasswordAuthentication no```.

  3. Type in ```sudo service ssh restart``` for the changes to occur.

8. Configure Cron Scripts to automatically update packages with stable updates [source](https://help.ubuntu.com/community/AutomaticSecurityUpdates)

  1. Install the package ```sudo apt-get install unattended-upgrades``` if not already installed.

  2. To enable it, type ```sudo dpkg-reconfigure --priority=low unattended-upgrades```.

9. Configure the local timezone to UTC. [source](https://help.ubuntu.com/community/UbuntuTime)

  1. Type the command ```sudo dpkg-reconfigure tzdata``` and choose the UTC option in the terminal menu.

  2. Use the ntp daemon for a more accurate synchronization over a network connection by typing the command ```sudo apt-get install ntp```.

10. Install Apache to serve a Python mod_wsgi application. [source](https://www.bogotobogo.com/python/Flask/Python_Flask_HelloWorld_App_with_Apache_WSGI_Ubuntu14.php)

  1. Install Apache with the command ```sudo apt-get install apache2```. [source](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-16-04)

  2. Install *mod_wsgi* with the following command ```sudo apt-get install libapache2-mod-wsgi```.

  3. Enable *mod_wsgi* ```sudo a2enmod wsgi```.

11. Install and configure PostgreSQL [source](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04).

  1. Install some neccessary Python packages for interacting with PostgreSQL with the command ```sudo apt-get install libpq-dev python-dev```.

  2. Install PostgreSQL with the command ```sudo apt-get install postgresql postgresql-contrib```.

  3. During the Postgres installation, a user named *postgres* was created to correspond to the *postgres* PostgreSQL administrative user.  You need to change to this user in order to perform any admin tasks.  The ```sudo su - postgres``` command changes you to the *postgres* user.

  4. Login to a Postgres session with the command ```psql```.

  5. Create a new user called 'catalog' with its password: ```CREATE USER catalog WITH PASSWORD 'catalogpasswordname';```.

  6. Give the *catalog* user the CREATEDB ability with the command: ```ALTER USER catalog CREATEDB;```.

  7. Create the catalog database owned by the *catalog* user: ```CREATE DATABASE catalog WITH OWNER catalog;```.

  8.
