# Item Application On Linux Web Server

## Description
The goals from the Udacity program state that:

You will take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

The application referred to is our item catalog app, which we created [here](https://github.com/hackinhukk/ItemCatalogUDC/tree/master/app)


## Step by step walkthrough

1.  Log into the remote VM as root user through ssh: ```ssh root@18.223.125.137```

2. Create a new user named (italics) grader: ```sudo adduser grader```

  1. Create a new file in the sudoers directory with ```sudo nano /etc/sudoers.d/grader```

  2. In the new file, add the text ```grader ALL=(ALL:ALL) ALL``` and then save the file (```control + O then press enter``` to save in nano, ```control + X``` to exit nano).

  3. Run the command ```sudo nano /etc/hosts```

  4. In order to prevent the error message ```sudo: unable to resolve host``` by adding the host```127.0.1.1 ip-10-20-52-12```

3. Update all currently installed packages

  1. ```sudo apt-get update```

  2. ```sudo apt-get upgrade```

  3. Install *finger*, by running ```apt-get install finger```.

4. Change SSH Port from 22 to 2200.  

  1. Run the command ```sudo nano /etc/ssh/sshd_config```.  Find the *Port* line and edit it to **2200**.

  2. Run ```sudo service ssh restart```.

  3. Now the way you can remotely login into the VM through ssh is by typing the command ```ssh -i ~/.ssh/LightsailDefaultPrivateKey-us-east-2.pem -p 2200 ubuntu@18.223.125.137```.  

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
