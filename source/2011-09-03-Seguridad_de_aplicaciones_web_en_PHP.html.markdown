---
title: Seguridad_de_aplicaciones_web_en_PHP
date: 2011-09-03
tags:
---
##Inyección SQL

!Definición
https://www.owasp.org/index.php/SQL_Injection

!Como prevenirlo
Para prevenirlo es necesario escapar las cadenas antes de insertarlas en la base de datos.   Normalmente esto lo hacen automáticamente las funciones de preparar y ejecutar consultas (como pg_prepare y pg_exec).

Si prefiere construir por si mismo consultas tenga en cuenta que la forma de escapar una cadena depende del motor de bases de datos que use.  Con PostgreSQL use la función pg_escape_string, o la siguiente porción para escapar la cadena $entrada y dejarla en $salida:
<pre>
 $salida = (!get_magic_quotes_gpc()) ? str_replace("'", "''", str_replace('\\', '\\\\', $entrada)) : $entrada;   
</pre>

Insistimos que la forma de escapar una cadena depende del motor de base de datos, el ejemplo anterior introduciría caracteres incorrectos en bases de datos con SQLite, pues en ese motor \\ no es un secuencia de escape para \.

Si prefiere un método más general, el paquete MDB2 de PEAR ofrece la función ```escape``` implementada para cada motor soportado (MySQL, !MySQLi, PostgreSQL, Oracle, Frontbase, Querysim, Interbase/Firebird, MSSQL, SQLite), vea                                        
http://pear.php.net/package/MDB2/docs/2.5.0b3/MDB2/MDB2_Driver_Common.html#methodescape  
Por ejemplo si la manija de la base de datos es  $bd use:
<pre>
$salida = $bd->escape($entrada);
</pre>

!Buscarlo en código fuente

En las fuentes de una aplicación PHP debe revisar inserciones, actualizaciones y consultas a la base de datos.

Si inserta datos que vienen de un formulario debe verificar que escapa los valores de los arreglos  _GET, _REQUEST y _POST, por ejemplo busque con:

<pre>
find . -name "*php" -exec grep  "\$_REQUEST\[" {} ';'
find . -name "*php" -exec grep  "\$_GET\[" {} ';'
find . -name "*php" -exec grep  "\$_POST\[" {} ';'
</pre>

----

## XSS - Cross Site Scripting

!Definición
https://www.owasp.org/index.php/Cross-site_Scripting_(XSS)

!Como prevenirlo

* Los textos que deben mostrarse en elementos HTML y que usan variables con entrada del usuario deben emplear la función ```htmlentities``` o la función ```htmlspecialchars```.
* Los textos por incluir en URLs con variables con entrada del usuario deben emplear la función urlencodee
*  WebGoat también recomienda escapar con estas funciones los valores con entrada del usuario antes de guardarlos en una base de datos, sin embargo de hacerse debe buscarse evitar conflicto con las otras dos medias recién presentadas para no escapar dos veces las mismas cadenas.

!Buscarlo en código fuente

Revise sitios donde se emita información en HTML, especialmente datos que provengan del usuario, no sólo en las variables  _POST, _GET, _REQUEST sin también en variables que incluyan el URL por ejemplo $_SERVER!['PHP_SELF'].

----

##CRSF

!Definición
https://www.owasp.org/index.php/CSRF

!Como prevenirlo
Cuando esté emitiendo un formulario incluya un campo escondido con un número aleatorio con un número aleatorio suficientemente grande y aleatorio, almacen el número también en la sesión:
<pre>
        if (!isset($_POST!['evita_csrf'])) {                                     
            $_SESSION!['sin_csrf'] = mt_rand(0, 100000000000);                           
        }                                                                       
          $this->addElement('hidden', 'evita_csrf', $_SESSION!['sin_csrf']);     
</pre>
Cuando usted reciba datos de un formulario compare el valor de la sesión con el valor del campo escondido del formulario:
<pre>
if (!isset($_POST!['evita_csrf']) || $_POST!['evita_csrf'] != $_SESSION!['sin_csrf']) {                                                                     
   die("Datos enviados no pasaron verificación CSRF ("  . 
       $_SESSION!['sin_csrf'] . ", " . $_POST!['evita_csrf'] . ")"  );                                                                  
}
</pre>

!Buscarlo en código fuente

Búsque en formularios generados por la aplicación si se insertó un valor aleatorio.

----

##Buffer overflow

!Definición
https://www.owasp.org/index.php/Buffer_overflow_attack

!Como prevenirlo

Si la longitud máxima de $out es $maxlong:   
<pre>                                                                           
$out = substr($in, 0, $maxlong);
</pre>

!Buscarlo en código fuente

----
Dominio Público. 2011. https://www.pasosdejesus.org/dominio_publico_colombia.html
