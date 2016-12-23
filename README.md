# Project: Item Catalog
###### Udacity Full Stack ND
#### About The Project
---
This project was created in part of Udacity's Full Stack Nanodegree curriculum.

This repository contains this sole README.md file outlining a baseline installment of a Linux server which will be used to host [Project 6](https://github.com/darvizu061/project_6_item_catalog) of my Udacity projects.

Public IP: 35.166.58.109
SSH PORT: 2200
Full Project URL: http://35.166.58.109/

Last Updated: Dec 21st, 2016
#### Walkthrough
___
Below is my step-by-step solution to problems 1-11 as outlined in Udacity's Project Details page.
##### 1 & 2 - Launch your Virtual Machine with your Udacity account & SSH into your server.
For this step, please follow Udacity's instructions to setup your free Amazon AWS environment. They will provide you with a unique IP address and private key which you will use to SSH into your server
##### 3 - Create a new user named grader
Checkpoint: At this point, you should already be logged into your provided server as root administrator.

To create user named grader:

`$ adduser grader`

optional - install finger to check user has been added :

`$ apt-get install finger`

`$ finger grader`


##### 4 - Give  grader the permission to sudo

Create file for grader in sudoers.d directory:

`$ sudo touch /etc/sudoers.d/grader`

Edit file to give sudo permission to grader:

`$ sudo nano /etc/sudoers.d/grader`

Add the following text to the file:

`grader ALL=(ALL) ALL`

Save file and exit nano editor.
###### Log In As Grader User  
Checkpoint: At this point, if you want to try to login to your server via SSH as grader you must do the following:

Generate key pairs on your LOCAL machine & give passphrase. This command will generate 2 files. One of these two files will have extension .pub which will be your public key.

`$ ssh-keygen`

Place public key on your REMOTE server. Run following command as root user.

`$ cp /.ssh/authorized_keys /home/grader/.ssh/authorized_keys`

Edit the /home/grader/.ssh/authorized_keys file, remove the current key in that file, and copy and paste your .pub key that you just generated.

`$ sudo nano /home/grader/.ssh/authorized_keys`

IMPORTANT - when we copied the authorized_keys from the .ssh directory, we also copied the same file permissions. Now we must make sure that the owner of the authorized_keys in the grader directory is in fact grader and NOT root.

`$ chown grader /home/grader/.ssh/authorized_keys`

Now you can log into your environment as grader from your local machine. Make sure that when you attempt login that you are using your newly generated .rsa key file and NOT the udacity_key.rsa key that Udacity provided.  
##### 5 - Update all currently installed packages
Get all installed packages

`$ sudo apt-get update`

Install new updates

`$ sudo apt-get upgrade` note: you may be prompted to hit y for yes or n for no on some package updates.
##### 6 - Change the SSH port from 22 to 2200 and disable root login
Edit ssh config file & change port 22 to 2200

`$ sudo nano /etc/ssh/sshd_config`

While editing the file also change `PermitRootLogin without-password` to `PermitRootLogin no` to disable root login with private key.

Add `AllowUsers grader` in that file to allow grader to log in via SSH.

Restart ssh service

`$ sudo service ssh reload`

note: when logging back into the Amazon environment as root or grader, you will have to add `-p2200` to your terminal command. For example:

`ssh -i ~/.ssh/udacity_key.rsa root@22.222.22.22 -p2200`
##### 7 - Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
Check UFW status to make sure it's inactive

`$ sudo ufw status`

Deny all incoming server requests in all ports

`$ sudo ufw default deny incoming`

Allow all outgoing server requests in all ports

`$ sudo ufw default allow outgoing`

Allow SSH on port 2200, HTTP on port 80 and NTP on port 123

`$ sudo ufw allow 2200/tcp`

`$ sudo ufw allow 80/tcp`

`$ sudo ufw allow 123/udp`

Enable ufw firewall

`$ sudo ufw enable`
##### 8 - Configure the local timezone to UTC
Run
`$ sudo dpkg-reconfigure tzdata`

When prompt, select None of the Above. Then select UTC.

##### 9 - Install and configure Apache to serve a Python mod_wsgi application

Install apache2 package

`sudo apt-get install apache2`

Check your IP address to see if it's working. You should see the default Apache index.html page.

Install mod_wsgi package. This package is for serving Python apps from Apache

`sudo apt-get install python-setuptools libapache2-mod-wsgi`

Restart Apache server

`$ sudo service apache2 restart`

##### 11 - Part 1 of 3 - Install git, clone and setup your Catalog App project (from your GitHub repository from earlier in the Nanodegree program) so that it functions correctly when visiting your server’s IP address in a browser.
Install git

`$ sudo apt-get install git`

Configure your name and email for commits

`$ git config --global user.name "YOUR NAME"`

`$ git config --global user.email "YOUR EMAIL ADDRESS"`

Extend Python packages to enable Apache to serve Flask apps.

`$ sudo apt-get install libapache2-mod-wsgi python-dev `

Enable mod_wsgi

`$ sudo a2enmod wsgi`

Create a flask application

`$ cd /var/www`

`$ sudo mkdir catalog`

`$ cd catalog`

`$ sudo mkdir catalog`

`$ cd catalog`

Create the application file

`$ sudo nano __init__.py`

The following code is a test app that will later be replaced.  Paste the code into `__init__.py`

```
from flask import Flask  
app = Flask(__name__)  
@app.route("/")  
def hello():  
  return "Veni vidi vici!!"  
if __name__ == "__main__":  
app.run()
```

Install pip installer

`$ sudo apt-get install python-pip`

Install virtualenv

`$ sudo pip install virtualenv`

Set virtual environment to venv

`$ sudo virtualenv venv`

Enable all permissions for the new virtual environment

`$ sudo chmod -R 777 venv`

Activate the virtual environment

`$ source venv/bin/activate`

Inside the virtual environment, install Flask

`$ pip install Flask`

Run test app

`$ python __init__.py`

Deactivate environment

`$ deactivate`

Configure And Enable New Virtual Host by creating virtual host config file

`$ sudo nano /etc/apache2/sites-available/catalog.conf`

Paste the code below and replace values with your server's values.
```
<VirtualHost *:80>
    ServerName PUBLIC-IP-ADDRESS
    ServerAdmin admin@PUBLIC-IP-ADDRESS
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
Enable virtual host

`$ sudo a2ensite catalog`

Create wsgi file

`$ cd /var/www/catalog`

`$ sudo vim catalog.wsgi`

Paste the code below into catalog.wsgi.

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'Add your secret key'
```
Restart Apache

`$ sudo service apache2 restart`

Clone your item project that you completed in the Full Stack ND program from github into the /var/www/catalog/catalog directory

`$ git clone https://github.com/your-project-url.git projectFolder`

Move the items from your projectFolder into the /var/www/catalog/catalog directory.

`$ mv projectFolder/* .`

Delete the empty projectFolder directory

`$ rm -rf projectFolder`

Make the .git file inaccessible from the browser by creating a .htaccess file.

`$ cd /var/www/catalog/`

`$ sudo vim .htaccess`

Paste in the following code:

`RedirectMatch 404 /\.git`

Now we need to install the needed moduales & packages

Activate the virtual environment:

`$ source venv/bin/activate`

Install httplib2, and requests module in venv:

`$ pip install httplib2`

`$ pip install requests`

Install oauth2client.client:

`$ sudo pip install --upgrade oauth2client`

Install SQLAlchemy:

`$ sudo pip install sqlalchemy`

Install the Python PostgreSQL adapter psycopg:

`$ sudo apt-get install python-psycopg2`

##### 10 - Install and configure PostgreSQL
Install PostgreSQL

`sudo apt-get install postgresql`

Install PostgreSQL needed module

`$ sudo apt-get install postgresql postgresql-contrib`

Rename your `application.py` file to `__init__.py`

Open the `database_setup.py` file

`$ sudo nano database_setup.py`

Change the line starting with 'engine' to

`engine = create_engine('postgresql://catalog:DBPASSWD@localhost/catalog')`

Do the same changes to your `__init__.py` file

Create needed linux user for psql

`$ sudo adduser catalog`

Change to postgres superuser:

`$ sudo su - postgres`

Connect to the psql shell

`$ psql`

Create user with LOGIN role and set a password inside the psql prompt.

`# CREATE USER catalog WITH PASSWORD 'DBPASSWD';`

(# stands for the command prompt in psql)

Allow the user to create database tables

`# ALTER USER catalog CREATEDB;`

Make sure user was in fact created by listing all current roles

`# \du`

Create database

`# CREATE DATABASE catalog WITH OWNER catalog;`

Connect to the catalog database

`# \c catalog`

Revoke all rights from the public

`# REVOKE ALL ON SCHEMA public FROM public;`

Grant only access to the catalog roles

`# GRANT ALL ON SCHEMA public TO catalog;`

Exit out of PostgreSQl and the postgres superuser

`# \q`

`$ exit`

Create postgreSQL database schema

`$ python database_setup.py`

##### 11 - Part 2 of 3 - Running the application

Now we will begin running the application.

Restart Apache server

`$ sudo service apache2 restart`

Go to your IP address to see your new project live.

##### Debugging

If you're getting a server error you can check the server error message file

`$ cat /var/log/apache2/error.log`

If trying to debug, it might be helpful to clear the server error file once in a while

`$ sudo bash -c 'echo > /var/log/apache2/error.log'`

If the `client_secrets.json` file cannot be found in the `__init__.py` file you will have to add the absolute path in the two locations in the file where `client_secrets.json` is being called like so:

`CLIENT_ID = json.loads(open(r'/var/www/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']`

And

`oauth_flow = flow_from_clientsecrets(r'/var/www/catalog/catalog/client_secrets.json', scope='')`

##### 11 - Part 3 of 3 - Get OAuth-Logins Working

Go to http://www.hcidata.info/host2ip.cgi to get your host name by inputing your server's public IP address.

Open the Apache configuration files for the web app

`$ sudo nano /etc/apache2/sites-available/catalog.conf`

Paste in the following line below ServerAdmin and replace HOSTNAME with your hostname

`ServerAlias HOSTNAME`

Enable the virtual host

`$ sudo a2ensite catalog`

Go to your project on the Google Developer Console: https://console.developers.google.com/project

Navigate to your API Manager

Add your host name and public IP-address to your Authorized JavaScript origins and your host name + callback functions to Authorized redirect URIs. example:

Authorized JavaScript origins

`http://c-33-33-33-33.hsd1.il.comcast.net`

Authorized redirect URIs

`http://c-33-33-33-33.hsd1.il.comcast.net/login`

`http://c-33-33-33-33.hsd1.il.comcast.net/gconnect`
