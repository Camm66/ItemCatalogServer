# Linux Server Configuration
Enclosed in this README file are instructions needed to create and configure a linux server that will deploy the [Item-Catalog-Web-App](https://github.com/Camm66/Item-Catalog-Web-App).

## Location:
* IP Address: `34.207.18.26`
* SSH port: `2200`
* URL: http://cameronmoralesweb.com/

## Server Configuration Instructions:
### 1. Create Server Instance
First, make an [AWS account](https://aws.amazon.com/) and create an AWS Lightsail instance.

Choose the following configuration options:
Instance image:
* Linux/Unix

Platform:
* OS only
* Ubuntu 16.04LTS 

Select a payment plan and create the instance.

### 2. Set up SSH
Download the SSH key from the AWS lightsail account page.
Rename the file
* `lightsailServerKey.pem`

Store the file in your ~/.ssh directory

Now log into the server:
* `ssh -i lightsailServerKey.pem ubuntu@34.207.18.26`

Change SSH port from 22 to 2200
* `sudo nano /etc/ssh/sshd_config`

** ``` # What ports, IPs and protocols we listen for
         Port 2200
```

Enforce key-bases authentication
* `sudo nano /etc/ssh/sshd_confid

** `PasswordAuthentication no`

Disable ssh login for root user
* `sudo nano /etc/ssh/sshd_confid

** `PermitRootLogin no`


Save and restart SSH
* `sudo service ssh restart`

### 3. Firewall rules
Configure the UFW(Uncomplicated Fire Wall) to all communication on the following:
1. 2200 (SSH)
1. 80 (HTTP)
1. 123 (NTP

Check status
* `sudo ufw status`

Deny all incoming requests
* `sudo ufw default deny incoming`

Allow all outgoing requests
* `sudo ufw default deny outgoing`

Allow SSH requests on port 2200
* `sudo ufw allow 2200` 

Allow HTTP requests on port 80
* `sudo ufw allow 80`

Allow NTP requests on port 123
* `sudo ufw allow 123`

Deny SSH requests on port 22
* `sudo ufw deny 22`

Turn on the firewall
* `sudo ufw enable`

Finally, on the visit the Lightsail instance networking page and configure its firewall to permit the designated ports

### 4. Update packages
Check for updates 
* `sudo apt-get update`

Check for upgrades 
* `sudo apt-get upgrade`
### 5. Configure time zone
Change time zone to __UTC__
* `sudo dpkg-reconfigure tzdata`
### 6. Install Apache2
Install apache
* `sudo apt-get install apache2` 

Start the server 
* `sudo service apache2 start`
### 7. Install Git
Install Git
* `sudo apt-get install git`
### 8. Clone Item-Catalog-Web-App
In __/var/www__ create catalog directory
* `sudo mkdir catalog`

Grant permissions 
* `sudo chmod go+rwx catalog`

In __/var/www__ clone the repository
* `git clone https://github.com/camm66/Item-Catalog-Web-App.git`
### 9. Create WSGI file
Install the mod-wsgi module
* `sudo apt-get install python-setuptools libapache2-mod-wsgi`

In __/var/www/catalog__ create the catalog.wsgi file
* `sudo nano catalog.wsgi`

Paste the following:

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'secretkey'
```

Rename project.py for deployment
* `sudo mv project.py __init__.py`
### 10. Install Flask
Install pip
* `sudo apt-get install python-pip`

Install Flask
* `pip install Flask`

Install Additional Dependencies
* `pip install flask flask_wtf httplib2 requests oauth2client sqlalchemy python-psycopg2`
### 11. Install Virtual Environment
Install venv
* `sudo apt-get install python-virtualenv`

Create new environment
* `sudo virtualenv venv`

Activate environment
* `source venv/bin/activate`

Grant permissions
* `sudo chmod -R 777 venv`
### 12. Configure Apache2
Create an apache2 .config file 

* `sudo nano /etc/apache2/sites-available/catalog.conf`

Enter the following:

```
<VirtualHost *:80>
                ServerName http://cameronmoralesweb.com
                ServerAdmin ubuntu@34.207.18.26
                WSGIScriptAlias / /var/www/catalog/catalog.wsgi
                <Directory /var/www/catalog>
                        Order allow,deny
                        Allow from all
                </Directory>
                <Directory /var/www/catalog/catalog/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /static /var/www/catalog/catalog/static
                <Directory /var/www/catalog/catalog/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /images /var/www/catalog/catalog/images
                <Directory /var/www/catalog/catalog/images>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /templates /var/www/catalog/catalog/templates
                <Directory /var/www/catalog/catalog/templates/>
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

### 13. Configure PostgreSQL
Install Postgres
  * `sudo apt-get install postgresql`
 
No remote connections are allowed
* `sudo vim /etc/postgresql/9.3/main/pg_hba.conf`
 
Login to postgres
* `sudo su - postgres`
* `psql`

Create User
* `CREATE USER catalog WITH PASSWORD 'secretpassword';`

Create database 'catalog'
* `ALTER USER catalog CREATEDB;` 
* `CREATE DATABASE catalog WITH OWNER catalog;`

Connect to the database 
* `\c catalog`

Revoke all rights 
* `REVOKE ALL ON SCHEMA public FROM public;`

Grant rights to catalog 
* `GRANT ALL ON SCHEMA public TO catalog;`

Logout and exit postgres
* `\q` 
* `exit`

Adjust database engine in the Flask Application (`__init__.py`, `database_setup.py`):
* `engine = create_engine('postgresql://catalog:secretpassword@localhost/catalog')`

Initialize the database
 * `python database_setup.py`

### 14. Restart Apache
* `sudo service apache2 restart`

Then visit the [URL](cameronmoralesweb.com) provided above.

## Third Party Resources
* [Deploy a Flask Application on Ubuntu](https://devops.profitbricks.com/tutorials/deploy-a-flask-application-on-ubuntu-1404/)
* [How To Use SSH to Connect to a Remote Server in Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-use-ssh-to-connect-to-a-remote-server-in-ubuntu)
* [PostgreSQL command line cheatsheet](https://gist.github.com/Kartones/dd3ff5ec5ea238d4c546)

