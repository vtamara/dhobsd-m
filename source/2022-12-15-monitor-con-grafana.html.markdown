---

title: Monitor para adJ con grafana, prometheus y smartmontools
date: 2022-12-15 12:00 UTC
tags: 

---


Para visualizar graficamente el desempeño de un servidor en cuanto a CPU,
memoria, uso de red y discos nos ha funcionado:

1. grafana para crear el tablero gráfico
2. prometheus con node_exporter  para incluir información de CPU, memoria y
   red
3. smartmontools, un script y el recolector de textos de prometheus para 
   incluir información de discos.

# 1. node_exporter

`node_exporter` es uno de los componentes de prometheus para exportar 
información del servidor donde corre.

Aún cuando su especialidad es Linux, en OpenBSD operan los siguientes
recolectores  (según documentación y prueba realizada en adJ 7.2):

* boottime
* cpu
* diskstats
* filesystem
* loadavg
* meminfo
* netdev
* os
* textfile
* time
* uname

Por omisión da resultados por el puerto TCP 9100 y responde con diversas
estadísticas producidas por los mencionados recolectores.

Ponlo en operación así:

1. Instala el paquete:
   <pre>
   doas pkg_add node_exporter
   </pre>
2. Edita `/etc/rc.d/node_exporter` para cambiar `daemon` por `servicio`.
3. Inicia el servicio con:
   <pre>
   doas rcctl -d enable node_exporter
   doas rcctl -d start node_exporter
   </pre>

Podrás comprobar la operación con:
<pre>
tail -f /var/log/servicio
curl http://localhost:9100/metrics | grep "node_"
</pre>


# 2. prometheus

Permite monitorear y alertar mediante (1) recolectores de información,
(2) una base de datos que está almacenando periodicamente
los resultados recolectados por `node_exporter` en `/var/prometheus`, 
(3) un lenguaje y motor para evaluar expresiones sobre la información 
recolectada y graficarla.

Ponlo en operación así:

1. Instala el paquete
   <pre>
   doas pkg_add node_exporter
   </pre>
2. Edita `/etc/prometheus/prometheus.yml`  para cambiar en la sección 
`scrape_configs`
    * `job_name` digamos por `servidor` o un nombre corto
    * `targets` cambia `localhost:9090` por `localhost:9100`
3. Edita `/etc/rc.d/prometheus` para:
    * Cambiar `dameon` por `servicio`
    * Agregar `--storage.tsdb.retention.time=600d` a `servicio_flags` 
      para mantener últimos 600 días
4. Inicialo con:
    <pre>
    doas rcctl -d enable prometheus
    doas rcctl -d start prometheus
    </pre>

Comprueba su operación con:
<pre>
tail -f /var/log/servicio
curl localhost:9090/graph
</pre>

Para experimentar con prometheus ingresa con un navegador
a <http://localhost:9090/graph> y utiliza expresiones como:
<pre>
 rate(node_cpu_seconds_total{mode="system"}[1m])
 node_filesystem_avail_bytes
 rate(node_network_receive_bytes_total[1m])
</pre>

# 3. smartmontools y exportación a texto recolectable por node_exporter

`smartmontools` son herramientas para monitorear discos duros, especialmente
los que tiene tecnología SMART que ayuda a predecir cuando fallará el disco.
Entre sus monitores está uno de temperatura y según la página del manual de
`smartctl`  una temperatura elevada es la principal causa de falla en
discos duros.

Viene por omisión en adJ, puedes instalarlo con:
<pre>
doas pkg_add smartmontools
</pre>

Puedes ver el listado de discos con:
<pre>
doas smartctl --scan
</pre>

Y detalles completos de alguno de los discos (e.g `/dev/sd0c`) con:
<pre>
doas smartctl -x /dev/sd0c
</pre>

En varios repositorios hay disponibles scripts para convertir del
formato de diagnóstico de smartmontools al recibido por el recolector
de textos planos de node_exporter. Hemos bifurcado y adaptado para adJ uno
en <https://github.com/vtamara/node-exporter-smartmon>. De allí  descarga el
guión `smartmon.sh` y ubicalo por ejemplo en `/root/guiones/smartmon.sh`

Crea un directorio para el recolector de textos planos de `node_exporter` por
ejemplo:
<pre>
mkdir -p /var/log/node-exporter/
</pre>

Y modifica `/etc/rc.d/node_exporter` para agregar:

<pre>
servicio_flags="--collector.textfile.directory=/var/log/node-exporter/"
</pre>

Reinicia `node_exporter` con `doas rcctl -d restart node_exporter`
y agrega una tarea cron para el usuario `root` (con `doas crontab -e`) que 
por ejemplo cada minuto envie información de discos a ese directorio:

<pre>
*  *  *  *  * /root/guiones/smartmon.sh 2>&1 > /var/log/node-exporter/smartmon.prom.tmp && mv /var/log/node-exporter/smartmon.prom.tmp /var/log/node-exporter/smartmon.prom
</pre>



# 4. grafana

Esta aplicación web permite crear tableros  para presentar las gráficas de
prometheus.

Ponla a operar así:

1. Instala el paquete con
   <pre>
   doas pkg_add grafana
   </pre>

2. Edita `/etc/rc.d/grafana` para cambiar `daemon` por `servicio`.
3. Habilítala e inicíala como servicio con:
   <pre>
   doas rcctl -d enable grafana
   doas rcctl -d start grafana
   </pre>

Podrás verla operando con un navegador en <http://localhost:3000>

El primer ingreso lo puedes hacer con el usuario `admin` y
la clave `admin` y entonces podrás cambiar la clave.

Si posteriormente necesitas cambiar clave, desde la terminal usa:
<pre>
grafana-cli --homepath=/usr/local/share/grafana/ --config /etc/grafana/config.ini admin reset-admin-password adminmon
</pre>

Agrega una fuente de datos desde el icono de configuración "Data source"
Usa <http://localhost:9090>

Administra tableros de control e importa el que está disponible en:
<https://findelabs.com/post/grafana-prometheus-monitoring-openbsd/>

Después ajústalo y agrégale dos paneles para el monitoreo de discos duros.


# 5. Referencias

* <https://findelabs.com/post/grafana-prometheus-monitoring-openbsd/>
* <https://prometheus.io/docs/guides/node-exporter/>
* <https://github.com/yuanying/node-exporter-smartmon>


