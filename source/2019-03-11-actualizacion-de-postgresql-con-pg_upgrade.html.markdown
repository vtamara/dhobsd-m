---

title: Actualización de PostgreSQL con pg_upgrade
date: 2019-03-11 15:55 UTC
tags: 

---

# Actualización de PostgreSQL con `pg_upgrade` en adJ

Es un método más rápido que el clásico (sacar volcados SQL, actualizar paquete y restaurar volcados).

Se documenta en el paquete `postgresql-pg_upgrade`, aunque para adJ 6.6 aplican los siguientes cambios:

1. Sacar los respaldos tipicos: i.e si está usando inst-adJ permitir que saque volcado en `pga-5.sql` y binarios copiados en `data--20200319.tar.gz` y detener cuando pregunte "Desea eliminar la actual versión de PostgreSQL"
2. En casos excepcionales, preparar datos.  No se requirió entre versiones 9-10, 10-11, pero de la 11 a la 12 debe quitar columnas OIDS de las diversas tablas con `ALTER TABLE x SET WITHOUT OIDS;`.  Puede ser, primero identificar bases con:
    ```
    $ doas su - _postgresql
    $ psql -U postgres -h /var/www/var/run/postgresql/
    postgres=# SELECT datname FROM pg_database;
    ```
y por cada base (excepto postgres, template0, template1) ejecutar:

    ```
    \c base
    base=# psql -U postgres -h /var/www/var/run/postgresql/
    SELECT format(
      'ALTER TABLE %I.%I.%I SET WITHOUT OIDS;',
      table_catalog,
      table_schema,
      table_name
    ) 
    FROM information_schema.tables
    WHERE table_schema = 'public' and table_type='BASE TABLE';
    base=# \gexec
    ```

3. Detener base anterior (digamos con `doas rcctl stop postgresql`) y  mover `/var/postgresql/data` a `/var/postgresql/data-11` (con `doas mv /var/postgresql/data /var/postgresql/data-11` )
4. Desinstalar paquetes de postgresql anteriores:
  ```
  doas pkg_delete postgresql-client postgresql-docs
  ```
5. Instalar paquetes `postgresql-client`, `postgresql-server` y `postgresql-contrib` nuevos (inicialmente no instalar `postgresql-docs` porque tiene conflicto con `postgresql-previous`).
  ```
  PKG_PATH=. doas pkg_add ./postgresql-server-12.2p0.tgz ./postgresql-contrib-12.2.tgz
  ```
6. De <http://adj.pasosdejesus.org/pub/AprendiendoDeJesus/6.6-extra/> descargar paquetes ```postgresql-pg_upgrade``` y ```postgresql-previous``` e instalarlos.
  ```
  PKG_PATH=. doas pkg_add -D unsigned ./postgresql-previous-11.6.tgz ./postgresql-pg_upgrade-12.2.tgz
  ```
7. Iniciar nueva base con clave de administrador de la anterior (suponiendo que está en el archivo `.pgpass` de la cuenta `_postgresql` como ocurre por omisión en adJ) con:
  ```
  doas su - _postgresql
  grep postgres .pgpass |  sed  -e  "s/.*://g" > /tmp/clave.txt
  initdb --encoding=UTF-8 -U postgres --auth=md5 --pwfile=/tmp/clave.txt  -D/var/postgresql/data
  ```
8. Mientras se restaura mantener configuración por omisión (no mover sockets) y cambiar `pg_hba.conf` de `data` y de `data-11` para modificar
  ```
  local all all md5
  ```
  por
  ```
  local all all trust
  ```
9. Iniciar restauración así:
  ```
  doas su - _postgresql
  pg_upgrade -b /usr/local/bin/postgresql-11/ -B /usr/local/bin -U postgres -d /var/postgresql/data-11/ -D /var/postgresql/data
  ```
  Si llega a fallar con:
  ```
   $ pg_upgrade -b /usr/local/bin/postgresql-11/ -B /usr/local/bin -U postgres -d /var/postgresql/data-11/ -D /var/postgresql/data
   Checking for presence of required libraries fatal
  Your installation references loadable libraries that are missing from the
  new installation. You can add these libraries to the new installation,
  or remove the functions using them from the old installation. A list of
  problem libraries is in the file:
  loadable_libraries.txt
  ```
  Seguramente faltó instalar `postgresql-contrib` que incluye `accent` y otros módulos.  Instalar y repetir
  
10. Iniciar nueva base con configuración por omisión de manera temporal

11. Actualizar estadísticas con 
  ```
  ./analyze_new_cluster.sh
  ```
12. Asegurar nueva clave.  Revisela con `cat /tmp/clave.txt` y establezcala con:
  ```
  psql -U postgres template1
  alter user postgres with password 'nuevaaqui';
  ```
13. Detener nuevamente servicio postgresql, modificar `/var/postgresql/data/postgresql.conf` para cambiar ubicación del socket y en general rehacer la configuración que tenía su base (e.g conexiones TCP, llaves, etc).
  ```
  unix_socket_directories = '/var/www/var/run/postgresql' # comma-separated list of directories
  ```
  En `data/pg_hba.conf` volver a dejar `md5` en lugar de `trust`
  
14. Iniciar servicio y comprobar operación

15. Una vez se complete con éxito se puede eliminar el cluster anterior
