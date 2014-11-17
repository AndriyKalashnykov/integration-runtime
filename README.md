Du behöver vagrant och virtualbox på din dator.

Starta miljön
=============

Starta maskinen med vagrant

`$ vagrant up`

Efter en stunds tuggande bör maskinen komma igång med följande komponenter installerade

* servicemix
* httpd (för att simulera Ladok Feeds via statiska filer)

Starta servicemix
=================

Starta servicmix i den virtuella miljön

`$ vagrant ssh`

Kontrollera att tjänsten servicemix är startad

`$ sudo service apache-servicemix status`

Håll sedan koll på filen /var/lib/servicemix/log/servicemix.log. 

Konsoll för servicemix
======================
Kör ssh mot servicemix. Lösenordet är "smx"

`$ ssh -p 8101 smx@localhost`

I konsollen kan du t.ex. titta på inkommande kön:

`activemq:browse --amqurl tcp://localhost:61616 ladok3-event-distribution`

eller följa loggen:

`log:tail`


Installera webbkonsoll för ActiveMQ samt Hawtio
===============================================

Om du vill, se följande adresser:

* <http://marcelojabali.blogspot.se/2011/08/how-to-enable-activemq-web-console-on.html>
* <http://hawt.io/getstarted/index.html>

	(Hawtio i korthet...) 
	 
	`features:addurl mvn:io.hawt/hawtio-karaf/1.4.4/xml/features`
	
	`features:install hawtio`
	
	Gå sedan till <http://localhost:8181/hawtio>
	
Installera klientcertifikat för Ladok3
================================
Klienten som används för att hämta händelser från Ladok's ATOM-gränssnitt finns i biblioteket <https://github.com/uppsala-university/ladok3>. 

Kopiera klientcertifikat för Ladok3 till katalogen `ladok3atom-client/src/main/resources/`. Certifikatet ska vara på PKCS 12-format.

I `ladok3atom-client/src/main/resources` finns en exempelfil för fordrade egenskaper. Använd den genom att döpa om den

`mv atomclient.properties.sample atomclient.properties`

Redigera sedan filen `ladok3atom-client/src/main/resources/atomclient.properties` för att innehålla rätt namn på certifikatfil och lösenord.

Driftsätt händelsehanteraren i vagrant-miljö
============================================
Bygg källkoden i `ladok3-event`, genom 

`$ mvn clean package`

Detta bygger ett s.k. "maven multi module project".

Nu går det att deploya till servicemix. Det görs genom att kopiera filer till deploy-katalogen som
ligger under "smx/deploy" relativt denna fil. Detta är en specialare för vagrant-miljöerna. En närmare
titt i filen provision/manifests/default.pp indikerar att puppet kommer länka om deploy-katalogen så 
att den hamnar på den delade disken mellan vagrant-maskinen och värd-maskinen. Detta gör det smidigare
att utveckla. 

Kopiera först klienten från `ladok3`

`$ cp ladok3atom-client/target/ladok3atom-reader-0.0.1-SNAPSHOT.jar smx/deploy/`

Driftsätt sedan integrationsadaptern som exekverar klienten från `ladok3-event`

`$ cp ladok3atom-event-adapter/target/ladok3atom-event-adapter-0.0.1-SNAPSHOT.jar smx/deploy/`

Därefter driftsätts den integrationsmodul som distribuerar händelserna till de system som ska prenumerera på händelserna

`$ cp ladok3-event-distribution/target/ladok3-event-distribution-0.0.1-SNAPSHOT.jar smx/deploy/`

Driftsätt sedan en konsument

`$ cp ladok3event-logdb-adapter/target/ladok3event-logdb-adapter-0.0.1-SNAPSHOT.jar smx/deploy/`

Deploy-katalog på den delade disken kommer överleva omstarter och ominstallationer (vagrant destroy) av den virtuella maskinen.

Driftsättning i ICKE-vagrant-miljö
========================================

Samma jar-filer som ovan men de ska kopieras in i `/opt/servicemix/apache-servicemix-5.1.2/deploy`
