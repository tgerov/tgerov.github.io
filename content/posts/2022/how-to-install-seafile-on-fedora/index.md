---
categories:
- Linux
date: "2022-06-30"
draft: false
tags:
- linux
- seafile
- fedora
title: How to install Seafile on Fedora
summary: How to install Seafile Server 9 Community Edition with MySQL/MariaDB on Fedora, including SystemD Units, Nginx and Let' s Encrypt.
---

This tutorial is based on official SeaFile [documentation](https://manual.seafile.com/deploy/using_mysql/) and it's adapted for [Fedora 36](https://getfedora.org).

# Install MariaDB Server
Install MariaDB server via yum/dnf

```bash
dnf install mariadb-server
```

Enable and start now MariaDB server

```bash
systemctl enable --now mariadb
```
# Secure MariaDB Server
Prepare MariadB server for production.
In this step we will set MariaDB root password, which will be required from SeaFile Server installation scripts.

Here is an exampele:
```
mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user. If you've just installed MariaDB, and
haven't set the root password yet, you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password or using the unix_socket ensures that nobody
can log into the MariaDB root user without the proper authorisation.

You already have your root account protected, so you can safely answer 'n'.

Switch to unix_socket authentication [Y/n] n
 ... skipping.

You already have your root account protected, so you can safely answer 'n'.

Change the root password? [Y/n] y
New password: [ENTER NEW ROOT PASSWORD HERE]
Re-enter new password: [RE-ENTER NEW ROOT PASSWORD HERE]
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```
# Install Seafile Deps
```bash
dnf install wget python3 python3-setuptools python3-pip python3-devel mariadb-devel gcc \
python3-pillow python3-pylibmc python3-jinja2 python3-sqlalchemy \
python3-pylibmc python3-ldap python3-mysqlclient python3-pycryptodomex \
memcached memcached-devel
```
# Install Python modules
```bash
pip3 install captcha django-pylibmc django-simple-captcha
```
# Setup SeaFile
Create SeaFile user with home directory /opt/seafile
```bash
adduser -b /opt seafile
```

Change user to seafile
```bash
su - seafile
```
Download the install package from the Seafile Download page.
We use Seafile CE version 9.0.6 as an example in the rest of this manual.

```bash
wget https://s3.eu-central-1.amazonaws.com/download.seadrive.org/seafile-server_9.0.6_x86-64.tar.gz
```
Uncompress the package by using tar

```bash
tar xf seafile-server_9.0.6_x86-64.tar.gz
```

Run the installation script
```bash
cd seafile-server-9.0.6/
./setup-seafile-mysql.sh
```

You will be asked several questions, including MariaDB password.

Here is an example:
```bash
./setup-seafile-mysql.sh
Checking python on this machine ...

-----------------------------------------------------------------
This script will guide you to setup your seafile server using MySQL.
Make sure you have read seafile server manual at

        https://download.seafile.com/published/seafile-manual/home.md

Press ENTER to continue
-----------------------------------------------------------------

What is the name of the server? It will be displayed on the client.
3 - 15 letters or digits
[ server name ] fedora-cloud

What is the ip or domain of the server?
For example: www.mycompany.com, 192.168.1.101
[ This server's ip or domain ] fedora.cloud.local

Which port do you want to use for the seafile fileserver?
[ default "8082" ]

-------------------------------------------------------
Please choose a way to initialize seafile databases:
-------------------------------------------------------

[1] Create new ccnet/seafile/seahub databases
[2] Use existing ccnet/seafile/seahub databases

[ 1 or 2 ] 1

What is the host of mysql server?
[ default "localhost" ]

What is the port of mysql server?
[ default "3306" ]

What is the password of the mysql root user?
[ root password ]

verifying password of user root ...  done

Enter the name for mysql user of seafile. It would be created if not exists.
[ default "seafile" ]

Enter the password for mysql user "seafile":
[ password for seafile ]

Enter the database name for ccnet-server:
[ default "ccnet-db" ]

Enter the database name for seafile-server:
[ default "seafile-db" ]

Enter the database name for seahub:
[ default "seahub-db" ]

---------------------------------
This is your configuration
---------------------------------

    server name:            fedora-cloud
    server ip/domain:       fedora.cloud.local

    seafile data dir:       /opt/seafile/seafile-data
    fileserver port:        8082

    database:               create new
    ccnet database:         ccnet-db
    seafile database:       seafile-db
    seahub database:        seahub-db
    database user:          seafile



---------------------------------
Press ENTER to continue, or Ctrl-C to abort
---------------------------------
Generating ccnet configuration ...

Generating seafile configuration ...

done
Generating seahub configuration ...

----------------------------------------
Now creating ccnet database tables ...

----------------------------------------
----------------------------------------
Now creating seafile database tables ...

----------------------------------------
----------------------------------------
Now creating seahub database tables ...

----------------------------------------

creating seafile-server-latest symbolic link ...  done




-----------------------------------------------------------------
Your seafile server configuration has been finished successfully.
-----------------------------------------------------------------

run seafile server:     ./seafile.sh { start | stop | restart }
run seahub  server:     ./seahub.sh  { start <port> | stop | restart <port> }

-----------------------------------------------------------------
If you are behind a firewall, remember to allow input/output of these tcp ports:
-----------------------------------------------------------------

port of seafile fileserver:   8082
port of seahub:               8000

When problems occur, Refer to

        https://download.seafile.com/published/seafile-manual/home.md

for information.
```

Start Seafile server via scripts and setup admin user
```bash
./seafile.sh start
./seahub.sh start
Starting seahub at port 8000 ...
----------------------------------------
It's the first time you start the seafile server. Now let's create the admin account
----------------------------------------

What is the email for the admin account?

[ admin email ] tg@local.local

What is the password for the admin account?
[ admin password ]

Enter the password again:
[ admin password again ]
----------------------------------------
Successfully created seafile admin
----------------------------------------
```
Stop SeaFile and SeaHub
```bash
./seafile.sh stop
./seahub.sh stop
```
# Create SystemD units to start Seafile at boot

Note: Exit from user seafile, commands below requires root privileges.

Create SeaFile Unit
```bash
vi /etc/systemd/system/seafile.service
```
The content of the file is:
```
[Unit]
Description=Seafile
After=network.target mariadb.service

[Service]
Type=forking
ExecStart=/opt/seafile/seafile-server-latest/seafile.sh start
ExecStop=/opt/seafile/seafile-server-latest/seafile.sh stop
LimitNOFILE=infinity
User=seafile
Group=seafile

[Install]
WantedBy=multi-user.target
```

Create SeaHub Unit:
```bash
vi /etc/systemd/system/seahub.service
```

The content of the file is:
```
[Unit]
Description=Seafile hub
After=network.target seafile.service

[Service]
Type=forking
ExecStart=/opt/seafile/seafile-server-latest/seahub.sh start
ExecStop=/opt/seafile/seafile-server-latest/seahub.sh stop
User=seafile
Group=seafile

[Install]
WantedBy=multi-user.target
```

Enable and start seafile and seahub:
```bash
systemctl enable --now seafile.service
systemctl enable --now seahub.service
```
# Setup Nginx as Reverse Proxy
Install Nginx
```bash
dnf install nginx
```

Enable and start Nginx service
```bash
systemctl enable --now nginx
```

Allow HTTP (80/tcp) and HTTPS (443/tcp) in Firewall
```bash
firewall-cmd --add-service={http,https} --permanent
firewall-cmd --reload
```

If SELinux is enabled on your host, allow HTTPD scripts and modules to connect to the network. 
```bash
setsebool -P httpd_can_network_connect 1
```

Create Nginx config file for seafile domain:
```bash
vi /etc/nginx/conf.d/seafile.conf
```

Copy the following sample Nginx config file into the just created seafile.conf and modify the content to fit your needs:

```
log_format seafileformat '$http_x_forwarded_for $remote_addr [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent" $upstream_response_time';

server {
    listen 80;
    server_name seafile.example.com;

    proxy_set_header X-Forwarded-For $remote_addr;

    location / {
         proxy_pass         http://127.0.0.1:8000;
         proxy_set_header   Host $host;
         proxy_set_header   X-Real-IP $remote_addr;
         proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
         proxy_set_header   X-Forwarded-Host $server_name;
         proxy_read_timeout  1200s;

         # used for view/edit office file via Office Online Server
         client_max_body_size 0;

         access_log      /var/log/nginx/seahub.access.log seafileformat;
         error_log       /var/log/nginx/seahub.error.log;
    }
    location /seafhttp {
        rewrite ^/seafhttp(.*)$ $1 break;
        proxy_pass http://127.0.0.1:8082;
        client_max_body_size 0;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_connect_timeout  36000s;
        proxy_read_timeout  36000s;
        proxy_send_timeout  36000s;

        send_timeout  36000s;

        access_log      /var/log/nginx/seafhttp.access.log seafileformat;
        error_log       /var/log/nginx/seafhttp.error.log;
    }
    location /media {
        root /opt/seafile/seafile-server-latest/seahub;
    }
}

```
Reload nginx configuration
```bash
systemctl reload nginx
```

# Issue Lets Encrypt SSL

Install certbot:
```bash
dnf install python3-certbot python3-certbot-nginx
```

Issue LetsEncrypt SSL and auto deploy it in nginx:

```bash
certbot --nginx
```

References: [SeaFile Deploy Manual](https://manual.seafile.com/deploy/)
