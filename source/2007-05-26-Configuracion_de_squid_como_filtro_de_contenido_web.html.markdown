---
title: Configuracion_de_squid_como_filtro_de_contenido_web
date: 2007-05-26
tags:
---
[Aprendieno de Jesus 3.9 | http://aprendiendo.pasosdeJesus.org] (adJ 3.9) incluye paquetes de ```squid``` en modo transparente y ```squidguard``` con una configuración sencilla,  que pueden emplearse para filtrar contenidos no deseados del web.  Esto es particularmente útil en colegios donde deben filtrarse por ejemplo páginas pornográficas como medida de responsabilidad frente a niñ@s del colegio.

!INSTALACIÓN

Descargue e instale los paquetes ```squid``` transparente y ```squidguard```.


!CONFIGURACIÓN

Implemente por lo menos las siguientes modificaciones al archivo ```/etc/squid/squid.conf``` en las secciones respectivas:

<pre>
http_port 127.0.0.1:3128

redirect_program /usr/local/bin/squidGuard -d

#http_access deny all
http_access allow all

visible_hostname servidor.pasosdeJesus.org

httpd_accel_host virtual
httpd_accel_port 80

httpd_accel_with_proxy on 

httpd_accel_uses_host_header on
</pre>

Verifique el archivo de configuración e inicialice el cache de ```squid``` con 
<pre>
squid -z
</pre>

Agregue a ```/etc/rc.local```:
<pre>
pgrep squid > /dev/null
if (test "$?" != "0" -a -x /usr/local/sbin/squid) then {
        echo -n ' squid';       /usr/local/sbin/squid
} fi;
</pre>

Puede iniciar con:
<pre>
/usr/local/sbin/squid
</pre>
y verificar que está funcionando con
<pre>
pgrep squid
</pre>
que debería presentar al menos 6 procesos.

Las bitácoras se pueden revisar en ```/var/squid/logs```

Si necesita que el proceso en ejecución vuelva a leer archivos de configuración:
<pre>
sudo squid -k reconfigure
</pre>

Por otra parte la configuración por defecto de ```squidguard``` requiere que en el directorio principal de su servidor web agregue un archivo ```blocked.php``` con el mensaje que debe aparecer cuando se bloquea un sitio.  En una configuraicón por defecto de Apache sería editar el archivo ```/var/www/htdocs/blocked.php``` y dejar un contenido como:

<pre>
<?php
echo "Sitio bloqueado".$_GET!['url']."<br>\n";
?>
</pre>


!PRUEBAS INICIALES

1. Que pueda verse el mensaje de sitios bloqueados con:
<pre>
lynx localhost/blocked.php?url=x
</pre>

2. Que ```squid``` responda en el puerto 3128
<pre>
telnet localhost 3128
</pre>

Ante algún comando como ```get index.html``` debe responder (aunque con un mensaje de error).

3. Que pueda usarse como proxy local por ejemplo de ```lynx```:
<pre>
export http_proxy=!http://127.0.0.1:3128
lynx !http://www.pasosdeJesus.org
</pre>
Debería permitirle ver el sitio web mientras que si intenta con ```!http://www.xxx.com``` debería bloquearlo pues es uno de los sitios bloqueados en la configuración por defecto del paquete 
```squidguard``` mencionado.

4. Agregue un dominio nuevo al archivo ```/usr/local/squidGuard/db/notgood/domains```
por ejemplo una nueva línea como:
<pre>
hotmail.com
</pre>
Vuelva a cargar la configuración de squid (```squid -k reconfigure```) y verifique que el proxy bloquee toda dirección en el sitio.

5. Agregue un URL nuevo al archivo 
```/usr/local/squidGuard/db/notgood/urls```
y compruebe que tras cargar nuevamente la configuración de ```squid``` también sea bloqueado.

6. En los computadores de la red interna podría configurar navegadores para que empleen el computador donde está squid como proxy en el puerto 3128.

! CONFIGURACIÓN COMO PROXY TRANSPARENTE

Una vez funciona como proxy no-transparente, puede activarlo como transparente (i.e que no se necesiten configurar computadores de la red interna para usarlo como proxy) modificando ```/etc/pf.conf``` para agregar
en la sección de reglas ```rdr```:
<pre>
rdr on $int_if inet proto tcp from any to any port 80 -> 127.0.0.1 port 3128
</pre>

y en la sección de reglas:
<pre>
pass in on $ext_if proto tcp from 192.168.1.0/24 to ($ext_if) port 80 keep state
pass out on $ext_if inet proto tcp from any to any port www keep state
</pre>
remplazando 192.168.1.0/24 por la especificación de su red interna.


----

vtamara@pasosdeJesus.org Esta información se cede al dominio público y se dedica a nuestro dulce Creador.  
