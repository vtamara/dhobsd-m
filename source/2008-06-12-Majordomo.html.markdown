---
title: Majordomo
date: 2008-06-12 12:00 UTC
tags:
---

```majordomo``` es un programa para administrar listas de correo.   Aunque hay otros programas análogos disponibles para OpenBSD y en particular  ```mailman``` que es muy configurable y tiene una interfaz web bastante usable, la ventaja de ```majordomo``` es que no requiere un servidor web.   ```mailman``` por defecto requiere el servidor web ejecutándose fuera de la jaula ```chroot``` lo que aumenta vulnerabilidad del sistema operativo.

A continuación presentamos uso, instalación y configuración con OpenSMTPD, se ha probado con exito en OpenBSD 5.9

##USO

Una lista de correo retransmite un correo enviado por un suscriptor autorizado a los demás suscriptores de la lista. 

Para suscribir/desuscribir correos y realizar otras operaciones debe enviar un correo a majordomo@listas.pasosdeJesus.org con comandos.  Por ejemplo:
<pre>
help
</pre>
recibirá en su correo los comandos que puede enviar.  Note que debe enviar los mensajes en el cuerpo del mensaje y no en el tema.

Algunos de los comandos más típicos son:

* ```lists``` que retorna las listas públicas disponibles en el servidor.
* ```info milista``` que da información sobre la lista ```milista```
* ```subscribe milista direccion@mi.dominio``` que suscribe el correo ```direccion@mi.dominio``` a la lista ```milista```
* ```unsubscribe milista direccion@mi.dominio``` que desuscribe un correo de la lista.
* ```who milista``` retorna la lista de suscriptores a la lista ```milista``` ---si tiene permiso para revisarla.
* ```get carta.txt milista``` descarga el archivo ```carta.txt``` relacionado con la lista ```milista```
* ```index milista``` retorna listado de archivos relacionados con la lista ```milista```
* ```which direccion@mi.dominio``` retorna listado de listas a las cuales está suscrita la dirección.
* ```end``` termina procesamiento de comandos del mensaje


##INSTALACIÓN Y CONFIGURACIÓN

* Instale el paquete ```majordomo```. ```cd /usr/ports/mail/majordomo; doas make install```
* Agregue el usuario ```servicio``` al grupo _majordomo editando /etc/groups o ejecutando ```doas usermod -G _majordomo servicio```
* Edite ```/etc/majordomo/majordomo.cf``` y asegurese de establecer al menos el nombre de la máquina (```whereami```).
* Establezca una archivo de aliases para su dominio en /etc/mail/smtpd.conf por ejemplo:
<pre>
table aliasesp db:/etc/mail/aliasesp.db
accept from any for domain "pasosdeJesus.org" alias <aliasesp> deliver to maildir
</pre>
* Agregue aliases iniciales en ```/etc/mail/aliasep```
<pre>
   majordomo: "|/usr/local/lib/majordomo/wrapper majordomo"
   Majordomo-Owner: postmaster
</pre>
* Cada vez que cambie ese archivo de aliases ejecute:
<pre>
 makemap -t aliases /etc/mail/aliasesp
</pre>
* Cambie dueño:
<pre>
chown _smtpd:_smtpd /usr/local/lib/majordomo/wrapper
</pre>

##CREACIÓN DE UNA NUEVA LISTA

<pre>
sudo su -
cd /var/spool/majordomo/lists
touch milista
$EDITOR milista.info
chmod 664 milista*
mkdir -p milista.archivo
chmod 775 milista.archivo
chown -R _smtpd:_smtpd milista*
chmod g-w milista
</pre>

Al editar el archivo ```milista.info``` ponga una descripción de la lista que
se presentará cuando los usuarios empleen el comando info.

En el archivo ```/var/spool/majordomo/lists/milista``` deje la lista
inicial de correos.

Después edite ```/etc/mail/aliases``` para agregar (remplace miusuario por el usuario que administrará la lista):

<pre>
milista:"|/usr/local/lib/majordomo/wrapper resend -l milista milista-saliente"
milista-approval: miusuario
owner-milista:   miusuario 
milista-owner:   miusuario 
milista-request: "|/usr/local/lib/majordomo/wrapper majordomo -l milista"
owner-milista-saliente: owner-milista
milista-saliente: :include:/var/spool/majordomo/lists/milista, \
 "| /usr/local/lib/majordomo/wrapper archive2.pl -a -m \
 -f /var/spool/majordomo/lists/milista.archivo/milista.archivo"
</pre>

Reconstruya bases de datos de correo con:
<pre>
cd /etc/mail
doas make
</pre>
y reinicie ```OpenSMTPD```:
<pre>
doas sh /etc/rc.d/smtpd -d restart
</pre>

Desde un cliente de correo envie a ```majordomo@listas.pasosdeJesus.org``` el mensaje:
<pre>
config milista milista.admin
</pre>

Esto generará un archivo inicial de configuración ```/var/spool/majordomo/lists/milista.config``` el cual debe editar 
para cambiar por lo menos las claves y establecer si se trata de una lista moderada o no moderada. Las variables, posiblemente, más relevantes son: 
* ```admin_passwd``` 
* ```approve_passwd```
* ```moderate``` 
* ```moderator```
* ```subscribe_policy```
* ```maxlength```

La configuración que ha realizado deja un archivo de la lista en el directorio ```/var/spool/majordomo/lists/milista.archivo```

Al enviar un correo a milista@listas.pasosdeJesus.org debería llegarle a todos los suscriptores, si no ocurre revise el correo que le debe responder. Un error común es dejar el archivo ```/var/spool/majordomo/lists/milista``` con permiso de escritura para el grupo que se cambia con:
<pre>
chmod g-w /var/spool/majordomo/lists/milista
</pre>

El moderador podrá suscribir direcciones enviando un correo a:
```majordomo@listas.pasosdeJesus.org``` con el cuerpo
<pre>
subscribe 
</pre>

##REFERENCIAS

* Documentación de Majordomo ```/usr/local/share/doc/majordomo/NEWLIST```
* Documentación de Majordomo para OpenBSD ```/usr/local/share/doc/majordomo/post-install-note```

------
Esta información se dedica al Espíritu Santo por su consuelo y se libera al dominio público. vtamara@pasosdeJesus.org 2008.
