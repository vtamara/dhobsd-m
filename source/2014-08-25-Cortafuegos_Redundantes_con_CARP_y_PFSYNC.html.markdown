---
title: Cortafuegos_Redundantes_con_CARP_y_PFSYNC
date: 2014-08-25
tags:
---
Suponemos:

Cortafuegos primario lo llamamos aleph
172.16.50.40
172.16.50.223
172.16.50.224

Cortafuegos secundario beth.


DESACTIVAR

* Poner comentario en ambos cortafuegos la linea del archivo  /etc/hostname.carp1
* Dejar en aleph en /etc/hostname.em0 sin comentario solo la linea
  inet 172.16.50.40 255.255.255.0 NONE
* Eliminar la interfaz carp1 en aleph y beth:
  sudo ifconfig carp1 destroy
* Reiniciar servicios de red en aleph y beth
  sudo sh /etc/netstart
* Reiniciar cortafuegos en ambos:
  sudo pfctl -f /etc/pf.conf

REACTIVAR


* Dejar sin comentario la línea de /etc/hostname.carp1 en aleph y beth
* Dejar en aleph en /etc/hostname.em0 sin comentario solo la línea
  inet 172.16.50.223 255.255.255.0 NONE
* Reiniciar servicios de red aleph y en beth.
  sudo sh /etc/netstart
* Reiniciar cortafuegos en ambos:
  sudo pfctl -f /etc/pf.conf


DIAGNOSTICAR

* Estado de interfaz carp (INIT, MASTER o BACKUP):
<pre>
ifconfig carp2
</pre>
* Bitacora de cambios de ```ifstated```:
<pre>
grep ifstated /var/log/servicio
</pre>
* Bitacora de cambios de ```carp```:
<pre>
grep carp /var/log/messages
</pre>
* Bitácora de cambios de ```pfsync```:
<pre>
grep pfsync /var/log/messages
</pre>
* Estado de carp:
<pre>
netstat -s -p carp                                                                                                           
</pre>
* Estado de pfsync:
netstat -s -p pfsync      
</pre>

REFERENCIAS

* http://openbsd-archive.7691.n7.nabble.com/carp-master-lt-gt-backup-problem-td96700.html 

