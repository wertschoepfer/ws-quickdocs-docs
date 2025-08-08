# Einstellungen & Optionen von QuickDocs

Für den optimalen Betrieb von QuickDocs können verschiedene Einstellungen angepasst werden.  
Zum Beispiel können Pfade für Log-Dateien oder die lokale Datenbank geändert werden.

Da QuickDocs auf ASP\.NET aufbaut sind hier einige Standard-Optionen zusätzlich möglich.

[🔙 Zurück zur Übersicht](_toc.md)

## Umgebungsvariablen

Da QuickDocs eine ASP\.NET Anwendung ist, sind Standard ASP\.NET / \.NET Umgebungsvariablen möglich:

-   [🌐 Konfiguration der Garbage Collection](https://learn.microsoft.com/en-us/dotnet/core/runtime-config/garbage-collector)
-   [🌐 Konfiguration des ASP\.NET Hosts](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/host/web-host?view=aspnetcore-8.0#host-configuration-values)

❗ Änderungen an Windows-System-/Benutzer-Umgebungsvariblen werden nur nach einem **Neustart von IIS** wirksam. Für dies in PowerShell als Admin `iisreset /restart` ausführen.

Für QuickDocs sind vorallem folgende Umgebungsvariablen entscheidend:

-   `SYNCFUSION_LICENSE_KEY`
    -   ❗ Ab QuickDocs v1.1 wird automatisch ein Standard-Syncfusion-Lizenzschlüssel verwendet, wenn nicht gesetzt.
    -   Wird für die Dokumentenverarbeitung benötigt und muss zum Start von QuickDocs gesetzt sein.
    -   Der Wert ist der Syncfusion-Lizenzsschlüssel, welcher von WS erstellt und bereitgestellt wird.
    -   Der Schlüssel ist immer genau für eine Version von Syncfusion gültig.
    -   Der Schlüssel kann die Version als Zusatz haben, Beispiel für Version 26: `SYNCFUSION_LICENSE_KEY_26`. Dies ermöglicht ein setzen von mehreren Schlüsseln auf dem gleichen Server.
    -   Im Fehlerfall (falscher oder kein Schlüssel) zeigt QuickDocs Informationen zu gefundenen und dem benötigten Schlüssel im Log an (Level = Fehler).
    -   Schlüssel werden folgend ausgewertet:
        -   Wird ein zur aktuell verwendeter Version passender Schlüssel anhand Namen gefunden, wird dieser bevorzugt, Beispiel: `SYNCFUSION_LICENSE_KEY_26`.
        -   Wird ansonsten der Schlüssel ohne Zusatz gefunden, wird dieser genommen, Beispiel: `SYNCFUSION_LICENSE_KEY`.
        -   Andere Schlüssel werden nicht verwendet.
-   `QUICKDOCS_APPSETTINGS_REPLACEMENT`
    -   Wird für das komplette Überschreiben der `appsettings.json`-Verwendet und ist nur in speziellen Fällen (insb. bei Verwendung von Docker/Containern) nötig.
-   `ASPNETCORE_ENVIRONMENT`
    -   Optional für das Debugging bzw. eine erleichterte Entwicklung.
    -   Mit Wert `Development` wird QuickDocs im **Development-Modus** gestartet.
    -   Der Standardwert für produktive Umgebungen ist `Production`. Dieser Wert ist auch aktiv, wenn keiner gesetzt wird.
    -   ❗ Sollte in produktiven Umgebungen nicht dauerhaft aktiv sein, da bestimmte Sicherheitsmechanismen deaktiviert werden und ggf. sicherheitsrelevante Fehlerinformationen (400/500) mit ausgegeben werden.
    -   ❗ Ist eine Standard-Einstellung von .NET und kann somit Einfluss auf alle laufenden .NET Anwendungen auf dem Server haben, wenn als Umgebungsvariable für das gesamte System gesetzt.

**Umgebungsvariablen** kann man nach der Installation auch im IIS direkt pro Server/Website/Anwendung spezifisch setzen:

-   Konfigurations-Editor der App/Website öffnen
-   Sektion (oben links): `system.webServer/aspNetCore`
-   Von: (oben rechts): `Default Web Site/quickdocs Web.config`
    -   ℹ️ Wird dann direkt in `C:\inetpub\wwwroot\quickdocs\web.config` gespeichert
-   Eigenschaft öffnen: `environmentVariables`
-   Umgebungsvariablen hinzufügen/anpassen

Siehe für erweiterte Szenarien auch folgenden externen Artikel zu Umgebungsvariablen und IIS:  
https://andrewlock.net/setting-environment-variables-in-iis-and-avoiding-app-pool-restarts/

## appsettings.json

Die Konfigurationsdatei ist in mehrere Sektionen aufgeteilt und enthält an den jeweiligen Stellen Kommentare und Referenzen für mehr Details.

Standardpfad auf IIS: `C:\inetpub\wwwroot\quickdocs\appsettings.json`

❗ Pfadangaben in der Datei müssen i.d.R. bei Windows-Systemen mit doppelten Strichen `\\` angegeben werden und sollten am besten absolut sein, können jedoch auch Umgebungsvariablen wie `%APPDATA%` enthalten.

❗ Einstellungen können zusätzlich im Webserver eingetragen werden oder per Umgebungsvariablen konfiguriert (.NET Standard) sein.

### Sektion: Config

Allgemeine, globale Konfigurationen für QuickDocs.

-   `Motd`: Eine "Message of the day", die angezeigt wird, wenn QuickDocs gestartet wird. Damit kann kontrolliert werden, ob die Einstellungen greifen.
-   `TemplateFolder`: Absoluter oder relativer Pfad zum Ordner in dem die Word-Vorlagen abliegen.
-   `TempFolder`: Absoluter oder releativer Pfad zum Ordner in dem temporäre Dateien abgelegt werden (Zwischenergebnisse der Umwandlung).
-   `ResultsFolder`: Absoluter oder relativer Pfad zum Ordner in dem die Ergebnis Word-Dateien abgelegt werden.
-   `AllowedFileNameRegex`: Regulärer Ausdruck für die erlaubten Dateinamen von verwendeten Word-Vorlagen. Aus Sicherheitsgründen wird die Einschränkung auf minimal notwendige Zeichen empfohlen. Verwenden Word-Vorlagen beispielsweise `_` könnte man folgenden Wert setzen: `"^[a-z0-9_]+$"`.
-   `LicenseFile`: Absoluter oder relativer Pfad zur QuickDocs-Lizenzdatei.
-   `ApiTokenSecret`: Das für die Validierung von API-Token zuständige Secret. Wird bei HTTP-Anfragen an bestimmte Feature verwendet. Sollte ein einzigartiges, starkes Passwort/Secret sein (mindestens 32 Zeichen). Das Secret muss identisch sein mit dem Secret, welches für die [📄 Generierung der API-Token verwendet wird](../integration/api-auth.md). Wird der Wert nicht gesetzt (`""` oder `null`) dann werden keine API-Token verwendet/benötigt (wird nicht empfohlen).

### Sektion: CleanupConfig

Steuert das Auto-Cleanup der Datenordner von QuickDocs.
Wenn nicht deaktiviert, wird bei jedem Start vom QuickDocs Prozess das Auto-Cleanup durchgeführt, welches alte Dateien löscht.
Wird das Auto-Cleanup nicht verwendet, sammeln sich Dateien in den Datenordnern, die selbstständig gelöscht werden müssen.

-   `Enabled`: "true" oder "false".
-   `IncludeTemp`: "true" oder "false". Soll der Inhalt des konfigurierten `TempFolder` (s. o.) bereinigt werden?
-   `IncludeResults`: "true" oder "false". Soll der Inhalt des konfigurierten `ResultsFolder` (s. o.) bereinigt werden?
-   `AgeLimitInMinutes`: Positive Ganzzahl. Gibt das maximale Alter eine Datei an, bevor sie vom Auto-Cleanup gelöscht wird. Angabe in Minuten. Ein Wert von "60" bedeutet, dass alle Dateien älter 1 Stunde gelöscht werden.

### Sektion: ConnectionStrings

Datenbank-Verbindungs-Zeichenfolgen von QuickDocs, hier können die Pfade für die Datenbankdateien angepasst werden.

### Sektion: Serilog

Dies ist die Log-Konfiguration von QuickDocs auf Basis von [🌐 Serilog](https://github.com/serilog/serilog).

#### Anpassung des Log-Level

Das `MinimumLevel` bestimmt, welche Informationen ins Log fließen.  
Mehr dazu in der [🌐 Serilog Dokumentation](https://github.com/serilog/serilog/wiki/Configuration-Basics#minimum-level)

Beispiel: Hier wird alles bis auf Fehler (und darüber) ignoriert.

```json
    "MinimumLevel": {
      "Default": "Error",
```

Beispiel: Hier wird alles (einschließlich Debug-Informationen) geloggt.

```json
    "MinimumLevel": {
      "Default": "Debug",
```

#### Anpassung der Log-Datei

Die Logdatei hat mehrere Einstellungen:

-   Pfad: Hier kann ein individueller Pfad angegegeben werden. Vor dem `.txt` wird automatisch das Datum eingefügt.
-   Rolling Interval: Gibt das Intervall an, in dem die Datei "gerollt" wird. Bei `Day` bedeutet das, jeden Tag wird eine neue Datei angefangen.
-   Retained File Count Limit: Bezogen auf das Rollen, kann hier definiert werden, wie viele Dateien beibehalten werden sollen.
-   Output Template: Hier wird der eigentliche Inhalt der Datei festgelegt.

Mehr dazu in der [🌐 Serilog Dokumentation](https://github.com/serilog/serilog-sinks-file)

```json
    "Name": "File",
    "Args": {
        "path": "C:\\Logs\\QuickDocs.txt",
        "rollingInterval": "Day",
        "retainedFileCountLimit": 30,
        "outputTemplate": "{Timestamp:yyyy-MM-dd HH:mm:ss.fff zzz} [{Level:u3}] {Message:lj}{NewLine}{Exception}"
    }
```

## Befehlszeilenargumente

Im Normalfall wird QuickDocs durch einen Webserver (IIS) gestartet, dabei wird `WS.QuickDocs.Api.dll` bzw. `WS.QuickDocs.Api.exe` gestartet, hier können auch Argumente konfiguriert werden.

Zusätzlich kann man QuickDocs auch direkt starten.

Folgende Argumente werden unterstützt:

### --stop-after-cleanup

QuickDocs wird gestartet und direkt nach dem Auto-Cleanup beendet.

Dies kann zum Beispiel in Verbindung mit einer Windows Aufgabe / einem Cronjob verwendet werden, um die Bereinigung der Datenordner durchzuführen, ohne auf den (automatischen) Start von QuickDocs zu warten.
