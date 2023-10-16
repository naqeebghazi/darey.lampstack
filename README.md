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

![Image](https://github.com/naqeebghazi/darey.lampstack/blob/main/images/MySQLserverinstallation.png?raw=true)

There is a password for logging into MySQL and a passowrd as a root user within MySQL. The prompts that follow are for both of these:

Once complete, log back into MySQL:
$ sudo mysql -p

![Image](https://github.com/naqeebghazi/darey.lampstack/blob/main/images/MySQLloginMessage.png?raw=true)

-p flag will prompt the user for the password used after changing the root user password

Notice that you need to provide a password to connect as the root user.
For increased security, it's best to have dedicated user accounts with less expansive privileges set up for every database, especially if you plan on having multiple databases hosted on your server.

[Note: At the time of this writing, the native MySQL PHP library mysqlnd doesn't support caching_sha2_authentication, the default authentication method for MySQL 8. For that reason, when creating database users for PHP applications on MySQL 8, you'll need to make sure they're configured to use mysql_native_password instead.]

Your MySQL server is now installed and secured. Next, we will install PHP, the final component in the LAMP stack.

## Install PHP

You'll need the following components when installing PHP:
- PHP 
- PHP-MySQL: allows phph to communicate with mysql databases
- LibApache2-mod-PHP: enables apache to handle phph files

To install all 3 packages at once:
$ sudo apt install php libapache2-mod-php php-mysql

Check php version:
$ php -v

This finalises the LAMP stack installation

## Enable PHP on the website

We now need to set up two virtual hosts within pur EC2 instance vm.

With the default DirectoryIndex settings on Apache, a file named index.html will always take precedence over an index.php file.
This is useful for setting up maintenance pages in PHP applications, by creating a temporary index.html file containing an informative message to visitors. Because this page will take precedence over the index.php page, it will then become the landing page for the application. Once maintenance is over, the index.html is renamed or removed from the document root, bringing back the regular application page.
In case you want to change this behavior, you'll need to edit the /etc/apache2/mods-enabled/dir.conf file and change the order in which the index.php file is listed within the DirectoryIndex directive:

$ sudo vim /etc/apache2/mods-enabled/dir.conf

Then in the vim editor, ensure this is whats written (i.e. index.php is prioritised):
<IfModule mod_dir.c>
       DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>

Then, reload Apache:
$ sudo systemctl reload apache2

Optional:
_ Create a PHP script to test PHP is installed and configured on the server. 

$ sudo mkdir /var/www/projectlamp/index.php _

Next, assign ownership of the directory with your current system user:
$ sudo chown -R $USER:$USER /var/www/projectlamp

Apache on Ubuntu 20.04 has one server block enabled by default that is configured to serve documents from the /var/www/html directory.
We will leave this configuration as is and will add our own directory next next to the default one.

Then, create and open a new configuration file in Apache’s sites-available directory using your preferred command-line editor. Here, we’ll be using vi or vim (They are the same by the way):

$ sudo vim /etc/apache2/sites-available/projectlamp.conf

Paste in the following bare-bones configuration by hitting on i on the keyboard to enter the insert mode, and paste the text:

<VirtualHost *:80>
    ServerName projectlamp
    ServerAlias www.projectlamp 
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/projectlamp
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

![Image](https://github.com/naqeebghazi/darey.lampstack/blob/main/images/apache2sitesavailableconf.png?raw=true)

With this VirtualHost configuration, we’re telling Apache to serve projectlamp using /var/www/projectlampl as its web root directory. 

### Enable the Virtual host

You can now use a2ensite command to enable the new virtual host:

$ sudo a2ensite projectlamp

You might want to disable the default website that comes installed with Apache. This is required if you’re not using a custom domain name, because in this case Apache’s default configuration would overwrite your virtual host. To disable Apache’s default website use a2dissite command , type:

$ sudo a2dissite 000-default

To make sure your configuration file doesn’t contain syntax errors, run:

$ sudo apache2ctl configtest

Finally, reload Apache so these changes take effect:

$ sudo systemctl reload apache2

Your new website is now active, but the web root /var/www/projectlamp is still empty. Create an index.html file in that location so that we can test that the virtual host works as expected

$ sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectlamp/index.html

Alternatively, paste the following into a new index.php file in the /var/www/projectlamp/ directory.

Now go to your browser and try to open your website URL using IP address:

http://<Public-IP-Address>:80

![Image](https://github.com/naqeebghazi/darey.lampstack/blob/main/images/phpinBrowser.png?raw=true)

If you see the text from ‘echo’ command you wrote to index.html file, then it means your Apache virtual host is working as expected.
In the output you will see your server’s public hostname (DNS name) and public IP address. You can also access your website in your browser by public DNS name, not only by IP – try it out, the result must be the same (port is optional)

http://<Public-DNS-Name>:80

You can leave this file in place as a temporary landing page for your application until you set up an index.php file to replace it. Once you do that, remember to remove or rename the index.html file from your document root, as it would take precedence over an index.php file by default.





