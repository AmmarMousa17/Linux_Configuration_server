
#Linux Congiguration Server

# Server details
IP address:`18.217.217.205`
URL: `ec2-18-217-217-205.us-east-2.compute.amazonaws.com`

# Configuration changes

## Add user Grader by root
`useradd -m -s /bin/bash grader`

## Add user grader to sudo group
- Create a file namded grader `sudo nano /etc/sudoers.d/grader`
  - Append `grader ALL=(ALL:ALL) ALL`
```
usermod -aG sudo grader
```

## Update packages

`apt-get update`

`apt-get upgrade` 



## Configure key-based authentication for grader user
```
mkdir /home/grader/.ssh
chown grader:grader /home/grader/.ssh
chmod 700 /home/grader/.ssh
cp /root/.ssh/authorized_keys /home/grader/.ssh/
chown grader:grader /home/grader/.ssh/authorized_keys
chmod 644 /home/grader/.ssh/authorized_keys
```

## Disable root login
- Change the following line in the file `/etc/ssh/sshd_config`:

- From `PermitRootLogin without-password` to `PermitRootLogin no`.


 - Restart ssh with `sudo service ssh restart`



## Change SSH port from 22 to 2200
Edit the file `/etc/ssh/sshd_config` and change the line `Port 22` to:

`Port 2200`



## Configuration (UFW)
 - `sudo ufw allow 2200/tcp`
  - `sudo ufw allow 80/tcp`
  - `sudo ufw allow 123/udp`
  - `sudo ufw enable`
  
## Change timezone to UTC

`sudo dpkg-reconfigure tzdata` and choose UTC

## Install Apache and Python mod_wsgi application
Install Apache:

`sudo apt-get install apache2`

Install the `libapache2-mod-wsgi` package:

`sudo apt-get install libapache2-mod-wsgi`

##Clone project 3

 - Install git using: `sudo apt-get install git`
  - `cd /var/www`
  - `sudo mkdir FSND`
  - `cd /FSND`
  - Clone your project from github `git clone https://github.com/AmmarMousa17/item-catalog.git`
  -copy contents of FSND into new directory named catalog
  - Create a catalog.wsgi file, then add this inside:
  ```
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/catalog/catalog")
  
  from catalog import app as application
  application.secret_key = 'supersecretkey'
  ```
 

## Install and configure PostgreSQL
-Install PostgreSQL with:

`sudo apt-get install postgresql postgresql-contrib`

-Create a PostgreSQL user called `catalog` with:

`sudo -u postgres createuser -P catalog`

-input uour password

-and run this command to create database

`sudo -u postgres createdb -O catalog catalog`
- Open the database setup file:  
  `sudo nano database_setup.py`
4. Change the line starting with "engine" to (fill in a password):  
  ```python engine = create_engine('postgresql://catalog:password@localhost/catalog')``` 

  ## (important) To avoid make 1 week in that error like me
-chande directory of client_secret to absolute path in projet.py
``` /var/www/catalog/catalog ```


## Install import libraries
Issue the following commands:
```
sudo apt-get install python-psycopg2 python-flask
sudo apt-get install python-sqlalchemy python-pip
sudo pip install oauth2client
sudo pip install requests
sudo pip install httplib2

```

## Configure and enable a new virtual host
-Create a virtual host config file in /etc/apache2/sites-available-chandr 
-paste this content into this file 

- Paste this code: 
  ```
  <VirtualHost *:80>
    ServerName 18.217.217.205
    ServerAdmin ammar_mousa17@hotmail.com
    ServerAlias HOSTNAME ec2-18-217-217-205.us-east-2.compute.amazonaws.com
    WSGIScriptReloading On
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi 
   <Directory /var/www/catalog/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel info
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
  ```
  
 -Enable the virtual host:
 ``` $ sudo a2ensite  catalog.conf```
-Restart Apache
``` sudo /etc/init.d/apache2 restart ```


## Google OAuth Login
 - Visit: https://console.developers.google.com/project
  - thrn go credentials
  -Add ``` ec2-18-217-217-205.us-east-2.compute.amazonaws.com ```` To Authorized redirect URIs
  
## for tailing the apache logs so I can see errors in real time:
 -https://www.liquidweb.com/kb/how-to-watch-server-logs-in-real-time
 
 
 ##Helpful Links :
 -https://discussions.udacity.com/c/nd004-p7-linux-based-server-configuration
 -https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
 -https://discussions.udacity.com/t/500-internal-server-error-and-mod-wsgi-warnings-in-error-logs/489043
 
