# python-linux-configuration
Setup process for a Python Web Application on an Ubuntu AWS Liighsail Instance
http://ec2-18-197-157-145.eu-central-1.compute.amazonaws.com

## Setting Up AWS
* Open an Amazon account
* Access the AWs Lightsail https://lightsail.aws.amazon.com/ls/webapp/home
* Click on the Create Instance
* Run through this porcess selecting the OS only Blueprint and then the Ubuntu configuration
* Once complete this will setup an instance

## Configuring Linux Server
* Commence by downloading your instance private key to your device
* Next under your instance home select the networking tab and configure the custom ports 2200 tcp and 123 udp and click save
### Updating Linux server
* ssh into the server using the key in your cli (ssh ubuntu@server-ip -i locationToKey)
* Update your ubuntu instance running the command: $ sudo apt-get update
* Then upgrade the packages using command: $ sudo apt-get upgrade
### Change SSH Port
* Open the sshd_config file $ sudo nano /etc/ssh/sshd_config
* Chnage the ssh port from 22 to 2200
* Write out the changes to the file
### Firewall Configuration
* Allow port 2200 using command $ sudo ufw allow 2200/tcp
* Allow port 80 for http access for out application $ sudo ufw allow 80/tcp
* Allow port 123 $sudo ufw allow 123/udp
* Enable firewall $sudo ufw enable (Ensure the above step are well confifgured to prevent locking yourself out of your instance)
* Restart ssh service using command $ sudo service ssh restart
### Adding Another User (grader)
* Switch to rot user using command $ sudo su
* Add user $ sudo adduser grader
* Grant grader sudo ability with command $ sudo nano /etc/sudoers.d/grader, and type grader ALL=(ALL:ALL) ALL.
### Generate key for Grader
* Switch to the newly created user using $ sudo su grader
* Ensure working directory is grader's home
* Create a file with the name authorized_keys inside the .ssh directory: $ sudo touch /.ssh/authorized_keys.
* Change the permission for the .ssh folder: $ sudo chmod 700 .ssh
* Change the permissions for the authorized_keys file: $ sudo chmod 644 /.ssh/authorized_keys
### Generate key locally
* In new terminal using the command set up a ssk key: $ ssh-keygen
* Pressing enter all the way through this will set up key in the .ssh/id_rsa.pub file
* Open this file using sudo cat ~/.ssh/id_rsa.pub
* Open the authorized_keys file created for the grader user: $ sudo nano /.ssh/authorized_keys
* Paste the content
* Restart the ssh service: $ sudo service ssh restart 
### Step C: Setting up web server and application
* Install apache2 using command $ sudo apt-get install apache2.
* Also install mod-wsggi for python to enable python run on apache using command: $ sudo apt-get install libapache2-mod-wsgi python-dev
* In public html directory in this case /var/www make directory catalogmanager
* In this directroy clone git repository (https://github.com/afolabiabass/catalogmanager)
* Following command would help install modules required for python enviroment
* Install pip: $ sudo apt-get install python-pip.
* $ sudo apt-get install python-pip
* $ sudo pip install Flask
* $ sudo pip install httplib2
* $ sudo pip install passlib
* $ sudo pip install sqlalchemy
* $ sudo pip install oauth2client
* $ sudo pip install sqlalchemy
* $ sudo pip install sqlalchemy_utils
* Rename the application file (app.py) to __init__.py 
#### Setup Database
* Install PostgreSQL database $ sudo apt-get install postgresql
* Create a new database named "catalogdb" $ sudo -u postgres psql -c "CREATE DATABASE catalogdb;"
* Create database user  catalog sudo -u postgres psql -c "CREATE USER catalog;"
* Set password for user catalog sudo -u postgres psql -c "ALTER USER catalog PASSWORD 'catalog';"
* Grant user all priviledges to the catalogdb database sudo -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE catalogdb to catalog;"
* Install psycopg2: sudo apt-get -qqy install postgresql python-psycopg2
* Run application database sudo python models.py
#### Configure mod-wsgi to link python to apache
* Create the catalogmanager.conf file: $ sudo nano /etc/apache2/sites-available/catalogmanager.conf
* Paste the text below inside the FlaskApp.conf file:  
	```  
	<VirtualHost *:80>  
		ServerName 18.197.157.145 
		ServerAdmin [af@mail.com](mail.com)  
		WSGIScriptAlias / /var/www/catalogmanager/catalogmanager.wsgi  
		<Directory /var/www/catalogmanager/catalogmanager/>  
			Order allow,deny  
			Allow from all  
		</Directory>  
		Alias /static /var/www/catalogmanager/catalogmanager/static  
		<Directory /var/www/catalogmanager/catalogmanager/static/>  
			Order allow,deny  
			Allow from all  
		</Directory>  
		ErrorLog ${APACHE_LOG_DIR}/error.log  
		LogLevel warn  
		CustomLog ${APACHE_LOG_DIR}/access.log combined  
	</VirtualHost>  
	```  
* Disable the default virtual host:  $ sudo a2dissite 000-default.conf
* Enable the new virtual host: $ sudo a2ensite catalogmanager.conf 
* Create file var/www/catalogmanager/catalogmanager.wsgi declare in the virtualhost config $ sudo nano var/www/catalogmanager/catalogmanager.wsgi
* Paste the text below into catalogmanager.wsgi

	```  
	#!/usr/bin/python  
	import sys  
	import logging  
	logging.basicConfig(stream=sys.stderr)  
	sys.path.insert(0,"/var/www/catalogmanager/")  

	from FlaskApp import app as application  
	application.secret_key = 'super_secret_key'  
	```
 * Finally, Restart apache  $ sudo service apache restart
 
### Guides:
* [https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps](https://www.digitalocean.com)
* [https://github.com/hicham-alaoui/ha-linux-server-config](https://github.com)
* [https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps](https://www.digitalocean.com)
* [https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2](https://www.digitalocean.com)
