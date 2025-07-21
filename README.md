Complete Guide for LAMP Stack & Laravel Server Setup on Ubuntu
This guide covers the complete process of setting up a LAMP (Linux, Apache, MySQL, PHP) stack on an Ubuntu server, configured for hosting projects like Laravel. It includes installing phpMyAdmin, securing the server with a free SSL certificate, setting file permissions, and configuring a reverse proxy for Laravel Reverb.
1. Initial Server Update
Always start by updating your package lists to ensure you have the latest versions and security patches.
Generated bash
sudo apt-get update
Use code with caution.
Bash
2. Install the LAMP Stack
Install Apache, MySQL, and PHP along with the necessary module for Apache to handle PHP files.
Generated bash
# Install Apache Web Server
sudo apt-get install apache2 -y

# Install MySQL Database Server
sudo apt-get install mysql-server -y

# Install PHP and the Apache PHP Module
sudo apt-get install php libapache2-mod-php -y
Use code with caution.
Bash
Restart Apache
After installation, restart Apache to ensure all services are running correctly.
Generated bash
sudo systemctl restart apache2
Use code with caution.
Bash
3. Secure MySQL Installation
Run the included security script to remove insecure defaults and lock down access to your database system. You will be prompted to set a root password, remove anonymous users, disable remote root login, etc.
Generated bash
sudo mysql_secure_installation
Use code with caution.
Bash
4. Install and Configure phpMyAdmin
phpMyAdmin is a web-based interface for managing your MySQL databases.
Install phpMyAdmin
The installer will ask you a few questions.
When prompted, select apache2 by pressing the spacebar, then press Enter.
Select Yes to configure the database for phpMyAdmin with dbconfig-common.
Generated bash
sudo apt-get install phpmyadmin php-mbstring -y
Use code with caution.
Bash
Enable phpMyAdmin Configuration
Enable the newly installed phpMyAdmin configuration for Apache and restart the server.
Generated bash
sudo a2enconf phpmyadmin.conf
sudo systemctl restart apache2
Use code with caution.
Bash
Troubleshooting: If the a2enconf command gives an error, the symbolic link may not have been created correctly. You can create it manually and try again.
Generated bash
sudo ln -s /etc/phpmyadmin/apache.conf /etc/apache2/conf-available/phpmyadmin.conf
sudo a2enconf phpmyadmin.conf
sudo systemctl restart apache2
Use code with caution.
Bash
Fix phpMyAdmin Login Issues (Authentication Method)
If you can't log in to phpMyAdmin with the root user, it's likely because MySQL 8+ uses a different authentication plugin. Change it back to the one phpMyAdmin expects.
Log in to the MySQL shell.
Generated bash
sudo mysql
Use code with caution.
Bash
Run the following SQL command, replacing StrongP@ssw0rd! with the password you want to use for the root user.
Generated sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'StrongP@ssw0rd!';
FLUSH PRIVILEGES;
EXIT;
Use code with caution.
SQL
5. Set Up Project File Permissions
Correct file permissions are crucial for security and for the web server to read/write files correctly (e.g., for uploads or caching).
General Web Root Permissions
Set ownership of the main web directory to your user (ubuntu in this case) and the www-data group.
Generated bash
sudo chown -R ubuntu:www-data /var/www/html
Use code with caution.
Bash
Project-Specific Permissions
For a specific project (e.g., a Laravel project at /var/www/html/test), set the correct ownership and permissions. This allows your user to manage files while giving the web server the access it needs.
Generated bash
# Set ownership for the project directory
sudo chown -R ubuntu:www-data /var/www/html/test

# Set directory permissions to 775 (rwxrwxr-x)
sudo find /var/www/html/test -type d -exec chmod 775 {} \;

# Set file permissions to 664 (rw-rw-r--)
sudo find /var/www/html/test -type f -exec chmod 664 {} \;
Use code with caution.
Bash
6. Configure Apache for a Domain/Subdomain
To host a site, create an Apache Virtual Host file. This tells Apache where to find the site's files and which domain it should respond to.
Create the Virtual Host File
Generated bash
sudo nano /etc/apache2/sites-available/your_domain.com.conf
Use code with caution.
Bash
Paste the following configuration. Replace your_domain.com with your domain and /var/www/html/test with your project's root directory. The AllowOverride All directive is essential for .htaccess files (like Laravel's) to work.
Generated apacheconf
<VirtualHost *:80>
    ServerName your_domain.com
    ServerAlias www.your_domain.com
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html/test

    <Directory /var/www/html/test>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
Use code with caution.
Apacheconf
Enable the New Site and Restart Apache
Generated bash
# Enable the new site configuration
sudo a2ensite your_domain.com.conf

# Enable Apache's rewrite module (essential for pretty URLs)
sudo a2enmod rewrite

# Test the configuration for errors
sudo apachectl configtest

# Restart Apache to apply all changes
sudo systemctl restart apache2
Use code with caution.
Bash
7. Secure Your Site with a Free SSL Certificate (Let's Encrypt)
Use Certbot to automatically obtain and install a free SSL certificate.
Generated bash
# Install Certbot and the Apache plugin
sudo apt install certbot python3-certbot-apache -y

# Run Certbot to get and install the certificate
# Replace the domains with your own. Certbot will automatically update your Apache config.
sudo certbot --apache -d your_domain.com -d www.your_domain.com
Use code with caution.
Bash
Certbot will create a new SSL configuration file (e.g., your_domain.com-le-ssl.conf), enable it, and set up an automatic redirect from HTTP to HTTPS.
8. Configure Apache for Laravel Reverb (Reverse Proxy)
If you are using Laravel Reverb, you need to configure Apache to act as a reverse proxy for the WebSocket connections.
Enable Required Apache Modules
Generated bash
sudo a2enmod proxy
sudo a2enmod proxy_http
sudo a2enmod proxy_wstunnel
sudo a2enmod rewrite
Use code with caution.
Bash
Update Your SSL Virtual Host Configuration
Edit the SSL configuration file that Certbot created.
Generated bash
sudo nano /etc/apache2/sites-available/your_domain.com-le-ssl.conf
Use code with caution.
Bash
The file should look like this. The key is adding the Rewrite rules and the ProxyPassReverse directive.
Generated apacheconf
<IfModule mod_ssl.c>
<VirtualHost *:443>
    ServerName your_domain.com
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html/test

    <Directory /var/www/html/test>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    # ==========================================================
    # START: REVERB REVERSE PROXY CONFIGURATION
    # ==========================================================
    RewriteEngine On
    RewriteCond %{HTTP:Upgrade} =websocket [NC]
    RewriteRule "^/?app/(.*)" "ws://127.0.0.1:8080/app/$1" [P,L]
    ProxyPassReverse "/app/" "ws://127.0.0.1:8080/app/"
    # ==========================================================
    # END: REVERB REVERSE PROXY CONFIGURATION
    # ==========================================================

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    SSLCertificateFile /etc/letsencrypt/live/your_domain.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/your_domain.com/privkey.pem
    Include /etc/letsencrypt/options-ssl-apache.conf
</VirtualHost>
</IfModule>
Use code with caution.
Apacheconf
Test and Restart Apache
Generated bash
sudo apachectl configtest
sudo systemctl restart apache2
Use code with caution.
Bash
9. Server & PHP Configuration Tuning
Install Composer
Install Composer, the dependency manager for PHP, globally.
Generated bash
sudo apt update
sudo apt install composer -y
Use code with caution.
Bash
You can then navigate to your project directory and install its dependencies.
Generated bash
# Check Composer version
composer --version

# Go to your Laravel project directory
cd /var/www/html/test

# Install PHP packages
composer install
Use code with caution.
Bash
Tune PHP for Large File Uploads
To allow large file uploads, you need to edit your php.ini file. The location can vary, but for Apache it's often at /etc/php/VERSION/apache2/php.ini.
Generated bash
# Example for PHP 8.1 - adjust the version number as needed
sudo nano /etc/php/8.1/apache2/php.ini
Use code with caution.
Bash
Find and update the following values.
Generated ini
; Set maximum size of a single uploaded file.
upload_max_filesize = 1024M

; Set maximum size of the entire POST request body.
; Must be greater than or equal to upload_max_filesize.
post_max_size = 1024M

; Maximum amount of time a script is allowed to run.
max_execution_time = 300

; Maximum time to parse input data, like POST and GET.
max_input_time = 300

; Maximum memory a script can consume.
memory_limit = 256M
Use code with caution.
Ini
Restart Apache to apply the PHP changes.
Generated bash
sudo systemctl restart apache2
Use code with caution.
Bash
10. Appendix: Useful & Potentially Destructive Commands
Show Hidden Files (like .env)
To make the ls command always show hidden files (those starting with a dot), you can create an alias.
Generated bash
echo "alias ls='ls -a'" >> ~/.bashrc
source ~/.bashrc
Use code with caution.
Bash
Now, whenever you type ls, it will run as ls -a.
🔥 Cleanly Remove a Let's Encrypt Certificate
If you need to completely remove a certificate to start over, use these commands. This is destructive and should only be done if you plan to issue a new certificate.
Generated bash
# Replace your_domain.com with the actual domain name
sudo rm -r /etc/letsencrypt/live/your_domain.com
sudo rm -r /etc/letsencrypt/archive/your_domain.com
sudo rm /etc/letsencrypt/renewal/your_domain.com.conf
Use code with caution.
Bash
⚠️ DANGER: Delete All Files and Folders in a Directory
This command will permanently delete everything inside the directory where you run it, including all hidden files and folders.
USE WITH EXTREME CAUTION. THERE IS NO UNDO. DOUBLE-CHECK YOUR CURRENT DIRECTORY WITH pwd BEFORE RUNNING.
Generated bash
rm -rf .[^.]* *
