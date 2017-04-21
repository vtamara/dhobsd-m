---
title: Pruebas del sistema con selenium-ide
date: 2017-04-21 02:34 UTC
tags:
---

Como parte del proceso de desarrollo ágil de una aplicación web
es importante entre otras:  (1) mantener requerimientos en forma
de historias de usuario y tareas, así como su seguimiento,
(2) que un equipo de desarrollo implemente los requerimientos 
y resuelva fallas, (3) que un equipo de pruebas busque fallas y
reporte

Las pruebas del sistema (https://en.wikipedia.org/wiki/System_testing) "se 
realizan a un sistema completo e integrado, para evaluar que cumpla con
los requerimientos especificados.   Las pruebas del sistema caen dentro
de la categoría de pruebas de caja negra y por tanto no deben requerir
conocer el diseño del código fuente o su lógica."

A continuación describimos como realizamos pruebas del sistema en
Pasos de Jesús usando Selenium-IDE, suponiendo que las fuentes de la 
aplicación (y de las pruebas) se almacenan en un repositorio de github.com 
y que el seguimiento al desarrollo se realiza en un tablero Trello.

# Tablero Trello

El tablero Trello sue suele tener columnas para:
* Historias de Usuarios (y detalles del proceso de desarrollo o contractuales) 
* Posibilidades de tareas por implmentar (donde se hace división de las 
  historias de usuario en tareas estimbales y se estima esfuerzo en horas).
* Haciendo (a donde se mueven tarjetas desde posibilidades de acuerdo
  a prioridades acordadas con el cliente o requeridas por el diseño
* Hecho, donde quedan tareas completas y fallas resueltas
* Comprometidas, donde quedan actividades de mantenimiento, soporte y logística

Solemos estimar que las pruebas del sistema requieren 1/4 del tiempo
de desarrollo y solemos dejar en el tablero en una columna "Comprometidas"
una tarjeta con título "P-1 Pruebas" donde se hace parte del seguimiento
de pruebas.

# Dinámica desarrollo/pruebas del sistema

De acuerdo a las prioridades acordadas con clientes (tipicamente en una reunión
al inicio de cada carrera) y del diseño de la carrera, el equipo 
de desarrollo debe buscar corregir fallas, implementar novedades, realizar 
otros tipos de pruebas (unidad, regresión),  poner en el repositorio 
github los cambios, desplegar cambios en sitio de desarrollo (y/o de
ensayo) y anunciar al equipo de pruebas un resumen de las fallas resueltas 
y de las novedades implementadas, esto suele hacerse con periodicidad 
semanal (o según el tipo de contrato).

Cada vez que el equipo de desarrollo anuncia cambios (suele ser
semanal), el equipo de pruebas:
* Verifica que siga funcionando lo que ya operaba
* Prueba las novedades buscando hacer fallar la aplicación
* Reportar en Trello (a más tarda  5 días después del anuncio
del equipo de desarrollo, si los anuncios de desarrollo son semanales
o si el tipo de contrato lo requiere hasta un día despueś).
* Actualiza pruebas del sistema en directorio test/seleniumide --el
  equipo de pruebas tiene a su cargo mantener al día este directorio
  con pruebas para Selenium-IDE que puedan reproducirse y funcionar
  (con excepciones que se mantienen en Trello)


# Pruebas del sistema

## A. VERIFICAR QUE LO QUE FUNCIONABA SIGUE FUNCIONANDO

1. Clonar o actualizar el repositorio que se va a probar.  
   Esto en particular actualizará el directorio `test/seleniumide` donde deben 
   estar casos de pruebas para las diversas funcionalidades del sistema
   con títulos que sugieran lo que se prueba y extensión `.selelnium`, así
   como una suit de pruebas (`pruebas-suit.selenium`) que agrupa todos los 
   casos de prueba.

2.  Empleando firefox con Selenium IDE instalado, ingresar a la aplicación 
    con un usuario y contraseña de administrador, iniciar Selenium IDE
    cargar la suit de pruebas y ejecutarla completa.

3. Por cada falla que se encuentre:
3.1 Reproducirla manualmente y asegurar que es una falla del sistema 
    (y no de las pruebas o de Selenium-IDE)
3.2 Si era una falla ya resuleta buscar la tarjeta donde se había reportado
    y pasarla de la columna "Hecho" del tablero Trelo a "Haciendo"
3.3 Si es una falla nueva, crear una nueva tarjeta Trello en la columna
    "Haciendo".  Iniciar el título con un código P-n donde n es un número
    consecutivo respecto a los de otras tarjetas.  Adjuntar a esta tarjeta
    prueba de Selenium-IDE que fallí, pantallazo y si es el caso
    detalles adicionales de como reproducir en comentarios.

4. Si se detienen las pruebas pero no por fallas en la aplicación sino en la 
   prueba para Selenium-ID o por cambios en funcionalidad de la aplicación
   pero sin errores, se mejora la prueba de Selenium-IDE para que pase 
   y se guarda y actualizan pruebas en github.

5. En la tarjeta P-1 del tablero Trello agregar un comentario  del estilo
   "Ejecutada suit de pruebas. 2 errores"
  

## B. PROBAR NOVEDADES

Por cada novedad o falla resuelta (no cubierta en casos de prueba
existentes):

1. Iniciar un nuevo caso de prueba en Selenium-IDE, guardar lo que se
   prueba.  Poner o renombrar para que el nombre corresponda a la
   funcionalidad que se prueba.

2. Referenciar en la tarjeta P-1 del tablero de Trello la novedad o falla 
   que está probando, en un comentario con la referencia a la novedad o
   falla probada (e.g Probadas F-12, R-16).

3. Si una falla supuestamente resuelta sigue fallando, devolver la tarjeta
   de la columna Hecho a Haciendo, agregar comentario, pantallazo y archivo
   para Selenium IDE para reproducirlo (no agregar a github casos de 
   prueba que fallan).

4. Si la prueba pasa agrega la prueba a la suit de pruebas para el sistema, 
   agrega el archivo al repositorio y actualizar en github.



