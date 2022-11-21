---
title: Quemar_DVD
date: 2009-09-03 12:00 UTC
tags:
---
Para quemar un DVD que tenga  los archivos ```/home/copia/a``` y ```/home/copia/b```

<pre>
sudo growisofs -Z /dev/rcd0c -R /home/copia/a /home/copia/b
</pre>

Para quemar un DVD con la imagen ISO ```dvd.iso```

<pre>
sudo growisofs -dvd-compat -Z /dev/rcd0c=dvd.iso
</pre>
