# Verwendung und Funktionen der REST-API

[🔙 Zurück zur Übersicht](_toc.md)

## API Definition

QuickDocs bietet eine Swagger-/Open-API-Dokumentation direkt auf dem Server unter `/swagger` an: `https://<server>/quickdocs/swagger.`

Hier sind alle vorhandenen HTTP-Endpunkte gelistet und beschrieben und können direkt getestet werden.

### Development-Modus Endpunkte

Im Developer-Modus (siehe [📄 Umgebungsvariablen im Administrationshandbuch](./../admin/settings.md#umgebungsvariablen)) sind weitere Endpunkte vorhanden.

Diese Endpunkte sind mit einem `!` gekennzeichnet, zum Beispiel `/status/!/config`.

Generell sind sie nur für temporäre Debug-Szenarien vorgesehen und sollten nicht dauerhaft produktiv eingesetzt werden.

Sie sind enthalten, um in Debug-Szenarien mehr Informationen über das System, die Runtime, etc. zu bekommen oder, um zur Erleichterung der Entwicklung Aktionen

## Verarbeitung von Word-Vorlagen mit Mapping

Endpunkt: `POST /process/mapping/<output-type>?templateName=`

Das Template wird im Argument `templateName` angegeben, es wird der Dateiname ohne Endung erwartet. Beispiel: `WS`.

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
        /* Alle Felder, die nicht in den MergeFields unten stehen aber im Template existierten, werden entfernt und bleiben nicht bestehen. */
        "RemoveEmptyMergeFields": true,
        /* Bei der Ersetzung der Inahlte der Marker wird das Format aus dem Template genommen. D.h. das jegliche `style`-Angaben im HTML ignoriert werden. */
        "KeepWordFormat": true,
        /* Falls im Dokument spezielle Felder wie IF oder PageNum verwendet werden kann hiermit die Aktualisierung dieser Felder angestoßen werden. Dadurch funktionieren auch IFs in Kombination mit Mergefeldern. */
        "UpdateSpecialFields": true
    },
    /* Hier werden die Text / XHTML-Marker gelistet. Der Name muss dem Marker im Template entsprechen.
     * Empfehlung: Marker in {{}} packen.
     * Der Text muss valides XHTML sein. Format kann per style-Attribut gesetzt werden.
     * Obwohl hier XHTML übergeben wird muss beachtet werden, dass dieses in Word-Format umgewandelt wird,
     *   der Inhalt wird daher nicht immer 1:1 umgesetzt wie erwartet.
     */
    "Marker": {
        "{{ErsterMarker}}": "<h1>Diese Überschrift wird für den Marker <code>{{ErsterMarker}}</code> eingesetzt.</h1>",
        "{{ZweiterMarker}}": "<p>Dieser Paragraph wird für den Marker <code>{{ZweiterMarker}}</code> eingesetzt.</p>"
    },
    /* Hier werden alle Mergefelder gelistet. Der Name muss dem MERGEFIELD-Namen im Template entsprechen.
     * Aktuell werden JR-Standard-Bilder in Form von «img:Name» nicht unterstützt, XHTML-Marker sind zu bevorzugen.
     */
    "MergeFields": {
        "erstesFeld": "Dieser Text wird für das Mergefeld erstesFeld eingesetzt.",
        "zweitesFeld": "Dieser Text wird für das Mergefeld zweitesFeld eingesetzt."
    },
    /* Hier werden alle Gruppen gelistet.
     * Gruppen werden entweder mit «TableStart:TableName» -> «TableEnd:TableName» gebaut oder mit «BeginGroup:GroupName» -> «EndGroup:GroupName».
     * Aktuell werden JR-Standard-Gruppen in Form von «RangeStart:TableName» -> RangeEnd:TableName» nicht unterstützt.
     */
    "MergeGroups": [
        {
            /* Name der Gruppe, muss dem Namen im Template (ohne Prefix) entsprechen, hier «TableStart:tabelle» . */
            "Name": "tabelle",
            /* Hier werden alle Mergefelder der Gruppe in "Sets" gelistet. Jedes Set ist danach eine Zeile bzw. ein Eintrag in der Gruppe. */
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
    /* Hier werden alle Textmarken (Bookmarks) gelistet. Der Name muss der Textmarke im Template entsprechen (ignoriert Groß-/Kleinschreibung).
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

## Templateverwaltung

Die Templates müssen auf dem Server, auf dem QuickDocs läuft, abgelegt sein und werden bei den jeweiligen Aktionen per Dateiname ausgewählt.
Aktuell ist keine Verwaltung ohne Zugriff zum Server möglich.

### Inspektion

Endpunkt: `GET /template/inspect/?templateName=`

Das Template wird im Argument `templateName` angegeben, es wird der Dateiname ohne Endung erwartet. Beispiel: `WS`.

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

<!-- TODO: Dokumentation Templateverwaltung -->
