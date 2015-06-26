---
title: Instalaci_n_de_Horde_e_Imp
date: 2006-09-19
tags:
---
Se han probado con OpenBSD 3.8 con Apache chroot y Horde 3.0.4

!Prerequisitos
* Servidor Apache con php5 y los siguientes módulos: php5-imap, php5-ldap, php5-mbstring, php5-mcrypt, php5-pear
* Pear con los siguientes paquetes: Mail, !Mail_Mime, Log, DB,  !Net_Socket, Date, Auth_SASL, Net_URL, HTTP_request, File, Net_SMTP.
* Horde requiere guardar preferencias en una base de datos. En nuestro caso empleamos PostgreSQL que puede usarse con facilidad con Apache chroot.
* Además como con el protocolo HTTP la transmisión de información de formularios se hace de forma plana, es recomendable que emplee Apache con SSL y permita el uso de Horde sólo con HTTPS, esto lo puede lograr agregando un VirtualHost en la sección de SSL del archivo de configuración de Apache o un Alias en tal sección, por ejemplo note la linea con el alias en:
<pre>
<!VirtualHost _default_:443>
#  General setup for the virtual host
!DocumentRoot /var/www/htdocs/
!ServerName www.pasosdeJesus.org
!ServerAlias pasosdeJesus.org
!ServerAdmin info@pasosdeJesus.org
!ErrorLog logs/error_log_ssl
!TransferLog logs/access_log_ssl

Alias /horde/ /var/www/horde/
</!VirtualHost>
</pre>

* Verifique que esté corriendo el servidor IMAP (mejor IMAP-SSL).

!Instalación de Horde

Seguir pasos del paquete y de horde/doc/INSTALL en particular:
* Instalar el paquete horde
* Ejecutar:
<pre>
cd /var/www/horde/
sudo ln -s ../horde/ /var/www/htdocs/horde

# su _postgresql
$ psql -h /var/www/tmp -d template1 -f scripts/sql/create.pgsql.sql
$ psql -h /var/www/tmp -qc "ALTER USER horde WITH PASSWORD 'Clave';" template1 
$ psql -h /var/www/tmp -d horde -f scripts/sql/horde_users.sql


$ psql -h /var/www/tmp -d horde -f scripts/sql/horde_prefs.sql 
$ psql -h /var/www/tmp -d horde -f scripts/sql/horde_datatree.sql

# cd config/
# for f in *.dist; do cp $f `basename $f .dist`; done
# rm hooks.php
</pre>

Después desde un navegador abra el URL ```https://localhost/horde/test.php```
y verifique que estén los modulos requeridos.  Algunos módulos ya están disponibles como paquetes, por ejemplo php5-imap.

Si faltan modulos de PECL, primero asegurese de tener versiones recientes de autoconf (digamos 2.59) y automake (digamos 1.4), después:
<pre>
# export AUTOMAKE_VERSION=1.4
# export AUTOCONF_VERSION=2.59
# pear install fileinfo*/
</pre>

Cada vez que agregue módulos reinicie el servidor web y vuelva a cargar la página de prueba:
<pre>
# apachectl stop
# . /etc/rc.conf
# httpd $httpd_flags
</pre>

Los paquetes de PEAR que hagan falta puede instalarlos con algo como:
<pre>
# pear install Date
</pre>

!Instalación de IMP
Una vez instalado Horde debe instalar el paquete ```imp```  y seguir las instrucciones disponibles  en ```/var/www/horde/imp/docs/INSTALL```, las
cuales pueden resumirse en:
<pre>
# cd imp/config/
# for foo in *.dist; do cp $foo `basename $foo .dist`; done
# httpd $httpd_flags
</pre>

!Caso chroot

Si Apache corre chroot deberá seguir los pasos descritos en http://bugs.horde.org/ticket/?id=711 que corresponden a: 

<pre>
# mkdir /var/www/etc
# grep "^www" /etc/master.passwd > /var/www/etc/master.passwd
# pwd_mkdb -d /var/www/etc/ /var/www/etc/master.passwd
# chmod a-w+r /var/www/etc/master.passwd
</pre>

!Configuración de Horde y de IMP

* Con un navegador ingrese a https://localhost/horde donde podrá configurar horde, en particular:
** Poner clave a usuario administrador antes de cambiar método de autenticación.
** Configurar como base de datos PostgreSQL con usuario horde y base horde
** Configurar la autenticación para que la haga la aplicación imp
** Agregar un usuario del sistema como administrador (separar uno de otro con coma).
** Configurar el envío de correo para que sea por SMTP usando como servidor localhost puerto 25.
** Configure un correo para reportar problemas.
* Configure también ```imp``` en particular:
** Si cuenta con ```ispell``` en chroot configurelo (para instalarlo puede usar el script ispell.sh disponible con fuentes de SIVeL http://sivel.sf.net).
** Emplee nombres de carpetas acordes con otros clientes de correo que use, por ejemplo Thunderbird por defecto deja enviados en "Sent", este nombre lo puede modificar en ```$_prefs['sent_mail_folder']``` del archivo ```/var/www/horde/imp/config/prefs.php```
** Edite el archivo ```/var/www/horde/imp/config/servers.php``` para que incluya:<pre>
$servers!['imap'] = array(
    'name' => 'Pasos de Jesús IMAP-SSL',
    'server' => 'correo.pasosdeJesus.org',
    'hordeauth' => false,
    'protocol' => 'imap/ssl/notls/novalidate-cert',
    'port' => 993,
    'folders' => 'maildir/',
    'namespace' => '',
    'maildomain' => 'pasosdeJesus.org',
    'smtphost' => 'localhost',
    'smtpport' => 25,
    'realm' => '',
    'preferred' => '',
    'dotfiles' => false,
    'hierarchies' => array()
);
</pre>

* Para que la aplicación principal sea IMP, modifique ```/var/www/horde/config/prefs.php``` y cambie ```value``` de ```horde``` a
```imp``` en la preferencia ```initial_application``` de forma que quede:

<pre>
$_prefs['initial_application'] = array(
    'value' => 'imp',
    'locked' => false,
    'shared' => true,
    'type' => 'select',
    'desc' => sprintf(_("What application should %s display after login?"),
                  $GLOBALS['registry']->get('name'))
);
</pre>

!Pruebas

Ingrese a ```https://localhost/horde/``` con un usuario del sistema y
verifique que pueda recibir y enviar correo.

Desde el navegador cliente es recomendable configurar idioma para que sea español.


!Otras aplicaciones Horde

Puede emplear también passwd para que los usuarios cambien su clave.
# Instale el paquete poppassd
# Instale el módulo passwd de Horde (quedará en ```/var/www/horde/passwd```)
# Configure el módulo para que use el controlador apropiado, edite ```/var/www/horde/passwd/config/backend.php``` para que sólo contenga:
<pre>
$backends['poppassd'] = array(
    'name' => 'Correo en Pasos de Jesus',
    'preferred' => '',
    'password policy' => array(
        'minLength' => 3,
        'maxSpace' => 0,
        'minUpper' => 1,
        'minLower' => 1,
        'minNumeric' => 1,
        'minSymbols' => 1
    ),
    'driver' => 'poppassd',
    'params' => array(
        'host' => '127.0.0.1',
        'port' => 106
    )
);
</pre>

También funcionan sin problema:
* turba: para manejar contactos

##Referencias:
* ```/var/www/horde/docs/INSTALL```
* ```/var/www/horde/imp/docs/INSTALL```

-----
Esta información se cede al dominio público y se dedica al padre que es Amor. 2006. vtamara@pasosdeJesus.org

