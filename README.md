# udacity-nanodegree-linux-configuration

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
* `ssh -i ~/.ssh/grader grader@13.125.214.180 -p 2200`, enter password 123456
 
## Installing Apache Server with mod_wsgi
* `sudo apt-get install apache2`
* Visit [http://13.125.214.180.xip.io/]()
* `sudo apt-get install libapache2-mod-wsgi`
* In file /etc/apache2/sites-enabled/000-default.conf add: `WSGIScriptAlias / /var/www/html/myapp.wsgi` right before closing `</VirtualHost>` tag.
* `sudo apache2ctl restart`
* Create a new file: `sudo nano /var/www/html/myapp.wsgi` with contents:
```python2
def application(environ, start_response):
    status = '200 OK'
    output = 'Hello Udacity!'
    response_headers = [('Content-type', 'text/plain'), ('Content-Length', str(len(output)))]
    start_response(status, response_headers)
    return [output]
```
* Visit [http://13.125.214.180.xip.io/]()
* Also, install PostgreSQL: `sudo apt-get install postgresql`
