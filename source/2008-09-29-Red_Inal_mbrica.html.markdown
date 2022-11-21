---
title: Red_Inal_mbrica
date: 2008-09-29 12:00 UTC
tags:
---
Se describe como configurar una red con WEP, usando adJ 4.2p1 y una tarjeta Intel PRO/Wireless 3945ABG.

!Configuración

Con ```dmesg``` esta tarjeta se ve como:
<pre>
wpi0 at pci3 dev 0 function 0 "Intel PRO/Wireless 3945ABG" rev 0x02: irq 10, MoW1, address 00:1b:77:d6:52:ee
</pre>

Como se explica en ```man wpi``` esta tarjeta tiene firmware que puede
instalarse con:
<pre>
sudo pkg_add     http://damien.bergamini.free.fr/packages/openbsd/wpi-firmware-2.14.1.5.tgz
</pre>


Una vez instalado al ejecutar ```ifconfig``` no se ve activa la interfaz wpi, y esto ocurrirá mientras no se establezca la identificación de la red, por ejemplo con:
<pre>
sudo ifconfig wpi0 nwid MIRED nwkey !0x123498a2d2
</pre>
una vez se vea activa la interfaz, en caso de un enrutador inalámbrico que resuelva con DHCP:
<pre>
sudo dhclient wpi0
</pre>

Tras esto al examinar con ```ifconfig``` se ve algo como:
<pre>
wpi0: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> mtu 1500
        lladdr 00:1b:77:d6:52:ee
        groups: wlan egress
        media: IEEE802.11 autoselect (OFDM36 mode 11g)
        status: active
        ieee80211: nwid MIRED chan 11 bssid 00:17:3f:ba:40:4a 56dB nwkey <not displayed> !100dBm
        inet6 fe80::21b:77ff:fed6:52ee%wpi0 prefixlen 64 scopeid 0x1
        inet 192.168.2.5 netmask 0xffffff00 broadcast 192.168.2.255
</pre>



Esta información se cede al dominijo público.  Se dedica al TODOPODEROSO y GLORIOSO.  2008. vtamara@pasosdeJesus.org
