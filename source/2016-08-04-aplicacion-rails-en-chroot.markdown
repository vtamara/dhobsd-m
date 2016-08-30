---
title: aplicacion-rails-en-chroot
date: 2016-08-04
tags:
---

# APLICACIÓN RAILS EN CHROOT

En adJ y OpenBSD es típico que las aplicaciones web corran en jaula chroot en ```/var/www``` con el usuario `www` y `www`. Por ejemplo esa es la configuración por omisión de PHP y todos los servidores web disponibles.

En el caso de Ruby on Rails no hay opciones para facilitar esto.  

Se puede hacer gradual.

##1. Gemas compartidas en /var/www/bundler

<pre>
doas mkdir /var/www/bundler
doas chown $USER:www /var/www/bundler
</pre>

En la aplicación en el archivo .bundle/config asegurese de tener:
<pre>
---
BUNDLE_PATH: "/var/www/bundler"
BUNDLE_DISABLE_SHARED_GEMS: true
</pre>

Para evitar conflictos con su archivo personal de configuración de bundle puede ejecutar:
<pre>
mv ~/.bundle/config ~/.bundle/config-antes
</pre>

Después ejecute 
<pre>
bundle install
</pre>
que intenta instalar las gemas, y aunque puede que no logre instalarlas todas, 
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


Las gemas que bundle install no logre instalar por problemas de permisos instalelas de manera individual con gem install pero añadiendo después de install la opción ```--install-dir /var/www/bundler/ruby/2.3/``` por ejemplo:
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
