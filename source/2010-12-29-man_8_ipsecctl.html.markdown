---
title: man_8_ipsecctl
date: 2010-12-29 12:00 UTC
tags:
---
##NOMBRE 
    ```ipsecctl``` - control de flujo de IPsec 


##SINOPSIS 
<pre>
     ipsecctl ![-dFkmnv] ![-D macro= valor] ![-f archivo] ![-s modificador]
</pre>

##DESCRIPCIÓN 

La utilidad ```ipsecctl``` controla los flujos que determinan qué paquetes deben ser procesados por IPsec. Permite configuración del conjunto de reglas, y la recuperación de la información de estado de SPD (Base de datos de políticas de seguridad - Security Policy Database) del kernel y SAD (Base de datos de asociaciones de seguridad - Security Asociation Database). También puede controlar ```isakmpd(8)``` y establecer túneles utilizando administración de claves automática con ```isakmpd(8)```. 
La gramática del conjunto de reglas se describe en ipsec.conf(5). 

Las opciones son las siguientes: 

```-D macro=valor``` 
             Define que ```macro``` se establezca en el ```valor``` dado en la línea de comandos.  Reemplaza la definición de ```macro``` en el conjunto de reglas. 

```-d```
             Cuando la opción ```-d``` es dada, los flujos especificados serán eliminados del SPD. De lo contrario, ```ipsecctl``` agregará flujos. 

```-F```
             La opción -F vacía el SPD y el SAD. 

```-f archivo ```
             Cargua las reglas contenidas en el ```archivo```. 

```-k```
             Mostrar material de llaves secretas cuando se imprimen las entradas activas de SAD. 

```-m```
             Continuamente mostrar todos los mensajes PF_KEY intercambiados con el kernel. 

```-n```
             No carga las reglas, solo las analiza. 

```-s modificador ```
             Muestra la bases de datos del kernel especificada ```modificador``` (puede ser abreviado): 

| ```-s flow``` | Muestra el conjunto de reglas cargadas en el SPD.|
| ```-s sa``` | Muestra las entradas activas SAD. |
| ```-s all``` | Muestra todo lo anterior. |

```-v```
             Produce una salida más prolija. Un segundo uso de ```-v``` producirá salida aún más prolija. 


##VER TAMBIÉN 

             ```[ipsec(4) | man 4 ipsec]```, ```tcp(4)```, ```[ipsec.conf(5) |man 5 ipsec.conf]```, ```[isakmpd(8) | man 8 isakmpd]```

##HISTORIA 
El programa ```ipsecctl``` apareció por primera vez en OpenBSD 3.8. 


OpenBSD 4.8 31 de mayo 2007 de OpenBSD 4.8

Traducción a español de Dimitri <trichotecene@yahoo.es> bajo licencia BSD, con adaptaciones de dominio público de vtamara@pasosdeJesus.org
