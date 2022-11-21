---
title: POP3S_e_IMAPS_en_OpenBSD_LDAP_Sendmail
date: 2005-12-20 12:00 UTC
tags:
---
Este artículo describe como configurar un servidor de correo OpenBSD para
que los usuarios puedan sacar sus correos usando POP3-SSL e IMAP-SSL ambos
autenticandose con un servidor LDAP.


##INTRODUCCIÓN

POP3 e IMAP son protocolos que permiten sacar/examinar correos de un servidor
de correos, para eventualmente transferirlos y permitir al usuario final
manejarlos en su estación de trabajo con su MUA (Mail User Agent - Agente de
Correo para el Usuario).

POP3 se describe en el RFC 1939 e IMAP4 en el RFC 1730. Tipicamente los
clientes y servidores de POP3 e IMAP transmiten la información sin encriptar
(incluyendo login y clave del usuario), por esto es buena práctica emplear
estos servicios sobre un canal SSL.

En el momento de este escrito, el único servidor POP3 de fuentes abiertas
que soporta autenticación con LDAP que conocemos es ```courier-pop3```. También
está disponible courier-imap que soporta IMAP con LDAP. Ambos cuenta con
paquetes para OpenBSD, sin embargos ambos esperan encontrar el correo que
recibe cada usuario en formato Maildir, por lo que es necesario acoplarlos
con ```sendmail``` que por defecto salva correos en formato ```MBOX```.

Suponemos que el sistema donde está el correo ya cuenta con una configuración
de LDAP como la presentada en el artículo [Autenticacion con OpenLDAP].


##INSTALACIÓN Y CONFIGURACIÓN

Instale los paquetes ```courier-imap```, ```courier-ldap```, ```courier-pop3``` y
```courier-utils```.

Todos estos paquetes se configuran en el directorio ```/etc/courier-imap```.
Después de instalarlos copie los archivos de configuración de ejemplo:
<pre>
# cp /usr/local/share/examples/courier-imap/* /etc/courier-ldap
</pre>
y editelos.

En ```imapd``` por lo menos debe cambiar:
<pre>
IMAPDSTART=YES
MAILDIRPATH=maildir
</pre>
En pop3d
<pre>
POP3DSTART=YES
MAILDIRPATH=maildir
</pre>

En ```authldap``` por lo menos debe cambiar:

<pre>
LDAP_SERVER           correo.pasosdeJesus.org

LDAP_BASEDN           ou=gente,dc=correo,dc=pasosdeJesus,dc=org

LDAP_BINDDN           ou=sendmail,dc=correo,dc=pasosdeJesus.org
LDAP_BINDPW           secret

LDAP_AUTHBIND         1

LDAP_FILTER (!(quota=-1))

LDAP_DOMAIN           correo.pasosdeJesus.org

LDAP_MAILDIR          maildir

#LDAP_DEFAULTDELIVERY defaultDelivery

LDAP_UID              uidNumber
LDAP_GID              gidNumber
</pre>

Para emplear SSL requiere certificados. Si tiene certificados firmados por
Autoridades Certificadoras para esto, úselos, pero si requiere generar
certificados autofirmados modifique también la información de los archivos
imapd.cnf y pop3d.cnf y ejecute:

<pre>
# mkimapdcert
# mkpop3dcert
</pre>

##FORMA DE USO Y PRUEBAS

!POP3-SSL

Para iniciar POP3S ejecute:
<pre>
sudo mkdir -p /var/run/courier-imap
sudo /usr/local/libexec/pop3d-ssl.rc start
</pre>
líneas que se recomienda agregue a ```/etc/rc.local``` si planea prestar
servicio continuo (y abrir el puerto apropiado del contrafuegos, ver
a continuación).

Para detener POP3S: 
<pre>
sudo /usr/local/libexec/pop3d-ssl.rc stop
</pre>

POP3 usa por defecto el puerto 110, al encriptarse sobre SSL es estándar
emplear el puerto 995. Para abrir ese puerto en el contrafuegos en
```/etc/pf.conf``` podría emplear una línea de la forma:
<pre>
pass in on $ext_if proto tcp to ($ext_if) port pop3s keep state
</pre>

Puede probar el funcionamiento del servidor con:
<pre>
$ openssl s_client -connect localhost:995 
</pre>

Una sesión típica puede ser (lo que está en negrillas es lo que sumerce debería teclear haciendo los cambios de acuerdo a login y clave):
<pre>
+OK Hello there.
user ejem2
+OK Password required.
pass ejem2
+OK logged in.
list
+OK POP3 clients that break here, they violate STD53.
1 17559
2 1128
3 2430
. 
</pre>

Para que pop3s maneje cuentas virtuales deben usarse en ```/etc/authldaprc```
las variables ```LDAP_GLOB_UID``` y ```LDAP_GLOB_GID```. Pero si se desea que
```pop3s``` emplee el uid y gid de cada cuenta no use las anteriores sino:
<pre>
LDAP_UID                uidNumber
LDAP_GID                gidNumber
</pre>

!IMAP-SSL

Para iniciar IMAPS:
<pre>
sudo mkdir -p /var/run/courier-imap
sudo /usr/local/libexec/imapd-ssl.rc start
</pre>
y para detenerlo:
<pre>
sudo /usr/local/libexec/imapd-ssl.rc stop
</pre>

En conexiones no encriptadas emplea el puerto 143, y para conexiones sobre
SSL el puerto 993 es estándar. Puede probarlo de manera análoga a POP3,
aunque el protocolo es diferente:
<pre>
$ openssl s_client -connect localhost:993
...

AB LOGIN pablo MiClave
AB OK LOGIN Ok.
BC SELECT "Inbox"
* FLAGS (\Draft \Answered \Flagged \Deleted \Seen \Recent)
* OK ![PERMANENTFLAGS (\* ...
* 19 EXISTS
* 19 RECENT
* OK [UIDVALIDITY 1119870003] Ok
* OK [MYRIGHTS "acdilrsw"] ACL
BC OK [READ-WRITE] Ok
ZZZZ LOGOUT
* BYE Courier-IMAP server shutting down
ZZZZ OK LOGOUT completed
</pre>


##ACOPLAMIENTO CON sendmail

Hay diversos métodos para lograr que sendmail guarde en formato Maildir, una
sencilla solución que no requiere mayores cambios es emplear para esto
```procmail```.

Instale el paquete ```procmail``` y en la cuenta de cada usuario que vaya a usar
POP3S o IMAPS con autenticación LDAP cree los archivos ```.forward``` y
```.procmailrc```, análogos a los siguientes (suponemos que se trata del
usuario ```pablo```):

En /home/pablo/.forward
<pre>
"| exec /usr/local/bin/procmail"
</pre>

En ```/home/pablo/.procmailrc```
<pre>
LINEBUF=4096
#VERBOSE=on
PMDIR=/home/pablo/
MAILDIR=$PMDIR/maildir/
FORMAIL=/usr/local/bin/formail
SENDMAIL=/usr/sbin/sendmail
#LOGFILE=$PMDIR/log

:0
* .*
/home/pablo/maildir/
</pre>

Deje además listo un directorio en formato Maildir en ```/home/pablo/maildir```
con:
<pre>
maildirmake /home/pablo/maildir
chown -R pablo:estudiante /home/pablo/maildir
</pre>
De esta forma cada vez que sendmail reciba un correo para el usuario local
```pablo```, ejecutará la línea del archivo ```/home/pablo/.forward```, la cual a
su vez ejecutará procmail para procesar el correo que llega por entrada
estándar. ```procmail``` empleará la configuración de ```/home/pablo/.procmailrc```
que le indica guardar todo correo que llegue a la cuenta en
```/home/pablo/maildir/``` (como se trata de un directorio, ```procmail```
identifica que debe salvar en formato ```Maildir```, si fuera un archivo
agregaría en formato ```MBOX```).

El usuario pablo podría probar su archivo de configuración de ```procmail```
modificando ```~/.procmail``` para quitar el comentario de la línea
<pre>
VERBOSE=on
</pre>
y ejecutando:
<pre>
$ cd /home/pablo
$ procmail 
Mensaje de prueba
Terminelo con Control-D
.
procmail: [21024] Fri Jul  1 18:32:30 2005
procmail: Assigning "PMDIR=/home/pablo/"
procmail: Assigning "MAILDIR=/home/pablo//maildir/"
procmail: Assigning "FORMAIL=/usr/local/bin/formail"
procmail: Assigning "SENDMAIL=/usr/sbin/sendmail"
procmail: Assigning "LOGFILE=/home/pablo/log"
</pre>
Tras lo cual debe encontrar un nuevo archivo en ```maildir/new``` con el mensaje
de prueba.

##CONCLUSIÓN

```sendmail``` en OpenBSD por defecto almacena correos que recibe en
```/var/mail``` en formato ```MBOX```. El formato Maildir para mantener correos
es propio de ```qmail``` y ```courier```, aunque ```sendmail``` no lo soporta por
defecto es posible configurarlo para habilitar servicios e implementaciones
que aprovechan este formato. Por ejemplo el protocolo IMAP además de ofrecer
las capacidades de POP3, permite mantener carpetas y algunas implementaciones
soportan cuotas, características que pueden aprovecharse cuando el formato
de los correos es Maildir.

Aunque el registro de seguridad de ```qmail``` es excelente, tal servidor de
correo no es de fuentes abiertas y su módelo de desarrollo no es óptimo para
administradores pues la versión oficial (garantizada por su autor) en
bastantes ocasiones debe ser ampliada con parches no oficiales de otros
desarrolladores, parches que no cuentan con la garantía de ```qmail``` y no son
incorporados de manera oficial.

Por otra parte la configuración de sendmail es bien compleja, su licencia es
GPL y su registro de seguridad no es el mejor. Sin embargo es un servidor muy
configurable y la versión incorporada en OpenBSD ha sido auditada y parchada,
por lo que no ha presentado fallas de seguridad desde Septiembre de 2003 (la
última conocida fue para OpenBSD 3.3).

Vale la pena evaluar otros servidores con excelente registro de seguridad como
Postfix (IBM Public License -- puede encontrarse una discusión de las razones
por las que no se incorporó en OpenBSD base buscando Postfix license OpenBSD).


##BIBLIOGRAFÍA

http://dantams.sdf-eu.org/guides/obsd_courier_imap.html http://es.tldp.org/Manuales-LuCAS/doc-tutorial-postfix-ldap-courier-spamassassin-amavis-squirrelmail

!PROCMAIL

http://pm-doc.sourceforge.net/pm-tips.html http://structio.sourceforge.net/guias/basico_OpenBSD/correo.html#procmail

!IMAP

http://www.linux-sec.net/Mail/SecurePop3/ http://talk.trekweb.com/~jasonb/articles/exim_maildir_imap.shtml

Información dedicada a Dios y liberada al dominio público. 2005. vtamara?pasosdeJesus
