---
title: Comparacion de navegadores tipo texto
date: 2015-06-26 01:17 UTC
tags:
---

El venerable lynx salió del sistema base en 5.6 por eso miramos alternativas
y las comparamos con lynx.

|                      | lynx       | w3m        | links      | elinks     |
| Licencia             | GPL        | MIT        | GPL        |  GPL       | 
| Soporta UTF-8        | No         | Si         | No(1)      |  Si        | 
| Descargas grandes    | Si         | No(2)      | Si         |  Si        | 
| Localizado en español| No         | No         | Si         |  Si        | 
| Colores              | No         | Si         | No         |  Si(3)     | 
| Fecha último         | 2014-03-09 | 2011-01-15 | 2014-12-24 | 2012-10-30 | 


* (1) links convierte vocales tildadas del español a no tildadas.
* (2) Hemos evidenciado fallas en w3m en adJ 5.6 al descargar archivos grandes (digamos 10M) que detienen la aplicación.
* (3) Es posible configrar elinks para presentar colores en tmux (siempre que se
defina export TERM=screen), xterm y consolas virtuales, agregando 
a ~/.elinks/elinks.conf con:
<pre>
set config.saving_style_w = 1
set terminal.xterm-color.utf_8_io = 1
set terminal.xterm-color.colors = 2
set terminal.xterm-color.charset = "utf-8"
set terminal.xterm-color.type = 1
set terminal.screen.utf_8_io = 1
set terminal.screen.colors = 2
set terminal.screen.charset = "utf-8"
set terminal.screen.type = 1
set terminal.wsvt25.utf_8_io = 0
set terminal.wsvt25.colors = 1
set terminal.wsvt25.charset = "ISO-8859-1"
set terminal.wsvt25.type = 0
set ui.language = "System"
</pre>

