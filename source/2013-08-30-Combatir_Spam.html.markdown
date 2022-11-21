---
title: Combatir_Spam
date: 2013-08-30 12:00 UTC
tags:
---
En un sistema de correo debe combatirse el spam en dos frentes:

# Evitando que llegue spam a las carpetas de usuarios (excepto la carpeta Junk)
# Evitando que el correo proveniente de la organización sea clasificado como spam por otros servidores de correo

##1. Evitar que llegue spam a carpetas fuera de Junk

Aunque OpenBSD ofrece un servicio de greylisting con spamd, hemos notado que requiere esfuerzo continuo de parte de administradore para mantener al día.

Recomendamos SpamAssassin e invertir tiempo con periodicidad (pero menos que con el greylisting de spamd) para actualizar reglas de filtrado.

##2. Evitar ser clasificado como spammer

Entre las prácticas necesarias está:

* Tener registro MX y PTR correctos
* Tener registro SPF
* Asegurar que la red interna no envia correos con Spam (por ejemplo hay virús que hacen eso).
* Revisar servicios de listas negras y sacar si es reportado

! Registros MX y  PTR

Algunos sistemas anti-spam examinan el registro MX de un dominio, buscan la IP asociada, hacen resolución DNS inversa de la IP y esperan que de el mismo nombre MX (ver {1}).
     
Por esto es necesario asociar el mismo nombre asociado al registro MX como registro PTR de la IP.  Esto lo debe hacer el proveedor de Internet y el procedimiento de un provedor a otro cambia, y con un mismo proveedor suele cambiar en el tiempo.

El registro MX se configura en el DNS de su dominio, es algo como:

## 3. Bibliografía

* http://www.mxpolice.com/email-security/importance-of-ptr-records-for-reliable-mail-delivery/

Dedicado al todopoderoso que devuelve la vista al ciego. [Mr 8:22-26 | http://www.biblegateway.com/passage/?search=Marcos%208:22-26&version=RVR1960 ]
