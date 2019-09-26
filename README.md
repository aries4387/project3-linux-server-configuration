# Project 3. Item Linux Server Configuration - Udacity
_______________________

## Step 1 : Get your server.

	Create Amazon Web Services account
	Create an instance - select image OS Only - Ubuntu
	Choose instance plan - $3.50 - first month free
	Give instance a host name: project3-udacity
	Create catalog-root on your local machine using ssh-keygen and connect to and log into the instance with SSH from terminal

	$ ssh -i ~/.ssh/catalog_root ubuntu@54.227.80.22
	passphrase : admin

	###Public IP: 54.227.80.22
		HTTP Port: 80
		SSH Port 2200

_______________________

## Step 2 : Secure your server.

	### Update all currently installed packages:
		sudo apt-get update
		sudo apt-get upgrade
		# security updates
		sudo apt-get install unattended-upgrades
		# install them manually
		sudo unattended-upgrades

	### Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it.
		sudo nano /etc/ssh/sshd_config - find port 22 and change it to 2200

	### Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
		sudo ufw status
		sudo ufw default deny incoming
		sudo ufw default allow outgoing
		sudo ufw allow 2200
		sudo ufw allow 80
		sudo ufw allow 123
		sudo ufw enable

_______________________

## Step 3 : Give grader access.

	### add new user grader
		sudo adduser grader

	### Give grader sudo access:
		sudo cat /etc/sudoers
		sudo ls /etc/sudoers.d
		sudo nano /etc/sudoers.d/grader

	### generate the key pair locally : ssh-keygen - linuxcourse
		passphrase: grader

	### add the public key for grader on server:
		create .ssh dir 
		mkdir .ssh
		touch .ssh.authorized_keys
		nano .ssh/authorized_keys
		chmod 700 .ssh
		chmod 644 authorized_keys

	### now you can ssh to server using grader user: 
		ssh grader@54.227.80.22 -p 2200 -i ~/.ssh/linuxcourse

	### disable password authentication
		sudo nano /etc/ssh/sshd_config - change passwordAuthentication to no
		set PermitRootLogin to no
		sudo service ssh restart

_______________________

## Step 3 : Install required components 


	sudo apt-get install python3
	sudo apt-get install python3-setuptools
	sudo apt-get install apache2 libapache2-mod-wsgi-py3

	sudo service apache2 restart

	sudo apt-get install postgresql
	sudo apt-get install git-core

	### create Database

		sudo -u postgres psql

		CREATE USER catalog WITH PASSWORD 'password';
		CREATE DATABASE catalog WITH OWNER catalog;
		\c catalog
		REVOKE ALL ON SCHEMA public FROM public;
		GRANT ALL ON SCHEMA public TO catalog;
		\q
		exit
_______________________

## Step 4 : Setup the Item catalog project

	In database_setup.py , populate_db.py and project2-item-catalog.py file change the DB info: 
	engine = create_engine('postgresql://catalog:password@localhost/catalog')

	git clone https://github.com/aries4387/project2-item-catalog.git

	### create configuration file for your project
		sudo nano /etc/apache2/sites-available/catalog.conf

		<VirtualHost *:80>
    	ServerName 54.227.80.22
    	ServerAlias 54.227.80.22.xip.io

    	WSGIScriptAlias / /var/www/catalog/wsgi.py

    	<Directory /var/www/catalog>
        	Order allow,deny
        	Allow from all
    	</Directory>
		</VirtualHost>


	### wsgi.py file:

		#!/usr/bin/python
		import sys
		sys.path.insert(0, '/var/www/')

		from catalog import app as application

		application.secret_key = 'admin'

	### Enable site:
		sudo a2ensite catalog  # enable site
		sudo service apache2 reload

_______________________

## Step 5 : Google Oauth setup

	https://console.developers.google.com/
	go to OAuth consent screen : add xip.io in authorized domains
	go to credentials:
		add  http://54.227.80.22.xip.io to Authorized JavaScript origins 
		add http://54.227.80.22.xip.io 	and http://54.227.80.22.xip.io/login/callback to authorized redirect URLs
		Download the new client_secrets.json and put it into catalog folder

## Now Run the application:  http://54.227.80.22.xip.io

## A list of used third-party resources as references to build this project is required.

Udacity videos
Stack overflow
github projects