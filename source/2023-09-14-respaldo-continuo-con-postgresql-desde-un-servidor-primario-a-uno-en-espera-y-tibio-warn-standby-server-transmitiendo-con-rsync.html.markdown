---

title: Respaldo continuo con PostgreSQL desde un servidor primario a uno en-espera y tibio (warn standby server) transmitiendo con rsync
date: 2023-09-14 19:04 UTC
tags: 

---
# Respaldo continuo con PostgreSQL desde un servidor primario a uno en-espera y tibio (warn standby server) transmitiendo con rsync

# 1. Introducción

Cómo se explica en 
<https://www.postgresql.org/docs/15/different-replication-solutions.html> 
hay diferentes mecanismos para hacer copias de respaldo que 
progresivamente van facilitando operación casi continua y alta disponibilidad:

* A nivel de hardware  y/o sistema operativo: Disco compartido (e.g NAS)
* A nivel de sistema operativo: Replicación de sistema de archivos
* A nivel de PostgreSQL y sistema operativo:
    * Envío de registros de escritura anticipada
    * Replicación lógica

Entre más alto nivel, más flexibilidad (por ejemplo con replicación lógica se 
puede replicar solo algunas tablas y sólo registros que cumplan un filtro, 
la base destino puede tener una versión diferente a la de la base origen e 
incluso una estructura diferente), pero también más involucrada la aplicación 
(por ejemplo el último se hace con ordenes de PostgreSQL).

Suponiendo que tiene un servidor primario y quiere mantener una copia
bastante actualizada de solo lectura de la base de datos completa en un
servidor en-espera puede usar el envió de registros de escritura anticipada.
En este escrito presentamos tal configuración suponiendo que los servidores
operan adJ.

# 2. Envío de registros de escritura anticipada con rsync

Como se explica en <https://www.postgresql.org/docs/15/warm-standby.html>
el primario se configura en modo de archivado continuo y el servidor en 
espera se configuran en modo de recuperación continua.

Supongamos que el servidor primario es un adJ con IP 192.168.1.5 y que el 
servidor en espera es otro adJ con IP  192.168.1.11, y que para transmitir 
los registros WAL del primario al servidor en espera se usa `rsync`.

## 1.1. Preparación del servidor en-espera

Asegurate de instalar PostgreSQL aunque no es necesario que inicialices
base de datos.

        doas pkg_add postgresql-server

Prepara el directorio donde llegarán los registros WAL, por ejemplo:

        doas mkdir /var/www/resbase/archivo_wal/
        doas chown _postgresql:_postgresql /var/www/resbase/archivo_wal/

Y alista una llave RSA para enviar sin clave con ssh desde la cuenta 
`_postgresql` del primario a la cuenta `_postgreql` del servidor en-espera:

        doas su - _postgresql
        ssh-keygen

Para que pueda ser un método automático, no le pongas clave

        cp ~/.ssh/id_rsa.pub ~/.ssh/authorized_keys

Copia la llave privada al servidor primario:

        scp ~/.ssh/id_rsa usuario@192.168.1.5:/tmp/

## 1.2. Configuración del servidor primario

Prepara el envío de archivos sin clave con ssh y pruébalo:

        doas su - _postgresql
        cp /tmp/id_rsa ~/.ssh/id_rsa_enespera
        scp /etc/motd 192.168.1.11:/var/postgresq/

La última orden debería copiar el archivo `/etc/motd` del primario a 
`/var/postgresql/motd` del servidor en-espera sin pedir clave.

Modifica `/var/postgresql/data/postgresql.conf` para que queden las líneas:

        wal_level = replica

        archive_mode = on

        archive_command = 'rsync -e "ssh -i /var/postgresql/.ssh/id_rsa_enespera" -a %p _postgresql@192.168.1.11:/var/www/resbase/archivo_wal/%f'

Tras esto debes reiniciar PostgreSQL en el primario

        doas rcctl -d restart postgresql

Debes verificar que a medida que realizas operaciones de escritura en la 
base del primario y que se crean registros WAL en 
`/var/postgresql/data/pg_wal`, estos se van enviando  al secundario donde 
van quedando en el directorio `/var/www/resbase/archivo_wal` y pueden 
verse como así:

        # ls -lat /var/www/resbase/archivo_wal/ 
        total 229624
        drwx------   2 _postgresql  _postgresql      2048 Sep 14 09:00 .
        -rw-------   1 _postgresql  _postgresql  16777216 Sep 14 08:10 000000010000000F0000006C
        -rw-------   1 _postgresql  _postgresql       341 Sep 14 08:10 000000010000000F0000006C.00000028.backup
        -rw-------   1 _postgresql  _postgresql  16777216 Sep 14 08:07 000000010000000F0000006B
        -rw-------   1 _postgresql  _postgresql  16777216 Sep 14 08:06 000000010000000F0000006A
        -rw-------   1 _postgresql  _postgresql  16777216 Sep 14 08:02 000000010000000F00000069
        -rw-------   1 _postgresql  _postgresql       341 Sep 14 08:02 000000010000000F00000069.00000028.backup
        -rw-------   1 _postgresql  _postgresql  16777216 Sep 14 07:58 000000010000000F00000068
        -rw-------   1 _postgresql  _postgresql  16777216 Sep 14 07:56 000000010000000F00000067
        -rw-------   1 _postgresql  _postgresql  16777216 Sep 14 07:56 000000010000000F00000066


## 1.3 Copia de respaldo inicial del primario al servidor en-espera

Desde la cuenta `_postgresql` del servidor primario ejecuta:

        pg_basebackup -U postgres --progress -h /var/www/var/run/postgresql/ -D - -Ft -X fetch | bzip2 > respaldo.tar.bz2

Y envía la copia generada al servidor en-espera

        scp respaldo.tar.bz2 _postgresql@192.168.1.11:/tmp/

Desde el servidor en-espera descomprime la copia con permisos precisos

        doas su -
        mv /tmp/respaldo.tar.bz2 /var/postgresql/
        cd /var/postgresql
        bunzip2 respaldo.tar.bz2
        mv data data-ant
        mkdir data
        chown _postgresql:_postgresql
        chmod 700 data
        cd data
        tar xvfp ../respaldo.tar
        rm pg_wal/*


## 1.4 Configuración de servidor en-espera y arranque

Configura `postgresql.conf` con:

        restore_command = 'cp /var/www/resbase/archivo_wal/%f %p'

        archive_cleanup_command = 'pg_archivecleanup /var/www/resbase/archivo_wal %r'

Para indicar que tu servidor operará en-espera y tibio:

        touch standby.signal
        chown _postgresql:_postgresql standby.signal


Inicia la base de datos

        rcctl -d start postgresql

El motor empezará a usar las bitácoras que halla en el archivo y en 
bitácora `/var/postgresql/logfile` debe quedar un mensaje que indica 
que está esperando el siguiente segmento de manera reiterativa:

```
2023-09-14 13:23:26.839 -05 [14004] LOG:  waiting for WAL to become available at F/6D000018
cp: /var/www/resbase/archivo_wal/000000010000000F0000006D: No such file or directory
cp: /var/www/resbase/archivo_wal/00000002.history: No such file or directory
```

A medida que se escriban datos en el servidor primario, que se generen
registros de escritura anticipada, que se transmitan al servidor en-espera
y que este los replique podrás examinar en modo de solo lectura en el
servidor en-espera la información más reciente que vaya cambiando en
el servidor primario.


# 3. Conclusión

Es una configuración relativamente fácil de implementar que dejará una copia
continuamente actualizada y de sólo lectura de la base de datos del servidor
primario en un servidor en-espera, tibio, es tibio porque la copia queda un poco
retrasada respecto al original.
