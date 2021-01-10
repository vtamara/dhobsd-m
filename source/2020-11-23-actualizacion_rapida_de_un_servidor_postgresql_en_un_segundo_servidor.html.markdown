---

title: Actualización rápida de un servidor PostgreSQL en un segundo servidor
date: 2020-11-23 16:00 UTC
tags: postgresql, alta disponibilidad

---

En sistemas de alta disponibilidad es necesario minimizar los tiempos 
de los mantenimientos, incluyendo la actualización.

En el caso de un servidor de bases de datos con PostgreSQL, una actualización 
del sistema operativo adJ que incluye actualización a la pila de software (con
PostgreSQL) puede tomar entre dos horas y varios días dependiendo de la 
cantidad de información en la base de datos y del método que se use
para actualizar (es crítico cuando se requiere actualizar la versión
mayor del motor de PostgreSQL, pues de usar `pg_dumpall` se requiere primero
volcado de la versión anterior y después restauración del volcado 
en la nueva versión).

Con un segundo servidor que pueda actualizarse primero para tomar el rol
de servidor principal, podría disminuirse el tiempo, pero no es trivial
como se explora en este documento, con mediciones bajo las siguientes
circunstancias

* Servidor fuente:
** Sistema operativo: OpenBSD/adJ 6.6
** Memoria: 32G
** Procesador: Intel Core i7-7700
** Versión de PostgreSQL: 12.2
** Tamaño de la base de datos: 22G medidos desde el sistema operativo 
con `du -sh /var/postgresql/data`.
* Servidor destino:
** Sistema operativo: OpenBSD/adJ 6.7
** Memoria: 32G
** Procesador: Intel Core i7-4790 
** Versión de PostgreSQL: 12.2


# Con `pg_dumpall` y `pg_dump`

Al sacar copias de respaldo con PostgreSQL hemos notado que el método más 
portable entre diversas versiones de PostgreSQL es el volcado que genera
`INSERT`s en SQL (con opciones `--inserts` y `--column-inserts` de `pg_dump` 
y de `pg_dumpall`).


| Opciones para `pg_dumpall` | Tamaño del volcado | Tiempo de volcado | Transferencia a 2do servidor | Velocidad |
|--|--|--|--|
| Ninguna | 12.7G | 317s | .040 G/s |
| `--inserts --columns-inserts` | 14.4G | 365s | .039 G/s |
| `--inserts` |  13.2G | 371s |  .035 G/s |
| `--column-inserts` |  14.4G | 364s | .039 G/s |


Según [SOR2020] las opciones `-j` y `-F` de `pg_dump` permitirían
mayor velocidad en la restauración de una base de datos con 
`pg_restore`.  Pero como `pg_dumpall` no soporta tales opciones
inicialmente hacemos una medición del uso de `pg_dump`. 
Cómo `pg_dump` sólo permite sacar  volcado de una base
de datos hicimos inicialmente la medición sacando copia de 
una base de 1.2G medida desde `psql` con 
`select pg_database_size('nombrebase');`

| Opciones para `pg_dump` | Tamaño del volcado | Tiempo de volcado | Velocidad |
|--|--|--|--|
| Ninguna | 6.9G | 187s | 0.03 G/s |
| `--column-inserts` | 7.3G | 183s |  0.03G/s |
| `--inserts` |  6.9G | 193s | 0.03 G/s |
| `-Fc` | 1.5G | 293s | 0.005 G/s |
| `-j 8 -Fd -f /tmp/volcado-pg8.dir` | 1.5G | 323s | 0.004 G/s |
| `-j 32 -Fd -f /tmp/volcado-pg32.dir` | 1.5G | 323s | 0.004 G/s |


Aunque son resultados desalentadores, con la esperanza que `pg_restore`
sea radicalmente más rápido que `psql` restaurando un volcado de `pg_dumpall`,
se hace un script que copia objetos globales con `pg_dumpall` y que 
copia una a una las bases de datos con `pg_dump`, es decir haría lo mismo que
`pg_dumpall`, según su documentación --ver [PGD13], pero con las opciones 
`-j 32 -Fd` que sería la más rápida para `pg_dump` y la que permitirìa
el `pg_restore` más veloz.


llama a `pg_dump` de forma reiterada y copia objetos gloables, notamos 
que `pg_dumpall` no soporta las opciones `-j` ni `-F` de `pg_dump` que 
bases por separado con esas opciones y las opciones globales con
`pg_dumpall` se impcomo setambién se copian
los objetos globales del cluster 

```
#!/bin/sh

rm -rf /tmp/pgd/*
mkdir -p /tmp/pgd/
opp="-h /var/www/var/run/postgresql/ -U postgres"
pg_dumpall $opp --globals-only > /tmp/pgd/globales.sql
for i in `psql $opp template1 \
  -c "SELECT datname FROM pg_database WHERE datname NOT IN ('template0',
  'template1', 'postgres') ORDER BY 1" | \
  grep -v datname | grep -v "rows)" | grep -v "\----" `; do
    echo $i;
    pg_dump $opp -C -j 32 -Fd -f /tmp/pgd/$i $i
done
```

Esta operación se completa en 12m3s i.e 723s y da
un archivo de 2.9G (i.e velociad de 0.04).

Toda la estructura de directorios se transmite al otro servidor
en 55s.



que podría ser se intentan  copiar todas las bases que se listan excluyendo
template0, template1 y postgres para un total de 23:

Para poder comparaar con pg_dumpall se intentaSe intenta con todo


# Referencias

* [PGD13] https://www.postgresql.org/docs/13/app-pg-dumpall.html
* [SOR2020] https://stackoverflow.com/questions/2094963/postgresql-improving-pg-dump-pg-restore-performance




