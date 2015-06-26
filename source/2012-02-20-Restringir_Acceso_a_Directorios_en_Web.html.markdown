---
title: Restringir_Acceso_a_Directorios_en_Web
date: 2012-02-20
tags:
---
##1. La situación

   Supongamos que en su directorio web público(http://www.ejemplo.org/~ffp)  tiene subdirectorios:
<pre>
FOTOS ALEJA
OTOSPARAMO
</pre>
   y desea compartir las fotos del paramo sólo con su amigo Ángel.

##2. Cómo lo hace

   Para lograrlo basta 
# ingresar al servidor,
# hacer una configuración para que el servidor solicite clave cuando se navegue en ese directorio y 
# informar a Ángel el usuario y la clave que le   asignó.

Vemoas estos pasos en detalle:
!2.1. Ingresar al servidor
Desde OpenBSD o Linux abra una terminal y teclee:
<pre>
 ssh correo.ejemplo.org
</pre>

Desde Windows descargue putty de
http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html y
después inicielo indique que se conectará al servidor correo.ejemplo.org.

!2.2. Configurar directorio
En el interprete de comandos ingrese los siguientes comandos que lo
cambiarán al directorio FOTOSPARAMO en su directorio público:
<pre>
cd /var/www/users/ffp/
ls
cd FOTOSPARAMO
mg .htaccess
</pre>
El último comando es un editor que le permite modificar el
archivo .htaccess el cual debe contener:
<pre>
!AuthName "Fotos Paramo para Angel"
!AuthType Basic
!AuthUserFile "/users/ffp/paramo"
!Require user angel
</pre>
Una vez pegue estas líneas (y las modifique de acuerdo a su caso),
salve y salga (en ```mg``` se salva con ```Control-x Control-s``` y se sale con
```Control-x Control-c```)
Después debe crear el archivo de claves paramo con una clave
inicial para su amigo, quien tendrá identificación angel (note que
es en minúsculas en este ejemplo):
<pre>
htpasswd -c paramo angel
</pre>
Deberá entonces dar la clave que Ángel usará 2 veces (al
hacerlo no verá la clave que digita ni asteriscos).

!2.3. Informar a su par
Tras esto ya puede llamar a Ángel e indicarle que ingrese al URL
http://www.ejemplo.org/~ffp/FOTOSPARAMO/ y que cuando le pida
clave ingrese angel con la clave que le asignó.


##3. Bibliografía

*  Uso Básico y Remoto de OpenBSD. http://structio.sf.net/guias/basico_OpenBSD
