---
title: WebGoat_in_OpenBSD
date: 2011-09-15
tags:
---
 !WebGoat is a J2EE application to learn about web application security.    Some of its exercises require modifications of its sources in Java and recompiling and deploying.   The process seems to be simple with the heavy Eclipse, but with some configuration it can be done from the command line in OpenBSD adJ 4.9.

see more context for this post at http://p2pu.org/es/groups/web-application-security/.

## Install

First install the required packages:
<pre>
  sudo pkg_add jdk maven subversion-1.6.15p0 p7zip
</pre>

In order to change the sources, it is a good idea to retrieve the latest:

<pre>
  cd /home/myuser/
  mkdir w
  cd w
  svn checkout http://webgoat.googlecode.com/svn/trunk/ .
</pre>

The sources will be at ```/home/myuser/w/webgoat```


Tomcat is required to run it, we found easier to use the precompiled version  included in the binary distribution of !WebGoat:


<pre>
  cd /home/myuser/w
  ftp "http://webgoat.googlecode.com/files/WebGoat-OWASP_Standard-5.3_RC1.7z"
  7z x !WebGoat-!OWASP_Standard-5.3_RC1.7z
</pre>


Then copy the tomcat and the startup script included and prepare to use it:

<pre>
  cd /home/myuser/w
  cp -rf !WebGoat-5.3_RC1/tomcat .
  cp !WebGoat-5.3_RC1/webgoat.sh .
  chmod +x webgoat.sh
</pre>

The script webgoat.sh checks for Java 1.6, since we have 1.7 it is necessary to change that version number by editing it:
<pre>
  vim webgoat.sh
  :%s/1.6/1.7/g
  :wq
</pre>

The Tomcat distribution includes a webgoat application that should be deleted:


<pre>
  cd /home/myuser/w
  find tomcat -name "webgoat*" | xargs rm -rf
  find tomcat -name "!WebGoat*" | xargs rm -rf
</pre>

The compilation process will be easier also by changing the POM file of webgoat, edit webgoat/pom.xml and add the following to the section plugins:
<pre>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>2.1.1</version>
            </plugin>
</pre>


## Compilation and deployment

To compile it the first time or after changing the sources 
<pre>
  cd /home/myuser/w/webgoat
  mvn clean
  mvn compile
  mvn war:war
</pre>

This will compile and create the war file. To deploy it:

<pre>
  cd /home/myuser/w
  cp webgoat/target/!WebGoat-5.3-SNAPSHOT.war tomcat/webapps/!WebGoat.war
</pre>

## Use

<pre>
  cd /home/myuser/w/webgoat/target
  ./webgoat.sh start8080
</pre>

After that you can open with a browser:

<pre>
    http://127.0.0.1:8080/WebGoat/attack
</pre>

Your progress in lessons will be stored in the directory ```/home/myuser/w/tomcat/webapps/!WebGoat/``` --don't delete it.


## References

* Maven Getting Started Guide http://maven.apache.org/guides/getting-started/index.html#How_do_I_deploy_my_jar_in_my_remote_repository

----

2011. vtamara@pasosdeJesus.org. Public domain according to the colombian legislation.
