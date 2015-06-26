---
title: ProtocoloEmergencias
date: 2008-10-31
tags:
---
## CASO: Falla de suministro electríco.

DIAGNÓSTICO:  Bombillos de la CPU están apagados.

SOLUCIÓN: Encender 

##CASO: Falla en conexión a proveedor de Intenret

DIAGNÓSTICO: Bombillo de conexión remota del modem/enrutador apagado. Al conectar un teléfono directamente a la toma y al escuchar no da tono. El cortafuegos responde ping, pero no el modem:


SOLUCIÓN: Llamar al proveedor y solicitar soporte.  Tener a manor n?mero  de NIT, dirección, eventualmente número de la línea telefónica.

## CASO: Falla en disco después de apagarse abruptamente 

DIAGNÓSTICO: Al conectar el monitor a la CPU se ve un mensaje del estilo:
<pre>
/dev/rwd0a ... run fsck_ffs ...
Manual repair is needed
...ENTER for sh
</pre>

SOLUCIÓN: Presionar ENTER, si pregunta tipo de terminal emplear vt220
una vez en el prompt de unix (símbolo #) ejectuar:
<pre>
	fsck_ffs -y
</pre>
Una vez termine la reparación manual reiniciar con:
<pre>
	reboot
</pre>

## CASO: Reiniciar servicios faltantes remotamente

<pre>
ssh miusuario@miservidor.org
sudo sh /etc/rc.local
</pre>

## CASO: Debe apagarse el servidor 

SOLUCIÓN: Ingresar a una cuenta en el servidor remotamente o desde la consola y dar:
<pre>
      sudo halt -p
</pre>
Para ingresar remotamente desde un sistema tipo Unix (como Linux) abrir una términal y teclear:
<pre>
      ssh miusuario@miservidor.org
</pre>
Desde Windows XP/Vista utilizar putty ---ver instrucciones en 
http://structio.sourceforge.net/guias/basico_OpenBSD/index.html#ssh

Si no resulta posible, apagar con el  botón de encendido --que podría
generar problemas en el disco que requieran reparación manual	como en el caso anterior.


##CASO: No permite ver correos

DIAGNÓSTICO: Ingresa a thunderbird o un cliente de correo para el web (como Horde) pero no le permite ver sus correos.

SOLUCIÓN: Intentar ingresar al servidor (con ssh) y reiniciar el servicio de correo y otros que puedan faltar con:
<pre>
    sudo sh /etc/rc.local
</pre>


##CASO: No recibe algunos correos

DIAGNÓSTICO: Algunas personas le avisan que le han enviado correos pero que rebotan o simplemente no los recibe.

SOLUCIÓN: Puede ser el sistema antispam.  Intentar pasando de listas grises a listas negras.  Ver
http://dhobsd.pasosdejesus.org/index.php?id=Listas+grises+con+spamd

##CASO: No permite iniciar SIVeL

DIAGNÓSTICO: Al intentar abrir desde el navegador aparece un mensaje como:
<pre>
DB Error: connect failed
[nativecode=pg_connect() [function.pg-connect]: Unable to connect to PostgreSQL server: 
could not connect to server: No such file or directory 
Is the server running locally and accepting connections on Unix domain socket "/tmp//.s.PGSQL.5432"?] **
</pre>


SOLUCIÓN: Posiblemente ingreso mal la clave de encripción, reinicie servicios faltantes y de correctamente la clave de encripción.  Puede reiniciar servicios faltantes de una de estas formas:
* Desde el menu de fluxbox Dispositivos->Iniciar Servicios Faltantes.
* Desde una términal o una consola virtual tecleando:
<pre>
sudo sh /etc/rc.local
</pre>
* Apagar y prender el computador

Después de iniciar servicios faltantes vuelva al navegador, presione el botón derecho del mouse y elija Recargar.
