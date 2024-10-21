# Anleitung für die Einrichtung von QuickDocs mit Docker auf Windows

Beschrieben ist die besipielhafte Einrichtung als Docker-Service auf einem beliebigen Windows PC.
Dies ist insb. zum Testen der Anwendung und der Nutzung von Docker gedacht.

[🔙 Zurück zur Übersicht](_toc.md)

## Voraussetzungen

-   Installierter Docker Desktop mit aktiven Linux-Containern
-   Aktuelles QuickDocs Image
-   Aktuelle Syncfusion Lizenz
-   Zum Testen: 1 Wordvorlage (mit MERGEFIELDs)

## Installation

Zuerst wird der sog. **Datenordner** vorbereitet. Bestehende Vorlagen können dabei direkt abgelegt werden.

-   **Datenordner** anlegen:
    -   `C:\QuickDocs`
    -   `C:\QuickDocs\Templates`
    -   `C:\QuickDocs\Results`
    -   `C:\QuickDocs\Temp`
    -   `C:\QuickDocs\Logs`
-   Bestehende **Vorlagen** (Word-Dateien) in `C:\QuickDocs\Templates` ablegen.

Danach eine Datei `C:\QuickDocs\docker-compose.yml` erstellen und folgenden Inhalt einfügen:

```yml
services:
    quickdocs:
        image: quickdocs:latest
        volumes:
            - C:/QuickDocs:/quickdocs:rw
        environment:
            - SYNCFUSION_LICENSE_KEY=$SYNCFUSION_LICENSE_KEY
        ports:
            - '8080:8080'
        deploy:
            resources:
                limits:
                    cpus: '1.00'
                    memory: 1G
        working_dir: /app
```

## Konfiguration

Bis auf die nötige Syncfusion Lizenz ist die Anwendung im Auslieferungszustand mit allen Standardeinstellungen ausführbar.
Dafür auf dem PC die Umgebungsvariable `SYNCFUSION_LICENSE_KEY` auf den aktuellen Schlüssel setzen.
Die Lizenz wird vom PC übernommen oder kann alternativ direkt in der `docker-compose.yml`-Datei angegeben werden.

### Abweichende Konfiguration

Für eine Abweichende QuickDocs-Konfiguration muss die sog. **appsettings.json** mit einer eigenen **appsettings.Custom.json** ersetzt werden.
Dafür zuerst folgende Umgebungsvariable setzen: `QUICKDOCS_APPSETTINGS_OVERWRITE=Custom`
Anschließend folgendes Volume hinzufügen: `C:/QuickDocs/appsettings.json:/quickdocs/appsettings.Custom.json:ro`
Danach kann eine beliebige `C:/QuickDocs/appsettings.json` verwendet werden.

### Schriftarten

Für eine Konfiguration der installierten **Schriftarten** kann folgender Befehl verwendet werden:
-> `docker compose -f C:\QuickDocs\docker-compose.yml run -it --rm -w /usr/share/fonts --entrypoint bash --user root quickdocs`

Weitere Informationen: [📄 Settings](settings.md)

### Datenbank

QuickDocs nutzt eine SQLite Datenbank. Zur Erstellung die Migration starten.
-> `docker compose -f C:\QuickDocs\docker-compose.yml run --entrypoint /app/WS.QuickDocs.Api.Migrate.exe quickdocs`

Dies erstellt die Datenbank im Datenordner.
Der genaue Pfad / `ConnectionString` kann in den [📄 Settings](settings.md) angepasst werden.

## Start

Ist der **Datenordner** vorbereitet, die **Konfiguration** gesetzt und die **Datenbank** erstellt kann QuickDocs gestartet werden.
-> `docker compose -f C:\QuickDocs\docker-compose.yml up -d quickdocs`

## Testen & Debuggen im Browser

Wurden alle Schritte oben ausgeführt, dann kann die Anwendung getestet werden.

-   http://localhost:8080/quickdocs/swagger -> sollte die Swagger UI anzeigen. Hier können Funktionen direkt getestet werden.
    Funktioniert dies aus irgendeinem Grund nicht, dann:
-   http://localhost:8080/quickdocs/status/ping -> sollte "Pong" ausgeben.

Zuerst kann man die **Logs** im QuickDocs-Datenordner oder die **Docker Logs** prüfen.

Für ausführlichere Informationen kann das `MinimumLevel` des Loggers auf `Debug` gesetzt werden.
Siehe [📄 Settings](settings.md#anpassung-des-log-level).

Weitere Analysen der Fehler sollten immer mit aktiviertem **Development-Modus** durchgeführt werden.
Siehe [📄 Settings](settings.md#umgebungsvariablen).
