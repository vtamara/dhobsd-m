---
title: Respaldo_de_Windows_a_OpenBSD
date: 2006-08-15
tags:
---
En ocasiones debe sacarse información de respaldo de un sistema Windows a un servidor OpenBSD, esto puede hacerse de forma automática con ayuda de WinScp.   Para automatizar por completo el proceso y mantener algo de seguridad, puede emplearse WinScp junto con pageant de Putty.

!Requerimientos en el servidor OpenBSD

# sshd corriendo y permitiendo conexiones del cliente Windows
# Una cuenta que usará para autenticarse desde Windows y para poner    los archivos, digamos la cuenta "sube"  --mejor en una partición    de tamaño limitado

!Requerimientos en el sistema Windows

# Putty con pageant y puttygen. Se ha probado con la versión 0.58. http://www.chiark.greenend.org.uk/~sgtatham/putty/
# WinSCP. Se ha probado con la versión 3.8.1. http://www.winscp.net/

!Autenticación con Pagent

Primero genere un par de llaves RSA sin clave en el sistema Windows con
puttygen.  Salve la llave privada (en este ejemplo queda en
c:\Archivos de Programa\PUTTY\sube-privada.ppk )
Exporte la llave pública para usarla con OpenSSH y agreguela al archivo .ssh/authorized_keys de su cuenta en el servidor OpenBSD.

Puede probar que la autenticación automática funciona así:
# Ejecute pageant y agregue la llave privada del usuario sube.
# Ejecute putty e ingrese al usuario sube --debe entrar sin pedirle clave.

!Archivo que automatiza la copia

El siguiente archivo  por lotes (digamos que sea c:\sist\remota.bat) se basa en uno escrito por Luis Alberto y realiza las siguientes operaciones:
# inicia pageant con la llave del usuario sube
# saca copia de una base de datos de PostgreSQL 
# Crea un script para WinSCP
# Ejecuta WinSCP

<pre>
echo off
C:\ARCHIV~1\PuTTY\pageant.exe C:\ARCHIV~1\PuTTY\sube-private.ppk
SET LOG_FILENAME_UP=up%DATE:~0,2%%DATE:~3,2%%DATE:~6,4%.backup
C:\PostgreSQL\8.0\bin\pg_dump -f C:\sist\bkup\%LOG_FILENAME_UP% -D -U usuario base
echo option batch on > C:\sist\remota.wscp
echo option confirm off >> C:\sist\remota.wscp
echo open sube:clavesube!@www.pasosdeJesus.org >> C:\sist\remota.wscp
echo cd /home/sube/base/ >> C:\sist\remota.wscp
echo put C:\Caso_up\bkup\%LOG_FILENAME_UP% >> C:\sist\remota.wscp
echo cd /home/sube/descargas/ >> C:\sist\remota.wscp
echo lcd c:\sist\server\default\deploy\jetspeed.ear\jetspeed.war\descargas >> C:\sist\remota.wscp
echo synchronize remote >> C:\sist\remota.wscp
echo close >> C:\sist\remota.wscp
echo exit >> C:\sist\remota.wscp
"c:\Archivos de programa\WinSCP3\WinSCP3.exe" /console /script=c:\sist\remota.wscp
</pre>

El script para WinSCP que se crea (C:\sist\remota.wscp) envia la copia de la base al servidor, y sincroniza un directorio en el servidor con un directorio del cliente.

!Programación de ejecución periódica

Este script que saca copia remota puede ejecutarse diariamente programando
una tarea desde Panel de Control->Tareas Programadas. 


----
Esta información se cede al dominio público y se dedica a Dios. vtamara@pasosdeJesus.org
