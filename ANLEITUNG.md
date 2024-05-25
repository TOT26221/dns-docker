
# Anleitung: Einrichtung eines DNS-Systems in Docker für Schüler

## Ziel
Diese Anleitung erklärt Schritt für Schritt, wie du ein DNS-System in Docker simulieren kannst. Du wirst einen DNS-Server und einen Client in separaten Docker-Containern einrichten und konfigurieren. Diese Übung hilft dir, die Grundlagen von DNS (Domain Name System) und Docker zu verstehen.

## Benötigte Werkzeuge
- Docker Desktop oder Docker Engine (installiert und lauffähig)
- Grundlegendes Verständnis von Terminal- oder Command-Prompt-Befehlen
- Texteditor deiner Wahl (z.B. Notepad++, Visual Studio Code)

## Schritte

### Schritt 1: Vorbereitung deiner Arbeitsumgebung
1. **Öffne das Terminal** (Linux/Mac) oder den Command Prompt (Windows).
2. **Erstelle ein neues Verzeichnis** für dein Projekt und wechsle in dieses Verzeichnis:
   ```
   mkdir dns-docker
   cd dns-docker
   ```

### Schritt 2: Erstellen des Docker-Netzwerks
1. **Erstelle ein Docker-Netzwerk** mit dem Namen `vdnsnet`:
   ```
   docker network create --gateway 192.168.0.1 --subnet 192.168.0.0/24 vdnsnet
   ```

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

## ERKLÄRUNG - BEZÜGLICH DER NEUSTEN VERÄNDERUNGEN - STAND 23.4.2024:
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

### Schritt 5: Erstellen der Docker Compose-Datei
1. **Erstelle eine Datei namens `docker-compose.yml`** in deinem Projektverzeichnis. Füge den folgenden Inhalt hinzu, um die Services zu definieren:
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

**version: '3'**: Version der Docker-Compose-Datei.

**services:** Definiert die Dienste.

## dns:
**build:** Baut das Image aus dem aktuellen Verzeichnis mit DNSDockerfile.

**container_name:** Setzt den Namen des Containers auf dns-server.

**networks:** Verbindet den Container mit dem Netzwerk vdnsnet und weist ihm die IP 192.168.0.10 zu.


## client:
**build:** Baut das Image aus dem aktuellen Verzeichnis mit ClientDockerfile.

**container_name:** Setzt den Namen des Containers auf dns-client.

**networks:** Verbindet den Container mit dem Netzwerk vdnsnet und weist ihm die IP 192.168.0.20 zu.

**depends_on:** Startet den client-Dienst erst nach dem dns-Dienst.


## networks:
**vdnsnet:** Verwendet ein externes Netzwerk namens vdnsnet.



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

### Schritt 8: Aufräumen
1. **Beende und entferne die Container**, wenn du fertig bist:
   ```
   docker-compose down
   ```

**Glückwunsch!** Du hast erfolgreich ein DNS-System in Docker eingerichtet und getestet. Diese Übung hilft dir, die Konzepte von DNS und die Nutzung von Docker für Netzwerksimulationen zu verstehen.
