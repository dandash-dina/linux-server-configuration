# linux-server-configuration


## Table of Contents

* [Project Description](#describe what this project about)
* [Project Content](#contents)



## Project Description
Take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, securing it
from a number of attack vectors and installing/configuring web and database servers and deploy one of your existing web applications
onto it.

IP address: 18.234.74.208

Accessible SSH port: 2200

Application URL: http://18.234.74.208.xip.io/



## Project Steps

1. Get server

2. Secure server

  a. Change the SSH port from 22 to 2200

  b. Configure the Uncomplicated Firewall (UFW) as SSH (port 2200), HTTP (port 80), and NTP (port 123).

3. Create a grader user and give it sudo permission.

4. Prepare to  Deploy project

5. Deploy the project



### Get server:

1. Log in to Lightsail.
2. Create an instance.
3. Write down your public and private IP addresses.
4. Click on Account Page and click the download button to download your
   Private Key. It is a .pem file. save it and name it 'yourname.pem'
5. Click the Networking tab and find the Add another at the bottom. Add
   port 123 and 2200.
6. Now ssh to your server using:
  ```
  $ ssh -i yourname.pem ubuntu@your IP address
  ```



### Secure the server:

1. Update and upgrade installed packages
  ```
  $ sudo apt-get update
  $ sudo apt-get upgrade
  ```

2. Change the SSH port from 22 to 2200
  ```
  $ sudo nano /etc/ssh/sshd_config file
  ```
Change the port number from 22 to 2200. Save and exit.

Restart SSH:
  ```
  $ sudo service ssh restart
  ```

*To check port 2200 wether working or not by
 ssh -i yourname.pem -p 2200 ubuntu@your IP address
 **

3. Configure the default firewall for Ubuntu to only allow incoming connections for SSH (port 2200), HTTP (port 80),and NTP (port 123).
 sudo ufw status                  # The UFW should be inactive.
 sudo ufw default deny incoming   # Deny any incoming traffic.
 sudo ufw default allow outgoing  # Enable outgoing traffic.
 sudo ufw allow 2200/tcp          # Allow incoming tcp packets on port 2200.
 sudo ufw allow www               # Allow HTTP traffic in.
 sudo ufw allow 123/udp           # Allow incoming udp packets on port 123.
 sudo ufw deny 22                 # Deny tcp and udp packets on port 53.
 sudo ufw enable.                 # Turn UFW on
 sudo ufw status                  #Check the status of UFW

The output should be like this:

 Status: active
  To          | Action        | From
------------- | ------------- | -------------
2200/tcp      | ALLOW         | Anywhere
80/tcp        | ALLOW         | Anywhere
123/udp       | ALLOW         | Anywhere
22            | DENY          | Anywhere
2200/tcp (v6) | ALLOW         | Anywhere (v6)
80/tcp (v6)   | ALLOW         | Anywhere (v6)
123/udp (v6)  | ALLOW         | Anywhere (v6)
22 (v6)       | DENY          | Anywhere (v6)


### Create grader user

1. To add user run
  ```
  $ sudo adduser grader
  ```

2. Give grader the permission to sudo
  ```
  $ sudo nano /etc/sudoers.d/grader
  ```

write in it
grader    ALL=(ALL:ALL) ALL

Save and exit.


3. Create an SSH key pair for grader
  a. In another terminal run
    ```
    $ ssh-keygen
    ```
  copy the key

  b. Create .ssh folder and authorized_keys file and add the key you generated to it  
    ```
    $ mkdir /home/grader/.ssh
    $ cd /home/grader/.ssh
    $ touch authorized_keys
    $ sudo nano authorized_keys
    ```
    and paste the key in it
    save and exis.

  c. Change ownership
    ```
    $ chown grader.grader /home/grader/.ssh
    ```
    add 'grader' to sudo group
    ```
    $ usermod -a G sudo grader
    ```
    change permissions for .ssh folder and authorized_keys file
    ```
    $ chmod 0700 /home/grader/.ssh/
    $ chmod 644 authorized_keys
    ```

   Restart SSH
    ```
    $ sudo service ssh restart
    ```

   d. On the local machine, cheking if the grader account working or not
    ```
    $ ssh -i ~/.ssh/your_key.rsa -p 2200 grader@your IP address
    ```


### Prepare to deploy the project

1. Configure the local timezone to UTC
  ```
  $ sudo dpkg-reconfigure tzdata    # Choose time zone UTC.
  ```
  
2. Install and configure Apache to serve a Python mod_wsgi application.
  ```
  $ sudo apt-get install apache2
  $ sudo apt-get install libapache2-mod-wsgi python-dev
  $ sudo a2enmod wsgi
  $ sudo service apache2 restart
  ```
3. Install Git
  ```
  $ sudo apt-get install git
  ```
4. Install and configure PostgreSQL:
  a.run
    ```
    $ sudo apt-get install libpq-dev python-dev
    $ sudo apt-get install postgresql postgresql-contrib
    $ sudo su - postgres
    $ psql
    ```

  b. Now create a user to create and set up the database. here the name of database catalog with user catalog
    ```
    $ CREATE USER catalog WITH PASSWORD 'catalog';
    $ ALTER USER catalog CREATEDB;
    $ CREATE DATABASE catalog WITH OWNER catalog;
    $ \c catalog    #Connect to database
    $ REVOKE ALL ON SCHEMA public FROM public;
    $ GRANT ALL ON SCHEMA public TO catalog;
    $ \q    # Quit the postgrel command line and then exit   
    ```
  c.use
    ```
    $ sudo nano project.py
    $ sudo nano database_setup.py
    $ sudo nano lotsofmenus.py
    ```
    command to change all engine to
    ```
    engine = create_engine('postgresql://catalog:catalog@localhost/catalog change you engine in database_setup.py also.
    ```
  d. Run the
  ```
  $ python database_setup.py
  $ python lotsofmenus.py
  $ sudo service apache2 restart
  ```



### Deploy the catalog project

1. Clone and setup catalog project from the GitHub repository
  ```
  $ cd /var/www
  $ sudo mkdir catalog
  $ sudo chown -R grader:grader catalog
  $ cd catalog
  ```
Now we clone the project from Github
  ```
  $ git clone [your GitHub project link] catalog
  ```

2. Create a .wsgi file
  ```
  $  sudo nano catalog.wsgi
  ```
and add the following thing into this file
```
   import sys
   import logging
   logging.basicConfig(stream=sys.stderr)
   sys.path.insert(0, "/var/www/catalog/")

   from catalog import app as application
   application.secret_key = 'supersecretkey'
```

3. Rename the project.py file to __init__.py using
  ```
  mv project.py __init__.py
  ```

 To change the path of client_secrets.json to  /var/www/catalog/catalog/client_secrets.json run
 ```
 $ sudo nano __init__.py
 ```

4. Install the virtual environment and dependencies
  ```
  $ sudo pip install virtualenv
  $ sudo virtualenv venv
  $ sudo chown -R grader:grader venv
  $ source venv/bin/activate
  $ sudo chmod -R 777 venv
  ```
You should see a (venv) appears before your username in the command line.

Now we need to install the Flask and other packages needed for this application
  ```
  $ sudo apt-get install python-pip (If needed)
  $ sudo pip install flask
  $ sudo pip install httplib2
  $ sudo pip install oauth2client
  $ sudo pip install sqlalchemy
  $ sudo pip install psycopg2  # sometimes it asks to install psycopg2-binary. You'll find this while executing your program
  $ sudo pip install requests
  ```

5. and change the host to your Amazon Lightsail public IP address and port to 80 and change the last line of your program to this
  ```
  $ sudo nano __init.py
  ```
change app.run(host='0.0.0.0', port=5000)
to     app.run()

6. Set up and enable a virtual host
 ```
 $ sudo nano /etc/apache2/sites-available/catalog.conf
 ```
Paste this code:
```
<VirtualHost *:80>
        ServerName [YOUR PUBLIC IP ADDRESS]
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

6. To Enable and Disable default Apache2
  ```
  $ sudo a2dissite 000-default.conf
  $ sudo a2ensite catalog
  $ sudo service ssh reload
  $ sudo service apache2 reload
  $ sudo service apache2 restart
  ```
 7. Launch the Web Application
  Open your browser to http://18.234.74.208.xip.io/

