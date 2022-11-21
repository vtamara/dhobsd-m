---

title: ClamAV como milter de Sendmail
date: 2005-12-20 13:18 UTC
tags:

---

En este artículo describimos como configurar el antivirus de correos clamav con 
clamav-milter en un OpenBSD 3.7 con sendmail.

##Configuración de ClamAV

Instalar desde portes, pues clamav depende de arc que está disponible sólo como 
porte:
<pre>
# cd /root/tmp
# ftp rt.fm/pub/OpenBSD/3.7/ports.tar.gz
# cd /usr
# tar xvfz /root/tmp/ports.tar.gz
# cd /usr/ports/security/clamav
# make install
</pre>

Edite ```/etc/clamd.conf``` y ```/etc/freshclam.conf```, en ambos quite la linea ```Example``` y active las apropiadas. En ambos emplee como usuario ```_clamav``` y emplee archivos logs.

Prepare archivos de log:
<pre>
# touch /var/log/freshclam.log
# chown _clamav:_clamav /var/log/fresclam.log
</pre>

Ejecutar ```freshclam``` para actualizar la base de datos de virus.
<pre>
# freshclam
</pre>

Vale la pena hacer esta actualización a diario. Puede ser agregando al archivo 
```/etc/daily.local```:
<pre>
if ![ -f /usr/local/bin/freshclam; ] then
        /usr/local/bin/freshclam
fi
</pre>

Puede asegurar que durante el arranque se ejecuta el servidor agregando a 
```/etc/rc.local```:
<pre>
if ![ -x /usr/local/sbin/clamd; ] then
        mkdir -p /var/run/clamav
        chown _clamav:_clamav /var/run/clamav
        echo -n " clamd"; /usr/local/sbin/clamd
        if ![ -x /usr/local/sbin/clamav-milter ] then
                echo -n " clamav-milter"; 
                /usr/local/sbin/clamav-milter ${clamav_milter_flags}
        fi
fi
</pre>

Y a ```/etc/rc.conf.local```:
<pre>
clamav_milter_flags="--max-children=2 -olb local:/var/run/clamav/clamav-milter.sock"
</pre>

##Configuración de ```sendmail``` para que soporte milter de ambos:

Asegurese de tener:
<pre>
WANT_LIBMILTER=1
</pre>
en ```/etc/mk.conf```

Descargue las fuentes del sistema base (src.tar.gz) e instalelas en 
```/usr/src``` después:
<pre>
cd /usr/src/gnu/usr.sbin/sendmail/
make
cd cf/cf
cp openbsd-proto.mc openbsd-proto-milter.mc
</pre>

Edite ```openbsd-proto-milter.ml``` para añadir:

<pre>
dnl
dnl End of masquerading section.
define(`confMILTER_MACROS_CONNECT',`b, j, _, {daemon_name}, {if_name}, {if_addr}')dnl
INPUT_MAIL_FILTER?(`clamav', `S=local:/var/run/clamav/clamav-milter.sock, T=S:4m;R:4m')dnl
define(`confINPUT_MAIL_FILTERS', `clamav')dnl
dnl Fin de milters
MAILER(local)dnl
MAILER(smtp)dnl
</pre>

Termine la configuración con:
<pre>
make openbsd-proto-milter.cf
cp openbsd-proto-milter.cf /etc/mail/sendmail.cf
</pre>

##Pruebas

La siguiente vez que reinicie su equipo debe quedar funcionando la nueva 
configuración, para probarla sin reiniciar:
<pre>
# . /etc/rc.conf
# mkdir -p /var/run/clamav
# chown _clamav:_clamav /var/run/clamav
# /usr/local/sbin/clamd
# pkill -9 sendmail
# sendmail $sendmail_flags
# /usr/local/sbin/clamav-milter $clamav_milter_flags
</pre>
En la bitácora de correo (```/var/log/maillog```) por cada correo que entra o sale por el servidor debe quedar una entrada del estilo:
<pre>
Aug  3 12:04:21 correo sm-mta15245?: j73H4I2N015245: Milter add: header: X-Virus-Scanned: 
  ClamAV version 0.83, clamav-milter version 0.83 on 192.168.6.108
Aug  3 12:04:21 correo sm-mta15245?: j73H4I2N015245: Milter add: header: X-Virus-Status: 
  Infected with Worm.Mydoom.I
Aug  3 12:04:21 correo sm-mta15245?: j73H4I2N015245: Milter: data, reject=554 5.7.1 virus 
  Worm.Mydoom.I detected by ClamAV - http://www.clamav.net
Aug  3 12:04:21 correo sm-mta15245?: j73H4I2N015245: to=<a@pasosdeJesus.org>, 
  delay=00:00:02, pri=100560, stat=virus Worm.Mydoom.I detected by ClamAV - 
  http://www.clamav.net
</pre>

##Un problema que puede surgir y solución

Nos parece que si su servidor maneja un volumen alto de correos, esta 
configuración puede reiniciar esporádicamente el computador o producir 
fallas en el kernel.

```clamav-milter``` permite distribuir la carga de la busqueda de virus en 
diversos servidores. Suponiendo que planee emplar 2 servidores (uno de 
recepción de correos y otro como antivirus), en el primero puede 
deshabilitar clamd, dejando solo clamav-milter mientras que en el segundo 
puede dejar solo ```clamd```.

En el que recibe correos debe iniciar clamav-milter con flags como:
<pre>
clamav_milter_flags="--server=192.168.3.10 -d --max-children=2 -q -e -olb local:
/var/run/clamav/clamav-milter.sock"
</pre>

Siendo ```192.168.3.10``` la IP del computador donde corre ```clamd```. En ese mismo 
el archivo de configuración ```/etc/clamd.conf``` puede incluir las lineas:
<pre>
!LogFile /var/log/clamd.log
!LogTime
!LogSyslog
!PidFile /var/run/clamd.pid
!TemporaryDirectory /tmp
!DatabaseDirectory /var/db/clamav
TCPSocket 3310
User _clamav
</pre>

asegurese de no dejar o dejar con comentarios las líneas ```!LocalSocket``` y 
```TCPAddr```.

El archivo ```/etc/clamd.conf``` del computador donde correrá sólo clamd 
puede incluir:
<pre>
!LogFile /var/log/clamd.log
!LogTime
!PidFile /var/run/clamd.pid
!DatabaseDirectory /var/db/clamav
TCPSocket 3310
User _clamav
</pre>

asegurandose de no dejar la línea ```!LocalSocket```.


##REFERENCIAS

* Mauricio Rivera
* Páginas man de ```clamd``` y ```clamav-milter```
