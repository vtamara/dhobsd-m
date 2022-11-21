---
title: Desarrollar_Lernanta_en_OpenBSD_adJ
date: 2011-04-29 12:00 UTC
tags:
---
''Lernanta'' es una plataforma para educación virtual, con triple licencia MPL/GPL/LGPL, pero desarrollada en un ambiente de alta solidaridad y fraternidad. Está escrita en ```python``` con el ambiente de desarrollo ```django```, migraciones con ```south``` y otras facilidades de ese lenguaje.   La labor de localización y en particular a español, así como la posibilidad de migraciones fue especialmente contribuida por el autor de este escrito.

La descripción para configurar un ambiente de desarrolo en Ubuntu con MySQL y bash se encuentra en inglés en: https://github.com/p2pu/lernanta/ 

Este escrito se inspira en ese, pero enfocandose en otro ambiente: OpenBSD adJ 4.8 con PostgreSQL y ```ksh``` y detallando el proceso de actualizaciones con migraciones (que por el momento está en el repositorio de vtamara y en espera de ser aceptado en el oficial de p2pu).


##1. Instalar ambiente de desarrollo

!1.1 Ambiente con interprete de comandos ```ksh```

adJ 4.8 incluye python-2.5.4 y py-setuptools-0.6.11, pero Lernanta requiere la versión 2.6, por lo que debe instalar pyhton-2.6, como se explica a continuación.

Configure un espejo de los paquetes de OpenBSD en PKG_PATH, por ejemplo agregue en ```~/.profile```:
<pre>
export PKG_PATH=ftp://ftp3.usa.openbsd.org/pub/OpenBSD/4.8/packages/i386/
</pre>
Y ejecutelo con ```. ~/.profile``` a continuación descargue e instale python-2.6.5:

<pre>
sudo pkg_add python-2.6.5
</pre>

Instale py-setuptools, que debe desacargar (versión 2.6) de http://pypi.python.org/pypi/setuptools
e instalar con:
<pre>
chmod +x ./setuptools-0.6c11-py2.6.egg
sudo ./setuptools-0.6c11-py2.6.egg 
</pre>

Instale PIL que debe descargar de http://www.pythonware.com/products/pil/ e instalar con:
<pre>
tar xvfz Imaging-1.1.7.tar.gz
cd Imaging-1.1.7
sudo python2.6 setup.py install
</pre>

A continuación instale paquetes de python con las herramientas de ese lenguaje:

<pre>
sudo easy_install-2.6 virtualenv
sudo easy_install-2.6 pip
sudo pip install virtualenvwrapper
</pre>

El último comando instalará el archivo de comandos para bash ```/usr/local/bin/virtualenvwrapper.sh``` que al ejecutarse con ```ksh``` e.g ```. /usr/local/bin/virtualenvwrapper.sh``` presentará errores como:
<pre>
ksh: /usr/local/bin/virtualenvwrapper.sh![297]: syntax error: `(' unexpected
</pre>

Como [se reportó | https://bitbucket.org/dhellmann/virtualenvwrapper/issue/96/to-work-with-ksh] al desarrollador de virtualenvwrapper.sh, en este caso puede convertirse con facilidad a ```ksh``` tras ejecutar ```alias source=.```, pues ese archivo de comandos lo único que utiliza particular de ```bash``` es inicialización de arreglos con paréntesis (ver http://tldp.org/LDP/abs/html/special-chars.html) y esto en ```ksh``` puede hacerse con ```set -A``` (ver ```man ksh``` o http://docstore.mik.ua/orelly/unix/ksh/ch06_03.htm).  Entonces edite ese archivo (e.g ```sudo vim  /usr/local/bin/virtualenvwrapper.sh```) y remplace
<pre>
args=( $(getopt blh "$@") )
</pre>
 por
<pre>
set -A args  $(getopt blh "$@")
</pre>
y de forma análoga las otras inicializaciones de arreglos para que queden:
<pre>
set -A COMPREPLY $(compgen -W "`virtualenvwrapper_show_workon_options`" -- ${cur})

set -A COMPREPLY $(cdvirtualenv && compgen -d -- "${cur}" )

set -A COMPREPLY $(cdsitepackages && compgen -d -- "${cur}" )
</pre>

Tras hacer estos cambios agregue a ```~/.profile```:
<pre>
# in .profile
alias source=.
export VIRTUALENVWRAPPER_PYTHON=/usr/local/bin/python2.6
export WORKON_HOME=$HOME/.virtualenvs
export !PIP_VIRTUALENV_BASE=$WORKON_HOME
export !PIP_RESPECT_VIRTUALENV=true
source /usr/local/bin/virtualenvwrapper.sh
</pre>

Y pruebe la ejecución (que la primera vez creara directorios en ```~/.virtualenvs```) con:
<pre>
. ~/.profile
</pre>

Después ejecute
<pre>
mkvirtualenv --python=/usr/local/bin/python2.6 --no-site-packages lernanta
</pre>

Que prepara el ambiente para lernanta. Tras esto debe tener disponible un nuevo comando ```workon lernanta``` que ayudará a establecer el ambiente python de lernanta y cambiara su prompt ```shalom$``` para convertirlo en ```(lernanta)shalom$``` y desde el cual debe realizar toda operación de desarrollo.

!1.2. Fuentes clonadas de github.com

El repositorio de fuentes de lernanta está en [github.com | http://github.com/p2pu/lernanta], por lo que requiere instalar git:
<pre>
sudo pkg_add git 
</pre>

y clonar un repositorio en algún directorio de su sistema (digamos en ```~/des```).

Si no desea crear una cuenta en github basta que clone uno de los repositorios y que comparta sus mejoras enviando parches a los desarrolladores.

Para clonar el repositorio de p2pu y comunicarse con desarrolladores en inglés (por ejemplo en la lista p2pu-dev a la que puede suscribirse desde http://lists.p2pu.org/mailman/listinfo/p2pu-dev ):
<pre>
mkdir ~/des
cd ~/des
git clone https://github.com/p2pu/lernanta.git
</pre>

O para clonar el repositorio de vtamara y comunicarse con él en español al correo vtamara@pasosdeJesus.org :
<pre>
mkdir ~/des
cd ~/des
git clone https://github.com/vtamara/lernanta.git
</pre>

Otra opción que tiene es crear una cuenta en github.com, establecer su llave RSA y hacer fork (copia) de uno de estos proyectos.  En ese caso no clonaría de esos URLs sino del suyo y después podría devolver contribuciones al proyecto que clonó haciendo solicitudes de pull.  http://help.github.com/linux-set-up-git/  .  Si elije esta vía y su nombre tiene acentos propios del español, sugerimos que una vez configure su nombre y correo, no emplee las variables de entorno !GIT_COMMITTER_NAME ni !GIT_AUTHOR_NAME y que codifique sus archivos ~/.gitconfig y .git/config de las fuentes de Lernanta en UTF-8 para evitar problemas en sus aportes, i.e ''commits'' (ver http://support.github.com/discussions/site/3560-author-not-showing-in-commits).


!1.3 Ambiente para python con PostgreSQL

Prepare le ambiente para python con:

<pre>
cd ~/des/lernanta
workon lernanta
pip install -r requirements/compiled.txt
pip install -r requirements/prod.txt
pip install -r requirements/dev.txt
</pre>

El entorno por defecto de lernanta supone que se usará ```mysql```. Lo necesario para usar PostgreSQL se instala con:
<pre>
pip install psycopg2
</pre>

!1.4 Configurar base de datos

Cree un usuario para PostgreSQL (digamos ```lernanta```) y una base de datos (digamos ```lernanta```) --cambiando ```miclave``` por una buena clave como las generadas por ```apg```:
<pre>
$ sudo su - _postgresql
$ createuser -h/var/www/tmp -Upostgres lernanta 
Shall the new role be a superuser? (y/n) n
Shall the new role be allowed to create databases? (y/n) n
Shall the new role be allowed to create more new roles? (y/n) n
$ createdb -h/var/www/tmp -Upostgres lernanta -Olernanta
$ psql -h/var/www/tmp -Upostgres template1                   
psql (8.4.7)
Type "help" for help.

template1=# alter user lernanta with password 'miclave';
ALTER ROLE
template1=# \q
exit
</pre>

A continuación desde el directorio ```~/des/lernanta``` trabajando en el ambiente lernanta, cree un archivo de configuración:
<pre>
cp settings_local.dist.py settings_local.py
</pre>
 y edite la sección DATABASES de ```settings_local.py```, cambiando la base ```default``` (preconfigurada para MySQL) de forma que quede:
<pre>
DATABASES = {

    'default': {
        'NAME': 'lernanta',
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'USER': 'lernanta',
        'PASSWORD': 'miclave',
        'HOST': '/var/www/tmp', # Socket en /var/www/tmp
        'OPTIONS': { },
    },
}
</pre>
y ponga en comentarios la sección ```drupal_users```.

Si clonó el repositorio de vtamara cree las tablas a partir del modelo en el código fuente con:

<pre>
./manage.py syncdb --noinput --migrate
</pre>

Si nota fallas con el comando anterior (mientras se soluciona un problema con migraciones en las fuentes) o si clonó el repositorio de p2pu, emplee la siguiente forma:

<pre>
./manage.py syncdb --noinput --all
./manage.py migrate --all --fake
</pre>


!1.5 Opcional: Configuración de vim como IDE 

Para programar python con vim con resaltado de sintaxis e indentación automática, después de instalar el paquete vim, configurelo para su usuario con:
<pre>
cp -rf /usr/local/share/vim/vim72/ ~/.vim
</pre>

El espaciado por defecto de lernanta lo puede configurar entre otras, agregando a su archivo ~/.vimrc:

<pre>
set expandtab
set tabstop=4
set shiftwidth=4
set foldmethod=marker
</pre>

##2. PRUEBAS

!2.1 Pruebas de regresión

<pre>
./manage.py test
</pre>


!2.2 Uso con un servidor web propio de django:

Desde la terminal ejecute:
<pre>
./manage.py runserver
</pre>

Después de esto abra un navegador en el mismo computador y como dirección ingrese ```http://127.0.0.1:8000```

##3. ACTUALIZACIONES

Las fuentes de lernanta se desarrollan rapidamente por lo que es importante que las actualice con frecuencia con:

<pre>
cd ~/des/lernanta
git fetch
git merge master
</pre>

Actualice dependencias con:
<pre>
pip install -r requirements/compiled.txt
pip install -r requirements/prod.txt
pip install -r requirements/dev.txt
</pre>

En ocasiones los cambios a las fuentes requieren cambios al modelo de base de datos, puede aplicarlos con:

<pre>
./manage.py syncdb --noinput --migrate
</pre>

Verifique si hay cambios importantes en ```settings_local.dist.py``` comparando con su archivo:
<pre>
diff settings_local.dist.py settings_local.py
</pre>
Y agregue lo necesario a su archivo ```settings_local.py```

##4. AYUDAS PARA EL DESARROLLO

* Una vez esté ejecutado ```./manage.py runserver``` puede ingresar a administrar la base de datos desde: http://127.0.0.1:8000/admin/
* Además de los mensajes en consola (en particular los correos), quedan otros rastros de la ejecución en ```lernanta.log```
* Si en medio del código requiere examinar variables, asegurese de tener al comienzo de la fuente
<pre>
import logging
log = logging.getLogger(__name__)
</pre>
y emplear emplear:
<pre>
log.debug("v=" + v)
</pre>
Esto agregará un mensjae en la bitácora ```lernanta.log``` que queda en el directorio de fuentes.


##5. USO CON APACHE Y FASTCGI

Es posible emplear lernanta con el módulo FastCGI de Apache, para esto:

!5.1 Instale el paquete mod_fastcgi y flup:
<pre>
sudo pkg_add mod_fastcgi
sudo  /usr/local/sbin/mod_fastcgi-enable
sudo apachectl stop
sudo sh /etc/rc.local
</pre>
posiblemente para que el servidor web inicie tendrá que reubicar la línea
<pre>!AddModule mod_fastcgi.c</pre>
para que quede después de
<pre>!LoadModule fastcgi_module     /usr/lib/apache/modules/mod_fastcgi.so</pre>

Instale flup desde el ambiente de lernanta con:
<pre>
workon lernanta
pip install flup
</pre>

!5.2 Configure el virtualhost
En el archivo de configuración de Apache ```/var/www/conf/httpd.conf``` active el módulo mod_rewrite y  agregue una sección como:
<pre>
<!VirtualHost 127.0.0.1:443 192.168.111.1:443>
    !ServerAdmin vtamara@pasosdeJesus.org
    !DocumentRoot /var/www/lernanta/
    !ServerName lernanta.pasosdeJesus.org

    !SSLEngine on
    !SSLCertificateFile  /etc/ssl/server.crt
    !SSLCertificateKeyFile /etc/ssl/private/server.key

    !ErrorLog logs/lernanta-error_log
    !CustomLog logs/lernanta-access_log common

    !FastCGIExternalServer /lernanta/lernanta.fcgi -socket /tmp/lernanta.sock

    !RewriteEngine ON
    !RewriteRule ^/(media.*)$ /$1 ![QSA,L,PT]
    !RewriteCond %{REQUEST_FILENAME} !-f
    !RewriteRule ^/(.*)$ /lernanta.fcgi/$1 ![QSA,L]
</!VirtualHost>
</pre>

Tras hacer el cambio reinicie Apache.

!5.3 Ubique lernanta dentro de la jaula chroot de Apache

Supogamos que la ubica en ```/var/www/lernanta``` asegure que puede operarla e inicie el servidor con:
<pre>
workon lernanta
sudo rm -f /var/www/tmp/lernanta.sock
sudo ./manage.py runfcgi debug=true workdir=/var/www/lernanta  socket=/var/www/tmp/lernanta.sock
</pre>
Cuando haga esto se iniciaran varios procesos python. A continuación permita a Apache emplear el socket con:
<pre>
sudo chown www:www /var/www/tmp/lernanta.sock
</pre>

Finalmente pruebe intentando ingresar a su sitio.

Para hacer más durable el servidor puede agregar a ```/etc/rc.local```:
<pre>
ps ax | grep "![p]ython ./manage" > /dev/null 2>&1
if ![ "$?" != "0" ]; then
        echo -n " lernanta";
        sudo su - miusuario -c "cd /var/www/lernanta; workon lernanta; \
                      sudo rm -f /var/www/tmp/lernanta.sock; \
                      sudo ./manage.py runfcgi debug=true workdir=/var/www/lernanta socket=/var/www/tmp/lernanta.sock ;\
                      sudo chown www:www /var/www/tmp/lernanta.sock"
fi
</pre>

!5.4 Modificaciones a fuentes

Si modifica o actualiza fuentes de P2PU es necesario borrar caches, terminando los procesos de python que inició y volviendo a cargarlos.


##6. AGRADECIMIENTOS

A Dios por la solidaridad.  A Jessica Ledbetter y Zuzel Vera, desarrolladoras de Lernanta, por su amabilidad y calidad.
