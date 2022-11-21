---
title: Busqueda_de_Textos_Completos_en_Dovecot
date: 2013-09-11 12:00 UTC
tags:
---

Método 1 con fts:

Editar /etc/dovecot/conf.d/90-plugin.conf  y poner:

mail_plugins = $mail_plugins fts fts_squat

plugin {
        fts = squat
        fts_squat = partial=4 full=10
}


Método 2 con lucene:

sudo pkg_add clucene-core

Instalar dovecot de adJ que incluye soporte para clucene.

Editar /etc/dovecot/conf.d/90-plugin.conf  y poner:

mail_plugins = $mail_plugins fts fts_lucene

<pre>
plugin {
  fts = lucene
  # Lucene-specific settings, good ones are:
  fts_lucene = whitespace_chars=@.
}
</pre>
