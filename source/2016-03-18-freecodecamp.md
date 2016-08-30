---
title: freecodecamp
date: 2016-03-18
tags:
---

Para usar freecodecamp sobre adJ 5.8

## Prerrequesitos

* gcc-4.9 y  g++-4.9 que se instalan por omisión en adJ 5.8

## Procedimiento

Actualizar npm

<pre>
doas npm install npm -g
</pre>

Asegurar que npm usa paquetes un poco más recientes de los compiladores gcc y g++:

<pre>
env CC=/usr/local/bin/egcc CXX=/usr/local/bin/eg++ npm install
doas npm install -g gulp
doas npm install -g bower
</pre>

## Ejecución

Esta aplicación requiere abrir más de 6000 descriptores de archivos, por lo 
que se recomienda:

* Aumentar límites del kernel. En sysctl.conf:
<pre>
kern.shminfo.shmmni=1024 
kern.seminfo.semmns=2048 
kern.shminfo.shmmax=50331648 
kern.shminfo.shmall=51200 
kern.maxfiles=20000 
</pre>

* Aumentar límite de la clase del usuario que lance aplicación. 
  Por ejemplo si es de la clase ```staff```:
<pre>
staff:\
        :datasize-cur=1536M:\
        :datasize-max=infinity:\
        :maxproc-max=512:\
        :maxproc-cur=256:\
        :openfiles-max=8090:\
        :openfiles-cur=8090:\
        :ignorenologin:\
        :requirehome@:\
        :tc=default:
</pre>

