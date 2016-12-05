# Linux Server Configuration

- Public IP: 35.164.165.70
- SSH PORT: 2200
- URL: http://ec2-35-164-165-70.us-west-2.compute.amazonaws.com/

## 1) Create Development Environment and SSH Into Server
Download Private Key

Move the private key file into the folder ~/.ssh (where ~ is your environment's home directory)

`mv ~/Downloads/udacity_key.rsa ~/.ssh/`

Open your terminal and type in

`chmod 600 ~/.ssh/udacity_key.rsa`

SSH into the server

`ssh -i ~/.ssh/udacity_key.rsa root@<PUBLIC_IP>`

## 2) Create a New User, Change User Permissions, Update All Currently Installed Packages

Create a new user named grader

`sudo adduser grader`

Give the grader sudo permission

`sudo visudo`

Add `grader ALL=(ALL:ALL) ALL` in the file right below `root ALL=(ALL:ALL) ALL` in the User Privilege Specification Header. Save file and exit

Update all currently installed packages

`sudo apt-get update`

`sudo apt-get upgrade`

## 3) Edit SSH Configuration, Create SSH Keys, and Login

`nano /etc/ssh/sshd_config`

- Change Port 22 to Port 2200
- Change PermitRootLogin without-password to PermitRootLogin no to disallow root login
- Change PasswordAuthentication from no to yes. We will change back after finishing SHH login setup
- Add `AllowUsers grader` to the file

`sudo service ssh reload`

## 4) Configure Uncomplicated Firewall

## 5) Configure Local Timezone to UTC

## 6) Install and Configure Apache to Serve a Python Mod-wsgi App, Github

`sudo apt-get install apache2`

`sudo apt-get install python-setuptools libapache2-mod-wsgi`

To avoid apr_sockaddr_info_get() failed error message

`sudo nano /etc/apache2/conf-available/servername.conf`

Change the server's name to localhost. Add the below line to the conf file

`ServerName localhost`

Enable the config file

`sudo a2enconf servername`

Restart the server

`sudo service apache2 restart`

Install and configure Github

`sudo apt-get install git`

`git config --global user.name <USERNAME>`

`git config --global user.email <EMAIL_ADDRESS>`

## 7) Create a Boilerplate Flask App

[Source: DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

We will configure the virtual host later.

Move to the /var/www directory

`cd /var/www`

Create the application directory structure using mkdir as shown. Replace "FlaskApp" with the name you would like to give your application. Create the initial directory FlaskApp by giving following command:

`sudo mkdir FlaskApp`

Move inside this directory using the following command:

`cd FlaskApp`

Create another directory FlaskApp by giving following command:

`sudo mkdir FlaskApp`

Then, move inside this directory and create two subdirectories named static and templates using the following commands:

`cd FlaskApp`
`sudo mkdir static templates`

Your directory structure should now look like this:

```
|----FlaskApp
|---------FlaskApp
|--------------static
|--------------templates
```

Now, create the __init__.py file that will contain the flask application logic.

`sudo nano __init__.py` 

Add following logic to the file:
```
from flask import Flask
app = Flask(__name__)
@app.route("/")
def hello():
    return "Hello, I love Digital Ocean!"
if __name__ == "__main__":
    app.run()
```
    
Save and close the file.

Setting up a virtual environment will keep the application and its dependencies isolated from the main system. Changes to it will not affect the cloud server's system configurations.

In this step, we will create a virtual environment for our flask application.

We will use pip to install virtualenv and Flask. If pip is not installed, install it on Ubuntu through apt-get.

`sudo apt-get install python-pip` 

If virtualenv is not installed, use pip to install it using following command:

`sudo pip install virtualenv`

Give the following command (where venv is the name you would like to give your temporary environment):

`sudo virtualenv venv`

Now, install Flask in that environment by activating the virtual environment with the following command:

`source venv/bin/activate`

Give this command to install Flask inside:

`sudo pip install Flask`

Next, run the following command to test if the installation is successful and the app is running:

`sudo python __init__.py`

It should display “Running on http://localhost:5000/” or "Running on http://127.0.0.1:5000/". If you see this message, you have successfully configured the app.

To deactivate the environment, give the following command:

`deactivate`

## 8) Configure and Enable a New Virtual Host

[Source: DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

Issue the following command in your terminal:

`sudo nano /etc/apache2/sites-available/FlaskApp`

NOTE: Newer versions of Ubuntu (13.10+) require a ".conf" extension for VirtualHost files -- run the following command instead:

`sudo nano /etc/apache2/sites-available/FlaskApp.conf`

Add the following lines of code to the file to configure the virtual host. Be sure to change the ServerName to your domain or cloud server's IP address:

```
<VirtualHost *:80>
		ServerName mywebsite.com
		ServerAdmin admin@mywebsite.com
		WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
		<Directory /var/www/FlaskApp/FlaskApp/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/FlaskApp/FlaskApp/static
		<Directory /var/www/FlaskApp/FlaskApp/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Save and close the file.

Enable the virtual host with the following command:

`sudo a2ensite FlaskApp`

Apache uses the .wsgi file to serve the Flask app. Move to the /var/www/FlaskApp directory and create a file named flaskapp.wsgi with following commands:

`cd /var/www/FlaskApp`
`sudo nano flaskapp.wsgi` 

Add the following lines of code to the flaskapp.wsgi file:

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")

from FlaskApp import app as application
application.secret_key = 'Add your secret key'
```

Now your directory structure should look like this:

```
|--------FlaskApp
|----------------FlaskApp
|-----------------------static
|-----------------------templates
|-----------------------venv
|-----------------------__init__.py
|----------------flaskapp.wsgi
```

Restart Apache with the following command to apply the changes:

`sudo service apache2 restart`

You may see a message similar to the following:

Could not reliably determine the VPS's fully qualified domain name, using 127.0.0.1 for ServerName 

This message is just a warning, and you will be able to access your virtual host without any further issues. To view your application, open your browser and navigate to the domain name or IP address that you entered in your virtual host configuration.

You have successfully deployed a flask application.
