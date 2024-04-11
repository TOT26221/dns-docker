
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

### Schritt 6: Starten der Container
1. **Starte die Container** mit Docker Compose:
   ```
   docker-compose up -d
   ```

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
