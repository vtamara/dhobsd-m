---
title: Uso_de_red_en_qemu
date: 2006-08-26 12:00 UTC
tags:
---
Para configurar red en qemu 0.8.1, digamos para que el sistema emulado
funcione con dirección 192.168.5.5:

* Copie el archivo de configuración:
  <pre>
  cp /usr/local/share/examples/qemu/qemu-ifup /etc/qemu-ifup
  </pre>
* Edite ```/etc/qemu-ifup``` y cambielo para que referencie su interfaz de red:
  <pre>
  ETHER=rl0
  </pre>
  y agregue al final:
  <pre>
  $SUDO ifconfig $1 192.168.5.1
  </pre>
* Modifique ```/etc/pf.conf``` para que haga NAT a la subred 192.168.5 y para 
  que permita conexión desde esa subred, por ejemplo:
  <pre>
  ...
  qemu_if="tun1"
  ...
  nat on $ext_if from !($ext_if) -> ($ext_if:0)
  ...
  pass in on $qemu_if proto tcp from any to any port ssh keep state
  </pre>
  y reinicielo
  <pre>
  pfctl -f /etc/pf.conf
  </pre>
* Ejecute ```qemu``` con opciones de red:
  <pre>
  qemu -net nic -net tap -cdrom pasos.iso disco.img
  </pre>
* El sistema emulado configurelo para que tenga dirección 192.168.5.5 y 
  que la compuerta sea 192.168.5.1, si es un OpenBSD, editando 
  ```/etc/hostname.ne3``` para que diga:
  <pre>
  inet 192.168.5.5 255.255.255.0 192.168.5.255
  </pre>
  y que ```/etc/mygate``` diga:
  <pre>
  192.168.5.1
  </pre>
  reconfigure la red con
  <pre>
  sh /etc/netstart
  </pre>


----
Esta información se cede al dominio público y se dedica a Dios. vtamara@pasosdeJesus.org
