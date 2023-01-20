---

title: Actualización de PostgreSQL con pg_upgrade
date: 2019-03-11 15:55 UTC
tags:

---

# Actualización de PostgreSQL con `pg_upgrade` en adJ

Documento actualizado: 2023-01-03

Es un método más rápido que el clásico (sacar volcados SQL, actualizar
paquete y restaurar volcados).

Se documenta en el paquete `postgresql-pg_upgrade`, que para el caso de adJ
pueden aplicarse de la siguiente forma:

1. Saca los respaldos típicos: i.e si está usando inst-adJ permitir que
   saque volcado en `pga-5.sql` y binarios copiados en `data--20200319.tar.gz`
   y detener cuando pregunte "Desea eliminar la actual versión de PostgreSQL"

2. En casos excepcionales, prepara datos.  No se requirió entre versiones
   9-10, 10-11, pero de la 11 a la 12 debe quitar columnas OIDS de las
   diversas tablas con `ALTER TABLE x SET WITHOUT OIDS;`.  Primero
   identificar bases y crear un script que por cada base llame a otro script
   que borre oids:

        $ doas su - _postgresql
        $ psql -U postgres -h /var/www/var/run/postgresql/
        postgres=# \t on
        postgres=# \o /tmp/quitaoids.sh
        postgres=# SELECT '/usr/local/adJ/pg_quita_oids.sh ' || datname FROM pg_database WHERE datname NOT IN ('template0', 'template1', 'postgres');
        postgres=# \q

   note que se exluyen las bases postgres, template0, template1.
   El script `/usr/local/adJ/pg_quita_oids.sh` está disponible en
   <https://github.com/pasosdeJesus/adJ/blob/master/arboldd/usr/local/adJ/pg_quita_oids.sh>.

   Una vez lo copie ejecute:

        $ sh /tmp/quita_oids.sh

3. Detén base anterior (digamos con `doas rcctl stop postgresql`) y
   mueve `/var/postgresql/data` a `/var/postgresql/data-14`
   (con `doas mv /var/postgresql/data /var/postgresql/data-14` )

4. Desinstala los paquetes de postgresql anteriores:

        doas pkg_delete postgresql-client postgresql-docs

5. Instala paquetes `postgresql-client`, `postgresql-server`,
   `postgresql-contrib`, `postgresql-previous` y `postgresql-pg_upgrade`
   (inicialmente no instales `postgresql-docs` porque tiene conflicto con
    `postgresql-previous`).

        cd 7.2-amd64/paquetes
        PKG_PATH=. doas pkg_add ./postgresql-server-15.1.tgz \
          ./postgresql-contrib-15.1.tgz postgresql-previous-14.6.tgz \
          postgresql-pg_upgrade-15.1.tgz

    Si está corriendo una versión de adJ anterior a la 6.6 puede encontrar
    los paquetes `postgresql-previous` y `postgresql-pg_upgrade` en
    <http://adj.pasosdejesus.org/pub/AprendiendoDeJesus/> en un directorio
    de la forma `6.5-extra`. Y al momento de insalarlo con `pkg_add` use la
    opción `-D unsigned`).

6. Inicia nueva base con clave de administrador de la anterior (suponiendo
   que está en el archivo `.pgpass` de la cuenta `_postgresql` como ocurre
   por omisión en adJ) con:

        doas su - _postgresql
        grep postgres .pgpass |  sed  -e  "s/.*://g" > /tmp/clave.txt
        LANG=C.UTF-8 initdb --encoding=UTF-8 -U postgres --auth=md5 \
          --pwfile=/tmp/clave.txt  -D/var/postgresql/data

7. Mientras se restaura mantén la configuración por omisión (no muevas
   el socket predeterminado) y cambia `pg_hba.conf` de `data` y de `data-14` 
   para modificar

        local all all md5

   por

        local all all trust

8. Inicia la restauración así:

        doas su - _postgresql
        qg_upgrade -b /usr/local/bin/postgresql-14/ -B /usr/local/bin \
          -U postgres -d /var/postgresql/data-14/ -D /var/postgresql/data

   Si llega a fallar con:

        $ pg_upgrade -b /usr/local/bin/postgresql-14/ -B /usr/local/bin \
          -U postgres -d /var/postgresql/data-14/ -D /var/postgresql/data
        Checking for presence of required libraries fatal
        Your installation references loadable libraries that are missing from the
        new installation. You can add these libraries to the new installation,
        or remove the functions using them from the old installation. A list of
        problem libraries is in the file:
        loadable_libraries.txt

   Seguramente te faltó instalar `postgresql-contrib` que incluye `accent`
   y otros módulos.  Instala y repite.

9. Inicia la nueva base con la configuración predeterminada de manera temporal
   con  `doas rcctl start postgresql`

10. Actualiza estadísticas con

        ./analyze_new_cluster.sh

11. Asegura nueva clave.  Revísala con `cat /tmp/clave.txt` y 
    establecela con:

        psql -U postgres template1
        ALTER USER postgres WITH PASSWORD 'nuevaaqui';

12. Detén nuevamente el servicio `postgresql`, modifica 
    `/var/postgresql/data/postgresql.conf` para cambiar ubicación del 
    socket y en general replica la configuración que tenía tu base 
    (e.g conexiones TCP, llaves, etc).

        unix_socket_directories = '/var/www/var/run/postgresql'

    En `data/pg_hba.conf` vuelve a dejar `md5` en lugar de `trust`

13. Inicia el servicio y comprueba la operación

14. Una vez completes con éxito puedes eliminar el cluster anterior con
    `./delete_old_cluster.sh`

15. Si detuviste la actualización de `inst-adJ.sh` vuelva a ejecutarlo 
    y a la pregunta "Desea eliminar la actual versión de PostgreSQL y los 
    datos asociados para actualizarla" responde "No"".


