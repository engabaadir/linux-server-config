
# Project: Linux Server Configuration
This is a fifth project for Udacity's 
[Full Stack Web Developer Nanodegree](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004)
## Project Overview
This project provides baseline installation of a Linux distribution on a virtual machine and preparing it to host web applications, this installation includes installing updates and configuring web and database servers.
1. The Linux distribution is Ubuntu 16.04.6 LTS.
1. The virtual private server is Amazon Lighsail.
1. The web application is my Restaurant Catalog project submitted earlier in this Nanodegree program.
1. The database server is PostgreSQL.
## Server Access Details
- Server IP Address: 54.86.80.183
- SSH Port: 2200
- SSH User Name: grader
- Application URL: http://somnewsonline.com/
## Configuration Steps:
Below are steps to configure web and database server
### 1. Set up SSH key
- Create lightsail virtual server instance
- Download the AWS provided default private key
- Change the private key file permission `chmod 600 LightsailDefaultKey-eu-central-1.pem`
- generate keys on local machine using `ssh-keygen` and then save the private key in *~/.ssh* on local machine
- Login ssh `ssh ubuntu@54.86.80.183 -p 22 -i C:/Users/PC/.ssh/LightsailDefaultKey-eu-central-1.pem`
### 2. Update and upgrade installed packages
   ```
   sudo apt-get update
   sudo apt-get upgrade  
   ```
### 3. Changing the SSH Port from 22 to 2200
Open the `/etc/ssh/sshd_config`:
	
   ```
   nano /etc/ssh/sshd_config
   Port 22 change it to Port 2200
   service ssh restart
   ssh root@54.86.80.183 -p 2200
   ```
### 4. Configure the Uncomplicated Firewall (UFW)
   ```
   sudo ufw allow 2200/tcp
   sudo ufw allow www
   sudo ufw allow 123/udp
   sudo ufw deny 22
   sudo ufw default deny incoming
   sudo ufw default allow outgoing
   sudo ufw enable
   sudo ufw status
   ```
### 5. Configure the local timezone to UTC
   ```
   sudo dpkg-reconfigure tzdata
   ```
### 6. Create user grader
   ```
   sudo adduser grader
   vim /etc/sudoers
   touch /etc/sudoers.d/grader
   ```
`vim /etc/sudoers.d/grader`, type in `grader ALL=(ALL:ALL) ALL,` and save it
### 7. Create an SSH key pair for grader
   ```
   su - grader
   mkdir .ssh
   touch .ssh/authorized_keys
   nano .ssh/authorized_keys

   Copy the public key content from local machine to this file and save, change the access level
   chmod 700 .ssh
   chmod 644 .ssh/authorized_keys
   ```
### 8. Install and configure PostgreSQL
- Postgresql Package Installation
  `sudo apt-get install postgresql`
- Switching to postgres user
	```
    sudo su - postgres
    psql
    \q
    exit
    ```
- Create a new user named restaurant and new database named restaurant
    ```
    sudo su - postgres
    CREATE USER restaurant WITH PASSWORD '4321';
    \du
    CREATE DATABASE restaurant;
    \l
    ```
### 9. Installing Apache Web Server
   ```
   sudo apt install apache2
   ```
### 10. Cloning the project application

1. CD into the below directory

   ```
   cd /var/www/
   ```
1. Clone your GitHub repository

	```
   	sudo git clone https://github.com/engabaadir/restaurant-catalog.git 
	```

### 11. Setting Up the VirtualHost Configuration

1. Create and set up a file called `restaurant_catalog.conf` to configure the virtual hosts:

   ```
   $ sudo nano /etc/apache2/sites-available/restaurant_catalog.conf
   ```

1. Past it with below lines
   ```
      <VirtualHost *:80>
                  ServerName 54.86.80.183
                  ServerAlias http://somnewsonline.com
                  ServerAdmin engabaadir@gmail.com
                  WSGIScriptAlias / /var/www/Catalog/restaurant_catalog.wsgi
                  <Directory /var/www/Catalog/>
                           Order allow,deny
                           Allow from all
                  </Directory>
                  Alias /static /var/www/Catalog/static
                  <Directory /var/www/Catalog/static/>
                           Order allow,deny
                           Allow from all
                  </Directory>
                  ErrorLog ${APACHE_LOG_DIR}/error.log
                  LogLevel warn
                  CustomLog ${APACHE_LOG_DIR}/access.log combined
      </VirtualHost>
   ```
1. Enable the virtual host:

   ```
   $ sudo a2ensite /etc/apache2/sites-available/restaurant_catalog.conf
   ```

1. Restart Apache server:

   ```
   $ sudo service apache2 restart
   ```
### 12. Create the .wsgi File

1. Cd into the `/var/www/Catalog/` directory and create a file named `restaurant_catalog.wsgi` like below:

   ```
   $ cd /var/www/Catalog/
   $ sudo nano restaurant_catalog
   ```
1. Past it with below  lines   
   ```
   #!/usr/bin/python3
   import sys
   import logging
   logging.basicConfig(stream=sys.stderr)
   sys.path.insert(0,"/var/www/Catalog")

   from views import app as application
   application.secret_key = 'super_secret_key'

   ```
1. Restart the Apache

	```
	sudo service apache2 restart
	```
### 13. Setup the required dependencies
   ```
   sudo apt-get install python-pip
   sudo pip install virtualenv
   sudo virtualenv venv
   source venv/bin/activate
   sudo pip install Flask
   pip install httplib2
   sudo apt-get install python3-oauth2client
   sudo apt-get install python3-requests
   sudo apt-get install python-requests
   sudo apt-get install  python3-sqlalchemy
   sudo apt-get python3-psycopg2
   ```
### 14. References:
- https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-uswgi-and-nginx-on-ubuntu-18-04
- https://www.postgresql.org/docs/current/static/sql-createuser.html
- https://www.codementor.io/abhishake/minimal-apache-configuration-for-deploying-a-flask-app-ubuntu-18-04-phu50a7ft


