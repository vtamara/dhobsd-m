---
title: Compilacion_OpenOffice
date: 2011-07-07 12:00 UTC
tags:
---
Mientras se completa traspaso a Apache, siguiendo las instrucciones de:

## 1. Compilación

http://wiki.services.openoffice.org/wiki/Documentation/Building_Guide/Getting_the_source

En UbuntuCE:


<pre>
sudo apt-get install mercurial
mkdir ~/des
cd ~/des
wget "http://hg.services.openoffice.org/bundle/DEV300.hg"
mkdir ooo
cd ooo
hg unbundle ../DEV300.hg
Mas de 30 minutos
hg pull http://hg.services.openoffice.org/DEV300
hg update
</pre>

Después instale paquetes requeridos por ejemplo:
<pre>
sudo apt-get install libarchive-zip-perl libcupsys2-dev libpam-dev openjdk-6-jdk ant 
</pre>
Pueden requerirse otros que verá cuando intente ejecutar:
<pre>
./configure --disable-mozilla --without-junit --disable-odk --disable-gstreamer --disable-librsvg
</pre>

Una vez instale todos los paquetes requeridos y se complete la configuración puede ejecutar:

<pre>
./bootstrap
</pre>
A continuación cada vez que requiera compilar:
<pre>
source LinuxX86Env.Set.sh
cd instsetoo_native 
build --all -P2
</pre>

## 2. Instalación y uso coexistiendo con libreoffice

http://wiki.services.openoffice.org/wiki/Run_OOo_versions_parallel

<pre>
cd ~/des/ooo
mkdir inst
cd inst
for i in ../instsetoo_native/unxlngx6.pro/pool_deb/*deb; do dpkg-deb -x $i .; done
</pre>

Y cada vez que se quiera ejecutar:
<pre>
export PATH=/home/ceas/des/ooo/inst/opt/openoffice.org3/program/
soffice.bin -calc
</pre>
