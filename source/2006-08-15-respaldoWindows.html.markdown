---
title: respaldoWindows
date: 2006-08-15
tags:
---
En ocasiones debe sacarse información de respaldo de sistemas Windows a servidores OpenBSD, esto puede hacerse de forma automática con ayuda de winscp.

El siguiente archivo  por lotes (digamos que sea c:\sist\remota.bat) saca copia de una base de datos de PostgreSQL y ejecuta un script para WinSCP.  El script envia la copia de la base al servidor, y sincroniza un directorio en el servidor con un directorio del cliente:

<pre>
echo off
SET LOG_FILENAME_UP=up%DATE:~0,2%%DATE:~3,2%%DATE:~6,4%.backup
C:\PostgreSQL\8.0\bin\pg_dump -f C:\sist\bkup\%LOG_FILENAME_UP% -D -U usuario base
echo option batch on > C:\sist\remota.wscp
echo option confirm off >> C:\sist\remota.wscp
echo open sube:clavesube@www.pasosdeJesus.org >> C:\sist\remota.wscp
echo cd /home/sube/base/ >> C:\sist\remota.wscp
echo put C:\Caso_up\bkup\%LOG_FILENAME_UP% >> C:\sist\remota.wscp
echo cd /home/sube/descargas/ >> C:\sist\remota.wscp
echo lcd c:\sist\server\default\deploy\jetspeed.ear\jetspeed.war\descargas >> C:\sist\remota.wscp
echo synchronize remote >> C:\sist\remota.wscp
echo close >> C:\sist\remota.wscp
echo exit >> C:\sist\remota.wscp
:"c:\Archivos de programa\WinSCP3\WinSCP3.exe" /console /script=c:\sist\remota.wscp
</pre>

La autenticación puede mejorarse

----
Esta información se cede al dominio público y se dedica a Dios. vtamara@pasosdeJesus.org
