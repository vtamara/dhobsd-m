---
title: nsd_unbound
date: 2014-10-29
tags:
---
OpenBSD 5.7 eliminará BIND (binario ```named```) que puede realizar funciones de servidor de nombres autoritario y servidor de nombres recursivo. Estas funcionalidades las puede tener con ```nsd``` y ```unbound``` respectivamente.  El sistema de OpenBSD (así como adJ) base incluye NSD desde 5.4 e incluye unbound desde 5.6 --unbound está como paquete en 5.5.  

En esta guia explicamos brevemente NSD y Unbound enfocados en sustituir un servidor BIND con vista autoriaria y vista recursiva.

#1. INTRODUCCIÓN

Un servidor autoritario para un dominio, es la fuente autoritaria sobre un dominio.    Es requerido para manejar un dominio y típicamente responde consultas que se hagan desde Internet más no desde la red local.   Si ya tiene un named con vistas operando, con NSD debe configurar la parte que están en la vista autoritaria  del archivo ```/var/named/etc/named.conf```

Un servidor recursivo recibe consultas de dominios y las reenvia a otros servidores --comenzando por los servidores raiz-- o si tiene respuesta a las consultas en su repositorio temporal fresco (cache) lo usa para responder.  Es útil para responder consultas de una red local rapidamente, y en tal caso debe responder consultas que se hagan desde la red interna pero no desde Internet, como posiblemente ocurre con la vista recursiva del archivo ```/var/named/etc/named.conf``` si tiene uno.


#2. CONFIGURACIÓN

##2.1 Servidor autoritario con NSD

Agregue a /etc/rc.conf.local la línea:
<pre>
nsd_flags="-c /var/nsd/etc/nsd.conf"
</pre>
e incluya ```nsd``` en la variable ```pkg_scripts```

El archivo de configuración principal ubiquelo es ```/var/nsd/etc/nsd.conf```,
por cada zona que maneje de manera autoritaria (es decir cada zona en la vista ```view "authoritative``` de /var/named/etc/named.conf) incluya líneas de la forma:
<pre>
zone:
  name: "miescuela.edu.co"
  zonefile: "miescuela.edu.co"
</pre>
Para que responda hacía Internet en un cortafuegos con IP pública (digamos 200.201.202.203) en el mismo archivo asegurese de dejar:
<pre>
ip-address: 200.201.202.203
</pre>

En el directorio ```/var/nsd/zones``` debe dejar un archivo de zona por cada zona que configure.  Afortunadamente NSD reconoce la misma sintaxis de archivos de zona que ```bind```, así que basta que copie los de las zonas autoritarías (que tipicamente se ubican en ```/var/named/master/```).  Un ejemplo de un archivo de zona /var/nsd/zones/miescula.edu.co``` es:


<pre>
$ORIGIN miescuela.edu.co.
$TTL 6h

@ IN SOA ns1.miescuela.edu.co. root.localhost. (
	2 ; Serial
	1d ; Refresco secundario
	6h ; Reintento secundario
	2d ; Expiracion secndaria
	1d ) ; Cache 

	NS	ns1
	A	200.201.202.203
	MX 	5	correo.miescuela.edu.co.
correo A	200.201.202.203
ns1	A	200.201.202.203
*	A	200.201.202.203
</pre>

Inicie el servicio con
<pre>
sudo sh  /etc/rc.d/nsd start
</pre>
(o reinicielo con ```restart``` en lugar de ```start```).

Revise posibles errores en las bitacoras ```/var/log/messages``` y ```/var/log/servicio```

Pruebe que responde con:
<pre>
dig @200.201.202.203 correo.miescuela.edu.co
</pre>
que debería dar la IP pública.

## 2.2. Servidor Recursivo con Unbodund

En ```/etc/rc.conf.local``` agregue
<pre>
unbound_flags="-c /var/unbound/etc/unbound.conf"
</pre>
Y a la varialbe ```pkg_scripts``` agreguele ```unbound```

Configurelo en ```/var/unbound/etc/unbound.conf```, asegurese de cambiar las siguientes líneas:
* Si su cortafuegos tiene en la red interna la IP 192.168.100.100 responda sólo a esa interfaz:
<pre>
interface: 192.168.100.100
</pre>
* Permita consultas desde la red interna:
<pre>
access-control: 192.168.100.0/24 allow
</pre>
* Las zonas autoritarias que nsd esté sirviendo también debe responderlas de manera autoritaria con unbound pero dirigiendo a la red interna, por ejemplo respecto al ejemplo de la sección anterior, suponiendo que en la red Interna el servidor que responde correo es 192.168.100.101:
<pre>
local-zone: "miescuela.edu.co." static
local-data: "correo.miescuela.edu.co. IN A 192.168.100.101"
local-data: "ns1.miescuela.edu.co. IN A 192.168.100.100"
local-zone: "100.168.192.in-addr.arpa." static
local-data-ptr: "192.168.100.101 correo.miescuela.edu.co."
local-data-ptr: "192.168.100.100 ns1.miescuela.edu.co."
</pre>

Inicie el servicio con
<pre>
sudo sh  /etc/rc.d/unbound start
</pre>

Revise posibles errores en las bitacoras ```/var/log/messages``` y ```/var/log/servicio```

Pruebe que responde con:
<pre>
dig @192.168.100.100 correo.miescuela.edu.co
</pre>

que debería dar la IP privada.


#3. REFERENCIAS

* {1} https://calomel.org/nsd_dns.html
* {2} http://eradman.com/posts/run-your-own-server.html
* {3} https://calomel.org/unbound_dns.html
