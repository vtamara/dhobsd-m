---
title: IPsec_en_OpenBSD
date: 2010-12-22
tags:
---
##IPsec

Se recomienda primero leer la traducción del [manual de IPsec(4) en OpenBSD | man 4 ipsec]

Para una práctica con IPsec conectando dos redes con cortafuegos OpenBSD se recomienda [Utilización de IPsec]

También es muy recomendable consultar la traducción del [manual de isakmpd(8) en OpenBSD| man 8 isakmpd] y del [manual de ipsecctl (8) | man 8 ipsecctl] 

##Examinando estado de IPsec

Hay una Base de datos de políticas de seguridad SPD (Security Policy Database) y una Base de datos de asociaciones de seguridad SAD (Security Association Database) implementadas en kernel según {1}

* ```ipsecctl -s sa``` muestra las entradas activas en SAD
* ```ipsecctl -s flow``` muestra reglas cargadas en la SPD

Por ejemplo si se tienen dos redes de área local conectadas a Internet así:

<pre>
                  LAN 1                                                        LAN 2

                        Cortafuegos 1                          Cortafuegos 2          
+---------------+      +----------------+                    +---------------+      +-------------+
| 192.168.60/24 | <--> | 192.168.60.1   |                    | 172.20.1.1    | <--> | 172.20.1/24 |
+---------------+      | 202.245.63.218 | <--> Internet <--> | 200.69.110.71 |      +-------------+
                       +----------------+                    +---------------+  
</pre>                                                             

Y se establece una VPN entre ellas, con IPsec en modo tunel, las entradas SPD vistas desde el cortafuegos 1 son:
<pre>
flow esp in from 200.69.110.71 to 202.245.63.218 peer 200.69.110.71 type use
flow esp out from 202.245.63.218 to 200.69.110.71 peer 200.69.110.71 type require
flow esp in from 172.20.1.0/24 to 202.245.63.218 peer 200.69.110.71 type use
flow esp out from 202.245.63.218 to 172.20.1.0/24 peer 200.69.110.71 type require
flow esp in from 200.69.110.71 to 192.168.60.0/24 peer 200.69.110.71 type use
flow esp out from 192.168.60.0/24 to 200.69.110.71 peer 200.69.110.71 type require
flow esp in from 172.20.1.0/24 to 192.168.60.0/24 peer 200.69.110.71 type use
flow esp out from 192.168.60.0/24 to 172.20.1.0/24 peer 200.69.110.71 type require
flow esp in from 172.20.1.0/24 to 202.245.63.218 peer 200.69.110.71 type use
flow esp out from 202.245.63.218 to 172.20.1.0/24 peer 200.69.110.71 type require
flow esp in from 172.20.1.0/24 to 192.168.60.0/24 peer 200.69.110.71 type use
flow esp out from 192.168.60.0/24 to 172.20.1.0/24 peer 200.69.110.71 type require
</pre>

Las entradas de SAD con información criptográfica (opción -k) y detalles (opción -vv):
<pre>
@0 esp tunnel from 200.69.110.71 to 202.245.63.218 spi !0x2d2148ab auth hmac-sha2-256 enc aes \
        authkey !0x61dac444bbc97245532dbd8e4fbdcc57c6e2a492bc68ab669b73495fc5d98ac2 \
        enckey !0x9388bd3fc0aa2093f041f88ff886d2f9
        sa: spi !0x2d2148ab auth hmac-sha2-256 enc aes
                state mature replay 16 flags 4
        lifetime_cur: alloc 0 bytes 0 add 1293629063 first 0
        lifetime_hard: alloc 0 bytes 0 add 1200 first 0
        lifetime_soft: alloc 0 bytes 0 add 1080 first 0
        address_src: 200.69.110.71
        address_dst: 202.245.63.218
        key_auth: bits 256: !61dac444bbc97245532dbd8e4fbdcc57c6e2a492bc68ab669b73495fc5d98ac2
        key_encrypt: bits 128: !9388bd3fc0aa2093f041f88ff886d2f9
        identity_src: type prefix id 0: 200.69.110.71/32
        identity_dst: type prefix id 0: 202.245.63.218/32
        src_mask: 255.255.255.255
        dst_mask: 255.255.255.0
        protocol: proto 0 flags 0
        flow_type: type use direction in
        src_flow: 200.69.110.71
        dst_flow: 192.168.60.0
        remote_auth: type rsa
@0 esp tunnel from 200.69.110.71 to 202.245.63.218 spi !0x3f6b5b47 auth hmac-sha2-256 enc aes \
        authkey !0x4c96f3b4d731cdffd372c15fc09aad077e5832f657b1872a83078da58f8f9fc8 \
        enckey !0x513cc494645b97232c8e58b859a2d69d
        sa: spi !0x3f6b5b47 auth hmac-sha2-256 enc aes
                state mature replay 16 flags 4
        lifetime_cur: alloc 0 bytes 1344 add 1293629094 first 1293629104
        lifetime_hard: alloc 0 bytes 0 add 1200 first 0
        lifetime_soft: alloc 0 bytes 0 add 1080 first 0
        address_src: 200.69.110.71
        address_dst: 202.245.63.218
        key_auth: bits 256: !4c96f3b4d731cdffd372c15fc09aad077e5832f657b1872a83078da58f8f9fc8
        key_encrypt: bits 128: !513cc494645b97232c8e58b859a2d69d
        identity_src: type prefix id 0: 200.69.110.71/32
        identity_dst: type prefix id 0: 202.245.63.218/32
        src_mask: 255.255.255.255
        dst_mask: 255.255.255.255
        protocol: proto 0 flags 0
        flow_type: type use direction in
        src_flow: 200.69.110.71
        dst_flow: 202.245.63.218
        remote_auth: type rsa
        lifetime_lastuse: alloc 0 bytes 0 add 0 first 1293629466
@0 esp tunnel from 200.69.110.71 to 202.245.63.218 spi !0x9b534a88 auth hmac-sha2-256 enc aes \
        authkey !0x693de5805bafa38de0fd49e4645b569bcb9621b2e0d43e3eb4d3b06c59471806 \
        enckey !0x7bae8e0c7abb6cb487c9b7b0bd50e039
        sa: spi !0x9b534a88 auth hmac-sha2-256 enc aes
                state mature replay 16 flags 4
        lifetime_cur: alloc 0 bytes 0 add 1293629057 first 0
        lifetime_hard: alloc 0 bytes 0 add 1200 first 0
        lifetime_soft: alloc 0 bytes 0 add 1080 first 0
        address_src: 200.69.110.71
        address_dst: 202.245.63.218
        key_auth: bits 256: !693de5805bafa38de0fd49e4645b569bcb9621b2e0d43e3eb4d3b06c59471806
        key_encrypt: bits 128: !7bae8e0c7abb6cb487c9b7b0bd50e039
        identity_src: type prefix id 0: 200.69.110.71/32
        identity_dst: type prefix id 0: 202.245.63.218/32
        src_mask: 255.255.255.0
        dst_mask: 255.255.255.0
        protocol: proto 0 flags 0
        flow_type: type use direction in
        src_flow: 172.20.1.0
        dst_flow: 192.168.60.0
        remote_auth: type rsa
@0 esp tunnel from 202.245.63.218 to 200.69.110.71 spi !0xaf79def1 auth hmac-sha2-256 enc aes \
        authkey !0xaee517fc262ef9931a29c8a0063ce2f6ad95343b60b13d08ddbec675b3d48810 \
        enckey !0xb2765081a9288d9eeb7a628dcd77decd
        sa: spi !0xaf79def1 auth hmac-sha2-256 enc aes
                state mature replay 16 flags 4
        lifetime_cur: alloc 0 bytes 0 add 1293629057 first 0
        lifetime_hard: alloc 0 bytes 0 add 1200 first 0
        lifetime_soft: alloc 0 bytes 0 add 1080 first 0
        address_src: 202.245.63.218
        address_dst: 200.69.110.71
        key_auth: bits 256: !aee517fc262ef9931a29c8a0063ce2f6ad95343b60b13d08ddbec675b3d48810
        key_encrypt: bits 128: !b2765081a9288d9eeb7a628dcd77decd
        identity_src: type prefix id 0: 202.245.63.218/32
        identity_dst: type prefix id 0: 200.69.110.71/32
        src_mask: 255.255.255.0
        dst_mask: 255.255.255.0
        protocol: proto 0 flags 0
        flow_type: type use direction out
        src_flow: 192.168.60.0
        dst_flow: 172.20.1.0
        remote_auth: type rsa
@0 esp tunnel from 200.69.110.71 to 202.245.63.218 spi !0xe600714c auth hmac-sha2-256 enc aes \
        authkey !0x474ebe7d6e3a9d87910264489902fd59808b15994b2c18571719140de7f6c2b8 \
        enckey !0x6e7c74d57373a09dca2810c221a92e73
        sa: spi !0xe600714c auth hmac-sha2-256 enc aes
                state mature replay 16 flags 4
        lifetime_cur: alloc 0 bytes 0 add 1293629140 first 0
        lifetime_hard: alloc 0 bytes 0 add 1200 first 0
        lifetime_soft: alloc 0 bytes 0 add 1080 first 0
        address_src: 200.69.110.71
        address_dst: 202.245.63.218
        key_auth: bits 256: !474ebe7d6e3a9d87910264489902fd59808b15994b2c18571719140de7f6c2b8
        key_encrypt: bits 128: !6e7c74d57373a09dca2810c221a92e73
        identity_src: type prefix id 0: 200.69.110.71/32
        identity_dst: type prefix id 0: 202.245.63.218/32
        src_mask: 255.255.255.0
        dst_mask: 255.255.255.255
        protocol: proto 0 flags 0
        flow_type: type use direction in
        src_flow: 172.20.1.0
        dst_flow: 202.245.63.218
        remote_auth: type rsa
@0 esp tunnel from 202.245.63.218 to 200.69.110.71 spi !0xedb60d48 auth hmac-sha2-256 enc aes \
        authkey !0x4c37375d027fbc60400e78f6d0eb0f177b568b937a8102cb15df4e48466e8a7f \
        enckey !0xff5c5dd007222688dc68ef7b13771f62
        sa: spi !0xedb60d48 auth hmac-sha2-256 enc aes
                state mature replay 16 flags 4
        lifetime_cur: alloc 0 bytes 1176 add 1293629094 first 1293629104
        lifetime_hard: alloc 0 bytes 0 add 1200 first 0
        lifetime_soft: alloc 0 bytes 0 add 1080 first 0
        address_src: 202.245.63.218
        address_dst: 200.69.110.71
        key_auth: bits 256: !4c37375d027fbc60400e78f6d0eb0f177b568b937a8102cb15df4e48466e8a7f
        key_encrypt: bits 128: !ff5c5dd007222688dc68ef7b13771f62
        identity_src: type prefix id 0: 202.245.63.218/32
        identity_dst: type prefix id 0: 200.69.110.71/32
        src_mask: 255.255.255.255
        dst_mask: 255.255.255.255
        protocol: proto 0 flags 0
        flow_type: type use direction out
        src_flow: 202.245.63.218
        dst_flow: 200.69.110.71
        remote_auth: type rsa
        lifetime_lastuse: alloc 0 bytes 0 add 0 first 1293629466
@0 esp tunnel from 202.245.63.218 to 200.69.110.71 spi !0xf20a28d4 auth hmac-sha2-256 enc aes \
        authkey !0x04d5bdca578ff0e04ffe9aeeb85b9627c71cde734356ce3854e8dad18b9543a2 \
        enckey !0x56bdd25d34b6aa99ffd5301c6ca96c26
        sa: spi !0xf20a28d4 auth hmac-sha2-256 enc aes
                state mature replay 16 flags 4
        lifetime_cur: alloc 0 bytes 0 add 1293629140 first 0
        lifetime_hard: alloc 0 bytes 0 add 1200 first 0
        lifetime_soft: alloc 0 bytes 0 add 1080 first 0
        address_src: 202.245.63.218
        address_dst: 200.69.110.71
        key_auth: bits 256: !04d5bdca578ff0e04ffe9aeeb85b9627c71cde734356ce3854e8dad18b9543a2
        key_encrypt: bits 128: !56bdd25d34b6aa99ffd5301c6ca96c26
        identity_src: type prefix id 0: 202.245.63.218/32
        identity_dst: type prefix id 0: 200.69.110.71/32
        src_mask: 255.255.255.255
        dst_mask: 255.255.255.0
        protocol: proto 0 flags 0
        flow_type: type use direction out
        src_flow: 202.245.63.218
        dst_flow: 172.20.1.0
        remote_auth: type rsa
@0 esp tunnel from 202.245.63.218 to 200.69.110.71 spi !0xf2d252c4 auth hmac-sha2-256 enc aes \
        authkey !0x4bea67fb5354b699dbec5d668c41138e9d63c9bd1729a93f5b617a5986aec0e7 \
        enckey !0x12d5d3fcf60f213546ba333e16c1bf3d
        sa: spi !0xf2d252c4 auth hmac-sha2-256 enc aes
                state mature replay 16 flags 4
        lifetime_cur: alloc 0 bytes 0 add 1293629063 first 0
        lifetime_hard: alloc 0 bytes 0 add 1200 first 0
        lifetime_soft: alloc 0 bytes 0 add 1080 first 0
        address_src: 202.245.63.218
        address_dst: 200.69.110.71
        key_auth: bits 256: !4bea67fb5354b699dbec5d668c41138e9d63c9bd1729a93f5b617a5986aec0e7
        key_encrypt: bits 128: !12d5d3fcf60f213546ba333e16c1bf3d
        identity_src: type prefix id 0: 202.245.63.218/32
        identity_dst: type prefix id 0: 200.69.110.71/32
        src_mask: 255.255.255.0
        dst_mask: 255.255.255.255
        protocol: proto 0 flags 0
        flow_type: type use direction out
        src_flow: 192.168.60.0
        dst_flow: 200.69.110.71
        remote_auth: type rsa
</pre>

```isakmpd``` puede generar un reporte de su operación al enviarle la señal ```USR1``` (señal 30) por ejemplo ```sudo pkill -30 isakmpd``` generará  ```/var/run/isakmpd.report``` que incluye información como:

<pre>
084640.619884 Report> sa_report: !0x2031a2c00 peer-200.69.110.71 phase 1 doi 1 flags !0x3
084640.619892 Report> sa_report: icookie fe0e6d17c589deb4 rcookie !1fc04b38f2e8f7d6
084640.619899 Report> sa_report: msgid 00000000 refcnt 3
084640.619906 Report> sa_report: life secs 3600 kb 0
084640.619913 Report> sa_report: suite 1 proto 1
084640.619920 Report> sa_report: spi_sz![0] 0 spi![0] !0x0 spi_sz![1] 0 spi![1] !0x0
084640.619936 Report> sa_report: initiator id: 202.245.63.218, responder id: 200.69.110.71, src: 2
02.245.63.218 dst: 200.69.110.71
084640.620214 Report> sa_report: !0x206e1f800 from-192.168.60.0/24-to-200.69.110.71 phase 2 doi 1
flags !0xb
084640.620221 Report> sa_report: icookie fe0e6d17c589deb4 rcookie !1fc04b38f2e8f7d6
084640.620229 Report> sa_report: msgid 304574ae refcnt 3
084640.620235 Report> sa_report: life secs 1200 kb 0
084640.620242 Report> sa_report: suite 1 proto 3
084640.620250 Report> sa_report: spi_sz![0] 4 spi![0] !0x21015c171 spi_sz![1] 4 spi![1] !0x20e13b090
084640.620265 Report> sa_report: initiator id: 202.245.63.218, responder id: 200.69.110.71, src: 2
02.245.63.218 dst: 200.69.110.71
084640.620273 Report> sa_report: spi![0]:
084640.620281 Report> bed3b2be
084640.620288 Report> sa_report: spi![1]:
084640.620296 Report> c113ef23
084640.621166 Report> sa_report: !0x208325c00 from-192.168.60.0/24-to-200.69.110.71 phase 2 doi 1
flags !0x3
084640.621173 Report> sa_report: icookie fe0e6d17c589deb4 rcookie !1fc04b38f2e8f7d6
084640.621181 Report> sa_report: msgid !0cc6e1c7 refcnt 3
084640.621188 Report> sa_report: life secs 1200 kb 0
084640.621195 Report> sa_report: suite 1 proto 3
084640.621202 Report> sa_report: spi_sz![0] 4 spi![0] !0x21015c0e0 spi_sz![1] 4 spi![1] !0x20e13bfe0
084640.621218 Report> sa_report: initiator id: 202.245.63.218, responder id: 200.69.110.71, src: 2
02.245.63.218 dst: 200.69.110.71
084640.621226 Report> sa_report: spi![0]:
084640.621234 Report> !2b69a42d
084640.621241 Report> sa_report: spi![1]:
084640.621249 Report> !404f11b7
084640.621647 Report> sa_report: !0x20eb7b200 from-202.245.63.218-to-200.69.110.71 phase 2 doi 1 f
lags !0x3
084640.621654 Report> sa_report: icookie fe0e6d17c589deb4 rcookie !1fc04b38f2e8f7d6
084640.621661 Report> sa_report: msgid cff2acaf refcnt 3
084640.621668 Report> sa_report: life secs 1200 kb 0
084640.621675 Report> sa_report: suite 1 proto 3
084640.621683 Report> sa_report: spi_sz![0] 4 spi![0] !0x21015cae0 spi_sz![1] 4 spi![1] !0x21015ca20
084640.621698 Report> sa_report: initiator id: 202.245.63.218, responder id: 200.69.110.71, src: 2
02.245.63.218 dst: 200.69.110.71
084640.621716 Report> sa_report: spi![0]:
084640.621714 Report> !139f6ac8
084640.621721 Report> sa_report: spi![1]:
084640.621729 Report> f658eccf
084640.621827 Report> sa_report: !0x208325e00 from-192.168.60.0/24-to-172.20.1.0/24 phase 2 doi 1
flags !0x3
084640.621835 Report> sa_report: icookie fe0e6d17c589deb4 rcookie !1fc04b38f2e8f7d6
084640.621842 Report> sa_report: msgid 11dbf573 refcnt 3
084640.621849 Report> sa_report: life secs 1200 kb 0
084640.621856 Report> sa_report: suite 1 proto 3
084640.621863 Report> sa_report: spi_sz![0] 4 spi![0] !0x21015ca00 spi_sz![1] 4 spi![1] !0x21015c871
084640.621879 Report> sa_report: initiator id: 202.245.63.218, responder id: 200.69.110.71, src: 2
02.245.63.218 dst: 200.69.110.71
084640.621887 Report> sa_report: spi![0]:
084640.621895 Report> f9d5371e
084640.621902 Report> sa_report: spi![1]:
084640.621909 Report> !3f73625d
084640.622061 Report> sa_report: initiator id: 202.245.63.218, responder id: 200.69.110.71, src: 2
02.245.63.218 dst: 200.69.110.71
084640.622069 Report> sa_report: spi![0]:
084640.622077 Report> 64285f41
084640.622084 Report> sa_report: spi![1]:
084640.622092 Report> !0f36a9ab
084640.622798 Report> udp_encap_report: fd 19 src 202.245.63.218:4500 dst 200.69.110.71:4500
084640.622806 Report> transport_report: transport !0x20685b200 flags 0 refcnt 1
084640.622817 Report> udp_report: fd 18 src 202.245.63.218:500 dst 200.69.110.71:500
084640.622825 Report> transport_report: transport !0x2082ff780 flags 0 refcnt 1
084640.622872 Report> udp_encap_report: fd 19 src 202.245.63.218:4500 dst 200.69.110.71:4500
084640.622880 Report> transport_report: transport !0x2058b1a00 flags 0 refcnt 1
084640.622890 Report> udp_report: fd 18 src 202.245.63.218:500 dst 200.69.110.71:500
084640.622898 Report> transport_report: transport !0x2058b1000 flags 0 refcnt 1
084640.623346 Report> udp_encap_report: fd 19 src 202.245.63.218:4500 dst 200.69.110.71:4500
084640.623354 Report> transport_report: transport !0x2061a4e80 flags 0 refcnt 1
084640.623364 Report> udp_report: fd 18 src 202.245.63.218:500 dst 200.69.110.71:500
084640.624046 Report> connection_report: connection from-192.168.60.0/24-to-172.20.1.0/24 next che
ck 32 seconds
084640.624054 Report> connection_report: connection from-192.168.60.0/24-to-200.69.110.71 next che
ck 32 seconds
084640.624061 Report> connection_report: connection from-202.245.63.218-to-172.20.1.0/24 next chec
k 32 seconds
084640.624069 Report> connection_report: connection from-202.245.63.218-to-200.69.110.71 next chec
k 32 seconds
084640.624647 Report> connection_report: passive connection from-192.168.60.0/24-to-172.20.1.0/24
local_id: 192.168.60.0/255.255.255.0, remote_id: 172.20.1.0/255.255.255.0
084640.624661 Report> connection_report: passive connection from-192.168.60.0/24-to-200.69.110.71
local_id: 192.168.60.0/255.255.255.0, remote_id: 200.69.110.71
084640.624675 Report> connection_report: passive connection from-202.245.63.218-to-172.20.1.0/24 l
ocal_id: 202.245.63.218, remote_id: 172.20.1.0/255.255.255.0
084640.624689 Report> connection_report: passive connection from-202.245.63.218-to-200.69.110.71 l
ocal_id: 202.245.63.218, remote_id: 200.69.110.71
084640.625218 Report> timer_report: event connection_checker(!0x20afef7c0) scheduled in 32 seconds
084640.625385 Report> timer_report: event exchange_free_aux(!0x202ad8140) scheduled in 32 seconds
084640.625716 Report> timer_report: event sa_hard_expire(!0x203204080) scheduled in 34 seconds
084640.625775 Report> timer_report: event sa_soft_expire(!0x20f0dfb80) scheduled in 320 seconds
084640.628326 Report> ![from-192.168.60.0/24-to-200.69.110.71]
084640.628333 Report> Phase=    2
084640.628340 Report> ISAKMP-peer=      peer-200.69.110.71
084640.628347 Report> Configuration=    phase2-from-192.168.60.0/24-to-200.69.110.71
084640.628354 Report> Local-ID= from-192.168.60.0/24
084640.628360 Report> Remote-ID=        to-200.69.110.71
084640.628524 Report> ![phase2-from-192.168.60.0/24-to-200.69.110.71]
084640.628531 Report> EXCHANGE_TYPE=    QUICK_MODE
084640.628538 Report> Suites=   QM-ESP-AES-SHA2-256-PFS-SUITE
084640.629318 Report> ![from-202.245.63.218-to-172.20.1.0/24]
084640.629325 Report> Phase=    2
084640.629332 Report> ISAKMP-peer=      peer-200.69.110.71
084640.629339 Report> Configuration=    phase2-from-202.245.63.218-to-172.20.1.0/24
084640.629346 Report> Local-ID= from-202.245.63.218
084640.629352 Report> Remote-ID=        to-172.20.1.0/24
084640.629545 Report> ![phase2-from-202.245.63.218-to-172.20.1.0/24]
084640.629552 Report> EXCHANGE_TYPE=    QUICK_MODE
084640.629559 Report> Suites=   QM-ESP-AES-SHA2-256-PFS-SUITE
084640.629892 Report> ![from-202.245.63.218-to-200.69.110.71]
084640.629899 Report> Phase=    2
084640.629906 Report> ISAKMP-peer=      peer-200.69.110.71
084640.629912 Report> Configuration=    phase2-from-202.245.63.218-to-200.69.110.71
084640.629919 Report> Local-ID= from-202.245.63.218
084640.629926 Report> Remote-ID=        to-200.69.110.71
084640.629932 Report>
084640.629984 Report> ![phase2-from-202.245.63.218-to-200.69.110.71]
084640.629991 Report> EXCHANGE_TYPE=    QUICK_MODE
084640.629997 Report> Suites=   QM-ESP-AES-SHA2-256-PFS-SUITE
084640.630574 Report> ![from-192.168.60.0/24-to-172.20.1.0/24]
084640.630581 Report> Phase=    2
084640.630588 Report> ISAKMP-peer=      peer-200.69.110.71
084640.630594 Report> Configuration=    phase2-from-192.168.60.0/24-to-172.20.1.0/24
084640.630601 Report> Local-ID= from-192.168.60.0/24
084640.630608 Report> Remote-ID=        to-172.20.1.0/24
084640.631482 Report> 200.69.110.71=    peer-200.69.110.71
084640.631802 Report> ![to-172.20.1.0/24]
084640.631809 Report> ID-type=  !IPV4_ADDR_SUBNET
084640.631815 Report> Network=  172.20.1.0
084640.631822 Report> Netmask=  255.255.255.0
084640.632571 Report> ![phase1-peer-200.69.110.71]
084640.632577 Report> Transforms=       AES-SHA-RSA_SIG
084640.632584 Report> EXCHANGE_TYPE=    ID_PROT
084640.632827 Report> ![peer-200.69.110.71]
084640.632834 Report> Phase=    1
084640.632841 Report> Address=  200.69.110.71
084640.632847 Report> Configuration=    phase1-peer-200.69.110.71
084640.634052 Report> ![to-200.69.110.71]
084640.634059 Report> ID-type=  IPV4_ADDR
084640.634065 Report> Address=  200.69.110.71
</pre>


```route -n show``` también muestra las tablas de enrutamiento (ver {3}) con la afectación de IPsec por ejemplo:

<pre>
Routing tables
...
Encap:
Source             Port  Destination        Port  Proto SA(Address/Proto/Type/Direction)
200.69.110.71/32   0     202.245.63.218/32  0     0     200.69.110.70/esp/use/in
202.245.63.218/32  0     200.69.110.71/32   0     0     200.69.110.70/esp/require/out
172.20.1/24        0     202.245.63.218/32  0     0     200.69.110.70/esp/use/in
202.245.63.218/32  0     172.20.1/24        0     0     200.69.110.70/esp/require/out
200.69.110.71/32   0     192.168.60/24      0     0     200.69.110.70/esp/use/in
192.168.60/24      0     200.69.110.71/32   0     0     200.69.110.70/esp/require/out
172.20.1/24        0     192.168.60/24      0     0     200.69.110.70/esp/use/in
192.168.60/24      0     172.20.1/24        0     0     200.69.110.70/esp/require/out
...
</pre>


##Interfaz para IPsec entre Aplicaciones y Kernel

OpenBSD implementa la interfaz especificada en el  RFC 2367 para que las aplicaciones pueden operar con el motor de llaves IPsec del kernel.    

Esta interfaz sólo puede ser usada por aplicaciones ejecutadas por root, se debe crear un socket de clase ```PF_KEY``` El siguiente  diagrama ilustra la situación.

<pre>
                     +----------------------------+
                     |Servicio de manejo de claves|                             
                     +----------------------------+
                       |                 |                                      
                       |                 |                                      
                       |                 |                   Aplicaciones       
               -------![PF_KEY]---------![PF_INET]-------------------------
                       |                 |                   Kernel del SO      
               +-----------------+     +-----------------+
               | Motor de llaves |     | TCP/IP,         |                      
               |  o  SADB        |-----| incluye IPsec   |                      
               +-----------------+     |                 |
                                       +-----------------+
                                             |                                  
                                         +-----------+
                                         | Interfaz  |                          
                                         | de red    |                          
                                         +-----------+
</pre>



Para agregar una asociación de seguridad (SA) se puede usar uno de dos métodos:
# Enviar un mensaje SADB_ADD con toda la información la asociación de seguridad
# Primero enviar una solicitud del valor del Índice de Parametros de Seguridad (Security Parameters Index - SPI) usando el mensaje SADB_GETSPI y después enviar un mensaje SADB_UPDATE con la Asociación de Seguridad completa.


##Implementación de AH

Este protocolo se describe en el RFC 4302 (sucesor de 2402 y 1826) y se implementa en ```/usr/src/sys/netinet/ip_ah.h``` y ```/usr/src/sys/netinet/ip_ah.c```.   La idea es inyectar un encabezado AH a todo datagrama IP con información de autenticación en un extremo de un SA para que esta sea verificada en el otro extremo.   La autenticación se hace calculando un condensado (hash) para la carga del datagrama en el origen del paquete, el cual se agrega al encabezado AH, después cuando el datagrama llega a su destino este debe recalcular el condensado de la carga y verificar que coincide con el recibido en el encabezado AH.

El encabezado AH se define en la estructura ```struct ah``` con sus campos: encabezado siguiente ```ah_nh``` de 8 bits, tamaño de la carga ```ah_hl``` de 8 bits, reservado ```ah_rv``` de 16 bits, SPI ```ah_spi``` de 32 bits y número de secuencia ```ah_rpl``` de 32 bits; después de estos campos viene el condensado cuyo tamaño depende del algoritmo criptográfico que se use (pero que debe completarse para que sea múltiplo de 32).

En el caso de IPv4 el encabezado AH se inyecta después del encabezado IP, como dice en {5}:
<pre>
            ----------------------------------
      IPv4  |enc IP orig   |    |     |      |
            |(con opciones)| AH | TCP | Datos|
            ----------------------------------
            |<------- autenticado   -------->|
                 excepto campos mutables
</pre>

En el caso de OpenBSD la implementación de AH usa el controlador criptográfico del núcleo (directorio ```crypto```) que emplea una estructura
struct cryptop *crp donde se especifica la operación criptográfica por realizar.  Este mismo controlador también es usado por ip_esp.c que implementa ESP y por ip_ipcomp.c que implementa IPcomp (para comprimir cargas antes de encriptarlas como se especificado en el RFC 2393).


Al establecer una SA se llama a la función ```ah_init```, que recibe una especificación de los algoritmos criptográficos por usar, inicializa campos de estructuras y retorna una sesión criptográfica inicializada.

La inyección del encabezado AH en un paquete IP la hace la función ```ah_output``` (que a su vez es llamada por ```ipsp_process_packet```), empleando la función ```crypto_dispatch(crp)``` después de llenar ```crp``` con valores criptográficos acordes a la SA --y usando ```ah_output_cb``` como retrollamada (callback).

La verificación de un datagrama con AH la hace la función ```ah_input```, que llena datos de un ```crp``` y llama a la función ```crypt_dispatch``` con función de retrollamada ```ah_input_cb``` 


##La infraestructura criptográfica de OpenBSD

Ver [Infraestructura criptográfica de OpenBSD]

Justamente relacionada con esta infraestructura están las fuentes de Netsec, publicitadas por la supuesta infiltración de código para espionaje por parte del FBI:  ```dev/pci/hifn7751.c ```

##Notas varias
En el kernel los archivos en los que se usa PF_KEY son:
<pre>
./gnu/usr.bin/perl/ext/Socket/Makefile.PL
./gnu/usr.bin/perl/ext/Socket/Socket.pm
./lib/libc/gen/sysctl.3
./sbin/isakmpd/DESIGN-NOTES
./sbin/isakmpd/isakmpd.8
./sbin/isakmpd/monitor.c
./sbin/isakmpd/pf_key_v2.c
./sbin/isakmpd/pf_key_v2.h
./sbin/ipsecctl/ipsecctl.8
./sbin/ipsecctl/ipsecctl.c
./sbin/ipsecctl/pfkey.c
./sbin/route/route.c
./sbin/route/show.c
./sbin/iked/pfkey.c
./usr.bin/netstat/main.c
./usr.bin/netstat/netstat.1
./usr.bin/netstat/route.c
./usr.bin/netstat/show.c
./usr.sbin/bgpd/pfkey.c
./usr.sbin/sasyncd/monitor.c
./usr.sbin/sasyncd/pfkey.c
./usr.sbin/sasyncd/sasyncd.h
./usr.sbin/npppd/common/ipsec_util.c
./sys/conf/GENERIC
./sys/conf/files
./sys/net/pfkey.c
./sys/net/pfkeyv2.c
./sys/net/pfkeyv2.h
./sys/net/pfkeyv2_convert.c
./sys/net/route.c
./sys/netinet/ip_ipsp.h
./sys/netinet/ip_spd.c
./sys/sys/socket.h
</pre>

A su vez emplea la función pfkey_flow implementada en el mismo archivo
y que escribe en el archivo con descriptor fd siendo fd  variable
estática global que es inicializada en la función pfkey_init apuntando
a un socket.

Por otra parte en el kernel en sys/net/pfkeyv2.c y sys/net/pfkey.c se usan
sockets, por ejemplo la función pfkeyv2_send de pfkeyv2.c en un comentario dice
manejar los mensajes de espacio de usuario hacía el kernel.

Por su parte isakmpd (fuentes en directorio /usr/src/sbin/isakmpd) tiene
unas notas de diseño que explican los comandos que puede recibir por
el FIFO  /var/run/isakmpd.fifo:

c       connect         Establece conexión con un par
C       configure       Agrega o retira entradas de configuración
d       delete          Borra un SA dados identificación de galletas y mensajes 
D       debug           Cambia nivel de depuración para una clase
p       packet capture  Habilita/deshabilita captura de paquetes
r       report          Rporta información de estado del servicio
t       teardown        Derribar una conexión
Q       quit            Termina el proceso isakmpd
R       reinitialize    Reinicializa isakmpd (vuelve a leer arch. conf.)
S       report SA       Reporta información del SA a /var/run/isakmp_sa
T       teardown all    Derriba todas las conexiones de fase 2

En su fuente sbin/isakmpd/pf_key_v2.c nuevamente emplea el socket
PF_KEY del kernel (ver función pf_key_v2_open), el descriptor lo deja en la
variable global pf_key_v2_socket.

Justamente este tipo de sockets es usado en el kernel por ejemplo en la
tabla de enrutamiento (fuente sys/net/route.c) para decidir si
reenvia tráfico a la interfaz encap (por defecto enc0).

Parece entonces que la comunicación entre aplicaciones y kernel para
todo lo relacionado con IPSec se hace con sockets del dominio PF_KEY,
que toca confirmar son manejados por sys/net/pfkeyv2.c


Noto que IPSec se implenta a nivel de kernel, en el
archivo de configuración (por ejmplo /usr/src/sys/conf/GENERIC)
hay un símbolo IPSEC:

        option          IPSEC           # IPsec

que determina si se incluye soporte o no en las fuentes del kernel con
estructuras de la forma:

#if defined(IPSEC)                                                              
...
#endif                                                                          

Los sitios donde se usa en el kernel pueden encontrarse con:
# cd /usr/src/sys                                                               
# sudo find . -exec grep -l "IPSEC" {} ';'        
./compat/linux/linux_error.c
./compat/svr4/svr4_errno.c
./conf/GENERIC
./dev/pci/hifn7751.c
./dev/pci/hifn7751reg.h
./dev/pci/if_txp.c
./dev/pci/if_txpreg.h
./dev/pci/safereg.h
./dev/pci/ubsec.c
./dev/pci/ubsecreg.h
./dev/pci/ixgbe_type.h
./kern/uipc_domain.c

./net/bpf.h
./net/if.h
./net/if_bridge.c
./net/if_pfsync.c
./net/pf.c
./net/pfkeyv2.c
./net/pfkeyv2.h
./net/pfkeyv2_convert.c
./net/route.c  
./netinet/in.h
./netinet/in_pcb.c

Esta bastante esparcido.  Revisando las fuentes de ipsecctl
( /usr/src/sbin/ipsecctl/ipsecctl.c )  noto que para añadir reglas
se usan las funciones ike_ipsec_establish y pfkey_ipsec_establish.

La primera se implementa en /usr/src/sbin/ipsecctl/ike.c y lo que
hace entre otras es enviar comandos al socket /var/run/isakmpd.fifo
que debe ser manejado por isakmpd



----
##BIBLIOGRAFÍA


* {1} man ipsecctl
* {2} man ipsec
* {3} man route
* {4} RFC 2367
* {5} RFC 2402 http://www.ietf.org/rfc/rfc2402.txt
* {6} RFC 4302 http://tools.ietf.org/html/rfc4302
* {7} Angelos D. Keromytis et al. The Design of the OpenBSD Cryptographic Framework. http://www.usenix.org/event/usenix03/tech/full_papers/keromytis/keromytis_html/main.html
* {8} http://netbsd.gw.com/cgi-bin/man-cgi?crypto+4+NetBSD-current
