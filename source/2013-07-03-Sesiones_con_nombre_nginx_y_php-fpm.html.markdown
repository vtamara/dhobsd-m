---
title: Sesiones_con_nombre_nginx_y_php-fpm
date: 2013-07-03
tags:
---
En OpenBSD 5.3 con nginx 1.2.6 y php-fpm 3.2.1 no operan bien sesiones con algunos nombre cuando no se configura bien nginx.


!Problema

Guarde el siguiente programa php en dos archivos digamos ps.php y ps2.php en una ruta utilizable por el servidor web (digamos ```/var/www/htdocs``` en una configuración por defecto):
<pre>
<?php

$ns = "psession";
if (session_name() != $ns) {
        session_name($ns);
        session_start();
        $_SESSION!['cuenta'] = !isset($_SESSION!['cuenta']) ? 0 :
                $_SESSION!['cuenta'] + 1;
        echo "Cuenta: " . $_SESSION !['cuenta'];
}

?>
</pre>


Si carga ```ps.php``` notará como la cuenta va aumentando, si empieza a cargar ```ps2.php``` la cuenta empezará en 0 y continuará aumentando mientras siga cargando el mismo archivo, pero nuevamente regresará a 0 si alterna el script.   Este comportamiento no ocurre por ejemplo con Apache y tampoco ocurre con el mismo nginx y php-fpm si no emplea sesiones con nombre.

!Solución

Encontramos que el problema era en el orden de directivas en nginx, no opera bien:
<pre>
        location /sivelborde/ {
            root /htdocs/;
        }

        location ~ ^/sivelborde/(.+\.php)$ {
            root /htdocs/;
            #alias      /htdocs/sivelborde/$1;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include        fastcgi_params;
        }
</pre>

Pero si funciona intercambiando el orden:
<pre>
        location ~ ^/sivelborde/(.+\.php)$ {
            root /htdocs/;
            #alias      /htdocs/sivelborde/$1;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include        fastcgi_params;
        }

        location /sivelborde/ {
            root /htdocs/;
        }
</pre>

! Problema 2

El mismo problema ocurre si se utiliza alias en lugar de root
