---
title: Manejador_de_listas_mlmmj
date: 2007-01-26 12:00 UTC
tags:
---
A diferencia de mailman, mlmmj puede emplearse en un OpenBSD que tenga un servidor web con chroot, pues se administra desde la línea de comandos.  
Lo mismo ocurre con majordomo, aunque la licencia de mlmmj es MIT mientras
que la de majordomo es restrictiva.

##INSTALACIÓN Y CONFIGURACIÓN

* Instale el paquete disponible entre los paquetes oficiales de OpenBSD.
* Agregue una tarea cron "sudo crontab -e":
<pre>
0 */2 * * * "/usr/local/bin/mlmmj-maintd -F -d /var/spool/mlmmj"
</pre>
* Cuando se usa mlmmj con sendmail, debe modificarse la configuración de sendmail:
<pre>
cd /usr/share/sendmail/cf/
vi openbsd-proto.mc
</pre>
para agregar (digamos antes de ```MAILER(local)dnl```):
<pre>
define(`LOCAL_SHELL_FLAGS', `eu9P')dnl 
</pre>
e instale la nueva configuración con
<pre>
make
make distribution
sudo pkill sendmail
. /etc/rc.conf
sendmail ${sendmail_flags}
</pre>


##ADMINISTRACIÓN DE LISTAS

!CREACIÓN
Puede crear una nueva lista (digamos l-amigos) con:
<pre>
sudo /usr/local/bin/mlmmj-make-ml
chown -R daemon /var/spool/mlmmj/l-amigos
</pre>
Agregue un alias en /etc/mail/aliases por ejemplo:
<pre>
l-amigos:  "|/usr/local/bin/mlmmj-recieve -L /var/spool/mlmmj/l-amigos/"
</pre>
Entre las diversas opciones que pueden configurarse están:
* Lista cerrada i.e no pueden suscribirse miembros por correo:
<pre>
touch /var/spool/mlmmj/l-amigos/control/closedlist
</pre>

!ADICIÓN DE USUARIOS

<pre>
/usr/local/bin/mlmmj-sub -L /var/spool/mlmmj/l-amigos -a miamigo@pasosdeJesus.org
</pre>


----
Esta información se cede al dominio público y se dedica al padre que es JUSTICIA. 2007. vtamara@pasosdeJesus.org
