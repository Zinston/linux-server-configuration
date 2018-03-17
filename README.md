# Linux Server Configuration

## IP Address
18.197.49.225

## SSH Port
2200

## Software installed
* Apache 2: `sudo apt-get install apache2`
* mod_wsgi: `sudo apt-get install libapache2-mod-wsgi`
* Postgres 9.5: `sudo apt-get install postgresql`
* Git: `sudo apt-get install git-all`
* Pip: `sudo apt-get install python-pip; sudo pip install --upgrade pip`

## Configuration changes made
* Upgraded the software: `sudo apt-get upgrade; sudo apt-get autoremove` (The following packages have been kept back: `linux-aws`, `linux-headers-aws`, `linux-image-aws`)
* Created a new user named `grader`: `sudo adduser grader`
* Gave `grader` `sudo` permissions: added a `grader` file to `/etc/sudoers.d` containing the line `grader ALL=(ALL) NOPASSWD:ALL`
* Installed a public key for `grader`: copied its content inside a new file at `.ssh/authorized_keys` while logged in as `grader` and set permissions like so: `chmod 700 .ssh; chmod 644 .ssh/authorized_keys`
* Setup and activate the firewall

| To                        | Action     | From          |
| -------------------------:|:----------:|:------------- |
| 22                        | DENY       | Anywhere      |
| 2200/tcp                  | ALLOW      | Anywhere      |
| 80/tcp                    | ALLOW      | Anywhere      |
| 123/tcp                   | ALLOW      | Anywhere      |
| 5432                      | ALLOW      | Anywhere      |
| 22 (v6)                   | DENY       | Anywhere (v6) |
| 2200/tcp (v6)             | ALLOW      | Anywhere (v6) |
| 80/tcp (v6)               | ALLOW      | Anywhere (v6) |
| 123/tcp (v6)              | ALLOW      | Anywhere (v6) |
| 5432 (v6)                 | ALLOW      | Anywhere (v6) |

* Edited `/etc/apache2/sites-enabled/000-default.conf`:
```
<VirtualHost *:80>
		ServerName mywebsite.com
		ServerAdmin admin@mywebsite.com
		WSGIScriptAlias / /var/www/Giftr/giftr.wsgi
		<Directory /var/www/Giftr/Giftr/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/Giftr/Giftr/appli/static
		<Directory /var/www/Giftr/Giftr/appli/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

* Added my app to `/var/www/Giftr`, along with a `giftr.wsgi` file containing the following lines:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/Giftr/")

from Giftr.run import app as application

application.secret_key='super_secret_key'
```

* Created a new database: `sudo su - postgres; psql; CREATE DATABASE giftr;`
* Created a new role in the database: `CREATE ROLE catalog WITH LOGIN;`

## Third-party resources used to complete this project
* [DigitalOcean: How To Secure PostgreSQL on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
* [DigitalOcean: How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
* [SQLAlchemy: Engine Configuration](http://docs.sqlalchemy.org/en/latest/core/engines.html)
* [Flask: mod_wsgi (Apache)](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/)
* [Stackoverflow: Python locale error: unsupported locale setting](https://stackoverflow.com/questions/14547631/python-locale-error-unsupported-locale-setting)
* [CodeASite: How do I find Apache http server log files?](http://www.codeasite.com/index.php/linux-a-apache/94-how-do-i-find-apache-http-server-log)
* [Postgresql Documentation](https://www.postgresql.org/docs/9.1/static/)
* [Psycopg2 Documentation](http://initd.org/psycopg/docs/module.html#psycopg2.connect)
* And of course the Udacity lessons and forums...
