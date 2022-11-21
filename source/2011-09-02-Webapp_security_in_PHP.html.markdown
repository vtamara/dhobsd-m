---
title: Webapp_security_in_PHP
date: 2011-09-02 12:00 UTC
tags:
---
##SQL Injection

!Definition
https://www.owasp.org/index.php/SQL_Injection

!Prevent it
To prevent it, it is necessary to escape the strings before inserting them in the database.
The way to escape depends on the database that you are using.  With PostgreSQL the following will escape the string $in:
<pre>
 $out = (!get_magic_quotes_gpc()) ? str_replace("'", "''", str_replace('\\', '\\\\', $in)) : $in;   
</pre>

We insist, the way to escape a string depends on the database, the previous example would introduce wrong characters in SQLite, since with that database \\ is not a escape sequence for \.

If you prefer a more general method, the Pear Package MDB2 offers the function ```escape``` implemented  for each supported database (MySQL, !MySQLi, PostgreSQL, Oracle, Frontbase, Querysim, Interbase/Firebird, MSSQL, SQLite), see                                        
http://pear.php.net/package/MDB2/docs/2.5.0b3/MDB2/MDB2_Driver_Common.html#methodescape  
For example if the handle of your database is $db use:
<pre>
$out = $db->escape($in);
</pre>

!Look for it in source code

In the sources of a PHP application you should check insertions or updates in the Database. 

If you insert data coming from a form you can check if you are escaping the values of the arrays _GET, _REQUEST and _POST, for example with:

<pre>
find . -name "*php" -exec grep  "\$_REQUEST\[" {} ';'
find . -name "*php" -exec grep  "\$_GET\[" {} ';'
find . -name "*php" -exec grep  "\$_POST\[" {} ';'
</pre>

----

## XSS - Cross Site Scripting

!Definition
https://www.owasp.org/index.php/Cross-site_Scripting_(XSS)

!Prevent it
For text that should be shown inside HTML elements use the function  htmlentities or htmlspecialchars.                                                                   

For text that should be included in an URL use  the function urlencode

!Look for it in source code
Check where the code outputs information for the HTML page, specially data coming from the user, not only in the variables _POST, _GET, REQUEST but also in variables that include the URL for example $_SERVER!['PHP_SELF'].

----

##CRSF

!Definition
https://www.owasp.org/index.php/CSRF

!Prevent it
When you are generating a form include a hidden field with a random number, store the number also in the session:
<pre>
        if (!isset($_POST!['evita_csrf'])) {                                     
            $_SESSION!['sin_csrf'] = mt_rand(0, 1000);                           
        }                                                                       
          $this->addElement('hidden', 'evita_csrf', $_SESSION!['sin_csrf']);     
</pre>
When you receive data from the form compare the session value with the hidden field of the form:
<pre>
if (!isset($_POST!['evita_csrf']) || $_POST!['evita_csrf'] != $_SESSION!['sin_csrf']) {                                                                     
   die("Datos enviados no pasaron verificación CSRF ("  . 
       $_SESSION!['sin_csrf'] . ", " . $_POST!['evita_csrf'] . ")"  );                                                                  
}
</pre>

!Look for it in source code

Look in the generated forms if a random value was inserted.

----

##Buffer overflow

!Definition
https://www.owasp.org/index.php/Buffer_overflow_attack

!Prevention
If the maximal length of $out is $maxlong:   
<pre>                                                                           
$out = substr($in, 0, $maxlong);
</pre>

!Look for it in source code

----
Public domain. 2011. 
