# Verwendung und Funktionen der REST-API

[🔙 Zurück zur Übersicht](_toc.md)

## API Definition

QuickDocs bietet eine Swagger-/Open-API-Dokumentation direkt auf dem Server unter `/swagger` an: `https://<server>/quickdocs/swagger.`  
Hier sind alle vorhandenen HTTP-Endpunkte gelistet und beschrieben und können direkt getestet werden.

**Beispiel-Aufruf an QuickDocs mit cURL:**

```sh
curl -X 'POST' \
  'http://quickdocs.server/process/mapping/word?templateName=WS' \
  -H 'accept: */*' \
  -H 'Content-Type: application/json' \
  -d '{}'
```

## Development-Modus

Im Development-Modus (siehe [📄 Aktivierung im Handbuch für die Administration](./../admin/settings.md#umgebungsvariablen)) sind weitere Endpunkte vorhanden.
Diese Endpunkte sind mit einem `!` gekennzeichnet, zum Beispiel `/status/!/config`.  
Generell sind diese Endpunkte nur für temporäre Debug-Szenarien vorgesehen und sollten nicht dauerhaft produktiv eingesetzt werden.

Zusätzlich gibt QuickDocs im Development-Modus auch Fehlermeldungen an HTTP-Anfragen aus, beispielsweise, wenn ein erforderliches Mapping nicht richtig ist:

```cs
Microsoft.AspNetCore.Http.BadHttpRequestException: Failed to read parameter "Mapping mapping" from the request body as JSON.
```

## API-Token

QuickDocs unterstützt sog. JSON Web Tokens als API-Token.  
Ein Token ist ein zusätzlicher Text, der bei jedem HTTP-Aufruf an die API mitgeschickt wird.  
Sind Tokens definiert und im Einsatz, dann sind bestimmte Endpunkte ohne Token nicht aufrufbar.  
So kann zum Beispiel eine Anwendung die Dokumentenverarbeitung nutzen,  
ohne Zugriff auf die Vorlagenverwaltung haben zu müssen.

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
Die Claims stellen dabei Statusinformationen wie Ausstellungsdatum, Ablaufdatum, Aussteller, etc. dar.
Zusätzlich gibt es Claims mit Angaben, die QuickDocs benötigt, diese sind:

-   `feature`:
    -   Beschreibung: Feature-Name, das mit diesem Token aufgerufen werden kann.
    -   Mögliche Werte:
        -   `template` für die Vorlagenverwaltung
        -   `process` für die Dokumentenverarbeitung
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

API-Token können selbstständig jederzeit mit Generatoren erstellt werden, einer davon ist https://www.jwt.io.  
Hier müssen zuerst die Claims rechts definiert werden, danach muss das Secret unten eingetragen werden,  
der Token wird dann links zum Rauskopieren angezeigt.  
[🌐 Hier kann das Beispiel von oben direkt online auf https://www.jwt.io verwendet werden](https://jwt.io/#debugger-io?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJodHRwczovL3d3dy53ZXJ0c2Nob2VwZmVyLml0Iiwic3ViIjoiV2VydHNjaMO2cGZlciBJVCBHbWJIIiwiYXVkIjoiV1MgUXVpY2tEb2NzIiwiaWF0IjoxNzI5MjY2Nzg5LCJleHAiOjE3MjkyNzAzODksInZlcnNpb24iOiJBbHBoYSIsImZlYXR1cmUiOlsidGVtcGxhdGUiLCJwcm9jZXNzIl0sIm5iZiI6MTcyOTI2Njc4OX0.xTNkHm1G7VKOJU_IUzgEk2QrofWpSRzXKYaj3QAEofk).

Im Development-Modus oder wenn das Secret nicht in QuickDocs definiert wurde sind keine Token für die Aufrufe an die API nötig.

Zum Testen des Tokens kann der Endpunkt `GET /status/token` aufgerufen werden.  
Wurde ein gültiges Token erkannt, dann werden zu diesem die Claims ausgegeben.

## Statusinformationen

Endpunkte: `GET|POST /status/...`

QuickDocs enthält einige Endpunkte mit denen Statusinformationen abgerufen werden können.  
Die genauen Endpunkte und Dokumentation dazu sind im Swagger zu finden.

## Dokumentenverarbeitung

Endpunkt: `POST /process/mapping/<output-type>`

Das Mapping muss als JSON (`application/json`) im Body mitgeschickt werden. Unten steht ein einfaches Beispiel mit Informationen zu den Eigenschaften.

Alle Angaben im Mapping sind optional, es muss nur geschickt werden, was benötigt wird.  
Alle Angaben im Mapping sind "Case-Insensitive", d.h. auf Groß-/Kleinschreibung muss nicht geachtet werden.

Bei der Verarbeitung kann abhängig vom gewählten Endpunkt auch das Ausgabeformat gewählt werden, folgende stehen zur Auswahl:

-   Word (binär, Download)
-   Word-Base64 (text)
-   PDF (binär, Download)
-   PDF-Base64 (text)

```json
{
    /* Globale Optionen für die Verarbeitung. Die Optionen sind alle optional. */
    "Options": {
        /* Alle Felder, die nicht in den MergeFields unten stehen aber in der Vorlage existierten,
         * werden entfernt und bleiben nicht bestehen.
         */
        "RemoveEmptyMergeFields": true,
        /* Bei der Ersetzung der Inahlte der Marker wird das Format aus der Vorlage genommen.
         * D.h. das jegliche `style`-Angaben im HTML ignoriert werden.
         */
        "KeepWordFormat": true,
        /* Falls im Dokument spezielle Felder wie IF oder PageNum verwendet werden kann hiermit die Aktualisierung dieser Felder angestoßen werden.
         * Dadurch funktionieren auch IFs in Kombination mit Mergefeldern.
         */
        "UpdateSpecialFields": true,
        /* Falls im Dokument automatische Inhaltsverzeichnisse verwendet werden können diese hiermit am Ende aktualisiert werden. */
        "UpdateTableOfContents": true,
        /* Hier kann eine beliebige Word-Vorlage als Base64 übergeben werden.
         * Diese wird dann statt der im "templateName"-Argument angegebenen Vorlage für die Verabeitung verwendet.
         * ❗ Verbraucht mehr Resourcen und sollte nur für spezielle Fälle eingesetzt werden. Vorlagen auf dem Server sind immer zu bevorzugen. Die mitgeschickte Vorlage wird auf dem Server für die Verarbeitung als Datei abgelegt.
         * Wichtig: Obwohl nicht verwendet, muss die angegebene Vorlage im "templateName"-Argument ebenfalls gültig sein.
         */
        "ReplacementTemplate": "<base64>"
    },
    /* Hier werden die Text / XHTML-Marker gelistet. Der Name muss dem Marker in der Vorlage entsprechen.
     * Empfehlung: Marker in {{}} packen.
     * Der Text muss valides XHTML sein. Format kann per style-Attribut gesetzt werden.
     * Obwohl hier XHTML übergeben wird muss beachtet werden, dass dieses in Word-Format umgewandelt wird,
     *   der Inhalt wird daher nicht immer 1:1 umgesetzt wie erwartet.
     */
    "Marker": {
        "{{ErsterMarker}}": "<h1>Diese Überschrift wird für den Marker <code>{{ErsterMarker}}</code> eingesetzt.</h1>",
        "{{ZweiterMarker}}": "<p>Dieser Paragraph wird für den Marker <code>{{ZweiterMarker}}</code> eingesetzt.</p>"
    },
    /* Hier werden alle Mergefelder gelistet. Der Name muss dem MERGEFIELD-Namen in der Vorlage entsprechen.
     * Bilder können als Base64 übermittelt werden, die Felder in der Vorlage müssen folgend aussehen: «Image:Name».
     * Alternativ sind die XHTML-Marker zu bevorzugen.
     */
    "MergeFields": {
        "erstesFeld": "Dieser Text wird für das Mergefeld erstesFeld eingesetzt.",
        "zweitesFeld": "Dieser Text wird für das Mergefeld zweitesFeld eingesetzt.",
        "bildFeld": "<base64>"
    },
    /* Hier werden alle Gruppen gelistet.
     * Gruppen werden entweder mit «TableStart:TableName» -> «TableEnd:TableName» gebaut oder mit «BeginGroup:GroupName» -> «EndGroup:GroupName».
     * Aktuell werden JR-Standard-Gruppen in Form von «RangeStart:TableName» -> RangeEnd:TableName» nicht unterstützt.
     */
    "MergeGroups": [
        {
            /* Name der Gruppe, muss dem Namen in der Vorlage (ohne Prefix) entsprechen, hier «TableStart:tabelle» . */
            "Name": "tabelle",
            /* Hier werden alle Mergefelder der Gruppe in "Sets" gelistet.
             * Jedes Set ist danach eine Zeile bzw. ein Eintrag in der Gruppe.
             */
            "MergeFieldSets": [
                /* Erstes Set / Erste Reihe */
                {
                    "erstesMergeFieldDerGruppe": "Dieser Text wird für das Mergefeld erstesMergeFieldDerGruppe in Zeile 1 eingesetzt.",
                    "zweitesMergeFieldDerGruppe": "Dieser Text wird für das Mergefeld zweitesMergeFieldDerGruppe in Zeile 1 eingesetzt."
                },
                /* Zweites Set / Zweite Reihe */
                {
                    "erstesMergeFieldDerGruppe": "Dieser Text wird für das Mergefeld erstesMergeFieldDerGruppe in Zeile 2 eingesetzt.",
                    "zweitesMergeFieldDerGruppe": "Dieser Text wird für das Mergefeld zweitesMergeFieldDerGruppe in Zeile 2 eingesetzt."
                }
            ]
        }
    ],
    /* Hier werden alle Textmarken (Bookmarks) gelistet.
     * Der Name muss der Textmarke in der Vorlage entsprechen (ignoriert Groß-/Kleinschreibung).
     * Es müssen nur Textmarken gelistet werden, die auch verarbeitet werden sollen (Restliche werden ignoriert).
     */
    "Bookmarks": [
        "Textmarke_1": {
            /* Der Inhalt der Textmarke wird entfernt, die Textmarke selbst bleibt erhalten. */
            "RemoveContent": true
        },
        "textmarke_2": {
            "RemoveContent": false
        }
    ]
}
```

Reihenfolge der Verarbeitung (für komplexere Kombinationen):

-   **Marker**, nach Reihenfolge im Mapping
-   **MergeGroups**, nach Reihenfolge im Mapping
-   **MergeFields**, nach Reihenfolge im Mapping
-   **Bookmarks**, nach Reihenfolge im Mapping

### Alternative ohne Mapping

Endpunkt: `POST /process/content/<output-type>`

Anstelle eines Mappings wird direkt XHTML-Content (`text/plain`) im Body erwartet. Der Inhalt wird dann direkt in die Wordvorlage eingefügt.

Bei der Verarbeitung kann abhängig vom gewählten Endpunkt auch das Ausgabeformat gewählt werden, folgende stehen zur Auswahl:

-   Word (binär, Download)
-   Word-Base64 (text)
-   PDF (binär, Download)
-   PDF-Base64 (text)

## Vorlagenverwaltung

Endpunkte: `GET /template/...`

Die Vorlagenverwaltung ist eine Gruppe von API-Endpunkten zum Auflisten, Löschen, Herunterladen, Hochladen und für die Inspektion von Word-Vorlagen.  
Zusätzlich zum direkten Verwenden von .docx-Dateien ist auch die Nutzung von Base64-Strings möglich.  
Alternativ können Vorlagen administrativ direkt auf dem QuickDocs Server im Vorlagenordner verwaltet werden.

### Liste aller Namen vorhandener Vorlagen abrufen

**Endpunkt:** `GET /template/names`

Beispielausgabe:

```json
["WS", "WS-Brief"]
```

### Vorlage als Word-Datei herunterladen

**Endpunkt:** `GET /template/file/word`

### Vorlage als Base64-String herunterladen

**Endpunkt:** `GET /template/file/word-base64`

### Bestehende Vorlage auf dem Server löschen

**Endpunkt:** `DELETE /template/file`

### Vorlage als Word-Datei hochladen

**Endpunkt:** `POST /template/file/word`

### Vorlage als Base64-String hochladen

**Endpunkt:** `POST /template/file/word-base64`

### Vorlage inspizieren

Endpunkt: `GET /template/inspect`

Gibt folgende Informationen als Beispiel zurück:

```json
{
    /* Namen der gefundenen Marker
     * Aktuell nur mit doppelten gewschweiften Klammern.
     */
    "marker": ["{{AdditionalInformation}}"],
    /* Namen und Felder der gefundenen Gruppen
     * Erkennt auch Gruppen, die eigentlich keine sind, diese bekommen den Prefix "INVALID_" bzw. "INVALID_RANGE_".
     */
    "mergeGroups": {
        "articles": ["name", "category", "quantity"],
        "INVALID_RANGE_JR_IncidentHistory": ["Step", "Description", "Username", "InDate", "OutDate"]
    },
    /* Namen der gefundenen Felder
     * Immer alle Felder, egal, ob diese in einer Gruppe sind oder nicht.
     * Gruppenstart/-Ende werden ebenfalls ohne Prefix ausgegeben.
     * Felder können mehrmals vorkommen, sie werden aufgelistet in der Reihenfolge, wie sie im Dokument gefunden werden.
     */
    "mergeFields": [
        "Incident",
        "applicantName",
        "inputDate",
        "dueDate",
        "highPriorityFlag",
        "supervisorName",
        "ratingFlag",
        "ratingFlag",
        "articleCostSum",
        "articles",
        "name",
        "category",
        "quantity",
        "articles",
        "INVALID_RANGE_JR_IncidentHistory",
        "Step",
        "Description",
        "Username",
        "InDate",
        "OutDate",
        "INVALID_RANGE_JR_IncidentHistory"
    ],
    /* Zusätzliche Informationen
     * Diese Angaben können unterschiedlich sein und sich später noch stark ändern.
     */
    "additionalDetails": {
        /* Die Namen von nicht erkannten Gruppen, und somit nicht unterstützten Feldern. */
        "unsupportedFields": ["RangeStart:JR_IncidentHistory", "RangeEnd:JR_IncidentHistory"],
        /* Die Namen aller gefundenen Bookmarks (auch versteckte) */
        "bookmarks": [],
        /* Sosntige gefundenen "Special Fields" */
        "specialFields": [
            {
                "fieldType": "FieldIf",
                "entityType": "Field",
                "fieldCode": " IF \u0013«highPriorityFlag»\u0015 = \"1\" \"Ja\" \"Nein\" "
            },
            {
                "fieldType": "FieldIf",
                "entityType": "Field",
                "fieldCode": " IF \u0013\u0015\u0013«ratingFlag»\u0015 = \"1\" \"Angenommen\" \"Abgelehnt\" "
            },
            {
                "fieldType": "FieldIf",
                "entityType": "Field",
                "fieldCode": " IF \u0013«stockFlag»\u0015 = \"1\" \"✅\" \"❌\" "
            }
        ],
        /* Allgemeine Angaben zum Dokument */
        "document": {
            "version": 0,
            "pageCount": 1,
            "paragraphCount": 3,
            "linesCount": 10,
            "wordCount": 208,
            "charCount": 1315
        },
        /* Allgemeine Angaben zur Vorlagendatei */
        "file": {
            "name": "WS.docx",
            "lastWriteTime": "2024-08-21T14:06:13.9865685+02:00",
            "lastAccessTime": "2024-09-19T15:15:53.1018944+02:00",
            "size": 20068
        }
    }
}
```
