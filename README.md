# udacity-nanodegree-linux-configuration

## Configuring SSH
* sudo vi /etc/ssh/sshd_config, inside the file change port from 22 to 2200
* service ssh restart
* Under networking tab in AWS Lightsail of the instance, add a custom firewall rule with TCP 2200
* Now, to access the server, run: `ssh ubuntu@13.125.214.180 -p 2200`
