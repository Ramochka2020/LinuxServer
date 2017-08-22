# Linux Server Configuration

This is a set of instructions on how to set up a Ubuntu Linux server to host my web application.


## Server details

The IP address is 34.213.165.152

The SSH port : 2200

The URL is: http://34.213.165.152 or http://ec2-34-213-165-152.compute-1.amazonaws.com/.

## Software to install during the configuration

- Apache2
- mod_wsgi
- PostgreSQL
- git
- pip
- httplib2
- Requests
- oauth2client
- SQLAlchemy
- Flask
- libpq-dev



## The Configuration steps

### Create an instance with Amazon Lightsail

1. Sign in to [Amazon Lightsail](https://amazonlightsail.com) using an Amazon Web Services account

2. Follow the 'Create an instance' link

3. Choose the 'OS Only' and 'Ubuntu 16.04 LTS' options

4. Choose a payment plan

5. Give the instance a unique name and click 'Create'

6. Wait for the instance to start up


### Connect to the my remote server from my terminal

1. Download the instance's default key form Amazon Lightsail Account page

2. Move the LightsailDefaultPrivateKey.pema file to your local directory ~/.ssh/

3. Change the directory permissions(write and read) chmod 700 ~/.ssh 

4. Login to your instance from your locla terminal 

 ..$ ssh -i ~/.ssh/LightsailDefaultPrivateKey.pema ubuntu@34.213.165.152

5. update currently installed packages by runing 
... $ sudo apt-get update 
... $ sudo apt-get upgrade

### Configure the firewall

1. Chang the SSH port from `22` to `2200` in the sshd_config file located at /etc/ssh

2. Restart SSH, by running $ sudo service ssh restart           

3. Allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

...$ sudo ufw allow 2200/tcp.

...$ sudo ufw allow 80/tcp.

...$ sudo ufw allow 123/udp.

...$ sudo ufw enable.

4. check the firewall status, Run $ sudo ufw status 

5. Update the external (Amazon Lightsail) firewall on the browser, change the firewall configuration to match the internal firewall settings above 

6. login to your instance

...ssh -i ~/.ssh/LightsailDefaultPrivateKey.pema.rsa -p 2200 ubuntu@34.213.165.152
    
### Create a new user named `grader`

1. Run $ sudo adduser grader and Fill out information 

2. switch to the `grader` user, run $ su - grader 

3. Give `grader` root privileges

...Edit the /etc/sudoers file, by running $ sudo visudo

......Search for a line that :`root  ALL=(ALL:ALL) ALL`

......Add the following line below it:`grader ALL=(ALL:ALL) ALL`

...Save and close the visudo file

4. Generate Key pair on my local machine using ssh-keygen and run it.

5. Login to virtual machine, Switch to `grader` user, and create a new directory. run $ mkdir .ssh

6. Run $ touch .ssh/authorized_keys to create the file 

7. Copy the contents of the authorized_keys file from your local machin, and paste them in the authorized_keys file at .ssh/ directory on the virtual machine

8. change the file permission so owner can read, write and execute Run $ chmod 700 .ssh 

9. Run $ chmod 644 .ssh/authorized_keys 

10. Log in as the grader using the following command:

...ssh -i ~/.ssh/grader_key -p 2200 grader@34.213.165.152

...pop-up window will ask for `grader`'s password


### Configure the local timezone to UTC

1. Run $ sudo dpkg-reconfigure tzdata, and follow the instructions (UTC is under the 'None of the above' category)

1. Test to make sure the timezone is configured correctly by running`date`

## Deploy My project

1. Install Apache, Run $ sudo apt-get install apache2 

2. Install the mod_wsgi package, to serve Flask applications, along with python-dev

...$ sudo apt-get install libapache2-mod-wsgi python-dev

...$ sudo service apache2 start

3. Install PostgreSQL, by running $ sudo apt-get install postgresql

...Connect as postgres user, by running $ sudo su - postgres 

...Connect to psql (the terminal for interacting with PostgreSQL) by running $ psql

...Create `catalog` user, by running CREATE USER catalog WITH PASSWORD 'password';

...Give the `catalog` user the ability to create databases # ALTER USER catalog CREATEDB;

...Create the 'catalog' database owned by catalog user, $ CREATE DATABASE catalog WITH OWNER catalog;

...Give user catalog permission to catalog database
......$ GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;

...Exit psql by running $ \q

...Switch back to the `grader` user, by running $ exit
	
4. Install git, Run $ sudo apt-get install git

5. Create a directory called 'FlaskApp' in the /var/www/ directory

6. clone the catalog project,run $ sudo git clone https://github.com/Ramochka2020/item-catalog.git

7. Rename 'item-catalog' directory to FlaskApp, $ sudo mv ./item-catalog/FlaskApp 

8. Move to /var/www/FaskApp/FlaskApp directory

9. Change the name of the app.py file to __init__.py, by running $ mv app.py __init__.py

10. Switch the database in the application from SQLite to PostgreSQL

...engine = create_engine('sqlite://catalog1.db') to 
   
...engine = create_engine('postgresql://catalog:password@localhost/catalog')

11. install pip,run $ sudo apt-get install python-pip

12.install the following dependenies 

...sudo pip install httplib2

...sudo pip install requests

...sudo pip install --upgrade oauth2client

...sudo pip install sqlalchemy

...sudo pip install flask

...sudo apt-get install libpq-dev

### Set up and enable a virtual host

1. Edite 000-default.conf fiel in the directory /etc/apache2/sites-available/ 

<VirtualHost *:80>
		ServerName mywebsite.com
		ServerAdmin admin@mywebsite.com
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

2. Enable the virtual host,run $ sudo a2ensite FlaskApp

...Run $ sudo service apache2 restart


### Create the catalog.wsgi File

1. create a file called catalog.wsgi in /var/www/FlaskApp

2. Add the following to the file:

    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/FlaskApp/")

    from FlaskApp import app as application
    application.secret_key = ''

3. Resart Apache: `sudo service apache2 restart`

4. Set up the database schema and populate the database

...1. While in the /var/www/FlaskApp/FlaskApp/ directory, run $ database_init.py

...2. Resart Apache again: $ sudo service apache2 restart

### Authenticate login through Google:

1. Add the complete file path for the client_secrets.json file in the __init__.py file

2. Change it from 'client_secrets.json' to '/var/www/FaskApp/FlaskApp/client_secrets.json' 

3. change the javascript_origins field to the IP address and AWS assigned URL of the host. 

"javascript_origins":["http://34.213.165.152", "http://ec2-34-213-165-152.us-west-2.compute.amazonaws.com"]

4. change the address in Google Developers Console -> API Manager -> Credentials, 
in the web client under "Authorized JavaScript origins".

5.launch the app, $ sudo python __init__.py and visit http://34.213.165.152


### Disable remote login of the root user

1. Edit the sshd_config and set PermitRootLogin to no and save the file
...$ sudo nano /etc/ssh/sshd_config

2. Restart ssh service
...$ sudo service ssh restart

### Sources

$ sudo tail -50 /var/log/apache2/error.log to see all the error in the server

Below is a list of sources I used to complete this project.

...[Udacity course about Linux and everything related to it]
(https://classroom.udacity.com/nanodegrees/nd004/parts/00413454014)

...[Udacity forums] (https://discussions.udacity.com/)

...<https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps>

...<https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04>
...<https://www.postgresql.org/docs/9.0/static/sql-createrole.html>

...Stackoverflow 
...<http://www.linfo.org/index.html>

...<https://httpd.apache.org/docs/2.4/>

...<https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux_OpenStack_Platform/2/html/Getting_Started_Guide/ch02s03.html>

...<https://askubuntu.com/questions/168280/how-do-i-grant-sudo-privileges-to-an-existing-user>

...<https://www.liquidweb.com/kb/how-to-add-a-user-and-grant-root-privileges-on-ubuntu-14-04/>

...<https://help.ubuntu.com/community/RootSudo#Re-disabling_your_root_account>

...<https://www.a2hosting.com/kb/getting-started-guide/accessing-your-account/disabling-ssh-logins-for-root>
