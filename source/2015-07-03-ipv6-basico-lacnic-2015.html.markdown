---
title: ipv6-basico-lacnic-2015
date: 2015-07-03 03:46 UTC
tags:
---

A continuación recopilo los videos referenciados (en el orden que son 
presentados) en el 2do curso de IPv6 introductorio ofrecido por 
[LACNIC Campus](http://campus.lacnic.net/) en 2015.  Después transcribo
algunos ejercicios que este curso me sucito realizar.


# Presentación y Contenidos

- [Presentación y Contenidos](https://youtu.be/kwvINZmmXeM?list=PLVpV7HTGyBXxoOrg4kp-cDe8u2ptl0rjN)

# Módulo 1: Introducción a IPv6


- [Definición del protocolo IPv6 - Parte 1](https://youtu.be/cVyhDp14fac)
- [Definición del protocolo IPv6 - Parte 2](https://youtu.be/2zjHCe0Y66I)
- [¿Por qué utilizar IPv6 hoy?](https://youtu.be/5IvTIzhrvC4)
- [Desarrollo de IPv6 en la región](https://youtu.be/qKJdCFFO3K0)

# Módulo 2: Agotamiento del espacio IPv4, Coexistencia y Transición ►

- [Proyecciones de agotamiento](https://youtu.be/-HIUT2ilCyU)
- [Espacio IPv4 disponible](https://youtu.be/XqHPrtiE2_I)
- [Espacio IPv6 disponible](https://youtu.be/iItLZ8IUorI)
- [Mecanismos de Transición](https://youtu.be/LQQD7-8E4c8)
- [Doble Pila](https://youtu.be/C_ekmx7v9m0)
- [Túneles](https://youtu.be/sE3rPzhiswM)
- [Traducción](https://youtu.be/gTLK6uyLPCc)
- [Recomendaciones generales](https://youtu.be/md80CcXGjY8)

# Módulo 3: Direccionamiento y Planes de numeración►

- [Cabezal básico IPv6](https://youtu.be/qoJkahpB5do)
- [Características generales]( https://youtu.be/qxVUOL9prHk)
- [Distribución y asignación de recursos]( https://youtu.be/sioBvZ7ciQE)
- [Proveedores]( https://youtu.be/IJ7sbiJqxnM)
- [Planes de numeración](https://youtu.be/xKFHp_tWOEM)


# Módulo 4: Autoconfiguración►

- [Protocolos asociados](https://youtu.be/Chzbg-il2z8)
- [Autoconfiguración stateless](https://youtu.be/D86t7ZITU2A)
- [Autoconfiguración stateful](https://youtu.be/Eiy4gHNxjaw)
- [Conclusiones](https://youtu.be/K3w_Gl3VCgI)

# Ejercicios (no incluidos como parte del curso de Lacnic, pero cedidos al dominio público por Vladimir Támara)

## Ejercicio 1: ping6 

Como consulté en www.ipv6tf.org, para verificar en adJ/OpenBSD que tengo habilitado IPv6 (estoy usando adJ 5.7):
```
ping6 ::1
```

Y bueno también ifconfig muestra lo0 (interfaz loopback) con:

```
inet6 fe80::1%lo prefixlen 64 scopeid 0x3
inet6 ::1 prefixlen 128
```

## Ejercicio 2: Prefijo IPv6 para hacer pruebas

Cómo no tengo dirección IPv6 (que bueno si LACNIC me diera una :) ), encontré que según el RFC 4193 ( https://tools.ietf.org/html/rfc4193 ) pueden hacerse pruebas con direcciones locales únicas (ULA - Uniq Local Address) que se generan de manera aleatoria usando una MAC de un equipo propio.

En https://www.sixxs.net/tools/grh/ula/ hay un formulario para facilitar el procedimiento y que además mantiene una base de datos para ayudar a evitar duplicados.

Por el momento estoy usando el prefijo fd4d:da20:9e55/48

## Ejercicio 3:  Dirección IPv6 en un computador

Con mi prefijo de prueba establecí temporalmente una dirección IPv6 en un computador así:

``` 
sudo ifconfig re0 inet6 fd4d:da20:9e55::1
``` 

Como esa interfaz ya tenía una IPv4 ha quedado con doble pila y al examinar con ifconfig re0 se ve:

``` 
inet 192.168.0.99 netmaks 0xffffff00 broadcast 192.168.0.255
inet6 fe80::eeb1:d7ff:febb:d7ff%re0 prefixlen 64 scopeid 0x1
inte6 fd4d:da20:9e55::1 prefixlen 64
``` 

La primera IPv6 corresponde a una dirección de enlace local (link local) que incluye la MAC de la tarjeta de red --y se usaría para autoconfiguración como se vió en el curso.

La segunda IPv6 es la que asigné.

Puedo probar con:

``` 
$ ping6 fd4d:da20:9e55::1

PING6(56=40+8+8 bytes) fd4d:da20:9e55::1 --> fd4d:da20:9e55::1
16 bytes from fd4d:da20:9e55::1, icmp_seq=0 hlim=64 time=0.041 ms
16 bytes from fd4d:da20:9e55::1, icmp_seq=1 hlim=64 time=0.035 ms
^C
--- fd4d:da20:9e55::1 ping6 statistics ---
2 packets transmitted, 2 packets received, 0.0% packet loss
round-trip min/avg/max/std-dev = 0.035/0.038/0.041/0.003 ms
``` 

Para mantener la configuración (doble pila) después de un reinicio se deja en /etc/hostname.re0:

``` 
inet 192.168.0.99 255.255.255.0 NONE
inet6 alias fd4d:da20:9e55::1 64
``` 

Se puede reiniciar la red para que tome estos valores con:

``` 
sudo sh /etc/netstart
``` 

## Ejercicio 4: Una mini-LAN con IPv6

El computador que configuré en el ejercicio anterior está en una LAN con otro que configuré con doble pila con las direcciones:

IPv4: 192.168.0.98
IPv6: fd4d:da20:9e55::2
Una vez configurados ambos es posible desde el primero hacer ping6 al segundo (o viceversa):


``` 
$ ping6 fd4d:da20:9e55::2
PING6(56=40+8+8 bytes) fd4d:da20:9e55::1 --> fd4d:da20:9e55::2
16 bytes from fd4d:da20:9e55::2, icmp_seq=0 hlim=64 time=0.209 ms
16 bytes from fd4d:da20:9e55::2, icmp_seq=1 hlim=64 time=0.189 ms
^C
--- fd4d:da20:9e55::2 ping6 statistics ---
2 packets transmitted, 2 packets received, 0.0% packet loss
round-trip min/avg/max/std-dev = 0.189/0.199/0.209/0.010 ms
``` 

## Ejercicio 5: Servicios sobre IPv6

He notado que algunos servicios ya están habilitados con IPv6, pueden verse en adJ con:

``` 
fstat | grep internet6
``` 

en algunos servidores con adJ que opero he constatado que los siguientes 
servicios ya tienen habilitado IPv6: Xorg, cupsd, nginx, smtpd, sshd, named, 
ftpd, rsync, dovecot

Así mismo diversas aplicaciones clientes los soportan incluyendo telnet, ssh, chrome.

Por ejemplo en la red armada en el ejemplo anterior, desde el segundo computador puedo ingresar al primero con:

``` 
ssh vtamara@fd4d:da20:9e55::1
``` 

O puedo copiar (usando la notación con paréntesis cuadrados explicada en el curso):

``` 
scp -r vtamara@[fd4d:da20:9e55::1]:/home/vtamara/comp/adJ/5.7b1-amd64 .
``` 
Por lo menos Ruby on Rails (para la cual adJ es buena pila) también tiene habilitado IPv6, por ejemplo la aplicación https://github.com/pasosdeJesus/cor1440 puede ejecutarse para escuchar en la interfaz con IPv6 con:

``` 
$ rails s -p4000 -bfd4d:da20:9e55::1

=> Booting WEBrick
=> Rails 4.2.3 application starting in development on http://fd4d:da20:9e55::1:4000
=> Run `rails server -h` for more startup options
=> Ctrl-C to shutdown server
[2015-06-27 09:33:08] INFO WEBrick 1.3.1
[2015-06-27 09:33:08] INFO ruby 2.2.2 (2015-04-13) [x86_64-openbsd]
[2015-06-27 09:33:08] INFO WEBrick::HTTPServer#start: pid=6077 port=4000
``` 
y desde Chromium puede verse en la dirección http://[fd4d:da20:9e55::1]:4000/



![IPv6](http://s6.postimg.org/awuvnuwg1/chromeipv6.jpg)

## Ejercicio 6:  Conectar a Internet IPv6 usando un tunnel broker con HE

En el vídeo sobre túneles del módulo 2, se mencionan páginas web que automatizan un lado de la creación de un túnel 6in4, una es https://tunnelbroker.net que use para este ejercicio.

La configuración la hice desde un servidor con adJ conectado a Internet con una IPv4 fija (digamos 2.2.2.2).

Me registré en tunnelbroker.net y solicité un túnel regular. Como "Punto final IPv4"  emplee la IP fija del servidor y como servidor de túnel emplee el más rápido, que para mi caso fue  Miami, FL, US 209.51.161.58  --medí velocidad haciendo ping a cada servidor posible.

Con estos datos tunnelbroker.net me asignó una IPv6 de cliente 2001:470:4:63c::2, generó una IPv6 de servidor 2001:470:4:63c::1, y me generó instrucciones para configurar OpenBSD / adJ:

``` 
ifconfig gif0 tunnel 2.2.2.2 209.51.161.58

ifconfig gif0 inet6 alias 2001:470:4:63c::2 2001:470:4:63c::1 prefixlen 128
route -n add -inet6 default 2001:470:4:63c::1
``` 

Tras ejecutar, desde el servidor ya pude hacer ping6 a la IPv6 del otro extremo del tunel:

``` 
ping6 2001:470:4:63c::1
``` 
Así como a otros sitios de Internet con dirección IPv6, por ejemplo con una de las IPv6 que aparecen en https://developers.google.com/speed/public-dns/docs/using fue:

``` 
ping6 2001:4860:4860::8888
``` 
También me operó resolución de nombres a direcciones sólo IPv6 como:
``` 
$ ping ipv6.test-ipv6.com
ping: unknown host: ipv6.test-ipv6.com
$ ping6 ipv6.test-ipv6.com
PING6(56=40+8+8 bytes) 2001:470:4:63c::2 --> 2001:470:1:18::119
16 bytes from 2001:470:1:18::119, icmp_seq=0 hlim=60 time=140.694 ms
16 bytes from 2001:470:1:18::119, icmp_seq=1 hlim=60 time=140.940 ms
^C
--- ipv6.test-ipv6.com ping6 statistics ---
2 packets transmitted, 2 packets received, 0.0% packet loss
round-trip min/avg/max/std-dev = 140.694/140.817/140.940/0.123 ms
``` 

La navegación con chrome operó bien por ejemplo con una dirección como http://[2a02:3b8:0:ace5::4]/ y tabién me operó con resolución de nombres con direcciones sólo IPv6 como http://ipv6.test-ipv6.com.

De hecho esa última página me confirmó las buenas posibilidades de la dirección IPv6 proveida por HE, del tunel y de mi servidor DNS para usar IPv6 (un named típico configurado en el mismo servidor adJ).

![Chrome sobre Tunel IPv6](http://s6.postimg.org/5bram5681/chrome_tunel_ipv6.jpg)

## Ejercicio 7:  Conectar sólo a IPv6 usando un proveedor que ofrezca IPv6 con autoconfiguraación

Basta conectar cable de proveedor que ofrezca IPv6 y dejar en 
```/etc/hostname.re0``` (cambiando ```re0``` por su interfaz de red):
<pre>
rtsol
</pre>

En el momento de este escrito (Oct.2015) el problema es conseguir proveedores
que ofrezcan IPv6 en Colombia, el autor ha hecho consulta directa con
diversos proveedores (como Claro y ETB, UNE ha sido elusivo y presta
muy mal servicio tras la compra por parte Tigo).  El único que ya
han desplegado conexiones y tiene planes comerciales es Mercanet, que 
infortunadamente sólo maneja precios para corporaciones muy grandes.

Se hizo una prueba desde Estados Unidos con el inseguro proveedor Comcast 
(ver http://aprendiendo.pasosdejesus.org/?id=Ataque+2013 )  donde se pudo
comprobar la conectividad solo IPv6:

![Prueba de conectividad solo IPv6](https://scontent-mia1-1.xx.fbcdn.net/hphotos-xpt1/v/t1.0-9/12003348_971753412880752_4844462289122407099_n.png?oh=c239c4f9c3e1f32c7e4cac024486eb89&oe=56CBF8B4)

Para poder resolver nombres en este escenario se requiere un servidor DNS 
que pueda responder a peticiones de dominios IPv6, por ejemplo los 
servidores públicos de Google con IP 8.8.8.8 pueden hacerlo, así que
basta configurar en ```/etc/resolv.conf```:
<pre>
order file bind
nameserver 8.8.8.8
</pre>

Diversos servicios muy conocidos en el momento de este escrito ya operan 
enteramente sobre IPv6 como Google, Facebook y Linkedin, (sin embargo
no Twitter). 

El autor no encontró sitios públicos en Colombia que operen con IPv6,
según se comenta en
[http://changux.co/ipv6-en-colombia-un-ano-despues-del-worldipv6launchday/]
la situación ha permanecido así desde 2013 --ese sitio 
opera sobre IPv6 pero no está ubicado en Colombia según 
http://www.tcpiputils.com/ipv6-geolocation-database. 
Según la misma base, paradoijicamente http://www.academiaipv6.org/ tampoco
está ubicado en Colombia.

## Ejercicio 8:  Conectar con pila doble usando un proveedor que ofrezca IPv6 e IPv4

Suponiendo que el mismo modem del proveedor permite configuración de IPv5 vía DHCP, basta dejar en ```/etc/hostname.re0``` (cambiando ```re0``` por su 
interfaz de red):
<pre>
dhcp
rtsol
</pre>

Al examinar con ipv6.test-ipv6.como se verá algo como:
![Prueba de conectividad con pila doble](https://fbcdn-sphotos-g-a.akamaihd.net/hphotos-ak-xpf1/v/t1.0-9/11951760_971750372881056_6940521321396463334_n.png?oh=10aee4de1223f0db5cfe5c1ec6f2648c&oe=56CD3FFB&__gda__=1452646545_585a924c37a9bca6fa17b0244b09194a)


