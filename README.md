#P5-Linux Server Configuration

* Launch your Virtual Machine.
    * Navigate to  [link](https://www.udacity.com/account#!/development_environment) and Create Development Environment.
    * Download private key
    * Move the private key file to the folder ~/.ssh. 
    * Set privileges, type in terminal: `chmod 600 ~/.ssh/udacity_key.rsa`
    * To connect type in terminal, type: `ssh -i ~/.ssh/udacity_key.rsa root@52.36.29.37`

* Update installed packages
    * Updating Available Package Lists:`apt-get update`
    * Upgrading installed Packages:`apt-get upgrade` 

* Add user named grader with password grader
    * `adduser grader`

* Give to grader sudo permission
    * In /etc/sudoers.d
    * `nano grader`
    * into "grader" file add `grader ALL=(ALL) NOPASSWD:ALL`  and save

* Set the privileges on this file so that only the owner (root) and the group (root) can read the file.
	 * 	 `chmod 440 grader`

* Generate the ssh keys on my local machine (I've a mac!) to enable key-based authentication and put it on server
 	 * 	 `ssh-keygen -t rsa` name of file UdacityCourse_rsa, no password
 	 * 	  `cat UdacityCourse_rsa.pub` copy output, and copy text in a file on server named authorized_keys with the following comands:

	* 	 `su grader`
	* 	 `cd ~`
	* 	 `mkdir .ssh`
	* 	 `cd .ssh`
	* 	 `nano authorized_keys`

* Set privileges to access only for 'grader' user and on the .ssh folder for only the 'grader' user.

	* 	 `chmod 644 authorized_keys`
	* 	 `cd ..`
	* 	 `chmod 700 .ssh`
* Exit out of the grader login again by typing:
	* 	 `exit`
* Disable login via ssh as root

	* 	 `cd /etc`
	* 	 `visudo`
	* 	 comment the line `#root   ALL=(ALL:ALL) ALL`
 	 
*  Change the SSH port from 22 to 2200
	* `nano /etc/ssh/sshd_config`
	* Change this line:
	* change `port 22` to `port 2200` and save
   * restart ssh service `service ssh reload`



* Configure the Uncomplicated Firewall (UFW) to only allow  incoming connections for SSH (port 2200), HTTP (port 80),  and NTP (port 123)
    * Check if UFW status is inactive with `ufw status`
    * Deny all incoming by default `ufw default deny incoming`
    * Allow outgoing by default `ufw default allow outgoing`
    * Allow SSH on port 2200 `ufw allow 2200/tcp`
    * Allow HTTP on port 80 `ufw allow 80/tcp`
    * Allow NTP on port 123 `ufw allow 123/udp`
    * Turn on firewall`ufw enable`


* Configure the local timezone to UTC
    * in terminal `dpkg-reconfigure tzdata` scroll to the bottom of the Continents list and select "None of the above"; in the second list, select UTC

* Install and configure Apache to serve a Python mod_wsgi application
    * `apt-get install apache2`
    * install mod_wsgi: `apt-get install libapache2-mod-wsgi python-dev`
    * To enable mod_wsgi, run the following command:
	* `a2enmod wsgi`
* Configure Apache2 to handle WSGI module
	* `nano /etc/apache2/sites-enabled/000-default.conf`
	
	*  ```
		<VirtualHost *:80>52.36.29.37
        ServerName http://ec2-52.36.29.37.us-west-2.compute.amazonaws.com/
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
        WSGIScriptAlias / /var/www/catalog.wsgi
        <Directory /var/www/html/>
                Order deny,allow
                Allow from all
        </Directory>

        Alias /secret "/var/www/html/P3_Catalog/secret"

        <Directory /var/www/html/P3_Catalog/secret/>
                Order allow,deny
                Allow from all
        </Directory>
</VirtualHost>

   * Restart apache service
   * `sudo service apache2 restart`

  * Create catalog.wsgi
	* `nano /var/www/catalog.wsgi`
	* 
	``` 
		#!/usr/bin/python
		import sys
		import logging
		logging.basicConfig(stream=sys.stderr)
		sys.path.insert(0,"/var/www/html/")

		from P3_Catalog import app as application
		application.secret_key = 'Add your secret key'
	```

  * install postgresql
   * `apt-get install postgresql`
   * `apt-get install python-psycopg2`


	* Login with user "postgres"
     * `su - postgres`
     * `psql`
   * Create user "catalog"
     * `CREATE USER catalog WITH PASSWORD 'catalog';`
   * Create database "catalog"
     * `CREATE DATABASE catalog WITH OWNER catalog;`
   * Grant privileges only to "catalog" user 
     * `\c catalog`
     * `REVOKE ALL ON SCHEMA public FROM public;`
     * `GRANT ALL ON SCHEMA public TO catalog;`
     * `\q`
     * `exit`
* Install GIT, clone repository movie directory and delete directory
	  * `apt-get install git`
	  * `cd /var/www/html`
	  * `git clone https://github.com/dmondello/P3-Catalog.git`
	  * Change name to prevent `mv P3-Catalog P3_Catalog`
     * Change in __init__.py , db_setup.py , moreteams.py PATH db engine
    * `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`

    * install pip
    * `apt-get install python-pip`
    * Install sqlalchemy
    * `pip install sqlalchemy`
    * Install Flask 
    * `pip install flask==0.9`
    * Install Flask-Login
    * `pip install Flask-Login==0.1.3`
    * Install oauth2client
    * `pip install oauth2client`
    * Install flask-seasurf
    * `pip install flask-seasurf`
    *  Install werkzeug
    * `pip install werkzeug==0.8.3`
    * Add IP to authorized JavaScript origins in credentials through Google Developers Console and change the value in client_secrets.json
    
* make .git inaccessible
    * in `/var/www/html/P3_Catalog/` create .htaccess file `nano .htaccess`
    * insert `RedirectMatch 404 /\.git`
* Install Glance to monitor system
	* `sudo pip install Glances`
* Install crontab
   * `sudo pip install crontab`
   * `crontab -e`
   *  Add this for update on saturday at 1 am
   * `0 1 * * 6 sudo apt-get update`
   * `5 1 * * 6 sudo apt-get upgrade` 
* Install "fail2ban"
   * `apt-get install fail2ban`
* Backup the config file
   * `cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
* Configure "Fail2Ban"
	* `nano /etc/fail2ban/jail.local`
* Stop and start Fail2ban
	* `sudo service fail2ban stop`
	* `sudo service fail2ban start`
 


REFERENCES

* https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04
* http://guide.debianizzati.org/index.php/Utilizzo_del_servizio_di_scheduling_Cron
* https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
* https://docs.google.com/document/d/1J0gpbuSlcFa2IQScrTIqI6o3dice-9T7v8EDNjJDfUI/pub?embedded=true
* https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps
* https://mauriziosiagri.wordpress.com/it/postgresql/postgresql-comandi-utili/
* https://www.digitalocean.com/community/tutorials/additional-recommended-steps-for-new-ubuntu-14-04-servers
* http://flask.pocoo.org/docs/0.10/deploying/mod_wsgi/#installing-mod-wsgi
