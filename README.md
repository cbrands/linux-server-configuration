# linux-server-configuration


# Update

## As I graduated I no longer need this server instance and I deleted it.


## How to use
You can visit the page at http://35.156.112.239/ note that google authentication does not work as I don't have a domain name and google authentication does not work with public ip adresses. But otherwise you can click through the application and see it work.

To log in server you need a key. Create a file ~/.ssh/grader, Open it with a texteditor and copy and paste the contents of "Notes to Reviewer" into the file. then save. The you can log infrom the terminal with the command
```
ssh -p 2200 -i ~/.ssh/grader grader@35.156.112.239
```
Note: these instruction represent how I log in from my mac. If you use Windows you need to do something similar with putty.

## Setup A secure server
### upgrading packages 
All packages were upgraded on 29th of august 2017 at 16:04 (Timezone westerneurope) with the following commands
```
sudo apt-get update
sudo apt-get upgrade
```
I got one remark that 
"A new version of /boot/grub/menu.lst is avalable, but the version installed currently has been locally modified. 

![warning screen](/screenshots/menulst.png "warning screen")

As I didn't know what amazon had locally configured I kept the original, which was default.

### ssh
* Use `sudo nano /etc/ssh/sshd_config` and then change Port 22 to Port 2200 , save & quit.
* Reload SSH using `sudo service ssh restart`.
Now the big orange button "connect using ssh" no longer works as it assumes port 22.
so I downloaded the private key and put it inside $HOME/.ssh. Now set permissions with

```
chmod 600 $HOME/.ssh/<your keypair file>
chmod 700 $HOME/.ssh
```
Now login with
```
ssh -p 2200 -i ~/.ssh/LightsailDefaultPrivateKey-eu-central-1.pem ubuntu@35.156.112.239
```

### Firewall
The firewall was configured by typing the following commands

```
# default block all incoming ports
sudo ufw default deny incoming
# default open all outgoing ports
sudo ufw default allow outgoing
# we had ssh configured on port 2200
sudo ufw allow 2200/tcp
# open http port
sudo ufw allow 80/tcp
# open ntp port
sudo ufw allow 123/udp
# enable firewall
sudo ufw enable
```
chaeck the firewall status by typing sudo ufw status

```
sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
123/udp                    ALLOW       Anywhere                  
2200/tcp (v6)              ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)             
123/udp (v6)               ALLOW       Anywhere (v6)    
```

### Configure Local timezone to UTC
Timezone can be set with `sudo dpkg-reconfigure tzdata`
* select 'None of the above'
* select 'UTC'
* select 'ok'

### Create user grader
```
sudo adduser grader
sudo nano /etc/sudoers.d/grader
```
This will open an empty file
type `grader ALL=(ALL:ALL) ALL`
save and exit

### setup ssh keys for grader
* On the local machine type `ssh-keygen -f ~/.ssh/grader`
* open the file ~/.ssh/grader.pub in a text editor
* On the server type
```
su grader
mkdir /home/grader/.ssh
nano /home/grader/.ssh/authorized_keys
```
* Copy the contents of ~/.ssh/grader.pub on the local machine to /home/grader/.ssh/authorized_keys on the server.
* set permissions on the server by typing the following commands
```
chmod 700 /home/grader/.ssh
chmod 644 /home/grader/.ssh/authorized_keys
sudo service ssh restart
exit  #to go back to user ubuntu
exit  #to logout
```
Now we can login using user grader
```
ssh -p 2200 -i ~/.ssh/grader grader@35.156.112.239
```

### disable root login and force key based authentication
```
sudo nano /etc/ssh/sshd_config

```
* set PasswordAuthentication to no 
* set PermitRootLogin to no
* save the file
* sudo service ssh restart

## Setup a webapplication server
### Apache
Type `sudo apt-get install apache2`. Now the default apache site can be admired at http://35.156.112.239/.

![apache screen](/screenshots/apache.png "apache screen")

### mod_wsgi
Type the following commands
```
sudo apt-get install libapache2-mod-wsgi-py3
sudo apache2ctl restart
```
### PostgreSQL
Install PostgreSQL by typing `sudo apt-get install postgresql`
Login as user "postgres" `sudo su - postgres`
Start the PostgreSQL shell `psql`
In the PostGreSQL shell type the following commands
```
CREATE DATABASE catalog;
CREATE USER catalog;
ALTER ROLE catalog WITH PASSWORD 'catalog';
GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
\q
```
Leave user postgres `exit`

### Get the source
The command `git --version` showed that the latest version of git was already installed
```
cd /var/www
git clone https://github.com/cbrands/ufsd-p4-petstore.git
sudo nano .htaccess
```
In this file add the following line
```
RedirectMatch 404 ufsd-p4-petstore/.git
```

### Adapt the source to the database
```
cd ufsd-p4-petstore/vagrant
sudo nano database_setup.py
```
replace
'sqlite:///petstore.db'
with
'postgresql://catalog:catalog@localhost/catalog' and add `import psycopg2` to the imports
save file
repeat for populate_database.py and catalog.py

### Prepare python
sudo apt-get install python3-pip
pip3 install psycopg2
pip3 install SQLAlchemy
pip3 install Flask
pip3 install oauth2client

### finish the application server
Create tables and populate the database with testdata.
```
python3 database_setup.py
python3 populate_database.py
```
Update apache configuration
```
sudo nano /etc/apache2/sites-enabled/000-default.conf
```
Inside the `<VirtualHost *:80>` tag paste `WSGIScriptAlias / /var/www/ufsd-p4-petstore/vagrant/mod_wsgi.wsgi`

Create a wsgi file
```
sudo nano /var/www/ufsd-p4-petstore/vagrant/mod_wsgi.wsgi
```
paste the following
```
#!/usr/bin/env python3
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/ufsd-p4-petstore/vagrant/")

from catalog import app as application
application.secret_key = 'super_secret_key'
```

restart apache with `sudo apache2ctl restart`

### And then it didn't work
Apparently some dependencies need to be installed using the ubuntu packagemanager in order to work with apache.
```
sudo apt-get install python3-flask
sudo apt-get install python3-sqlalchemy
sudo apt-get install python3-psycopg2
sudo apt-get install python3-oauth2client
```

## Yes it works
Time for a little dance. :-)

## And finally!
One more
```
sudo apt-get update
sudo apt-get upgrade
```

## Sources used
Most of the information I needed came from the udacity fullstack nanodegree videos.
Other sources.
* [Onetomarket](https://www.onetomarket.nl/blog/seo/301-redirect-tutorial/)
* [Amazon](https://forums.aws.amazon.com/thread.jspa?threadID=162514)
* [Ubuntu](https://askubuntu.com/questions/386928/default-permissions-for-var-www)
* [Postgresql](https://wiki.postgresql.org/wiki/Psycopg2#Installation)
* [Stackoverflow](https://stackoverflow.com/questions/6587507/how-to-install-pip-with-python-3)
* [Tero Karvinen](http://terokarvinen.com/2016/deploy-flask-python3-on-apache2-ubuntu)
* [Stackoverflow](https://stackoverflow.com/questions/34785690/execute-hello-world-with-flask-importerror-no-module-named-flask)