---
title: SoporteServidorCorreo
date: 2010-08-25
tags:
---
Las siguientes operaciones se hacen para administrar un servidor de correo.  Hay otras operaciones que se hacen con el [Cliente de Correo].


## MONITOREO Y SOPORTE

! Examinar bitacoras de correo
<pre>
sudo less /var/log/maillog
</pre>

Otras archivadas están comprimidas y numeradas por ejemplo puede verse la 5 con:
<pre>
sudo zless /var/log/maillog.5.gz
</pre>

Cada linea especifica un envio, una recepcion o una falla, en cada una el  remitente se ve despues de ```from= ``` y el receptor despues de ```to= ``` eventualmente tras ```ctrladdr= ```

En el paginador ```less``` es util la letra ```G``` para ir al final, ```/``` para buscar hacia adelante y ```?``` para buscar hacia atras

Por ejemplo para buscar correos enviados o recibidos por la cuenta webmaster

<pre>
sudo zgrep webmaster /var/log/maillog*
</pre>


! Reiniciar servidor de correo

Para ver si el servicio de correo esta  operando
<pre>
pgrep sendmail
</pre>

Para reiniciarlo:
<pre>
sudo pkill sendmail
sudo sh /etc/rc.local
</pre>

! Reiniciar servidor

<pre>
sudo reboot
</pre>

Cuando arranca deben darse 3 claves de encripción.  Si se opera desde el mismo servidor cuando aparece:
<pre>
Encriptyion Key:
</pre>

Remotamente ingresando al servidor y ejecutando:

<pre>
sudo sh /etc/rc.local
</pre>


## ADMINISTRACIÓN DE CUENTAS

! Ver cuentas del sistema

<pre>
sudo less /etc/passwd
</pre>

! Crear una nueva cuenta o actualizar directorios de una cuenta existente:

<pre>
sudo su -
cd servidor/audita
./ndirper.sh nuevacuenta
</pre>


! Borrar una cuenta

<pre>
sudo rmuser webmaster
</pre>

! Cambiar clave de una cuenta

<pre>
sudo passwd comunicaciones
</pre>
