---
title: squid_y_Dansguardian
date: 2011-05-05
tags:
---
Dansguardian con squid permiten filtar Internet de contenidos no apropiados.


##1. Instalación

Primero instale el paquete ```squid```, configurelo para que al menos acepte conexiones del mismo computador en ```/etc/squid/squid.conf```:
<pre>
  acl localnet src 127.0.0.1/32
</pre>

Asegure que podrá abrir suficientes archivos y tendrá memoria agregando a ```/etc/login.conf``` 
```
squid:\
        :datasize=1500M:\
        :openfiles=4096:\
        :tc=daemon:
```
y después ejecutando
```
cd /etc
sudo cap_mkdb /etc/login.conf

```

Inicielo con 
<pre>
/etc/rc.d/squid start
</pre>

Podrá verificar que está ejecutandose con ```pgrep squid``` o 
con ```/etc/rc.d/squid check``` la bitácora queda en 
```/var/squid/logs/access.log```

Pruebe que opera configurando como proxy http en el navegador del computador 
donde instaló por el puerto 3128.    Por ejemplo iniciando chrome con:
```
chrome --proxy-server=http://127.0.0.1:3128
```

Puede asegurar arranque en cada inicio con:
```
sudo rcctl enable squid
```

Instale a continuación el paquete ```dansguardian```.  
No  hay mucho que configurar en ```/etc/dansguardian/dansguardian.conf```
excepto el idioma:
<pre>
language = 'spanish'
</pre>

Pruebe que opera configurando como proxy http en el navegador del computador 
donde instaló por el puerto 8080.    Por ejemplo iniciando chrome con:
```
chrome --proxy-server=http://127.0.0.1:8080
```
que debe bloquear sitios de contenido inapropiado.

Asegure que arranca en cada inicio con:
```
sudo rcctl enable squid
```

##2. Configuración como proxy transparente

Si ejecuta squid y dansguardian en el mismo computador que es cortafuegos
basta que agregue a /etc/pf.conf:

pass in quick log inet proto tcp to port 80 divert-to 127.0.0.1 port 8080


##3. Uso 

```/etc/dansguardian/lists/banned.....```
Debe ser modificando  ```/etc/dansguardian/lists/bannedregexpurllist```
o
```/etc/dansguardian/lists/bannedregexpheaderlist```
Puede verse una corta explicación de los archivos en 
http://ciclo.tuinstituto.es/?q=node/412



##4. Referencias

* Documentación del paquete squid para OpennBSD (ubicada en /usr/local/share/doc/pkg-readmes/squid-3.5.1 )
