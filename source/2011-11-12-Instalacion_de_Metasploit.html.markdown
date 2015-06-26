---
title: Instalacion_de_Metasploit
date: 2011-11-12
tags:
---
Metasploit Framework es un ambiente para realizar auditorias de seguridad. Puede instalarse en OpenBSD adJ 4.9 (con PostgreSQL preinstalado) para realizar auditorias en las organizaciones donde tenga autorización.

##Instalación

Instale los paquetes necesarios
<pre>
sudo pkg_add ruby-gems ruby-sqlite3 ruby-iconv ruby-postgres ruby-pcap subversion-1.6.15p0
</pre>
Cree enlaces para ruby si aún no lo ha hecho:
<pre>
sudo ln -s /usr/local/bin/ruby18 /usr/local/bin/ruby
sudo ln -s /usr/local/bin/gem18 /usr/local/bin/gem
</pre>


Cree una base de datos de PostgreSQL para mantener datos de Metasploit:
<pre>
sudo su - _postgresql
createuser -Upostgres -h/var/www/tmp msf
psql -h/var/www/tmp -Upostgres template1
  ALTER USER msf WITH PASSWORD 'msf';
createdb -Upostgres -Omsf -h/var/www/tmp msf
</pre>

Descargue las fuentes de Metasploit de http://www.metasploit.com

Una vez descargas actualicelas con frecuencia usando:
<pre>
cd msf3
./msfupdate
</pre>

Para emplear el sniffer incluido debe compilar ```external/pcaprub```, sin embargo para hacerse en OpenBSD antes debe aplicarse el parche disponible en: http://dev.metasploit.com/redmine/issues/5420 por ejemplo desde las fuentes con:
<pre>
ftp "http://dev.metasploit.com/redmine/attachments/1736/parche-pcaprub"
patch < parche-pcaprub
cd external/pcaprub
sudo ruby extconf.rb
sudo make
sudo make install
</pre>

##Uso

Una completa referencia se encuentra en {1}.

Inicia la consola con:
<pre>
./msfconsole
</pre>

Para indicar la base de datos:
<pre>
db_driver postgresql
db_connect msf:msf@localhost/msf
</pre>

!Sniffer
Una vez configurada una base de datos,  para iniciar el sniffer:
<pre>
sudo ./msfconsole
use auxiliary/sniffer/psnuffle 
exploit
</pre>

Para ver el progreso puede ingresar en otra consola:
<pre>
creds
</pre>


!Escaneo de puertos
Una vez configurada una base de datos para hacer un escaneo a las direcciones 10.0.0.1 a 10.0.0.255:
<pre>
db_nmap 10.0.0.0/24
## Bibliografía

* http://www.offensive-security.com/metasploit-unleashed/
