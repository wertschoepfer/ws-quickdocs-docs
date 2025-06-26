# Verwendung und Funktionen der REST-API

QuickDocs unterstützt sog. JSON Web Tokens als API-Token für eine sichere Authentifizierung.

[🔙 Zurück zur Übersicht](_toc.md)

## API-Token

Ein Token ist ein zusätzlicher Text, der bei jedem HTTP-Aufruf an die API mitgeschickt wird.
Sind Tokens definiert und im Einsatz, dann sind bestimmte Endpunkte ohne Token nicht aufrufbar.  
So kann zum Beispiel eine Anwendung die Dokumentenverarbeitung nutzen, ohne Zugriff auf die Vorlagenverwaltung haben zu müssen.

**Nötiger Header:**

-   Name: `Authorization`
-   Wert: `Bearer <api-token>`

Auf der Swagger-Seite kann man dafür ganz oben den Button "Authorize" verwenden.  
Hier kann man ein Token eintragen und dann "Log in" betätigen.  
Danach werden alle Anfragen über die Swagger-Seite das Token verwenden.

**Beispiel Aufruf mit cURL:**

```js
curl -X 'GET' \
  'http://<server>/quickdocs/status/token' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...'
```

### Claims

In Token sind Informationen enthalten, sog. **Claims**, sowie eine Signatur zur Validierung basierend auf einem **Secret**.  
Die Claims stellen dabei teils erforderliche teils optionale Informationen wie Ausstellungsdatum, Ablaufdatum, Aussteller, etc. dar.
Zusätzlich gibt es Claims mit Angaben, die QuickDocs benötigt, diese sind:

-   `feature`:
    -   Beschreibung: Feature-Name, das mit diesem Token aufgerufen werden kann.
    -   Mögliche Werte:
        -   `template` für die Vorlagenverwaltung
        -   `process` für die Dokumentenverarbeitung
        -   `transform` für die Dokumenttransformation
        -   `admin` für die Administration
    -   Hinweis: Der Claim kann mehrmals vorkommen bzw. mit mehreren Werten verwendet werden (Beispiel: `["template", "process"]`)

**Beispiel mit Dokumentenverarbeitung und Vorlagenverwaltung:**

```json
{
    "iss": "https://www.wertschoepfer.it",
    "sub": "Wertschöpfer IT GmbH",
    "aud": "WS QuickDocs",
    "iat": 1729266789,
    "exp": 1729270389,
    "version": "Alpha",
    "feature": ["template", "process"],
    "nbf": 1729266789
}
```

### API-Token-Secret

Ein Token wird mit einem **Secret** signiert. Diese Signatur wird in QuickDocs gegengeprüft/validiert.

Wir empfehlen bei der [📄 Einrichtung direkt API-Token zu aktivieren](../admin/iis.md#konfiguration). Ist dies der Fall, dann besteht bereits ein Secret.  
Das in QuickDocs hinterlegte Secret muss dann für die Erstellung der API-Token verwendet werden.

Wurde noch kein Secret definiert, kann ein beliebiges verwendet werden.  
Es muss jedoch mindestens 32 Zeichen haben und sollte möglichst stark/komplex und einzigartig sein.

### Generierung

QuickDocs bietet den Endpunkt `POST /admin/token` zur einfachen Generierung neuer API-Token an.

API-Token können auch jederzeit mit anderen Mitteln erstellt werden, eines davon ist https://www.jwt.io.  
Hier müssen zuerst die Claims (s. o.) rechts definiert werden, danach muss das Secret unten eingetragen werden,  
der Token wird dann links zum Rauskopieren angezeigt.  
[🌐 Hier kann das Beispiel von oben direkt online auf https://www.jwt.io verwendet werden](https://jwt.io/#debugger-io?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJodHRwczovL3d3dy53ZXJ0c2Nob2VwZmVyLml0Iiwic3ViIjoiV2VydHNjaMO2cGZlciBJVCBHbWJIIiwiYXVkIjoiV1MgUXVpY2tEb2NzIiwiaWF0IjoxNzI5MjY2Nzg5LCJleHAiOjE3MjkyNzAzODksInZlcnNpb24iOiJBbHBoYSIsImZlYXR1cmUiOlsidGVtcGxhdGUiLCJwcm9jZXNzIl0sIm5iZiI6MTcyOTI2Njc4OX0.xTNkHm1G7VKOJU_IUzgEk2QrofWpSRzXKYaj3QAEofk).

Im Development-Modus oder wenn das Secret nicht in QuickDocs bei der Installation definiert wurde sind keine Token für die Aufrufe an die API nötig.

Zum Testen des Tokens kann der Endpunkt `GET /status/token` aufgerufen werden.  
Wurde ein gültiges Token erkannt, dann werden zu diesem die Claims ausgegeben.
