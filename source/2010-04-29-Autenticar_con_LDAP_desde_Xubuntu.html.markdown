---
title: Autenticar_con_LDAP_desde_Xubuntu
date: 2010-04-29
tags:
---
Tenemos un servidor **LDAP** corriendo en **OpenBSD**. Queremos **autenticar** ''clientes'' que tienen  **Xubuntu/Linux CE**.

## En el servidor OpenBSD

En OpenBSD instale y configure openldap-server (ver por ejemplo [Autenticacion con OpenLDAP] aunque no se requiere login_ldap).  

Para manejar cuentas fuera de los scripts del artículo anterior se sugiere instalar phpldapadmin en el servidor OpenBSD.

## En el cliente Xubuntu

Los paquetes ```auth-client-config``` y ```libnss-ldap``` nos ayudarán a esta labor de configurar la autenticación de un cliente Xubuntu usando LDAP. 

En una ''terminal'' digitamos:

```sudo aptitude install libnss-ldap```

Durante la instalación de este paquete tenemos que responder detalles acerca de nuestro servidor LDAP. Esta configuraión quedará en ```/etc/ldap.conf```.

Una vez configurado el ```libnss-ldap``` habilitamos el perfil de LDAP ejecutando en una terminal:


```sudo auth-client-config -t nss -p lac_ldap```

* ```-t``` : para modificar solamente ```/etc/nsswitch.conf```
* ```-p``` : nombre del fichero para habilitar
* ```lac_ldap``` : perfil de  **auth-client-config** que es parte del paquete **ldap-uth-config**.

Ahora con la utilidad **pam-auth-update** configuramos el sistema para usar LDAP en la autenticación. Para esto ejecutamos:

```sudo pam-auth-update```

Al ejecutar ```pam-auth-update``` nos muestra un menú en el cual elegiremos LDAP como mecanismo de autenticación.

!Pruebas

Tras la configuración mencionada debe poder autenticarse como cualquiera de los usuarios del servidor LDAP.  La identificación del usuario y del grupo será la incluida en el servidor LDAP.

Si tiene problemas ingresando en modo gráfico intente primero ingresar en una consola texto y despues examine errores por ejemplo en /

!Manejo de usuarios y grupos

El paquete **ldapscripts** contiene scripts que nos ayudan en el manejo de usuarios y grupos en LDAP.

Para instalar el paquete, ejecutamos en una terminal:

```sudo aptitude install ldapscripts```

Una vez instalado el paquete, editamos el fichero **ldapscripts.conf** que se encuentra en ```/etc/ldapscripts/ldapscripts.conf```, haciendo los cambios correspondientes en los siguientes entornos:


```SERVER="ldap://localhost"``` : Aquí en lugar de localhost colocamos la IP del servidor LDAP.
```BINDDN="cn=admin,dc=example,dc=com"``` : Datos de nuestro servidor LDAP. Nos quedó así:

```BINDDN="cn=Manager,dc=servidor,dc=saulodetarso,dc=edu,dc=co"```

```BINDPWDFILE="/etc/ldapscripts/ldapscripts.passwd"``` : Queda tal cual

```SUFFIX="dc=example,dc=com"```  Nos solicita datos del sevidor LDAP. Para nuestro caso nos quedó:

```SUFFIX="dc=servidor,dc=saulodetarso,dc=edu,dc=co"``` 

Datos de grupos y usuarios como lo tengamos en nuestro servidor:

```GSUFFIX="ou=Groups"``` 
   
```USUFFIX="ou=Users"```   

Nos quedó:
```GSUFFIX="ou=grupos"```   
 
```USUFFIX="ou=gente"``` 

```MSUFFIX="ou=Machines"``` : Queda igual

Editado el archivo ```ldapscripts.conf```, ahora nos queda autenticar el acceso al directorio. Para esto, en una terminal,  ejecutamos, teniendo en cuenta que en lugar de 'secret' colocamos la clave del administrador del servidor LDAP.

``` sudo sh -c 'echo -n 'secret' > /etc/ldapscripts/ldapscripts.passwd```

``` sudo chmod 400 /etc/ldapscripts/ldapscripts.passwd```


!Referencias

https://help.ubuntu.com/9.10/serverguide/C/openldap-server.html    





