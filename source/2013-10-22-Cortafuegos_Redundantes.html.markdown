---
title: Cortafuegos_Redundantes
date: 2013-10-22
tags:
---
# 1. CONFIGURACIÓN

Siguiendo {1}

# 2. OPERACIÓN

# 3. RESPALDO DE OTROS SERVICIOS

Configure el cortafuegos 2 para recibir archivos por rsync, y envielos con periodicidad desde el cortafuegos 1 ---o cuando haya cambios en los archivos de los servicios que respaldará.

SMTP intentar que sea backup el cortafuegos 2.

Seguimos el procedimiento de {5} que nos ha permitido preservar usuarios y permisos.

En el cortafuegos 2:

* Cree el directorio donde recibirá los archivos ```mkdir /var/respaldoc1/```
* Agregue un usuario `rsyncer`
* Dele permiso de utilizar `doas` para poner los archivos que `rsync` envíe, ejecutando ```doas vi /etc/doas.conf``` y agregue la línea ```permit nopass rsyncer as root cmd /usr/local/bin/rsync```

En el cortafuegos 1 como usuario root:
* Genere llaves (sin clave para automatizar) con ```ssh-keygen```
* Agregue la llave pública a `~/.ssh/authorized_keys` del usuario `rsyncer en el cortafuegos2: scp /root/.ssh/id_rsa2.pub rsyncer@c2:.ssh/authorized_keys

En el cortafuegos 2 como usuario rsyncer:
* Edite ```authorized_keys``` y al comienzo de la línea agregue ```from="192.168.2.1",command="sudo -E /usr/local/bin/rrsync /var/respaldoc1"```

En el cortafuegos 1:
* Pruebe
* Ejecute
* Cree el archivo /usr/local/respaldaenc2 con
/usr/local/bin/rsync --rsh=ssh -ravz  /var rsyncer@192.168.2.49:/
/usr/local/bin/rsync --rsh=ssh -ravz  /etc rsyncer@192.168.2.49:/
* Agregue a /etc/daily.local
<pre>
/usr/local/bin/respaldaenc2
sudo chmod +x /usr/local/bin/respaldaenc2
</pre>

De esta manera se realizará copia diaria (hacia la medianoche) de los directorios /etc y /var del primer cortafuegos al segundo en /var/respaldoc1/etc y /var/respaldoc1/var

Sería ideal configurar cada servicio que opere en el primer cortafuegos para que copie los archivos que cambien.


# 4. REFERENCIAS

* {1} https://calomel.org/pf_carp.html
* {2} http://www.countersiege.com/doc/pfsync-carp/
* {3} man rsync, man rsyncd.conf
* {4} http://everythinglinux.org/rsync/
* {5} http://www.v13.gr/blog/?p=216
