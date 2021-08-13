**Table of Contents**
- [1. How to Install WordPress with LEMP](#1-how-to-install-wordpress-with-lemp)
  - [1.1. Introduction](#11-introduction)
  - [1.2. Prerequisites](#12-prerequisites)
  - [1.3. Create MySQL Database and User for WordPress](#13-create-mysql-database-and-user-for-wordpress)
  - [1.4. Configuring Nginx for WordPress](#14-configuring-nginx-for-wordpress)
  - [1.5. Downloading WordPress](#15-downloading-wordpress)
  - [1.6. Setting up the WordPress Configuration File](#16-setting-up-the-wordpress-configuration-file)
  - [1.7. Completing the Installation Throught Web Interface](#17-completing-the-installation-throught-web-interface)

# 1. How to Install WordPress with LEMP

## 1.1. Introduction

**WordPress**, one of the most popular content management system (CMS), allows users to host flexible blogs and websites using MySQL and PHP processing.

## 1.2. Prerequisites
- A linux system (Ubuntu 20.04)
- A web server (Nginx)
- A database (MySql)
- PHP

Follow [How to Install Linux, Nginx, MySQL, PHP (LEMP Stack) on Ubuntu](/Linux/how-to-install-linux-nginx-mysql-php-lemp-stack-on-ubuntu.md) to setup LEMP stack

## 1.3. Create MySQL Database and User for WordPress

Log in to MySQL root (administrative) account using `sudo` if MySQL is configured to use `auth_socket` authentication plugin.
```
sudo mysql
```

Otherwise, use the following command instead.
```
mysql -u root -p
```

You will be prompted for the password set for MySQL root account.

Once logged in, create a `database` for WordPress. Replace `wordpress` to any name you like.
```sql
mysql> CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
```

> **Note:** Every MySQL statement must end in a semi-colon ';'

Next, create a `user` that will use exclusively to operate on new database.

You can replace `wordpressuser` and `password` to any username and password you like and `wordpress` replace to your database name.
```sql
/* create an account and set password */
mysql> CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY 'password';

/* grant permission for new account creatGed to access database*/
mysql> GRANT ALL ON wordpress.* TO 'wordpressuser'@'localhost';
```

Run following command to show all database
```sql
mysql> show databases
```

## 1.4. Configuring Nginx for WordPress

```nginx
server {
  listen 80;
  # can change 'wordpress.jh.com' to the name you like
  server_name wordpress.jh.com;

  # root path to store wordpress
  root /var/www/wordpress;

  # do not want to log request for /favicon.ico,  /robots.txt and static file
  location = /favicon.ico {
    log_not_found off;
    access_log off;
  }

  location = /robots.txt {
    log_not_found off;
    access_log off;
    allow all;
  }

  location ~*\.(css|gif|ico|jpeg|jpg|js|png)$ {
    expires max;
    log_not_found off;
  }

  location / {
    index index.php;

    # try_files $uri $uri/ =404;
    # instead of returning a 404 error as the default option,
    # control is passed to the index.php file with the request arguments
    try_files $uri $uri/ /index.php$is_args$args;
  }
        
  # pass PHP scripts to FastCGI server
  location ~ \.php$ {
    
    include snippets/fastcgi-php.conf;
    
    # With php-fpm (or other unix sockets):
    fastcgi_pass unix:/run/php/php7.2-fpm.sock;
  }
}
```

## 1.5. Downloading WordPress

Download the latest version of WordPress compressed file to temporary folder

```
cd /tmp
curl -LO https://wordpress.org/latest.tar.gz
```

> **Note:**<br>
> `-LO` to get directly to the source of the compressed file<br>
> `-L` to redo the request to the new location whenever it encounters a redirect<br>
> `-O` writes the output of our remote file with a local file that has the same name

Extract wordpress compressed file
```
tar xzvf latest.tar.gz
```

Copy `wp-config-sample.php` to `wp-config.php` that WordPress actually reads
```
cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php
```

Copy entire WordPress folder to root
```
sudo cp -a /tmp/wordpress/. /var/www/wordpress
```

> **Note:** <br>
> `-a` to make sure the permissions are maintained <br>
> `.` at the end to indicate everthing within the directory should be copied

Assign ownership to the `www-data` user and group. Nginx need to be able to read and write WordPress files in order to serve the website and perform automatic updates
```
sudo chown -R www-data:ww-data /var/ww/wordpress
```

## 1.6. Setting up the WordPress Configuration File

WordPress provides a secure generator for these values so that you don’t have to come up with values on your own

Grab secure value from the WordPress secret key generator
```
curl -s https://api.wordpress.org/secret-key/1.1/salt
```

> **Warning:** It is important that you request unique values each time. Do Not copy the values shown below

```
define('AUTH_KEY',         '$EeH^Xfp6eJ^Tuh5E6x+yyPpNjW}Ok&0p#6tfu,iXD(G=w}UP;+hK%2$KeTvpRt+');
define('SECURE_AUTH_KEY',  '+5$R^ODT7oo/Q!DAnTB]B+t9#l.@7PVD|C)L#oGdZ{|@#Y%V4&[WhGtkO+}si[+J');
define('LOGGED_IN_KEY',    '17~(TR^nGqtw<Y=Y3*jF6@i})@;Oo0J*Oo712Tu;3o1WkhZ&x--vxqz%s!c-mV#J');
define('NONCE_KEY',        'K!|E/=WT=bgx>R^$3d]t.Akfd0;k{1j+fsapK|k0SgBA--kRsQ*EX{3i-k{V c$7');
define('AUTH_SALT',        'e9z}EsDNBxzx|@,h^3B7}g-oDwuf(Mgh;r^.EuaJ&;9XRFt}4dA<#8}DokMG2)Y:');
define('SECURE_AUTH_SALT', '$1sO0:M>PrzN5yYGLJA++RYp|k#@)+F>j;Vhlv{>{2,8tFLd^}M0!yQ?Z@J4S&$5');
define('LOGGED_IN_SALT',   'pF,9HLPpxX5Yk*{O[wF:Z!1KN+9_f>ilmdA,{V6orhQ&;!X6E(`I=8Wvr!|%/+JD');
define('NONCE_SALT',       '(?-+HZEv:M4ZqSiex|vj3bDV|>xI<)4Q~{.1H}KPt7d<2fiEnm]T,zL.&~q`?gWd');
```

Replace the secret key grabed from WordPress secret key generator
```
define( 'AUTH_KEY',         'put your unique phrase here' );
define( 'SECURE_AUTH_KEY',  'put your unique phrase here' );
define( 'LOGGED_IN_KEY',    'put your unique phrase here' );
define( 'NONCE_KEY',        'put your unique phrase here' );
define( 'AUTH_SALT',        'put your unique phrase here' );
define( 'SECURE_AUTH_SALT', 'put your unique phrase here' );
define( 'LOGGED_IN_SALT',   'put your unique phrase here' );
define( 'NONCE_SALT',       'put your unique phrase here' );
```

Nex, adjust the `DB_NAME`, the `DB_USER`, and `DB_PASSWORD` that was configured within MySQL.

```bash
#/** The name of the database for WordPress */
define( 'DB_NAME', 'database_name_here' );

#/** MySQL database username */
define( 'DB_USER', 'username_here' );

#/** MySQL database password */
define( 'DB_PASSWORD', 'password_here' );

define('FS_METHOD', 'direct');
```

## 1.7. Completing the Installation Throught Web Interface

Server configuration is complete, navigate to your server’s domain name or public IP address
```
wordpress.jh.com
```
Now, you can select the language you want to use

![select language](/Linux/img/install-wordpress_language.jpg)

Next, you will come to the main setup page, enter all the information and press install wordpress

![setup page](/Linux/img/install-wordpress_setup-page.jpg)

After installation completed, you can login to admin page

![login](/Linux/img/install-wordpress_login.jpg)

Once you log in, you will be taken to the WordPress administration dashboard

![dashboard](/Linux/img/install-wordpress_dashboard.jpg)