---
title: Instalaci_n_de_Drupal_con_Gallery2
date: 2006-10-02 12:00 UTC
tags:
---
Drupal es un manejador de contenidos bastante popular; Gallery2 permite mantener albums de fotos.  Aunque la licencia de estos programas es GPL, ambos pueden usarse en un sistema OpenBSD para complementar una plataforma de comunicaciones como en:

http://www.cristianosporlapaz.info

Este artículo describe la instalación y configuración de cada uno de estos programas, así como la inmersión de Gallery2 en Drupal.

##Prerequisitos

Servidor web corriendo (suponemos que chroot en ```/var/www```), Servidor PostgreSQL (suponemos socket en ```/var/www/tmp```), PHP5 con los módulos GD y PGSQL.   Hemos probado la configuración descrita en OpenBSD 3.8 con Drupal 4.7.3 y Gallery 2.1.2.


##Instalación de Drupal

Del sitio de drupal (http://www.drupal.org) descargue drupal-4.7.x y descomprimalo en ```/var/www/htdocs/drupal```

Cree un usuario y una base de datos en PostgreSQL:
<pre>
createuser -Ad -h /var/www/tmp drupal
createdb -h /var/www/tmp/ --encoding=UNICODE --owner=drupal drupal
cd /var/www/htdocs/drupal/
psql -h /var/www/tmp/ -f database/database.pgsql drupal drupal
</pre>

Después prepare un sitio en drupal, digamos ```ejemplo.misitio.org```, y edite el archivo ```settings.php```:
<pre>
cd sites
cp -rf default ejemplo.misitio.org
cd ejemplo.misitio.org
vi settings.php
</pre>

Suponiendo que la clave del usuario drupal en PostgreSQL es Drupal!,
modifique por lo menos 
<pre>
$db_url = '!pgsql://drupal:Drupal!@localhost/drupal';
</pre>

!Primeras labores de Configuración de Drupal

Una vez instale podrá ingresar a su sitio desde un navegador y crear la primera cuenta ---con la cual administrará la instalación.

Entre las primeras labores que puede efecutar una vez instalado y funcionando:
* Configurar interfaz en Español:
** En el menu administer/modules active el módulo locale y salve la configuración.
** Descargue la localización para español para su versión de drupal de http://drupal.org/project/Translations y descomprimala (quedará entre otros un archivo es/es.po)
** En el nuevo menu administer/localization elija Import, después proporcione la ubicación de es/es.po 
** Una vez suba el archivo, en la configuración de localización active la traducción a Español y elijala como la traducción por defecto.
* Crear un rol para administradores
** Desde el menu administrar/control de acceso, tabulador "roles"
** Cree un nuevo rol administrador
** En el tabulador "permisos", habilite todas las opciones para el rol administrador.
* Crear usuarios
** En el menú administrar/usuarios en el tabulado "añadir usuario"
* Preparar directorio para que Drupal escriba
** Creelo:
<pre>
mkdir /var/www/archivos-drupal
sudo chown www:www /var/www/archivos-drupal/
</pre>
** Configurelo en el menú admistrar/opciones


##Instalación de Gallery2

Descarge una versión reciente de Gallery 2 de http://gallery.menalto.com/downloads

Descargue la versión completa (full) pues incluye módulos requeridos para la integración con Drupal.



!Configuración de Gallery2

Descomprima en un directorio utilizable desde el web, por ejemplo en
```/var/www/htdocs/gallery2/```

Aliste un directorio para los datos del programa:
<pre>
sudo mkdir /var/www/g2data
chown www:www /var/www/g2data/
</pre>
Aliste una base de datos:
<pre>
sudo su - _postgresql
createuser -h /var/www/tmp -Ad gallery2
createdb -h /var/www/tmp -U gallery2 gallery2 -E UNICODE
</pre>

Después abra el URL correspondiente al sitio de instalación por ejemplo: ```http://localhost/gallery2```

Tendrá que:
* Autenticarse creando un archivo login.txt en el directorio de Gallery con una cadena.
* Ingresar ```/g2data``` como escritorio donde escribir
* Crear un archivo y dar permiso de escritura mientras se completa configuración:
<pre>
touch config.php
chmod 666 config.php
</pre>
* Desactivar el módulo Registro. Asegurarse de tener módulos "Bloques de imagenes" e "Image Frame" y activarlos.
Una vez complete la instalación no olvide:
<pre>
chmod 644 config.php
</pre>


##Inmersión de Gallery2 en Drupal

Descargue el módulo de Drupal para Gallery de:
http://ftp.osuosl.org/pub/drupal/files/projects/gallery-4.7.0.tar.gz
Descomprimalo y copielo al directorio modules de drupal:
<pre>
mv gallery /var/www/htdocs/drupal/modules/
</pre>

Como administrador de Drupal en el menu Administrar/Módulos active el nuevo
módulo que debe aparecer "gallery".

Después podrá configurarlo en el nuevo menú Administrar/Opciones/gallery, basta que como URI de Gallery emplee ```/gallery2/```, que active autodetección y salve la configuración.


##Instalación Multisitio de Drupal

Si varios desea que varios sitios compartan la misma instalación de drupal, cree un subdominio/dominio para el sitio que apunte a la IP del servidor, cree un dominio virtual en Apache, por ejemplo:
<pre>
<!VirtualHost 201.245.63.134>
    !ServerAdmin vtamara@pasosdeJesus.org
    !DocumentRoot /var/www/htdocs/drupal/
    !ServerName militarismo.cristianosporlapaz.info
    Alias /files/ /var/www/htdocs/drupal/sites/militarismo.cristianosporlapaz.info/files/
    Alias /htdocs/drupal/sites/militarismo.cristianosporlapaz.info/files/files/ /var/www/htdocs/drupal/sites/militarismo.cristianosporlapaz.info/files/
    !ErrorLog logs/militarismo-error_log
    !CustomLog logs/militarismo-access_log common
</!VirtualHost>
</pre>

Cree en ```/var/www/htdocs/drupal/sites``` un directorio por cada sitio (para que cada uno pueda tener espacio de archivos, temas y módulos personalizados), por ejemplo:
<pre>
mkdir /var/www/htdocs/drupal/sites/militarismo.cristianosporlapaz.info
cd /var/www/htdocs/drupal/sites/
cp defaults/settings.php militarismo.cristianosporlapaz.info/
cd militarismo.cristianosporlapaz.info/
mkdir files
mkdir themes
mkdir modules
</pre>


Edite el archivo ```settings.php``` para configurar una de las siguientes bases de datos:
* Una nueva base de datos (puede ser con nuevo usuario), los cuales también deberá crear con su motor de bases de datos
* Usar una base existente pero poniendo un prefijo a todas las tablas del nuevo sitio, por ejemplo: 
<pre>
$db_prefix='militarismo_';
</pre>
* Usar una base de datos existentes y compartir algunas de las tablas existentes.  Por ejemplo para compartir sólo información de autenticación con la instalación principal:
<pre>
$db_prefix = array(
 'default'   => 'militarismo_',
 'users'     => '',
 'sessions'  => '',
 'role'      => '',
 'authmap'   => '',
 'sequences' => '',
);
</pre>

Finalmente ingrese a install.php  (e.g militarismo.cristianosporlapaz.info/install.php) para crear y poblar tablas iniciales.


##Referencias

* Documentación de Drupal: http://drupal.org/handbooks
* Instalación de Gallery2: http://codex.gallery2.org/index.php/Gallery2:How_do_I_Install_Gallery2
* Inmersión de gallery2 en drupal: http://drupal.galleryembedded.com/index.php/Installation

----
Esta información se cede al dominio público y se dedica al padre que es VERDAD. 2006. vtamara@pasosdeJesus.org
