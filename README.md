# Deploying catalog app on aws linux server

This README details the steps I took to set up, configure and deploy the ```Catalog App``` on a linux server hosted by AWS.

## Start and configure AWS Lightsail Instance

1. Create a developer's account with Amazon Web Services at [https://aws.amazon.com/](https://aws.amazon.com/)
2. Create a lightsail instance.  I chose ```OS Only``` and ```Ubuntu 16.04 for my blueprint.
3. Connect via SSH and update server with ```sudo apt-get update``` and ```sudo apt-get upgrade```
4. Change timezone to UTC:
        
        $ sudo dpkg-reconfigure tzdata

Resources used:
- Udacity Project ```Getting started with lightsail```
- [https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt](https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt)
        

## Create new user: ```grader```

1. Create new user with: ```$ sudo adduser grader```
2. Give ```grader``` sudo permissions with: ```usermod -aG sudo username```

Resources used:
- Udacity Deploying to Linux Servers: Lesson 2
- [https://linuxize.com/post/how-to-create-a-sudo-user-on-ubuntu/](https://linuxize.com/post/how-to-create-a-sudo-user-on-ubuntu/)

## Generating and adding key-pair authentication for ```grader```

1. From your local terminal, run ```ssh-keygen``` to create a private and public key pair.  A ```graderKey2``` and ```graderKey2.pub``` file were created in my local machine's user' .ssh directory.
2. Back on your AWS ssh connection, change to user ```grader``` with the command: ```su - grader```
3. create ```.ssh``` directoring using: ```mkdir .ssh```
4. create ```authorized_keys``` file with ```sudo nano .ssh/authorized_keys```
5. paste the public key you generated on your local machine in ```authorized_keys```
6. give your ```.ssh``` directory and ```authorized_keys``` file the proper permissions with:

        $ chmod 700 .ssh
        $ chmod 644 .ssh/authorized_keys
        
7. you can now connect to your server via ssh on your local terminal with:

        $ ssh grader@54.144.201.169 -i ~/.ssh/graderKey2

8. make sure key baised authentication is required by checking ```passwordAuthentication no``` in:

        $ sudo nano /etc/ssh/sshd_config

9. Restart the ssh if any changes were made with: ```sudo service ssh restart```

Resources used:
- Udacity Deploying to Linux Servers: Lesson 2

## Opening ```Catalog App```

Open your favorite browser and go to [http://localhost:5000/](http://localhost:5000/)

## Accessing ```Catalog App API```

Two API endpoints are available for developers:

1. Access JSON file of ```All Categories```.  Use this link: [http://localhost:5000/catalog/JSON/](http://localhost:5000/catalog/JSON/)
2. Access JSON file of ```All Items```.  Use this link: [http://localhost:5000/catalog/items/JSON/](http://localhost:5000/catalog/items/JSON/)
