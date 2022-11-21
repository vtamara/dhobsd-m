---
title: bundler doas
date: 2016-03-30 12:00 UTC
tags:
---

A partir de OpenBSD y adJ 5.8 se eliminó ```sudo``` del sistema base (sigue como 
paquete).  Se ha remplazó por ```doas```, pero ```bundler``` (que en Ruby
ayuda a crear gemas y a manejar gemas de una aplicación) típicamente usa 
```sudo```. 

## 1. Usar ```sudo```
Basta instalar el paquete como usuario root:

```
su -
pkg_add sudo
```

## 2. Dejar paquetes en directorio escribible por usuario 

Primer tenga en cuenta que puede instalar gemas que se requieren 
globalmente (como bundler y rails) con el procedimiento habitual, por ejemplo:

```
doas gem install bundler
```
los binarios quedarán en ```/usr/local/bin``` pero con posfijo 23 (o la 
versión de ruby), así que asegurse  de hacer enlaces, por ejemplo:
```
doas ln -sf /usr/local/bin/bundler23 /usr/local/bin/bundler
doas ln -sf /usr/local/bin/bundle23 /usr/local/bin/bundle
```

Para evitar problemas de permisos con ```bundler```, asegurese de 
configurar la ruta ```~/.bundler``` como destino de gemas  instaladas 
con bundler en ```~/.bundle/config```, lo hace rápidamente con: 
```
bundler config path ~/.bundler
```

De esta forma las gemas que se instalen con ```bundler``` tendrán como 
ruta de instalación ```~/.bundler/``` 
por lo que allí se crearán directorios como ```bin```, ```extensions``` 
y ```gems```

En general ```bundle install``` hará su labor bien, excepto para gemas que 
requieran ```sudo``` (como ```bcrypt```, ```pg```) o para las que requieran 
opciones de configuración especiales (como ```nokogiri```).

Para las que requieren ```sudo``` lo que puede hacer es instalarlas con 
```gem```  pero en el directorio apropiado:

```
doas gem install --install-dir ~/.bundler/ pg -v '0.18.4'
```

Para las que requieren opciones especiales, recomendamos almacenar las opciones 
como parte de la configuración de bundle:

```
bundle config build.nokogiri --use-system-libraries --with-xml2-config=/usr/local/bin/xml2-config
```
y posteriormente instalar:

```
doas gem install --install-dir ~/.bundler/ nokogiri -v '1.6.7.2'
```


