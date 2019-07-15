# ServerConfig
Graded project to set up a linux web server

### Server Details
**URL** https://lightsail.aws.amazon.com/ls/webapp/eu-west-1/instances/ColUbuntu/connect

**IP Address** 34.245.51.53
#### Summary Of Software Installed
* Apache web Server
* postgreSQL
* Flask
* SQLAlchemy
* mod_wsgi, pip, psycopg2
* Reinstalled python modules httplib2, requests due to import errors
* finger
* System packages updated

#### Summary Of Configurations Made
##### Create grader login
* Created a new user called grader and gave this this user sudo privileges by creating a _grader_ file in the _/etc/sudoers.d_ directory
* Generated a public / private key pair using the command ssh-keygen on my local computer
* Installed the public key on the server by copying the text from the local public key file into the _authorized_keys_ file.  Used the following commands in the grader home directory to create / access this file:
```
mkdir .ssh
cd .ssh
sudo nano authorized_keys
```

##### Disable Remote root Login / Enforce RSA Key Authentication / Change SSH Default Port
```
sudo nano /etc/ssh/sshd_config
```
* In this file changed _#PermitRootLogin prohibit-password_ to _#PermitRootLogin no_
* Also checked password authentication is set to _no_ here to ensure RSA key based authentication.  This was the case and seems to be the AWS Lightsail default.  Specifically parameters _PasswordAuthentication_ and _ChallengeResponseAuthentication_
* Changed Port from 22 to 2200 in sshd_config.  Also added _Custom_ Port 2200 to AWS firewall to accept connections. Then ran:
```
sudo service sshd reload
````

##### Firewall Activation
* Added added _Custom_ Port 123 to AWS firewall to accept NTP connections
* Ran the following code:
```
sudo ufw status
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow 123/udp
sudo ufw enable
sudo ufw status
```
* Originally mistakenly added port 123 as TCP but then added UDP port and denied TCP port 123.  This is visible in rules

* Deleted Port 22 from AWS Lightsail Firewall open connections

##### postgreSQL Configuration
* With the aim of replacing SQLite database in project 2 catalog application, logged onto postgres as user _postgres_
* Created a database called _league_ and a user called _www-data_, the default apache user,  with password, and granted all privileges on all database tables in the _league_ database to this user.

##### Serve web application _catalog_ created for project 2
 * Cloned repo https://github.com/colm29/catalog to /var/www/catalog (changed directory owner to ubuntu (default instance login) in order to clone to here)
 * Created _fb_clientsecrets.json_ and changed any references to this file in the _application.py_ file to absolute path where previously it was relative
 * Moved assignment of _app.secret_key_ to main code block as it was previously in the following block, where the condition would not be met in production:
 ```python
 if __name__ == '__main__':
    app.secret_key = 'super_secret_key'
    app.debug = True
    app.run(host='0.0.0.0', port=8000)
```
Secret key is now generated as follows:
```python
import os
app.secret_key = os.urandom(16)
```

* Replaced SQLite database connection lines of code via SQLAlchemy to postgres database, passing in username and password for _www-data_ user.  Affected files were _application.py_, _db_setup.py_ and _lotsofteams.py_
```python
engine = create_engine('postgresql://www-data:wwwdata2019@localhost/league')
```
* Ran _db_setup.py_ followed by _lotsofteams.py_ to create and then populate database
* In this application directory, _/var/www/catalog_, created _catalog.wsgi_ to specify the application path,flask application name and specific file location
 ```python
 import sys
 sys.path.insert(0,'/var/www/catalog')
from application import app as application
```

* Changed owner of /etc/apache2 to ubuntu in order to edit config file
* Created /etc/apache2/sites-available/catalog.conf:
```<VirtualHost *>
Servername example.com
WSGIScriptAlias / /var/www/catalog/catalog.wsgi
WSGIDaemonProcess hello
<Directory /var/www/catalog>
WSGIProcessGroup hello
WSGIApplicationGroup %{GLOBAL}
Order deny,allow
Allow from all
</Directory>
</VirtualHost>
```
* Disabled the default apache site, enabled my application's site and reloaded the apache service
```
sudo a2dissite 000-default.conf
sudo a2ensite myApp.conf
sudo service apache2 reload

##### 3rd Party Resources Used
* Official Flask docs, inc. _mod_wsgi, secret_key, packages_
  * https://flask.palletsprojects.com/en/1.0.x/
* Web App Deployment articles
  * https://umar-yusuf.blogspot.com/2018/02/deploying-python-flask-web-app-on.html
  * https://www.matthealy.com.au/blog/post/deploying-flask-to-amazon-web-services-ec2/
* Official Python, postgreSQL, Apache, Lightsail docs
* Stack Overflow and mediatemple
  * https://mediatemple.net/community/products/dv/204643810/how-do-i-disable-ssh-login-for-the-root-user
* Postgreguide.com and medium.com for user setup
  * http://postgresguide.com/setup/users.html
  * https://medium.com/coding-blocks/creating-user-database-and-adding-access-on-postgresql-8bfcd2f4a91e
