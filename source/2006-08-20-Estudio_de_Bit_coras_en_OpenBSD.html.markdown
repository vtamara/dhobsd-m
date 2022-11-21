---
title: Estudio_de_Bit_coras_en_OpenBSD
date: 2006-08-20 12:00 UTC
tags:
---

Continuamos el trabajo con los datos de Corporación Compromiso, ingresamos a la terminal: 
<pre>
ssh -L10443:base443 sivel@compromisoong.org
</pre>
Pasamos al navegador y digitamos la dirección
<pre>
https://localhost:10443/
</pre>

* Regresamos a la terminal para cambiar de directorio: `cd /var/log`
* Si se nos olvida en qué directorio nos encontramos digitamos `pwd`
* Otro comando aprendido es `hostname`, el cual nos muestra en qué máquina estamos  trabajando (en caso de olvido), en este caso la respuesta sería: `super.compromisoong.org`

Otros comandos:

* `ls` es para listar
* `less` para mirar un archivo, `less authlog` nos muestra el registro de ingresos a nuestro sistema.  El comando shift + g nos lleva al final


Si deseamos averiguar quien  ha intentado ingresar a nuestro sistema `less authlog` nos muestra el IP del usuario así: 
<pre>
invalid user test from 211.157.113.89
</pre>
para una mayor claridad y ubicación de la persona que intentó ingresar a nuestro sistema digitamos el comando 
<pre>
geoiplookup 211.157.113.89
</pre>
y enter, su resultado fue China.  Si queremos averiguar el nombre que corresponde a esa IP utilizamos el comando 
<pre>
dig -x 211.157.113.89
</pre>
 como resultado nos mostró `mail.chinacomm.com.cn`.   
Buscamos esa dirección en un navegador, averiguamos su contacto y 
mediante el progama mail y el editor vi pudimos redactarle y enviarle un 
mensaje electrónico.  Como ejemplo le enviamos el mensaje a 
`webmaster@chinacomm.com.cn` y en el tema (subject) escribimos: 
`Attempt to enter compromisoong.org from your IP on 16.Aug.2006 at 7:10:41.`  
Otra forma de emplear un correo electrónico es con `mutt` y con el editor `vi` 
en la terminal.  Algunos de los comandos de `vi` son:

* Para entrar a modo inserción tecleamos la letra `i`.
* Esc + dd borra una línea, 
* Esc + x borra un caracter
* Esc + wq! guardar y salir
* Salir q, Y envía el mensaje.

Cuando leemos root en el sistema se refiere al administrador.

# Continuamos con BITACORAS Y SUS COMANDOS:

* `sudo` ejecuta un comando como usuario root (una vez configurado) útil 
  por ejemplo para ver un archivo que tiene permiso de lectura solo para el 
  propietario.
* `sudo less secure`  muestra todos los comandos que se han ejecutado como 
  usuario root.
* `sudo less daemon`  muestra los servicios que están funcionando en el 
  computador.  Por ejemplo el `ntpd` es el servidor de tiempo que sincroniza 
  el reloj del servidor con el de otros servidores a nivel mundial.

Para revisar  bitácoras antiguas digitamos `cd` y oprimimos enter, lo cual nos 
manda al directorio personal (`/home/sivel`).

* Crear un directorio: `mkdir` seguido del nombre que le quieras dar.
* `cp` es un comando para copiar, por ejemplo para copiar toda la información 
  que contiene `/var/log` en mi directorio digitamos: 
<pre>
sudo cp /var/log/* .
</pre>

Encontramos varios archivos comprimidos que no pudimos leer (terminan en `gz`) 
para descomprimir utilizamos el `gzip`.  Por ejemplo 
<pre>
gzip -d authlog.0.gz
</pre>
Como resultado el nombre del archivo cambia pasó a: `authlog.0`

* Otro comando es el `grep`, es un buscador: ejemplo 
<pre>
grep "211.157.113.89" authlog.0
</pre>
* `wc` para conteo, el cual trabaja con `-c` para conteo de bytes o 
  caracteres; `-w`, para conteo de palabras en el archivo, `-l` cuenta el 
  número de líneas.
* `man` para leer los manuales
* comando para descomprimir todo: `sudo gzip -d *.gz`
* `grep "211.157.113.89" authlog* | wc-l` su resultado es el número de veces 
  que el texto 211.157.113.89 aparece en todos los archivos que comienzan 
  con `authlog`
* Buscar una cadena en todos los archivos del directorio actual: 
  `grep "211.157.113.89" *`
* Presentar los archivos en los que aparece una cadena: 
  `sudo grep "tamara" * -l`
* `w` muestra qué usuarios están en línea
* `whoami` muestra en qué usuario estoy
* crear un usuario `sudo adduser nombre`

