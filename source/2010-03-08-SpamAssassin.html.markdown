---
title: SpamAssassin
date: 2010-03-08
tags:
---
 !SpamAssassin es un filtro de correo que suele emplearse en OpenBSD adJ con procmail y con correos en formato Maildir.   


##Configuración y Operación

Ver http://structio.sourceforge.net/guias/servidor_OpenBSD/servicios-correo.html#spam


El servicio ```spamd``` espera conexiones del cliente ```spamc``` para aplicar una secuencia de reglas a un correo y dar un puntaje.

El servicio puede operarse con ```/etc/rc.d/spamassassin```, por ejemplo para iniciarlo:

<pre>
sudo /etc/rc.d/spamassassin start
</pre>

Para iniciar el servicio como parte de la secuencia de agregue ```spamassassin``` en la variable ```pkg_scripts``` de ```/etc/rc.local```.

Debe configurar procmail para reenviar correos a spamc, este cambiará el encabezado del correo para agregar el resultado de !SpamAssassin.  
Si el encabezado incluye un puntaje de Spam alto (mayor a 5 por defecto) o  incluye el encabezado ```X-Spam-Status: Yes```, debe tratarse como Spam y enviarse a un carpeta para esto --la estándar para la mayoría de clientes de correo es Junk--.

El archivo .procmailrc de un usuario ```ejemplo``` debería entonces incluir:
<pre>
:0fw: spamassassin.lock
* < 512000
| spamc

:0:
* ^X-Spam-Status: Yes
/home/ejemplo/Maildir/.Junk/
</pre>

Las reglas que usa se encuentran en ```/usr/local/share/spamassassin/``` y en ```/etc/mail/spamassassin/```.  Cada regla asigna un puntaje.
 

##Mantenimiento

Debe ejecutarse periódicamente (por ejemplo agregando a ```/etc/weekly.local```):
<pre>
sudo sa-update
</pre>


##Recetas para bloquear correos o solucionar problemas

!Bloquear correo en codificación rusa

Receta de http://www.void.gr/kargig/blog/2011/05/01/block-spam-with-russian-encoding-using-spamassassin/

Agregar a ```/etc/mail/spamassassin/local.cf```

<pre>
header !LOCAL_CHARSET_BLOCKED Subject:raw =~/\=\?(koi8-r|windows-1251)\?/i
score !LOCAL_CHARSET_BLOCKED 5
describe !LOCAL_CHARSET_BLOCKED Contains charsets that are not acceptable
</pre>


!Si supone que fechas posteriores al 2010 son muy futuras:

Editar ```/usr/local/share/spamassassin/72_active.cf``` y cambiar
<pre>
##{ !FH_DATE_PAST_20XX
header   !FH_DATE_PAST_20XX      Date =~ /20![1-9]![0-9]/ ![if-unset: 2006]
describe !FH_DATE_PAST_20XX      The date is grossly in the future.
##} !FH_DATE_PAST_20XX
</pre>
por
<pre>
##{ !FH_DATE_PAST_20XX
header   !FH_DATE_PAST_20XX      Date =~ /20![2-9]![0-9]/ ![if-unset: 2006]
describe !FH_DATE_PAST_20XX      The date is grossly in the future.
##} !FH_DATE_PAST_20XX
</pre>
