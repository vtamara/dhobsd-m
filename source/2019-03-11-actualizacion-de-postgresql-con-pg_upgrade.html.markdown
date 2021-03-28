---

title: Actualización de PostgreSQL con pg_upgrade
date: 2019-03-11 15:55 UTC
tags: 

---

# Actualización de PostgreSQL con `pg_upgrade` en adJ

Documento actualizado: 2021-03-28

Es un método más rápido que el clásico (sacar volcados SQL, actualizar paquete y restaurar volcados).

Se documenta en el paquete `postgresql-pg_upgrade`, que para el caso de adJ pueden aplicarse de la siguiente forma:

1. Sacar los respaldos tipicos: i.e si está usando inst-adJ permitir que saque volcado en `pga-5.sql` y binarios copiados en `data--20200319.tar.gz` y detener cuando pregunte "Desea eliminar la actual versión de PostgreSQL"
2. En casos excepcionales, preparar datos.  No se requirido entre versiones 9-10, 10-11, pero de la 11 a la 12 debe quitar columnas OIDS de las diversas tablas con `ALTER TABLE x SET WITHOUT OIDS;`.  Primero identificar bases y crear un script que por cada base llame a otro script que borre oids:

```
    $ doas su - _postgresql
    $ psql -U postgres -h /var/www/var/run/postgresql/
    postgres=# \t on
    postgres=# \o /tmp/quitaoids.sh
    postgres=# SELECT '/usr/local/adJ/pg_quita_oids.sh ' || datname FROM pg_database WHERE datname NOT IN ('template0', 'template1', 'postgres');
    postgres=# \q
```
note que se exluyen las bases postgres, template0, template1.  El script `/usr/local/adJ/pg_quita_oids.sh` está disponible en `https://github.com/pasosdeJesus/adJ/blob/master/arboldd/usr/local/adJ/pg_quita_oids.sh`.  Una vez lo copie ejecute:

```
    $ sh /tmp/quita_oids.sh
```

3. Detener base anterior (digamos con `doas rcctl stop postgresql`) y  mover `/var/postgresql/data` a `/var/postgresql/data-12` (con `doas mv /var/postgresql/data /var/postgresql/data-12` )
4. Desinstalar paquetes de postgresql anteriores:
  ```
  doas pkg_delete postgresql-client postgresql-docs
  ```
5. Instalar paquetes `postgresql-client`, `postgresql-server`, `postgresql-contrib`, `postgresql-previous` y `postgresql-pg_upgrade` (inicialmente no instalar `postgresql-docs` porque tiene conflicto con `postgresql-previous`).
  ```
  cd 6.8-amd64/paquetes
  PKG_PATH=. doas pkg_add ./postgresql-server-13.2.tgz ./postgresql-contrib-13.2.tgz postgresql-previous-12.5.tgz postgresql-pg_upgrade-13.2.tgz
  ```
  Si está corriendo una versión de adJ anterior a la 6.6 puede encontrar los paquetes `postgresql-previous` y `postgresql-pg_upgrade` en  <http://adj.pasosdejesus.org/pub/AprendiendoDeJesus/> en un directorio de la forma `6.5-extra`. Al momento de insalarlo con `pkg_add` use la opción `-D unsigned`).

6. Iniciar nueva base con clave de administrador de la anterior (suponiendo que está en el archivo `.pgpass` de la cuenta `_postgresql` como ocurre por omisión en adJ) con:
  ```
  doas su - _postgresql
  grep postgres .pgpass |  sed  -e  "s/.*://g" > /tmp/clave.txt
  initdb --encoding=UTF-8 -U postgres --auth=md5 --pwfile=/tmp/clave.txt  -D/var/postgresql/data
  ```
7. Mientras se restaura mantener configuración por omisión (no mover sockets) y cambiar `pg_hba.conf` de `data` y de `data-12` para modificar
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
  pg_upgrade -b /usr/local/bin/postgresql-12/ -B /usr/local/bin -U postgres -d /var/postgresql/data-12/ -D /var/postgresql/data
  ```
  Si llega a fallar con:
  ```
   $ pg_upgrade -b /usr/local/bin/postgresql-12/ -B /usr/local/bin -U postgres -d /var/postgresql/data-12/ -D /var/postgresql/data
   Checking for presence of required libraries fatal
  Your installation references loadable libraries that are missing from the
  new installation. You can add these libraries to the new installation,
  or remove the functions using them from the old installation. A list of
  problem libraries is in the file:
  loadable_libraries.txt
  ```
  Seguramente faltó instalar `postgresql-contrib` que incluye `accent` y otros módulos.  Instalar y repetir
  
9. Iniciar nueva base con configuración por omisión de manera temporal con  `doas rcctl start postgresql`

10. Actualizar estadísticas con 
  ```
  ./analyze_new_cluster.sh
  ```
11. Asegurar nueva clave.  Revisela con `cat /tmp/clave.txt` y establezcala con:
  ```
  psql -U postgres template1
  alter user postgres with password 'nuevaaqui';
  ```
12. Detener nuevamente servicio postgresql, modificar `/var/postgresql/data/postgresql.conf` para cambiar ubicación del socket y en general rehacer la configuración que tenía su base (e.g conexiones TCP, llaves, etc).
  ```
  unix_socket_directories = '/var/www/var/run/postgresql' # comma-separated list of directories
  ```
  En `data/pg_hba.conf` volver a dejar `md5` en lugar de `trust`
  
13. Iniciar servicio y comprobar operación

14. Una vez se complete con éxito se puede eliminar el cluster anterior
