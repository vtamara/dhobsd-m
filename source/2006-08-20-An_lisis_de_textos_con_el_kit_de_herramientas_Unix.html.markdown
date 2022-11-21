---
title: An_lisis_de_textos_con_el_kit_de_herramientas_Unix
date: 2006-08-20 12:00 UTC
tags:
---

Una de las grandes fortalezas de OpenBSD como servidor y cortafuegos es 
precisamente la seguridad. Pero ¿qué es la seguridad?, ¿cómo se manifiesta?. 
En la mayoría de las veces se trata de ataques a través de internet, ataques 
que buscan entre otras cosas: acceder a nuestras máquinas para sacar 
información modificarla, o usar nuestra máquina para que realice trabajos 
que el atacante quiere hacer, generalmente maliciosos.

OpenBSD deja un registro muy completo en /var/log conocidas como bitácoras. 
Para este ejercicio se van a usar algunas herramientas básicas:

- less        : ver una archivo 
- grep        : muestra cadenas en un archivo
- find        : busca una cadena en un archivo o directorio
- wc          : cuenta cadenas en una archivo
- dig         : muestra información de un dominio
- geoiplookup : muestra localización de una ip, no es muy precisa
- gzip        : descomprime archivos


A continuación un ejemplo de auditoria de una bitacora OpenBSD:

1. hacer una copia de las bitácoras de ```/var/log``` en un directorio peronal.
     - `cd /home/personal/`  Crea un directorio para copiar el contenido de 
        las bitácoras
     - `mkdir bk_log` Se ubica en ese directorio
     - `cd bk_log` Al hacer pwd debe aparecer `/home/personal/bk_log`
     - `sudo cp /var/log/* .`

2. Descomprimir los archivos de la bitacora en su directorio personal
   - gzip -d *.gz

  Es posible que antes de descomprimir se deba cambiar los permisos para 
  poder ver los archivos:
  <pre>
  $ sudo chown usuario:usuario *
  </pre>

3. Revisar los archivos a fin de determinar puntos críticos sobre los   
   archivos 
   - `authlog` Muestra los accesos de los usuarios permitidos y rechazados
   - `secure`  Muestra los comandos de los administradores sudo 
   - `daemon`  Los programas qu están corriendo en la máquina.
   - `less authlog`

4. Para ver las líneas en las que aparece una cadena en especial, una ip 
   por ejemplo:
   - `grep "211.157.113.89" authlog`

   De esta forma podemos ver datos claves como ip desde donde se hace el 
   ataque o intento de ingreso, fecha y hora con precisión.

5. Para mostrar cuantas veces está esa ip en ese archivo 

   - `grep "211.157.113.89" authlog | wc -l`

6. Para determinar en que archivos de la bitácoara hay información de esta ip 

   `$ find . -exec grep -l "211.157.113.89" {} ';'`

7. Para determinar en que lugar geográfico fue registrado el DNS del dominio

  - `$ dig -x 211.157.113.89`
  - `$ geoiplookup  211.157.113.89`

  Luego de ubicar el dominio de la ip de donde proviene el ataque, se 
  recomienda enviarle un mensaje en inglés señalando el ataque, con datos 
  como:

  - Dominio que recibio el ataque,
  - Dirección ip desde la que se dirigió el ataque
  - La hora y la fecha precisa

Todas las herramientas usadas en este apartado less, find, grep, wc tienen sus 
repectivos manuales los cuales se pueden consultar con ```man``` por ejemplo:

$ man grep 


