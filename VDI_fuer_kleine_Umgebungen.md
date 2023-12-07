Virtuelle Desktops 
für kleine Umgebungen
Burkhard Obergöker
Version 1.1, 2021-01-03

# 1.  Ziel und Zweck
Das vorliegende Konzept bietet eine Lösung zur Bereitstellung einer Virtual-Desktop-Lösung (VDI), die im Rahmen von Online-Schulungen benutzt werden kann, also ohne eine persönliche Präsenz.

Grundsätzlich sollen dabei folgende Eigenschaften erfüllt werden:
    * Für jeden/jede Teilnehmer*in soll ein dediziertes Desktop-System bereit gestellt werden, das einem Desktop-PC aus dem x86-Technikbereich entspricht. Das verwendete Betriebssystem für den einzelnen Desktop soll dabei frei wählbar sein.
    * Das System soll für die Teilnehmer von beliebigen Plattformen aus genutzt werden können, ohne dass eine spezielle Software installiert werden muss. Zu bevorzugen ist eine Web-Browser-basierte Lösung.
    * Die Realisierung soll möglichst auf Open-Source-Komponenten basieren, zumindest muss aber eine freie Nutzung der verwendeten Software gewährleistet sein, damit weder eine Abhängigkeit zu einem Hersteller besteht, noch Lizenzkosten zu berücksichtigen sind. 
    * Das System soll je nach Anforderung skalierbar sein, damit höhere Anforderungen durch leistungsstärkere Hardware oder durch zusätzliche Komponenten ausgeglichen werden können
    * Für Teilnehmende wie auch für den Dozierende muss eine eigene Umgebung zusammenstellbar sein, damit Umfang und Übersicht je nach Notwendigkeit angepasst werden kann. 

# 2.  Komponenten des Systems
## Hardware-Basis
Das Basissystem enthält alle weiteren Komponenten, die zur Realisierung benötigt werden. Hierbei wird von einem Rechnersystem ausgegangen, dass auf x86_64-kompatibler Hardware basiert. Die Ausprägung ist dabei nicht relevant, sofern die benötigte Software dafür verfügbar ist. Im vorliegenden Beispiel wird ein System verwendet, dass mit 6 CPU-Kernen arbeitet, die auch eine von KVM unterstützte Virtualisierungsschnittstelle bieten. Weiterhin werden 32GB RAM und 2 TB Festplattenkapazität zur Verfügung gestellt. Diese Ausstattung reicht je nach Anforderung für 3 bis 5 virtuelle Desktops.
Bei der Auswahl der Komponenten sollte auf Stabilität und Leistungsfähigkeit geachtet werden, üblicherweise sind Server-Komponenten (Xeon/Opteron, ECC-Speicher, Server-Festplatten) die bessere Wahl.

## Basis-Betriebssystem
Als zu verwendendes Betriebssystem für das Basissystem wird Linux gewählt, als Distribution ist hier entweder Debian 11 (Bullseye) oder Ubuntu 22.04 zu verwenden. Andere aktuelle Distributionen sind sicherlich auch verwendbar, doch bezieht sich die nachfolgende Beschreibung auf diese beiden Varianten. Dieses Basis-System kann, muss aber nicht als Arbeitsplatz zur Steuerung des Virtualisierers verwendet werden, daher kann auf eine Grafische Oberfläche verzichtet werden, sofern ein separater Arbeitsplatz für die Systemsteuerung genutzt wird.

## Virtualisierer
Als Virtualisierer wird Oracle Virtualbox verwendet, das die Virtualisierungsform „KVM“ verwendet, um seine Virtuellen Maschinen bereit zu stellen. Bei der Konfiguration ist darauf zu achten, dass die „Headless“ Variante verwendet wird..
Virtualbox stellt die Konsolen der virtualisierten Maschinen per default über das VNC-Protokoll zur Verfügung, das leider einige Schwächen in der Tastatur- und Multimedia-Übertragung mit sich bringt. Daher wird zusätzlich das „Extension Pack“ benötigt, das als zusätzliche Schnittstelle das RDP bereit stellt und damit einen größeren Funktionsumfang bietet.
Hier ist anzumerken, dass Die Software „Virtualbox“ von Oracle unter die GnuGPL gestellt wirde, nicht jedoch das „Extension Pack“, das in dieser Konfiguration verwendet wird. Tatsächlich ist das ExtensionPack für private Zwecke, für Ausbildung frei benutzbar, nicht aber für kommerzielle Zwecke. Genaueres ist der FAQ-Seite von Oracle zu entnehmen: https://www.virtualbox.org/wiki/Licensing_FAQ. 
Als Bedienungsoberfläche für den Virtualisierer wird „phpVirtualBox“ zusammen mit dem Web-Server Apache und PHP verwendet. Der Zugang und der Zugriff zu diesem System sollte entsprechend abgesichert werden, damit niemand unbefugt Zugriff auf die Virtualisierungssteuerung erhält. 

## Benutzer-Umgebung
Die für die End-Benutzer*innen bereitgestellte Oberfläche wird über Apache Guacamole zusammen mit Apache Tomcat und Java OpenJDK realisiert. Es handelt sich dabei um eine Vermittlungs-Ebene zwischen verschiedenen Konsolen-Oberflächen-Protokollen und dem Web-Browser der nutzenden Person. Zusätzlich wird eine Steuerungs-Umgebung mit diversen Schnittstellen zu Authentifizierungssystemen (LDAP, Datenbanken) bereit gestellt. In der beschriebenen Konfiguration wird aber der Einfachheit halber die native XML-Datei basierte Konfiguration genutzt. 
Diese Benutzerumgebung wird nicht zusammen mit der Steuerung der Virtualisierungs-Umgebung betrieben, sondern in einem separaten System, in diesem Fall auf einer eigenen VM im Virtualisierer, damit auf keinen Fall der Zugriff auf das Basissystem möglich wird, sollte die Benutzer-Oberfläche missbräuchlich genutzt werden, wie z.B. durch Einschleusung von Malware .

# 3. Schematischer Aufbau
Das Basisystem wird mit Betriebssystem sowie dem Virtualisierer (VirtualBox) und dessen Web-Schnittstelle (phpVirtualBox) ausgerüstet. Der Webserver bietet dabei lediglich die Kontroll-Oberfläche für den Virtualsierer, beides soll unabhängig von Benutzereingriffen automatisch starten, wenn das Betriebssystem gestartet wird. Dieses System wird mit einer breitbandigen (mind. 1GBit/s) Netzwerkschnittstelle an das lokale Netzwerk angebunden, im Idealfall werden zwei unterschiedliche Anschlüsse für die Administration und die VDI-Nutzung verwendet, 

Innerhalb des Virtualisierers wird eine Virtuelle Maschine (VM) zur Bereitstellung der Benutzer-Schnittstelle (Guacamole) bereitgestellt. Ihre Netz-Schnittstelle wird öffentlich verfügbar gemacht, so dass Teilnehmer wie auch Dozierende auf dieses System mittels Web-Browser Zugriff bekommen. 
Weiterhin werden beliebig viele VMs für die Bereitstellung an die Teilnehmenden erzeugt, beschränkt durch die Ressourcen des umgebenden Basis-Systems
Der Guacamole-Server übernimmt seinerseits die Kommunikation zu den Konsolen der virtuellen Maschinen, die sich ihrerseits in einem gekapselten Netz und somit nicht im öffentlich zugänglichen Bereich befinden. Weiterhin übernimmt er auch die Authentifizierung sowie die Auswahl und Bereitstellung der Maschinen an die Teilnehmenden.

# 4.  Realisierung
##  Basis-System
Das Basis-System wird mit einem Linux ausgestattet, das die benötigten Komponenten aufnehmen kann. Die nachfolgenden Beschreibungen beziehen sich auf Debian 11 oder Ubuntu 20.04. Der Vorgang selbst soll hier aber nicht beschrieben werden, weil er bereits hinreichend an anderen stellen beschrieben wurde.  Daher sei hier auf eine Standard-Installation eines Debian- oder Ubuntu-Systems verwiesen:
Debian: https://www.debian.org/releases/testing/installmanual
Ubuntu: https://help.ubuntu.com/lts/installation-guide/index.html 

## VirtualBox „Headless“
Nach der Installation des Linux-Betriebssystem sind die benötigten Ergänzungen und Anpassungen für die Virtualisierung vorzunehmen. Die folgenden Anweisungen sind als Shell-Befehle zu verstehen und als User „root“ auszuführen, sofern nichts anderes beschrieben wird.
Zuerst werden die benötigten Software-Pakete installiert:
apt install virtualbox virtualbox-ext-pack

`apt install apache2 php php-mysql libapache2-mod-php php-soap php-xml`

Anschließend wird der Runtime-User „vbox“ für Virtualbox angelegt. Das Passwort (hier: XXXXX) muss gegen ein selbst gewähltes, hinreichend komplexes ersetzt werden:
```
useradd -G vboxusers -d /opt/vbox -u 150 -s /bin/bash -m vbox

passwd vbox
New password: XXXXX
```

Die Konfigurationsdatei /etc/default/virtualbox muss angepasst werden, so dass sie diese Zeile enthält:
`VBOXWEB_USER=vbox`

In der Systemd-Startdatei /lib/systemd/system/vboxweb.service ist in vielen Fällen die falsche PID-Datei angegeben. Sie muss korrigiert werden. Diese Zeile muss in der genannten Datei enthalten sein:
`PIDFile=/var/run/vboxweb-service.sh`
Sollte as nicht der Fall sein, wird die Datei  /lib/systemd/system/vboxweb.service nach  /etc/systemd/system/vboxweb.service kopiert und der PIDFile Eintrag dort entsprechend geändert.

Danach muss der Dienst mit diesen Befehlen neu gestartet werden:
```
systemctl daemon-reload
systemctl start vboxweb
```
Zum Test, ob der Service korrekt gestartet wurde, kann netstat verwendet werden. Hier muss mindestens eine Zeile mit dem Prozess "vboxwebsrv" ausgegeben werden:
```
netstat -lnp|grep vbox

tcp6       0      0 ::1:18083               :::*                    LISTEN      337850/vboxwebsrv
```

Um die Web-Oberfläche einzurichten, muss zunächst phpvirtualbox als Paket herunter geladen und innerhalb der Apache-Konfiguration bereit gestellt werden:
cd /opt/vbox

```
wget https://github.com/phpvirtualbox/phpvirtualbox/archive/develop.zip

unzip develop.zip

mv phpvirtualbox-develop /var/www/phpvirtualbox
```

In dem neu erzeugten Verzeichnis befindet sich eine Beispiel-Konfigurationsdatei config.php-example, die als Grundlage für eine eigene Konfigurationsdatei verwendet werden kann. Zunächst wird sie nach config-php kopiert:
```
cd /var/www/phpvirtualbox
chown -R www-data. .
cp config.php-example config.php
```
In dieser neuen Datei config.php muss für den User „vbox“ das oben festgelegte Passwort im Klartext (!) eingetragen werden:
```
/* Username / Password for system user that runs VirtualBox */
var $username = 'vbox';
var $password = 'XXXXXXX';
```

In derselben Datei muss die Kommunikationsschnittstelle mit VirtualBox bekannt gegeben weden:
```
/* SOAP URL of vboxwebsrv (not phpVirtualBox's URL) */
var $location = 'http://localhost:18083/';
```

Ist das Extension Pack (virtualbox-ext-pack) und das Paket freerdp2-x11 installiert, kann RDP verwendet werden, um die Konsole der VMs zu verwenden. Ein- oder umgeschaltet wird der RDP-Server mit dem Befehl
VBoxManage setproperty vrdeextpack "Oracle VM VirtualBox Extension Pack"

## VM-Installation mit phpVirtualbox
PhpVirtualBox wird über einen Browser  geöffnet, die Adresse ergibt sich aus dem Namen oder der Adresse  des Basis-Systems und dem erzeugten Verzeichnis im Apache Web-Verzeichnis. „phpvirtualbox“: http://serverrechner/phpvirtualbox Benutzername wie auch Passwort lauten initial „admin“, das sofort geändert werden muss. Weitere Benutzer können anschließend manuell hinzu gefügt werden.
Für den aktuellen Zweck wird zuerst eine VM angelegt, die das Benutzer-Interface (Guacamole) bereit stellt. Dazu muss über die Funktion „New“ eine Neue VM angelegt werden. In dem erscheinenden Dialog werden folgende Einstellungen vorgenommen:
    * Name: VDI
    * Type: Linux
    * Version: Ubuntu (64-bit)
    * Memory size: 3072 MB
    * Hard Disk: „Create a virtual Hard disk now“
    * Hard Disk file type: VDI 
    * Storage: Dynamically allocated
    * File size: 20GB
Anschließend muss die neu erstellte VM noch einmal geändert werden damit das Netz-Interface auf „Bridged“ umgestellt werden kann. Das ist in sofern wichtig, weil anderenfalls weder das spätere Benutzerinterface freigegeben noch die Konsolen der weiteren VMs erreicht werden können. Des Weiteren wird in das virtuelle DVD-Laufwerk eine passende Ubuntu-Installations-ISO-Datei „eingelegt“, so dass die Installation durchgeführt werden kann.
Nach erfolgreicher Erzeugung der VM sollte die Konfiguration wie im folgenden Bild aussehen (die weiteren VMs im linken Bereich werden später hinzu gefügt).

Nun muss diese erste VM mit dem Betriebssystem installiert werden, wofür ein Zugang zu der Konsole benötigt wird. Da jetzt noch nicht das Guacamole-User-Interface existiert, muss die Installation direkt über einen RDP Zugang erfolgen. Dazu muss vor dem Start der VM, die Konfiguration so geändert werden, dass diese Konsole auch von außerhalb des Basis-Rechners erreichbar ist. Leider kann dieses nicht in der Web-Oberfläche erfolgen, sondern erfordert einen manuellen Eingriff in die Konfiguration der VM. Die zu ändernde Datei befindet sich im Home-Directory des oben angelegten Users „vbox“, also nach der beschreibenen Konfiguration 
/opt/vbox/VirtualBox Vms/VDI/VDI.vbox

Dort befindet sich in der in dem Abschnitt „RemoteDisplay“ die IP-Adresse, an die die RDP-Funktion gebunden wird. Sie muss die Adresse des Basis-Systems enthalten. In diesem Beispiel wäre es 192.168.1.10 und sähe demnach so aus:
```
     <RemoteDisplay enabled="true"> 
       <VRDEProperties> 
          <Property name="TCP/Address" value="192.168.1.10"/> 
          <Property name="TCP/Ports" value="9000"/> 
       </VRDEProperties> 
     </RemoteDisplay>
```

Wird nun die VM gestartet, kann mit einem RDP-Client die Konsole geöffnet werden. Da unter „TCP/Ports“ der Wert 9000 angegeben wurde, lautet ein Aufruf mit dem Programm xfreerdp:
`xfreerdp /v:192.168.1.10:9000`

Oder alternativ auch:
`rdesktop-vrdp -kde 192.168.1.10:9000`

Die Ubuntu- Installation kann nun in dieser Konsole analog zu dem Basis-System erfolgen, eine grafische Oberfläche muss hier jedoch nicht installiert werden.
Anschließend können weitere VMs eingerichtet werden, so wie sie für die Schulung benötigt werden.
In diesem Beispiel wurde eine VM ohne eigene Festplatte, aber mit einem DVD-Laufwerk eingerichtet, das eine „Live“- Installations-Datei enthält, wie sie beispielsweise von ubuntu.com herunter geladen werden kann. 

Bei jeder weiteren VM sollte manuell der Port festgelegt werden, um für jede Maschine einen festen Port zuzuordnen, damit im Guacamole-Interface auch die „richtige“ Maschine erreicht werden kann:

Weiterhin muss darauf geachtet werden, dass als Netzwerk „NAT“ verwendet wird, damit keine Kollisionen mit dem lokalen Netz entstehen. In größeren Umgebungen kann hier natürlich ein separates, geroutetes Netz zurück gegriffen werden.
Das Admin-Interface bietet weiterhin die Möglichkeit, VMs zu „klonen“, zu im- und exportieren sowie Snapshots von dem aktuellen Zustand anzulegen. Letztes bietet den Vorteil, dass die im Rahmen einer Schulung vorgenommenen Änderungen wieder zurück genommen werden können.
Ein Umfangreiches Handbuch zu diesem System findet sich unter https://www.virtualbox.org/manual/

## Installation des Guacamole Severs
In der oben erzeugten VM („VDI“) wird das Benutzer-Interface eingerichtet, über das die Schulungs-VMs den Teilnehmer*innen zur Verfügung gestellt werden. Es wird analog zu dem Basisystem mit einem Minimal-Linux (Ubuntu/Debian) ausgerüstet und anschließend mit den folgenden Schritten mit dem Guacamole-Server ausgestattet. Sofern im Text nicht anders beschrieben, werden alle Befehle in der Shell (bash) mit root-Rechten ausgeführt:
Als Vorbereitung werden die benötigten Software-Pakete mit diesem Befehl installiert (alles in einer Zeile):
```
apt-get install make gcc g++ libcairo2-dev libjpeg-turbo8-dev libpng-dev libtool-bin libossp-uuid-dev libavcodec-dev libavutil-dev libswscale-dev freerdp2-dev freerdp2-x11 libpango1.0-dev libssh2-1-dev libvncserver-dev libtelnet-dev libssl-dev libvorbis-dev libwebp-dev -y
```

Anschließend erfolgt die Installation des Tomcat-Servers, der die Webserver-Funktion übernehmen wird:
`apt-get install tomcat9 tomcat9-admin tomcat9-common tomcat9-user -y`

Obwohl die Guacamole-Software im Paketsystem des Ubuntu oder Debian auffindbar ist, wird diese nicht verwendet, da sie fehlerbehaftet und veraltet ist. Statt dessen wird die Kompilierung und Installation manuell vorgenommen, was aber ebenfalls nicht sehr aufwändig ist:
Zunächst erfolgt der Download des Guacamole-Servers, der die RDP-Verbindung mit den Konsolen der VMs aufnehmen wird. Dazu wird der Ordner /opt als Basis verwendet.:
```
cd /opt

# der folgende Befehl wird in eine einzige Zeile geschrieben
wget https://downloads.apache.org/guacamole/1.2.0/source/guacamole-server-1.2.0.tar.gz

Anschließend erfolgt die Kompilierung des Guacamole-Servers. 
tar -xvzf guacamole-server-1.2.0.tar.gz

cd guacacd guacamole-server-1.2.0
./configure --with-init-dir=/etc/init.d

make
make install

ldconfig
```

Die Software ist nun installiert und kann mit dem Systemd-System gestartet werden
```
systemctl daemon-reload
systemctl enable guacd
systemctl start guacd
```

Zur Kontrolle wird der Status des Service ausgegeben, dessen Zustand „running“ wiedergeben sollte:
`systemctl status guacd`

Als Guacamole-Clients wird der Teil des Systems bezeichnet, der die Kommunikation mit dem/der Benutzer*in aufnimmt. Es handelt sich hier um eine Tomcat-Applikation, die lediglich in das richtige Verzeichnis gebracht werden muss. Zuerst wird der Tomcat-Server gestoppt.
systemctl stop tomcat9

Anschließend wird das Applikationspaket als „WAR“-Datei herunter geladen. Der Befehl wird komplett in eine einzige Zeile geschrieben.
`wget https://downloads.apache.org/guacamole/1.2.0/binary/guacamole-1.2.0.war`

Die heruntergeladene Datei wird in ein neues Verzeichnis kopiert und dieses anschließend mit dem Tomcat-Server verlinkt:
```
mkdir /etc/guacamole
mv guacamole-1.2.0.war /etc/guacamole/guacamole.war

ln -s /etc/guacamole/guacamole.war /var/lib/tomcat9/webapps/
```
Anschließend werden der Guakamole-Server und der Client neu gestartet
```
systemctl start tomcat9
systemctl restart guacd
```

##  Konfiguration des Guacamole-Clients
Die Datei /etc/guacamole/guacamole.properties definiert die Verbindung zwischen dem Tomcat-System (Client) und dem Guacamole-System (Server). Im aktuellen Beispiel werden diese Einstellungen in diese Datei geschrieben:
```
guacd-hostname: localhost
guacd-port:    4822
user-mapping:    /etc/guacamole/user-mapping.xml
```

Zusätzlich werden noch Konfigurationsverzeichnisse und eine Umgebungsvariable für den Tomcat-Server benötigt:
```
mkdir /etc/guacamole/extensions
mkdir /etc/guacamole/lib
echo "GUACAMOLE_HOME=/etc/guacamole" >> /etc/default/tomcat9
```

Der Verweis aus der guacamole.properties konfiguriert die Benutzerauthentifizierung als Text in der neu zu erstellenden  XML-Datei /etc/guacamole/user-mapping.xml. Alle Benutzer-Kennungen werden hier festgelegt und einer oder mehrerer Konsolen zugeordnet. In diesem Beispiel wird dem User „admin“ zwei VMs zugeordnet, den Usern VHS01 und VHS02 jeweils nur eine, gleichlautende Konsole. Bindeglied ist hier IP-Adresse und Port, wie sie in der VM-Konfiguration verwendet wurden. Passwörter können hier entweder im Klartext oder als verschlüsselter Hash abgelegt werden.
```
<user-mapping>
    <authorize
            username="admin"
            password="XXXXXX">

        <connection name="VHS01">
            <protocol>rdp</protocol>
            <param name="hostname">192.168.1.10</param>
            <param name="port">9001</param>
            <param name="server-layout">de-de-qwertz</param>
            <param name="ignore-cert">true</param>
            <param name="width">1440</param>
            <param name="height">900</param>
        </connection>
        <connection name="VHS02">
            <protocol>rdp</protocol>
            <param name="hostname">192.168.1.10</param>
            <param name="port">9002</param>
            <param name="server-layout">de-de-qwertz</param>
            <param name="ignore-cert">true</param>
            <param name="width">1440</param>
            <param name="height">900</param>
        </connection>
    </authorize>

    <authorize
            username="VHS01"
            password="XXXXXX">

        <connection name="VHS01">
            <protocol>rdp</protocol>
            <param name="hostname">192.168.1.10</param>
            <param name="port">9001</param>
            <param name="server-layout">de-de-qwertz</param>
            <param name="ignore-cert">true</param>
            <param name="width">1440</param>
            <param name="height">900</param>
        </connection>
    </authorize>

    <authorize
            username="VHS02"
            password="XXXXXX">

        <connection name="VHS02">
            <protocol>rdp</protocol>
            <param name="hostname">192.168.1.10</param>
            <param name="port">9002</param>
            <param name="server-layout">de-de-qwertz</param>
            <param name="ignore-cert">true</param>
            <param name="width">1440</param>
            <param name="height">900</param>
        </connection>
    </authorize>

</user-mapping>
```

Nach der Änderung der Konfigurationsdatei ist es sinnvoll die Services, zumindest aber den Tomcat-Service neu zu starten:
```
systemctl restart tomcat9
systemctl restart guacd
```

## Einbindung in Apache
In einigen Fällen lohnt es sich, die Tomcat-Dienste über einen Webserver zu kapseln, beispielsweise um gemeinsame SSL-Zertifikate nutzen zu können. In diesem Fall wird zunächst der Webserver Apache installiert, was recht einfach mit dem Befehl
apt install apache2

geschieht. Damit dieser Verbindungen direkt zum Tomcatserver weiterleitet, der statt auf 80 den Port 8080 verwendet, muss die Datei /etc/apache2/sites-available/guacamole.conf mit diesem Inhalt erzeugt werden
```
       <Location /guacamole/> 
           Require all granted 
           ProxyPass http://localhost:8080/guacamole/ flushpackets=on 
           ProxyPassReverse http://localhost:8080/guacamole/ 
       </Location> 

       <Location /guacamole/websocket-tunnel> 
           Require all granted 
           ProxyPass ws://localhost:8080/guacamole/websocket-tunnel 
           ProxyPassReverse ws://localhost:8080/guacamole/websocket-tunnel 
       </Location>
```
Anschließend muss sie mit dem Befehlen
a2enmod guacamole
`systemctl restart apache2`

aktiviert werden. Nun kann auch Letsencrypt für die Erzeugung eines SSL-Zertifikates genutzt werden, wie beispielsweise unter https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-ubuntu-20-04-de beschrieben.
## Benutzerinterface von Guacamole
Um die VMs nutzen zu können, muss die Adresse des in 4.4 erzeugten und gestarteten Guacamole-Servers geöffnet werden. Die URL setzt sich aus dem Namen des Servers und dem Verzeichnis „/guacamole“ zusammen, so dass sie dem Schema „http://guacamoleserver/guacamole/“ entspricht.
Das User-Interface Guacamole bietet die in der Konfigurationsdatei angelegten Einträge als Liste, bereits geöffnete Verbindungen werden zu Übersicht im oberen Bereich als „Thumbnails“ dargestellt.

Wird ein solches Element geöffnet, wird der Desktop in seiner aktuellen Auflösung im Browserfenster eingepasst, was bedeutet, dass die Ansicht durch vergrößern/verkleinern des Browserfensters angepasst werden kann. Daher ist zu empfehlen, das Browserfenster erst auf die gewünschte Größe zu setzen und anschließend den sichtbaren Desktop auf eine passende Auflösung einzustellen. Sollten Ränder abgeschnitten werden, hilft üblicherweise, den Browserinhalt zu aktualisieren (Taste F5)


Ein gemeinsames Betrachten oder Bearbeiten ist möglich, so dass Hilfestellungen während einer Sitzung gegeben werden können. Voraussetzung ist allerdings die aktivierte Option „Allow Multiple Connections“ in der Display-Konfiguration der VM unter Virtualbox. 
In einer geöffneten Sitzung kann der „Back“-Button des Browsers wieder auf die Auswahl zurück führen, oder aber der direkte Link auf die Guacamole-Oberfläche. Ein Lesezeichen ist hier hilfreich.
Wurden in der aktuellen Sitzung bereits andere Desktops geöffnet, erscheinen diese als Vorschau in der unteren rechten Ecke mit dem aktuellen – wenn auch verkleinerten – Inhalt. Ein schneller Wechsel ist auf diese Weise möglich.
Ein umfangreiches Handbuch ist unter https://guacamole.apache.org/doc/gug/ zu finden.

# 5.  Fazit
Die vorliegende Konfiguration stellt ein VDI-System dar, dass  für eine Schulung in kleinem Rahmen nutzbar ist und eine überschaubare Menge an Komponenten verwendet. Sie benötigt wenig Ressourcen und ist zudem fast vollständig mit Freier Software oder aber zumindest mit frei nutzbarer Software aufgebaut. 
Die Leistungsfähigkeit des System hängt direkt von der Hardware ab und ist nicht an Lizenz-Keys, rechtlichen oder anderen Einschränkungen gebunden. Ein Wechsel auf andere Hardware oder auch eine Erweiterung durch zusätzliche Hardware-Komponenten ist somit jederzeit möglich, so dass eine Adaption an eine größere Umgebung möglich ist.
Durch die Offenheit des Systems können weiterhin Teil-Komponenten durch andere, gegebenenfalls passendere Komponenten ausgetauscht werden, ohne dass das Gesamtkonzept darunter leidet. Sie kann daher auch als Grundlage für ähnliche Anforderungen genutzt werden.
