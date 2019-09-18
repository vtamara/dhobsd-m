---

title: Wordpress en adJ
date: 2019-09-14 00:58 UTC
tags: 

---

# 1. Descargar, descomprimir y ubicar fuentes de Wordpress

De <https://es.wordpress.org> descargue la versíon más reciente.

        cd ~/servidor
        ftp https://es.wordpress.org/latest-es_ES.tar.gz

Descomprima y copie en el directorio destino (tipicamente en `/var/www/htdocs/`)

        tar xvfz latest-es_ES.tar.gz
        doas mv wordpress /var/www/htdocs/misitio

# 2. Crear base de datos en MariaDB con usuario 

En este ejemplo se crearía la base `mibase` el usuario es `miusuario` con clave `micontraseña`:

        mysql -u root -p --socket=/var/www/var/run/mysql/mysql.sock
        MariaDB [(none)]> CREATE DATABASE mibase;
        MariaDB [(none)]> GRANT ALL PRIVILEGES ON mibase.* TO miusuario@localhost IDENTIFIED BY 'micontraseña';

Después puede probar ingresar a esa base como el usuario:

        mysql -u miusuario -p --socket=/var/www/var/run/mysql/mysql.sock mibase


# 3. Configure fuentes

Ubiquese en el directorio del sitio y copie el ejempo de archivo wp-config.php:

        cd /var/www/htdocs/misitio
        doas cp wp-config-sample.php wp-config.php

Editelo para cambiar algunas variables, por ejemplo:

        doas vim wp-config.php

El objetivo es  cambiar 5 datos necesarios en las variables

        define('DB_NAME', 'mibase');

        define('DB_USER', 'miusuario');

        define('DB_PASSWORD', 'micontraseña');

        define('DB_HOST', 'localhost'); //dirección IP o nombre de máquina donde corre la base

        $table_prefix = 'wp_';

Así como las frases aleatorias, 


# 4. Ajuste permisos para asegurar el sitio

A nivel de archivos es necesario cambiar el dueño de los archivos y directorios del sitio 
a quien lo vaya a administrar a nivel de sistema operativo, 
asi mismo el grupo debe ser www en el caso de adJ. Por ejemplo

        doas chown -R jperez:www /var/www/htdocs/misitio

Por seguridad, deben establecerse permisos para permitir lectura a www pero 
minimizar posiblidad de escritura a lo indispensable 
(buscando evitar que un posible pueda escribir en el sistema de archivos).  
Por eso sugerimos agregar al usuario administrador 
(en este ejemplo jperez) al grupo `www` editando `/etc/groups`
y poniendolo al final de la línea que empiza con `www`

Y después ajuste permisos del sitio:

        doas chmod -R go-w /var/www/htdocs/misitio
        doas chmod -R g+w /var/www/htdocs/misitio/wp-content


# 5. Configure `nginx` y cortafuegos

El siguiente ejemplo por añadir a `/etc/nginx.conf`
emplearía <https://misitio.org:26443>

```nginx
   server {
            server_name misitio.org www.misitio.org;
            listen 26443 ssl;
            ssl_certificate /etc/ssl/misitio.org.crt;
            ssl_certificate_key /etc/ssl/private/misitio.org.key;
            error_log logs/misitio-error.log;
            access_log logs/misitio-access.log;

            ssl_protocols TLSv1.2;
            ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-
SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA3
84:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DH
E-DSS-AES256-SHA:DHE-RSA-AES256-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!3DES:!MD5:!PSK';
            ssl_prefer_server_ciphers on;
            ssl_dhparam "/etc/ssl/dhparams.pem";

            root         /htdocs/misitio.org/;
            index index.php;

            location = /favicon.ico {
                    log_not_found off;
                    access_log off;
            }

            location = /robots.txt {
                    allow all;
                    log_not_found off;
                    access_log off;
            }

            location ~ ^(.+\.php)$ {
                    root /htdocs/misitio.org/;
                    try_files $uri =404;
                    fastcgi_intercept_errors on;
                    include        fastcgi_params;
                    fastcgi_param  SERVER_NAME      $host;
                    fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
                    fastcgi_index  index.php;
                    fastcgi_pass   unix:/var/run/php-fpm.sock;
            }

    }
```

Reinicie nginx:

        doas rcctl restart nginx

Si es el caso configure el cortafuegos para permitir usar el puerto, por ejemplo 
en `/etc/pf.conf`:

        pass in on $ext_if proto tcp to ($ext_if) port 25443,26443} keep state

Y reinicie PF:

        doas pfctl -f /etc/pf.conf

# 6. Complete instalación

Ingrese al sitio con un navegador y complete la instalación, pedirá tiítulo para el sitio, primer usuario administrador y su clave.


# 7. Facilite administración con `wp-cli`

Después es recomdable que instale wp-cli para realizar operacoines administrativas desde la línea de ordenes.

Por ejemplo si un plugin se llama fg-spip-to-wp, lo instala con:

        ./wp-cli.phar plugin install fg-spip-to-wp


