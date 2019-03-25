# Full Stack Web Developer Exercise 4: Item Catalog

## Objective

Take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

- IP address: 18.224.65.159
- Accessible SSH port: 2200
- Application URL: http://18.224.65.159/ or http://ec2-18-224-65-159.us-east-2.compute.amazonaws.com/


## Step Walkthrough

### 1.  Start a new Ubuntu Linux server instance on Amazon Lightsail

- Login into [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/resources) (requires registration).
- Click `Create instance`.
- Choose `Linux/Unix` platform, `OS Only` and  `Ubuntu 16.04 LTS`.
- Choose Instance Plan.
- Provide Instance Name.
- Click the `Create` button to create instance.
- Wait for instance start up.

**Reference**
- Amazon Lightsail, [Create an Amazon Lightsail instance](https://lightsail.aws.amazon.com/ls/docs/en/articles/how-to-create-amazon-lightsail-instance-virtual-private-server-vps).


### 2.  SSH into the server

- From the `Account` menu on Amazon Lightsail, click on `SSH keys` tab and download the Default Private Key.
- Move this private key file named `LightsailDefaultPrivateKey-*.pem` into the local folder `~/.ssh` and rename it `lightsail_key.rsa`.
- In terminal, type: `chmod 600 ~/.ssh/lightsail_key.rsa`.
- To connect to the instance via the terminal: `ssh -i ~/.ssh/lightsail_key.rsa ubuntu@18.224.65.159`


## Secure the server

### 3.  Update and upgrade installed packages

```
sudo apt-get update
sudo apt-get upgrade
```


### 4.  Change the SSH port from 22 to 2200

- Edit the `/etc/ssh/sshd_config` file: `sudo nano /etc/ssh/sshd_config`.
- Change the port number (line 5) from `22` to `2200`.
- Save and Exit file.
- Restart SSH: `sudo service ssh restart`.


### 5.  Configure the Uncomplicated Firewall (UFW)

- Configure the default firewall to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123):
  ```
  sudo ufw status                  # The UFW should be inactive.
  sudo ufw default deny incoming   # Deny any incoming traffic.
  sudo ufw default allow outgoing  # Enable outgoing traffic.
  sudo ufw allow 2200/tcp          # Allow incoming tcp packets on port 2200.
  sudo ufw allow www               # Allow HTTP traffic in.
  sudo ufw allow 123/udp           # Allow incoming udp packets on port 123.
  sudo ufw deny 22                 # Deny tcp and udp packets on port 53.
  ```

- Turn UFW on: `sudo ufw enable`:
  ```
  Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
  Firewall is active and enabled on system startup
  ```

- Verify status of UFW roles: `sudo ufw status`:

  ```
  Status: active

  To                         Action      From
  --                         ------      ----
  2200/tcp                   ALLOW       Anywhere                  
  80/tcp                     ALLOW       Anywhere                  
  123/udp                    ALLOW       Anywhere                  
  22                         DENY        Anywhere                  
  2200/tcp (v6)              ALLOW       Anywhere (v6)             
  80/tcp (v6)                ALLOW       Anywhere (v6)             
  123/udp (v6)               ALLOW       Anywhere (v6)             
  22 (v6)                    DENY        Anywhere (v6)
  ```

- Exit the SSH connection: `exit`.

- Click on the `Manage` option of the Amazon Lightsail Instance.

- Click the `Networking` tab and change the firewall configuration to match the internal firewall settings above.

- Allow ports 80(TCP), 123(UDP), and 2200(TCP), and deny the default port 22.

- From local terminal, run: `ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@18.224.65.159` to login with new settings applied


**Reference**
- Official Ubuntu Documentation, [UFW - Uncomplicated Firewall](https://help.ubuntu.com/community/UFW).
- nixCraft, [How to setup a UFW firewall on Ubuntu 16.04 LTS server](https://www.cyberciti.biz/faq/howto-configure-setup-firewall-with-ufw-on-ubuntu-linux/).


### 6.  Configure firewall to monitor failed login attempts and ban attackers by installing `Fail2Ban`

- Install Fail2Ban: `sudo apt-get install fail2ban`.
- Install sendmail for email notice: `sudo apt-get install sendmail iptables-persistent`.
- Create local file to override certain settings in .conf file: `sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`.
- Change the settings in `/etc/fail2ban/jail.local` file:
  ```
  set bantime = 600
  action = %(action_mwl)s
  ```
- Under `[sshd]` change `port = ssh` by `port = 2200`.
- Restart the service: `sudo service fail2ban restart`.

**Reference**
- DigitalOcean, [How To Protect SSH with Fail2Ban on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04).
- Linode, [Use fail2ban to secure your server] (https://www.linode.com/docs/security/using-fail2ban-for-security/)


### 7.  Automatically install updated packages

- Install the software: `sudo apt-get install unattended-upgrades`.
- Edit the `/etc/apt/apt.conf.d/50unattended-upgrades`file
    - Uncomment the line `${distro_id}:${distro_codename}-updates` to enable general updates.
- Modify `/etc/apt/apt.conf.d/20auto-upgrades` file as follows:
  ```
  APT::Periodic::Update-Package-Lists "1";
  APT::Periodic::Download-Upgradeable-Packages "1";
  APT::Periodic::AutocleanInterval "7";
  APT::Periodic::Unattended-Upgrade "1";
  ```
- Enable package: `sudo dpkg-reconfigure --priority=low unattended-upgrades`.
- Restart Apache: `sudo service apache2 restart`.

**Reference**
- Official Ubuntu Documentation, [Automatic Updates](https://help.ubuntu.com/lts/serverguide/automatic-updates.html).


### 8.  Update packages to most recent versions

- Issue the following commands:
  ```
  sudo apt-get update
  sudo apt-get dist-upgrade
  sudo shutdown -r now
  ```

**Reference**
- DigitalOcean, [Updating Ubuntu 14.04 -- Security Updates](https://www.digitalocean.com/community/questions/updating-ubuntu-14-04-security-updates).


### 9.  Create a new account named `grader`

- While logged in as `ubuntu`, add user: `sudo adduser grader`.
- Enter a password (twice).


### 10.  Give `grader` sudo permission

- Edit sudoers file: `sudo visudo`.
- Search for the following line:
  ```
  root    ALL=(ALL:ALL) ALL
  ```

- Add a line below it giving sudo privilege to `grader`.
  ```
  root    ALL=(ALL:ALL) ALL
  grader  ALL=(ALL:ALL) ALL
  ```

- Save and Exit.

- Verify `grader` has sudo permissions:
    - `su - grader` and enter the password
    - `sudo -l` and enter the password again

**Resource**
- Secure1 Hosting, [How To Add and Delete Users on an Ubuntu 14.04 VPS] (https://secure1hosting.com/add-delete-users-ubuntu-14-04-vps/)


### 11.  Create an SSH key pair for `grader`

- On local machine:
  - Run `ssh-keygen`
  - Choose name for file to hold key (`grader_key`) in the local directory `~/.ssh`
  - Enter passphrase twice
  - Two files will be generated (`~/.ssh/grader_key` and `~/.ssh/grader_key.pub`)
  - Run `cat ~/.ssh/grader_key.pub`
  - Copy the contents of the file
  - Log in to the grader's virtual machine
- On the grader's virtual machine:
  - Create a new directory called `~/.ssh` (`mkdir .ssh`)
  - Run `sudo nano ~/.ssh/authorized_keys` and paste the content into this file
  - Save and Exit
  - Give the permissions: `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`
  - Check in `/etc/ssh/sshd_config` file if `PasswordAuthentication` is set to `no`
  - Restart SSH: `sudo service ssh restart`
- On local machine, run: `ssh -i ~/.ssh/grader_key -p 2200 grader@18.224.65.159`


**Reference**
- DigitalOcean, [How To Set Up SSH Keys](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2).


### 12.  Configure the local timezone to UTC

- While logged in as `grader`, configure the time zone: `sudo dpkg-reconfigure tzdata`. You should see something like that:

  ```
  Current default time zone: 'America/Chicago'
  Local time is now:      Wed Mar 20 10:09:47 CDT 2019.
  Universal Time is now:  Wed Mar 20 15:09:47 UTC 2019.
  ```

**References**
- Ubuntu Wiki, [UbuntuTime](https://help.ubuntu.com/community/UbuntuTime)
- Ask Ubuntu, [How do I change my timezone to UTC/GMT?](https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt/138442)


### 13.  Install and configure Apache to serve a Python mod_wsgi application

- While logged in as `grader`, install Apache: `sudo apt-get install apache2`.
- Enter public IP of the Amazon Lightsail instance into browser. If Apache is working, you should see Apache welcome screen.

- Install the Python 3 mod_wsgi package:  
 `sudo apt-get install libapache2-mod-wsgi-py3`.
- Enable `mod_wsgi` using: `sudo a2enmod wsgi`.

**References**
- Full Stack Python, [WSGI Servers](https://www.fullstackpython.com/wsgi-servers.html)

### 14.  Install and configure PostgreSQL

- While logged in as `grader`, install PostgreSQL:
 `sudo apt-get install postgresql`.
- PostgreSQL should not allow remote connections. In the  `/etc/postgresql/9.5/main/pg_hba.conf` file, you should see:
  ```
  local   all             postgres                                peer
  local   all             all                                     peer
  host    all             all             127.0.0.1/32            md5
  host    all             all             ::1/128                 md5
  ```

- Switch to the `postgres` user: `sudo su - postgres`.
- Open PostgreSQL interactive terminal with `psql`.
- Create the `catalog` user with a password and give them the ability to create databases:
  ```
  postgres=# CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';
  postgres=# ALTER ROLE catalog CREATEDB;
  ```

- List the existing roles: `\du`. The output should be like this:
  ```
                                     List of roles
   Role name |                         Attributes                         | Member of
  -----------+------------------------------------------------------------+-----------
   catalog   | Create DB                                                  | {}
   postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
  ```

- Exit psql: `\q`.
- Switch back to the `grader` user: `exit`.
- Create a new Linux user called `catalog`: `sudo adduser catalog`
- Enter password and fill out information.
- Give to `catalog` user the permission to sudo. Run: `sudo visudo`.
- Search for the lines that looks like this:
  ```
  root    ALL=(ALL:ALL) ALL
  grader  ALL=(ALL:ALL) ALL
  ```

- Below this line, add a new line to give sudo privileges to `catalog` user:
  ```
  root    ALL=(ALL:ALL) ALL
  grader  ALL=(ALL:ALL) ALL
  catalog  ALL=(ALL:ALL) ALL
  ```

- Save and Exit.
- Verify that `catalog` has sudo permissions
- As `catalog`, create a database: `createdb catalog`.
- Run `psql`
- Run `\l` to verify creation of new database. Output should be like this:
  ```
                                    List of databases
     Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
  -----------+----------+----------+-------------+-------------+-----------------------
   catalog   | catalog  | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
   postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
   template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
             |          |          |             |             | postgres=CTc/postgres
   template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
             |          |          |             |             | postgres=CTc/postgres
  (4 rows)
  ```
- Exit psql: `\q`.
- Switch back to the `grader` user: `exit`.

**Reference**
- DigitalOcean, [How To Secure PostgreSQL on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps).


### 15.  Install git

- While logged in as `grader`, install `git`: `sudo apt-get install git`.


### 16.  Clone and setup the Item Catalog project from the GitHub repository

- While logged in as `grader`, create `/var/www/catalog/` directory.
- Navigate to that directory and clone the catalog project:<br>
`sudo git clone https://github.com/tcu93/catalog catalog`.
- From `/var/www`, change ownership of the `catalog` directory to `grader` using: `sudo chown -R grader:grader catalog/`.
- Navigate to the `/var/www/catalog/catalog` directory.
- Rename `project.py` to `__init__.py`.

- Make the following modification within the `__init__.py`, (line 353):
  ```
  #app.run(host='0.0.0.0', port=5000)
  app.run()
  ```

- In `database_populate.py` (line 9):
   ```
   # engine = create_engine("sqlite:///catalog.db")
   engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')
   ```


### 17.  Authenticate login through Google (oauth2)

- Go to [Google Developers Platform](https://console.cloud.google.com/).
- Click `APIs & services` (left menu).
- Click `Credentials`.
- Create an OAuth Client ID (under the Credentials tab), and add http://18.224.65.159 and
http://ec2-18-224-65-159.us-east-2.compute.amazonaws.com as authorized JavaScript
origins.
- Add http://ec2-18-224-65-159.us-east-2.compute.amazonaws.com/oauth2callback
as authorized redirect URI.
- Download the corresponding JSON file, open it et copy the contents.
- Open `/var/www/catalog/catalog/client_secret.json` and paste the previous contents into the this file.
- Replace the client ID within the `templates/login.html` file (line 19).
- Edit `__init__.py` (line 24) adding the fully qualified path to the client_secrets.json file


### 18.  Install virtual environment and dependencies

- While logged in as `grader`, install pip: `sudo apt-get install python3-pip`.
- Install the virtual environment: `sudo apt-get install python-virtualenv`
- Change to the `/var/www/catalog/catalog/` directory.
- Create the virtual environment: `sudo virtualenv -p python3 venv3`.
- Change the ownership to `grader` with: `sudo chown -R grader:grader venv3/`.
- Activate the new environment: `. venv3/bin/activate`.
- Install the following dependencies:
  ```
  pip install httplib2
  pip install requests
  pip install --upgrade oauth2client
  pip install sqlalchemy
  pip install flask
  sudo apt-get install libpq-dev
  pip install psycopg2
  ```

- Run `python3 __init__.py` and you should see:
  ```
  * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
  ```

- Deactivate the virtual environment: `deactivate`.

**Reference**
- Real Python, [Python Virtual Environments: A Primer](https://realpython.com/python-virtual-environments-a-primer/).
- Flask documentation, [virtualenv](http://flask.pocoo.org/docs/0.12/installation/).


### 19.  Set up and enable a virtual host

- Add the following line in `/etc/apache2/mods-enabled/wsgi.conf` file
to use Python 3.

  ```
  #WSGIPythonPath directory|directory-1:directory-2:...
  WSGIPythonPath /var/www/catalog/catalog/venv3/lib/python3.5/site-packages
  ```

- Create `/etc/apache2/sites-available/catalog.conf` and add the
following lines to configure the virtual host:

  ```
  <VirtualHost *:80>
	  ServerName 18.224.65.159
    ServerAlias ec2-18-224-65-159.us-west-2.compute.amazonaws.com
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

- Enable virtual host: `sudo a2ensite catalog`.
  ```
  Enabling site catalog.
  To activate the new configuration, you need to run:
    service apache2 reload
  ```

- Reload Apache: `sudo service apache2 reload`.

**Resource**
- Medium, [Python 3.5, Flask, Apache2, Mod_WSGI3 on Ubuntu 16.04](https://medium.com/@esteininger/python-3-5-flask-apache2-mod-wsgi3-on-ubuntu-16-04-67894abf9f70)
- [Getting Flask to use Python3](https://stackoverflow.com/questions/30642894/getting-flask-to-use-python3-apache-mod-wsgi)


### 20.  Set up Flask application

- Create `/var/www/catalog/catalog.wsgi` file
- Add the following to the file:

  ```
  activate_this = '/var/www/catalog/catalog/venv3/bin/activate_this.py'
  with open(activate_this) as file_:
      exec(file_.read(), dict(__file__=activate_this))

  #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/catalog/catalog/")
  sys.path.insert(1, "/var/www/catalog/")

  from catalog import app as application
  application.secret_key = "..."
  ```

- Restart Apache: `sudo service apache2 restart`.

**Resource**
- Flask documentation, [Working with Virtual Environments](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/#working-with-virtual-environments)


### 21.  Set up the database schema and populate the database

- Edit `/var/www/catalog/catalog/database_populate.py`.

- Add the following to the beginning of the file:

  ```
  import sys
  sys.path.insert(0, "/var/www/catalog/catalog/venv3/lib/python3.5/site-packages")
  ```

- Add the following lines after creation of session:

  ```
  session.query(Item).delete()
  session.query(Category).delete()
  session.query(User).delete()
  ```

- From the `/var/www/catalog/catalog/` directory,
activate the virtual environment: `. venv3/bin/activate`.
- Run: `python data.py`.
- Deactivate the virtual environment: `deactivate`.


### 22.  Disable default Apache site

- Disable the default Apache site: `sudo a2dissite 000-default.conf`.

  ```
  Site 000-default disabled.
  To activate the new configuration, you need to run:
    service apache2 reload
  ```

- Reload Apache: `sudo service apache2 reload`.


### 23.  Final Steps

- Change the ownership of the project directories: `sudo chown -R www-data:www-data catalog/`.
- Restart Apache again: `sudo service apache2 restart`.
- Open your browser to http://18.224.65.159 or http://ec2-18-224-65-159.us-east-2.compute.amazonaws.com.


## Miscellaneous helpful commands

 - Apache server log: `sudo tail /var/log/apache2/error.log`.
 - Apache server restart: `sudo service apache2 restart`.


## Folder structure

My resulting project folder structure is as follows:

```
/var/www/catalog
    |-- catalog.wsgi
    |__ /catalog             # Application Module
         |-- __init__.py
         |-- database_populate.py
         |-- database_setup.py
         |__ client_secrets.json
         |-- /static
              |__ styles.css
         |-- /templates
              |-- categories.html
              |-- deleteItem.html
              |-- editItem.html
              |-- items.html
              |-- login.html
              |-- newItem.html
              |__ singleItem.html
         |-- /venv3          # Virtual Environment
```

## Helpful GitHub Resources

  - [boisalai](https://github.com/boisalai/udacity-linux-server-configuration)
  - [iliketomatoes](https://github.com/iliketomatoes/linux_server_configuration)
  - [rrjoson](https://github.com/rrjoson/udacity-linux-server-configuration)
  - [aviaryan](https://github.com/aviaryan/flask-postgres-apache-server)
  - [harushimo](https://github.com/harushimo/linux-server-configuration)

## Special Attribution

  Special Thanks to [boisalai](https://github.com/boisalai) for the most helpful README of all
