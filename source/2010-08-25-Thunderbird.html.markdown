---
title: Thunderbird
date: 2010-08-25
tags:
---
##Instalación

Para Windows descarguelo de http://www.mozillamessaging.com/thunderbird/

En Ubuntu desde el gestor de paquetes synaptic busque el paquete Thunderbird
e instalelo.


##Configuración

El servidor entrante emplea el protocolo IMAP sus datos de configuración son:
* Servidor: correo.mofrapaz.org
* Encripción: Con cifrado SSL
* Puerto: 993
* Usuario: su usuario en el servidor (sin dominio @mofrapaz.org)

El servidor saliente emplea el protocolo SMTP con los siguientes datos de configuración:
* Servidor: correo.mofrapaz.org
* Encripción: Con cifrado SSL
* Puerto: 465
* Autenticación: requiere el mismo usuario y clave del correo entrante



|Advertenia: asegurese de ver "Servidor de correo IMAP" al examinar la configuración de la cuenta (si dice "Servidor de correo POP" debe eliminar la cuenta y volver a configurarla).|

Otros detalles de configuración que sugerimos:

* Emplear encabezados Antispam de SpamAssassin
* Eliminar el correo basura cada 15 días


## Diccionario en Español

Se trata de un complemento para Thunderbird:

# Descarguelo de https://addons.mozilla.org/en-US/thunderbird/browse/type:3
# Vaya a Herramientas->Complementos
# Eliga nuevo complemento y de la ruta donde descargó el diccionario.
# Reinicie Thunderbird
# Para configurar el idioma redacte un correo y desde herramientas eliga revisar ortografía. Idioma: Español/Colombia (o es_CO).
