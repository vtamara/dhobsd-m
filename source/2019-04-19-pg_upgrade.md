# Actualización de PostgreSQL con `pg_upgrade` en adJ

Es un método más rápido que sacar volcados SQL, actualizar paquete y restaurar volcados.

Se documenta en el paquete `postgresql-pg_upgrade` pero con estos cambios:

1. Sacar los respaldos tipicos: i.e volcado en `pga-5.sql` y binarios copiados en `data--20180319.tar.gz`
2. Detener base anterior, mover `/var/postgresql/data` a `/var/postgresql/data-10`
3. Desinstalar paquetes de postgresql anteriores:
  ```
  pkg_delete postgresql-client postgresql-docs
  ```
4. Instalar paquetes ```postgresql-client``` y ```postgresql-server``` nuevos (no instalar ```postgresql-docs``` aún porque tiene conflicto con ```postgresql-previous```).
  ```
  PKG_PATH=. doas pkg_add ./postgresql-server-11.2p0.tgz ./postgresql-contrib-11.2.tgz
  ```
5. De extra-amd64 descargar paquetes ```postgresql-pg_upgrade``` y ```postgresql-previous``` e instalarlos.
  ```
  PKG_PATH=. doas pkg_add -D unsigned ./postgresql-previous-10.6.tgz ./postgresql-pg_upgrade-11.2.tgz
  ```
6. Generar nueva clave e iniciar nueva base así:
  ```
  doas su - _postgresql
  apg | grep -v "'" | head -n 1 > /tmp/clave.txt 
  initdb --encoding=UTF-8 -U postgres --auth=md5 --pwfile=/tmp/clave.txt  -D/var/postgresql/data
  ```
  (si no se especifica ```LC_ALL```, la bases template  inician con ```es_CO.UTF-8```)
  Dejar la clave en `/var/postgresql/.pgpass`:
  ```
  c=`cat /tmp/clave.txt`  
  echo "*:*:*:postgres:$c" > .pgpass
  ```
7. Mientras se restaura mantener configuración por omisión (no mover sockets) y cambiar pg_hba.conf de data y de data-10 para modificar
  ```
  local all all md5
  ```
  por
  ```
  local all all trust
  ```
8. Iniciar restauración así:
  ```
  doas su - _postgresql
  pg_upgrade -b /usr/local/bin/postgresql-10/ -B /usr/local/bin -U postgres -d /var/postgresql/data-10/ -D /var/postgresql/data
  ```
Si llega a fallar con:
  ```
   $ pg_upgrade -b /usr/local/bin/postgresql-10/ -B /usr/local/bin -U postgres -d /var/postgresql/data-10/ -D /var/postgresql/dat
   Checking for presence of required libraries fatal
  Your installation references loadable libraries that are missing from the
  new installation. You can add these libraries to the new installation,
  or remove the functions using them from the old installation. A list of
  problem libraries is in the file:
  loadable_libraries.txt
  ```
  Seguramente faltó instalar `postgresql-contrib` que incluye `accent` y otros módulos.  Instalar y repetir
9. Iniciar nueva base con configuración por omisión de manera temporal
10. Actualizar estadísticas con 
  ```
  ./analyze_new_cluster.sh
  ```
11. Detener nuevamente servicio postgresql, modificar `/var/postgresql/data/postgresql.conf` para cambiar ubicación del socket y activar tcp en localhost 5432
  ```
  unix_socket_directories = '/var/www/var/run/postgresql' # comma-separated list of directories
  ```
  En `data/pg_hba.conf` volver a dejar `md5` en lugar de `trust`
12. Iniciar servicio y comprobar operación
13. Una vez se complete con éxito se puede eliminar el cluster anterior
