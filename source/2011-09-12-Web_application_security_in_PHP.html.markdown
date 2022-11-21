---
title: Web_application_security_in_PHP
date: 2011-09-12 12:00 UTC
tags:
---
##HTTP Response Header Splitting

When the headers of an HTTP Response can be changed by user input (for example redirecting to a URL that can be controlled by user), he/she could insert new lines and more headers. 

See also https://www.owasp.org/index.php/HTTP_Response_Splitting

!Example
Taken from http://www.securiteam.com/securityreviews/5WP0E2KFGK.html
<pre>
<?php
header ("Location: " . $_GET!['page']);
?>
</pre>
If this script is hrhs.php call it as: 
<pre>
hrhs.php?page=%!0aContent-Type: text/html%0aHTTP/1.1 200 OK%!0aContent-Type: text/html%0a%0a%3Chtml%3E%3Cfont color=red%3Ehey%3C/font%3E%3C/html%3E
</pre>

!Prevent it

PHP haredened with directive ```suhosin.multiheader = On``` in ```php.ini``` is not vulnerable to this atack.  With it trying to inject a new line in a header produces:
<pre>
![Mon Sep 12 11:42:46 2011] ![error] PHP Warning:  Header may not contain more 
than a single header, new line detected. in /path/hrs.php on line 2
</pre>

Otherwise you can remove "\n" and "\r" from the URL:
<pre>
$p = str_replace(array("\n", "\r"), array("", ""), $_GET!['page']);
header ("Location: $p");
</pre>

!Look for it in sources

Look for header() and check it doesn't have variables controlled by user that could include "\n", "\r".


----
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
* If you need to emit text in HTML that includes user input, use the function  ```htmlentities``` or the function ```htmlspecialchars```. 
* If you need to use an URL that can include user input, use  the function urlencode


!Look for it in source code
Check where the code outputs information for the HTML page, in data coming from the user, not only in the variables _POST, _GET, REQUEST but also in variables that include the URL for example $_SERVER!['PHP_SELF'].

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
Public domain accorging to Colombian legislation. https://www.pasosdeJesus.org/dominio_publico_colombia.html. 2011. 
