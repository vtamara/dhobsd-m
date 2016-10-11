---
title: Autenticacion_con_OpenLDAP
date: 2005-12-06
tags:
---
En este artículo describimos como autenticar usuarios en OpenBSD 3.8 empleando ```login_ldap``` y un directorio manejado por OpenLDAP.

## INTRODUCCIÓN

LDAP (Lightweight Directory Access Protocol) es un protocolo para mantener e intercambiar información almacenada en directorios (i.e bases de datos especiales).
 Un uso típico de LDAP es mantener en un servidor información de los usuarios de una organización para permitir su autenticación en los computadores (e.g nombres, apellidos, dirección, teléfono, login, clave).

OpenLDAP es una implementación de LDAPv3 de fuentes abiertas que cuenta con paquetes oficiales para OpenBSD 3.8

El paquete ```login_ldap``` agrega a los esquemas de autenticación de OpenBSD uno que consulta en un servidor LDAP. En particular puede consultar un servidor LDAP que corre en el mismo computador donde se hace la autenticación, lo cual facilita los pasos requeridos para mantener seguridad y por esto será el esquema aquí descrito.

## INSTALACIÓN Y CONFIGURACIÓN

La instalación de todos los paquetes es directa:
<pre>
# pkg_add $PKG_PATH/openldap-client-2.2.27p0.tgz
# pkg_add $PKG_PATH/openldap-server-2.2.27p0.tgz
# pkg_add $PKG_PATH/login_ldap-3.3.tgz
</pre>

También se requiere el esquema ```authldap.schema``` que puede descargar de Internet (http://cvs.sourceforge.net/viewcvs.py/*checkout*/courier/libs/authlib/authldap.schema) o que está incluido en el paquete ```courier-authlib-ldap``` (disponible desde
OpenBSD 4.1).

### OpenLDAP

Para configurar el servidor, si no existen comience por crear un usuario (```_openldap```) y un grupo (```_openldap```):
<pre>
# groupadd -g 544 _openldap
# useradd -s /sbin/nologin -c "Servidor LDAP" -g _openldap -m \
  -d /var/openldap-data/ -u 544 _openldap
</pre>

Copie los esquemas y el archivo de configuración del servidor:
<pre>
# mkdir /etc/openldap
# cp /usr/local/share/examples/openldap/slapd.conf /etc/openldap/
# cp -rf /usr/local/share/examples/openldap/schema /etc/openldap/
</pre>

Asegurse de copiar también authldap.schema, por ejemplo si instala ```courier-authlib-ldap``` con:
<pre>
# cp /usr/local/share/examples/courier-authlib/authldap.schema /etc/openldap/schema
</pre>

Edite ```/etc/openldap/slapd.conf``` agregando las líneas siguientes:
<pre>
include               /etc/openldap/schema/core.schema
include               /etc/openldap/schema/cosine.schema
include               /etc/openldap/schema/nis.schema
include               /etc/openldap/schema/inetorgperson.schema
include               /etc/openldap/schema/openldap.schema
include               /etc/openldap/schema/authldap.schema

allow bind_v2
</pre>

y modifique las líneas ```database```, ```suffix```, ```rootdn``` y ```rootpw``` para que se ajusten a su organización:
<pre>
database      ldbm
suffix        "dc=correo,dc=pasosdeJesus,dc=org"
rootdn        "cn=Manager,dc=correo,dc=pasosdeJesus,dc=org"
rootpw secret
</pre>

Note que: 
* se permite LDAPv2 porque es el empleado por ```login_ldap```
* se emplea Berkely DB para mantener la información del directorio (como se explica en ```man slapd.conf``` las posibilidades son ```bdb, dnssrv, ldap, ldbm, meta, monitor, null, passwd, perl, shell, sql```, o ```tclse```) y que la clave del directorio debe ser mejor que la presentada (i.e remplace ```secret``` por una buena clave). En lugar de poner la clave plana también es posible poner la cadena generada con:
<pre>
# slappasswd -v -u -h {CRYPT} -s secret
</pre>
que en el caso de la clave '```secret```' es '```{CRYPT}uPUCy906TIu/k```'

La configuración por defecto emplea ```/var/openldap-data``` como directorio para mantener las bases de datos, así que ejecute:
<pre>
# mkdir /var/openldap-data
# chown _openldap:_openldap /var/openldap-data
# chmod 700 /var/openldap-data
</pre>

Cada vez que modifique el archivo de configuración del servidor, puede verificarlo con:
<pre>
# slaptest
</pre>
Para iniciar el servidor LDAP en modo de depuración para ver posibles errores:
<pre>
# /usr/local/libexec/slapd -d 1
</pre>
Tras verificar el funcionamiento, para que en cada arranque se inicie el servidor puede agregar a ```/etc/rc.local```:
<pre>
#LDAP
if ![ -x /usr/local/libexec/slapd ]; then
        echo -n " slapd"; /usr/local/libexec/slapd
fi
</pre>
Cuando lo requiera podrá detener el servidor con:
<pre>
# pkill slapd
</pre>
Puede verificar que el servidor está corriendo con:
<pre>
ldapsearch -x -b 'dc=correo,dc=pasosdeJesus,dc=org' '(objectclass=*)'
</pre>

### login_ldap

La configuración de este paquete es sencilla pues sólo debe ejecutar 
```enable-login-ldap``` y agregar algunas líneas a ```/etc/login.conf``` pero sin agregar espacios en blanco al final de cada una (por este motivo es mejor no emplear copiar & pegar sino incluir en ```/etc/login.conf``` el contenido de ```/usr/local/share/login_ldap/login_ldap.conf``` y modificarlo
 ---en ```vi``` emplee ```:r /usr/local/share/login_ldap/login_ldap.conf``` ):
<pre>
ldap:\
        :auth=-ldap:\
        :x-ldap-server=localhost:\
        :x-ldap-port=389:\
        :x-ldap-noreferrals:\
        :x-ldap-uscope=subtree:\
        :x-ldap-basedn=ou=gente,dc=correo,dc=pasosdeJesus,dc=org:\
        :x-ldap-binddn=cn=Manager,dc=correo,dc=pasosdeJesus,dc=org:\
        :x-ldap-bindpw=secret:\
        :x-ldap-filter=(&(objectclass=posixAccount)(uid=%u)):\
</pre>

Si ```login.conf``` se utiliza con ```cap_mkdb```, parece que esta clase de login debe ponerse en orden alfabético. 

##ADICIÓN DE DATOS Y USUARIOS

Una vez esté corriendo ```slapd``` deberá iniciar un directorio para su organización y los usuarios que se autenticarán. Puede agregar estos datos con el programa ```ldapadd```. Programa que recibe datos en formato ldif, por ejemplo leidos de un archivo. Un primer archivo con datos de la organización puede ser ```org.ldif``` y contener:
<pre>
dn:     dc=correo,dc=pasosdeJesus,dc=org
objectClass:    dcObject
objectClass:    organization
o:      Pasos de Jesús
dc:     correo

dn: cn=Manager,dc=correo,dc=pasosdeJesus,dc=org
objectClass: organizationalRole
cn: Manager

dn:ou=gente, dc=correo,dc=pasosdeJesus,dc=org
objectClass:    top
objectClass:    organizationalUnit
ou:     gente

dn:ou=grupos,dc=correo,dc=pasosdeJesus,dc=org
objectClass:    top
objectClass:    organizationalUnit
ou:     grupos

dn:ou=sendmail,dc=correo,dc=pasosdeJesus,dc=org
ou: sendmail
objectClass: top
objectClass: organizationalUnit
userPassword: sendmail
</pre>

Nota: Al agregar información verifique no dejar espacios en blanco al final de cada línea.

Se pueden agregar ```org.ldif``` con:
<pre>
ldapadd -x -D "cn=Manager,dc=correo,dc=pasosdeJesus,dc=org" -W \
  -h correo.pasosdeJesus.org -f org.ldif
</pre>

Note que el archivo ldif recién presentados y los que siguen emplean "campos" de los esquemas ```/etc/openldap/{cosine.schema,nis.schema,authldap.schema}```

Además de poder revisar los mensajes que ```slapd``` genere al ejecutarse en modo de depuración, podrá consultar los datos ingresados al directorio con:
<pre>
ldapsearch -x -b 'dc=correo,dc=pasosdeJesus,dc=org' '(objectclass=*)'
</pre>

Con el actual esquema de autenticación de OpenBSD cada usuario que pueda ingresar al sistema debe tener una entrada en el archivo ```/etc/passwd```, por esto para agregar un usuario que se autenticará via LDAP primero debe ejecutar algo como:
<pre>
# useradd -m -d /home/pablo -s /bin/sh -L ldap pablo
</pre>
que agregará un usuario ```pablo``` con grupo ```users```, clase de login ```ldap``` e inicialmente sin clave (la clave y otros datos se agregan al directorio LDAP). Revise el uid y el gid de este usuario; por ejemplo con ```vipw``` o con ```id pablo```

Tras tener el ```uid``` y ```gid``` del nuevo usuario podrá crear un registro ldif :
<pre>
dn:uid=pablo,ou=gente,dc=correo,dc=pasosdeJesus,dc=org
uid: pablo
cn: Pablo
sn: Ramirez
userPassword: !MiClave
loginShell: /bin/ksh
uidNumber: 1003
gidNumber: 10
homeDirectory: /home/pablo/
shadowMin: -1
shadowMax: 999999
shadowWarning: 7
shadowInactive: -1
shadowExpire: -1
shadowFlag: 0
objectClass: top
objectClass: person
objectClass: posixAccount
objectClass: shadowAccount
objectClass: !CourierMailAccount
mail: pablo\@correo.pasosdeJesus.org
mailbox: maildir/
quota: 0
</pre>

Para que posteriormente puede autenticar con Courier-POP tenga en cuenta que la línea mail: pablo@correo.pasosdeJesus.org tenga el nombre completo del servidor de correo que ha empleado al agregar datos al directorio (en lugar de sólo pablo@pasosdeJesus.org).

Al igual que la clave general del servidor LDAP, en lugar de la clave plana del usuario puede poner la retornada por slappasswd.

Si emplea tildes o eñes, tenga en cuenta que el archivo ldif debe tener codificación Unicode (se ha probado UTF-8 con éxito). Esto lo puede lograr bien empleando un editor que soporte UTF-8 o recodificando el archivo. Por ejemplo puede recodificar un texto (```p.ldif```) de ASCII a UTF-8 con:

recode txt..UTF-8 p.ldif

Tras agregar el usuario al directorio (con ```ldapadd``` como se explicó antes) puede intentar busquedas como las que efectuará ```login_ldap```. Primero una busqueda general:
<pre>
ldapsearch -P 2 -H !ldap://localhost:389 -w secret \
  -D "cn=Manager,dc=correo,dc=pasosdeJesus,dc=org" -x \
  -b 'dc=correo,dc=pasosdeJesus,dc=org'
</pre>

y después un usuario particular (el recién ingresado):
<pre>
ldapsearch -P 2 -H !ldap://localhost:389 -w secret \
  -D "cn=Manager,dc=correo,dc=pasosdeJesus,dc=org" \
  -x -b "ou=gente,dc=correo,dc=pasosdejesus,dc=org" \
  "(&(objectClass=posixGroup)(memberUid=pablo))"
</pre>
Si las busquedas le funcionaron también debería funcionar:
<pre>
/usr/libexec/auth/login_-ldap -d -s login pablo ldap; echo $?
</pre>

así como:
<pre>
$ login
login: pablo
...
</pre>

## ELIMINACIÓN DE INFORMACIÓN

Para eliminar un usuario puede emplear
<pre>
ldapdelete -v -P 2 -H !ldap://localhost:389 -w secret \
  -D "cn=Manager,dc=correo,dc=pasosdeJesus,dc=org" \
  -x  'uid=pablo,ou=gente,dc=correo,dc=pasosdeJesus,dc=org'
</pre>

Si necesita borrar el directorio completo utilice:
<pre>
# pkill slapd
# rm /var/openldap-data/*
</pre>

## EFICIENCIA

Para lograr búsquedas veloces deben crearse indices sobre los campos que
se realizan las búsquedas y configurarse caches.   Cada índice se configura en el archivo ```/etc/openldap/slapd.conf``` por ejemplo las líneas:

<pre>
index   objectClass     eq
index   mail            eq
index   uid             eq
cachesize 1000000
dbcachesize 10000000
</pre>

Configuran caches e índices sobre los campos ```objectClass```, ```mail``` y ```uid```.   Los índices creados son apropiados para comprobar igualdad entre
una cadena buscada y un registro.

Después de hacer esta configuración deben regenerarse índices con:

<pre>
# pkill slapd
# slapindex
</pre>

y después reiniciar ```slapd```

## SEGURIDAD

Es recomendable emplear SSL para encriptar las conexiones que consultan a ```slapd```.  Para esto es indispensable tener un certificado para el servidor firmado por una autoridad certificadora (CA), así que tiene tres opciones para obtener tal certificado:
* Usar una autoridad certificadora oficial/comercial
* Usar una autoridad certificadora no-oficial/no-comercial
** http://www.cacert.org --no verifica identidad de quienes solicitan el certificado
** http://www.pasosdeJesus.org --exige conocer a la persona que hace la solicitud
* Crear y usar su propia autoridad certificadora (ver Howto de LDAP con SSL/TLS) (ver sección de referencias)

En todos los casos tendrá que crear un certificado para su servidor y una 
solicitud para que sea firmado por su autoridad certificadora, 
ver http://structio.sourceforge.net/guias/servidor_OpenBSD/apache.html#ssl

Además de su certificado firmado (digamos ```/etc/ssl/server.crt```), la llave del mismo (digamos ```/etc/ssl/private/server.key```), necesita la llave pública de la autoridad certificadora (por ejemplo en ```/etc/ssl/cacert.pem```).

Después agregue en ```/etc/openldap/slapd.conf```

<pre>
!TLSCipherSuite HIGH:MEDIUM:+SSLv2
!TLSCACertificateFile /etc/ssl/cacert.pem
!TLSCertificateFile /etc/ssl/server.crt
!TLSCertificateKeyFile /etc/ssl/private/server.key
!TLSVerifyClient allow
</pre>

Modifique su script de arranque  ```/etc/rc.local``` para que inicie slapd recibiendo conexiones por dos puertos: 389 (ldap) para conexiones planas y 636 (ldaps) para conexiones encriptadas.
<pre>
#LDAP
if ![ -x /usr/local/libexec/slapd ]; then
        echo -n " slapd"; /usr/local/libexec/slapd -h "ldap:/// ldaps:///"
fi
</pre>

Detenga y reinicie ```slapd``` con el nuevo flag y pruebe amas conexiones con:

<pre>
$ telnet localhost 389
</pre>

y 
<pre>
# openssl s_client -connect localhost:636 -showcerts -state \
-CAfile /etc/ssl/cacert.pem
</pre>

este último además de dejarlo en una sesión con el servidor, debe indicar:

<pre>
    Verify return code: 0 (ok)
</pre>

Note que es indispensable la llave pública de la autoridad certificadora para lograr conexiones exitosas con el puerto 636.

Es recomndable que emplee ```pf``` para bloquear los puertos en los que no ofrezca servicios y eventualmente para abrir el puerto 636.

## AUTOMATIZACIÓN

* Configuración general: [ http://dhobsd.pasosdeJesus.org/index.php?binary=internal%3A%2F%2Fd06c5dae76d1241a1893128185c33ddf.bin | DN.sh]
* Agrega datos LDIF  (requiere DN.sh): [http://dhobsd.pasosdeJesus.org/index.php?binary=internal%3A%2F%2F6b69fdd2fbd7610e8b449ec1f30a992b.bin | agldif.sh]
* Busca una identificación  (requiere DN.sh):   [ http://dhobsd.pasosdeJesus.org/?binary=internal%3A%2F%2Fe4df41d582636c83f39444b89275b3dd.bin | busca.sh]
* Agrega un usuario (requiere DN.sh y agldif.sh.  También requiere paquete courier-utils):   [http://dhobsd.pasosdeJesus.org/index.php?binary=internal%3A%2F%2Fab1d6b2d8c853d5b5eaf69bfec95504e.bin  | agusuario.sh ]
* Elimina un usuario (requiere DN.sh):   [http://dhobsd.pasosdeJesus.org/?binary=internal%3A%2F%2Fed8a1bfab903e4612a3080bdcdacc89c.bin | elimusuario.sh ]
* Cambia clave de un usuario (requiere DN.sh): [http://dhobsd.pasosdeJesus.org/index.php?binary=internal%3A%2F%2Feb75a2d65e15b4620c4c7bcb963be2e1.bin | camclave.sh]


## CONCLUSIONES

Aunque el esquema de autenticación de OpenBSD requiera crear una cuenta del sistema por cada usuario LDAP a autenticar, esta configuración resulta apropiada para:

* Mantener de manera uniforme tanta información de cada usuario del sistema como lo requiera una organización (la que puede mantenerse en ```/etc/passwd``` es mínima y no es apropiada para toda organización).
* Facilitar la consulta de información de usuarios.
* Puede hacerse más pública la información que se desee empleando el OpenLDAP LDAP Root Service (ver http://www.openldap.org/faq/data/cache/393.html )
* Permitir autenticación tanto con el mecanismos estándar de OpenBSD como con el servidor LDAP (por ejemplo Courier-POP3D puede autenticar directamente con el servidor LDAP).

## BIBLIOGRAFÍA

* Páginas ```man slapd```, ```slapd.conf```, ```login_ldap```
* http://www.openldap.org/doc/admin22/quickstart.html
* http://www.thenewpush.com/jahia/Jahia/cache/offonce/pid/46#10
* http://www.os3.nl/~hjblok/practical/LIA/ldap.html

Inconveniente con ```/etc/login.conf```:

* http://monkey.org/openbsd/archive/misc/0212/msg00773.html
* http://www.pantek.com/library/general/lists/openbsd.org/misc/msg02844.html

La necesidad de permitir LDAPv2 se vió en un correo, disponible hasta hace poco en el cache de google.

Seguridad y autoridad certificadora:
* http://www.openldap.org/pub/ksoper/OpenLDAP_TLS_howto.html#2.1

Información dedicada al Padre protector y liberada al dominio público. 2005. vtamara@pasosdeJesus.org
