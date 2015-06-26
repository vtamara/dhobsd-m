---
title: Recuperar_Subparticiones_con_scan_ffs
date: 2009-10-21
tags:
---
Si algún programa borra la lista de subparticiones (o labels) de una partición de OpenBSD, pero no sobreescribe toda la partición del BIOS, existe la posibilidad de reconstruirla.

!Diagnósticando

# Al intentar iniciar por el OpenBSD no resulta posible.
# Al iniciar con un CD de instalación de OpenBSD y a la línea de comandos y ejecutar ```disklabel /dev/rwd0c``` o ```disklabel /dev/rsd0c``` según el disco duro.  Aparece una etiqueta en blanco.

El borrado mencionado por ejemplo ocurrió el 20.Oct.2009 tras usar resize2fs desde un instalador de Ubuntu, en un sistema dual (Ubuntu y OpenBSD) en el que se quería redimensionar una partición destinada a Linux. 


!scan_ffs

El programa scan_ffs incluido en el sistema base de OpenBSD permite buscar el comienzo de etiquetas FFS.  Para usarlo puede instalar un sistema OpenBSD mínimo en la partición que en el BIOS está destinada a OpenBSD, debe ser mínimo porque estará sobreescribiendo  datos de sus antiguas subparticiones.

Una vez instalado emplee scan_ffs para buscar entre el primer y último bloque de la partición destinada a OpenBSD, scan_ffs reportará los bloques donde encuentre posibles comienzos de sistemas FFS, emplee esa información y su conocimiento de la organización previa del disco para volver a construir la tabla de subparticiones con disklabel.  
