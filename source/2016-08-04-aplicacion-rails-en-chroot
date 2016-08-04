---
title: aplicacion-rails-en-chroot
date: 2016-08-04
tags:
---

Se puede hacer gradual.

1. Gemas compartidas en /var/www/bundler

<pre>
mkdir /var/www/bundler
chown $USER:www /var/www/bundler
</pre>

En la aplicación en el archivo .bundler/config asegurese de tener:
<pre>
---
BUNDLE_PATH: "/var/www/bundler"
BUNDLE_DISABLE_SHARED_GEMS: true  
</pre>

Una vez con esto, ejecute 
<pre>
bundle install
</pre>
que intenta instalar las gemas, y aunque puede que no logre instalarlas todas, 
pero debería dejar en ```/var/www/bundler``` una estructura como la siguiente:
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


