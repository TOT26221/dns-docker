
# Anleitung: Einrichtung eines DNS-Systems in Docker für Schüler

## Ziel
Diese Anleitung erklärt Schritt für Schritt, wie du ein DNS-System in Docker simulieren kannst. Du wirst einen DNS-Server und einen Client in separaten Docker-Containern einrichten und konfigurieren. Diese Übung hilft dir, die Grundlagen von DNS (Domain Name System) und Docker zu verstehen.

## Benötigte Werkzeuge
- Docker Desktop oder Docker Engine (installiert und lauffähig)
- Grundlegendes Verständnis von Terminal- oder Command-Prompt-Befehlen
- Texteditor deiner Wahl (z.B. Notepad++, Visual Studio Code)
- 
## ERKLÄRUNG - BEZÜGLICH DER NEUSTEN VERÄNDERUNGEN - STAND 26.4.2024:

## Schritte

### Schritt 1: Vorbereitung deiner Arbeitsumgebung
1. **Öffne das Terminal** (Linux/Mac) oder den Command Prompt (Windows).
2. **Erstelle ein neues Verzeichnis** für dein Projekt und wechsle in dieses Verzeichnis:
   ```
   mkdir dns-docker
   cd dns-docker
   ```
**mkdir dns-docker:** Erstellt ein neues Verzeichnis mit dem Namen dns-docker.

**cd dns-docker:** Wechselt in das neu erstellte Verzeichnis dns-docker.


### Schritt 2: Erstellen des Docker-Netzwerks
1. **Erstelle ein Docker-Netzwerk** mit dem Namen `vdnsnet`:
   ```
   docker network create --gateway 192.168.0.1 --subnet 192.168.0.0/24 vdnsnet
   ```

**docker network create:** Erstellt ein neues Docker-Netzwerk.

**--gateway 192.168.0.1:** Legt die Gateway-Adresse des Netzwerks auf 192.168.0.1 fest. Das Gateway ist der Ausgangspunkt für Netzwerkverbindungen außerhalb des Docker-Netzwerks.

**--subnet 192.168.0.0/24:** Definiert das Subnetz des Netzwerks als 192.168.0.0/24, was bedeutet, dass die IP-Adressen im Bereich von 192.168.0.0 bis 192.168.0.255 liegen.

**vdnsnet:** Der Name des zu erstellenden Netzwerks.

Zusammengefasst erstellt dieser Befehl ein neues Docker-Netzwerk mit einem bestimmten Gateway und Subnetz.

### Schritt 3: Konfiguration des DNS-Servers
1. **Erstelle die Zonendatei `db.mis`** für deine Domain `mis.lan` mit einem Texteditor und speichere sie in deinem Projektverzeichnis. Füge die folgenden Inhalte ein:
   ```zone
   ; Zone: MIS.LAN.
   $ORIGIN MIS.LAN.
   $TTL 3600
   @ IN SOA mis.lan. hostmaster.mis.lan. (
   2024032401 ;serial
   3600       ;refresh
   600        ;retry
   604800     ;expire
   86400      ;minimum ttl
   )
   @       IN NS  dns1.mis.lan.
   dns1    IN A   192.168.0.10
   client-1 IN A  192.168.0.20
   ```


markdown

### DNS-Zone-Datei Erklärung

### TTL (Time To Live)
```
$TTL 86400
Dies gibt die Standard-Gültigkeitsdauer von DNS-Einträgen in Sekunden an. Hier beträgt die TTL 86400 Sekunden, was 24 Stunden entspricht. Das bedeutet, dass DNS-Einträge für diese Domain 24 Stunden lang zwischengespeichert werden, bevor sie erneut abgefragt werden müssen.

SOA (Start of Authority) Eintrag
```

```
@ IN SOA ns1.mis.lan. admin.mis.lan. (
              2024050501 ; Serial
              7200       ; Refresh
              1800       ; Retry
              1209600    ; Expire
              86400      ; Minimum TTL
)
```

Dieser Eintrag definiert den primären Nameserver für die Zone (ns1.mis.lan.) und die E-Mail-Adresse des Administrators (in diesem Fall admin.mis.lan. wobei das "@"-Zeichen durch einen Punkt ersetzt wird). Die Zahlen in Klammern sind:

**Serial:** Eine Versionsnummer für die Zone. Diese muss bei jeder Änderung der Zone erhöht werden.

**Refresh:** Gibt an, wie oft sekundäre Nameserver die Zone aktualisieren sollen (in diesem Fall alle 2 Stunden).

**Retry:** Gibt an, wie lange sekundäre Nameserver warten sollen, bevor sie nach einem fehlgeschlagenen Aktualisierungsversuch erneut versuchen (hier 30 Minuten).

**Expire:** Gibt an, wie lange sekundäre Nameserver die Zone vorhalten sollen, wenn sie keine Aktualisierung erhalten können (hier 14 Tage).

**Minimum TTL:** Die minimale TTL, die für negative Antworten gilt (hier 24 Stunden).

## Nameserver (NS) Eintrag



```@ IN NS ns1.mis.lan. ```

Dieser Eintrag gibt den autoritativen Nameserver für die Zone an. Das @-Zeichen steht für die Root-Domain der Zone (mis.lan.).

## Mail Exchanger (MX) Eintrag


```
@ IN MX 10 mail.mis.lan.

```

Dieser Eintrag definiert den Mailserver für die Domain. Die Zahl 10 ist die Priorität, wobei niedrigere Zahlen eine höhere Priorität haben. E-Mails an mis.lan. werden an mail.mis.lan. weitergeleitet.

A Einträge


```
ns1     IN A 192.168.100.10
mail    IN A 192.168.100.25
www     IN A 192.168.100.30
```

**ns1 IN A 192.168.100.10:** Weist den Hostnamen ns1.mis.lan. die IP-Adresse 192.168.100.10 zu.

**mail IN A 192.168.100.25:** Weist den Hostnamen mail.mis.lan. die IP-Adresse 192.168.100.25 zu.

**www IN A 192.168.100.30:** Weist den Hostnamen www.mis.lan. die IP-Adresse 192.168.100.30 zu.

**CNAME (Canonical Name) Eintrag**


```
ftp IN CNAME www.mis.lan.
```
Dieser Eintrag gibt an, dass der Hostname ftp.mis.lan. ein Alias für www.mis.lan. ist. Anfragen an ftp.mis.lan. werden an www.mis.lan. weitergeleitet.

TXT Eintrag



```@ IN TXT "v=spf1 mx -all" ```
Dieser Eintrag definiert einen Textstring, der für SPF (Sender Policy Framework) verwendet wird. Der SPF-Eintrag hilft, E-Mail-Spoofing zu verhindern, indem er festlegt, welche Server berechtigt sind, E-Mails im Namen der Domain zu senden. Hier bedeutet v=spf1 mx -all, dass nur der Mailserver, der im MX-Eintrag definiert ist, E-Mails für diese Domain senden darf.



# DNS-Zone-Datei Erklärung

## TTL (Time To Live)
```
$TTL 86400
```

Dies gibt die Standard-Gültigkeitsdauer von DNS-Einträgen in Sekunden an. Hier beträgt die TTL 86400 Sekunden, was 24 Stunden entspricht. Das bedeutet, dass DNS-Einträge für diese Domain 24 Stunden lang zwischengespeichert werden, bevor sie erneut abgefragt werden müssen.

SOA (Start of Authority) Eintrag


```
@ IN SOA ns1.mis.lan. admin.mis.lan. (
              2024050501 ; Serial
              7200       ; Refresh
              1800       ; Retry
              1209600    ; Expire
              86400      ; Minimum TTL
)```
Dieser Eintrag definiert den primären Nameserver für die Zone (ns1.mis.lan.) und die E-Mail-Adresse des Administrators (in diesem Fall admin.mis.lan. wobei das "@"-Zeichen durch einen Punkt ersetzt wird). Die Zahlen in Klammern sind:

**Serial:** Eine Versionsnummer für die Zone. Diese muss bei jeder Änderung der Zone erhöht werden.

**Refresh:** Gibt an, wie oft sekundäre Nameserver die Zone aktualisieren sollen (in diesem Fall alle 2 Stunden).

**Retry:** Gibt an, wie lange sekundäre Nameserver warten sollen, bevor sie nach einem fehlgeschlagenen Aktualisierungsversuch erneut versuchen (hier 30 Minuten).

**Expire:** Gibt an, wie lange sekundäre Nameserver die Zone vorhalten sollen, wenn sie keine Aktualisierung erhalten können (hier 14 Tage).

**Minimum TTL:** Die minimale TTL, die für negative Antworten gilt (hier 24 Stunden).

Nameserver (NS) Eintrag


```@ IN NS ns1.mis.lan.```
Dieser Eintrag gibt den autoritativen Nameserver für die Zone an. Das @-Zeichen steht für die Root-Domain der Zone (mis.lan.).

Mail Exchanger (MX) Eintrag


```@ IN MX 10 mail.mis.lan.```
Dieser Eintrag definiert den Mailserver für die Domain. Die Zahl 10 ist die Priorität, wobei niedrigere Zahlen eine höhere Priorität haben. E-Mails an mis.lan. werden an mail.mis.lan. weitergeleitet.

A Einträge


```ns1     IN A 192.168.100.10
mail    IN A 192.168.100.25
www     IN A 192.168.100.30```
ns1 IN A 192.168.100.10: Weist den Hostnamen ns1.mis.lan. die IP-Adresse 192.168.100.10 zu.
mail IN A 192.168.100.25: Weist den Hostnamen mail.mis.lan. die IP-Adresse 192.168.100.25 zu.
www IN A 192.168.100.30: Weist den Hostnamen www.mis.lan. die IP-Adresse 192.168.100.30 zu.
CNAME (Canonical Name) Eintrag


ftp IN CNAME www.mis.lan.
Dieser Eintrag gibt an, dass der Hostname ftp.mis.lan. ein Alias für www.mis.lan. ist. Anfragen an ftp.mis.lan. werden an www.mis.lan. weitergeleitet.

TXT Eintrag


@ IN TXT "v=spf1 mx -all"
Dieser Eintrag definiert einen Textstring, der für SPF (Sender Policy Framework) verwendet wird. Der SPF-Eintrag hilft, E-Mail-Spoofing zu verhindern, indem er festlegt, welche Server berechtigt sind, E-Mails im Namen der Domain zu senden. Hier bedeutet v=spf1 mx -all, dass nur der Mailserver, der im MX-Eintrag definiert ist, E-Mails für diese Domain senden darf.


2. **Erstelle ein Dockerfile** für den DNS-Server (`DNSDockerfile`) und füge den folgenden Inhalt hinzu:
   ```Dockerfile
   FROM ubuntu/bind9
   COPY db.mis /etc/bind/db.mis
   CMD ["named", "-g", "-c", "/etc/bind/named.conf", "-u", "bind"]
   ```

### Schritt 4: Konfiguration des Client-Containers
1. **Erstelle ein Dockerfile** für den Client (`ClientDockerfile`) und füge den folgenden Inhalt hinzu:
   ```Dockerfile
   FROM ubuntu:20.04
   RUN apt-get update && apt-get install -y dnsutils
   CMD ["bash"]
   ```
 **FROM ubuntu:20.04**: Verwendet das Ubuntu 20.04 Basis-Image.
 
**RUN apt-get update && apt-get install -y dnsutils**: Führt eine Aktualisierung der Paketlisten durch und installiert die dnsutils-Pakete, die DNS-Tools wie dig und nslookup enthalten.

**CMD ["bash"]**: Startet eine Bash-Shell, wenn der Container läuft.

**Zusammengefasst erstellt dieser Dockerfile ein Image basierend auf Ubuntu 20.04, installiert DNS-Tools und startet eine Bash-Shell, wenn der Container ausgeführt wird.**

### NEU:
**Verwende das offizielle Ubuntu-Image als Basis**
FROM ubuntu:20.04

**Installiere erforderliche Tools**
```
RUN apt-get update && \
    apt-get install -y \
    iputils-ping \
    dnsutils \
    net-tools \
    bash-completion \
    mtr-tiny \
    nmap \
    traceroute \
    netcat \
    iproute2 \
    tcpdump \
    isc-dhcp-client && \
    rm -rf /var/lib/apt/lists/* && \
    apt-get clean

Das Start-Kommando hält den Container am Laufen, um Interaktion zu ermöglichen
CMD ["bash"]
```
**Altes Dockerfile:**

Installiert nur dnsutils für grundlegende DNS-Aufgaben wie nslookup und dig.

Sehr einfach und minimalistisch, geeignet für grundlegende DNS-Abfragen.

**Neues Dockerfile:**

**Installiert eine Vielzahl von Netzwerk- und Diagnosetools:**

**iputils-ping:** Zum Pingen von Hosts.

**dnsutils:** Für DNS-Tools wie dig und nslookup

**net-tools:** Enthält Netzwerk-Tools wie ifconfig.

**bash-completion:** Ermöglicht Tab-Vervollständigung in der Bash.

**mtr-tiny:** Ein Netzwerk-Diagnosetool.

**nmap:** Ein Netzwerk-Scanner.

**traceroute:** Verfolgt die Route zu einem Host.

**netcat:** Ein vielseitiges Netzwerk-Diagnosetool.

**iproute2:** Ein Paket für Netzwerkkonfigurations- und Diagnose-Tools wie ip.

**tcpdump:** Ein Netzwerkanalysetool.

**isc-dhcp-client:** Ein DHCP-Client.

Führt ein Aufräumen durch, um den Container schlanker zu halten:

Entfernt temporäre Paketlisten.

Führt apt-get clean aus, um den Cache zu leeren.

Bietet eine umfassendere Einrichtung, die für vielfältige Netzwerkaufgaben und -analysen nützlich ist.

**Gemeinsamkeiten:**

Beide starten eine Bash-Shell, wodurch der Container aktiv und interaktiv bleibt.

### Schritt 5: Erstellen der Docker Compose-Datei
1. **Erstelle eine Datei namens `docker-compose.yml`** in deinem Projektverzeichnis. Füge den folgenden Inhalt hinzu, um die Services zu definieren:
   **ALT:**
```yaml
   version: '3'
   services:
     dns:
       build:
         context: .
         dockerfile: DNSDockerfile
       container_name: dns-server
       networks:
         vdnsnet:
           ipv4_address: 192.168.0.10
     client:
       build:
         context: .
         dockerfile: ClientDockerfile
       container_name: dns-client
       networks:
         vdnsnet:
           ipv4_address: 192.168.0.20
       depends_on:
         - dns
   networks:
     vdnsnet:
       external: true
   ```
   **NEU:**
```yaml
version: '3.8'

services:
  dns1:
    container_name: dns1
    build:
      context: .
      dockerfile: DNSDockerfile
    ports:
      - "53:53/udp"
      - "53:53/tcp"
    volumes:
      - "./named.conf.default-zones:/etc/bind/named.conf.default-zones"
      - "./db.mis:/etc/bind/db.mis"
    networks:
      publicnet:
        ipv4_address: 192.168.50.10
      privatenet:
        ipv4_address: 192.168.100.10

  dns2:
    container_name: dns2
    build:
      context: .
      dockerfile: DNSDockerfile
    ports:
      - "54:54/udp"
      - "54:54/tcp"
    volumes:
      - "./named.conf.default-zones:/etc/bind/named.conf.default-zones"
      - "./db.mis:/etc/bind/db.mis"
    networks:
      publicnet:
        ipv4_address: 192.168.50.11
      privatenet:
        ipv4_address: 192.168.100.11

  client1:
    container_name: client1
    build:
      context: .
      dockerfile: ClientDockerfile
    dns:
      - 192.168.100.10
    networks:
      privatenet: {}
    stdin_open: true
    tty: true

  client2:
    container_name: client2
    build:
      context: .
      dockerfile: ClientAlpineDockerfile
    dns:
      - 192.168.100.11
    networks:
      privatenet: {}
    stdin_open: true
    tty: true
    depends_on:
      - dns2

networks:
  publicnet:
    driver: bridge
    ipam:
      config:
        - subnet: 192.168.50.0/24
  privatenet:
    driver: bridge
    ipam:
      config:
        - subnet: 192.168.100.0/24
 ```
**ALt:**
**version: '3'**: Version der Docker-Compose-Datei.

**services:** Definiert die Dienste.

**dns:**
**build:** Baut das Image aus dem aktuellen Verzeichnis mit DNSDockerfile.

**container_name:** Setzt den Namen des Containers auf dns-server.

**networks:** Verbindet den Container mit dem Netzwerk vdnsnet und weist ihm die IP 192.168.0.10 zu.


**client:**
**build:** Baut das Image aus dem aktuellen Verzeichnis mit ClientDockerfile.

**container_name:** Setzt den Namen des Containers auf dns-client.

**networks:** Verbindet den Container mit dem Netzwerk vdnsnet und weist ihm die IP 192.168.0.20 zu.

**depends_on:** Startet den client-Dienst erst nach dem dns-Dienst.


**networks:**
**vdnsnet:** Verwendet ein externes Netzwerk namens vdnsnet.

## NEU:
**Version**
Dies legt die Version des Docker Compose Formats fest, das verwendet wird. In diesem Fall ist es Version 3.8.

**Services**
Dies definiert die verschiedenen Container, die gestartet werden sollen.

**DNS1**

Container-Name: Der Container heißt "dns1".

Build: Der Container wird aus einem Dockerfile namens DNSDockerfile im aktuellen Verzeichnis erstellt.

Ports: Bestimmte Ports auf dem Host werden zu Ports im Container weitergeleitet, damit der DNS-Server erreichbar ist.

Volumes: Dateien oder Verzeichnisse vom Host werden in den Container gemountet, um Konfigurationsdateien bereitzustellen.

Netzwerke: Der Container wird mit zwei Netzwerken verbunden, einem öffentlichen und einem privaten, mit jeweils festen IP-Adressen.

**DNS2**

Ähnlich wie DNS1, aber mit anderen Ports und IP-Adressen, um einen zweiten DNS-Server bereitzustellen.

**Client1**

Container-Name: Der Container heißt "client1".

Build: Der Container wird aus einem Dockerfile namens ClientDockerfile im aktuellen Verzeichnis erstellt.

DNS: Der Container verwendet den DNS-Server 192.168.100.10.

Netzwerk: Der Container ist mit dem privaten Netzwerk verbunden.

Interaktives Terminal: Erlaubt die Nutzung eines interaktiven Terminals.

**Client2**

Ähnlich wie Client1, aber verwendet ein anderes Dockerfile (ClientAlpineDockerfile) und hängt vom Start des dns2-Containers ab.

**Netzwerke**

Publicnet: Ein öffentliches Netzwerk mit dem Subnetz 192.168.50.0/24.

Privatenet: Ein privates Netzwerk mit dem Subnetz 192.168.100.0/24.

**nochmal die Veränderungen:**
In der alten Docker Compose Datei wird Version 3 verwendet, während in der neuen Datei Version 3.8 verwendet wird, was neuer ist und mehr Funktionen unterstützt. Die alte Datei definiert zwei Services, dns und client, während die neue Datei vier Services hat: **dns1**, **dns2**, **client1** und **client2**.

Der alte dns-Service hat eine feste IP-Adresse, während in der neuen Datei **dns1** und **dns2** unterschiedliche Ports und feste IP-Adressen in zwei verschiedenen Netzwerken haben. Der alte client-Service hat ebenfalls eine feste IP-Adresse und hängt vom dns-Service ab. In der neuen Datei haben **client1** und **client2** dynamische IP-Adressen im privaten Netzwerk, wobei **client2** vom **dns2-Service** abhängt.

Die alte Datei verwendet ein externes Netzwerk namens **vdnsnet**. Die neue Datei definiert zwei interne Netzwerke, publicnet und privatenet, mit festgelegten Subnetzen. Darüber hinaus sind in der neuen Datei Ports und Volumes für **dns1** und **dns2** definiert, um DNS-Dienste und Konfigurationsdateien bereitzustellen. Zudem sind stdin_open und tty für **client1** und **client2** aktiviert, um interaktive Terminals zu ermöglichen.

Insgesamt ist die neue Datei detaillierter und flexibler, unterstützt mehrere DNS-Server und Netzwerke und ermöglicht eine bessere Konfiguration und Isolation der Dienste.




### Schritt 6: Starten der Container
1. **Starte die Container** mit Docker Compose:
   ```
   docker-compose up -d
   ```

**docker-compose up:** Startet die im Docker-Compose-Datei definierten Container.

**-d:** Führt die Container im Hintergrund aus (detached mode).

### Schritt 7: Testen der DNS-Auflösung
1. **Führe eine DNS-Abfrage** vom Client-Container aus:
   ```
   docker exec dns-client dig @192.168.0.10 dns1.mis.lan
   ```

**docker exec dns-client:** Führt einen Befehl im laufenden dns-client Container aus.

**dig @192.168.0.10 dns1.mis.lan:** Verwendet das dig-Tool, um eine DNS-Abfrage für dns1.mis.lan an den DNS-Server mit der IP-Adresse 192.168.0.10 zu senden.

Diese Abfrage prüft, ob der DNS-Server korrekt konfiguriert ist und die DNS-Namensauflösung funktioniert.

**dig (Domain Information Groper)** ist ein Befehlszeilenwerkzeug zur Abfrage von DNS-Servern und zur Anzeige detaillierter Informationen über DNS-Einträge. Es wird häufig zur Diagnose und Fehlerbehebung von DNS-Problemen verwendet, da es detaillierte Antworten und Debugging-Informationen liefert. dig ist flexibel und unterstützt verschiedene Abfragetypen wie A, MX, TXT, und viele mehr.

**Eine Alternative** zu **dig** ist **nslookup**, das ebenfalls DNS-Informationen abfragt, jedoch in einer einfacheren, weniger detaillierten Ausgabeform, was für grundlegende DNS-Abfragen ausreichend sein kann.





### Schritt 8: Aufräumen
1. **Beende und entferne die Container**, wenn du fertig bist:
   ```
   docker-compose down
   ```

**docker-compose down:** Dieser Befehl beendet alle laufenden Container, die durch Docker Compose gestartet wurden, und entfernt sie zusammen mit den Netzwerken, die in der Docker-Compose-Datei definiert sind. Er sorgt dafür, dass alle Ressourcen sauber aufgeräumt werden, sobald sie nicht mehr benötigt werden.


**Glückwunsch!** Du hast erfolgreich ein DNS-System in Docker eingerichtet und getestet. Diese Übung hilft dir, die Konzepte von DNS und die Nutzung von Docker für Netzwerksimulationen zu verstehen.
