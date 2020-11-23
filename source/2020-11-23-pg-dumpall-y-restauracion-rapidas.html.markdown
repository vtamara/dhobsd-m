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

| Opciones para `pg_dumpall` | Tamaño del volcado | Tiempo de volcado | Velocidad |
|--|--|--|--|
| Ninguna | 12.7G | 317s | .040 G/s |
| `--inserts --columns-inserts` | 14.4G | 365s | .039 G/s |
| `--inserts` |  13.2G | 371s |  .035 G/s |
| `--column-inserts` |  14.4G | 364s | .039 G/s |

Para el caso de `pg_dump` hicimos inicialmente la medición sacando copia de una sola base
de datos de 1.2G medida desde `psql` con `select pg_database_size('nombrebase');`

| Opciones para `pg_dump` | Tamaño del volcado | Tiempo de volcado | Velocidad |
|--|--|--|--|
| Ninguna | 6.9G | 187s | 0.03 G/s |
| `--column-inserts` | 7.3G | 183s |  0.03G/s |
| `--inserts` |  6.9G | 193s | 0.03 G/s |
| `-j 8 -Fd -f /tmp/volcado-pg8.dir` |   |  | |


# Referencias

* https://stackoverflow.com/questions/2094963/postgresql-improving-pg-dump-pg-restore-performance




