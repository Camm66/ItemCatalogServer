# Linux Server Configuration
Enclosed in this README file are instructions needed to create and configure a linux server that will deploy the [Item-Catalog-Web-App](https://github.com/Camm66/Item-Catalog-Web-App).

## Location:
* IP Address: `34.207.18.26`
* SSH port: `2000`
* URL: http://cameronmoralesweb.com/home

## Server Configuration Instructions:
### 1. Create Server Instance
### 2. Set up SSH
### 3. Firewall rules
### 4. Update packages
### 5. Configure time zone
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
* `sudo chmod catalog`
In __/var/www__ clone the repository
* `git clone https://github.com/aaayumi/item-catalog-udacity.git`
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
* `pip install bleach httplib2 request oauth2client sqlalchemy python-psycopg2`
### 11. Install Virtual Environment
Install venv
* `sudo apt-get install python-virtualenv`
Create new environment
* `sudo virtualenv venv`
Activate environment
* `source venv/bin/activate
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
                WSGIScriptAlias / /var/www/catalog/flaskapp.wsgi
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
                Alias /images /var/www/catalog/catalog/static/images
                <Directory /var/www/catalog/catalog/static/images>
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
  * `sudo apt-get install PostgreSQL`
 
No remote connections are allowed
* `sudo vim /etc/postgresql/9.3/main/pg_hba.conf`
 
Login to postgres
* `sudo su - postgres`
* `psql`

Create User
* `CREATE USER catalog WITH PASSWORD 'secretpassword'`

Create database 'catalog'
* `ALTER USER catalog CREATEDB` 
* `CREATE DATABASE catalog WITH OWNER catalog`

Connect to the database 
* `\c catalog`

Revoke all rights 
* `REVOKE ALL ON SCHEMA public FROM public`

Grant rights to catalog 
* `GRANT ALL ON SCHEMA public TO catalog`

Logout and exit postgres
* `\q` 
* `exit`

Adjust database engine in the Flask Application (`__init__.py`, `database_setup.py`):
* `engine = create_engine('postgresql://catalog:password@localhost/catalog')`

Initialize the database
 * `python database_setup.py`

### 14. Restart Apache
* `sudo service apache2 restart`

Then visit the [URL](cameronmoralesweb.com) provided above.
