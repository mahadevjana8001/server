```apache
<IfModule mod_ssl.c>
<VirtualHost *:443>
    ServerAdmin webmaster@localhost
    ServerName test.mahadevjana.dev
    DocumentRoot /var/www/html/test

    <Directory /var/www/html/test>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/test.mahadevjana.dev/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/test.mahadevjana.dev/privkey.pem

</VirtualHost>
</IfModule>
