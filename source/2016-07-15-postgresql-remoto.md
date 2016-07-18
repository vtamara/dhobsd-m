---
title: Base PostgreSQL remota
date: 2016-07-15
tags:
---

# Base PostgreSQL remota

PostgreSQL permite conexiones remotas y cifradas, así que la aplicación 
puede estar en un servidor y la base de datos en otra.

Para la operación cifrada se requiere un certificado para el servidor y un 
certificado para cada usuario de la base de datos que se emplee en
conexiones remotas.  Los certificados para los clientes deben tener el CN
con el nombre del usuario que hará la conexión.   
Por lo mismo en lugar de comprar certificados para esto es más práctico 
tener una autoridad certificadora que pueda firmarlos.

## 1. Autoridad certificadora SSL

Las operaciones con SSL depende en cliente y en servidor de la librería 
LibreSSL (en otros sistemas OpenSSL). Esta incluye el programa
```openssl``` para hacer varias operaciones, incluyendo operaciones
de una autoridad certificadora. 

Un certificado SSL siempre va con una llave privada (el certificado es la 
llave pública).   

El proceso para crear un certificado es:

1. Crear la llave privada para el certificado (extensión .key)
2. Generar el certificado (llave pública) pero sin firma (extensión .csr)
3. Firmar el certificado con una autoridad certificadora y generar el certificado 
4. Usar el certificado firmado junto con la llave privada para realizar conexiones (el certificado firmado se compartirá, mientras que la llave privada no)

Los archivos intermedios pueden examinarse así:

* Solicitudes: ```openssl req -noout -text -in client.csr```
* Llaves: ```openssl rsa -check -in client.key ```
* Certificados: ```openssl x509 -noout -text -in client.crt```

La autoridad certificadora no es más que un certificado autofirmado que 
se configura y usa consistentemente como autoridad certificadora.

## 2. Configuración de servidor

En el servidor deben quedar certificados del servidor en 
```/var/postgresql/data```:

* root.crt Autoridad certificadora (igual a server.crt)
* root.crl Lista de revocación
* server.crt Certificado del servidor
* server.key Llave privada del servidor

Por cada cliente que se va a conectar debe configurarse en 
```/var/postgresql/data/pg_hba.conf``` el/los usuarios que
se conectarán.  Contrario a lo especificado en la documentación de 
PostgreSQL en casos de SSL en ese archivo sólo nos han funcionado 
líneas de la forma: 

```
hostssl all usuario 192.168.100.11/32 cert clientcert=1
```

Es decir conexión SSL exigiendo certificado al cliente y que la autenticación 
sea por certificado. Lo cual también exige que el certificado del cliente
tenga el CN igual al usuario.

## 3. Configuración de cada cliente

Para cada usuario debe hacerse un certificado que se ubica en 
cada comptuador cliente en ```~/.postgresql/{usuario.crt, usuario.key}```  
donde usuario debe correponder al usuario en la base de datos y 
al CN del certificado.

Desde el servidor puede generar y firmar certificado para cliente (cambie 
```usuario``` por el usuario PostgreSQL dueño de la bae de datos y 
que usara desde los clientes para conectarse):

<pre>
doas su - 
cd /var/postgresql/data
openssl genrsa -des3 -out usuario.key 1024
openssl rsa -in usuario.key -out usuario.key
openssl req -new -key usuario.key -out usuario.csr -subj '/C=CO/ST=Cundinamarca/L=Bogota/O=Pasos de Jesús/CN=usuario'
openssl x509 -req -in usuario.csr -CA root.crt -CAkey server.key -out usuario.crt -CAcreateserial
</pre>

A continuación copie el certificado generado (```usuario.crt```) y la 
llave privada (```usuario.key```) al computador cliente donde se usará:

<pre>
scp usuario.key usuario.crt mius@192.168.100.11:~/.postgresql/
</pre>

En el servidor edite el archivo ```/var/postgresql/data/pg_hba.conf``` 
y asegurese de agregar una línea para el usuario y el computador cliente:

```
hostssl all usuario 192.168.100.11/32 cert clientcert=1
```

Reinicie PostgreSQL.

```
doas sh /etc/rc.d/postgresql -d restart
```

Desde el cliente ejecute:
<pre>
doas chmod 0600 /home/usis/.postgresql/usuario.key
</pre>
y pruebe la conexión asegurando que se usa el certificado 
del usuario respectivo:
<pre>
PGSSLCERT=/home/usis/.postgresql/usuario.crt \
PGSSLKEY=/home/usis/.postgresql/usuario.key  \
psql -h192.168.100.21 -Uusuario usuario
</pre>

Configure la aplicación para que en cada arranque o uso establezca:

<pre>
PGSSLCERT=/home/usis/.postgresql/usuario.crt 
PGSSLKEY=/home/usis/.postgresql/usuario.key
</pre>

### 3.1 Clientes en PHP

Copie las llaves dentro de la jaula chroot, haga que el dueño sea
www:www e incluya en alguna fuente
usada antes de las conexiones a base de datos (por ejemplo intente en
index.php):
<pre>
putenv('PGSSLCERT=/ojs/certs/ojs.crt');
putenv('PGSSLKEY=/ojs/certs/ojs.key');
</pre>

### 3.2 Clientes en Ruby on Rails

En ```config/database.yml``` debe verse algo como:
<pre>
username: usuario
host: 192.168.100.21
sslmode: "require"
</pre>

y al hacer operaciones que usen base de datos (rails dbconsole, iniciar unicorn, etc) asegurese de ejecutarlas en un ambiente donde se definan bien las variables PGSSLCERT y PGSSLKEY, por ejemplo:
<pre>
PGSSLCERT=/home/usis/.postgresql/usuario.crt \
PGSSLKEY=/home/usis/.postgresql/usuario.key \
rails dbconsole
</pre>

## 4. Referencias

* https://www.howtoforge.com/postgresql-ssl-certificates

### 5. Agradecimientos

Siempre a Dios.  En particular por este artículo a CINEP donde se ha podido 
implementar y permiten ceder al dominio público esta información.

