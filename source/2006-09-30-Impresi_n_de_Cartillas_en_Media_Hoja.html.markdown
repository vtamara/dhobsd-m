---
title: Impresi_n_de_Cartillas_en_Media_Hoja
date: 2006-09-30
tags:
---
¿Alguna vez ha necesitado generar con OpenBSD una cartilla para imprimir y distribuir?  Un formato apropiado para cartillas es media hoja oficio, por ejemplo puede formarse una con una pila de 3 o 4 hojas tamaño oficios doblando la pila por la mitad.

Puede verse un ejemplo de esto en:
http://fpv.gfc.edu.co/?id=Cartilla+Resolucion+de+Conflictos

donde puede descargar dos archivos !PostScript para imprimir cada uno de las caras de 3 hojas de papel oficio.

En este escrito describimos como se generó la mencionada cartilla.

##Preparando el contenido

Un wiki (como este) es una herramienta muy apropiada para editar y publicar contenidos, pero no para imprimirlos.  Este wiki empela ErfurtWiki, así que un primer paso es pasar el contenio del marcado de ErfurtWiki a LaTeX.  Esto puede facilitarse con un script AWK
[ewiki2latex.awk | http://dhobsd.pasosdeJesus.org/index.php?binary=internal%3A%2F%2F052ac673d56dde29da1b3e81cf84f2c1.bin]

Este script puede recibir un modo de operación en la variable de ambiente MODO, los posibles son:
* ```sin-pre=1``` No incluye preámbulo LaTeX
* ```cartilla=1``` incluye preámbulo LaTeX para una cartilla como la descrita
* Vació:  Genera un documento LaTeX

##LaTeX

El preámbulo LaTeX para cartillas debe incluir:
<pre>
\documentclass[11pt,spanish]{article}
\usepackage[dvips,paperheight=8.5in,paperwidth=6.5in]{geometry}
\geometry{verbose,tmargin=10mm,bmargin=20mm,lmargin=15mm,rmargin=25mm,headheight=15mm,headsep=5mm,footskip=5mm}
</pre>
Para que las dimensiones de la página correspondan a media hoja tamaño
oficio (colombiano) en modo horizontal ("landscape").

El archivo LaTeX debe ser mejorado (para incluir diseño y correcciones a detalles que el script no convierta) y una vez completado puede procesarse para generar un DVI con:
<pre>
TEXINPUTS=$$TEXINPUTS:/usr/local/lib/hevea latex cartilla.tex
</pre>
Si el documento incluye bibliografía, también requiere:
<pre>
bibtex cartilla
TEXINPUTS=$$TEXINPUTS:/usr/local/lib/hevea latex cartilla.tex
TEXINPUTS=$$TEXINPUTS:/usr/local/lib/hevea latex cartilla.tex
</pre>

El archivo generado (cartilla.dvi) puede procesarse para producir un
PostScript con:
<pre>
dvips -tlegal -o cartilla.ps cartilla.dvi
</pre>

note que se específica tamaño ```legal``` para que ese sea el tamaño que quede en el PostScript. 

##Modificaciones al !PostScript

En el !PostScript generado deben reorganizarse las páginas del documento  con:
<pre>
        psbook -s12 cartilla.ps /tmp/cartilla-book.ps
</pre>
Lo cual genera ```/tmp/cartilla-book.ps``` presumiendo que la cartilla será de 12 páginas (i.e cabrán en 3 hojas, cada una por cara y cara y cada cara con 2 páginas de cartilla).   Si el documento procesado tienen más de 12 páginas, ```psbook``` generará varios librillos (en inglés "Signatures") cada una de 12 páginas ---pensando en
posteriormente coser cada cuadernillo y pegar todos sobre el lomo de un libro.

Después puede organizarse las páginas renumeradas en páginas tamaño oficio horizontales (de forma que vayan 2 páginas de las renumeradas en cada página del documento por generar):
<pre>
psnup -w8.5in -h33cm -W6.5in -H8.5in -2 /tmp/cartilla-book.ps \
      /tmp/cartilla-pregv.ps
</pre>

El paso anterior generará ```/tmp/cartilla-pregv.ps```, como el tamaño legal es diferente al tamaño oficio, debe subirse un poquito (85 puntos) el PostScript generado con:
<pre>
pstops "(0,85)" /tmp/cartilla-pregv.ps cartilla-gv.ps
</pre>
para producir cartilla-gv.ps.

Finalmente pueden separarse hojas pares (cara 1) de impares y reorganizar las impares para que no deban reorganizarse manualmente tras imprimir con:

<pre>
psselect -q -o -r cartilla-gv.ps cartilla-imp1.ps
psselect -q -e cartilla-gv.ps cartilla-imp2.ps
</pre>

Entonces se habrá generado un lado en ```cartilla-imp1.ps``` y el otro en ```cartilla-imp2.ps```.

##Referencias

* Lo central sobre modificación al PostScript se encontró en el estupendo Usenet: http://groups.google.com/group/comp.text.tex/msg/372c9bbc54a89b3d?q=ghostview+reorder&hl=en&lr=&ie=UTF-8&oe=UTF-8&rnum=2

------
Esta información se cede al dominio público y se dedica al padre que es JUSTICIA. 2006. vtamara@pasosdeJesus.org
