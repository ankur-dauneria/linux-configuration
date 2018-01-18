<p align="center">
  <a href="#"><img src="title.png"></a>
</p>
<p align="center">
  <a href="https://opensource.org/licenses/MIT" target="_blank">
    <img src="https://img.shields.io/badge/License-MIT-green.svg" alt="License">
  </a>
  <a href="#" target="_blank">
    <img src="https://img.shields.io/badge/documentation-complete-green.svg" alt="Documentation">
  </a>
</p>

> Configuring the Linux server and installing the item-catalog web application. It includes installing updates, securing it, and installing/configuring web and database servers.

* EC2 URL : http://ec2-35-154-108-134.ap-south-1.compute.amazonaws.com/

* Local IP : http://35.154.108.134/

* Port : 2200

# Configuration Steps

## Step 1 : Lauch of virtual machine

* Go to https://lightsail.aws.amazon.com/ls/webapp/home/instances

* Create a instance

## Step 2: SSH into your server

* Go to Lightsail Accounts, click SSH keys tab and download the SSH Key.

* ```ssh -i ~/Downloads/<SSH Key> ubuntu@35.154.108.134```

## Step 3: Create a new user grader

* `sudo adduser grader`

* `sudo apt-get install finger`

* `finger grader`

## Step 4: Give grader the permission to sudo

* sudo nano /etc/sudoers.d/grader`
    * Add line `grader ALL=(ALL:ALL) ALL`

## Step 5: Update all installed packages

* Update all currently installed packages
    * `sudo apt-get update`

* Upgrade all
    * `sudo apt-get upgrade`

## Step 6: Change the SSH port from 22 to 2200

* `nano /etc/ssh/sshd_config`
    * Change the port from 22 to 2200
    * Change `PermitRootLogin without-password` to `PermitRootLogin no`
    * Change `PasswordAuthentication` from `no` to `yes`.
    * Add `AllowUsers grader`, to allow login using grader
* Add port `2200` as `Custom TCP port` on Lightsail web interface under networking tab
* restart ssh
    * `sudo service ssh restart`

## Step 7: SSH keys generation

* generate SSH keys
    * `ssh-keygen` on local computer
        * enter `grader`
        * press enter for empty paraphrase
* Login using grader user
    * `su - grader`
* Make directory `.ssh`
    * `mkdir .ssh`
* Create file `authorized_keys`
    * `touch .ssh/authorized_keys`
    * `nano .ssh/authorized_keys`
        * Copy contents of `/home/ubuntu/grader.pub` to `.ssh/authorized_keys`
        * save it
    * `chmod 700 .ssh`
    * `chmod 600 .ssh/authorized_keys`
* `nano /etc/ssh/sshd_config`
    * Change `PasswordAuthentication` from `yes` to `no`.
* `sudo service ssh restart`

## Step 8: Login using grader user

*  Using the generated private keys on local computer, login into system
    *  ```ssh -i /path/to/SSH-Keys -p 2200 grader@35.154.108.134```

## Step 9: Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80) and NTP (port 123)

* Check the firewall status
    * `sudo ufw status`
* Block all incoming connections on all ports using
    * `sudo ufw default deny incoming`
* Allow outgoing connections on all port using
    * `sudo ufw default allow outgoing`
* Allow incoming connections for SSH port (2200) using
    * `sudo ufw allow 2200/tcp`
* Allow incoming connection for HTTP (port 80) using
    * `sudo ufw allow 80/tcp`
* Allow incoming connection for NTP port (123) using
    * `sudo ufw allow 123/udp`
* Check the added rule using
    * `sudo ufw show added`
* Enable the firewall
    * `sudo ufw enable`
* Check firewall status
    * `sudo ufw status`

## Step 10: Confirefure timezone to UTC

* `sudo dpkg-reconfigure tzdata`

## Step 11: Install and Configure Apache to server a Python mod_wsgi application

* Install Apache weeb server:
    * `sudo apt-get install apache2`
* Open a browser and open your public IP address, e.g. http://35.154.108.134/. It should say 'It works!'.
    * Install mod_wsgi for serving Python apps from Apache and helper package python-setuptools:
        * `sudo apt-get install python-setuptools libapache2-mod-wsgi`
    * Restart Apache server for mod_wsgi to load:
        * `sudo service apache2 restart`
    * Create an empty Apache config with the hostname:
        * `echo ServerName HOSTNAME | sudo tee /etc/apache2/conf-available/fqdn.conf`
    * Enable the new config file:
        * `sudo a2enconf fqdn`

## Step 12: Install git, clone and setup your Catalog app project:

### Step 12.1 : Install and configure git

* Install Git
    * `sudo apt-get install git`
* Setup your name
    * `git config --global user.name "YOUR NAME"`
* Setup your email
    * `git config --global user.email "YOUR EMAIL ADDRESS`

### Step 12.2 : Setup for delployment a Flask Application on Ubuntu

* Extnd Python with additional packages that enable Apache to server Flask application
    * `sudo apt-get install libapache2-mod-wsgi python-dev`
* Enable mod_wsgi
    * `sudo a2enmod wsgi`
* Create a Flask app
* Go to `www` directory
    * `cd /var/www`
* Setup a directory (`catalog`) for the app
    * `sudo mkdir catalog`
    * Go to catalog
        * `cd catalog`
        * `sudo mkdir catalog`
        * `cd catalog`,
        * `sudo mkdir static templates`
* Create file that contain Flask application logic:
    * `sudo nano __init__.py`
    * Paste the following code:
        ```python
             from flask import Flask
             app = Flask(__name__)

             @app.route("/")
             def hello():
                 return "Hello!"

             if __name__ == "__main__":
                 app.run()
        ```
    * Install Flask
        * `sudo apt-get install python-pip`
        * Install virtualenv:
            * `sudo pip install virtualenv`
        * Set virtual environment to name 'venv':
            * `sudo virtualenv venv`
        * Enable all permissions
            * `sudo chmod -R 777 venv`
        * Activate the virtual environment
            * `source venv/bin/activate`
        * Install Flask inside the virtual env
            * `pip install Flask`
        * Run the app:
            * `python __init__.py`
        * Deactivate the environment:
            * `deactivate`
        * Configure and enable new virtual host
        * Create the virtual host config file:
            * `sudo nano /etc/apache2/sites-available/catalog.conf`
        * Paste the following lines of code and change name and address according to the application:
            * Check hostname using http://www.hcidata.info/host2ip.cgi against the public IP mention in Amazon virtual instance.
              This hosname is YOUR-HOSTNAME

            ```
            <VirtualHost *:80>
                ServerName PUBLIC-IP-ADDRESS
                ServerAdmin admin@PUBLIC-IP-ADDRESS
                ServerAlias YOUR-HOSTNAME
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
        * Enable the virtual host:
            * `sudo a2ensite catalog`
        * Create the .wsgi File and Restart Apache
            * Create wsgi file:
                * `cd /var/www/catalog` and
                * `sudo vim catalog.wsgi`
            * Paste in the following lines of code
                ```
                #!/usr/bin/python
                import sys
                import logging
                logging.basicConfig(stream=sys.stderr)
                sys.path.insert(0,"/var/www/catalog/")

                from catalog import app as application
                application.secret_key = 'Add your secret key'
                ```
            * Restart Apache:
                * `sudo service apache2 restart`

### Step 12.3: Clone GitHub repository and make it web inaccessible

* Clone `item-catalog` project solution repository on GitHub:
    * 'sudo git clone https://github.com/ankur-dauneria/item-catalogue.git`
* Move all content of created `item-catalog` directory to `/var/www/catalog/catalog/`-directory and delete the leftover empty directory.
* Make the GitHub repository inaccessible:
    * Create and open .htaccess file:
    * `cd /var/www/catalog/` and `sudo vim .htaccess`
        Paste in the following:
        `RedirectMatch 404 /\.git`

### Step 12.4: Install needed packages

* Activate virtual environment:
    * `source venv/bin/activate`
* Install httplib2 module in venv:
    * `pip install httplib2`
* Install requests module in venv:
    * `pip install requests`
* Install flask.ext.seasurf
    * `sudo pip install flask-seasurf`
* Install oauth2client.client
    * `sudo pip install --upgrade oauth2client`
* Install SQLAlchemy:
    * `sudo pip install sqlalchemy`
* Install the Python PostgreSQL adapter psycopg:
    * `sudo apt-get install python-psycopg2`

* If some using requirements.txt file, then also use
    * `sudo pip install -r requirements.txt`

* Note: Some may need to be installed globally while other can work when locally installed

### Step 12.5:  Install and configure PostgreSQL

* Install PostgreSQL:
    * `sudo apt-get install postgresql postgresql-contrib`
* Check that no remote connections are allowed (default):
    * `sudo vim /etc/postgresql/9.5/main/pg_hba.conf`
* Open the database setup file:
    * `sudo vim database_setup.py`
* Change the line starting with "engine" to (fill in a password):
    * `python engine = create_engine('postgresql://catalog:PW-FOR-DB@localhost/catalog')`
* Change the same line in `application.py` respectively
    * Change the location to raw location in python using `r` e.g: r'/var/www/catalog/catalog/' for client_secret.json file
    * Change directory using `os.chdir(r'/var/www/catalog/catalog/')` to the top before accessing client_secret.json file
* Rename application.py:
    * `sudo mv application.py __init__.py`
* Create needed linux user for psql:
    * `sudo adduser catalog (choose a password)`
* Change to default user postgres:
    * `sudo su - postgres`
* Connect to the system:
    * `psql`
* Add postgre user with password:
* Create user with LOGIN role and set a password:
    `# CREATE USER catalog WITH PASSWORD 'PW-FOR-DB';` (`#` stands for the command prompt in psql)
* Allow the user to create database tables:
    `ALTER USER catalog CREATEDB;`
* List current roles and their attributes:
    `\du`
* Create database:
    * `CREATE DATABASE catalog WITH OWNER catalog;`
* Connect to the database catalog
    * `\c catalog`
* Revoke all rights:
    * `REVOKE ALL ON SCHEMA public FROM public;`
* Grant only access to the catalog role:
    * `GRANT ALL ON SCHEMA public TO catalog;`
* Exit out of PostgreSQl and the postgres user:
    * `\q`, then `exit`
* Create postgreSQL database schema:
    * `python database_setup.py`

###  Step 12.6: Run application

* Restart Apache:
    `sudo service apache2 restart`
* Open a browser and put in your public ip-address as url, e.g. http://ec2-35-154-108-134.ap-south-1.compute.amazonaws.com/ - if everything works, the application should come up
* If getting an internal server error, check the Apache error files:
    * `sudo tail -20 /var/log/apache2/error.log`


### Step 12.7: Get OAuth-Logins Working

* To get the Google+ authorization working:
    * Go to the project on the Developer Console: https://console.developers.google.com/project
    * Navigate to APIs & auth > Credentials > Edit Settings
    * add your host name and public IP-address to your Authorized JavaScript origins and your host name + oauth2callback to Authorized redirect URIs, e.g. http://ec2-35-154-108-134.ap-south-1.compute.amazonaws.com/oauth2callback
    * Update data-client id in `templates/login.html`
    * restart apache
        * `sudo service apache2 restart`

### Step 12.8: Give permission to img folder to allow saving deleting of images files

* Give permission to `static/img` folder
    * chmod 777 -R `static/img`

### Step 12.9: Install Monitor application Glances

* `sudo pip install Glances`

* Run glances - `glances`


## References

* https://github.com/stueken/FSND-P5_Linux-Server-Configuration

* https://github.com/anumsh/Linux-Server-Configuration

* https://github.com/SteveWooding/fullstack-nanodegree-linux-server-config