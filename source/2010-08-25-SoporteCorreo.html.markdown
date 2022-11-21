---
title: SoporteCorreo
date: 2010-08-25 12:00 UTC
tags:
---
! INTRODUCCIÓN

El servicio de correo electrónico en Internet requiere servidores donde se almacenen los correos, un programa en cada servidor que los almacene y programas para que un usuario envie correos o reciba.  Para el envio y consulta de correos los programas siguen protocolos.

Un usuario que envía un correo, inicia una sesión en su computador, abre un cliente de correo, redacta el correo y presiona un botón de enviar.
Al hacerlo el programa de correo debe analizar la dirección electrónica del destinatario e indagar el servidor de correo que debe recibirlo, una vez lo hace emplea el protocolo SMTP para enviar el correo y el servidor si está operando emplea el mismo protocolo para recibirlo.

Para revisar su correo un usuario debe emplear un programa que se conecté al servidor que almacena sus correoscon el protocolo POP o con el protocolo IMAP, enviar su usuario y clave y recibir los correos --con IMAP los correos quedan en el servidor mientras que con POP son trasmitidos al programa que los solicitó y después borrados.


##CORREO SEGURO?

Los puntos de vulnerabilidad son:

* El computador del usuario final y el programa que usa para leer correo.  Por ejemplo si un atacante logra controlar su computador empleando alguna vulnerabilidad de un programa.  Se recomienda OpenBSD pero por facilidad UbuntuCE actualizado.  Thunderbird o al navegar con Chrome o Firefox.
* El medio de transmisión de información y los puntos intermedios por donde pasa.  Es recomendable que el correo se envie encriptado, pero esto requiere que tanto el servidor que envia como el que recibe lo soporten.  En el caso de Mofrapaz esto se hace:
** Siempre ocurre cuando se envia de mofrapaz.org a mofrapaz.org pues no sale del servidor la información
** A servidores del SINCODH que tengan servidor de correo (pues pasosdeJesus está en el SINCODH): www.nocheyniebla.org, justapaz.org, sismamujer.org
** A servidores que soporten ESTMP con TLS, por ejemplo riseup.net, reiniciar.org
** La conexión a otros servidores ira desencriptada y podrá ser vista en computadores intermedios como el proveedor de Internet.
* El servidor que recibe o almacena el correo.  Por esto no se recomiendan servicios públicos como hotmail o gmail para información sensible.  El administrador de un servidor siempre puede examinar lo que se ha almacenado en el servidor.  Gmail, hotmail y diversos proveedores en Colombia no tendrían herramientas jurídicas para evitar inspecciones judiciales ni allanamientos.  En caso de robo del servidor, quien lo roba tendría acceso a la información.  En el caso de Mofrapaz la información se mantiene en un servidor de la organización y se mantiene encriptada con copia de respaldo --por el momento al mismo servidor, sacada cada día.  Si se explota remotamente una debilidad o vulnerabilidad del servidor pueden quedar expuestos correos.  En nuestro caso usamos el sistema con menos vulnerabilidades.



##CLIENTES DE CORREO

Hay gran variedad de clientes de correo, unos funcionan en el web desde un navegador y otros son programas.  Típicamente tienen una bandeja de entrada (eventualmente con carpetas) a la que llegan los correos, los cuales se pueden responder, eliminar o reenviar.

En paginas dedicadas, detallamos configuración de dos de ellos:
* [Thunderbird] que es un programa
* [Roundcubemail] que es un cliente web

