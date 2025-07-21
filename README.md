<IfModule mod_ssl.c>
<VirtualHost *:443>
    ServerAdmin webmaster@localhost
    ServerName test.mahadevjana.dev
    DocumentRoot /var/www/html/test

    # Add the same flexible block here
    <Directory /var/www/html/test>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    # ... all your SSLCertificateFile lines and other config ...
    # ... leave them as they are ...

</VirtualHost>
</IfModule>
