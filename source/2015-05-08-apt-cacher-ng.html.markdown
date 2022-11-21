---
title: apt-cacher-ng
date: 2015-05-08 12:00 UTC
tags:
---
# Cache de paquetes Ubuntu en una red local con ```apt-cacher-ng```


Para hacer más rápida la descarga de paquetes de Ubuntu vale la pena configurar uno de los computadores de la red local (que llamaremos sevidor en este escrito)
para que sea repositorio de paquetes y les sirva a los demás computadores de la red.

## 1. CONFIGURACIÓN EN SERVIDOR
En el computador que tendrá el repositorio ejecute (según instrucciones de {1}):

<pre>
sudo apt-get update
sudo apt-get install -udV apt-cacher-ng
sudo apt-get install apt-cacher-ng
sudo sh /etc/init.d/apt-cacher-ng restart
</pre>

En el computador donde corre ```apt-cacher``` podrá ver el repositorio local en: ```/var/cache/apt-cacher-ng/```

Y con un navegador podrá comprobar su operación visitando el puerto 3142, por ejemplo
http://127.0.0.1:3142

! 1.1 Agregar paquetes ya descargados

Para agregar paquetes ya descargados en el servidor de apt-cacher-ng (tomado de {4}):

<pre>
test -x /var/cache/apt-cacher-ng/_import || sudo mkdir -p -m 2755 /var/cache/apt-cacher-ng/_import
sudo mv -uf /var/cache/apt/archives/*.deb /var/cache/apt-cacher-ng/_import/
sudo chown -R apt-cacher-ng.apt-cacher-ng /var/cache/apt-cacher-ng/_import
</pre>

Antes de importar los paquetes configure algún cliente y actualice índices.  

Después abra en el servidor el puerto 4132 con algún navegador. Digamos desde el mismo servidor ```http://127.0.0.1:3142```  vaya al enlace ```Statistics report and configuration page``` y al final de la página presione el botón ```Import```.

Tomará un tiempo mientras identifica cada archivo con la entrada que le corresponde en el índice.


## 2. CONFIGURACIÓN EN CLIENTES

Después en cada cliente de la red local (incluyendo el mismo donde estará el repositorio) debe configurarse el uso del cache:

* Configuración en Synaptic: ingrese a Synaptic al menu Configuración -> Preferencias, pestaña Red y eliga un proyx, la dirección será la del computador donde corre apt-cacher y el puerto 3142 (sin autenticación).

* Configuración de apt-get: ejecute en una terminal:
<pre>
echo 'Acquire::http { Proxy "http://192.168.190.25:3142"; };' | sudo tee /etc/apt/apt.conf.d/01apt-cacher-ng-proxy
</pre>

remplazando 192.168.190.25 por la IP del computador donde correo el Cache. Y  a continuacion actualice indices con ```sudo apt-get update```


