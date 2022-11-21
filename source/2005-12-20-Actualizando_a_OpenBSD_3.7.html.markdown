---
title: Actualizando_a_OpenBSD_3.7
date: 2005-12-20 12:00 UTC
tags:
---
##Actualizando a OpenBSD 3.7

La actualización resulta bastante directa.

* Descargue el sistema base y versiones para 3.7 de los paquetes que tenga instalados (para traer los paquetes puede emplear el script ```herram/bajapaq.sh``` que va en las fuentes de ```usuario_OpenBSD```).
* Cree un disquete de inició y empleelo para arrancar el computador que vaya a actualizar, suponiendo que descargó los juegos de instalación en el mismo computador indique la ruta donde se encuentran los juegos de instalación (durante la actualización se montan los sistemas de archivos en ```/mnt```). Si tiene varios computadores por actualizar puede configurar uno para que sirva los archivos de instalación por ftp o por http. También es posible omitir la creación de disquette, usando el kernel bsd.rd de OpenBSD 3.7, basta copiarlo en / y arrancar por este (durante el arranque ```boot> boot bsd.rd```).
* Una vez complete la actualización reinicie.

Después de actualizar eñ sistema base, casi todos los paquetes de 3.6 seguirán funcionando (una notable excepción es Mozilla). Desde el directorio donde haya descargado los paquetes para 3.7, tras refinar los que desea, puede actualizar los paquetes con las nuevas opciones de ```pkg_add```:
<pre>
for i in `ls`; do echo $i;
  sudo pkg_add -F libdepends -F boguslibs -F installed -r $i;
done 2> e
</pre>

Este paso puede repetirse dos veces para facilitar resolución de dependencias.

Los paquetes que no puedan actualizarse porque tienen "unsafe operations" eliminelos manualmente con pkg_delete y tras esto agreguelos con:
<pre>
for i in `ls`; do echo $i;
sudo pkg_add $i;
done 2> e
</pre>

En algunos paquetes puede requerir más tiempo resolviendo dependencias (examine los problemas con less e), por ejemplo php4-core debe ser desinstalado manualmente primero y después la nueva versión debe ser instalada (sin olvidad activar con phpxs -s).

O por ejemplo el mensaje:
<pre>
  New package ispell-3.2.06p0 contains unsafe operations
</pre>
obliga a ejecutar:
<pre>
  # pkg_delete -f dependencies ispell
  # pkg_add ./ispell-3.2.06p0.tgz
  # pkg_add ./ispell-spanish-3.2.06p0.tgz
  # ispell-config
</pre>
A continuación siga los pasos de la sección "updates to /etc" de http://www.openbsd.org/faq/upgrade37.html y finalmente tendrá 3.7 funcionando con paquetes nuevos.

Entre las novedades a nivel de paquetes he notado:

* paquete ```scribus``` para hacer diagramación de libros (funciona bien).
* paquete ```sodipodi``` para hacer graficos vectoriales (funciona bien).
* paquete ```qemu``` para emular muy rápido un PC (o un PowerPC o un Sparc).

para el autor de este escrito es una lastima que todos sean GPL.

Entre cosas que tenía OpenBSD 3.6:

* ```vlc``` para ver videos (MPG, MOV, OGG) o DVDs, escuchar audio (MPG, OGG) o CDs de audio o mostrar audio o video trayendo de la red por diversos protocolos. Hasta el momento sólo he probado con un .mov de una manifestación en Chicago por la CDPSJA (el mismo día que fue la manifestación aquí frente a la embajada de EUA, así como en otras 13 ciudades) y se ve bien.

Información dedicada a Dios y liberada al dominio público. vtamara@pasosdeJesus.org 2005
