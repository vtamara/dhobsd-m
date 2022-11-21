---
title: Internet_por_telefono_movil
date: 2011-12-30 12:00 UTC
tags:
---
Es posible conectarse a Internet mediantun teléfono movil con plan de datos.


!COMCEL

En el caso de Comcel con un teléfono movil Nokia E72 puede usar la siguiente configuración en ```/etc/ppp/ppp.conf```:
<pre>
comcel:
 set device /dev/cuaU0
 set dial "ABORT ERROR ABORT BUSY ABORT NO\\sCARRIER TIMEOUT 5 \
           \"\" ATZ OK-ATZ-OK AT+CGDCONT=1,\\\"IP\\\",\\\"internet.comcel.com.co
\\\" OK \\dATD\\T TIMEOUT 40 CONNECT"
 set phone "*99#"
 set speed 460800
 set login
 set authname "COMCELWEB"
 set authkey "COMCELWEB"
 #enable mssfixup
 set timeout 0
 set ifaddr 10.0.0.1/0 10.0.0.2/0 0.0.0.0 0.0.0.0
 add! default HISADDR
 #add default HISADDR
 enable dns
 disable ipv6cp
</pre>

Al momento de conectar el teléfono por el puerto USB elegir "PC suite" en el teléfono y desde OpenBSD ejecutar:
<pre>
sudo ppp -auto comcel
</pre>
Que después de un tiempo debe asignarle una IP que puede revisar con
<pre>
ifconfig
</pre>



##BIBLIOGRAFIÁ

* http://www.mail-archive.com/misc@openbsd.org/msg91410.html
* Configuración en Ubuntu 11.10 para Comcel.
