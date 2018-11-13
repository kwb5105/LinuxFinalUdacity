# Kyle's Linux server configuration

Linux Server setup for Fullstack Nanodegree project. 

## Logging into the server
Instructions for signing into the linux server with "grader" user access and a defined list of installs that are currently making the system operate.

- URL: http://catalog.34.229.24.54.xip.io/
- ServerName: catalog.34.229.24.54
- ServerAlias: catalog.34.229.24.54.xip.io
- IP Address: 34.229.24.54

## log into server as grader
1. Copy private key provide in the SSH folder of github
2. Clone file onto local computer
3. open up terminal 
4. Run the following command
    `ssh grader@34.229.24.54 -p 2200 -i ~/(file address of private key)`
5. Passphase for key is `password`

## Server configurations

1. Installs and updates i've ran.
   - `sudo apt-get update`
   - `sudo apt-get upgrade`
   - `sudo apt-get install finger`
   - `sudo apt-get install apache2`
   - `sudo apt-get install postgresql`
   - `sudo apt-get install libapache2-mod-wsgi`
   - `sudo apt autoremove`
   - `sudo apt-get install git`
   - `sudo apt-get install python-pip`
   - `sudo pip install --upgrade pip`
   - `sudo pip install flask packaging oauth2client redis passlib flask-httpauth`
   - `sudo pip install Flask`
   - `sudo pip install sqlalchemy flask-sqlalchemy psycopg2-binary bleach requests`
   - `sudo apt-get install postgresql `
   - `sudo pip install --upgrade pip`
   - `sudo pip install virtualenv`
   - `sudo apt-get install libpq-dev python-dev`
    
2. Set up security settings
   - `sudo nano /etc/ssh/sshd_config` 
    change port from 22 to 2200
    set PermitRootLogin from prohibit-password to No


3. Set up the firewall 
   - `sudo service ssh restart`
   - `sudo ufw allow 2200/tcp`
   - `sudo ufw allow 80/tcp`
   - `sudo ufw allow 123/udp`
   - `sudo ufw default deny incoming`
   - `sudo ufw default allow outgoing`
   - `sudo ufw allow ssh`
   - `sudo ufw allow www`
   - `sudo ufw allow ntp`
   - `sudo ufw allow https`
   - `sudo ufw enable`
   - `sudo ufw status`

4. Set up the time zone
   - `sudo dpkg-reconfigure tzdata (Select UTC)`
   - `sudo reboot`

5. Set up the 'Grader" user and provided sudo access
   - `sudo adduser grader`
    Make a grader file within sudoers
   - `sudo nano /etc/sudoers.d/grader.` 
    provide all access:
   - `grader ALL=(ALL:ALL) NOPASSWD:ALL`

6. Setting Up SSH for grader.
    Make sure to be logged in as grader:
   - `mkdir .ssh`
   - `sudo nano .ssh/authorized_keys`
    Within file, add in the public key
    sudo chmod 700 .ssh
    sudo chmod 644 .ssh/authorized_keys
    Once set up, need to update the sshd_config:
   - `sudo nano /etc/ssh/sshd_config` 
    Change PasswordAuthentication No
   - `sudo service ssh restart`

7. Set up the database
    Create postgres database.
    create postgres user
    Provide access to user for that database schema

8. Clone catalog project repo from github
   - `cd /var/www`
   - `sudo mkdir catalog`
   - `cd catalog`
   - `git clone https://github.com/kwb5105/fullstack-nanodegree-vm.git catalog`
   - `cd catalog`
    Change name of catalog.py to __init__.py
   - `mv catalog.py __init.py__`

9. Change existing sqlite "engine" creates to postgres with user and database along with other .py file updates
   update this line in the following files:
   *engine = create_engine('postgresql://postgres:password@localhost/catalog')
  - `sudo nano database_setup.py`
  - `sudo nano loadCatalogItems.py`
  - `sudo nano __init__.py`
   within the __init__.py file also update the Client ID folder location:
  - `CLIENT_ID = json.loads(open('/var/www/catalog/catalog/vagrant/catalog/client_secrets.json', 'r').read())['web']['client_id']`
   *Change app.run(host='0.0.0.0', port=5000) >> app.run()


10. Install virtual environment within the catalog folder
   - `sudo virtualenv venv`
   - `source venv/bin/activate`
   - `python -m venv env`
    Install requirements so i can run database.py and loadCatalogItems.py
   - `sudo python database_setup.py`
   - `sudo python loadCatalogItems.py`
   - `sudo a2ensite catalog`
    exit out of virtual machine
    
    
12. Create catalog.wsgi file
   - `sudo nano /var/www/catalog/catalog/vagrant/catalog/catalog.wsgi`

    Details:
    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/catalog/catalog/vagrant/")

    from catalog import app as application
    application.secret_key = 'kyles_secret_key''

13. Update apache virtual host data file to reflex folder location and IP information
   - `sudo nano /etc/apache2/sites-available/catalog.conf`
    
    Details:
    <VirtualHost *:80>
        ServerName catalog.34.229.24.54
        ServerAlias catalog.34.229.24.54.xip.io
        ServerAdmin admin@34.229.24.54
        WSGIScriptAlias / /var/www/catalog/catalog/vagrant/catalog/catalog.wsgi
        <Directory /var/www/catalog/catalog/>
        Order allow,deny
        Allow from all
        </Directory>
        Alias /static /var/www/catalog/catalog/vagrant/catalog/static
        <Directory /var/www/catalog/catalog/vagrant/catalog/static/>
        Order allow,deny
        Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>


### Note, I have referenced the github readme file of Angela Carpio while setting up and troubleshooting my application. ###
- https://github.com/bad2thuhbone/Linux-Server-Project-6/blob/master/README.md 
