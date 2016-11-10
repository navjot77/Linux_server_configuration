# Linux_server_configuration
This project is about hosting a web application on virtual machine. This project includes steps to configure Linux distribution on virtual machine. I have used Amazon AWS instance to host the web application. The web application used here is project Item catalog developed using Flask framework and ORM, which was project 5 of Full Stack nanodegree. The project can be found on my github account. Below is screen shot of deployed project using aws instance. 

 ![](https://cloud.githubusercontent.com/assets/15641327/20162780/13ddb47e-a6ac-11e6-92c6-ead031ad5728.png)

This project is reachable under: http://ec2-35-162-67-243.us-west-2.compute.amazonaws.com/.
SSH port : 2200. Public-Ip-Address: 35.162.67.243
This AWS instance was provided free by Udacity. And it may expire soon. But one can follow same steps to configure own virtual machine. 
##ABOUT PROJECT
The Item Catalog project allows to edit/delete/update an item in particular category. The same framework can be used for creating restaurant menu, movie ticket application etc. For authorization, I have used oAuth2.0 authentication with facebook and google+.  The original project uses sqlite database which is achieved through sqlAlchemy. 
For deployment purpose, I will be using postgresql and apache2 web server. 

##STEPS
###1.	Launch your Virtual Machine with your Udacity account. 

1. Get new public ip address from Udacity Account and download the private key.
2. Move the private key into ~/.ssh folder. (if old key with same name exist, then rename or move key to other folder).
3. Change owner’s permission to 700 using
  `$ chmod 700 ~/.ssh/udacity_key.rsa`



###2.	SSH into your server
1.	Ssh into AWS instance:

  `$ ssh –i ~/.ssh/udacity_key.rsa root@public-ip-address`


###3.	Create a new user named grader
1. Now as we are login into remote machine i.e. the virtual machine that needs to be configured. Therefore we will be adding new user as: 
  `$ adduser grader`

###4.	Give the grader the permission to sudo
1. Type and enter command:
`$ visudo`
2.At the end of file, append following line:
`grader ALL=(ALL:ALL) ALL`
 Save changes and exit

###5.	Update all currently installed packages

1.	Get list of packages to be upgrader:
`$ sudo apt-get update`
2. Install all packages:
`$ sudo apt-get upgrade`

###6.	Change the SSH port from 22 to 2200

1.	By default, ssh is configured to port 22. One can change the port using:

	`$ vi /etc/ssh/sshd_config`
	1. Change port 22 to 2200
	2.	Change password Authentication to yes
	3.	Append line : ‘Allowusers grader’

2.	Restart ssh service:
`$ sudo service ssh restart`
3.	Now exit the remote connection using command ‘exit’
4.	Reconnect to remote machine on port ‘2200’ and using user ‘grader’ using
`$ ssh grader@public-ip-address –p 2200`
5.	Creating ssh key-pair
One have to generate key at local machine, so using command ‘exit’, one can exit remote machine. Now enter the following command to generate key-pair:
`$ ssh-keygen`
One can save key with any name, at any location. Also user can enter paraphrase for further security on prompt.  The above command will generate two files: one with .pub extension which is public file and needs to be placed at server, and other file is private file which is used for connection with server if key based authentication is enabled.
6.	Login to remote machine as grader and create .ssh directory at home.
`$ mkdir ~/.ssh`
Create a file name: authorized_keys in this folder and copy the content of public key in this file.
7. Now change the permission of the folder .ssh to 700 and .ssh/authorized_keys to 644.
8.	To enable key based authentication, open the file /etc/ssh/sshd_config, and change PasswordAuthentication from No to Yes.
9.	To simplify Login from local machine, one can edit file ~/.ssh/config and add following line :
`Host graderLogin
        HostName 35.162.67.243
        Port 2200
        User grader
        IdentityFile ~/.ssh/id_rsa`

Now to login into remote machine, one can simply type ‘ssh graderLogin’ and enter.

###7.	Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
1.	First, check ufw status and if it is enabled, then disable it.
`$ sudo ufw status`
2.	Deny All Incoming conenctions
`$ sudo ufw default deny incoming`
3.	Allow tcp at port 80, ntp at port 123.
`$ sudo ufw allow 2200/tcp
$ sudo ufw allow 80/tcp
$ sudo ufw allow 123/udp`
4.	Enable the firewall
`$ sudo ufw enable`

###8.	Configure the local timezone to UTC

Type the following command and then select the time zone.

`$ sudo dpkg-reconfigure tzdata`


###9.	Install and configure Apache to serve a Python mod_wsgi application

1.	Install Apache2 
`$ sudo apt-get install apache2`

To check its working or not, open the public-address-ip on web browser.
Apache just returns a file requested or the index.html file if no file is defined within the URL. 
2.	 To configure Apache to hand-off certain requests to an application handler - mod_wsgi. The first step in this process is to install mod_wsgi:

`$ sudo apt-get install libapache2-mod-wsgi python-dev `
3,	To configure Apache to handle requests using the WSGI module. You’ll do this by editing the /etc/apache2/sites-enabled/000-default.conf file. This file tells Apache how to respond to requests, where to find the files for a particular site and much more.
4.	Restart Apache: $ sudo apache2ctl restart

###10.	Install git and setup your project:
1.	Install git
`$ sudo apt-get install git`
2.	Set your name and email id:
`$ git config --global user.name "YOUR NAME"
$ git config --global user.email "YOUR EMAIL ADDRESS"`

3.	 Deploying Flask application on Ubuntu 
	1.Enable mod_wsgi
	`$ sudo a2enmod wsgi`
	2.	Create the web application
	$ cd /var/www
	$ sudo mkdir catalog
	$ cd catalog and $ sudo mkdir catalog
	$ cd catalog and $ sudo mkdir static templates
	$ sudo vi __init__.py
		`from flask import Flask  
		 app = Flask(__name__)  
		 @app.route("/")  
			def hello():  
					return "Check: Successful"  
			if __name__ == "__main__":  
					app.run() `
	3.	Install FlasK
	$ sudo apt-get install python-pip
	$ sudo pip install virtualenv
	$ sudo virtualenv venv
	$ sudo chmod -R 777 venv
	$ source venv/bin/activate
	$ pip install Flask
	$ pip install Flask-SQLAlchemy
	$ sudo apt-get install libpq-dev python-dev
	$ pip install psycopg2
	4.	Run the application and check on localhost 
	python __init__.py
	5.	`Deactivate`
	6.	Configure and Enable a New Virtual Host
	$ sudo vi /etc/apache2/sites-available/catalog.conf
		`<VirtualHost *:80>
				ServerName PUBLIC-IP-ADDRESS
				ServerAdmin admin@PUBLIC-IP-ADDRESS
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
		</VirtualHost>`

	7.	Enable virtual host
	`$ sudo a2ensite catalog`

	8.	Create the .wsgi File and Restart Apache
	$ cd /var/www/catalog and $ sudo vim catalog.wsgi
		`#!/usr/bin/python
		import sys
		import logging
		logging.basicConfig(stream=sys.stderr)
		sys.path.insert(0,"/var/www/catalog/")

		from catalog import app as application
		application.secret_key = 'Add your secret key'`

	9.	Restart Apache:
	`$ sudo service apache2 restart`

3.	Clone GitHub repository and make it web inaccessible
	1.	Clone project itemCatalog from github at localtion /var/www/catalog/catalog
	Git clone https://github.com/navjot77/Project5_udacity_flask/tree/master/project
	2.	Make github repo accessible
		1.	Create and open .htaccess file in directory /var/www/catalog.
		2.	Paste in the following:
		`RedirectMatch 404 /\.git`	
4. Install packages:
	1.	Activate virtual environment venv:
	$ source venv/bin/activate
	2. Install httplib2 module in venv:
	$ pip install httplib2
	3.	Install requests module in venv:
	$ pip install requests
	4.	Install oauth2client.client:
	$ sudo pip install --upgrade oauth2client
	5.	Install SQLAlchemy:
	$ pip install Flask-SQLAlchemy
	6.	Install the Python PostgreSQL adapter psycopg:
	$ pip install psycopg2



###11.	Install and configure PostgreSQL:
1.	Install PostgreSQL:
`$ sudo apt-get install postgresql postgresql-contrib`
2.	Open the database setup file and change the engine settings for postgresql:
`$ sudo vim database_setup.py
python engine = create_engine('postgresql://catalog:PW-FOR-DB@localhost/catalog')`
3.	Change the same line in application.py and lotsofmeny.py respectively
4.	Rename application.py:
`$ mv application.py __init__.py`
5.	Create needed linux user for psql:
`$ sudo adduser catalog`
6.	Change to default user postgres:
`$ sudo su - postgre`
7.	Connect to the system:
`$ psql`
8.	Add postgre user with password: 
	1.	Create user with LOGIN role and set a password:
	`# CREATE USER catalog WITH PASSWORD 'PW-FOR-DB'; (# stands for the command prompt in psql and end the command with ;)`
	2.	Allow the user to create database tables:
	`# ALTER USER catalog CREATEDB;`
9.	Create database:
`# CREATE DATABASE catalog WITH OWNER catalog;`
10.	Connect to the database catalog # \c catalog
11.	Revoke all rights:
`# REVOKE ALL ON SCHEMA public FROM public;`
12.	Grant only access to the catalog role:
`# GRANT ALL ON SCHEMA public TO catalog;`
13.	Exit out of PostgreSQl and the postgres user:
# \q, then $ exit
14.	Create postgreSQL database schema:
`$ python database_setup.py `
15.	Create dummy menu catalog descriptions:
`$ python lotsofmenu.py`

###12.	Run Application

	`$ sudo service apache2 restart`

###13.	    Get oAuth working

1. Get host name for your public-ip-address at  http://www.hcidata.info/host2ip.cgi 
2.	Open file /etc/apache2/sites-available/catalog.conf and add line below ServerAdmin:
ServerAlias HOSTNAME ec2-35-162-67-243.us-west-2.compute.amazonaws.com
3. Enable virtual host
`$ sudo a2ensite catalog`
4.Vist google/facebook developers page and add the url of web page to redirect URIs and authorized java script origins.

###14.	     References
1. https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps
2. http://askubuntu.com/questions/16650/create-a-new-ssh-user-on-ubuntu-server
3. https://help.ubuntu.com/community/UFW
4. https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
5. https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps
6. http://killtheyak.com/use-postgresql-with-django-flask/
7. http://blog.trackets.com/2013/08/19/postgresql-basics-by-example.html
8. http://stackoverflow.com/questions/10572498/importerror-no-module-named-sqlalchemy
9. http://stackoverflow.com/questions/28253681/you-need-to-install-postgresql-server-dev-x-y-for-building-a-server-side-extensi





