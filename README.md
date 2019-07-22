Linux Server Configuration Project
Server Details
IP address : 18.197.20.177

SSH port : 2200

## Configuration steps
1. Create an instance in AWS Lightsail
	Go to AWS Lightsail and create a new account / sign in with your account.

	Click Create instance and choose Linux/Unix,OS only Ubuntu 16.04LTS

	Choose a payment plan (the cheapest plan is enough for now and it's free for first month)

	Click Create button to create an instance.

2. Set up SSH key
	Go to account page from your AWS account. You will find your SSH key there.

	Download your SSH key, the file name will be like LightsailDefaultPrivateKey-*.pem

	Navigate to the directory where your file is stored in your terminal.

	Run chmod 600 LightsailDefaultPrivateKey-*.pem to restrict the file permission.

	Change name to lightsail_key.rsa.

	Run a command ssh ssh ubuntu@18.197.20.177 -p 2200 -i /c/Users/Naqiib/.ssh/id_rsa

	Then test URL http://somalishare.com/

	User grader
	password:12345

3. Change SSH port from 22 to 2200
	Edit /etc/ssh/sshd_config file by sudo nano /etc/ssh/sshd_config

	Change port from 22 to 2200

	Save the change by Control + X and exit from nano with Y

	Restart SSH with sudo service ssh restart

4. Set up Uncomplicated Fire Wall (UFW)
	Configure UDW to allow only incoming request from port2200(SSH)

	sudo ufw status -- utf should be inactive

	sudo ufw default deny incoming -- deny all incoming requests

	sudo ufw default deny outgoing-- allow all outgoing requests

	sudo ufw allow 2200/tcp -- allow incoming ssh request

	sudo ufw deny 22 -- deny incoming request for port 22

	sudo ufw enable -- enable ufw

	sudo ufw status -- check current status of ufw

	Go to AWS page and set up relevant ports from networking tab.

5. Create a new user called grader and give an access
	Run sudo adduser grader to create a new user called grader

	Create a new directory in sudoer directory with sudo nano /etc/sudoers.d/grader

	Add grader ALL=(ALL:ALL) ALL in nano editor

	Run sudo nano /etc/hosts

	Set SSH keys for grader user with ssh-keygen in your local machine.

	Copy the generated SSH to a virtual environment.

	Run the following command in your virtual environment.

	su - grader

	mkdir .ssh

	touch .ssh/authorized_keys

	nano .ssh/authorized_keys and copy your generated SSH key here.

	Reload SSH with service ssh restart

	Then now you can login grader user.

	Disable rootlogin.

	Open /etc/ssh/sshd_config and find PermitRootLogin and change it to no.

6. Update all packages
	Run sudo apt-get update and sudo apt-get upgrade

7. Set up local time zone
	Run sudo dpkg-reconfigure tzdata and choose UTC

8. Install Apache application and wsgi module
	Run sudo apt-get install apache2 to install apache

	Run sudo apt-get install python-setuptools libapache2-mod-wsgi to install mod-wsgi module

	Start the server sudo service apache2 start

9. Install git
	Run sudo apt-get install git

	Configure your username and email. git config --global user.name <username> and git config --global user.email <email>

10. Clone the project
	Run cd /var/www and sudo mkdir MotorsApp

	Change the owner to grader sudo chown -R grader:grader MotorsApp

	Run sudo chmod MotorsApp to give a permission to clone the project.

	Switch to the MotorsApp directory and clone the MotorsApp project.

	cd MotorsApp and git clone https://github.com/AyanleJunior/MotorsApp.git


	Run sudo nano MotorsApp.wsgi and add the following code.

	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0, "/var/www/MotorsApp/")

	from MotorsApp import app as application
	application.secret_key = 'secret'
	Modify filenames to deploy on AWS.

	Rename catalog.py to __init__.py

	mv catalog.py __init__.py

11. Install virtual environment and Flask framework
	Install pip, sudo apt-get install python-pip

	Run sudo apt-get install python-virtualenv to install virtual environment

	Create a new virtuall environment with sudo virtualenv venv and activate it source venv/bin/activate

	Change permissions to the viertual environment folder sudo chmod -R 777 venv

	Install Flask pip install Flask and dependencies pip install bleach httplib2 request oauth2client sqlalchemy python-psycopg2.

12. Configure Apache
	Create a config file sudo nano /etc/apache2/sites-available/MotorsApp.conf

	Paste the following code

	<VirtualHost *:80>
		ServerName 18.197.20.177
		ServerAdmin Naqiib122@gmail.com
		WSGIScriptAlias / /var/www/MotorsApp/MotorsApp.wsgi
		<Directory /var/www/MotorsApp/MotorsApp/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/MotorsApp/MotorsApp/static
		<Directory /var/www/MotorsApp/MotorsApp/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>
	Enable the new virtual host sudo a2ensite MotorsApp

13. Install and configure PostgressSQL
	Run sudo apt-get install PostgreSQL

	Check if no remote connections are allowed with sudo vim /etc/postgresql/9.3/main/pg_hba.conf

	Login to postgress sudo su - postgres and psql

	Create a new user CREATE USER MotorsApp WITH PASSWORD 'password'

	Create a DB called 'MotorsApp' with ALTER USER MotorsApp CREATEDB and CREATE DATABASE MotorsApp WITH OWNER MotorsApp

	Connect to the DB with \c MotorsApp

	Revoke all rights REVOKE ALL ON SCHEMA public FROM public

	Change a grand from public to MotorsApp GRANT ALL ON SCHEMA public TO MotorsApp

	Logout from postgress and return to the grader user \q and exit

	Change the engine inside Flask application.

	engine = create_engine('sqlite:///catalog.db?check_same_thread=False')

	Set up the DB with python /var/www/MotorsApp/item-MotorsApp-udacity/database_setup.py

14. Restart Apache
	Run sudo service apache2 restart and check http://18.197.20.177/
	
15. 15. Third-party resources
	https://help.ubuntu.com
	https://www.postgresql.org/docs/current/static/sql-createuser.html
	http://flask.pocoo.org/docs/0.12/
	https://www.ostechnix.com/how-to-change-apache-ftp-and-ssh-default-port-to-a-custom-port-part-1/
 