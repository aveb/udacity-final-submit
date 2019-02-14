# Deploying catalog app on aws linux server

This README details the steps I took to set up, configure and deploy the ```Catalog App``` on a linux server hosted by AWS.

The Catalog app is a simple Flask app that allows visitors to view categorized lists and descriptions of related items.  Authorized users can create and delete their respective items.

Visit the app by going to: [http://54.144.201.169/](http://54.144.201.169/)

## Start and configure AWS Lightsail Instance

1. Create a developer's account with Amazon Web Services at [https://aws.amazon.com/](https://aws.amazon.com/)
2. Create a lightsail instance.  I chose ```OS Only``` and ```Ubuntu 16.04 for my blueprint.
3. Connect via SSH and update server with ```$ sudo apt-get update``` and ```$ sudo apt-get upgrade```
4. Change timezone to UTC:
        
        $ sudo dpkg-reconfigure tzdata

Resources used:
- Udacity Project ```Getting started with lightsail```
- [https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt](https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt)
        

## Create new user: ```grader```

1. Create new user with: ```$ sudo adduser grader```
2. Give ```grader``` sudo permissions with: ```$ sudo usermod -aG sudo username```

Resources used:
- Udacity Deploying to Linux Servers: Lesson 2
- [https://linuxize.com/post/how-to-create-a-sudo-user-on-ubuntu/](https://linuxize.com/post/how-to-create-a-sudo-user-on-ubuntu/)

## Generating and adding key-pair authentication for ```grader```

1. From your local terminal, run ```ssh-keygen``` to create a private and public key pair.  A ```graderKey2``` and ```graderKey2.pub``` file were created in my local machine's user' .ssh directory.
2. Back on your AWS ssh connection, change to user ```grader``` with the command: ```$ su - grader```
3. create ```.ssh``` directoring using: ```$ mkdir .ssh```
4. create ```authorized_keys``` file with ```$ sudo nano .ssh/authorized_keys```
5. paste the public key you generated on your local machine in ```authorized_keys```
6. give your ```.ssh``` directory and ```authorized_keys``` file the proper permissions with:

        $ chmod 700 .ssh
        $ chmod 644 .ssh/authorized_keys
        
7. you can now connect to your server via ssh on your local terminal with:

        $ ssh grader@54.144.201.169 -i ~/.ssh/graderKey2

8. make sure key baised authentication is required by checking ```passwordAuthentication no``` in:

        $ sudo nano /etc/ssh/sshd_config

9. Restart the ssh if any changes were made with: ```$ sudo service ssh restart```

Resources used:
- Udacity Deploying to Linux Servers: Lesson 2

## Changing Port and Configuring Firewall

1. Change SSH port to 2200 by running ```$ sudo nano /etc/ssh/sshd_config``` and changing port from 22 to 2200.
2. From your lightsail instance console, find ```Networking``` and add ```port 2200``` and ```port 123```
3. Restart your SSH service with ```$ sudo service ssh restart```.  You will now need to log out and back into SSH using ```$ ssh grader@54.144.201.169 -i ~/.ssh/graderKey2 -p 2200```
4. Configure and enable your firewall by allowing the following:

        $ sudo ufw allow 2200/tcp
        $ sudo ufw allow 123/udp
        $ sudo ufw allow 80/tcp
        $ sudo ufw enable

Resources used:
- Udacity Deploying to Linux Servers: Lesson 2
- Udacity Student Hub Project: Linux Server Configuration

## Installing Apache2 and importing project from git

1. Install apache2 using ```$ sudo apt-get install apache2```
2. Visit [54.144.201.169](54.144.201.169) to make sure apache is working.
3. Install mod-wsgi with ```$ sudo apt-get install libapache2-mod-wsgi python-dev```
4. Enable mod-wsgi with ```$ sudo a2enmod wsgi```
5. run ```$ cd /var/www``` and create catalog directory with ```$ mkdir catalog```
6. install and initial ```git``` and clone the ```catalog``` git repository.
7. cd into ```catalog``` and you should have all of the files for your application

Resources used:
- [https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

## Installing virtual environment and updating project dependences

1. make sure pip is installed with ```$ sudo apt-get install python-pip```
2. Install virtual environment with ```$ sudo pip install virtualenv```
3. create your virtual environment with ```$ sudo virtualenv venv``` and activate it with ```$ source venv/bin/activate```
4. install and update project dependences:

        $ sudo pip install Flask
        $ sudo pip install httplib2 request oauth2client sqlalchemy
        $ sudo pip install psycopg2-binary
        $ sudo pip install --upgrade oauth2client

Resources used:
- [https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
- [https://github.com/CHBaker/Linux-Server-Configuration](https://github.com/CHBaker/Linux-Server-Configuration)
 
## Creating and configuring database

The original ```Catalog App```'s database needs to be changed to ```posgresql``` and engine paths need to be updated.

1. Edit the db_setup.py file with ```$ sudo nano db_setup.py``` and change create_engine path to ```postgresql://catalog:catalog@localhost/catalog```
2. Edit the add_categories.py file with ```$ sudo nano add_categories.py``` and change create_engine path to ```postgresql+psycopg2://catalog:catalog@localhost/catalog```
3. Edit ```__init__.py``` with ```$ sudo nano __init__.py``` and change engine path to ```postgresql+psycopg2://catalog:catalog@localhost/catalog```
4. Wile editing ```__init__.py```, also update path for client_secrets.json to ```/var/www/catalog/catalog/client_secrets.json```
5. Install Posgresql with ```$ sudo apt-get install libpq-dev python-dev``` and ```$ sudo apt-get install postgresql postgresql-contrib```
6. Change to posgresql user and login with ```$ sudo su - postgres``` and ```$ psql```
7. Create user, set permissions and exit psql shell with:

        # CREATE USER catalog WITH PASSWORD 'catalog';
        # ALTER USER catalog CREATEDB;
        # CREATE DATABASE catalog WITH OWNER catalog;
        # \c catalog;
        $ exit

8. Initialize database with ```$ sudo python db_setup.py```
9. Add data to database with ```$ sudo python add_categories.py```
10. Make sure app will run with no errors by running: ```$ sudo python __init__.py```
11. Exit out of virtual environment with ```$ deactivate```

 Resources used:
 - [https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
 - [https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04)
 - [https://github.com/CHBaker/Linux-Server-Configuration](https://github.com/CHBaker/Linux-Server-Configuration)

## Configure and enable virtual host

1. Run ```$ sudo nano /etc/apache2/sites-available/catalog.conf```
2. Add the following into the file and save:

        <VirtualHost *:80>
                ServerName 54.144.201.169
                ServerAdmin admin@54.144.201.169
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
        </VirtualHost>
        
3. Enable virtual host with ```$ sudo a2ensite FlaskApp```
4. change directory and create wsgi file with ```$ cd /var/www/catalog``` and ```$ sudo nano catalog.wsgi```
5. Add the following code to ```catalog.wsgi```:

        #!/usr/bin/python
        import sys
        import logging
        logging.basicConfig(stream=sys.stderr)
        sys.path.insert(0,"/var/www/catalog/")

        from catalog import app as application
        application.secret_key = 'Add your secret key'
        
 6. Restart Apache with ```$ sudo service apache2 restart```
 
Resources used:
- [https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

## Accessing app

Direct the browser of your choice to [http://54.144.201.169/](http://54.144.201.169/)

*Please note that authentication is unavailable at this time as app has not been connected to a purchased domain name and google oauth2 no longer accepts .xip.io.*
