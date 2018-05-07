# Linux Server Configuration

Linux Server Configuration project files for the Udacity Fullstack Nanodegree.

IP Address: 18.188.115.41

Port: 2200

URL: [http://ec2-18-188-115-41.us-east-2.compute.amazonaws.com](http://ec2-18-188-115-41.us-east-2.compute.amazonaws.com)

### Create Ubuntu VM Instance

* Create an Amazon Lightsail instance using the OS Only + Ubuntu option.

* Download the default key pair file and log into the instance using SSH:

```
ssh ubuntu@[IP-ADDRESS] -i [/PATH/TO/YOUR-KEY-PAIR-FILE]
```

### Update packages

* Update all packages and linux distro:

```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade
```

### Add Grader User

**On local machine:**

* Generate a key pair to be used by the `grader` user:

```
ssh-keygen
```

* Choose a location for the key pair file and save (e.g `~/.ssh/lightsail-grader`)

**On remote machine:**

* Add **grader** user:

```
sudo adduser grader
```

* Enter password and details for user

* Change to grader home directory and create an `.ssh` directory:

```
cd /home/grader
sudo mkdir .ssh
sudo nano .ssh/authorized_keys
```

* Copy and paste the contents of the `lightsail-grader.pub` file from the local machine and save

* Set permissions and owner/group on `.ssh` recursively:

```
sudo chown -R grader:grader .ssh
sudo chmod -R 700 .ssh
```

### Give **grader** sudo privileges

* Become the root user

```
sudo -i
```

* Create a new sudoer file in the `/etc/sudoers.d` directory:

```
cd /etc/sudoers.d
nano grader
```

* Copy and paste the following into the file and save:

```
grader ALL=(ALL) NOPASSWD:ALL
```

* Change permissions on file and exit sudo:

```
chmod 440 grader
```

* Exit as root user:

```
exit
```

### Change default SSH port and turn off SSH passwords

* Edit the `sshd_config` file:

```
sudo nano /etc/ssh/sshd_config
```

* Change port to 2200:

```
Port 2200
```

* Disable the root login:

```
PermitRootLogin no
```

* Enable authorized_keys to be used:

```
AuthorizedKeysFile      %h/.ssh/authorized_keys
```

* Disable password authentication:

```
PasswordAuthentication no
```

* Restart `ssh`:

```
sudo service ssh restart
```

### Configure Uncomplicated Firewall (UFW)

* Allow/deny access to ports:

```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow www
sudo ufw allow 2200/tcp
sudo ufw deny 22
sudo ufw enable
```

* Exit `ssh` session:

```
exit
```

### Allow connection to UFW ports for Lightsail instance

* Log in to your Lightsail dashboard

* Select the instance and click on Networking

* Under firewall, select edit rules

* Delete the SSH rule

* Select Add Another

* From the dropdown, select Custom TCP and enter 2200

* Save

* SSH into your newly created and configured `grader` user on port 2200:

```
ssh grader@[IP-ADDRESS] -i [PATH/TO/GRADER-KEY-PAIR-FILE] -p 2200
```

### Configure Timezone

* Run the TimeZone setup tool and select your timezone:

```
sudo dpkg-reconfigure tzdata
```

### Apache2 and WSGI

* Install the apache2 web server and Python3 WSGI mod:

```
sudo apt-get install apache2 libapache2-mod-wsgi-py3
```

* Disable default virtual servers:

```
sudo a2dissite *
```

### Install GIT

* Install the GIT package:

```
sudo apt-get install git
```


### Setup Web Application

* Change directory to `/var/www`:

```
cd /var/www
```

* Create a new directory to contain the web application and `cd` into it:

```
sudo mkdir catalog
cd catalog
```

* Clone the GIT repo containing the web application files into a directory called `catalog`:

```
sudo git clone https://github.com/kfernandes29/CatalogItems.git catalog
```

### The WSGI file

* Change directory to `/var/www/catalog`:

```
cd /var/www/catalog
```

* Create the `app.wsgi` file:

```
sudo nano app.wsgi
```

* Copy and paste this into the `app.wsgi` file and save it:

```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/catalog/")
sys.path.insert(1, "/var/www/catalog/")

from catalog import app as application
application.secret_key = "my_secret_key"
```

### Installing VirtualEnvironment and Dependencies

* Install the VirtualEnvironment package:

```
sudo apt-get install python3-virtualenv
```

* Change to the `/var/www/catalog` directory:

```
cd /var/www/catalog
```

* Set up a virtual environment:

```
sudo python3 -m venv env
```

* Change permissions:

```
sudo chmod -R 777 env
```

* Activate the enviroment:

```
source env/bin/activate
```

* Install dependencies:

```
pip install flask requests sqlalchemy psycopg2 python-slugify oauth2client passlib
```

* Deactivate:

```
deactivate
```

### Configure PostgreSQL

* Install PostgreSQL:

```
sudo apt-get install postgresql
```

* Switch to postgres user:

```
sudo su - postgres
```

* Enter psql:

```
psql
```

* Create database:

```
CREATE DATABASE catalog;
```

* Create a role `catalog` and grant all privileges on `catalog` database:

```
CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';
GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
```

* Exit PSQL and log out of `postgres` user:

```
\q
exit
```

### Configure Apache2 Server

* Change directory to `/etc/apache2/sites-available`:

```
cd /etc/apache2/sites-available
```

* Create a file for the virtual host:

```
sudo nano catalog.conf
```

* Copy and paste these contents into the file:

```
WSGIPythonHome "/var/www/catalog/env/bin/python3"
WSGIPythonPath "/var/www/catalog/env/lib/python3.5/site-packages"

<VirtualHost *:80>
    ServerName catalog
    ServerAlias 18.219.72.227
    WSGIScriptAlias / /var/www/catalog/app.wsgi
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

* Enable the site:

```
sudo a2ensite catalog
```

* Restart the apache server:

```
sudo service apache2 restart
```

### Seed the Database

* Change directory to `/var/www/catalog`:

```
cd /var/www/catalog
```

* Activate the virtual environment:

```
source env/bin/activate
```

* Run the `database_seeder.py` file:

```
python3 catalog/database_seed.py
```

* Deactivate the virtual enviroment:

```
deactivate
```

# Credits

[https://www.linode.com/docs/development/python/create-a-python-virtualenv-on-ubuntu-1610/](https://www.linode.com/docs/development/python/create-a-python-virtualenv-on-ubuntu-1610/)
