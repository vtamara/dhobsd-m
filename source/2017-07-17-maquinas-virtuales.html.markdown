---
title: Máquinas virtuales en OpenBSD (y adJ)
date: 2017-07-17 02:34 UTC
tags:
---

# Máquinas virtuales en OpenBSD (y adJ)

OpenBSD (y adJ) 6.1 para adm64 pueden manejar un monitor
de máquinas virtuales (vmm) al operar sobre procesadores con 
tecnología VT-x y paginación anidada o extendida.

Este monitor puede ejecutar varias máquinas virtuales huespedes
de manera independiente del anfitrión, cada una tiene una CPU virtual,
interfaces de red virtuales, imagenes de disco virtuales y puertos seriales
virtuales.

Se ha probado con éxito con OpenBSD 6.1 como huesped, que utiliza los
controladores virtio para interfaz de red, disco, memoria, generador
de números aleatorios y adaptador SCSI.  

Por lo mismos es una herramienta muy oportuna para probar nuevas
versiones de adJ y de OpenBSD.

# Prueba mínima

La ejecución de las diversas máquinas virtuales depende del servicio
```vmd```, el cual puede iniciar las máquinas virtuales que se configuren
en ```/etc/vm.conf``` y otras que se inicien por línea de ordenes.

Verifique que está en ejecución con
<pre>
pgrep vmd
</pre>

Si no está ejecutandose use:
<pre>
doas rcctl enable vmd
doas rcctl start vmd
</pre>

El control de las máquinas virtuales desde el interprete de ordenes
se hace con ```vmctl``` que facilita operar con las máquinas configuradas
en ```/etc/vm.conf``` u otras que se especifiquen en la línea de ordenes.

Por ejemplo cree un disco virtual de 16G con:
<pre>
doas mkdir /vmm/
doas vmctl create /vmm/mv1.img -s 16G
</pre>

Edite el archivo `/etc/vm.conf` para que su contenido sea:
<pre>
switch "local" {
    add em0
    add tap0
    add tap1
}

vm "mv1" {
    memory 512M
    kernel "/bsd.rd"
    disk "/vmm/mv1.img"
    interface {
        switch "local"
        lladdr 00:20:91:00:00:01
    }
}
</pre>

E inicie la máquina virtual con:
<pre>
vmctl start mv1
</pre>

Notará que arranca mediante el kernel /bsd.rd de la raíz de su disco.

# Recetas

Para ver las máquinas virtuales en ejecución:
<pre>
vmctl status 
</pre>



# Referencias

* man vmm
* man vmd
* man vmctl
* mancomandos virtio
* http://www.h-i-r.net/2016/10/openbsd-vmm-guided-tour-phpmysql-walk.html
* http://www.h-i-r.net/2017/04/openbsd-vmm-hypervisor-part-2.html
* https://www.openbsd.org/faq/faq6.html
* https://software.intel.com/sites/default/files/m/4/1/9/7/c/25039-final_cpu_1027.pdf
