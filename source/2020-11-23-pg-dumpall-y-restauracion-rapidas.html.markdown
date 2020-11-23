---

title: pg_dumpall y restauracion rápidas
date: 2020-11-23 16:00 UTC
tags: postgresql

---

Al sacar copias de respaldo con PostgreSQL hemos notado que el método más 
portable entre diversas versiones de PostgreSQL es el volcado que genera
SQL (con opciones `--inserts` y `--column-inserts` de `pg_dump` o 
`pg_dumpall`).

Sin embargo ese no es el método más rápido. Para buscar el más veloz
hicimos mediciones en un sistema con las siguientes características:

* Sistema opeativo : distribución adJ 6.6 de OpenBSD 
* Memoria: 32G
* Procesador: Intel Core i7-7700
* Versión de PostgreSQL: 12.2

Notamos que `pg_dumpall` no soporta las opciones `-j` ni `-F` de `pg_dump` por
lo que haremos mediciones separadas.

Para el caso de `pg_dumpall` hicimos medición con un grupo de bases de 
datos de 22G medidos desde el sistema operativo 
con `du -sh /var/postgresql/data`.

| Opciones para `pg_dumpall` | Tamaño del volcado | Tiempo de volcado | 
|--|--|--|
| Ninguna | 12.7G | 317s | 
| `--inserts --columns-inserts` | 14.4G | 365s |
| `--inserts` |  13.2G | 371s |
| `--column-inserts` |  14.4G | 364s |

Para el caso de `pg_dump` hicimos la medición sacando copia de una base
de datos de 1.2G medida desde `psql` con `select pg_database_size('nombrebase');`

| Opciones para `pg_dump` | Tamaño del volcado | Tiempo de volcado | 
|--|--|--|
| | 12.7G | 317s | 
| `--inserts --columns-inserts` | 14.4G | 360s |
| `--inserts` |  14.4G | 360s |
| `--column-inserts` |  14.4G | 360s |
| -j 8 -Fd -f /tmp/volcado-pg.dir | 

Siguiendo
https://stackoverflow.com/questions/2094963/postgresql-improving-pg-dump-pg-restore-performance




