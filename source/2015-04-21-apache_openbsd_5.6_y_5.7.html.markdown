---
title: apache_openbsd_5.6_y_5.7
date: 2015-04-21
tags:
---
Dado que algunas instalaciones de adJ y OpenBSD 5.6 y 5.7, pueden requerir Apache 1.3.29 (incluido en el sistema base hasta OpenBSD 5.5), explicamos como puede mantenerlo al menos en adJ/OpenBSD 5.6 y 5.7.

## La historia

Hasta adJ/OpenBSD 5.5, se incluía en el sistema base como servidor web un histórico Apache 1.3.29 con parches de OpenBSD. Sin embargo  por su obsolecencia en OpenBSD y adJ 5.6 se sacó del sistema base y se dejaron ```nginx``` y un par de nuevos servicios desarrollados por OpenBSD: ```httpd``` y ```relayd```.

Para OpenBSD 5.6 el Apache que había en el sistema base de 5.5 se pasó al paquete (```apache-httpd-openbsd```).  No se recomienda su uso, pero mientras usted avanza en migración puede usarlo como se explica a continuación.


## Configuración y uso

Instale el paquete:
<pre>
sudo pkg_add apache-httpd-openbsd
</pre>

En el archivo ```etc/rc.conf.local```  haga los siguientes cambios (parcialmente explicados en {2}):
# Renombre ```httpd_flags``` por ```apache_flags```
# En la variable ```pkg_scripts``` remplace ```httpd``` por ```apache``` (y  de requerirse saque ```nginx```).

Modifique el archivo de configuración ```/var/www/conf/httpd.conf```, el cambio evidente es modificar la ruta de los módulos activos para que sean cargados de ```/usr/local/lib/apache/``` en lugar de ```/usr/lib/apache```

Puede probar reiniciar el servidor completo para asegurar que el Apache 1.3.29 arranca también, o bien iniciar sólo el servicio con:
<pre>
sudo sh /etc/rc.d/apache start
</pre>


## El futuro

Apache 2 ha aumentado características, pero ha perdido instalaciones a nivel de servidores mientras que ```nginx``` ha ido ganando (ver {1}).  ```nginx``` aunque tiene una licencia BSD, ha venido aumentando en características, no necesariamente implementadas de forma segura (ver {3}) por lo que ha sido cada vez más dificil de mantener en el sistema base de OpenBSD.  Por esto los desarrolladores de OpenBSD han considerado a futuro eliminar el obsoleto porte ```apache-httpd-openbsd``` y sacar ```nginx``` del sistema base para dejarlo como porte junto con el porte de Apache 2.   Así que proximamente seguramente en sistema base quedarán sólo ```httpd``` y ```relayd```, mientras que Apache 2 y nginx quedarán como paquetes (y otros servidores web  menos populares, pero no ```apache-httpd-openbsd```).


Por esto se recomienda:
* Migrar a OpenBSD ```httpd``` los sitios web estáticos o dinámicos que puedan usar FastCGI y que no requieran reescritura por parte del servidor web.  Puede ver casos en [estas instrucciones | httpd].   Si sus intentos no fructifican al menos podrá reusar la configuración pues son muy similares la sintaxis de OpenBSD ```httpd``` y de ```nginx```.
* Migrar a ```relayd``` los proxys que operen sobre Apache o sobre ```nginx```
* Los sitios dinámicos que requieran algo de reescritura migrarlos a ```nginx``` (ver {4})
* Los sitios web que tengan complejas reglas de reescritura migrarlos a Apache 2 (ver {5}).


## Bibliografía

* {1} http://news.netcraft.com/archives/2015/04/20/april-2015-web-server-survey.html
* {2} http://www.openbsd.org/faq/upgrade56.html#Pkgup
* {3} http://www.openbsd.org/papers/httpd-asiabsdcon2015.pdf
* {4} http://marc.info/?l=openbsd-misc&m=142016708910990&w=2
* {5} http://marc.info/?l=openbsd-misc&m=141989291407273&w=2
