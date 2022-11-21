---
title: Respaldo_altroot
date: 2014-08-13 12:00 UTC
tags:
---
Para respaldar un disco de un sistema OpenBSD adJ, puede conectarse 
un segundo disco de las mismas características y 
configurar una copia casi-espejo diaria del primer disco en el segundo.

Decimos casi-espejo porque la partición raiz se copiará a manera de espejo con dd (para permitir arrancar por esa), pero la de otras particiones por velocidad se copian con ```rsync```.


##CONFIGURACIÓN

Supondremos que ambos discos son de las mismas características y cada uno tiene 3 particiones: raiz ```/```, ```/home``` y ```/var```

La particion raiz no debe ser tan grande pues es la que se copia con ```dd```.

La copia de la particion raiz con ```dd``` la hace OpenBSD en ```/etc/daily``` cuando el segundo disco se monta como ```/altroot``` y cuando se pone
<pre>
ROOTBACKUP=1
</pre>
en ```/etc/daily.local```  (ver http://www.openbsd.org/faq/faq14.html#altroot )

Los sistemas de archivos de ```/var``` y ```/home``` también se copian diariamente dejando en ```/etc/daily.local```:
<pre>
  /usr/local/bin/rsync -ravzp /var/* /altroot/var/
  /usr/local/bin/rsync -ravzp /home/* /altroot/home/
</pre>
Aseguramos el montaje durante el arranque con lo siguiente en ```/etc/rc.local``` (cambiar sd1 por wd1 de acuerdo al hardware):
<pre>
/sbin/mount | /usr/bin/grep "\/sd1e" > /dev/null 2>&1
if (test "$?" != "0") then {
    /sbin/fsck -y /dev/sd1e
    /sbin/mount /dev/sd1e /altroot/var/
} fi;

/sbin/mount | /usr/bin/grep "\/sd1d" > /dev/null 2>&1
if (test "$?" != "0") then {
    /sbin/fsck -y /dev/sd1d
    /sbin/mount /dev/sd1d /altroot/home
} fi;
</pre>


## CONCLUSIÓN

Con este tipo de respaldo en caso de desastre en el primer disco basta poner el segundo disco como primario y se tendrá la configuración del dia anterior.

-----
Cedido al dominio público, dedica al Dios, quien es Dios de orden (Santiago 3:16).

