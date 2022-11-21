---
title: copiar-servidor
date: 2016-03-07 12:00 UTC
tags:
---

Para copiar el disco de un servidor en otro

## Prerrequesitos
Antes instalar ```rsync``` en ambos

## Procedimiento

Digamos que el servidor origen tiene IP 192.168.1.1 y que el servidor destino tiene IP 192.168.1.2:

En servidor destino:

1. Cree una estructura de particiones análoga en destino
2. Cree una cuenta ```rsyncer``` en grupo ```wheel``` para recibir copia

En servidor origien:

- Desde la cuenta ```root``` cree un par de llaves RSA:
```
ssh-keygen -t rsa
```
- Envíe la llave pública como llave autorizada unica para rsyncer en servidor destino:
```
scp /root/.ssh/id_rsa.pub rsyncer@192.168.1.2:.ssh/authorized_keys
```
Para enviar la copia, ejecute en el servidor origen:
```
/usr/local/bin/rsync --delete  --rsh=ssh -rauvz  -e "ssh" --rsync-path="doas /usr/local/bin/rsync" "/home/" "rsyncer@192.168.1.2:/home/"
```
