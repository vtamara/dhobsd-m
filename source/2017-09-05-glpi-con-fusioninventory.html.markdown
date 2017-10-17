---

title: Instalación y uso de GLPI con Fusion Inventory
date: 2017-09-05 10:45 UTC
tags: 

---

GLPI es un sistema para manejar solicitudes de servicio realizadas a una 
Oficina de Tecnologías de la Información.

Fusion Inventory es un agente con versiones para diversos sistemas operativos 
(incluyuendo Andoid, Mac, Linux, *BSD y Windows) que se instala en los 
computadores de una red y que envía el inventario del computador a GLPI, 
para automatizar la labor de inventariado.

 
# 1. Instalación de GLPI en adJ

Instale mariadb como se describe en 
<http://pasosdejesus.github.io/servidor_adJ/mariadb.html>

Cree una usuario, una base de datos y una clave para que ese usuario maneje 
la base de datos.

Instale el paquete php-mysqli (descargue uno reciente para su adJ reciente 
por ejemplo de http://adj.pasosdejesus.org/pub/AprendiendoDeJesus/6.1-amd64-paquetes-extra/ remplazando 6.1 por su versión )

Descargue glpi de <http://www.glpi-project.org/spip.php?article41>, 
descomprima y mueva fuentes a `/var/www/htdocs/`

Configure nginx para ingresar por SSL con subdirectorio glpi 
(parte de la siguiente configuración proviene de
<https://github.com/remicollet/remirepo/blob/master/glpi/glpi/glpi-nginx.conf>
):
 
<code>
    server {
        server_name 192.168.1.9 192.168.10.2;

        root /htdocs/;
        listen       443;
        index        index.php index.html index.htm;
        client_max_body_size 400m;
        ssl                  on;
        ssl_certificate      /etc/ssl/server.crt;
        ssl_certificate_key  /etc/ssl/private/server.key;

        location = /glpi {
                alias /var/www/htdocs/glpi/;
        }

        location /glpi/ {
                root /htdocs/;
                index index.php;

                location ~ ^/glpi/files/(.+)$ {
                        deny all;
                }
                location ~ ^/glpi/config/(.+)$ {
                        deny all;
                }
                location ~ ^/glpi/scripts/(.+)$ {
                        deny all;
                }
                location ~ ^/glpi/locales/(.+)$ {
                        deny all;
                }
                location /glpi/install/mysql {
                        deny all;
                }

                location ~ ^/glpi/install/(.+\.php)$ {
                        allow 127.0.0.1;
                        allow ::1;
                        deny all;

                        try_files $uri =404;
                        fastcgi_intercept_errors on;
                        include        fastcgi_params;
                        fastcgi_param  SERVER_NAME      $host;
                        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
                        fastcgi_pass   unix:run/php-fpm.sock;
                }

                location ~ ^/glpi/(.+\.php)$ {
                        try_files $uri =404;
                        fastcgi_intercept_errors on;
                        include        fastcgi_params;
                        fastcgi_param  SERVER_NAME      $host;
                        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;

                        fastcgi_pass   unix:run/php-fpm.sock;
                }
        }
    }
</code>

Dar permiso de escritura a los directorios config y files:

<code data-language="sh">
chown www:www config
chown -R www:www files
</code>

Reinicie nginx y examine el URL del servidor en directorio glpi.

Por ejemplo https://192.168.1.5/glpi/

Siga las instrucciones de instalación en pantalla.

Una vez termine renombre el directorio install (digamos install-ya)

## 1.1. Configuración de GLPI

* Cree usuarios del departamento de TI
* Cree grupos de la organización con su jerarquía.
* Crear lugares con oficinas

## 1.2 Actualización

Saque una copia de respaldo tanto de las fuentes como de la base de datos.

<code>
mysqldump -h 192.168.10.21 -u glpi -p glpi > copia/2017-07-26.mysql
</code>

### 1.2.1 Actualizar sitio de ensayo

<code>
cd tiensayo
</code>

Eliminar base anterio y vovler a crear, desde 192.168.10.21:

<code>
 mysql -u root -p

> create database glpiensayo;

> GRANT ALL PRIVILEGES ON glpiensayo.* to glpi@192.168.10.11 identified by 'majbookpye'; 
</code>

Poblar con la copia recién hecha


<code>
mysql -u glpi -h 192.168.10.21 -p glpiensayo < ../ti/copia/2017-07-26.mysql
</code>

Probar desde <https://miong.org.co/tiensayo/>

Renombre el directorio `install-ya` a `install`

Actualice fuentes sobreescribiendolas con nueva versión y eliminando lo 
que no usará (usando git es sencillo, suponiendo que se hizo bifurcación del 
repositorio y está en origin mientras que el principal está en upstream)

<code>
git checkout master
git pull --rebase upstream master
git push origin master
git checkout 9.1/bugfixes
git pull upstream 9.1/bugfixes
git push origin 9.1/bugfixes
</code>

Ajustar permisos de config:

<code>
doas chown www config
doas chmod g+w config
</code>

Intente ingresar a GLPI desde la interfaz web, esto revisará instalación 
y actualizará base de datos tras algunas confirmaciones.

Vuelva a mover `install` a `install-ya` y borre `install-ya/install.php`

Vuelva a a poner permisos seguros a `config`

<code>
chmod g-w config
</code>

Actualice plugins y activelos 


# 2. Integración con Fusion Inventory

Fusion Inventory se debe comunicar periodicamente con el servidor de GLPI 
para mantener actualizado el inventario.  Primero debe instalarse el plugin 
en GLPI y después configurarse como tarea periódica.

Para comunicarse con GLPI, Fusion Inventory requiere una URL en el servidor 
de GLPI.  Esta URL puede verla el super-administrador desde el menú 
Administración->Entity y allí en la pestaña FusionInventory.


# 3. Referencias

Documentción oficial de GLPI: http://glpi-project.org/spip.php?rubrique18


