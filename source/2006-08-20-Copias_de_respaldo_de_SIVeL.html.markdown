---
title: Copias_de_respaldo_de_SIVeL
date: 2006-08-20 12:00 UTC
tags:
---
CREACION DE UNA COPIA DE RESPALDO

Para crear una copia de respaldo se siguen los siguentes pasos:

# Terminal (ingresar a una terminal)
# ```ssh sivel-demo@base``` (entrar a sivel en computador base)
# ```cd /home/sivel/sivel``` (cambiar de directorio al de sivel)
# ```ls``` (mirar el listado de archivos)
# ```less confrespaldo.sh``` ver configuración de respaldos, en particular debe verse el valor de la variable ```rlocal```, es una línea de la forma ```rlocal= /var/bakbase/``` que indica el directorio donde quedarán las copias.
# ```cd /var/bakbase``` (cambiar de directorio)
# ```ls -l``` (mirar archivos con detalles) 
# ```cd /home/sivel/sivel/``` (cambiar nuevamente al directorio de SIVeL).
# ```./pgdump.sh``` (dar el comando para sacar una copia de la base el cual nos da el resultado al final con la fecha y hora de la copia.)
