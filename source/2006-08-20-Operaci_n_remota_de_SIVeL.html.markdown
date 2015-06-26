---
title: Operaci_n_remota_de_SIVeL
date: 2006-08-20
tags:
---
Para iniciar la operación remota de Sivel se ingresa al cortafuegos,  en éste caso el de Compromiso, a través de dos formas de conexión:
# Protocolo ssh: conexiones típicas entre sistemas unix que transmiten información encriptada
# Protocolo ssl: conexiones de navegador a sitios web seguros (https), también las transmisiones son encriptadas.

Para iniciar la conexión ssh que después permita conectarse a la base:
<pre>
ssh -L10443:base:443 sivel@compromisoong.org
</pre>

donde ```base``` es el nombre del servidor al interior de la organización y 443 es el n?mero de puerto de HTTPS (protocolo HTTP sobre SSL, es decir conexiones web encriptadas).

Una vez se establece la conexión debe digitarse la clave del usuario ```sivel``` en el cortafuegos.

Con el comando anterior además de lograrse una conexión al cortafuegos, se inica un tunel entre el puerto 10443 de la máquina local y el puerte 443 del servidor ```base```

Para emplear el tunel, ingresar en un navegador (mozilla-firefox) la dirección:
<pre>
https://localhost:10443
</pre>

Tras aceptar el certificado del sitio web seguro, aparece la pantalla de autenticación de SIVeL, donde debe digitarse el nombre de usuario y la clave para ingresar al SIVeL.
