---
title: Infraestructura_criptogr_fica_de_OpenBSD
date: 2011-06-09
tags:
---
El OCF (OpenBSD cryptographic Framework) como se describe en {7} es una capa de rutinas criptográficas implementada en el kernel que provee acceso uniforme a tarjetas criptográficas aceleradoras por medio de un API bien diseñado.   Puede ser usada por otras partes del kernel y a nivel de usuario con algoritmos simétricos y asimétricos.  Ofrece balanceo de carga, migración de sesión y encadenamiento de algoritmos.   Puede emplear hasta el 95% del desempeño máximo de tarjetas aceleradoras.

!API para el kernel

El API descrito en {7} para OpenBSD 4.8 consta de:

<pre>
    int32 t crypto_get_driverid();
    int crypto_register();
    int crypto_kregister();
    int crypto_unregister();
</pre>
    Usado por controladores de dispositivos para registrar y desregistrar soporte de algoritmos simétrico y asimétricos con el OCF.
<pre>
    void crypto_done();
    void crypto_kdone();
</pre>
   Llamado por controladores de dispositivos al completar requerimientos (simétricos y asimétricos respectivamente).

<pre>
    int crypto_newsession();
</pre>
    Llamado por consumidores de servicios criptográficos (tales como la pila IPsec) que quieren establecer una nueva sesión con la infraestructura. En caso de éxito, el primero argumento contendrá el Identificador de Sesión (SID - Session Identifier).  El segundo argumento contiene toda la información necesaria para que el controlador establezca la sesión (llaves, algoritmos, desplazamientos, etc).  El tercer argumento indica si sólo es aceptable acelaración por hardware.
 
<pre>
    int crypto_freesession();
</pre>
    Llamado para terminar una sesión previamente establecida.
 
<pre>
    int crypto_dispatch();
</pre>
    Llamada para procesar un requerimiento, encapsulado en su argumento. Los campos en esa estructura contienen:
* El SID
* La longitud total en bytes del colchon (buffer) por procesar,
* La longitud total del resultado, el cual para operaciones criptográficas simétricas será la misma longitud de la entrada.
* El tipo de colchon, como se usa con la rutina ```malloc()``` del kernel. Será usado si la infraestructura necesita localizar un nuevo bufer para el resultado ( o para reformatear la entrada).
* La rutina que el OCF deberá llamar al completar el requerimiento, bien con éxito o no.
* El tipo de error, si se encuentra alguno. Si se retorna el código de error EAGAIN, el SID ha cambiado.  El consumidor debe guardar el nuevo SID y usarlo en todos los requerimientos subsiguientes.  En este caso, el requerimiento puede ser reenviado de inmediato.  Este mecanismo es usado por la infraestructura para realizar migración de sesión (mover una sesión de un controlador a otro, por disponibilidad, desempeño u otra consideración).
* Un mascara de bits de banderas asociadas con este requerimiento.  
*  Los colchones de entrada y salida.  El colchón de entra puede ser una cadena ```mbuf``` o un colchon contiguo (como lo identifiquen las banderas).  El colchón de salida será del mismo tipo.
* Un apuntador a datos opacos. Es pasado por la infraestructura criptog&#341;afica sin alterar y se espera que lo use la aplicación llamadora.
* Una lista encadenada de descriptores de operaciones, la cual indica que operaciones deben aplicarse, y en que secuencia, a los datos de entrada. Los descriptores indican donde debe comenzar cada operación, la longitud de los datos por procesar, donde deben ponerse los resultados en el colchón de salida, las llaves por usar y varias banderas específicas por operación (e.g. que vector de inicialización emplear para cifrado en modo CBC).
 
<pre>
    int crypto_kdispatch();
</pre>
    Similar a ```crypto_dispatch()``` para operaciones de llave pública.


Los controladores del kernel que se registran como proveedores de estas funciones son:
| __Archivo__ | __Descripción__ |
| arch/amd64/amd64/via.c | Procesadores VIA  con funciones criptográficas |
| arch/amd64/amd64/aesni.c | Funciones criptográficas Intel |
| arch/i386/i386/via.c | Procesadores VIA  con funciones criptográficas |
| arch/i386/pci/glxsb.c | Bloque de seguridad en procesadores AMD Geode LX |
| dev/pci/hifn7751.c | Invertex AEON, Hifn {7751,7811,7951,7955,7956}, !PowerCrypt,  !PowerCrypt 5x, XL-Crypt, !NetSec 7751, Soekris Engineering {vpn1201,vpn1211,vpn1401,vpn1411} hifn(4)|
| dev/pci/lofn.c |  lofn(4)|
| dev/pci/noct.c |  Acelerador Netoctave NSP2000 noct(4)|
| dev/pci/nofn.c |  nofn(4)|
| dev/pci/safe.c | Aceleradores !SafeNet {1141,1741} safe(4)|
| dev/pci/ubsec.c | Aceleradores Bluesteel {5501,5601}, Broadcom {BCM5801, BCM5802, BCM5805, BCM5820, BCM5821, BCM5822, BCM5823, BCM5825, BCM5860, BCM5861, BCM5862}. ubsec(4) |


Los consumidores de estos servicios en el kernel son:

| __Archivo__ | __Descripción__ |
|dev/softraid_crypto.c | RAID por software. softraid(4) |
|netinet/ip_ah.c | IPsec AH. ipsec(4) |
|netinet/ip_esp.c | IPsec ESP. ipsec(4) |
|netinet/ip_ipcomp.c | IPcomp, compresión IP |


Nota: Otras partes del kernel no emplean el API descrito pero si funciones de encripción como ```uvm/uvm_swap_encrypt``` que puede encriptar el área de intercambio dependiendo de la configuración de sysctl  y sus variables      vm.swapencrypt.enable, vm.swapencrypt.keyscreated, vm.swapencrypt.keysdeleted.



!API a nivel de usuario

Se describe brevemente en [crypto(4) | man 4 crypto], con un poco más de detalle en {7} y aún más explicito en {8}. Su aplicación puede estudiarse en libssl o con más brevedad en algunas pruebas de regresión:

./regress/sys/crypto/aes/aestest.c
./regress/sys/crypto/aesctr/aesctr.c
./regress/sys/crypto/aesxts/aes_xts.c
./regress/sys/crypto/auth/md5.c
./regress/sys/crypto/enc/des3.c

El dispositivo ```/dev/crypto``` puede emplarse mientras esté activado el estado del kernel ```kern.usercrypto``` con ```sysctl```. Cuando está activo por defecto sólo permite lectura al usuario ```root```, por lo que las aplicaciones deben ejecutarse como este usuario.

En caso de que no cuente con equipo criptográfico puede emplear rutinas implementadas por software en el kernel si se ha activado el estado del kernel ```kern.cryptodevallowsoft``` con sysctl.  La función activa_cript() del programa presentado más adelante intenta activar para el superusuario el API criptográfico del kernel, así como las rutinas por software. 

Como se ve en [crypto(4) | man 4 crypto] y mejor en {8} los servicios que pueden usarse con ```/dev/crypto``` incluyen:
* Privacidad: Encripción/desencripción simétrica con 3DES-CBC, Blowfish-CBC, Cast-CBC, !SkipJack-CBC, AES-CBC y ARC4.
*  Verificación de Integridad: Condensados md5, md5-hmac, md5-kpdk, sha1 (no está operando en OpenBSD 4.8), sha1-hmac, sha1-kpdk
*  Operaciones con llaves asimétricas: !CRK_MOD_EXP, !CRK_MOD_EXP_CRT, !CRK_DSA_SIGN, !CRK_DSA_VERIFY, !CRK_DH_COMPUTE_KEY

La utilización con una sesión de los algoritmos simétricos y condensados requiere:
# Abrir el dispositivo ```/dev/crypto```
# Recibir una manija de archivo para abrir sesiones enviando CRIOGET al dispositivo ```/dev/crypto``` con ```ioctl```.
# Crear una sesión de tipo ```struct session_op``` y pasarla al dispositivo con ```ioctl``` y la constante ```CIOGSESSION```.
# Establecer parámetros criptográficos en una estructura de tipo ```struct crypt_op cr_op``` y pasarlos al dispositivo solicitando que se realice la operación criptográfica con ```ioctl``` y la constante ```CIOCCRYPT```.  Entre los parámetros están: ```cr_op->op``` que indica si se cifra o descifra, ```cr_op->len``` con longitud del mensaje claro y del mensaje cifrado, ```cr_op->src``` que apunta al mensaje claro (en caso de solicitar encripción), ```cr_op->dst``` que apunta al colchón donde quedará mensaje cifrado, ```cr_op->mac``` hash de una vía, ```cr_op->iv``` vector de inicialización.
#  Cerrar sesión pasando registro ses de la estructura de sesión con ```ioctl``` y constante ```CIOCFSESSION```.

El siguiente ejemplo muestra como puede cifrarse un mensaje de 16 bytes
con el algoritmo 3-DES.

<pre>
/** 
 *  Encripta un mensaje usando 3-DES con infraestructura criptográfica 
 *      de OpenBSD 
 *  Basado en fuentes de prueba de regresión
 *   regress/sys/crypto/enc/des3.c 
 *  Para compilar usar:
 *  	cc -o des3dev des3dev.c
 *  Ejecutelo como usuario root:
 *  	sudo ./des3dev
 */

#include <sys/types.h>
#include <sys/param.h>
#include <sys/ioctl.h>
#include <sys/sysctl.h>
#include <crypto/cryptodev.h>
#include <err.h>
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

// Tamaño máximo del mensaje claro y cifrado
#define SZ 16

/**
 * Imprime representación hexadecimal del colchon b de tamaño l
 */
void imphex(unsigned char b![], size_t l) 
{
	int i;
	for (i=0; i< l; i++) {
		printf("%02x", b![i]);
	}
	printf("\n");
}


/**
 * Activa infraestructura criptográfica para usuarios y por software si
 * no hay hardware de criptografía
 */
void activa_cript(void)
{
        int mib![2], old, new;
        size_t olen, nlen;

        olen = nlen = sizeof(new);

        mib![0] = CTL_KERN;
        mib![1] = KERN_USERCRYPTO;
        if (sysctl(mib, 2, &old, &olen, NULL, 0) < 0) {
                err(1, "sysctl falló al leer usercrypto");
        }
        printf("kern.usercrypto=%i\n", old);
        if (old ``` 0) {
                printf("Activando kern.usercrypto\n");
                mib![0] = CTL_KERN;
                mib![1] = KERN_USERCRYPTO;
                new = 1;
                if (sysctl(mib, 2, &old, &olen, &new, nlen) < 0) {
                        err(1, "sysctl falló al escribir usercyrpto");
                }
        }

        mib![0] = CTL_KERN;
        mib![1] = KERN_CRYPTODEVALLOWSOFT;
        if (sysctl(mib, 2, &old, &olen, NULL, 0) < 0) {
                err(1, "sysctl falló al leer cryptodevallowsoft");
        }
        printf("kern.cryptodevallowsoft=%i\n", old);
        if (old ``` 0) {
                printf("Activando kern.cryptodevallowsoft\n");
                mib![0] = CTL_KERN;
                mib![1] = KERN_CRYPTODEVALLOWSOFT;
                new = 1;
                if (sysctl(mib, 2, &old, &olen, &new, nlen) < 0) {
                        err(1, "sysctl falló al escribir userdevallowsoft");
                }
        }
}


int
main(int argc, char **argv)
{
	unsigned char key![24] = "012345670123456701234567", //Llave
		iv![8] = "12345678";   	// Vector Inicialización
	unsigned char b1![SZ],   			    // Entrada
		b2![SZ];					    // Salida

	memset(b1, 0, sizeof(b1));
	printf("Mensaje por encriptar con 3DES:    ");
	scanf("%15s", b1);

	printf("Mensaje claro en hexadecimal es:   ");
	imphex(b1, SZ);

	memset(b2, 0, sizeof(b2));

	activa_cript();

	struct session_op session;
	struct crypt_op cryp;
	int cryptodev_fd = -1, fd = -1;

	if ((cryptodev_fd = open("/dev/crypto", O_RDWR, 0)) < 0) {
		warn("/dev/crypto");
		exit(1);
	}
	if (ioctl(cryptodev_fd, CRIOGET, &fd) ``` -1) {
		warn("CRIOGET failed");
		exit(1);
	}
	memset(&session, 0, sizeof(session));
	session.cipher = !CRYPTO_3DES_CBC;
	session.key = (caddr_t) key;
	session.keylen = sizeof(key);
	if (ioctl(fd, CIOCGSESSION, &session) ``` -1) {
		warn("CIOCGSESSION");
		exit(1);
	}
	memset(&cryp, 0, sizeof(cryp));
	cryp.ses = session.ses;
	cryp.op = COP_ENCRYPT;
	cryp.flags = 0;
	cryp.len = SZ;
	cryp.src = (caddr_t) b1;
	cryp.dst = (caddr_t) b2;
	cryp.iv = (caddr_t) iv;
	cryp.mac = 0;
	if (ioctl(fd, CIOCCRYPT, &cryp) ``` -1) {
		warn("CIOCCRYPT");
		exit(1);
	}
	if (ioctl(fd, CIOCFSESSION, &session.ses) ``` -1) {
		warn("CIOCFSESSION");
		exit(1);
	}
	close(fd);
	close(cryptodev_fd);

	printf("Mensaje cifrado en hexadecimal es: ");
	imphex(b2, SZ);

	return 0;
}
</pre>


!Criptografía en la librería C de OpenBSD 

Las rutinas de criptografía por software además de estar disponibles por la infraestructura criptográfica de OpenBSD, están disponibles en librerías (principalmente libcrypto).     Para el caso de 3-DES está disponible en la librería libdes, como lo ejemplifica el siguiente programa que puede ser ejecutado por un usuario sin privilegios (a diferencia del ejemplo de la sección anterior).

<pre>
/** 
 *  Encripta un mensaje usando 3-DES de la librería libdes de OpenBSD 
 *  Basado en fuentes de prueba de regresión
 *   regress/sys/crypto/enc/des3.c 
 *  Para compilar usar:
 *  cc -o des3lib -ldes des3lib.c
 */

#include <crypto/des.h>
#include <stdio.h>
#include <string.h>

// Tamaño máximo del mensaje claro y cifrado
#define SZ 16

/**
 * Imprime representación hexadecimal del colchon b de tamaño l
 */
void imphex(unsigned char b[], size_t l) 
{
	int i;
	for (i=0; i< l; i++) {
		printf("%02x", b![i]);
	}
	printf("\n");
}


int
main(int argc, char **argv)
{
	unsigned char key![24] = "012345670123456701234567", //Llave
		iv![8] = "12345678";   			    // Acelerador
	des_key_schedule ks1, ks2, ks3;  // Llave transformada para 3DES
	unsigned char b1![SZ],   			    // Entrada
		b2![SZ];					    // Salida

	memset(b1, 0, sizeof(b1));
	printf("Mensaje por encriptar con 3DES:    ");
	scanf("%15s", b1);

	printf("Mensaje claro en hexadecimal es:   ");
	imphex(b1, SZ);

	/* Preparamos salida dejandola en blanco */
	memset(b2, 0, sizeof(b2));

	/* Configuración de llaves */
        des_set_key((void *) key, ks1);
        des_set_key((void *) (key+8), ks2);
        des_set_key((void *) (key+16), ks3);

	/* Ver man des_ede3_cbc_encrypt:
	    void !DES_ede3_cbc_encrypt(const unsigned char *input,
               unsigned char *output, long length, DES_key_schedule *ks1,
               DES_key_schedule *ks2, DES_key_schedule *ks3, DES_cblock *ivec,
               int enc);
	       que implementa encripción DES CBC triple exterior con tres
	       llaves, como lo usa SSL.
	 */
        des_ede3_cbc_encrypt((void *)b1, (void*)b2, sizeof(b1), ks1, ks2, ks3,
	    (void*)iv, DES_ENCRYPT);

	printf("Mensaje cifrado en hexadecimal es: ");
	imphex(b2, SZ);

	return 0;
}
</pre>

Al comparar la ejecución de este programa y el de la sección anterior debe obtener los mismos resultados cuando ingresa las mismas cadenas por encriptar, por ejemplo:
<pre>
$ sudo ./des3dev
Mensaje por encriptar con 3DES:    hola
Mensaje claro en hexadecimal es:   !686f6c61000000000000000000000000
kern.usercrypto=1
kern.cryptodevallowsoft=1
Mensaje cifrado en hexadecimal es: !3c4dfac03881693d561f1b9a8eeced91

$ ./des3lib
Mensaje por encriptar con 3DES:    hola
Mensaje claro en hexadecimal es:   !686f6c61000000000000000000000000
Mensaje cifrado en hexadecimal es: !3c4dfac03881693d561f1b9a8eeced91
</pre>

!Implementación

En tiempo de compilación del kernel, se incluye o no soporte para esta infraestructura con el símbolo ```CRYPTO```

En tiempo de ejecución del kernel la utilización de esta infraestructura primero depende de alungas  variables sysctl (```kern.usercrypto```, ```kern.cryptodevallowsoft```, ```kern.userasymcrypto```) que se detallan más adelante.

Durante el arranque del kernel, se crea un hilo de ejecución del kernel para esta infraestructura en ```kern/init_main.c```


!BIBLIOGRAFÍA

* {1} man ipsecctl
* {2} man ipsec
* {3} man route
* {4} RFC 2367
* {5} RFC 2402 http://www.ietf.org/rfc/rfc2402.txt
* {6} RFC 4302 http://tools.ietf.org/html/rfc4302
* {7} Angelos D. Keromytis et al. The Design of the OpenBSD Cryptographic Framework. http://www.usenix.org/event/usenix03/tech/full_papers/keromytis/keromytis_html/main.html
* {8} http://netbsd.gw.com/cgi-bin/man-cgi?crypto+4+NetBSD-current
