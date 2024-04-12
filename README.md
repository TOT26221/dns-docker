
# DNS-System in Docker

Dieses Repository enthält die notwendigen Dateien und eine Anleitung, um ein einfaches DNS-System in Docker einzurichten. Es umfasst einen DNS-Server (Bind9) und einen Client, der den DNS-Server für Namensauflösungen verwendet und ihn anpingen kann.

## Voraussetzungen

Bevor Sie beginnen, stellen Sie sicher, dass Sie Docker auf Ihrem System installiert haben. Die Anleitung ist für Linux, Mac und Windows geeignet, sofern Docker verfügbar ist.
## Bedingungen bei der Arbeit mit diesem repository

Bei benutzung wird gefordert das man in einem eigenem Branch arbeitet.
## Schnellstart

1. **Klonen Sie dieses Repository** oder laden Sie die Dateien in ein lokales Verzeichnis herunter.
2. **Aufbauen des Netzwerkes**
    ```bash
      docker network create --gateway 192.168.0.1 --subnet 192.168.0.0/24 vdnsnet
      ```
3. **Navigieren Sie in das Projektverzeichnis** und starten Sie die Docker-Container mit:
   ```bash
   docker-compose up -d
   ```
4. **Testen Sie die DNS-Auflösung und das Pingen** vom Client zum DNS-Server:
   - DNS-Auflösung testen:
     ```bash
     docker exec client dig @192.168.0.10 dns1.mis.lan
     ```
   - Ping den DNS-Server:
     ```bash
     docker exec client ping -c 4 dns1.mis.lan
     ```

## Dateistruktur

- `DNSDockerfile`: Dockerfile für den Aufbau des DNS-Servers.
- `ClientDockerfile`: Dockerfile für den Aufbau des Client-Containers, inklusive `ping` und `dig`.
- `db.mis`: Die Zonendatei für das DNS-System.
- `docker-compose.yml`: Docker Compose-Datei zur Definition und zum Starten der Dienste.

## Anleitung

Für eine detaillierte Anleitung, wie Sie das DNS-System von Grund auf selbst einrichten können, siehe [ANLEITUNG.md](ANLEITUNG.md).

## Beiträge

Fühlen Sie sich frei, Forks zu erstellen, Issues zu eröffnen und Pull-Requests zu senden. Jede Verbesserung oder Vorschlag ist willkommen.

## Bei Fehlern
  ```bash
      git pull
  
      docker compose build
  ```
## Lizenz

Dieses Projekt ist unter der MIT Lizenz lizenziert. Siehe [LICENSE](LICENSE) für Details.
