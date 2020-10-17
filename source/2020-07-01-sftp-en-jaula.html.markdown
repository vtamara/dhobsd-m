---
title: SFTP limitado a Jaula
date: 2020-07-01 00:00 UTC
tags:
---

Supongamos que requiere que unos usuarios puedan ingresar solo por sftp a
una jaula de archivos que incluya sus archivos y otros directorios y archivos
pero no al sistema de archivos completo.
Para esto agregue un grupo, digamos `jaulasftp`, al archivo `/etc/group`
con un gid no usado (digamos 1002):

        jaulasftp:*:1002:

Indique a sshd la configuración especial para estos usuarios agregando a
`/etc/ssh/sshd_config` :

```
Match group jaulasftp
  ForceCommand internal-sftp
  ChrootDirectory /restovar/jaulasftp
  PermitTunnel no
  AllowAgentForwarding no
  AllowTcpForwarding no
  X11Forwarding no
```

Note que no se le permite realizar túneles ni reenvíos y sólo puede
usar el sistema `internal-sftp` confinado a `/restovar/jaulasftp`

Por ejemplo con `doas vipw` edite cada usuario por limitar (en el siguiente
ejemplo `uslim`) para asegurar que su línea es de la forma siguiente:

        uslim:$2b$10$dEPqLC7YSmilUNURQXp2AeiptJVJ38H6ZsiI25w7fisMboDkBCZy.:1005:1002::0:0:Usuario Limitado:/home/uslim:/bin/false

Teniendo en cuenta que para este usuario de ejemplo `uslim`:

1. Su grupo principal es el 1002 - es decir `jaulsftp`
2. No tiene interprete de ordenes, es decir que es `/bin/false`
3. Su directorio personal es `/home/uslim` pero dentro de jaula, pues
   el administrador ubicará su directorio en `/restovar/jaulasftp/home/uslim`


Cree el directorio `/restovar/jaulasftp/home/uslim` y ponga la información
del directorio personal del usuario `uslim`.

Cuando ese usuario ingrese vía sftp verá como raíz del sistema de
archivos lo que haya en `/restovar/jaulasftp`  y podrá ingresar a los
directorios y archivos dentro de esa jaula en la medida que el sistema de
permisos típico se lo permita.


### Referencias y lecturas recomendadas 

Las siguientes páginas man: sshd 8, sftp 1.

Separación de privilegios:
<http://www.counterpane.com/alert-openssh.html>

Página web: <http://www.openssh.com>

Ejemplo de jaula sftp: <https://www.golinuxcloud.com/sftp-chroot-restrict-user-specific-directory/>

