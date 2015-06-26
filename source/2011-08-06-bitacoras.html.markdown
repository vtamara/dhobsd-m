---
title: bitacoras
date: 2011-08-06
tags:
---
logger permite ingresar eventos en una bitácora del sistema por ejemplo

logger -p user.notice "Este es un mensaje"

Agregara el mensaje en la bitacora asociada a la facilidad user (por defecto /var/log/messages) con prioridad notice.

Las prioridades (definidas en /usr/include/syslog.h) posibles son:

* emerg  El sistema es inusable
* alert  Debe actuarse de inmediato
* crit    Condicón crítica
* err     Cóndicion erronea
* warning  Condición de advertencia
* notice  Normal pero en condición significativa
* info  informativo
* debug  mensaje a nivel de depuración.

Los nombres de faciliades son:
* kern Mensaje del kernel
* user mensaje a nivel de  usuario
* mail Sistema de correo
* service Servicio del sistqema  (en otros sistemas se llama daemon pero en adJ ese término se considera obsoleto).
* auth o security  mensaje de autorización
* syslog Generado internamente por syslogd
* lpr Susbistema de impresión
* news Subsistema de noticias en red
* uucp Subsistema UUCP
* cron Servicio de reloj
* authpriv mensjae de autorización/seguridad privado
* ftp Servicio ftp
* local0 a local7 reservados para uso local

Los mensajes enviados con funciones en C on con el programa logger son procesados por el servicio syslogd.  Este servicio al inicio lee su configuración del archivo /etc/syslogd.conf, el cual por defecto asocia facilidades y servicios así:

|kern.debug;syslog,user.info                             /var/log/messages
auth.info                                               /var/log/authlog
authpriv.debug                                          /var/log/secure
cron.info                                               /var/cron/log
daemon.info                                             /var/log/daemon
ftp.info                                                /var/log/xferlog
lpr.debug                                               /var/log/lpd-errs
mail.info                                               /var/log/maillog
o #uucp.info                                              /var/log/uucp
 | |

Desde programas en C puede incluirse syslog.h y pueden emplearse las siguientes funciones:

void    closelog(void);
void    openlog(const char *, int, int);
int     setlogmask(int);
void    syslog(int, const char *, ...)
    __attribute__((__format__(__syslog__,2,3)));
void    vsyslog(int, const char *, __va_list);
void    closelog_r(struct syslog_data *);
void    openlog_r(const char *, int, int, struct syslog_data *);
int     setlogmask_r(int, struct syslog_data *);
void    syslog_r(int, struct syslog_data *, const char *, ...)
     __attribute__((__format__(__syslog__,3,4)));
void    vsyslog_r(int, struct syslog_data *, const char *, __va_list);


En adJ además de las facilidades típicas puede emplearse la facilidad service
