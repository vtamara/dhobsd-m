Para copiar el disco de un servidor en otro

Antes instalar rsync en ambos

Digamos que el servidor origen tiene IP 192.168.1.1 y que el servidor destino tiene IP 192.168.1.2:

En servidor destino:
1. Crear una estructura de particiones análoga en destino
2. En el servidor destino crear una cuenta rsyncer en grupo wheel para recibir copia

En computador origien:
1. Desde la cuenta root del computador origen crear un para de llaves RSA:
ssh-keygen -r rsa
2. Envíe la llave pública como llave autorizada unica para rsynce en servidor destino:

3. Generar un par de llaves Poner llave publica en destino /home/rsyncer/.ssh/authorized_keys
3. En el servidor origen ejecutar:
/usr/local/bin/rsync --delete  --rsh=ssh -rauvz  -e "ssh" --rsync-path="sudo /usr/local/bin/rsync" "/home/" rsyncer@192.168.1.2:/home/"
