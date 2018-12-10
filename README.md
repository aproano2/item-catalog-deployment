# Deployment of the Item Catalog App

This repo includes the work for the last project of the Udacity Full Stack Nanodegree. The application to be deployed can be found in [here](https://github.com/aproano2/item-catalog).

The server used to deploy the application can be accessed as follows:

- IP address: 54.188.60.170

- Open SSH port: 2200

- Application URL: [http://54.188.60.170.xip.io](http://54.188.60.170.xip.io)

## Steps:

1. Create an Ubuntu 16.04 instance in [Amazon Lightsail](https://lightsail.aws.amazon.com). Upgrade packages:
```
sudo apt-get update
sudo apt-get upgrade
```
2. The SSH default port is changed to 2200 by modifying the `/etc/ssh/sshd_config` file. (Note: On the Amazon Lightsail dashboard, port 2200 must be enabled). Restart SSH service:
```
sudo service ssh restart
```
3. Enable firewall rules:
```
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/udp
sudo ufw enable
```
4. Create a grader user, add sudo access and ssh access:
```
sudo adduser grader
sudo echo "grader ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/grader
sudo -s grader
mkdir /home/grader/.ssh
echo <PUBLIC KEY> > /home/grader/.ssh/authorized_keys
chmod 700 /home/grader/.ssh
chmod 600 /home/grader/.ssh/authorized_keys
```
5. Configure UTC time:
```
sudo dpkg-reconfigure tzdata
```
6. Installl required packages and start apache service:
```
sudo apt-get install git apache2 postgresql
sudo apt-get install libapache2-mod-wsgi python-dev python-pip
sudo a2enmod wsgi
sudo service apache2 start
```
7. Deploy application:
```
cd /var/www
sudo mkdir FlaskApp
cd FlaskApp
sudo git clone https://github.com/aproano2/item-catalog.git FlaskApp
cd FlaskApp
mv application.py __init__.py
```
8. Create database as `postgres` user:
```
psql
postgres=# CREATE ROLE catalog WITH LOGIN PASSWORD 'insecure';
postgres=# ALTER ROLE catalog CREATEDB;
```
9. Configure host by adding the following files:

```
cat  /etc/apache2/sites-available/FlaskApp.conf
<VirtualHost *:80>
    ServerName 54.188.60.170
    ServerAdmin alejo2q@gmail.com
    WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
    <Directory /var/www/FlaskApp/FlaskApp/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/FlaskApp/FlaskApp/static
    <Directory /var/www/FlaskApp/FlaskApp/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

cat /var/www/FlaskApp/flaskapp.wsgi
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")

from FlaskApp import app as application
application.secret_key = 'Your Secret'

sudo a2ensite FlaskApp
```

10. Restart Apache
```
sudo service apache2 restart
```
