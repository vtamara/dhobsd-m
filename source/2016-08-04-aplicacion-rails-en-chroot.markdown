---
title: aplicacion-rails-en-chroot
date: 2016-08-04 12:00 UTC
tags:
---

# APLICACIÓN RAILS EN CHROOT

En adJ y OpenBSD es típico que las aplicaciones web corran en jaula chroot en ```/var/www``` con el usuario `www` y `www`. Por ejemplo esa es la configuración por omisión de PHP y todos los servidores web disponibles.

En el caso de Ruby on Rails no hay opciones para facilitar esto.  

## 1. Aplicación rails en producción en jaula en /var/www

En análogía a aplicaciones php, pueden correr con el usuario www, pero dado que se requieren límites más amplios para algunas aplicaciones, es mejor dejar este usuario en la clase de login servicio (o su versión obsoleta daemon) con vipw:
<pre>
www:*:67:67:servicio:0:0:HTTP Server:/var/www:/sbin/nologin 
</pre>
y que esa clase tenga límites suficientemente amplios como se explica en {3}

Se ubica la aplicación en /var/www y se instalan las gemas que requiere con:
<pre>
bundle install --deployment
</pre>


Se prepara un entorno ruby para correr chroot:
https://github.com/pasosdeJesus/adJ/blob/ADJ_5_9/arboldd/usr/local/adJ/ruby-chroot-www.sh

Los grandes pasos de este script son:

1. Tener actualizado ruby fuera de chroo
2. Instala paquetes ruby y node completos y entorno de ruby en jaula chroot $dchroot
3. Agrega librerías compartidas que pueda requerir la aplicación
4. Ejecuta ldconfig

Lo que sigue es ejecutar la aplicación en modo producción, hemos empleado unicorn con exito para esto, lo que requiere es:
* Ajusta permisos (por ejemplo tmp, log y public/assets deben ser escribibles po www).
* 

Ejecutar la aplicación, OJO no como root sino como www:ww pues está documentado como root puede salirse de la jaula.

##2. Aplicación rails en desarrollo en jaula /var/www

###2.1 Configuración general

<pre>
doas mkdir /var/www/bundler
doas chown $USER:www /var/www/bundler
</pre>

Para evitar conflictos con su archivo personal de configuración de bundle ejecute:
<pre>
mv ~/.bundle/config ~/.bundle/config-antes
</pre>

###2.2 Configuración de cada aplicación 

En la aplicación en el archivo .bundle/config asegurese de tener:
<pre>
---
BUNDLE_PATH: "/var/www/bundler"
BUNDLE_DISABLE_SHARED_GEMS: true
</pre>

Después ejecute 
<pre>
bundle install
</pre>
que intentará instalar las gemas, y aunque puede que no logre instalarlas todas, 
debería instalar algunas y dejar en ```/var/www/bundler``` una estructura como la siguiente:
<pre>
$ tree -L 3 -d /var/www/bundler/
/var/www/bundler/
`-- ruby
    `-- 2.3
        |-- bin
        |-- build_info
        |-- bundler
        |-- cache
        |-- doc
        |-- extensions
        |-- gems
        `-- specifications

10 directories
</pre>

Las gemas que ```bundle install``` no logre instalar por problemas de permisos instalelas de manera individual con ```gem install``` pero añadiendo después de install la opción ```--install-dir /var/www/bundler/ruby/2.3/``` por ejemplo:
<pre>
doas gem install --install-dir /var/www/bundler/ruby/2.3/ raindrops -v '0.17.0' 
</pre>

En el caso de ```nokogiri``` se requieren más opciones en adJ:
<pre>
doas gem install --install-dir /var/www/bundler/ruby/2.3/ nokogiri -v '1.6.8' -- --use-system-libraries
</pre>

Es recomendable hacer esto para aplicaciones en desarrollo y en producción del mismo servidor, así todas compartirán las gemas.

En cada aplicación cuando vaya a relizar alguna acción, asegurese de preceder todo ejecutable (e.g rails, unicorn) con ```bundle exec```, por ejemplo para lanzar servidor de desarrollo:
<pre>
bundle exec rails s
</pre>

###2.3 Actualización de gemas tras actualizar sistema operativo y/o ruby

Las librerías compartidas es posible que dejen de operar cuando actualice versión de librerías de las que dependen o del sistema operativo o de ruby.   Tipicamente se reconstruyen con ```gem pristine``` pero hemos notado que este opera sólo sobre las gemas instaladas en ```/usr/local/lib/ruby/gems``` así que recomendamos:

1. Actualizar gemas nativas instaladas junto con ruby y gemas instaladas en ```/usr/local/lib/ruby/gems```
<pre>
doas gem update --system
QMAKE=qmake-qt5 make=gmake MAKE=gmake doas gem pristine --all
</pre>

2. Actualizar gemas instaladas en ```/var/www/bundler```
<pre>
gem update --install-dir /var/www/bundler/ruby/2.3/
</pre>
Las que no puedan actualizarse por permisos, reinstalarlas con install, por ejemplo:
<pre>
doas gem install --install-dir /var/www/bundler/ruby/2.3/ nokogiri -- --use-system-libraries
</pre>

También podrá encontrar los que generar librerías y requieren reinstalación con:
<pre>
find /var/www/bundler -name "*.so"
</pre>
y algo como
<pre>
doas gem install --install-dir /var/www/bundler/ruby/2.3/  nio4r bcrypt debug_inspector kgio raindrops unicorn websocket-driver json byebug ffi io-console pg
</pre>




## 3. Referencias

* {3} http://pasosdejesus.github.io/usuario_adJ/configuracion-de-algunos-portes.html#ruby
