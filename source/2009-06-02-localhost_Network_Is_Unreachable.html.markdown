---
title: localhost_Network_Is_Unreachable
date: 2009-06-02
tags:
---
¿ En OpenBSD Alguna vez le ha fallado ```ping 127.0.0.1``` ?

!Diagnóstico:

```ping 127.0.0.1``` reporta  ```Network is Unreachable```

por otra parte ```route -n show```  no muestra la entrada:

<pre>
127.0.0.1          127.0.0.1          UH         2     5926 33160    48 lo0
</pre>

!Solución

En ```/etc/rc.local``` agregar:

<pre>
route -n show  | grep "127.0.0.1.*127.0.0.1" > /dev/null 2> /dev/null
if (test "$?" != "0") then {
        sudo route add 127.0.0.1 127.0.0.1
} fi;
</pre>

----
Dominio público.  Dedicado al Señor que me da Gozo y Paz. vtamara@pasosdeJesus.org
