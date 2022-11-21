---
title: PHP_Ruby_Python_en_OpenBSD_5
date: 2012-01-23 12:00 UTC
tags:
---
El desarrollo de PHP no ha sido continuo, las fechas de publicación de algunas versiones mayores (con cambios grandes) recientes son:
* 5.0 13.Jul.2004
* 5.1 24.Nov.2005
* 5.2 2.Nov.2006
* 5.3 30.Jun.2009
* 5.4 1.Mar.2012
* 5.5 20.Jun.2013

Posiblemente por los problemas al migrar de una versión a otra las aplicaciones, el desarrollo de PHP estuvo algo quieto entre las versiones 5.2 - 5.3 y 5.3 - 5.4.   Posiblemente 5.5 es un intento de dar más vitalidad al desarrollo del lenguaje, tomando ideas entre otros de Ruby (como yield).

Por su parte las librerías PEAR, que sonarían como gran candidato para ser el entorno oficial, en parte por su caracter de Bazar no han sido actualizadas, y algunas de hecho no serán actualizadas para PHP 5.4, como Date, ver: http://pear.php.net/bugs/bug.php?id=20030

Esto muestra que cada vez será más díficil mantener aplicaciones que usen PEAR.  La versión del paquete principal ha quedado en 1.9.4 desde 2011.  PEAR2 que es el nuevo desarrollo para PHP 5.3 y siguientes no ha tenido acogida, no hay suficientes paquetes y los que hay no necesariamente son compatibles con los de PEAR.  Hay una discusión de 2011 que apunta en esta dirección: http://www.sitepoint.com/forums/showthread.php?730355-PEAR-Still-Relevant

La situación para adJ 5.4 ha sido:
* OpenBSD incluye PHP 5.2 y PHP 5.3, los paquetes para pear operan con PHP 5.3.
* Es posible compilar PHP 5.5 retroportado, pero no es posible instalar Pear (hay incompatilibad en versiones publicadas de Archive_Tar, ha sido resuelta en rama estable, el go-pear.php usa la versión de desarrollo, pero emplea condensado de forma que al cambiarse --para hacer menos manual la instalación-- no funciona de inmediato).
* Es posible compilar PHP 5.4 retroportado y usar Pear de OpenBSD 5.4, pero hay incompatibilidades que no serán resueltas (como en Date).  Al intentar correr SIVeL, en Consulta_WEB resulta:  Call to a member function handle() on a non-object in /pear/lib/HTML/QuickForm/Controller.php on line 141, y en la ficha de captura: upstream prematurely closed connection while reading response header from upstream.

Las opciones en el momento de este escrito son:

* Ayudar en el desarrollo de Pear2 que se ha migrado a github buscando más desarrolladores hasta lograr tener lo suficiente para migrar aplicaciones grandes basadas en Pear.
* Cambiar a un framework para PHP como Zend o como Synfony, que se han mantenido más a la par con el desarrollo del lenguaje.
* Cambiar a otro lenguaje y framework apropiados para escribir aplicaciones Web como Python con Django o como Ruby con Ruby on Rails.
** Con respecto a Python con Django es bastante utilizable en OpenBSD 5.x con Apache y el módulo de reescritura --aunque falta probar que funcione bien HTTPS.  No dejan de ocurrir fallas, especialmente con actualizaciones dado el rápido desarrollo de Python y Django.
** Ruby y Ruby on Rails tienen un desarrollo aún más rápido que Python, su plataforma objetivo ha sido principalmente Linux, aunque los portes para OpenBSD 5.3, 5.4, 5.5 han ido bastante a la par con el desarrollo (esfuerzo de Jeremy Evans).  

   

Las fechas de desrrollo de Ruby on Rails han sido:
* 2.3 16.Mar.2009
* 3.0 29.Ago.2010
* 3.1 31.Ago.2011
* 3.2 20.Ene.2012
* 4.0 25.Jun.2013

La situación para adJ 5.4 ha sido:

* OpenBSD 5.4 incluye Ruby 2.0.0.247
* Es directo retroportar Ruby 2.0.0.353
* Rails 4.0.2 (que es el estable más reciente en el momento de este escrito) opera de inmediato empleando instrucciones para 5.3
