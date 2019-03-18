# Project Description

This project configures an ubuntu instance to run a web application built based on Flask framework. The web server is hosted on AWS Lightsail for a standalone server setup based on apache, and deploy the flask app through mod_wsgi module.

# Software installation
Ubtuntu LTS 16.04 is chosen as the OS when creating Lightsail instance.

## Install python modules

Install python pip, then use pip to install Flask, sqlalchemy, requests (required by Google Sign-in) and Google Sign-in modules.

```
sudo apt install python-pip
pip install Flask
pip install sqlalchemy
pip install requests
pip install --upgrade google-api-python-client
```

## Install Apache

```
sudo apt-get update
sudo apt-get install apache2
```

After install, apache comes with a default website which can be access through
```
curl localhost
```

Or though the public ip address.

## Install and configure mod_wsgi

```
sudo apt-get install libapache2-mod-wsgi
apachectl -M | grep wsgi
```

mod-wsgi should be enabled after install. If not, enable it manually.

```
sudo a2enmod wsgi
```

The application is stored in /var/www/Item-Catalog. The main python application file is /var/www/Item-Catalog/application.py. Then in this directory, create a wsgi file with the following content.

```
# modify path before importing app
import sys
sys.path.insert(0, '/var/www/Item-Catalog')

from application import app as application
```

The following configuration are created as /etc/apache2/sites-available/myapp.conf

```
<VirtualHost *>
    ServerName localhost

    WSGIDaemonProcess myapp user=ubuntu group=ubuntu threads=5
    WSGIScriptAlias / /var/www/Item-Catalog/myapp.wsgi

    <Directory /var/www/Item-Catalog>
        WSGIProcessGroup myapp
        WSGIApplicationGroup %{GLOBAL}
        Order deny,allow
        Allow from all
    </Directory>
</VirtualHost>
```

Enable the site. To prevent the default Apache page to catch the virtual host, disable it. Then reload apache server.

```
sudo a2ensite myapp
sudo a2dissite 000-default
sudo service apache2 reload
```

# grader user

Create grader user and provide sudo access.

```
adduser grader
usermod -aG sudo grader
```


# Security configuration
Enable the basic firewall using ufw.

```
sudo ufw status
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw deny ssh
sudo ufw allow 2200
sudo ufw allow www
sudo ufw enable
```

Add the following line to ```/etc/ssh/sshd_config``` to enable ssh on port 2200 but not port 22, and disable root login entirely.

```
# Port 22
Port 2200
PermitRootLogin no
```

Patch the software to the latest version to minimize the potential vulnerabilities.

```
sudo apt-get update
sudo apt-get upgrade
```

Use the GUI of AWS Lightsail of VM instance config, in the Networking tab, add Firewall rule to allow TCP 2200.



# Common problems and solutions
#### Apache shows the default page despite the virtual host config matches exactly the application site.
Disable default site by a2dissite 000-default.

#### Apache gives 500 internal server error and /var/log/apache2/error.log gives the following message: "Target WSGI script '/var/www/Item-Catalog/myapp.wsgi' cannot be loaded as Python module."
Make sure the path of your app is in the python path before loading the application file or module. As illustrated in above wsgi config file example, add the path before importing application.

#### Apache gives Internal Server Error and /var/log/apache2/error.log gives the following message: "RuntimeError: The session is unavailable because no secret key was set.  Set the secret_key on the application to something unique and secret."
The setting of flask app secret key needs to be executed upon import. If it is inside an if block to be exectuted when called directly, it has to be moved to module global level.

# Reference
- [Install apache on ubuntu](https://tutorials.ubuntu.com/tutorial/install-and-configure-apache#1)
- [Install mod_wsgi for flask deployment](http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/)
- [Install Google API client libraries](https://developers.google.com/api-client-library/python/start/installation)
- [mod_wsgi Quick Configuration Guide](https://modwsgi.readthedocs.io/en/develop/user-guides/quick-configuration-guide.html)
- [Virtual Host configuration in Apache](https://www.guru99.com/apache.html#7)

