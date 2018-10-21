# Linux Server Configuration
Enclosed in this README file are instructions needed to create and configure a linux server that will deploy the [Item-Catalog-Web-App](https://github.com/Camm66/Item-Catalog-Web-App).
## Location:
* IP Address: `34.207.18.26`
* SSH port: `2000`
* URL: http://cameronmoralesweb.com/home

## Server Configuration Steps:
### 1. Create Server Instance
### 2. Set up SSH
### 3. Firewall rules
### 4. Update packages
### 5. Configure time zone
### 6. Install Apache2
### 7. Install Git
### 8. Clone Item-Catalog-Web-App
### 9. Install Virtual Environment
### 10. Install App Dependencies
### 11. Configure Apache
### 12. Configure PostgreSQL
Install Postgres
  * `sudo apt-get install PostgreSQL`
 
No remote connections are allowed
  * `sudo vim /etc/postgresql/9.3/main/pg_hba.conf`

Adjust database engine in the Flask Application (`__init__.py`, `database_setup.py`):
* `engine = create_engine('postgresql://catalog:password@localhost/catalog')`

Initialize the database
 * `python database_setup.py`

### 13. Restart Apache
* `sudo service apache2 restart`

Then visit the [URL](cameronmoralesweb.com) provided above.
