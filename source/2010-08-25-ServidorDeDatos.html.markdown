---
title: ServidorDeDatos
date: 2010-08-25 12:00 UTC
tags:
---
Se trata de un servidor conectado a Internet donde puede almacenar su información para revisarla dentro y fuera de la organización.  Muy apropiado para usar con dispositivos móviles (netbooks, celulares, iPhone, Black Berry) que tienen poco espacio para almacenar datos pero conectividad a Internet.

Es además el método que sugerimos para compartir información con otros
usuarios de la organización e incluso externos.   También puede almacenar su información privada pues los datos se mantienen en una partición encriptada y en el directorio correcto solo usted (bueno y el/la administrador@) tendrán acceso.

## Configuración inicial

Desde UbuntuCE vaya a Lugares->Conectar al Servidor.
* Servidor: www.mofrapaz.org
* Protocolo: ssh
* Puerto: 22
* Usuario: sucuentadecorreo
* Directorio: /home/sucuentadecorreo
* Añadir como marcador, por ejemplo como sucuenta-mofrapaz


##USO

Una vez lo haga tendrá un nuevo marcador con el que podrá abrir rápidamente la conexión sin volver a configurar (en caso de que desee eliminarlo, abra nautilus y con click derecho sobre el marcador tendrá la opción).

Cada vez que inicie una conexión abriendo el marcador, le pedirá su clave --la misma del correo-- y abrirá su carpeta personal en el servidor. Allí hay algunos enlaces que le permitirán realizar las operaciones más comunes.


! Publicar información en el web en espacio personal

Conéctese al servidor y elija la carpeta ```public_html```, allí puede poner el archivo que desea publicar.  Si por ejemplo usted es el usuario ```pablo``` y el archivo se llama ```video.avi``` después de publicar puede probar el URL:

 ``` http://www.mofrapaz.org/~pablo/video.avi ```

Si requiere enviar un adjunto muy grande para el correo, puede publicarlo en su espacio web personal y enviar el enlace al destinatario para que lo descargue.   

! Poner información privada para otro usuario de Mofrapaz

Un método sencillo es enviarle un correo electrónico pues los que provienen de ```mofrapaz.org``` con destinatario en ```mofrapaz.org``` no salen del servidor y se mantienen en la partición encriptada.  

En todo caso, también puede conectarse al servidor y navegar por los directorios hasta llegar a ```/var/maildir``` allí podrá elegir el usuario al cual desea ponerle un archivo, ingresar a su directorio ```recibe ```, aunque no pueda ver los archivos disponibles allí, podrá poner el que desea compartirle y avisarle.  

! Poner información privada para compartir con todos los usuarios de Mofrapaz

Conéctese al servidor y entre a la carpeta ```compartidoenc```  (o navegue por los directorios hasta llegar a ```/var/maildir/compartido```)  dejé allí el archivo por compartir y avise al grupo.

! Recibir información enviada por otros usuarios

Si la enviaron a todo el grupo debe estar en la carpeta ```compartidoenc```.
Si se la enviaron sólo a usted ingrese a la carpeta ```enc``` y allí
a ```recibidos``` donde debe encontrar la información que le enviaron.  Otras carpetas de su espacio a las que sólo puede ingresar usted y el adminsitrador son: ```maildir``` y ```privado``` (la primera tiene sus correos electrónicos).


! Almacenar información privada

Conéctese al servidor e ingrese a la carpeta ```enc``` desde  esta pase a la carpeta ```privado``` donde sólo usted, (y el adminsitrador) podrá escribir
y leer.  Si requiere más privacidad encripte directamente en su computador la información por ejemplo con !TrueCrypt y úbiquela en el servidor para usarla remotamente.
 

## Sobre el espacio

Borrar lo innecesario pues solo son 12G para correo y datos. 

## Recomendaciones hardware

Preferimos estrategia de Google: no invertir en costosos servidores sino en distribuir proceso en varios computadores de buena calidad pero económicos y fáciles de actualizar/remplazar.

* Inmediato plazo: Adquirir otro disco duro SATA de 1T y al menos 1G más de memoria RAM
* Mediano plazo: Adquirir otro servidor para dividir funcionalidad entre cortafuegos y servidor
* Largo plazo: Replicar cortafuegos y replicar servidor para distribuir carga.

