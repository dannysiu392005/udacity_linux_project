# Linux Server Configuration
This repo is the source code of the Full Stack Web Developer Nanodegree Project
VI - Linux Server Configuration

# IP Address
The IP address is 18.216.177.62 and the URL is http://18.216.177.62/ and the port used to ssh is 2200

# Summary of software installed
- click==6.7
- Flask==0.12.2
- httplib2==0.10.3
- itsdangerous==0.24
- Jinja2==2.10
- MarkupSafe==1.0
- psycopg2==2.7.3.2
- SQLAlchemy==1.1.15
- Werkzeug==0.12.2

# Summary of configurations made
- Update all currently installed packages by
    - `sudo apt-get update`
    - `sudo apt-get upgrade`
- Change the SSH port from 22 to 2200
    - Locate the line `PORT 22` in `/etc/ssh/sshd_config`
    - Change `22` to `2200`
    - Restart the service by `sudo service ssh restart` (if not ok, try `sudo service sshd restart`)
- Configure firewall
    - `sudo ufw default deny incoming`
    - `sudo ufw default allow outgoing`
    - `sudo ufw default allow www`
    - `sudo ufw default allow 2200/tcp`
    - `sudo ufw default allow 123/tcp`
    - `sudo ufw enable`

# Create a user grader and give grader root access
- Run `sudo adduser grader` to add a new user `grader`
- Run `sudo vim /etc/sudoers.d/grader`
- Add content `grader ALL=(ALL) NOPASSWD:ALL` and then save and quite

# Create an SSH key pair for grader using the ssh-keygen
- Generate the key on your local machine using ssh-keygen
- On the server, login as `grader` by `su - grader`
- Go to the directory `/home/grader`
- Type `mkdir .ssh`
- Type `vim .ssh/authorized_keys` and paste the public key into `authorized_keys` and save
- Type `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`

# Configure the local timezone to UTC
- `sudo timedatectl set-timezone Etc/UTC`

#  Install and configure Apache to serve a Python mod_wsgi application
- `sudo apt-get install apache2`
- `sudo apt-get install libapache2-mod-wsgi`

# Install and configure PostgreSQL
- `sudo apt-get install postgresql`
- Remote connections are not allowed by checking `/etc/postgresql/9.3/main/pg_hba.conf` is empty or does not exist
- Database Configuration
    - Login as `postgres` by `sudo su - postgres`
    - Enter PostgreSQL by `psql`
    - Create a database by `CREATE DATABASE catalog;`
    - Create a user by `CREATE USER catalog;`
    - Set password by `ALTER ROLE catalog WITH PASSWORD 'p@s5wOrd';`
    - Grant permission by `GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`

# Install git
- `sudo apt-get install git`

# Deploy the Item Catalog using git
- Go to the directory `/var/www`
- Type `sudo mkdir FlaskApp` and `cd` into `FlaskApp`
- Git clone the item catalog project
- Rename the directory to `FlaskApp`
- The directory structure will be
`|var`
`|--www`
`|----FlaskApp`
`|------FlaskApp`
`|--------static`
`|--------templates`
- Rename `application.py` to `__init__.py`
- You may need to change the credentials in `fb_client_secrets.json` and in `/templates/login.html`
- Change `engine = create_engine('sqlite:///categoryitemwithusers.db')` to `engine = create_engine('postgresql://catalog:p@s5wOrd@localhost/catalog')`
- Use full path for reading fb credentials, in `__init__.py`, change 
    - `app_id = json.loads(open('fb_client_secrets.json', 'r').read())['web']['app_id']` to `app_id = json.loads(open('/var/www/FlaskApp/FlaskApp/fb_client_secrets.json', 'r').read())['web']['app_id']`
    and 
    - `app_secret = json.loads(open('fb_client_secrets.json', 'r').read())['web']['app_secret']` to `app_secret = json.loads(open('/var/www/FlaskApp/FlaskApp/fb_client_secrets.json', 'r').read())['web']['app_secret']`
- Install pip by `sudo apt-get install python-pip`
    - You may want to install the softare listed in **Summary of software installed** or skip it for now and pay attention to the final step
- Create the databse schema by `sudo python database_setup.py`

# Configure and Enable a New Virtual Host
- `sudo vim /etc/apache2/sites-available/FlaskApp.conf`
- Add the following content into it
    ```
    <VirtualHost *:80>
        ServerName 18.216.177.62
        ServerAdmin dannysiu392005@gmail.com
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
- Enable the virtual host by `sudo a2ensite FlaskApp`

# Create the .wsgi File
- `cd /var/www/FlaskApp`
- `sudo vim flaskapp.wsgi`
- Add the following content
  ```
  #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0,"/var/www/FlaskApp/")

  from FlaskApp import app as application
  application.secret_key = 'This is my secret key'
  ```

# Restart Apache
- `sudo service apache2 restart`

# Final step
- When visiting your own webapp, you may encounter error, type `sudo tail –f /var/log/apache2/error.log` to look for the error. For example `ImportError: No module named werkzeug.exceptions`. You can install it by `sudo -H pip install werkzeug`. Keep repeating this step until there is no error.

# Reference
- [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
- [ImportError: No module named werkzeug.exceptions](https://github.com/martin-riedl/CALexa/issues/1)
- [How to Allow Remote Connection to PostgreSQL Database using psql](http://www.thegeekstuff.com/2014/02/enable-remote-postgresql-connection/?utm_source=tuicool)
- [Changing the SSH Port for Your Linux Server](https://ca.godaddy.com/help/changing-the-ssh-port-for-your-linux-server-7306)
- [How do I change my timezone to UTC/GMT?](https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt)

# License
This project is released under the [MIT License](https://opensource.org/licenses/MIT).

