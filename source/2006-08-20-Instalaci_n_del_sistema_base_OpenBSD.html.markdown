---
title: Instalaci_n_del_sistema_base_OpenBSD
date: 2006-08-20
tags:
---
El OpenBSD debe ser instalado en una particion primaria del Disco Duro

Nos aseguramos que arranque por el CD o el disquete y se dijo que  que para incidir  el arranque habia que oprimir F8 o Del (dependiendo del BIOS) y así obteniamos la posibilidad de arrancar por un CD o un disquete, prestablecido para el arranque. Obviamente antes de arrancar la instalación fuimos advertidos sobre la necesidad de salvar la información que estaba en la partición donde instalaríamos el programa.
Para cambiar el tamaño de un sistema de archivos (y una partición) puede
usarse el programa ```parted``` que corre sólo en Linux, por lo que es necesario iniciar el computador con un LiveCD (como Ubuntu) o con un disquette de arranque de Linux que incluya ```parted```.

Seguidamente recordamos que para crear un diskette de arranque teniamos que utilizar el siguiente procedimietno,
# Descargar la imagen, por ejemplo de http://paud.sf.net
# Para copiar la imagen en un disquete desde un sistema OpenBSD 
<pre>
dd if=paud-2.0.3.img of=/dev/fd0a
</pre>
si es un Linux usar ```/dev/fd0```

Luego buscamos una partición donde instalar, comenzamos ejecutando ```df -h``` para precisar las particiones montadas, también puede usarse ```fdisk /dev/hda``` (en OpenBSD usar ```/dev/rwd0c```); luego procedimos a ejecutar el comando ```h``` para revisar la ayuda y después ```p``` y asi revisar los tamaños:
* Encontramos que  que el disco duro de la máquina  estaba divdida en 6 parte  y que no se encotraban en orden numérico, pero que cada una de ellas tenía  una capacidad de memoria ocupada y otra parte en blanco y que sus particiones se encontraban en 3 nombres, cuatro de ellas en linux , una extendida y otra swap. 
* Aqui entramos a escoger cuál de ellas era más conveniente para tomar como espacio de instalación y para ello, hicimos algunas conversiones y unos cálculos para determinar la función y capacidad de trabajo de acuerdo a sus  disposiciones en la máquina. Entonces procedimos con cada una de ellas  medainte la designación hda1, hda2, hda3 y hda4, revisamos desde donde comenzaba y dónde terminaba la partición donde instalarímos el programa que nos permitiría contar con la herramienta para mejorar nuestras bases de datos. Antes de continuar conocimos el manejo de algunas órdenes como ```m``` para ver ayuda, ```n``` para adicionar una nueva partición y ```p``` para ver las particiones disponibles.

Los pasos a seguir para la instalación son:

# Es preciso que insistamos en que si se hace mal algo en este punto el resultado será de una pérdida significativa de datos. Si se va a realizar sobre un dispositivo con datos importantes, puede ser conveniente practicar antes con un dispositivo de "usar y tirar?, además de hacer una buena copia de seguridad.
# Redimensionar el sistema de archivo en caso de tener otro sistema operativo, bien sea Linux o Windows.
# Crear una partición en el espacio liberado.
# Instalar el sistema base de OpenBSD. La instalación se hace desde un cd, disquette o desde un servidor FTP.

Nota: La identificación del disco duro maestro del primero puerto IDE en Open BSD es wd0

Para el redimensionamiento del sistema de archivos utilizamos un disquette que contenga la herramienta ```parted```, por ejemplo PAUD.

PAUD fue descargado del sitio http://paud.sf.net. Posteriormente se crea una imagen en un disquete, para esto utilizamos:
<pre>
dd if=paud-2.0.3.img of=/dev/fd0
</pre>
```dd``` es un comando que permite poner imagenes a bajo nivel.

Antes de redimensionar debemos averiguar cual es la partición en la que se va a ejecutar parted. Para esto utilizamos ```dh -f```  que muestra las particiones montadas.

Para arrancar por el disquete se debe configurar la BIOS para que arranque por el disuqete, esto se hace cuando el computador arranca presiondando
F8 (o DEL o alguna tecla dependiendo de la BIOS) y se elige arrancar por el disquete.

Luego de arrancar por el disquete PAUD que es un sistema Linux comenzamos  a redimensionar con la herramienta parted.

Una vez se redimensione uno de los sistemas de archivos, se contará con
espacio en la partición donde estaba, este espacio debe reasignarse a una
partición primaria para OpenBSD.  Esto puede hacerse mejor con el programa
```fdisk``` de OpenBSD que está en el instalador de ese sistema, por lo que es buen momento para iniciar la instalación de OpenBSD.   Para esto puede arrancar por el CD o por un disquette.

Un ejemplo de una sesión con fdisk es:

<pre>
You will now create a single MBR partition to contain your OpenBSD data. 
This partition must have an id of 'A6'; must *NOT* overlap other partitions;
and must be marked as the only active partition.

The 'manual' command describes all the fdisk commands in detail.

Disk: wd0 geometry: 2586/240/63 39100320 Sectors? Offset: 0 Signature: 0xAA55
Starting Ending LBA Info:
#: id C H S - C H S start: size?
*0: 06 0 1 1 - 202 239 63 63: 3069297? DOS > 32MB
1: 00 0 0 0 - 0 0 0 0: 0? unused 
2: 00 0 0 0 - 0 0 0 0: 0? unused 
3: 00 0 0 0 - 0 0 0 0: 0? unused
Enter 'help' for information fdisk: 1> help
help Command help list manual Show entire OpenBSD man page for fdisk 
reinit Re-initialize loaded MBR (to defaults) 
setpid Set the identifier of a given table entry 
disk Edit current drive stats edit Edit given table entry 
flag Flag given table entry as bootable 
update Update machine code in loaded MBR 
select Select extended partition table entry MBR 
print Print loaded MBR partition table 
write Write loaded MBR to disk 
exit Exit edit of current MBR, without saving changes 
quit Quit edit of current MBR, saving current changes
abort Abort program without saving current changes

fdisk: 1>
</pre>

Luego revisamos la raíz del programa: pwd, ls /home para ver los correos del archivos y /var/.0 la base de datos y acomodamos la partición swap era 500 megas. Posateriormetne establecimos una contraseña y ususario que se llamó red y entonces se dijo que  que para buscar ayudas podiamos ver la dirección de guias http://structio.sourceforge.net/guias.
 Se enrutaron un administrador del sistema que se denominó root y el sivel como un ususario del sistema y se señalaron los pasos fundamentales para activar el cargador de arranque
# root
# editar ```/etc/lilo/.conf``` para agregar
<pre>
other = /dev/hda4
       Label =openBSD
       table = /dev/hda
</pre>
y ejecutar
<pre>
lilo
</pre>


Si se usa Windows NT/XP/2000 puede emplearse la herramienta bootpart para configurar el manejador de arranque (```NTLDR```), tal herramienta está incluida en el CD Aprendiendo de Jesús y en Internet en:
http://www.winimage.com/bootpart.htm

Una vez ubicada por ejemplo en c:\part, abrir un interprete de comandos (Inicio -> Ejecutar -> cmd) y ejecutar:
<pre>
cd c:\part
bootpart
</pre>
para ver una lista de los sistemas operativos detectados en las diversas particiones.

Identifique el número que corresponde a la partición de OpenBSD (tipo A6), supondremos para este ejemplo que es el 1.  Después ejecute:
<pre>
bootpart 1 LBA ```C:\OpenBSD.prt``` OpenBSD
</pre>
remplazando 1 por el número de partición de OpenBSD.

Estos creará el archivo ```C:\OpenBSD.prt``` con el comienzo de la partición de OpenBSD y habrá una nueva entrada en el menú de arranque con etiqueta OpenBSD.


