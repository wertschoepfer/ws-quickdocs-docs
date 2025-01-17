# Anleitung für die Einrichtung von QuickDocs mit Docker auf Windows

Beschrieben ist die besipielhafte Einrichtung als Docker-Service auf einem beliebigen Windows PC.  
Dies ist insb. zum Testen der Anwendung und der Nutzung von Docker gedacht.

Siehe auch [🌐 die offizielle Dokumentation von Docker](https://docs.docker.com/manuals/).

[🔙 Zurück zur Übersicht](_toc.md)

## Voraussetzungen

-   Installierter Docker Desktop mit aktiven Linux-Containern
-   Aktuelles QuickDocs Image
-   Aktueller Syncfusion-Lizenzschlüssel
-   Aktuelle QuickDocs-Lizenzdatei (`wsqd.lic`)
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

Bis auf nötige Lizenzangaben ist die Anwendung im Auslieferungszustand mit allen Standardeinstellungen ausführbar.

Standardmäßig wird `/quickdocs` im Container als Datenordner angesehen (siehe Volume/Mount oben).  
Zusätzlich ist `/app` der Ort, an dem QuickDocs liegt / ausgeführt wird.

Die Konfiguration kann folgend eingesehen werden:  
-> `C:\QuickDocs\: docker compose run --rm --entrypoint "bash -c" quickdocs "cat appsettings.json"`

### Abweichende Konfiguration

Für eine Abweichende QuickDocs-Konfiguration muss die sog. **appsettings.json** mit einer eigenen **appsettings.Custom.json** ersetzt werden.  
Dafür zuerst folgende Umgebungsvariable setzen: `QUICKDOCS_APPSETTINGS_REPLACEMENT=Custom`  
Anschließend folgendes Volume hinzufügen: `C:/QuickDocs/appsettings.json:/quickdocs/appsettings.Custom.json:ro`  
Danach kann eine beliebige `C:/QuickDocs/appsettings.json` verwendet werden.

Alternativ zur manuellen Anpassung der `docker-compose.yml`  
kann auch eine Erweiterung `C:\QuickDocs\docker-compose.override.yml` erstellt werden, mit folgendem Inhalt:
(wird automatisch von `docker compose` [🌐 gemergt](https://docs.docker.com/compose/how-tos/multiple-compose-files/merge/))

```yml
services:
    quickdocs:
        volumes:
            - C:/QuickDocs/appsettings.json:/app/appsettings.Custom.json:ro
        environment:
            - QUICKDOCS_APPSETTINGS_REPLACEMENT=Custom
```

Wir empfehlen alle manuellen Überschreibungen/Anpassungen/Erweiterungen hier einzufügen, damit spätere Updates einfacher sind.

### Schriftarten

Für eine Konfiguration der installierten **Schriftarten** kann folgender Befehl verwendet werden:  
-> `C:\QuickDocs\: docker compose run -it --rm -w /usr/share/fonts --entrypoint "bash" --user root quickdocs`

Weitere Informationen: [📄 Settings](settings.md)

### Datenbank

QuickDocs nutzt eine SQLite Datenbank. Zur Erstellung die Migration starten.  
-> `C:\QuickDocs\: docker compose run --rm --entrypoint "/app/WS.QuickDocs.Api.Migrate.exe" quickdocs`

Dies erstellt die Datenbank im Datenordner.  
Der genaue Pfad / `ConnectionString` kann in den [📄 Settings](settings.md) angepasst werden.

## Lizenzen

### Syncfusion-Lizenzschlüssel

Auf dem PC die Umgebungsvariable `SYNCFUSION_LICENSE_KEY` auf den aktuellen Schlüssel setzen.  
Die Lizenz wird vom PC übernommen oder kann alternativ direkt in der `docker-compose.yml`-Datei angegeben werden.

### QuickDocs-Lizenz

Die (gültige) QuickDocs Lizenzdatei in den Datenordner `C:\QuickDocs\wsqd.lic` ablegen.

Der genaue Pfad zur Lizenz kann in den [📄 Settings](settings.md) definiert werden.

## Start

Ist der **Datenordner** vorbereitet, die **Konfiguration** gesetzt und die **Datenbank** erstellt kann QuickDocs gestartet werden.  
-> `C:\QuickDocs\: docker compose up -d quickdocs`

### Warnungen beim Starten

Üblicherweise werden beim Starten von QuickDocs 2 Warnungen ausgegeben (sowohl in der Konsole, als auch im Log).  
Die Meldungen sind nicht relevant für die Ausführung von QuickDocs und können ignoriert werden.

**Beispiel:**

```
Storing keys in a directory '/root/.aspnet/DataProtection-Keys' that may not be persisted outside of the container.
Protected data will be unavailable when container is destroyed. For more information go to https://aka.ms/aspnet/dataprotectionwarning

No XML encryptor configured. Key {212da261-ee3d-4de8-827c-1afeaa63f0a7} may be persisted to storage in unencrypted form.
```

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
