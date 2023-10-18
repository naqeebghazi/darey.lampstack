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

You have Nginx installed to serve your content and MySQL installed to store and manage your data. Now you can install PHP to process code and generate dynamic content for the web server.

While Apache embeds the PHP interpreter in each request, Nginx requires an external program to handle PHP processing and act as a bridge between the PHP interpreter itself and the web server. This allows for a better overall performance in most PHP-based websites, but it requires additional configuration. You’ll need to install php-fpm, which stands for “PHP fastCGI process manager”, and tell Nginx to pass PHP requests to this software for processing. Additionally, you’ll need php-mysql, a PHP module that allows PHP to communicate with MySQL-based databases. Core PHP packages will automatically be installed as dependencies.

To install these 2 packages at once, run:

$ sudo apt install php-fpm php-mysql
When prompted, type Y and press ENTER to confirm installation.

You now have your PHP components installed.

## Configure nginx to use PHP

Create the root web directory for your_domain as follows:

$ sudo mkdir /var/www/projectLEMP

Assign ownership of the directory with the $USER environment variable, which will reference your current system user:

$ sudo chown -R $USER:$USER /var/www/projectLEMP

Open a new configuration file in Nginx’s sites-available directory using your preferred command-line editor. Here, we’ll use nano:

$ sudo nano /etc/nginx/sites-available/projectLEMP

Paste this into the new file:

#/etc/nginx/sites-available/projectLEMP

server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}

![etcnginx_serverCode](https://github.com/naqeebghazi/darey.LEMPstack/blob/main/images/etcnginx_serverCode.png?raw=true)


Here’s what each of these directives and location blocks do:

listen — Defines what port Nginx will listen on. In this case, it will listen on port 80, the default port for HTTP.
root — Defines the document root where the files served by this website are stored.
index— Defines in which order Nginx will prioritize index files for this website. It is a common practice to list index.html files with a higher precedence than index.php files to allow for quickly setting up a maintenance landing page in PHP applications. You can adjust these settings to better suit your application needs.
server_name — Defines which domain names and/or IP addresses this server block should respond for. Point this directive to your server’s domain name or public IP address.
location / — The first location block includes a try_files directive, which checks for the existence of files or directories matching a URI request. If Nginx cannot find the appropriate resource, it will return a 404 error.
location ~ \.php$ — This location block handles the actual PHP processing by pointing Nginx to the fastcgi-php.conf configuration file and the php7.4-fpm.sock file, which declares what socket is associated with php-fpm.
location ~ /\.ht— The last location block deals with .htaccess files, which Nginx does not process. By adding the deny all directive, if any .htaccess files happen to find their way into the document root ,they will not be served to visitors.
When you’re done editing, save and close the file. If you’re using nano, you can do so by typing CTRL+X and then y and ENTER to confirm.

Activate your configuration by linking to the config file from Nginx’s sites-enabled directory:

sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
This will tell Nginx to use the configuration next time it is reloaded. You can test your configuration for syntax errors by typing:

$ sudo nginx -t
You shall see following message:

    nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    nginx: configuration file /etc/nginx/nginx.conf test is successful

If any errors are reported, go back to your configuration file to review its contents before continuing.

We also need to disable default Nginx host that is currently configured to listen on port 80, for this run the following. This may not be necessary if the default file is not present. 

$ sudo unlink /etc/nginx/sites-enabled/default

When you are ready, reload Nginx to apply the changes:

$ sudo systemctl reload nginx

![nginxcomplete](https://github.com/naqeebghazi/darey.LEMPstack/blob/main/images/nginxcomplete.png?raw=true)

Your new website is now active, but the web root /var/www/projectLEMP is still empty. Create an index.html file in that location so that we can test that your new server block works as expected:

$ sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html

If you get a permission denied like below, ensure the projectLEMP directory does not have an index.html file and then run the command again.

![sudoechoindex](https://github.com/naqeebghazi/darey.LEMPstack/blob/main/images/sudoechoIndexhtml.png?raw=true)

Now go to your browser and try to open your website URL using IP address:

http://<Public-IP-Address>:80
If you see the text from ‘echo’ command you wrote to index.html file, then it means your Nginx site is working as expected.
In the output you will see your server’s public hostname (DNS name) and public IP address. You can also access your website in your browser by public DNS name, not only by IP – try it out, the result must be the same (port is optional)
![browserURLnginx](https://github.com/naqeebghazi/darey.LEMPstack/blob/main/images/browserURLnginx.png?raw=true)

http://<Public-DNS-Name>:80
You can leave this file in place as a temporary landing page for your application until you set up an index.php file to replace it. Once you do that, remember to remove or rename the index.html file from your document root, as it would take precedence over an index.php file by default.

![DNSbrowser](https://github.com/naqeebghazi/darey.LEMPstack/blob/main/images/DNSbrowser.png?raw=true)

Your LEMP stack is now fully configured. In the next step, we’ll create a PHP script to test that Nginx is in fact able to handle .php files within your newly configured website.

## Testing PHP with Nginx

LEMP stack is now fully set up. 

To test/validate that nginx can corrcetly handle .php files do this:

- Use vim to create a info.php file in the /var/www/projectLEMP/ directory.
- Paste into the info.php file:
    <?php
    phpinfo();

- Access this php file in your web browser:
    http://`server_domain_or_IP`/info.php

You will see this page with detailed information about your server, thus demonstrating a successful test:

![phpPagebrowser](https://github.com/naqeebghazi/darey.LEMPstack/blob/main/images/phpPagenginx.png?raw=true)

Once confirmed, delete the info.php file as keeping it poses a security risk due to the server information it has on it. Use this command:

$ sudo rm /var/www/your_domain/info.php


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





