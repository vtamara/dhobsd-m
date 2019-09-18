---

title: Actualización de PostgreSQL con pg_upgrade
date: 2019-03-11 15:55 UTC
tags: 

---

Es un método más rápido que sacar volcados SQL, actualizar paquete 
y restaurar volcados.

Se documenta en el paquete postgresql-pg_upgrade pero con estos cambios:

1. Antes de actualizar sacar los respaldos tipicos: i.e volcado en 
   pga-5.sql y binarios copiados en data--20180319.tar.gz

2. Si se está actualizando adJ con inst-adJ.sh, detener el proceso cuando 
   pregunte "Desea eliminar la actual versión de PostgreSQL"

3. Detener base anterior

4. Desinstalar postgresql-docs y postgresql-client (que desinstalar otro 
   incluyendo postgresql-server)

5. Mover /var/postgresql/data a /var/postgresql/data-9.6

v=`cat data/PG_VERSION`
mv data data-$v

6. Instalar paquete postgresql-server (que instalará postgresql-client) y
   postgresql-contrib. No 
   instalar postgresql-docs aún porque tiene conflicto con 
   postgresql-previous

cd ../6.4-amd64/paquetes/
PKG_PATH=. doas pkg_add postgresql-server

7.Instalar paquetes postgresql-pg_upgrade y postgresql-previous se
  descargan de directorio 6.4-extra

  PKG_PATH=. doas pkg_add -D unsigned postgresql-previous-10.6.tgz
  PKG_PATH=. doas pkg_add -D unsigned postgresql-pg_upgrade-11.2.tgz   

8. Inicializar base para nueva version asi:

apg | head -n | > /tmp/clave.txt

LC_ALL=C doas su - _postgresql -c "initdb --encoding=UTF-8 -U postgres --auth=md5 --pwfile=/tmp/clave.txt  -D/var/postgresql/data"

(si no se especifica LC_ALL, la bases template  inician con es_CO.UTF-8)

Para facilitar operacion es recomendable poner la clave de /tmp/clave.txt
en /var/postgresql/.pgpass

9. Para realizar la restauración se sugiere mantener configuración por 
   omisión (no mover sockets) pero cambiar pg_hba.conf de data y de 
   data-$v para modificar

   local all all md5

por

    local all all trust


10. Iniciar restauración así:
  doas su _postgresql -c "pg_upgrade -b /usr/local/bin/postgresql-$v/ -B /usr/local/bin -U postgres -d /var/postgresql/data-$v/ -D /var/postgresql/data"

12. Una vez se complete con exito lo anterior actualizar estadísticas con 

 ./analyze_new_cluster.sh

13. Se puede eliminar el cluster anterior
  ./delete_old_cluster.sh

14. Aplicar los cambios tipicos al archivo de configuración
    /var/postgresql/data/postgresql.conf
    Dejar unix_socket_directories en /var/www/var/run/postgresql
    y dejar listen_address en 127.0.0.1
Iniciar servicio postgres de la forma típica

15. Si se requiere cambiar clave de usuario postgres y replicar en .pgpass

16. Detener servicio postgres

17. Restaurar pg_hba.conf
local   all             all                                     md5

18. Verificar que ingresa como postgres con la clave de .pgpass y
    que están las bases actualizadas

19. Continuar actualización con adJ si se había interrumpido pero sin
    crear respaldo, ni eliminar PostgreSQL, ni restaurar volcado
