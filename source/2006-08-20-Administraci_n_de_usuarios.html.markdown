---
title: Administraci_n_de_usuarios
date: 2006-08-20
tags:
---
Desde otra máquina ingresamos a la máquina que tiene OpenBSD cuya IP es 192.168.3.6 utilizando ssh:
<pre>
ssh !momo@192.168.3.6
</pre>

Estando ya en OpenBSD, desde root ejecutamos

<pre>
# adduser
</pre>

al ejecutar adduser me solicita el nombre de usuario
<pre>
Enter username []:nono
</pre>
damos enter, y después me solicito el nombre completo
<pre>
Enter full name []:Momo Cuama
</pre>
Luego de responder varias preguntas me pide el password
<pre>
Enter password []:
</pre>
Y luego repetir el password
<pre>
Enter password again []:
</pre>
al final nos muestra lo siguiente:
<pre>
Name:        nono
Password:    ****
Fullname:    momo cuama
Uid:         1014
Gid:         1014 (nono)
Groups:      nono
Login Class: default
HOME:        /home/nono
Shell:       /bin/ksh
OK? (y/n) [y]: y
Added user ``nono''
Copy files from /etc/skel to /home/nono
Add another user? (y/n) [y]: n
Goodbye!
</pre>
