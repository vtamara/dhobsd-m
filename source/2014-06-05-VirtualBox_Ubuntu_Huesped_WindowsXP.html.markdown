---
title: VirtualBox_Ubuntu_Huesped_WindowsXP
date: 2014-06-05
tags:
---

Para que el escritorio quede en la carpeta compartida por virtualbox (digamos ~/win).

1.  Como dice http://superuser.com/questions/177326/windows-xp-how-to-move-desktop-and-favorites-locations use XP Powertoys para cambiar el escritorio del usuario a la carpeta \\VBOXSVR\win 

2. Para que tenga buena velocidad usar el controlador de red Intel PRO/1000MT y como dice http://forum.notebookreview.com/linux-compatibility-software/421811-howto-speed-up-shared-folder-access-time-virtualbox-windows-guest.html cambiar en c:\windows\system32\drivers\etc\hosts 
<pre>
127.0.0.1 localhost
</pre>
por
<pre>
127.0.0.1 localhost vboxsvr
</pre>
