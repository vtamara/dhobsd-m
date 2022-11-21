---
title: PHP_y_DB_en_OpenBSD_3.6
date: 2005-12-20 12:00 UTC
tags:
---
May.18.2005 vtamara@pasosdeJesus.org

Actualizar a PHP4.3.11:
<pre>
$ sudo pkg_delete -f dependencies php4-core
$ export PKG_PATH=ftp://pasosdeJesus.org/pub/AprendiendoDeJesus/progreso/paquetes/
$ sudo pkg_add $PKG_PATH/php4-extensions-4.3.11.tgz
$ sudo phpxs -s
$ sudo pkg_add $PKG_PATH/php4-pear-4.3.11.tgz
</pre>
Instalar y activar otras extensiones que requiere por ejemplo:
<pre>
$ sudo pkg_add $PKG_PATH/php4-pgsql-4.3.11.tgz
$ sudo phpxs -a pgsql
</pre>

Descargar ```sqlite.so``` que ya compilamos:
<pre>
$ ftp  ftp://pasosdeJesus.org/pub/AprendiendoDeJesus/progreso/pear-php4.3.11/sqlite.so
$ sudo mv sqlite.so /var/www/lib/php/modules/
</pre>
Parece que ```pearcmd.php``` del paquete PEAR tiene una falla, compilamos modificandolo para que sacara una copia del directorio donde quedaba la librería antes de salir (cuando sale la borra).

Agregar a ```/var/www/conf/php.ini```:
<pre>
extension="sqlite.so"
</pre>
y reiniciar Apache con las mismas opciones con las que arranca:

<pre>
$ sudo apachectl stop
$ . /etc/rc.conf.local
$ sudo httpd $httpd_flags
</pre>

Puede probarse con el siguiente script que crea ejemplo.db:

<pre>
<?php

        # Tomada de http://www.php-mag.net/itr/online_artikel/psecom,id,447,nodeid,114

        // open the database - will create it if it doesn't
        // exist.
        $db = sqlite_open("ejemplo.db") or
        die("failed to open/create the database");
        // now create the sample table
        sqlite_query($db, "CREATE TABLE sample(email, name)");
        // fill it with some data
        $data = array(
                'wez?php' => 'Wez Furlong',
                'helly?php' => 'Marcus Boerger',
                'derick?php' => 'Derick Rethans',
                // sorry to all the other PHP developers
                // for not listing them here too...
        );
        foreach ($data as $email => $name) {
                $email = sqlite_escape_string($email);
        foreach ($data as $email => $name) {
                $email = sqlite_escape_string($email);
                $name = sqlite_escape_string($name);
                sqlite_query($db,
                "INSERT INTO sample(email, name) "
                ."VALUES ('$email', '$name')");
        }
        // Now pull it out again
        $res = sqlite_query($db, "SELECT name, email from sample");
        if (!$res) {
                // This shouldn't happen :)
                echo "No data";
        } else {
                while ($row = sqlite_fetch_array($res)) {
                        echo "row: $rowname? -> $rowemail?\n";
                }
        }
?>
</pre>

Y desde la línea de comandos:

<pre>
$ php prueba-sqlite.php
row: Wez Furlong -> wez?php
row: Marcus Boerger -> helly?php
row: Derick Rethans -> derick?php
</pre>

Finalmente pude instalar DB de Pear:

<pre>
$ sudo pear install DB
</pre>

y si ya hizo la prueba anterior puede complementarla con:

<pre>
<?php

# http://pear.php.net/manual/en/package.database.db.intro-dsn.php
# En juego con prueba-sqlite.php --tras mv example.db db/ejemplo.db


require_once 'DB.php';

$dsn = 'sqlite:////home/yo/public_html/ejemplo.db?mode=0666?';
$options = array(
        'debug'       => 2, 
        'portability' => DB_PORTABILITY_ALL?,
);

$db =& DB::connect($dsn, $options);
if (PEAR::isError($db)) { 
        die($db->getMessage());
}

$res = $db->query("SELECT name, email from sample");
$row=array();
while ($res->fetchInto($row)) {
        echo "row: $row0? -> $row1?\n";
}

$db->disconnect();

?>
</pre>
ejecutando desde la línea de comandos:
<pre>
$ php prueba-sqlite-db.php
row: Wez Furlong -> wez?php 
row: Marcus Boerger -> helly?php
row: Derick Rethans -> derick?php
</pre>
