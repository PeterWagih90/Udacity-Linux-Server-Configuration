# Udacity Linux Server Configuration

> Peter Wagih

### IP and URL:
> Public IP: 18.195.32.138  
> Host name: ec2-18-195-32-138.eu-central-1.compute.amazonaws.com

[LIVE DEMO](http://ec2-18-195-32-138.eu-central-1.compute.amazonaws.com) (This demo may not be available after while use it as reference only)

### Software:
* Openssh Server
* Apache2
* PostgreSQL
* GIT
* mod_wsgi
* Python
* virtualenv
* Flask
* requests  
* httplib2  
* sqlalchemy 
* psycopg2
* oauth2client 
* render_template
* sqlalchemy_utils  
* redirect

### How To:  
#### Amazon Lightsail
1. Create Lightsail account and new Instance
2. Connect using SSH
3. Download private key
4. In the Networking tab, add two new custom ports - 123 and 2200
#### Server configuration
5. Place private key in .ssh
6. `$ chmod 600 ~/.ssh/LightsailDefaultPrivateKey-us-east-2.pem`
7. `$ ssh -i ~/.ssh/LightsailDefaultPrivateKey-us-east-2.pem ubuntu@18.195.32.138 `
#### Create new account grader with password 852456
8. `$ sudo su -`
9. `$ sudo nano /etc/sudoers.d/grader`
    Add `grader ALL=(ALL:ALL) ALL`
10. `$ sudo nano /etc/hosts`
    Under `127.0.1.1:localhost` add `127.0.1.1 ip-18-195-32-138`
#### Install updates and finger package
11. `$ sudo apt-get update`
    `$ sudo apt-get upgrade`
    `$ sudo apt-get install finger`
#### Keygen
12. In a new terminal, `$ ssh-keygen -f ~/.ssh/grader.rsa with password 852456
13. `$ cat ~/.ssh/grader.pub`
14. In the original terminal, `$ cd /home/grader`
15. `$ mkdir .ssh`
16. `$ touch .ssh/authorized_keys`
17. `$ nano .ssh/authorized_keys`
18. Permissions:
    `$ sudo chmod 700 /home/grader/.ssh`
    `$ sudo chmod 644 /home/grader/.ssh/authorized_keys`
19. `$ sudo chown -R grader:grader /home/grader/.ssh`
20. `$ sudo service ssh restart`
21. To disconnect:
    `$ ~.`
22. `$ ssh -i ~/.ssh/udacity_key.rsa grader@18.195.32.138`
#### Enforce key based authentication
23. `$ sudo nano /etc/ssh/sshd_config`
24. Find the PasswordAuthentication line and change text after to `no`
25. `$ sudo service ssh restart`
#### Change port
26. `$ sudo nano /etc/ssh/sshd_config`
27. Find the Port line and change `22` to `2200`
28. `$ sudo service ssh restart`
29. `$ ~.`
30. `$ ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@18.195.32.138`
#### Disable root login
31. `$ sudo nano /etc/ssh/sshd_config`
32. Find the PermitRootLogin line and edit to `no`
33. `$ sudo service ssh restart`
#### Configure UFW
34. `$ sudo ufw allow 2200/tcp`
    `$ sudo ufw allow 80/tcp`
    `$ sudo ufw allow 123/udp`
    `$ sudo ufw enable`
#### Install Apache and GIT
35. `$ sudo apt-get install apache2`
    `$ sudo apt-get install libapache2-mod-wsgi python-dev`
    `$ sudo apt-get install git`
#### Enable mod_wsgi
36. `$ sudo a2enmod wsgi`
    `$ sudo service apache2 start`
#### Setup Folders
37. `$ cd /var/www`
    `$ sudo mkdir catalog`
    `$ sudo chown -R grader:grader catalog`
    `$ cd catalog`
#### Clone Catalog Project
38. `$ git clone https://github.com/PeterWagih90/BookCatalogApp.git catalog`
#### Create .wsgi file
39. `$sudo nano catalog.wsgi`
```
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/catalog/")

    from catalog import app as application
    application.secret_key = 'super_secret_key'
```
40. Rename the application.py to __init__.py
#### Virtual Machine
41. `$ sudo pip install virtualenv`
    `$ sudo virtualenv venv`
    `$ source venv/bin/activate`
    `$ sudo chmod -R 777 venv`
#### Install flask and other packages
42. `$ sudo apt-get -H install python-pip`  
    `$ sudo pip -h install Flask`  
    `$ sudo pip -h install Requests`  
    `$ sudo pip -h install httplib2`  
    `$ sudo pip -h install sqlalchemy`  
    `$ sudo pip -h install psycopg2`  
    `$ sudo pip -h install oauth2client`  
    `$ sudo pip -h install render_template`  
    `$ sudo pip -h install sqlalchemy_utils`  
    `$ sudo pip -h install redirect`  
43. `$nano __init__.py`
    Change the client_secrets.json lines to /var/www/catalog/catalog/client_secrets.json
44. Change the host to 18.195.32.138 and port to 80
#### Configure virtual host

45. `$ sudo nano /etc/apache2/sites-available/catalog.conf`
```
    <VirtualHost *:80>
    ServerName [18.195.32.138]
    ServerAlias [ec2-18-195-32-138.eu-central-1.compute.amazonaws.com]
    ServerAdmin admin@18.195.32.138
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
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
```
 
#### Database
46. `$ sudo apt-get install libpq-dev python-dev`  
    `$ sudo apt-get install postgresql postgresql-contrib`  
    `$ sudo su - postgres`  
    `$ psql`  
47. `$ CREATE USER catalog WITH PASSWORD 'password';`
    `$ ALTER USER catalog CREATEDB;`
    `$ CREATE DATABASE catalog WITH OWNER catalog;`
    Connect to database `$ \c catalog`
    `$ REVOKE ALL ON SCHEMA public FROM public;`
    `$ GRANT ALL ON SCHEMA public TO catalog;`
    Quit the postgres command line: `$ \q` and then `$ exit`
48. `$ nano __init__.py`
     Edit database_setup.py, and __init__.py and alotofbookstoadd.py files to change the database engine from `sqlite://catalog.db` to                   `postgresql://catalog:password@localhost/catalog`
49. Add `ec2-18-195-32-138.eu-central-1.compute.amazonaws.com` to Authorized JavaScript Origins and Authorised redirect URIs on Google Developer Console.
50. `$ sudo service apache2 restart`

### References:
https://github.com/callforsky/udacity-linux-configuration  
https://github.com/mulligan121/Udacity-Linux-Configuration

### Screenshots:

![](screenshots/site1.png)
![](screenshots/site2.png)
![](screenshots/site3.png)
![](screenshots/site4.png)
![](screenshots/site5.png)
