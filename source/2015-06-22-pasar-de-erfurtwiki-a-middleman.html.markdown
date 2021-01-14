---
title: Pasar de Erfurtwiki a Middleman
date: 2015-06-22 21:22 UTC
tags:
---

# Middleman

Middleman es un entorno para generación de páginas estatícas.  Entre sus 
ventajas contamos:

- Emplea herramientas tipicas de rails
- Permite generar facilmente un blog
- Los artículos del blog se escriben en Markdown
- Tiene amplia [documentacion](https://middlemanapp.com/basics/blogging/)

# Conversión de Erfurtwiki a Middleman

Para convertir de Erfurtwiki (corriendo en PostgreSQL) a
Middleman hicimos esto:

* Sacamos un volcado de articulos de la base PostgreSQL en formato CSV:
<pre>
COPY (SELECT DISTINCT pagename, 
    (TIMESTAMP 'epoch' + created * INTERVAL '1 SECOND')::DATE, 
    version, content 
  FROM ewiki 
  WHERE (pagename, version) IN (
    SELECT pagename, max(version) FROM ewiki GROUP BY 1) ORDER BY 2) 
  TO '/tmp/e' with csv;  
</pre>
* Se limpió el CSV generado
* Se convirtieron los CSV a archivos individuales con una minimas conversiones a Markdown con:
<pre>
require 'csv'
def ew_a_md(ew)
  md = ew.gsub("==", '```')
  md.gsub!("!!!", "#")
  md.gsub!("!!", "##")
  md.gsub!("^\!", "###")
  return md
end

CSV.foreach "/tmp/e" do |row|
  t = row[0].gsub(/[^0-9A-Za-z.\-]/, '_')
  f = row[1]
  n = f + "-" + t + ".html.markdown"
  puts n
  io = open n, "w"
  io.puts "---"
  io.puts "title: #{t}"
  io.puts "date: #{f}"
  io.puts "tags:"
  io.puts "---"
  io.puts ew_a_md row[3]
  io.close
end
</pre>

# Uso tipico

Para crear un articulo de titulo "Compracion de Navegadores":
middleman article "Comparacion de Navegadores"

