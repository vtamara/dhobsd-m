---
title: Atenticar_con_LDAP_desde_Ubuntu
date: 2013-09-26 12:00 UTC
tags:
---
Tenemos un servidor **LDAP** corriendo en OpenBSD. Queremos autenticar ''clientes'' que tienen  **Ubuntu/PEAR OS/Xubuntu/Linux**.

## 1. Servidor

En su red interna debe configurar un servidor LDAPS en el puerto 663.  Si desea emplear ```ldapd``` que es el servidor LDAP nativo de OpenBSD vea  [ldapd].

## 2. Cliente Ubuntu

Descargué la versión más reciente del archivo de comandos disponible en: 
https://gist.github.com/vtamara/6890392.

Ejecutelo con:
<pre>
chmod +x prepclienteldap.sh
sudo ./prepclienteldap.sh dc=pasosdeJesus,dc=org 192.168.2.1
</pre>
remplazando ```dc=pasosdeJesus,dc=org``` por el DN de su servidor LDAP  y ```192.168.2.1``` por la IP donde corre su servicio LDAPS en el puerto 636.

Este archivos de comandos instalará los paquetes necesarios, que a su vez harán algunas preguntas de configuración como se muestra a continuación (las puede volver a contestar con ```sudo dpkg-reconfigure ldap-auth-config```):

[http://dhobsd.pasosdejesus.org/sites/DHOBSD.PASOSDEJESUS.ORG/spages/uldap1.png]

[http://dhobsd.pasosdejesus.org/sites/DHOBSD.PASOSDEJESUS.ORG/spages/uldap2.png]

[http://dhobsd.pasosdejesus.org/sites/DHOBSD.PASOSDEJESUS.ORG/spages/uldap3.png]

[http://dhobsd.pasosdejesus.org/sites/DHOBSD.PASOSDEJESUS.ORG/spages/uldap4.png]

[http://dhobsd.pasosdejesus.org/sites/DHOBSD.PASOSDEJESUS.ORG/spages/no-local-root.png]

[http://dhobsd.pasosdejesus.org/sites/DHOBSD.PASOSDEJESUS.ORG/spages/no-login.png]

[http://dhobsd.pasosdejesus.org/sites/DHOBSD.PASOSDEJESUS.ORG/spages/uldap5.png]

[http://dhobsd.pasosdejesus.org/sites/DHOBSD.PASOSDEJESUS.ORG/spages/uldap6.png]


El archivo de comandos también configurará el sistema de autenticación para que cree carpetas cada vez que ingresa por primera un usuario de LDAP y para agregarlo a grupos típicos de un usuario en Ubuntu.   También ejecutará ```pam-auth-update``` que presenta un diálogo como el siguiente; marque 'activate mkhomedir' y 'activate /etc/security/group.conf':

 
[http://dhobsd.pasosdejesus.org/sites/DHOBSD.PASOSDEJESUS.ORG/spages/uldap7.png]


A continuación el archivo de comandos configurará ```/etc/ldap/ldap.conf``` que es usado por ```ldapsearch``` y otras utilidades LDAP.

Las instrucciones finales de ese archivo son importantes:

# Ejecute ```ldapsearch -x``` para confirmar conexion a servidor LDAP
# Ejecute ```getent group``` y verifique que al final se listan los grupos del directorio LDAP
# Ejecute ```su - usuarioldap``` para verificar que logra ingresar, que crea el directorio /home/users/usuarioldap y que los grupos listados con 'groups' son los típicos de un usuario en Ubuntu


## 3. Referencias

* {1} https://help.ubuntu.com/community/LDAPClientAuthentication
