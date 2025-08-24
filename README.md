# ✅ Ubuntu LAMP Stack + phpMyAdmin + SSL + Laravel + Reverb Setup

This README provides a step-by-step guide to set up a complete development or production server with Apache, MySQL, PHP, phpMyAdmin, SSL, Laravel deployment, file permissions, and Reverb WebSocket support.

---

## 🔧 1. System Update

```bash
sudo apt-get update
```
```bash
sudo apt upgrade
```

## 🛠️ 2. Install MySQL

```bash
sudo apt-get install mysql-server
```

## 🔐 3. Secure MySQL

```bash
sudo mysql_secure_installation
```

## 🌐 4. Install Apache

```bash
sudo apt-get install apache2
```

## ⚙️ 5. Install PHP

```bash
sudo apt-get install php libapache2-mod-php
```

## 🔁 6. Restart Apache

```bash
sudo systemctl restart apache2
```

## 🧰 7. Install phpMyAdmin

```bash
sudo apt-get install phpmyadmin php-mbstring
```

## 🔗 8. Enable phpMyAdmin Config

```bash
sudo a2enconf phpmyadmin.conf
```

## 🛠️ 9. Fix if phpMyAdmin is not working

```bash
sudo ln -s /etc/phpmyadmin/apache.conf /etc/apache2/conf-available/phpmyadmin.conf
sudo a2enconf phpmyadmin.conf
```

## 🔁 10. Restart Apache Again

```bash
sudo systemctl restart apache2
```

## 🔑 11. Login to MySQL

```bash
sudo mysql
```

## 🔐 12. Change MySQL Root Password for phpMyAdmin

```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'StrongP@ssw0rd!';
```

## 🚫 13. Disable phpMyAdmin

```bash
sudo a2disconf phpmyadmin.conf
sudo systemctl restart apache2
```

---

## 📁 14. File Permissions
```bash
sudo chown -R www-data:www-data /var/lib/phpmyadmin
```
```bash
sudo chmod 733 /var/lib/phpmyadmin/
```
```bash
sudo chown -R ubuntu:www-data /var/www
```
```bash
sudo chown -R www-data:www-data /var/lib/phpmyadmin
```

## 📁 15. Laravel Project File Permissions

```bash
sudo chown -R ubuntu:www-data /var/www/html/test
sudo find /var/www/html/test -type d -exec chmod 775 {} \;
sudo find /var/www/html/test -type f -exec chmod 664 {} \;
```

---

## 🔍 16. Locate phpMyAdmin DB config

```bash
sudo nano /etc/phpmyadmin/config-db.php
```

Example contents:

```php
$dbuser='root';
$dbpass='Mahadev@2580369';
$dbserver='localhost';
$dbname='phpmyadmin';
```

Then update password in MySQL:

```bash
sudo mysql -u root -p
```

```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Mahadev@2580369';
FLUSH PRIVILEGES;
EXIT;
```

---

## 🔐 17. Install Free SSL with Certbot

```bash
sudo apt install certbot python3-certbot-apache
sudo certbot --apache -d test.domain.com -d www.test.domain.com
```

## 🔄 18. Show Hidden Files

```bash
echo "alias ls='ls -a'" >> ~/.bashrc
source ~/.bashrc
```

## 🧹 19. Delete All Files and Folders (Use with caution)

```bash
rm -rf .[^.]* *
```

---

## 🌐 20. HTTP Domain Configration

```apache
sudo nano /etc/apache2/sites-available/api.campaigningtool.com.conf
```

```apache
<VirtualHost *:80>
    ServerAdmin janamahadev14@gmail.com
    ServerName admin.campaigningtool.com
    DocumentRoot /var/www/admin.campaigningtool.com

    <Directory /var/www/admin.campaigningtool.com>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    RewriteEngine on
    RewriteCond %{SERVER_NAME} =admin.campaigningtool.com
    RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
</VirtualHost>
<IfModule mod_ssl.c>
<VirtualHost *:443>
    ServerAdmin janamahadev14@gmail.com
    ServerName admin.campaigningtool.com
    DocumentRoot /var/www/admin.campaigningtool.com

    <Directory /var/www/admin.campaigningtool.com>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    # Reverb WebSocket Proxy Setup
    RewriteEngine On
    RewriteCond %{HTTP:Upgrade} =websocket [NC]
    RewriteRule "^/?app/(.*)" "ws://127.0.0.1:8080/app/$1" [P,L]
    ProxyPassReverse "/app/" "ws://127.0.0.1:8080/app/"

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    SSLCertificateFile /etc/letsencrypt/live/admin.campaigningtool.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/admin.campaigningtool.com/privkey.pem
    Include /etc/letsencrypt/options-ssl-apache.conf
</VirtualHost>
</IfModule>
```

---

## ✅ Enable Required Apache Modules

```bash
sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_http
sudo a2enmod proxy_wstunnel
sudo apachectl configtest
sudo systemctl restart apache2
```

---

## 🎼 21. Composer Installation

```bash
sudo apt update
sudo apt install composer -y
composer --version
```

In Laravel project:

```bash
composer install
```

---

## 🧠 22. PHP Settings (php.ini)

Edit PHP config (usually in `/etc/php/7.x/apache2/php.ini`):

```ini
upload_max_filesize = 1024M
post_max_size = 1024M
memory_limit = 128M
max_execution_time = 300
max_input_time = 300
```

---

## 🗂 23. Apache Root Directory Override

Edit `/etc/apache2/apache2.conf`:

```apache
<Directory /var/www/>
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>
```

---

## 🔥 24. Delete SSL Certificate (if needed)

```bash
sudo rm -r /etc/letsencrypt/live/mahadevjana.dev
sudo rm -r /etc/letsencrypt/archive/mahadevjana.dev
sudo rm /etc/letsencrypt/renewal/mahadevjana.dev.conf
```

---

## 🔥 25. Apache Configration
When Apache is installed, the /etc/apache2/sites-available folder contains two default configuration files: **000-default.conf** and **default-ssl.conf**. These files are responsible for handling the default virtual hosts.
Before setting up a new domain with Certbot, it's recommended to first disable and delete these default site files to avoid conflicts:

🔧 Disable Sites
```bash
sudo a2dissite file_name.conf
sudo systemctl reload apache2
```

🔧 Enable Sites
```bash
sudo a2ensite file_name.conf
sudo systemctl reload apache2
```
---

## 🔥 26. Enable header in server
```bash
sudo a2enmod headers
```
## 🔥 27. Apache Configuration: Block Direct IP Access
Purpose:This configuration ensures that any requests made directly to the server’s IP address (instead of a configured domain name) are blocked or redirected.
```bash
<VirtualHost *:80>
    # Catch-all for HTTP requests without a matching ServerName
    ServerName _default_
    RedirectMatch 504 ^/(.*)$
</VirtualHost>

<VirtualHost *:443>
    # Catch-all for HTTPS requests without a matching ServerName
    ServerName _default_
    DocumentRoot /var/www/default

    # Enable SSL
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/default.crt
    SSLCertificateKeyFile /etc/ssl/private/default.key

    # Restrict access to the default directory
    <Directory /var/www/default>
        Options -Indexes
        Require all denied
    </Directory>

    # Send HTTP 504 for all direct IP requests
    RedirectMatch 504 ^/(.*)$
</VirtualHost>
```

## 🔥 28. Apache Supervisor control command

| Command                                | Description                                  |
| -------------------------------------- | -------------------------------------------- |
| `sudo supervisorctl status`            | Shows all managed processes and their status |
| `sudo supervisorctl start <process>`   | Start a process                              |
| `sudo supervisorctl stop <process>`    | Stop a process                               |
| `sudo supervisorctl restart <process>` | Restart a process                            |
| `sudo supervisorctl reread`            | Detect new/changed config files              |
| `sudo supervisorctl update`            | Reload Supervisor with new configs           |
| `sudo service supervisor start`        | Start Supervisor service                     |
| `sudo service supervisor restart`      | Restart Supervisor service                   |
| `sudo service supervisor stop`         | Stop Supervisor service                      |


✅ All done. Your Laravel app with MySQL, Apache, SSL, and Reverb is now fully configured and secure.
