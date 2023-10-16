# darey.lampstack

## Installing the LAMP tech stack

The LAMP technology stack consists of the following:
- Linux OS
- Apache web server
- MySQL database
- Prgramming Language (PHP most common)

This stack forms the foundation of one of the most common platforms for developing dynamic web applications:

## Install Apache

$ sudo apt update
$ sudo apt install apache2 

### Test that apache is running:

$ sudo systemsctl status apache2

### Check the server can be reached locally:

$ curl http://127.0.0.1:80
or
$ curl http://localhost:80

### Check it can be reached via the internet:

$ http://<PublicIP>:80
or
$ curl -s http://169.254.169.254/latest/meta-data/public-ipv4

## Install MySQL

$ sudo apt install mysql-server
Type Y for any prompts

Log into MySQL console:
$ sudo mysql

Once logged in, run the security script that comes with MySQL pre-installed:
$ ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';
$ exit

Then in linux, start the interactive script:
$ sudo mysql_secure_installation

There is a password for logging into MySQL and a passowrd as a root user within MySQL. The prompts that follow are for both of these:

Once complete, log back into MySQL:
$ sudo mysql -p

-p flag will prompt the user for the password used after changing the root user password

Notice that you need to provide a password to connect as the root user.
For increased security, it's best to have dedicated user accounts with less expansive privileges set up for every database, especially if you plan on having multiple databases hosted on your server.

[Note: At the time of this writing, the native MySQL PHP library mysqlnd doesn't support caching_sha2_authentication, the default authentication method for MySQL 8. For that reason, when creating database users for PHP applications on MySQL 8, you'll need to make sure they're configured to use mysql_native_password instead.]

Your MySQL server is now installed and secured. Next, we will install PHP, the final component in the LAMP stack.
