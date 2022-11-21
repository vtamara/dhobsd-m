---
title: WebGoat_en_OpenBSD
date: 2011-09-26 12:00 UTC
tags:
---
 !WebGoat es una aplicación J2EE para aprender sobre seguridad de aplicaciones web.    Algunos de sus ejercicios requieren modificación de sus fuentes en Java, recompilación e implantación (deployment).  El proceso parece simple con el pesado Eclipse, pero con algo de configuración puede hacerse desde la línea de comandos en OpenBSD adJ 4.9.

Vea más sobre el contexto de esta artículo en http://www.p2pu.org/es/groups/seguridad-de-aplicaciones-web/

## Instalación

Primero instale los paquetes requeridos
<pre>
  sudo pkg_add jdk maven subversion-1.6.15p0 p7zip
</pre>

Para modifcar las fuentes, es buena idea sacar las más recientes del repositorio:

<pre>
  cd /home/myuser/
  mkdir w
  cd w
  svn checkout http://webgoat.googlecode.com/svn/trunk/ .
</pre>

Las fuentes quedarán en ```/home/myuser/w/webgoat```


Se requiere Tomcat para ejecutar !WebGoat, nos paree más fácil emplear la versión precompilada que se incluye con la distribución binaria de !WebGoat:


<pre>
  cd /home/myuser/w
  ftp "http://webgoat.googlecode.com/files/WebGoat-OWASP_Standard-5.3_RC1.7z"
  7z x !WebGoat-!OWASP_Standard-5.3_RC1.7z
</pre>


A continuación copie Tomcat y el archivo de comandos para iniciarlo incluido en esa distribución binaria y preparelos para usarlos:

<pre>
  cd /home/myuser/w
  cp -rf !WebGoat-5.3_RC1/tomcat .
  cp !WebGoat-5.3_RC1/webgoat.sh .
  chmod +x webgoat.sh
</pre>

El archivo de comandos ```webgoat.sh``` verifica que esté Java 1.6, como tenemos la versión 1.7 es necesario  cambiar el número de versión editandolo:

<pre>
  vim webgoat.sh
  :%s/1.6/1.7/g
  :wq
</pre>

La distribución de Tomcat incluye un directorio ```webgoat``` que debe eliminarse:


<pre>
  cd /home/myuser/w
  find tomcat -name "webgoat*" | xargs rm -rf
  find tomcat -name "!WebGoat*" | xargs rm -rf
</pre>

El proceso de compilación será más facil modificando el archivo POM de webgoat. Edite ```webgoat/pom.xml``` y añada lo siguiente a la sección ```plugins```:
<pre>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>2.1.1</version>
            </plugin>
</pre>


## Compilación e Implantación

Para compilar la primera vez o después de cambiar las fuentes
<pre>
  cd /home/myuser/w/webgoat
  mvn clean
  mvn compile
  mvn war:war
</pre>

Esto compila y crea el archivo war.  Para implantarlo:

<pre>
  cd /home/myuser/w
  cp webgoat/target/!WebGoat-5.3-SNAPSHOT.war tomcat/webapps/!WebGoat.war
</pre>

## Utilización

Asegurese de haber definido la variable de entorno JAVA_HOME por ejemplo en su archivo ```~/.profile```:
<pre>
export JAVA_HOME=/usr/local/jdk1.7.0/
</pre>
Puede iniciar Tomcat --que leerá el archivo .war de WebGoat-- con:

<pre>
  cd /home/myuser/w/
  ./webgoat.sh start8080
</pre>

Después de esto puede abrir en un navegador:

<pre>
    http://127.0.0.1:8080/WebGoat/attack
</pre>

Puede emplear usuario ```guest``` con la misma clave o usuario ```admin``` con clave ```webgoat```.

Su progreso en las lecciones quedará en el directorio  ```/home/myuser/w/tomcat/webapps/!WebGoat/users/``` --no lo borre o respaldelo antes de cada nueva implantación.


## Referencias

* Maven Getting Started Guide http://maven.apache.org/guides/getting-started/index.html#How_do_I_deploy_my_jar_in_my_remote_repository

----

2011. vtamara@pasosdeJesus.org. Dominio público de acuerdo a la legislación colombiana.
