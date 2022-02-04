---

title: Solucionar Let's Encrypt vencidos el 30/Sep/2021
date: 2021-11-27 16:18 UTC
tags: 

---

# Solucionar Let's Encrypt vencidos el 30/Sep/2021

Una solución rápida y en español simple, junto con una explicación del 
problema de los certificados raíces de Autoridad Certificadora que 
vencieron para clientes y usuarios.

Esta es adaptación a español del artículo de Eric Fossas 
'Fix Expired Let’s Encrypt 9/30' disponible en
<https://ericfossas.medium.com/fix-expired-letsencrypt-9-30-18f775581dfd>


![Foto de una cadena](https://miro.medium.com/max/700/1*2oxB8aEbJgcYaCF_JcwJjQ.jpeg)

## 1. Introduccción

Aquí está la cuestión, Let's Encrypt como Autoridad Certificadora (CA)
usa un certificado raíz con "firma-cruzada". Es como si tuviera dos 
certificados raíz de Autoridad Certificadora, pero tu dispositivo sólo 
necesita confiar en uno de ellos, no en ambos.

Esto fue de ayuda para Let's Encrypt cuando comenzó porque el 
certificado raíz `DST Root CA X3` era más antiguo y ya era de confianza
para muchos dispositivos y así pudieron esperar mientras su nuevo certificado
raíz `ISRG Root X1` era añadido a la mayoría de dispositivos.

El certificado `DST Root CA X3` expiró el 30/Sep/2021 y si estás
viendo un error de certificado (como `ERR_CERT_DATE_INVALID`), entonces
es por una de dos razones:

1. Que no cuentas con el nuevo certificado `ISRG Root X1` o bien
2. Una falla con OpenSSL que no le permite ignorar el certificado antiguo 
   `DST Root CA X3.`

## 2. Corregir el problema

### 2.1 Posibilidad 1. Certificado Raíz de Autoridad Certificadora faltante

Bien, esto no es lo mejor.  Puede que tu dispositivo no tenga el certificado
raíz `ISRG Root X1`. Ponerlo en tu computador puede o no ser simple.

#### 2.1.1 Windows

Los computadores Windows mantienen los certificados de Autoridades
Certificadoras al día.  Asegurate de estar ejecutando actualizaciones
de seguridad.

#### 2.1.2 MacOS

Para dispositivos que no mantienen al día sus certificados raíces de
Autoridades Certificadoras, como MacOS, la solución "fácil" es
actualizar el sistema operativo.

Así que si corres una versión antigua de Mac, ve a Preferencias del Sistema
y pulsa Actualizar el Sistema.

¿No quieres actualizar tu sistema operativo?  Tendrás que descargar el 
certificado `ISRG Root X1` de 
<https://letsencrypt.org/certificates> y añadirlo a tus certificados
de confianza.

Esta no es una gran opción para usuarios con dificultades con la tecnología,
pero aquí está la explicación para alguien  que quiera hacerlo en Mac:

Ve a este enlace y guarda el archivo con el certificado:
<https://letsencrypt.org/certs/isrgrootx1.pem>

Abre  la aplicación `Keychain Access`.
(comando + barra_espaciadora, y busca Keychain Access)
Pulsa “System” al lado izquierdo. Arrastra el archivo con el certificado
a Keychain Access para agregarlo en la cadena de llaves “System”.

Desplaza la lista y ubica “ISRG Root X1”. Y haz doble click.

![Pantallazo ubicando ISRG Root X1](https://miro.medium.com/max/700/1*fiH59fbWvuSUrAAxdCojaQ.jpeg)


Pulsa la flecha junto a “Trust”. Después junto a “When using this certificate”,
pulsa el desplegable y selecciona “Always Trust”.

![Elegir Always Trust](https://miro.medium.com/max/700/1*hv8GwHHSbM-RAtx5SgIVsQ.jpeg)

Así añades el certificado a tu lista de certificados raíz de 
Autoridades Certificadoras.

Es la misma idea con cualquier dispositivo: descarga el certificado y 
añadelo a la lista de certificados de confianza.

#### 2.1.3. iPhone

Lo descargas, haces doble click y eso te permitirá instalarlo.  
Después le das confianza desde 
`Settings > General > About > Certificate Trust Settings`.

#### 2.1.4. Android

lo descargas, después lo instalas en 
`Settings > Security > Encryption & Credentials > Install A Certificate > CA Certificate`.

#### 2.1.5. OpenBSD

El archivo con certificados de Autoridades Certificadoras está en
`/etc/ssl/certs.pem`, allí se mantienen precedidos, cada uno de
una parte del texto claro del certificado.

Puedes generar un texto claro del certificado con:
<pre>
openssl x509 -text -noout -in isrgrootx1.pem > isrgrootx1.txt
</pre>

Que se verá así:

<pre>
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            82:10:cf:b0:d2:40:e3:59:44:63:e0:bb:63:82:8b:00
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=US, O=Internet Security Research Group, CN=ISRG Root X1
        Validity
            Not Before: Jun  4 11:04:38 2015 GMT
            Not After : Jun  4 11:04:38 2035 GMT
        Subject: C=US, O=Internet Security Research Group, CN=ISRG Root X1
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (4096 bit)
                Modulus:
                    00:ad:e8:24:73:f4:14:37:f3:9b:9e:2b:57:28:1c:
                    87:be:dc:b7:df:38:90:8c:6e:3c:e6:57:a0:78:f7:
...
                    ad:4e:e6:d9:8b:3a:c6:dd:27:51:6e:ff:bc:64:f5:
                    33:43:4f
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Key Usage: critical
                Certificate Sign, CRL Sign
            X509v3 Basic Constraints: critical
                CA:TRUE
            X509v3 Subject Key Identifier: 
                79:B4:59:E6:7B:B6:E5:E4:01:73:80:08:88:C8:1A:58:F6:E9:9B:6E
    Signature Algorithm: sha256WithRSAEncryption
         55:1f:58:a9:bc:b2:a8:50:d0:0c:b1:d8:1a:69:20:27:29:08:
         ac:61:75:5c:8a:6e:f8:82:e5:69:2f:d5:f6:56:4b:b9:b8:73:
...
         bd:ec:89:9b:e9:17:43:df:5b:db:5f:fe:8e:1e:57:a2:cd:40:
         9d:7e:62:22:da:de:18:27
</pre>

Elimina las secciones `Signature Algorithm` y agrega a continuación 
el certificaod en formato PEM, puedes precederlo con un comentario:

<pre>
### Internet Security Research Group

=== /C=US/O=Internet Security Research Group/CN=ISRG Root X1
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            82:10:cf:b0:d2:40:e3:59:44:63:e0:bb:63:82:8b:00
    Signature Algorithm: sha256WithRSAEncryption
        Validity
            Not Before: Jun  4 11:04:38 2015 GMT
            Not After : Jun  4 11:04:38 2035 GMT
        Subject: C=US, O=Internet Security Research Group, CN=ISRG Root X1
        X509v3 extensions:
            X509v3 Key Usage: critical
                Certificate Sign, CRL Sign
            X509v3 Basic Constraints: critical
                CA:TRUE
            X509v3 Subject Key Identifier: 
                79:B4:59:E6:7B:B6:E5:E4:01:73:80:08:88:C8:1A:58:F6:E9:9B:6E
SHA1 Fingerprint=CA:BD:2A:79:A1:07:6A:31:F2:1D:25:36:35:CB:03:9D:43:29:A5:E8
SHA256 Fingerprint=96:BC:EC:06:26:49:76:F3:74:60:77:9A:CF:28:C5:A7:CF:E8:A3:C0:AA:E1:1A:8F:FC:EE:05:C0:BD:DF:08:C6
-----BEGIN CERTIFICATE-----
MIIFazCCA1OgAwIBAgIRAIIQz7DSQONZRGPgu2OCiwAwDQYJKoZIhvcNAQELBQAw
TzELMAkGA1UEBhMCVVMxKTAnBgNVBAoTIEludGVybmV0IFNlY3VyaXR5IFJlc2Vh
cmNoIEdyb3VwMRUwEwYDVQQDEwxJU1JHIFJvb3QgWDEwHhcNMTUwNjA0MTEwNDM4
WhcNMzUwNjA0MTEwNDM4WjBPMQswCQYDVQQGEwJVUzEpMCcGA1UEChMgSW50ZXJu
ZXQgU2VjdXJpdHkgUmVzZWFyY2ggR3JvdXAxFTATBgNVBAMTDElTUkcgUm9vdCBY
MTCCAiIwDQYJKoZIhvcNAQEBBQADggIPADCCAgoCggIBAK3oJHP0FDfzm54rVygc
h77ct984kIxuPOZXoHj3dcKi/vVqbvYATyjb3miGbESTtrFj/RQSa78f0uoxmyF+
0TM8ukj13Xnfs7j/EvEhmkvBioZxaUpmZmyPfjxwv60pIgbz5MDmgK7iS4+3mX6U
A5/TR5d8mUgjU+g4rk8Kb4Mu0UlXjIB0ttov0DiNewNwIRt18jA8+o+u3dpjq+sW
T8KOEUt+zwvo/7V3LvSye0rgTBIlDHCNAymg4VMk7BPZ7hm/ELNKjD+Jo2FR3qyH
B5T0Y3HsLuJvW5iB4YlcNHlsdu87kGJ55tukmi8mxdAQ4Q7e2RCOFvu396j3x+UC
B5iPNgiV5+I3lg02dZ77DnKxHZu8A/lJBdiB3QW0KtZB6awBdpUKD9jf1b0SHzUv
KBds0pjBqAlkd25HN7rOrFleaJ1/ctaJxQZBKT5ZPt0m9STJEadao0xAH0ahmbWn
OlFuhjuefXKnEgV4We0+UXgVCwOPjdAvBbI+e0ocS3MFEvzG6uBQE3xDk3SzynTn
jh8BCNAw1FtxNrQHusEwMFxIt4I7mKZ9YIqioymCzLq9gwQbooMDQaHWBfEbwrbw
qHyGO0aoSCqI3Haadr8faqU9GY/rOPNk3sgrDQoo//fb4hVC1CLQJ13hef4Y53CI
rU7m2Ys6xt0nUW7/vGT1M0NPAgMBAAGjQjBAMA4GA1UdDwEB/wQEAwIBBjAPBgNV
HRMBAf8EBTADAQH/MB0GA1UdDgQWBBR5tFnme7bl5AFzgAiIyBpY9umbbjANBgkq
hkiG9w0BAQsFAAOCAgEAVR9YqbyyqFDQDLHYGmkgJykIrGF1XIpu+ILlaS/V9lZL
ubhzEFnTIZd+50xx+7LSYK05qAvqFyFWhfFQDlnrzuBZ6brJFe+GnY+EgPbk6ZGQ
3BebYhtF8GaV0nxvwuo77x/Py9auJ/GpsMiu/X1+mvoiBOv/2X/qkSsisRcOj/KK
NFtY2PwByVS5uCbMiogziUwthDyC3+6WVwW6LLv3xLfHTjuCvjHIInNzktHCgKQ5
ORAzI4JMPJ+GslWYHb4phowim57iaztXOoJwTdwJx4nLCgdNbOhdjsnvzqvHu7Ur
TkXWStAmzOVyyghqpZXjFaH3pO3JLF+l+/+sKAIuvtd7u+Nxe5AW0wdeRlN8NwdC
jNPElpzVmbUq4JUagEiuTDkHzsxHpFKVK7q4+63SM1N95R1NbdWhscdCb+ZAJzVc
oyi3B43njTOQ5yOf+1CceWxG1bQVs5ZufpsMljq4Ui0/1lvh+wjChP4kqKOJ2qxq
4RgqsahDYVvTH9w7jXbyLeiNdd8XM2w9U/t7y0Ff/9yi0GE44Za4rF2LN9d11TPA
mRGunUHBcnWEvgJBQl9nJEiU0Zsnvgc/ubhPgXRR4Xq37Z0j4r7g1SgEEzwxA57d
emyPxgcYxn/eR44/KJ4EBs+lVDR3veyJm+kXQ99b21/+jh5Xos1AnX5iItreGCc=
-----END CERTIFICATE-----

</pre>

Emplea un editor de texto para agregarlo al archivo `/etc/ssl/certs.pem,`
se recomienda agregarlo en orden alfabético.

#### 2.1.6. Linux Amazon

En Linux Amazon, los certificados raíz de Autoridades Certificadoras pueden
encontrarse en dos archivos:
`/etc/pki/tls/certs/ca-bundle.crt` y `/etc/pki/tls/certs/ca-bundle.trust.crt`

Se almacenan en ambos.  Revisa el formato y con un editor agrega
el nuevo de una forma similar a la presentada en la sección 2.1.5.

#### 2.1.7 Linux Ubuntu

En Ubuntu, los certificados raíces de Autoridades Certificadoras pueden 
encontrarse en: `/etc/ssl/certs`
Se almacenan individualmente. 
Por ejemplo para eliminar los certificados vencidos ejecuta 
`ls -l | grep DST` y elimina el archivo resultante.


### 2.2. Posibilidad 2: falla de OpenSSL

Si tu computador corre una versión antigua de OpenSSL, tiene una falla. 
Se supone que debería ignorar el certificado expirado
`DST Root CA X3` dado que el certificado `ISRG Root X1` es aún válido,
pero infortunadamente no lo hace.

La forma difícil de resolverlo es actualizando tu OpenSSL. La forma fácil de
arreglarlo es eliminando ese certificado antiguo de la lista de 
Autoridades Certificadoras en tu dispositivo.

Independiente del sistema operativo, basta encontrar los certificados raíces
de Autoridades Certificadoras y eliminar el
`DST Root CA X3` de allí (mira en la sección anterior donde puedes
consultar donde ver los certificados raíces en diversos sistemas
operativos).



## 3. Resumen

Así que en resumen, lo recomendable es actualizar el sistema operativo si
ves errores de certificados vencidos. 

Si eres un usuarios técnicamente eficiente, puedes eliminar el certificado
`DST Root CA X3` y/o instalar el certificado `ISRG Root X1.`
También si lo requieres podrías actualizar OpenSSL (que podría ser
difícil si tienes un sistema operativo antiguo).


