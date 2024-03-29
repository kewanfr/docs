# Installation de PhpMyAdmin

## Prérequis

Avoir un domaine ou sous-domaine sur lequel on souhaite mettre le panel phpmyadmin qui point vers notre machine. Créer un enregistrer A ou CNAME, par exemple: pma.domaine.fr.

* Nginx (déjà installé si vous utiliser un panel comme pterodactyl)
* Cerbot

(Installation de certbot):

```
sudo apt update
sudo apt install -y certbot
sudo apt install -y python3-certbot-nginx
```

## Installation du panel

### Etape 1

On commence par créer le dossier, télécharger et mettre les fichiers sources de pma dans le dossier de nginx.

```
mkdir /var/www/phpmyadmin && mkdir /var/www/phpmyadmin/tmp/ && cd /var/www/phpmyadmin
wget https://files.phpmyadmin.net/phpMyAdmin/5.2.1/phpMyAdmin-5.2.1-all-languages.tar.gz
tar xvzf phpMyAdmin-5.2.1-all-languages.tar.gz
mv phpMyAdmin-5.2.1-all-languages/* /var/www/phpmyadmin
```

### Etape 2

```
chown -R www-data:www-data * 
mkdir config
chmod o+rw config
cp config.sample.inc.php config/config.inc.php
chmod o+w config/config.inc.php
```

## Configuration du serveur web

### Certficat SSL

Il faut tout d'abord générer le certificat ssl avec certbot:

```
certbot certonly
```

On choisit la 1ère option, puis on rentre notre domaine, par exemple: pma.domaine.fr

### Configuration NGINX

Créer le fichier avec `nano /etc/nginx/sites-available/phpmyadmin.conf`

Remplacer \<domain> par votre nom de domaine

<pre data-title="phpmyadmin.conf" data-line-numbers><code>server {
    listen 80;
<strong>    server_name &#x3C;domain>;
</strong>    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
<strong>    server_name &#x3C;domain>;
</strong>
    root /var/www/phpmyadmin;
    index index.php;

    # allow larger file uploads and longer script runtimes
    client_max_body_size 100m;
    client_body_timeout 120s;

    sendfile off;

    # SSL Configuration
<strong>    ssl_certificate /etc/letsencrypt/live/&#x3C;domain>/fullchain.pem;
</strong><strong>    ssl_certificate_key /etc/letsencrypt/live/&#x3C;domain>/privkey.pem;
</strong>    ssl_session_cache shared:SSL:10m;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256';
    ssl_prefer_server_ciphers on;

    # See https://hstspreload.org/ before uncommenting the line below.
    # add_header Strict-Transport-Security "max-age=15768000; preload;";
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Robots-Tag none;
    add_header Content-Security-Policy "frame-ancestors 'self'";
    add_header X-Frame-Options DENY;
    add_header Referrer-Policy same-origin;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param PHP_VALUE "upload_max_filesize = 100M \n post_max_size=100M";
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param HTTP_PROXY "";
        fastcgi_intercept_errors off;
        fastcgi_buffer_size 16k;
        fastcgi_buffers 4 16k;
        fastcgi_connect_timeout 300;
        fastcgi_send_timeout 300;
        fastcgi_read_timeout 300;
        include /etc/nginx/fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }
}
</code></pre>

On enregistre le fichier et on applique la configuration

```
# You do not need to symlink this file if you are using CentOS.
sudo ln -s /etc/nginx/sites-available/phpmyadmin.conf /etc/nginx/sites-enabled/phpmyadmin.conf

systemctl restart nginx
```

## Configurations supplémentaires

Pour que pma fonctionne correctement, il faut encore effectuer quelques commandes

```
cp /var/www/phpmyadmin/config/config.inc.php /var/www/phpmyadmin
```

Afin d'activer la connexion avec les cookies, il faut ouvrir et éditer le fichier `/var/www/phpmyadmin/config.inc.php` ainsi que remplir la clé `blowfish_secret` avec une chaine de texte de 32 caractères.

```
rm -rf /var/www/phpmyadmin/config
rm -rf /var/www/phpmyadmin/setup
```

