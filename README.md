# Linux  Server Configuration
In this project I will take a baseline installation of a Linux server and prepare it to host my web applications. I will secure your server from a number of attack vectors, install and configure a database server, and deploy one of my existing web applications onto it.

## Server details

Server Info

Public IP: 13.232.87.13

Port: 2200


## Getting Started
This project uses Amazon Lightsail to create a Linux server instance.

Get your server.

Start a new Ubuntu Linux server instance on Amazon Lightsail.

1. Log in!
2. Create an instance
3. Choose an instance image: Ubuntu (OS only)
4. Choose your instance plan (lowest tier is fine)
5. Give your instance a hostname
6. Wait for startup
7. Once the instance has started up, follow the instructions provided to SSH into your server.

## Start vagrant

Steps:

```
$ vagrant init ubuntu/trusty64
$ vagrant up
$ vagrant ssh
```

## SSH into your Server

Amazon Lightsail enforces usage of an SSH key to connect to the instance. Download the SSH key from your account to connect to the server with thee default user ubuntu.

1. Download private key from the SSH keys section in the Account sectionn on Amazon Lightsail. The file name should be like LightsailDefaultPrivateKey-us-east-2.pem
2. Create a new file named lightsail_key.rsa under ~/.ssh folder on your local machine.
3. Copy and paste content from downloaded private key file to lightsail_key.rsa
4. Set file permission as owner only : $ chmod 600 ~/.ssh/lightsail_key.rsa
5. SSH into the instance: $ ssh -i ~/.ssh/lightsail_key.rsa ubuntu@13.232.87.13

## Configuration changes

## Update all currently installed packages
This command will update your package source list.

```
sudo apt-get update
```

This command will actually update the software.

```
sudo apt-get upgrade
```

## Secure your server

Change the SSH port from 22 to 2200

1. Open the config file:
```
 $ sudo nano /etc/ssh/sshd_config
```

then change the line of Port 22 to: Port 2200
save the file and restart
```
$ sudo service ssh restart

```

## Configure The Firewall
Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

1.block all incoming connections on all ports:
```
$ sudo ufw default deny incoming
```

2.Allow outgoing connection on all ports:
```
$ sudo ufw default allow outgoing
```

3 .Allow incoming TCP packets on port 2200 to allow SSH: 
```
$ sudo ufw allow 2200/tcp
```

4.Allow incoming connections for HTTP on port 80:
```
$ sudo ufw allow www
```

5.Allow incoming UDP packets on port 123 to allow NTP:
```
$  sudo ufw allow 123/udp
```

6.Close port 22: 
```
sudo ufw deny 22
```

7. To check the rules being added
```
$sudo ufw show added
```

8.Enable firewall: 
```
$ sudo ufw enable
```

9.To check the status of the firewall, use:
```
$ sudo ufw status
```

10.Update the firewall configuration on Amazon Lightsail website under Networking. Delete default SSH port 22 and add port 80, 123, 2200.

11.Then restart the SSH service
```
$ sudo service ssh restart
```

12.you can now ssh in via the new port 2200:
```
$ ssh -i ~/.ssh/lightsail_key.rsa ubuntu@13.232.87.13 -p 2200
```

## Create a new user named grader
1. Create a new user account named grader.
This done by this command:
```
 $ sudo adduser grader
```
The password was set to grader

2. Give  grader the permission to sudo.

first create the grader file
```
$ sudo touch /etc/sudoers.d/grader
```

The  password was set to grader
open to edit 
```
$ sudo nano /etc/sudoers.d/grader
```
put this line

```
grader ALL=(ALL) NOPASSWD:ALL
```

3. Create an SSH key pair for grader using the ssh-keygen tool.
1.Generate an encryption key on your local machine
```
$ ssh-keygen -t rsa
```

2.Place the public key on the server
. On your local machine, read the generated public key cat ~/.ssh/id_rsa.pub
. On your virtual machine,save the public key in /home/grader/authorized_keys

```
$ nano /home/grader/authorized_keys
```

.Runon your virtual machine to change file permission

```
 $ chmod 700 .ssh 
 $ chmod 644 .ssh/authorized_keys
```

. Restart SSH:

``` $ sudo service ssh restart
```


.login to grader

```
$ ssh -i ~/.ssh/id_rsa -p 2200 grader@13.232.87.13
```

## Prepare to deploy your project.

9. Configure the local timezone to UTC.
```
sudo dpkg-reconfigure tzdata
```
select none of the above, then UTC

10.Install and configure Apache to serve a Python mod_wsgi application.

1.Install and Configure Apache2, mod-wsgi and python

Install Apache:
``` 
$ sudo apt-get install apache2
```

```
 $sudo apt-get install libapache2-mod-wsgi python-dev 
```

Enable mod_wsgi:
```
 $ sudo a2enmod wsgi
```


Restart Apache: 
```
$ sudo service apache2 restart
```

. Install and configure PostgreSQL:

```
$ sudo apt-get install postgresql
```

Check if no remote connections are allowed :
```
sudo cat /etc/postgresql/9.5/main/pg_hba.conf
```

This how the  file look:
```
#
# Database administrative login by Unix domain socket
local   all             postgres                                peer

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
# Allow replication connections from localhost, by a user with the
# replication privilege.
#local   replication     postgres                                peer
#host    replication     postgres        127.0.0.1/32            md5
#host    replication     postgres        ::1/128                 md5
```

Create a new database user named catalog :

Login as postgres User (Default User):
```
sudo su - postgres
```

Connect to PostgreSQL:
```
 $ psql
```

Create user catalog with LOGIN role: 
```
# CREATE ROLE catalog WITH PASSWORD 'password';
```

Allow user to create database tables:
``` 
# ALTER USER catalog CREATEDB;
```
Create database:
```
 # CREATE DATABASE catalog WITH OWNER catalog;
```

Connect to database catalog:
``` 
# \c catalog
```
Revoke all the rights: 
```
# REVOKE ALL ON SCHEMA public FROM public;
```
Grant access to catalog:
```
 # GRANT ALL ON SCHEMA public TO catalog;
```
Exit psql: 
```
\q 
```
Exit user postgres: 
```
exit
```

Create new Linux user called catalog and new database
Create a new Linux user: 
```
$ sudo adduser catalog
```
Give catalog user sudo access:
```
$ sudo visudo
```
Add
``` 
$ catalog ALL=(ALL:ALL) ALL
```
under line $ root ALL=(ALL:ALL) ALL
Save and exit the file

12. Install git.
```
$sudo apt-get install git
```

## Deploy the Item Catalog project.

13. Clone and setup your Item Catalog project from the Github repository you created earlier in this Nanodegree program.
.Make a catalog named directory in /var/www
```
  $ sudo mkdir /var/www/catalog
```
.Change the owner of the directory catalog
```
$ sudo chown -R ubuntu:ubuntu catalog/
```
.clone the git repository
```
$ sudo git clone https://github.com/nadiaalshezawy/Built-Catalog-Item-Application.git catalog
```
.CD to /var/www/catalog/catalog
Change file application.py to init.py: 
```
$ mv project.py __init__.py
```
Change line app.run(host='0.0.0.0', port=8000) to app.run() in init.py file

## Install packages
```
$ sudo apt-get install python-psycopg2 python-flask
$ sudo apt-get install python-sqlalchemy python-pip
$ sudo pip install --upgrade pip (not necessary)
$ sudo pip install oauth2client
$ sudo pip install requests
$ sudo pip install httplib2
```
14. Setup and enble a virtual host
.Make .git file inaccessable

.add line ```RedirectMatch 404 /\.git```

Create file:
```
 $ sudo touch /etc/apache2/sites-available/catalog.conf
```

add these lines to the file
```
<VirtualHost *:80>
        ServerName 13.232.87.13
        ServerAdmin nadiaalshezawy@gmail.com
        WSGIScriptAlias / /var/www/catalog/catalog/catalog.wsgi
        <Directory /var/www/catalog/catalog/>
            Order allow,deny
            Allow from all
            Options -Indexes
        </Directory>
        Alias /static /var/www/catalog/catalog/static
        <Directory /var/www/catalog/catalog/static/>
            Order allow,deny
            Allow from all
            Options -Indexes
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
```

Enable the virtual host just created
```
$ sudo a2ensite catalog.conf
```

To make these changes live restart Apache2
```
$ sudo service apache2 restart
```
Create file:
```
 $ sudo touch /var/www/catalog/catalog/catalog.wsgi
```

write this to the file
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/catalog/")

from catalog import app as application

application.secret_key = 'super_secret_key'
```

Restart Apache: 
```
$ sudo service apache2 reload
```

.Edit the database path
Use sudo nano on each of the files( __init__.py, database_setup.py, and lotsofitems.py. Find the line 
```
 engine = create_engine('sqlite:///categoriesitem.db')
```
and replace it with the following: 
```
engine = create_engine('postgresql://catalog:password@localhost/catalog')
```
.Disable defualt Apache page
```
$ sudo a2dissite 000-defualt.conf
```
Restart Apache: 
```
$ sudo service apache2 reload
```

. create the catalog.wsgi file and add these line to the content:

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application

application.secret_key = 'super_secret_key'

```


. Create database by navigating to the folder (/var/www/catalog/catalog) and run these two command
```
 $ python database_setup.py
 $ python lotsofmenus.py
```

Note : when uploading the database , the description column in catalog was truncated the character even it was working perfectly on local host, to solve this issue had to resize String(250) to String(5000) charcater .

.create a new project on Google API Console and download client_scretes.json file
Copy and paste contents of downloaded client_secrets.json to the file with same name under directory /var/www/catalog/catalog/client_secrets.json and
update the path in the __init__.py to (/var/www/catalog/catalog/client_secrets.json).




## Author
 Nadia Ahmed
