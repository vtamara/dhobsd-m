---
title: man_8_isakmpd
date: 2010-12-29 12:00 UTC
tags:
---
##NOMBRE
     ```isakmpd``` - ISAKMP/Oakley también conocido como servicio para adminsitración de llaves IKEv1

##RESUMEN
<pre>
     isakmpd ![-46adKLnSTv] ![-c arch-config] ![-D clase=nivel] ![-f fifo]
             ![-i arch-pid] ![-l arch-bitacoradepaquetes] ![-N puerto-encapudp]
             ![-p puerto-escucha] ![-R arch-reporte]
</pre>

##DESCRIPCIÓN

El servicio ```isakmpd``` establece asociaciones de seguridad para tráfico encriptado y/o autenticado.  En el momento, y probablemente por siempre, esto significa tráfico [ipsec(4) | man 4 ipsec].  Tradicionalmente, ```isakmpd``` se configuraba usando el formato de archivo ```[isakmpd.conf(5)|man 5 isakmpd.conf]```.  Un formato nuevo y mucho más simple ahora está disponible: [ipsec.conf(5) | man 5 ipsec.conf].

```isakmpd``` implementa el protocolo IKEv1 que se define en los estándares ISAKMP/Oakley (RFC 2408), IKE (RFC 2409), y el Internet DOI (RFC 2407). El protocolo más nuevo IKEv2, como se define en el RFC 4306, no es soportado por ```isakmpd``` sino por ```iked(8)```. Por consiguiente las referencias a IKE en este documento son con respecto a IKEv1 y no a IKEv2.

La forma en la que ```isakmpd``` opera es manteniendo una configuración interna así como una base de datos de políticas que describe que clases de SAs se negocia, y escucha esperando diversos eventos que disparen esas negociaciones.  Los eventos que controlan a ```isakmpd``` son señales, llamados desde el kernel via un socket ```PF_KEY```, y finalmente por eventos programados y disparados por temporizadores que cumplen sus tiempos.

El principal uso de ```isakmpd``` es implementar "redes virtuales privadas" (Virtual Private Networks - VPN).  La habilidad para proveer redundancia se hace disponible por intermedio de ```carp(4)``` y ```sasyncd(8)```.   Para otros usos, se requiere más conocimiento del protocolo IKEv1.  Los RFCs mencionados más adelante son posibles puntos de inicio.

En el arranque ```isakmpd``` se divide en dos procesos por separación de privilegios.  El hijo sin privilegios se encierra a si mismo con ```chroot(8)``` a ```/var/empty```.  El proceso privilegiado se comunica con el hijo, lee archivos de configuración  e información de PKI, y se ata a puertos privilegiados por este.  Ver la sección OBJECIONES más adelante.

Las opciones son:

```-4 | -6```
             Estas opciones controlan que familias de direcciones usará ```isakmpd``` (```AF_INET``` y/o ```AF_INET6```).  Por defecto usa ambas IPv4 e IPv6.

```-a```
             Si se da, ```isakmpd``` no establece flujos automáticamente por defecto.  En lugar de esto los flujos pueden configurarse manualment usando ```ipsec.conf(5)``` o con programas como ```bgpd(8)```.  Así que ```isakmpd``` solo se encarga de establecer SAs.

```-c arch-config```
             Si se da, la opción, ```-c``` especifica un archivo de configuración alterno a ```/etc/isakmpd/isakmpd.conf```.  Como este archivo puede contener información sensitiva, debe ser leible sólo por el usuario que ejecuta el servicio.  ```isakmpd``` releerá el archivo de configuración cuando se le envía la seña SIGHUP.

             Note que esta opción sólo aplica a archivos de configuración en el formato de ```[isakmpd.conf(5) | man 5 isakmpd.conf]```, y no a los del formato ```[ipsec.conf(5) | man 5 ipsec.conf]```.

```-D clase=nivel```
             Clase de depuración. Es posible especificar este argumento muchas veces.  Recibe un parámetro de la forma ```clase=nivel```, donde tanto  ```clase``` como ```nivel``` son números. ```clase``` denota una clase de depuración y ```nivel``` el nivel al cual desea que esa clase limite la impresión de mensajes de depuración (i.e. todas las impresiones de depuración por encima del nivel especificado no se mostrarán).  Si la clase se deja en `A', todas las clases de depuración se dejan en el nivel especificado.

             Los valores válidas para ```clase``` son:
|0 | Miscelanea |
|1  |Transporte |
|2  |Mensajes |
|3  |Criptografía |
|4  |Temporizadores|
|5  | Dependiente del sistema |
|6  | SA |
|7  | Intercambios |
|8  | Negociaciones |
|9  | Politícas |
|10 | Interfaz de usuario FIFO |
|A |  Todos |

             Los niveles que actualmente pueden usarse están entre 0 y 99.

```-d```
             La opción ```-d``` se usa para hacer que el servicio se ejecute en primer plano enviando errores a sálida estándar.

```-f fifo```
             La opción ```-f``` especifíca el FIFO (también conocido como tubería con nombre) donde el servicio escucha las peticiones de usuario.  Si la ruta dada es un guión (`-'), ```isakmpd``` escuchará por entrada estándar.

```-i arch-pid```
             Por defecto el PID (identificador de proceso) del servicio se escribirá en ```/var/run/isakmpd.pid```.  Esta ruta puede cambiarse al especificar otra como argumento a la opción ```-i```. Note que sólo se permiten rutas que comiencen con ```/var/run```.

```-K```
             Al especificar esta opción ```isakmpd``` no lee el archivo de configuración de políticas y no se efecutan chequeos de ```keynote(4)```.  Esta opción puede usarse cuando las políticas para flujos y establecimiento de SAs se realiza con otros programas como [ipsecctl(8) | man 8 ipsecctl] o ```bgpd(8)```.

```-L```
             Habilita captura de paquetes IKE. Cuando se da esta opción ```isakmpd``` capturará a un archivo una copia no encriptada de la negociación de paquetes que está enviando y recibiendo.  Este archivo después puede ser leido por ```tcpdump(8)``` y otras utilidades que usan ```pcap(3)```.

```-l arch-bitacoradepaquetes```
             Como la opción anterior ```-L```, pero captura al archivo especificado.  Note que sólo se permiten rutas que comiencen con ```/var/run```.

```-N puerto-encapudp```
             La opción ```-N``` especifica el puerto al cual el servicio se atará para esperar tráfico UDP encapsulado.

```-n```
             Cuando la opción ```-n``` es dada, el kernel no tomará parte en las negociaciones.  Este es un modo no destructivo, por así decirlo, en el sentido que no alterará SAs en la pila de IPsec.

```-p puerto-escucha```
             La opción -p especifica el puerto al cual el servicio se atará para escuchar.

```-R arch-reporte```
             Cuando envíe la señal ```SIGUSR1``` a ```isakmpd```, este reportará su estado interno a un archivo, normalmente ```/var/run/isakmpd.report```, pero esto puede cambiarse al dar un nombre de archivo como argumento a la bandera ```-R```.  Note que sólo se permiten rutas que comiencen con ```/var/run```.

```-S```
             Esta opción se emplea en configuraciones donde se usa ```sasyncd(8)``` y ```carp(4)``` para proveer redundancia. ```isakmpd``` comienza en modo pasivo y no iniciará conexión alguna ni procesará tráfico hasta que ```    sasyncd``` haya determinado que el computador es el maestro carp. Además ```isakmpd``` no borrará SAs al apagarse enviando mensajes de eliminación a todos los pares.

```-T```
             Cuando esta opción se da, se deshabilita el cruce-NAT e ```isakmpd``` no promocionará soporte de cruce-NAT a sus pares.

```-v```
             Habilita registros verbosos en bitacoras, ```isakmpd``` es silencioso y emite mensajes sólo cuando ocurre un error o una advertencia. Con registros verbosos ```isakmpd``` reporta completaciones exitosas de la fase 1 (Principal y Agreviso) e intercambios de fase 2 (Rápida) --la información e intercambios de transacciones no generan información de estado adicional.

!LA INTERFAZ DE USUARIO FIFO
     Cuando ```isakmpd``` comienza, crea un FIFO (tubería con nombre) donde escucha requerimientos de usuario.  Todos los comandos comienzan con una sóla letra, seguida de opciones específicas del comando.  Los comandos disponibles son:

| ```C add``` ![sección]:etiqueta=valor |
| ```C rmv``` ![sección]:etiqueta=valor |
| ```C rm``` ![sección]:etiqueta |
| ```C rms``` ![sección] |
| ```C set``` ![sección]:etiqueta=valor |
| ```C set``` ![sección]:etiqueta=valor ```force``` |

             Que actualizan la configuración del ```isakmpd``` en ejecución de forma atómica. `set' establece un valor de configuración que consta de la tripleta sección, etiqueta y valor. `set' fracasará si la configuración ya contiene una sección con la etiqueta nombrada; use la opción `force' para cambiar este comportamiento. `add' añade un valor de configuración a la lista de la etiqueta de configuración dada, a menos que el valor ya esté en la lista.  `rm' elimina una etiqueta de una sección.  `rms' elimina una sección entera.  `rmv' elimina una entrada de la lista, dando reversa así a una operación `add'.

             NOTA: Enviar a ```isakmpd``` una señal ```SIGHUP``` o una "R" por el FIFO hace nulas las actualizaciones realizadas a la configuración.

| ```C get``` ![sección]:etiqueta |

             adquiere el valor de configuración de la sección y etiqueta especificada.  El resultado es almacenado en ```/var/run/isakmpd.result```.

|  ```c <nombre>``` |

             Inicia una conexión con nombre, si está detenida o inactiva.

|  ```D``` <clase> <nivel> |
|  ```D A``` <nivel> |
|     ```D T```     |

             Pone la clase de depuración <clase> al nivel <nivel>.  Si la <clase> se especifica como `A', el nivel se aplica a todas las clases de depuración.  ```D T``` cambia el nivel de todas las clase de depuración a zero.  Otro comando ```D T``` las volverá a cambiar a sus estados previos.

|   ```d``` <galletas> <idmens> |

             Elimina la SA especificada del sistema.  Especifique <idmens> como `-' para concordar con una SA de fase 1.

|    ```M active```  |
|    ```M passive``` |

             Pone ```isakmpd``` en modo activo o pasivo.  En modo pasivo no se envían paquetes a los pares.

|     ```p on```![=<ruta>] |
|     ```p off```   |

             Habilita o deshabilita captura de paquetes IKE como textos claros.  Al habilitarla opcionalmente especificar a que archivo debe ```isakmpd``` enviar los paquetes.

|     ```Q```       |

             Termina limpiamente el servicio, como cuando se envía la señal SIGTERM.

|     ```R```       |

             Reinicializa ```isakmpd```, como cuando se envía la señal ```SIGHUP```.

|     ```r```       |

             Reporta el estado interno a un archivo. Ver la opción ```-R```.  Lo mismo que cuando se envía una señal ```SIGUSR1```.

|     ```S``` |

             Reporta información de todas las SAs conocidas en el archivo ```/var/run/isakmpd.result```.

|     ```T```       |

             Tumba todas la conexiones de modo rápido.

|     ```t``` ![<fase>] <nombre> |

             Tumba la conexión con nombre, si está activa.  Como <nombre> puede usarse la etiqueta especificada en ```[isakmpd.conf(5) | man 5 isakmpd.conf]``` o la dirección IP del computador remoto.  El parametro opcional <fase> especifica si debe borrarse una SA de fase 1 o de fase 2.  El valor `main' indica una conexión de fase 1; el valor `quick' una conexión de fase 2.  Si no se especifica fase, se asume `quick'.

!ESTABLECIENDO UNA INFRAESTRUCUTURA PÚBLICA DE LLAVES IKE (PUBLIC KEY INFRASTRUCTURE - PKI)

     Para usar autenticación basada en llaves públicas, debe haber una infraestructura administrando el firmado de llaves.  Bien ya hay una PKI existente del cual ```isakmpd``` debe formar parte, o habrá necesidad de iniciar una.  El procedimiento para usar una PKI pre-existente varia dependiendo de la Autoridad Certificadora (Certificate Autorihty - CA)  actual, y por tanto no se cubre aquí, excepto que mencionamos que debe usarse ```openssl(1)``` para craer un Requerimiento de Certificado de Firmas (Certificate Signing Request - CSR) que la CA entienda.

     Existen una diversidad de métodos para permitir la autenticación:

* Frase clave (Passprhase):  Este método no usa llaves del todo, pero depende de una frase clave compartida..
* Llaves de máquinas (Host keys):  Llaves públicas se usan para autenticar.  Ver a continuación AUTENTICACIÓN DE LLAVES PÚBLICAS.
* Certificados X509: Los Certificados X509 se usan para autenticar.  Ver AUTENTICACIÓN X509 más adelante.
* Certificados Keynote:  Los certificados Keynote se usan para autenticar. Ver AUTENTICACIÓN KEYNOTE más adelante.

     Cuando se configura ```isakmpd``` para autenticación basada en llaves y en certificados, la etiqueta ````Transforms```' en ```[isakmpd.conf(5) | man 5 isakmpd.conf]``` debe incluir ````RSA_SIG```'. Por ejemplo, la transformación ````3DES-SHA-RSA_SIG```' significa: encripción 3DES, condensando SHA, autenticación usando llaves RSA.

!AUTENTICACIÓN DE LLAVE PÚBLICA

     Es posible almacenar llaves públicas confiables para hacerlas utilizables directamente por ```isakmpd``` y obviar la necesidad de usar certificados. Las llaves deben ser salvadas en formato PEM (ver openssl(1)) y deben ser nombradas y almacenadas bajo la siguiente fórmula simple:

* Para identidades IPv4:    /etc/isakmpd/pubkeys/ipv4/A.B.C.D
* Para identidades IPv6:    /etc/isakmpd/pubkeys/ipv6/abcd:abcd::ab:bc
* Para identidades FQDN:    /etc/isakmpd/pubkeys/fqdn/foo.bar.org
* Para identidades UFQDN:   /etc/isakmpd/pubkeys/ufqdn/user@foo.bar.org

     Dependiendo de la campo ```ID-type``` de ```[isakmpd.conf| man 5 isakmpd.conf]```(5), las llaves pueden ser nombradas por su dirección IPv4 (IPV4_ADDR o ##IPV4_ADDR_SUBNET), las direcciones IPv6 (IPV6_ADDR o ##IPV6_ADDR_SUBNET), nombre de dominio plenamente calificado (fully qualified domain name - FDQN), nombre de usuario y dominio plenamente calificados (USER_FQDN), o ID de llave (```KEY_ID```).

     Por ejemplo, ```isakmpd``` puede autenticar usando llaves pre-generadas si la llave pública local, por defecto ```/etc/isakmpd/local.pub```, se copia a la compuerta remota como ```/etc/isakmpd/pubkeys/ipv4/dir.ip.compuerta.local``` y la llave pública de la compuerta remota es copiada a la compuerta local como ```/etc/isakmpd/pubkeys/ipv4/dir.ip.compuerta.remota```  Por supuesto, que pueden generarse nuevas llaves (al usuario no se le exige utilizar las llaves pregeneradas).  En este ejemplo, ```ID-type``` también debería establecerse en IPV4_ADDR o #IPV4_ADDR_SUBNET en ```[isakmpd.conf(5)| man 5 isakmpd.conf]```.

!AUTENTICACIÓN X509
     X509 es un marco para certificados de llaves públicas.  Los certificados pueden generarse con ```openssl(1)``` y proveen un medio para autenticación PKI.  En el siguiente ejemplo, una CA es creada junto con certificados de máquina para ser firmados por la CA.

     1.   Cree su propia Autoridad Certificadora (CA)

          Cree un certificado raíz autofirmado.  El certificado CA debe llamarse ```ca.crt```, y su llave privada ```ca.key```:

<pre>
                # openssl req -x509 -days 365 -newkey rsa:1024 \
                        -keyout /etc/ssl/private/ca.key \
                        -out /etc/ssl/ca.crt
</pre>
          ```openssl req``` solicitará la información que incorporará a la solicitud del certificado.  La información ingresada comprende un Nombre Distinguido (Distinguished Name - DN).  Son pocos campos, pero algunos pueden dejarse en blanco.  Para algunas campos habrá un valor por defecto; si se ingresa `.' el campo quedará en blanco.

     2.   Crear Requerimientos de Firma de Certificados (Certificate Signing Requests - CSRs) para los pares IKE.  Los CSRs son firmados con una llave privada pre-generada.

          Este paso, así como el siguiente, necesita realizarse para cada par.  Además el último paso necesita realizarse para cada ID que su par quiere tener.  El 10.0.0.1 siguiente simboliza ese ID, en este caso un ID IPv4, y debe ser cambiado en cada invocación.  Se le pedirá un DN para cada ejecución. Se recomienda codificar el nombre común en la ID, y debe ser único.
<pre>
                # openssl req -new -key /etc/isakmpd/private/local.key \
                        -out /etc/isakmpd/private/10.0.0.1.csr
</pre>
          Ahora tome estos requerimientos de firma de certificado a su CA y proceselos como se muestra a continución.  Un campo extensión ```subjectAltName``` debe agregar al certificado.  Remplace 10.0.0.1 con la dirección IP que ```isakmpd``` usará como identidad del certificado.
<pre>
                # env CERTIP=10.0.0.1 openssl x509 -req \
                        -days 365 -in 10.0.0.1.csr \
                        -CA /etc/ssl/ca.crt -CAkey /etc/ssl/private/ca.key \
                        -CAcreateserial -extfile /etc/ssl/x509v3.cnf \
                        -extensions x509v3_IPAddr -out 10.0.0.1.crt
</pre>
          Para un certificado FQDN,  ejecute:
<pre>
                # env CERTFQDN=somehost.somedomain openssl x509 -req \
                        -days 365 -in somehost.somedomain.csr \
                        -CA /etc/ssl/ca.crt -CAkey /etc/ssl/private/ca.key \
                        -CAcreateserial -extfile /etc/ssl/x509v3.cnf \
                        -extensions x509v3_FQDN -out somehost.somedomain.crt
</pre>
          Si ```CERTFQDN``` está siendo usado, asegurese que el campo  ```subjectAltName``` del certificado se especifique usando ```srcid``` en ```[ipsec.conf(5)|man 5 ipsec.conf].  Una configuración similar se requiere si  ```[isakmpd.conf(5)|man 5 isakmpd.conf] se usa en su lugar.

          Ponga el certificado (el archivo terminado en ```.crt```) en ```/etc/isakmpd/certs/``` en su sistema local. También lleve el certificado CA ```/etc/ssl/ca.crt``` y pongalo en ```/etc/isakmpd/ca/```.

     Para revocar certificados, cree un archivo Lista de Revocación de Certificados (CRL) e instalelo en el directorio ```/etc/isakmpd/crls/``` Ver ```openssl(1)``` y el subcomando `crl' para más información.

!AUTENTICACIÓN KEYNOTE
     Keynote es un marco para administración de confianzas.  Las llaves puede generarse usando ```keynote(1)``` y proveer medios alternativos de autenticación a ```isakmpd```.
     Ver ```keynote(4)``` para más información.

##ARCHIVOS
     ```/etc/isakmpd/ca/```
             El directorio donde se mantienen certificados CA.

     ```/etc/isakmpd/certs/```
             El directorio donde se mantienen certificados IKE, tanto certificado(s) local como los de los pares, si se ha elegido mantenerlos permanentemente.

     ```/etc/isakmpd/crls/```
             El directorio donde se mantienen los CRLs.

     ```/etc/isakmpd/isakmpd.conf```
             El archivo de configuración.  Como este archivo contiene información sensitiva no debe ser leible por nadie más que el usuario que ejecuta ```isakmpd```.

     ```/etc/isakmpd/isakmpd.policy```
             El archivo de configuración de políticas de keynote.  Los mismos requerimientos de permisos que ```isakmpd.conf```.

     ```/etc/isakmpd/keynote/```
             El directorio donde se mantienen credenciales de !KeyNote.

     ```/etc/isakmpd/private/```
             El directorio donde se mantienen llaves locales privadas usadas para autenticación de llave pública.  Por defecto, el archivo de comandos de arranque del sistema ```rc(8)``` genera un par de llaves cuando comienza, si no existen.  El par de llaves está en ```local.key```, y una copia de la llave pública, apropiada para transferir a otros computadores se extrae en  ```/etc/isakmpd/local.pub```.  Debe haber un certificado para ```local.key``` en el directorio de certificados ```/etc/isakmpd/certs/```.  ```local.key``` tiene los mismos requerimientos de permisos que ```isakmpd.conf```.

     ```/etc/isakmpd/pubkeys/```
             El directorio donde se mantienen llaves públicas confiables.  Las llaves deben nombrarse de la forma descrita antes.

     ```/var/run/isakmpd.fifo```
             El FIFO usado para controlar manualmente ```isakmpd```.

     ```/var/run/isakmpd.pcap```
             El archivo por defecto para captura de paquetes IKE.

     ```/var/run/isakmpd.pid```
             El PID del servicio actual.

     ```/var/run/isakmpd.report```
             El archivo reporte que se escribe cuando se recibe la señal SIGUSR1.

     ```/var/run/isakmpd.result```
             El archivo reporte escrito cuando se envia el comando `S' o el comando `C get' al FIFO de comandos.


##VER TAMBIÉN
     ```openssl(1)```, ```getnameinfo(3)```, ```pcap(3)```, ```[ipsec(4) |man 4 ipsec]```, ```[ipsec.conf(5)|man 5 ipsec.conf]```,
     ```[isakmpd.conf(5)|man 5 isakmpd.conf]```, ```[isakmpd.policy(5)| man 5 isakmpd.policy]```, ```[iked(8)|man 8 iked]```, ```sasyncd(8)```, ```ssl(8)```,
     ```tcpdump(8)```

##HISTORIA
     El protocolo para manejo de llaves ISAKMP/Oakley se describe en RFC 2407, RFC 2408, y RFC 2409.  NAT-Traversal se describe en RFC 3947.  Esta implementación fue realizada en 1998 por Niklas Hallqvist y Niels Provos, financiados por Ericsson Radio Systems.

##OBJECIONES
     Cuando se almacena una llave pública confiable para una identidad IPv6, la forma más eficiente de representar direcciones, i.e. "::" en lugar de ":0:0:0:", debe usarse o el proceso de correspondencia fallará.  ```isakmpd``` usa la salida de ```getnameinfo(3)``` para las traducciones dirección-a-nombre. El proceso privilegiado sólo permite conexiones al puerto por defecto 500 o a puertos no privilegiados (>1024).  No es posible cambiar las interfaces a las que ```isakmpd``` escucha sin un reinicio.

     Para configuraciones redundantes,```sasyncd(8)``` debe reiniciarse manualmente cada vez que se reinicia ```isakmpd```.

OpenBSD 4.8                      Junio 7, 2010                      OpenBSD 4.8


