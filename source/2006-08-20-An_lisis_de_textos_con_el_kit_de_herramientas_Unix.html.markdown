---
title: An_lisis_de_textos_con_el_kit_de_herramientas_Unix
date: 2006-08-20
tags:
---
Una de las grandes fortalezas de OpenBSD como servidor y cortafuegos es precisamente la seguridad. Pero qué es la seguridad?, cómo se manifiesta?. En la mayoría de las veces se trata de ataques a través de internet, ataques que buscan entre otras cosas: acceder a nuestras máquinas para sacar información modificarla, o usar nuestra máquina para que realice trabajos que el atacante quiere hacer, generalmente maliciosos.

OpenBSD deja un registro muy completo en /var/log conocidas como bitácoras. para este ejercicio se van a usar algunas herramientas básicas:<br>

- less        : ver una archivo 
- grep        : muestra cadenas en un archivo
- find        : busca una cadena en un archivo o directorio
- wc          : cuenta cadenas en una archivo
- dig         : muestra información de un dominio
- geoiplookup : muestra localización de una ip, no es muy precisa
- gzip        : descomprime archivos


A continuación un ejemplo de auditoria de una bitacora OpenBSD:

1. hacer una copia de las bitácoras de ```/var/log``` en un directorio peronal.<br>
     - cd /home/personal/<br>
   - Crea un directorio para copiar el contenido de las bitácoras<br>
     - mkdir bk_log<br>
   - Se ubica en ese directorio<br>
     - cd bk_log<br>
   - Al hacer pwd debe aparecer /home/personal/bk_log<br>
     - sudo cp /var/log/* .<br>

2. Descomprimir los archivos de la bitacora en su directorio personal<br>
   - gzip -d *.gz<br>

Es posbile que antes de descomprimir se deba cambiar los permisos para poder ver los archivos:<br>

$ sudo chown usuario:usuario * <br>

3. Revisar los archivos a fin de derterminar puntos críticos sobre los   
   archivos <br>
   - authlog Muestra los accesos de los usuarios permitidos y rechazados<br>
   - secure  Muestra los comandos de los administradores sudo<br> 
   - daemon  Los programas qu están corriendo en la máquina.<br>
     
     - less authlog <br>

4. Para ver las líneas en las que aparece una cadena en especial, una ip por ejemplo:<br>
   - grep "211.157.113.89" authlog<br>

De esta forma podemos ver datos claves como ip desde donde se hace el ataque o intento de ingreso, fecha y hora con precisión.<br>

5. Para mostrar cuantas veces está esa ip en ese archivo <br>

   - grep "211.157.113.89" authlog | wc -l<br>

6. Para determinar en que archivos de la bitácoara hay información de esta ip 
   <br>
   $ find . -exec grep -l "211.157.113.89" {} ';'<br>

7. Para determinar en que lugar geográfico fue registrado el DNS del dominio

  - $ dig -x 211.157.113.89
  - $ geoiplookup  211.157.113.89

Luego de ubicar el dominio de la ip de donde proviene el ataque, se recomienda enviarle un mensaje en inglés señalando el ataque, con datos como:

- Dominio que recibio el ataque,<br>
- Dirección ip desde la que se dirigió el ataque<br>
- La hora y la fecha precisa<br>

 <br>
Todas las herramientas usadas en este apartado less, find, grep, wc tienen sus repectivos manuales los cuales se pueden consultar con ```man``` por ejemplo:<br><br>

$ man grep ...


   
