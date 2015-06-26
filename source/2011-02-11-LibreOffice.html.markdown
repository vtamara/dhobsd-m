---
title: LibreOffice
date: 2011-02-11
tags:
---
Aplicaciones de Ofitmática descendientes de OpenOffice. Su primera versión es la 3.3.0 liberada el 25.Ene.2011

!Instalación de !LibreOffice 3.3.0 en Ubuntu 10.04 y 10.10:


1. Eliminar por completo OpenOffice con: 
<pre>
   sudo apt-get remove openoffice*.*
</pre>

2. Descargar de http://www.libreoffice.org/download/ la versión .deb, son 3 archivos
*!LibO_3.3.!0_Linux_x86_install-deb_en.US.tar.gz                                   
*!LibO_3.3.!0_Linux_x86_langpack-deb_es.tar.gz                                     
*!LibO_3.3.!0_Linux_x86_helppack-deb_es.tar.gz  

Descomprimirlos en su carpeta personal. Esto se hace pulsando sobre el archivo con el botón izquierdo.  Se abre un ventana en Gestor de Archivadores. Resaltar el nombre del archivo que aparece y pulsar "extraer".  Renombrar las carpetas descomprimidas para que correspondan (en el mismo orden) a:

* install
* lang
* help

Instalar el sistema base en inglés con los siguientes comandos.  Nota que si su Ubuntu está en español, entonces sustituya "Desktop" por "Escritorio".  Si la descompresión es a otra carpeta, ponga el nombre de esa carpeta en vez de "Desktop":

<pre>
sudo dpkg -i ~/Desktop/install/DEBS/*.deb
</pre>
Instalar menús con:
<pre>
sudo dpkg -i ~/Desktop/install/DEBS/desktop-integration/*.deb
</pre>
Instalar localización en español con:
<pre>
sudo dpkg -i ~/Desktop/lang/DEBS/*.deb
</pre>
E instalar ayuda en español con:
<pre>
sudo dpkg -i ~/Desktop/help/DEBS/*.deb
</pre>


##REFERENCIAS

* {1} http://askubuntu.com/questions/7373/how-to-install-libreoffice-replacing-openoffice-org
