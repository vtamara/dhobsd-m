---
title: Utilizaci_n_de_IPsec
date: 2010-12-22 12:00 UTC
tags:
---
IPsec es un protocolo seguro que hace parte de IPv6. Ha sido portado a IPv4 y a diferencia de otros sistemas operativos, es fácil de usar y configurar en OpenBSD. La descripción presentada a continuación se ha probado en OpenBSD 4.2 y se basa en {1}.

##CONEXIÓN DE 2 LANs QUE EMPLEAN ENRUTADORES OPENBSD

Suponga que la primera LAN tiene direcciones de la forma 192.168.181.x y un enrutador con dirección pública 211.245.63.134.

La segunda LAN tiene direcciones privadas de la forma 192.168.178.x y un enrutador con dirección pública 210.93.171.42.

!Configuración de IPsec

En el enrutador de la primera LAN agregue al archivo ```/etc/ipsec.conf```:

<pre>
ike esp from 192.168.181.0/24 to 192.168.178.0/24 peer 210.93.171.42
ike esp from 192.168.181.0/24 to 210.93.171.42 peer 210.93.171.42
ike esp from 211.245.63.134 to 192.168.178.0/24 peer 210.93.171.42
ike esp from 211.245.63.134 to 210.93.171.42 peer 210.93.171.42
</pre>

En la segunda en el mismo archivo agregue:

<pre>
ike esp from 192.168.178.0/24 to 192.168.181.0/24 peer 211.245.63.134
ike esp from 192.168.178.0/24 to 211.245.63.134 peer 211.245.63.134
ike esp from 210.93.171.42 to 192.168.181.0/24 peer 211.245.63.134
ike esp from 210.93.171.42 to 211.245.63.134
</pre>

!Configuración de cortafuegos

En el enrutador de la primera red en el archivo ```/etc/pf.conf``` agregue:

<pre>
set skip on { lo enc0 }
...
pass quick on $ext_if from 210.93.171.42 port {500, 4500}
</pre>

En el enrutador de la segunda LAN en el mismo archivo agregue:
<pre>
set skip on { lo enc0 }
...
pass quick on $ext_if from 211.245.63.134 port {500, 4500}
</pre>

Recuerde reiniciar PF en ambos enrutadores con ```pfctl -f /etc/pf.conf```

Puede verificar que no hay restricciones entre las dos redes empleando un servicio restringido a usuarios externos de una red desde la otra, o por ejemplo verificando los puertos disponibles desde el primer enrutador:
<pre>
nmap 210.93.171.42
</pre>

!Configuración de llaves para ```isakmpd```

Copie la llave pública de isakmpd del primer enrutador en el segundo:

<pre>
sudo scp /etc/isakmpd/private/local.pub usuario!@210.93.171.42:/tmp
</pre>
haga lo análogo de la segunda red a la primera:
<pre>
sudo scp /etc/isakmpd/private/local.pub usuario!@211.245.63.134:/tmp
</pre>

Después copielos a sus destinos, en el primer enrutador:

<pre>
sudo cp /tmp/local.pub /etc/isakmpd/pubkeys/ipv4/210.93.171.42
</pre>
y en el segundo
<pre>
sudo cp /tmp/local.pub /etc/isakmpd/pubkeys/ipv4/211.245.63.134
</pre>
Si por algún motivo su enrutador no tiene la llave privada, reinicie o examine instrucciones para generarla en ```/etc/rc```

!Pruebas

Para probar puede emplear el programa ```screen``` o bien varias términales desde X-Windows:

* Primera terminal del enrutador de primera red ejecute ```sudo isakmpd -Kd```
* Primera terminal del enrutador en la segunda red ejecute también ```sudo isakmpd -Kd```
* Segunda terminal del primer enrutador ejecute sudo ```ipsecctl -f /etc/ipsec.conf```
* En la segunda terminal de segundo enrutador también ejecute sudo ```ipsecctl -f /etc/ipsec.conf```

Una vez lo haga en las ventanas con ```isakmpd``` podrían verse mensajes esporádicos. En las ventanas de ```ipsecctl``` puede examinar flujos con:

<pre>
ipsecctl -sa
</pre>

y probar ```ping``` a la otra red interna (en ocasiones funciona ping de una red a otro pero no viceversa, y sólo después de un tiempo empieza a funcionar el otro sentido).   También puede monitorear la actividad
de conexión en tiempo real con ```ipsecctl -m```

Una vez establecido el flujo puede verificar que toda conexión se transmite encriptada por ejemplo examinando en el segundo nodo, el flujo proveniente del primero:

<pre>
sudo tcpdump -i xl1 -ttt -n -X host 211.245.63.134
</pre>

y en el primero iniciando una petición por red al segundo como:
<pre>
lynx http://210.93.171.42
</pre>

En el segundo nodo ```tcpdump``` debe presentar la información encriptada (note ```spi``` en el ejemplo siguiente):

<pre>
Jun 08 17:17:40.327329 esp 211.245.63.134 > 210.93.171.42 spi 0x2A9DAE0A seq 73 len 100
  0000: 4500 0078 a3f0 0000 4032 5960 c9f5 3f86  E..x??..@2Y`É??.
  0010: c85d ab2a !2a9d ae0a 0000 0049 e3db 6289  ?]"**.?....!I??b.
  0020: d4c8 b023 a44e 758f 69ea 2bd5 !1f4f 510f  ???#?Nu.iê+?.OQ.
  0030: 270c 9e24 b866 51ba 8219 !5e9e 65af 25e3  '..$?fQ?..^.e?%?
  0040: c75b 18d8 e70e 446c 16ba 007e 0459 5c59  ?[.??.Dl.?.~.Y\Y
  0050: cc2a                                     ?*
</pre>

!Persitencia entre arranques

Una vez compruebe que funciona el servicio puede configurar el archivo de arranque para que en cada arranque se establezca la conexión IPsec, para esto en cada nodo edite el archivo ```/etc/rc.conf.local``` y además de las reglas típicas para PF, agregue:
<pre>
ipsec="YES"
isakmpd_flags="-K"
</pre>

En el archivo ```/etc/rc.local``` agregue:

<pre>
pgrep isakmpd > /dev/null 2> /dev/null
if (test "$?" != "0" -a "$isakmpd_flags" != "NO") then {
        echo -n ' isakmpd';
        isakmpd $isakmpd_flags;
        sleep 1;
        ipsecctl -f /etc/ipsec.conf
} fi;
</pre>

!Operación

En  ocasiones si uno de los computadores conectados con IPSec sale de servicio cuando vulve a entrar se demora en establecer conexión IPSec, he notado que sirve ejecutar:

<pre>
sudo ipsecctl -f /etc/ipsec.conf
</pre>

en ambos nodos (mientras corra isakmpd que puede comprobarse con 
```pgrep isak```), cuando se establece la conexión IPSec si había conexiones IP
abiertas se caen, y para que sea bidireccional suele servir ejecutar ping
al otro nodo desde cada uno de los dos.

Aunque suelen funcionar las conexiones de protocolos TCP típicos como ssh, http, hay inconvenientes con ftp cuando el servidor está en una de las redes internas en configuración DMZ.

!Operación continúa

Al menos con versiones 4.2 y 4.3 de OpenBSD para mantener en forma la
conexión IPsec se ha requerido hacer ping constante y simultaneamente
desde los cortafuegos a conectar.   Esto puede automatizarse sincronizando hora de ambos cortafuegos con NTP y programanda una tarea cron que cada minuto que haga un ping al otro cortafuegos.


##BIBLIOGRAFÍA

* {1} Zero to IPSec in 4 minutes. Dragos Ruiu. 2006. http://www.securityfocus.com/infocus/1859
* {2} Páginas del manual ipsec, ipsec.conf e isakmpd
* {3} http://www.packetmischief.ca/openbsd/doc/ipsec.html

----
Esta información se cede al dominio público y se dedica al Padre que es VERDAD. 2008. vtamara@pasosdeJesus.org
