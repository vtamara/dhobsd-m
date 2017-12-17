---
title: Pruebas del sistema con sideex
date: 2017-12-15 02:34 UTC
tags:
---

# Pruebas del sistema con sideex

Como parte del proceso de desarrollo ágil de una aplicación web
es importante entre otras:  (1) mantener requerimientos en forma
de historias de usuario y tareas, así como su seguimiento,
(2) que un equipo de desarrollo implemente los requerimientos 
y resuelva fallas, (3) que un equipo de pruebas busque fallas, las
encuentre y reporte al equipo de desarrollo.

Las pruebas del sistema  "se realizan a un sistema completo e integrado, 
para evaluar que cumpla con los requerimientos especificados.   
Las pruebas del sistema caen dentro de la categoría de pruebas de caja
negra y por tanto no deben requerir conocer el diseño del código fuente
o su lógica." (<https://en.wikipedia.org/wiki/System_testing>)

A continuación describimos como realizamos pruebas del sistema en
Pasos de Jesús usando sideex (http://sideex.org/), suponiendo que las fuentes de la 
aplicación (y de las pruebas) se almacenan en un repositorio de github (https://github.com)
y que el seguimiento al desarrollo se realiza en un tablero Trello (https://trello.com).

## 1. Tablero Trello

El tablero Trello suele tener columnas para:

- Historias de Usuarios. Así como detalles del proceso de desarrollo o contractuales

- Posibilidades de tareas por implementar. Donde se hace división de las 
  historias de usuario en tareas estimbales y se estima esfuerzo en horas.

- Haciendo. A donde se mueven tarjetas desde posibilidades de acuerdo
  a prioridades acordadas con el cliente o requeridas por el diseño

- Hecho. Donde quedan tareas completas y fallas resueltas

- Comprometidas. Donde quedan actividades de mantenimiento, soporte y logístico/administrativas

Solemos estimar que las pruebas del sistema requieren 1/4 del tiempo
de desarrollo y solemos dejar en el tablero en la columna "Comprometidas"
una tarjeta con título "P-1 Pruebas" donde se hace parte del seguimiento
de pruebas.

## 2. Dinámica desarrollo - pruebas del sistema

De acuerdo a las prioridades acordadas con clientes (tipicamente en una reunión
al inicio de cada carrera) y del diseño de la carrera, el equipo 
de desarrollo debe buscar corregir primero fallas en el código fuente, 
refactorizar, implementar novedades,  realizar otros tipos de pruebas 
(unidad, regresión), poner en el repositorio github los cambios, desplegar 
cambios en sitio de desarrollo (y/o de ensayo) y anunciar al equipo de pruebas 
un resumen de las fallas resueltas y de las novedades implementadas (eventualmente
con indicaciones sobre que aspectos concentrar las pruebas), esto suele hacerse
con periodicidad semanal (o según el tipo de contrato).

Cada vez que el equipo de desarrollo anuncia cambios, el equipo de pruebas:

- Verifica que siga funcionando lo que ya operaba
- Prueba las novedades buscando hacer fallar la aplicación
- Reporta en Trello (a más tardar  5 días después del anuncio
  del equipo de desarrollo, si los anuncios de desarrollo son semanales
  o si el tipo de contrato lo requiere hasta un día despueś).
- Actualiza pruebas del sistema en directorio ```test/sideex``` --el
  equipo de pruebas tiene a su cargo mantener al día este directorio
  con pruebas para sideex que puedan reproducirse y funcionar
  (con excepciones que se mantienen en Trello)

## 3. Pruebas del sistema

### 3.1 Verificar que lo que funcionaba siga funcionando

a. Clonar o actualizar el repositorio que se va a probar.  
   Esto en particular actualizará el directorio `test/sideex` donde deben 
   estar casos de pruebas para las diversas funcionalidades del sistema
   con títulos que sugieran lo que se prueba en una suit de pruebas 
   (`pruebas-suit.selenium`) que agrupa todos los  casos de prueba.

b.  Empleando Chrome con sideex instalado, ingresar a la aplicación en
    el sitio de desarrollo (o de ensayo) con un usuario y contraseña de administrador 
    (de prueba),  iniciar sideex, cargar la suit de pruebas y ejecutarla completa.

c. Por cada falla que se encuentre:

c.1 Reproducirla manualmente y asegurar que es una falla del sistema 
    (y no de las pruebas o de sideex)

c.2 Si era una falla ya resuleta buscar la tarjeta donde se había reportado
    y pasarla de la columna "Hecho" del tablero Trello a "Haciendo"

c.3 Si es una falla nueva, crear una nueva tarjeta Trello en la columna
    "Haciendo".  Iniciar el título con un código ```F-n``` donde n es un número
    consecutivo respecto a las demás tarjetas.  Adjuntar a esta tarjeta
    la prueba de Selenium-IDE que falló, un pantallazo y si es el caso
    detalles adicionales de como reproducir en comentarios.

c.4. Si se detienen las pruebas pero no por fallas en la aplicación sino en la 
   prueba para sideex o por cambios en funcionalidad de la aplicación
   pero sin errores, se mejora la prueba de sideex para que pase 
   y se guarda y actualizan pruebas en github.

c.5. En la tarjeta P-1 del tablero Trello agregar un comentario  del estilo
   "Ejecutada suit de pruebas. 2 errores"
   
c.6 Si hay dificultades que no logran superarse para describir un caso de prueba
    con sideex guardar el caso de prueba en un solo archivo en la carpeta
    test/sideex/con_error
  

### 3.2 Probar Novedades

Por cada novedad o falla resuelta que reporte el equipo de desarrollo:

1. Abrir sideex y la suit de pruebas e iniciar un nuevo caso de prueba, 
   guardar lo que se prueba.  Poner nombre o renombrar para que el nombre 
   corresponda a la funcionalidad que se prueba.   

2. Referenciar en la tarjeta P-1 del tablero de Trello la novedad o falla 
   que está probando, en un comentario con la referencia a la novedad o
   falla probada (e.g Probadas F-12, R-16).

3. Si una falla supuestamente resuelta sigue fallando, examinar posibles
   comentarios del equipo de desarrollo en la tarjeta y si es el caso devolver 
   la tarjeta de la columna Hecho a Haciendo, agregar comentario, pantallazo y archivo
   para sideex para reproducirlo (los casos de pruebas que fallan por 
   dificultades con sideex almacenar en `test/sideex/con_error`.

4. Si la prueba pasa agregar la prueba a la suit de pruebas con un nombre
   acorde a la prueba, agrega el archivo al repositorio y actualizar en github.


### 3.3 Ayudas para crear cada caso de prueba

Un caso de pruebas en sideex consta de una serie de ordenes selenese, 
cada orden selenese puede tener cero, uno o dos argumentos.  El primer 
argumento cuando existe típicamente es un selector del elemento al que se aplica 
la orden y el segundo cuando existe es un valor.  Por ejemplo la orden ```clickAt``` 
utiliza un selector para identificar el elemento sobre el cual realizar una 
pulsación del ratón pero no requiere valor.  La orden ```type``` requiere los 
dos argumentos, el selector que indica el elemento donde se escribirá y el 
valor que escribirá.

Al guardar un caso de prueba, se almacenan las ordenes en un formato de tabla 
HTML, donde cada fila es una orden y con 3 columnas: la primera para el nombre
del comando, la segunda para el selector y la tercera para el valor.

Las ayudas que se presentan a continuación son mínimas, se recomienda consultar
la ayuda de sideex (botón Help o disponible en 
http://www.seleniumhq.org/docs/02_selenium_ide.jsp ), así como la especificación de 
cada orden selenese disponible en la pestaña Reference de la interfaz de
Selenium-IDE cuando se enfoca la orden en un caso de prueba.

* En general suponer que ya se ha iniciado sesión en la aplicacioń con la cuenta
  administrativa de prueba y que está en la pantalla incial de la aplicación. 
* De requerirse crear nuevos elementos pero con nombres que no no puedan interferir
  con una aplicación de producción (en caso de ejecutarse sobre una).   Por 
  ejemplo nombres AAAA.
* En general cada caso de prueba debe eliminar los elementos que cree (excepión 
  si la suit de pruebas completa elimina con casos de prueba finales lo que 
  crearon los iniciales --digamos un usuario)
* Es bueno utilizar ordenes selenese assert (el más típico debe ser 
  ```assertElementPresent```) que verifiquen que en un momento dado de la prueba 
  el estado de la aplicación sea el esperado sin lugar a dudas.
* Aunque sideex se inspiró en Selenium-IDE, en general no requiere ordenes que 
  esperen la presencia de un element como ```waitForElementPresent```
