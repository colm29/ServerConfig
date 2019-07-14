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
##### postgreSQL Configuration
* With the aim of replacing SQLite database in project 2 catalog application, logged onto postgres as user _postgres_
* Created a database called _league_ and a user called _www-data_, the default apache user,  with password, and granted all privileges on all database tables in the _league_ database to this user.

##### Serve web application _catalog_ created for project 2
 * Cloned repo https://github.com/colm29/catalog to /var/www/catalog (changed directory owner to ubuntu (default instance login) in order to clone to here)
 * Created _fb_clientsecrets.json_ and changed application references to absolute path
 * Moved assignment of _app.secret_key_ to main code block as it was previously in the following block, where the condition would not be met in production:
 ```
 if __name__ == '__main__':
    app.secret_key = 'super_secret_key'
    app.debug = True
    app.run(host='0.0.0.0', port=8000)
```

* Replaced SQLite database connection lines of code via SQLAlchemy to postgres database, passing in username and password for _www-data_ user.  Affected files were _application.py_, _db_setup.py_ and _lotsofteams.py_
```
engine = create_engine('postgresql://www-data:wwwdata2019@localhost/league')
```
* In this directory created _catalog.wsgi_ to specify the application path,flask application name and specific file location
 ```
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
