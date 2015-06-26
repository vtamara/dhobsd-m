---
title: Listas_grises_con_spamd
date: 2007-02-10
tags:
---
```spamd``` es un servicio particular de OpenBSD que funciona en llave con ```pf``` para bloquear correos provenientes de IPs listadas en índices actualizados vía Internet.   Puede operar en 3 modos:
* Lista negra (blacklisting)
* Lista gris (graylisting)
* Atrapar grises (gray trapping)

Los detalles de configuración aquí presentados son para OpenBSD 4.2.

##Lista Negra

En este modo hay una lista de IPs prohibidas, todo correo proveniente de una de estas IPs es bloqueado ---en realidad más que sólo bloqueado porque ```spamd``` hace perder tiempo a los MTA que se conectan desde esas IPs.

!Configuración

No se requiere instalar paquetes, todo hace parte del sistema base.

* Editar ```/etc/mail/spamd.conf```.  La configuración por defecto bloquea IPs listadas por Spew así como IPs de China y Korea. Descarga copias de estas listas del sitio oficial de OpenBSD, aunque hay otros sitios, ver http://www.openbsd.org/spamd/
* Iniciar ```spamd``` ejecutando ```/usr/libexec/spamd -b```.  Debe arracarse junto con el sistema, agregando a ```/etc/rc.conf.local```:
<pre>
spamd_flags="-v -b"          # for normal use: "" and see spamd-setup(8)
spamd_black=""
spamd_setup_flags="-b"          # for normal use: "" and see spamd-setup(8)
</pre>
* Preparar ```pf``` para bloquear direcciones que estén en la tabla ```spamd``` (lista negra). Para esto además de asegurarse de que ```pf``` se inicie durante el arranque, ejecutar ```pfctl -f /etc/pf.conf``` después de agregar a ```/etc/pf.conf``` (o quitar comentarios):
<pre>
table <spamd> persist
no rdr on { lo0, lo1 } from any to any
rdr pass inet proto tcp from <spamd> to any port smtp -> 127.0.0.1 port spamd   
</pre>
* Ejecutar ```/usr/libexec/spamd-setup -b``` Este programa descarga copias del listado de spammers tal como se configure en ```/etc/spamd.conf```, después envía el listado a ```spamd``` por el puerto 8026 (o el especificado en ```/etc/services``` como ```spamd-cfg```) y actualiza las tabla ```<spamd>``` de ```pf```.  Esta operación debe hacerse periódicamente para lo cual puede agregarse al crontab del administrador (ejecutar como root ```crontab -e```):
<pre>
1       1       *       *       *       /usr/libexec/spamd-setup -b
</pre>


Una vez en operación ```pf``` redirigirá al puerto 8025  (o el que se configure en ```/etc/services``` como ```spamd```) toda conexión proveniente de una IP que esté en la lista negra.   ```spamd``` rechaza toda conexión que recibe (haciendo perder tiempo al spammer).

!Pruebas

Puede ver como spamd hará perder tiempo al MTA de cada IP listada en la lista negra ejecutando:
<pre>
telnet localhost 8025
</pre>

Puede ver la lista de IPs bloqueadas por pf con:
<pre>
pfctl -t spamd -T show
</pre>
En el momento de este escrito son 74932.

Por la opción ```-v``` al iniciar spamd puede ver en la bitácora de servicios ```/var/log/daemon``` los correos rechazados.

##Lista Gris

En este caso hay una lista negra, una gris y una blanca.

* Es análoga al caso anterior para direcciones que estén en la lista negra.
* Se recibe todo correo que provenga de direcciones que están en la lista blanca.
* Toda dirección que no esté en la lista negra ni en la blanca pasa primero a una lista gris y se solicita al servidor que envió reenviar el mensaje, si el mensaje es reenviado la dirección se pasa a la lista blanca.

La configuración es tal como la de listas negras, sólo que 
* en ```/etc/pf.conf``` debe ser:
<pre>
table <spamd> persist
table <spamd-white> persist
no rdr on { lo0, lo1 } from any to any
rdr pass inet proto tcp from <spamd> to any port smtp -> 127.0.0.1 port spamd
rdr pass inet proto tcp from !<spamd-white> to any port smtp -> 127.0.0.1 port spamd
</pre>
* En ```/etc/rc.conf.local``` :
<pre>
spamd_flags="-v"          # for normal use: "" and see spamd-setup(8)
spamlogd_flags=""       # use eg. "-i interface" and see spamlogd(8)
</pre>
* Si se arranca ```spamd``` no debe usarse la opción ```-b``` (puede ser sin opciones o con ```-g```).
* ```spamd-setup``` no debe usarse con el flag ```-b``` ni al arrancarse desde la línea de comandos en el ```crontab```.

------
Esta información se cede al dominio público y se dedica al Padre que es MISERICORDIA. 2008. vtamara@pasosdeJesus.org
