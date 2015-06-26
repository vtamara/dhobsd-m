---
title: Webmaking_101_hosting
date: 2012-01-11
tags:
---
Thanks to God I have my own web servers as well as name servers that I use for hosting webpages.  They run OpenBSD adJ, so the following example to setup  http://ejemplohtml.pasosdeJesus.org is for that distribution:

! 1. Prepare webpage(s)

First I choose a server to host the website and login for example:
<pre>
ssh sn2.pasosdeJesus.org
</pre>
then  I prepare an accesible directory and put there the HTML pages, the main one should be called ```index.html``` : 
<pre>
cd /var/www/htdocs/
mkdir ejemplohtml
cd ejemplohtml
vi index.html
</pre>
and edit the webpages, with my prefered editor vim  I  use the example described at http://dhobsd.pasosdejesus.org/index.php?id=Webmaking+101+world 

! 2. Prepare webserver
I edit the configuration of the webserver (```/var/www/conf/httpd.conf```) to include a new ```!VirtualHost```:
<pre>
<!VirtualHost 201.245.63.134:80 127.0.0.1:80>
    !ServerAdmin vtamara@pasosdeJesus.org
    !DocumentRoot /var/www/htdocs/ejemplohtml/
    !ServerName ejemplohtml.pasosdeJesus.org

    !ErrorLog logs/pasosdeJesus-error_log
    !CustomLog logs/pasosdeJesus-access_log common
</!VirtualHost>
</pre>
After that I restart the webserver
<pre>
sudo apachectl stop
sudo sh /etc/rc.local
</pre>
If needed I can look for errors in ```/var/log/service``` and ```/var/www/logs/```


!3.  Prepare DNS

I login to the main DNS server for ```pasosdeJesus.org```, and edit the domain zone ```/var/www/master/pasosdeJesus.org``` to add  the subdomain associated with the IP address of ```sn2.pasosdeJesus.org``` (that is 201.245.63.134):
<pre>
ejemplohtml     IN      A       201.245.63.134
</pre>
And then I increase the serial number at the beginning of file  (I prefer to use the date and a serial):
<pre>
2012011001 ; Serial
</pre>
Then restart the DNS server:
<pre>
sudo pkill named
sudo sh /etc/rc.local
</pre>
I test from another computer by doing:
<pre> 
dig @sn1.pasosdeJesus.org ejemplohtml.pasosdeJesus.org
</pre>
I can see also log errors in ```/var/log/messages``` and ```/var/log/service```

Finally I propagate the change by logging into slave DNS server, deleting the copy of the zone (```/var/named/slave/pasosdeJesus.org```) and restarting ```named```.

!4. Ready

After this immeadiatly I can visit: http://ejemplohtml.pasosdeJesus.org   --I have improved the version posted there.
