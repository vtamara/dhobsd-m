---
title: squid_y_Dansguardian
date: 2011-05-05
tags:
---
Dansguardian con squid permiten filtar Internet de contenidos no apropiados.


##1. Instalación

Primero instale el paquete squid, configurelo para que al menos acepto conexiones del mismo computador:

<pre>
acl localnet src 127.0.0.1/32
</pre>
Inicielo con 

<pre>
/etc/init.d/squid start
</pre>

Cuando squid opera deja una bitacora en /var/squid/logs/access.log 
Y pruebe que opera configurando como proxy http en firefox el computador donde instaló por el puerto 3128.  
Cuando squid opera deja una bitacora en /var/squid/logs/access.log 

Instale a continuación el paquete dansguardian.  No  hay mucho que configurar excepto el idioma:
<pre>
language = 'spanish'
</pre>



##2. Uso 
/etc/dansguardian/lists/banned.....
Debe ser modificando  /etc/dansguardian/lists/bannedregexpurllist
o
/etc/dansguardian/lists/bannedregexpheaderlist 
Puede verse una corta explicación de los archivos en 
http://ciclo.tuinstituto.es/?q=node/412


