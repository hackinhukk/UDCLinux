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

11. Install git [source](https://www.digitalocean.com/community/tutorials/how-to-install-git-on-ubuntu-16-04)

  1. Type the command ```sudo apt-get install git```.

  2. Configure your username for accurate commit statements: ```git config --global user.name "Your User Name"```.

  3. Configure your email for accuate commit statements: ```git config --global user.email "youremail@domain.com"```

  **note**: to check that your configs are working, type in the command ```git config --list``` and it will show your user.name and user.email configurations.

12. Clone the Catalog application from Github

  1. Type in the command ```cd /var/www```.  Then create the catalog directory with the command ```sudo mkdir catalog```.

  2. Change owner for the *catalog* folder with the command ```sudo chown -R grader:grader catalog```.

  3. Move inside that newly created folder ```cd catalog``` and clone the catalog repository from Github with the command ```git clone https://github.com/hackinhukk/ItemCatalog.git catalog```

  4. Make a *catalog.wsgi* file to run the application over the *mod_wsgi* application with the command ```nano catalog.wsgi```.  The file contents should look like this:
  ```
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0,"/var/www/catalog/")

  from catalog import app as application
  application.secret_key = 'super_secret_key'
  ```

  5. Go inside the /var/www/catalog/catalog directory.  Rename *main.py* to **init.py** with the command ```mv main.py __init__.py```


13. Install Flask and other dependencies [source](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

  1. Go to the /var/www/catalog/ directory.  Install the virtualenv package with the following command ```sudo pip install virtualenv```.

  2. Go to the *catalog* directory with ```cd /var/www/catalog```.  Then create a new virtual environment by typing in ```sudo virtualenv venv```.

  3. Activate the virtual environment with the command ```source venv/bin/activate```.

  4. Change the virtual environment folder permissions with the command ```sudo chmod -R 777 venv```.

  5. Install Flask with the following command: ```pip install Flask```.

  6. Install all the other project's dependencies ```pip install flask sqlalchemy requests httplib2 oauth2client psycopg2-binary```.

14. Update path of client_secrets.json file

  1. Type in the command ```nano __init__.py```

  2. Change client_secrets.json path to ```/var/www/catalog/catalog/client_secrets.json```.

15. Setup and enable a new virtual host

  1. Create a virtual host config file with the command ```sudo nano /etc/apache2/sites-available/catalog.conf```.

  2. Paste in the following lines of code:
```
<VirtualHost *:80>
    ServerName 172.26.9.101
    ServerAlias 18.221.52.69
    ServerAdmin admin@172.26.9.101
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

  3. Disable the default virtual host with the command ```sudo a2dissite 000-default.conf```.

  4. Enable the catalog app virtual host by typing in ```sudo a2ensite catalog.conf```.

  5. To make these Apache2 changes live, we need to reload Apache with the command ```sudo service apache2 reload```.


16. Install and configure PostgreSQL [source](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04).

  1. Install some necessary Python packages for interacting with PostgreSQL with the command ```sudo apt-get install libpq-dev python-dev```.

  2. Install PostgreSQL with the command ```sudo apt-get install postgresql postgresql-contrib```.

  3. During the Postgres installation, a user named *postgres* was created to correspond to the *postgres* PostgreSQL administrative user.  You need to change to this user in order to perform any admin tasks.  The ```sudo su - postgres``` command changes you to the *postgres* user.

  4. Login to a Postgres session with the command ```psql```.

  5. Create a new user called 'catalog' with its password: ```CREATE USER catalog WITH PASSWORD 'catalogpasswordname';```.

  6. Give the *catalog* user the CREATEDB ability with the command: ```ALTER USER catalog CREATEDB;```.

  7. Create the catalog database owned by the *catalog* user: ```CREATE DATABASE catalog WITH OWNER catalog;```.

  8. Connect to the DB with the command ```\c catalog```.

  9. Revoke all rights with the command ```REVOKE ALL ON SCHEMA public FROM public;```.

  10. Limit the permissions so only the *catalog* user can create tables with the command ```GRANT ALL ON SCHEMA public TO catalog;```.

  11. Log out from PostgreSQL with the command ```\q```.  Then type ```exit``` to return to the *grader* user.
(TNT STOPPED HERE then went to git and onwards)
  12.  In your __init__.py and database_setup.py files change the create engine line to ```engine = create_engine('postgresql://catalog:catalogpasswordname@localhost/catalog')```

  13. Setup the database with the command ```python /var/www/catalog/catalog/database_setup.py```.  Then follow it with the command ```python /var/www/catalog/catalog/catalogitems.py```.

  14. To block remote connections to the database, we have to open the follow file and edit it with the command ```sudo nano /etc/postgresql/9.5/main/pg_hba.conf``` and edit it, if necessary, to make it look like:

  ```
  local    all                 postgres                                peer
  local    all                 all                                     peer
  host     all                 all               127.0.0.1/32          md5
  host     all                 all               ::1/128               md5
  ```

  15. Type in the command ```sudo service apache2 restart```.  Your web server should be viewable at the address *18.221.52.69*
