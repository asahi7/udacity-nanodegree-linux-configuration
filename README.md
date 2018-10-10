# Udacity Nanodegree Linux Configuration Project

## Overview

This is the required project of Udacity Full Stack Nanodegree course. The goal is to upload Flask [project](https://github.com/asahi7/udacity-nanodegree-catalog-app) to AWS Lightsail and configure it to be available on the Internet. The website is available at http://13.125.214.180.xip.io.

For graders: SSH port is 2200, the password is '123456' (without quotes). Some of the software installed is listed in /home/grader/requirements.txt and others are listed in this file.

Resources used:
* https://www.jakowicz.com/flask-apache-wsgi/
* https://stackoverflow.com

## Configuring SSH
* sudo vi /etc/ssh/sshd_config, inside the file change port from 22 to 2200
* service ssh restart
* Under networking tab in AWS Lightsail of the instance, add a custom firewall rule with TCP 2200
* Now, to access the server, run: `ssh ubuntu@13.125.214.180 -p 2200`

## Adding a new sudo user and create SSH key-pair
* `sudo adduser grader`
* `sudo usermod -aG sudo grader`
* Run: `ssh-keygen -t rsa` on my local machine, named keys as grader
* Copied contents of grader.pub into ~/.ssh/authorized_keys of grader user. Changed the user from root to grader by using command: `sudo su - grader`
* `ssh -i ~/.ssh/grader grader@13.125.214.180 -p 2200`, enter password '123456' (without quotes)
 
## Installing Apache Server with mod_wsgi
* `sudo apt-get install apache2`
* Visit http://13.125.214.180.xip.io/
* `sudo apt-get install libapache2-mod-wsgi`
* In file /etc/apache2/sites-enabled/000-default.conf add: `WSGIScriptAlias / /var/www/html/myapp.wsgi` right before closing `</VirtualHost>` tag.
* `sudo apache2ctl restart`
* Create a new file: `sudo vi /var/www/html/myapp.wsgi` with contents:
```python2
def application(environ, start_response):
    status = '200 OK'
    output = 'Hello Udacity!'
    response_headers = [('Content-type', 'text/plain'), ('Content-Length', str(len(output)))]
    start_response(status, response_headers)
    return [output]
```
* Visit http://13.125.214.180.xip.io/
* Also, install PostgreSQL: `sudo apt-get install postgresql`

## Prerequisites to running Item Catalog Project
* Firstly, let's copy item catalog project from local to remote machines: `scp -r -i ~/.ssh/grader -P 2200 catalog grader@13.125.214.180:~/`, password: '123456' (without quotes)
* Copy requirements.txt from local machine: `scp -r -i ~/.ssh/grader -P 2200 requirements.txt grader@13.125.214.180:~/`
* Installing requirements with pip: `pip install -r requirements.txt `
* Create a `catalogdb` database: `sudo -u postgres psql`, and after: `create database catalogdb;`
* In the catalog folder, run: `python init_db.py`, this will initialize database tables
* Add following contents to the .wsgi application file (this is the wrapper for our applicatio) `sudo vi /var/www/html/myapp.wsgi`:
```
import sys
sys.path.append('/home/grader/catalog')
from catalog import app as applicatio
```
* Add following contents to the file: `sudo vi /etc/apache2/sites-enabled/000-default.conf`:
```
<VirtualHost *:80>
        ServerName my.catalog
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
        WSGIDaemonProcess catalog user=grader group=grader threads=5 home=/var/www/html/
        WSGIScriptAlias / /var/www/html/myapp.wsgi
        <directory /var/www/html>
                WSGIProcessGroup catalog
                WSGIApplicationGroup %{GLOBAL}
                WSGIScriptReloading On
                Order deny,allow
                Allow from all
        </directory>
</VirtualHost>
```
* Configure your authorized domains for your project on: https://console.developers.google.com/ and upload a new client_secret.json to the server
* Lastly, do `sudo apache2ctl restart`
* Now you can visit http://13.125.214.180.xip.io/
