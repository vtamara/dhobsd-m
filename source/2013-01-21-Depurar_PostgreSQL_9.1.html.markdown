---
title: Depurar_PostgreSQL_9.1
date: 2013-01-21
tags:
---
En general en adJ (OpenBSD) 6.1, no es fácil depurar aplicaciones que manejan múltiples  procesos.   A continuación presentamos como hacerlo en el caso de PostgreSQL.

## Prerequisitos

Suponemos que:
* Tiene un porte de PostgreSQL "especial" copiando de ```/usr/ports/database/postgresql``` a ```/usr/ports/mystuff/database/postgresql``` y actualizando la versión y siguiendo sugerencia de {2} añadiendo las opciones de configuración ```--enable-debug``` y ```--enable-cassert```.
* Ya compiló las fuentes del porte con ```sudo make``` y  puede encontrarlas en ```/usr/ports/pobj/postgresql-9.6.6/postgresql-9.6.6/```

## Procedimiento

Detenga el servicio PostgreSQL

<pre>
sudo sh /etc/rc.d/postgres stop
</pre>

Inicie con su versión ya compilada en modo de depuración --recomendable dentro de una sesión de ```tmux``` para que pueda examinar la copiosa salida en modo de depuración (```-d 5```):

<pre>
$ cd /usr/ports/pobj/postgresql-9.6.6/postgresql-9.6.6/src/backend
$ sudo su _postgresql -c "./postgres -D /var/postgresql/data -d 5"
</pre>

A continuación inicie un cliente en otra terminal y examine el número del nuevo proceso ```postgres``` que lo atiende:
<pre>
$ psql -h/var/www/tmp -Uusuario mibase
=# SELECT pg_backend_pid(); 
</pre>
En otra terminal, puede usar ese PID (digamos 1234) para "pegarse" al proceso en ejecución con ```gdb```:
<pre>
$ sudo gdb /usr/ports/pobj/postgresql-9.6.6/postgresql-9.6.6/src/backend/postgres
> (gdb) attach 1234
</pre>

Tras esto habrá interumpido el proceso y ya puede depurar normalmente, por ejemplo poniendo un punto de ruptura al comienzo de una función, digamos ```!DefineCollation``` (puede examinarlas en las fuentes), y continuando la ejecución:
<pre>
(gdb) breakpoint !DefineCollation
(gdb) cont
</pre>

Desde el cliente de PostgreSQL ya puede dar comandos.
<pre>
=# CREATE COLLATION esp (LOCALE='es_CO.UTF-8');
</pre>

Y desde la sesión de ```gdb``` una vez el punto de ruptura se alcanzado, puede dar comandos para avanzar como ```next``` o ```step``` o examinar el estado o las demás operaciones de ```gdb```, por ejemplo:

<pre>
Breakpoint 1, !DefineCollation (names=!0x21058dac0, parameters=!0x21058dbe8)
    at collationcmds.c:49
49              !DefElem    *fromEl = NULL;
(gdb) next
50              DefElem    *localeEl = NULL;
(gdb) print names
$1 = (List *) !0x21058dac0
(gdb) print *names
$2 = {type = !T_List, length = 1, head = !0x21058da98, tail = !0x21058da98}
</pre>

##Dedicatoria y Bibliografía

Este material de dominio público se dedica al que dijo que son bienaventurados quienes oyen la palabra de Dios y la guardan (Lc 11:28).

* {1} http://www.biblegateway.com/passage/?search=Lucas+11&version=RVR1960
* {2} http://wiki.postgresql.org/wiki/Developer_FAQ
* {3} http://dirac.org/linux/gdb/06-Debugging_A_Running_Process.php

