# Funktionen der QuickDocs-API

[🔙 Zurück zur Übersicht](_toc.md)

## Die Funktionen sind in Funktiosngruppen (Feature) unterteilt, welche auch im Swagger ersichtlich sind:

-   [➡️ Statusinformationen](#statusinformationen)
-   [➡️ Administration](#administration)
-   [➡️ Dokumentenverarbeitung](#dokumentenverarbeitung)
-   [➡️ Dokumenttransformation](#dokumenttransformation)
-   [➡️ Vorlagenverwaltung](#vorlagenverwaltung)

## Statusinformationen

Endpunkte: `GET|POST /status/...`

QuickDocs enthält einige Endpunkte mit denen Statusinformationen abgerufen werden können.  
Die genauen Endpunkte und Dokumentation dazu sind im Swagger zu finden.

## Administration

Endpunkte: `GET|POST /admin/...`

QuickDocs enthält einige Endpunkte mit denen administrative Aufgaben ausgeführt werden können.  
Die genauen Endpunkte und Dokumentation dazu sind im Swagger zu finden.

## Dokumentenverarbeitung

Endpunkt: `POST /process/mapping/<output-type>`

Kernlogik von QuickDocs um **Word-Vorlagen mit Daten aus einem JSON-Mapping zu füllen**.

Das Mapping muss als JSON (`application/json`) im Body mitgeschickt werden.
Die gewünschte Vorlage wird per Parameter `templateName` in der URL bestimmt, die Vorlage muss auf dem Server existieren.

Bei der Verarbeitung kann abhängig vom gewählten Endpunkt auch das Ausgabeformat gewählt werden, folgende stehen zur Auswahl:

-   Word (binär, Download)
-   Word-Base64 (text)
-   PDF (binär, Download)
-   PDF-Base64 (text)
-   Bild (PNG oder JPEG), mit Angabe der Seitenzahl
-   Syncfusion (DocumentEditor SFDT JSON), optional optimiert

QuickDocs verwendet die aus Microsoft Word bekannten `MAILMERGE / Seriendruck` **Serienfeldern (MergeFields)** zur Füllung von Daten.  
Zusätzlich gibt es sog. **Marker (Marker)**, welche HTML statt Text erlauben und so Formatierung mitschicken lassen.  
Ob Marker oder Serienfelder eingesetzt werden ist abhängig von der Vorlage und der Einsatz ist kombinierbar.  
Beide Feldarten unterstützen auch die Mehrfachausfüllung für Gruppen, Tabellen, Aufzählen etc. in **Marker-Gruppen (MarkerGroups)** und **Serienfeld-Gruppen (MergeGroups)**.

Neben dem Einsetzen von Werten kann man mit QuickDocs auch **Textmarken (Bookmarks)** im Dokument ein- und ausblenden.

Über **Optionen (Options)** kann QuickDocs abschließend weiter konfiguriert werden bzw. es können optionale Feature ein-/ausgeschaltet werden.

Die Reihenfolge der Verarbeitung (für komplexere Kombinationen ggf. zu berücksichtigen):  
**Marker-Gruppen** -> **Serienfeld-Gruppen** -> **Marker** -> **Serienfelder** -> **Bookmarks**s

Alle Angaben im Mapping sind optional, es muss nur geschickt werden, was benötigt wird.  
Die Reihenfolge der Angaben im Mapping ist nicht festgelegt und hat i. d. R. keine Auswirkung auf die Funktion.

Alle Angaben im Mapping sind "Case-Insensitive", d.h. auf Groß-/Kleinschreibung muss nicht geachtet werden.  
Wichtig: Dies gilt nicht für Namen von Feldern/Markern, die im Dokument definiert wurden.

Wir empfehlen alle Felder (Marker und Serienfelder) eindeutig zu benennen, sodass Verwechslungen bei der Verarbeitung außerhalb und innerhalb von Gruppen vermieden wird. Beispielsweise könnte es sonst einen Konflikt zwischen einem Einzel-Serienfeld `«name»` und einem gleichnamigen Gruppen-Serienfeld `«name»` geben.

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
        /* Falls im Dokument spezielle Felder wie `IF` oder `PageNum` verwendet werden kann hiermit die Aktualisierung dieser Felder angestoßen werden.
         * Dadurch funktionieren auch `IF`s in Kombination mit Mergefeldern.
         */
        "UpdateSpecialFields": true,
        /* Falls im Dokument automatische Inhaltsverzeichnisse verwendet werden können diese hiermit am Ende aktualisiert werden. */
        "UpdateTableOfContents": true,
        /* Hier kann eine beliebige Word-Vorlage als Base64 übergeben werden.
         * Diese wird dann statt der im "templateName"-Argument angegebenen Vorlage für die Verabeitung verwendet.
         * ❗ Verbraucht mehr Resourcen und sollte nur für spezielle Fälle eingesetzt werden. Vorlagen auf dem Server sind immer zu bevorzugen. Die mitgeschickte Vorlage wird auf dem Server für die Verarbeitung als Datei temporär abgelegt.
         * Wichtig: Obwohl nicht verwendet, muss die angegebene Vorlage im "templateName"-Argument ebenfalls gültig sein.
         */
        "ReplacementTemplate": "<base64>",
        /* Falls das Dokument bereits mit JobPDF von JobRouter verwendet wurde enthält es ggf. Felder mit Präfixen die normalerweise nicht unterstützt werden (zum Beispiel für Gruppen oder Bilder).
           Mit dieser Option wandelt QuickDocs automatsich zuerst alle JobRouter Präfixe in unterstützte um, bevor diese verarbeitet werden.
           Beispiele:
           - Für Tabellen/Gruppen (MergeGroups): «RangeStart:articles» + «RangeEnd:articles» -> «TableStart:articles» + «TableEnd:articles»
           - Für Bilder: (MergeFields) «Img:Logo» -> «Imgage:Logo»
         * Dadurch funktionieren auch diese in den Vorlagen ganz normal und die Vorlage kann ohne Anpassung aus JobPDF übernommen werden.
         * Wir empfehlen bei Komplikationen eine Anpassung der Vorlagen auf die Standardpräfixe von QuickDocs.
         */
        "SupportJobRouterPrefixes": true,
    },
    /* Hier werden die Einzel-Marker gelistet. Der Name muss dem Marker in der Vorlage entsprechen.
     * In der Vorlage ist ein Marker einfacher Text, i. d. R. in {{}} gepackt.
     * Der Wert muss valides XHTML sein. Format kann per style-Attribut gesetzt werden.
     * Obwohl hier XHTML übergeben wird muss beachtet werden, dass dieses in Word-Format umgewandelt wird,
     *   der Inhalt wird daher nicht immer 1:1 umgesetzt wie erwartet.
     */
    "Marker": {
        "{{ErsterMarker}}": "<h1>Diese Überschrift wird für den Marker <code>ErsterMarker</code> eingesetzt.</h1>",
        "{{ZweiterMarker}}": "<p>Dieser Paragraph wird für den Marker <code>ZweiterMarker</code> eingesetzt.</p>"
    },
    /* Hier werden alle Marker-Gruppen gelistet.
     * Gruppen werden entweder mit «TableStart:TableName» -> «TableEnd:TableName» gebaut oder mit «BeginGroup:GroupName» -> «EndGroup:GroupName».
     * Für eine Unterstützung von JR-Standard-Gruppen in Form von «RangeStart:TableName» -> Siehe oben die Option "SupportJobRouterPrefixes".
     */
    "MarkerGroups": [
        {
            /* Name der Gruppe, muss dem Namen in der Vorlage (ohne Prefix) entsprechen, hier «TableStart:tabelle» . */
            "Name": "tabelle",
            /* Hier werden alle Marker der Gruppe in "Sets" gelistet.
             * Jedes Set ist danach eine Zeile bzw. ein Eintrag in der Gruppe.
             * Marker innerhalb von Gruppen müssen für optimale Ergebnisse den Namen der Gruppe mit einem : voranstellen `Gruppe:`.
             */
            "MarkerSets": [
                /* Erstes Set / Erste Reihe */
                {
                    "{{tabelle:ErsterMarker}}": "Dieser Text wird für den Marker <code>tabelle:ErsterMarker</code> in Zeile 1 eingesetzt.",
                    "{{tabelle:ZweiterMarker}}": "Dieser Text wird für den Marker <code>tabelle:ZweiterMarker</code> in Zeile 1 eingesetzt.",
                },
                /* Zweites Set / Zweite Reihe */
                {
                    "{{tabelle:ErsterMarker}}": "Dieser Text wird für den Marker <code>tabelle:ErsterMarker</code> in Zeile 2 eingesetzt.",
                    "{{tabelle:ZweiterMarker}}": "Dieser Text wird für den Marker <code>tabelle:ZweiterMarker</code> in Zeile 2 eingesetzt.",
                }
            ]
        }
    ],
    /* Hier werden alle Einzel-Serienfelder gelistet. Der Name muss dem MERGEFIELD-Namen in der Vorlage entsprechen.
     */
    "MergeFields": {
        "erstesFeld": "Dieser Text wird für das Mergefeld erstesFeld eingesetzt.",
        "zweitesFeld": "Dieser Text wird für das Mergefeld zweitesFeld eingesetzt.",
        /* Bilder können als Base64 übermittelt werden, die Felder in der Vorlage müssen folgend aussehen: «Image:Name».
         * Alternativ sind die XHTML-Marker zu bevorzugen.
        */
        "bildFeld": "<base64>"
    },
    /* Hier werden alle Serienfeld-Gruppen gelistet.
     * Gruppen werden entweder mit «TableStart:TableName» -> «TableEnd:TableName» gebaut oder mit «BeginGroup:GroupName» -> «EndGroup:GroupName».
     * Für eine Unterstützung von JR-Standard-Gruppen in Form von «RangeStart:TableName» -> Siehe oben die Option "SupportJobRouterPrefixes".
     */
    "MergeGroups": [
        {
            /* Name der Gruppe, muss dem Namen in der Vorlage (ohne Prefix) entsprechen, hier «TableStart:tabelle» . */
            "Name": "tabelle",
            /* Hier werden alle Serienfelder der Gruppe in "Sets" gelistet.
             * Jedes Set ist danach eine Zeile bzw. ein Eintrag in der Gruppe.
             */
            "MergeFieldSets": [
                /* Erstes Set / Erste Reihe */
                {
                    "erstesMergeFieldDerGruppe": "Dieser Text wird für das Serienfeld erstesMergeFieldDerGruppe in Zeile 1 eingesetzt.",
                    "zweitesMergeFieldDerGruppe": "Dieser Text wird für das Serienfeld zweitesMergeFieldDerGruppe in Zeile 1 eingesetzt."
                },
                /* Zweites Set / Zweite Reihe */
                {
                    "erstesMergeFieldDerGruppe": "Dieser Text wird für das Serienfeld erstesMergeFieldDerGruppe in Zeile 2 eingesetzt.",
                    "zweitesMergeFieldDerGruppe": "Dieser Text wird für das Serienfeld zweitesMergeFieldDerGruppe in Zeile 2 eingesetzt."
                }
            ]
        }
    ],
    /* Hier werden alle Textmarken (Bookmarks) gelistet.
     * Der Name muss der Textmarke in der Vorlage entsprechen (ignoriert Groß-/Kleinschreibung).
     * Es müssen nur Textmarken gelistet werden, die auch verarbeitet werden sollen (restliche werden ignoriert).
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

### Alternative ohne Mapping

Endpunkt: `POST /process/content/<output-type>`

Anstelle eines Mappings wird direkt XHTML-Content (`text/plain`) im Body erwartet. Der Inhalt wird dann direkt in die Wordvorlage eingefügt.

**Veralteter Endpunkt:** Gleiches Verhalten hat man bei einem Mapping mit genau einem **Marker**, daher empfehlen wir diesen Endpunkt hier nicht weiter zu verwenden.

Bei der Verarbeitung kann abhängig vom gewählten Endpunkt auch das Ausgabeformat gewählt werden, folgende stehen zur Auswahl:

-   Word (binär, Download)
-   Word-Base64 (text)
-   PDF (binär, Download)
-   PDF-Base64 (text)

## Dokumenttransformation

Neben der reinen Dokumentenverarbeitung bietet QuickDocs auch Funktionen zur Transformation von Dokumenten an.

### Seitentrennung in PDF-Dateien

Endpunkt: `POST /transform/split-pages/pdf`

Trennt eine hochgeladene Word- oder PDF-Datei in einzelne Seiten auf.  
Die Seiten werden als einzelne PDF-Dateien in einer ZIP-Datei durchnummeriert als Dateidownload zurückgegeben.

### Seitentrennung in Bilddateien

Endpunkt: `POST /transform/split-pages/image`

Trennt eine hochgeladene Word- oder PDF-Datei in einzelne Seiten auf.  
Die Seiten werden als einzelne Bilddateien (PNG oder JPEG) in einer ZIP-Datei durchnummeriert als Dateidownload zurückgegeben.

### Dokumentverbindung

Endpunkt: `POST /transform/merge-documents`

Verbindet alle hochgeladenen Word- oder PDF-Dokumente.  
Das Ergebnis wird als Word- oder PDF-Datei als Dateidownload zurückgegeben.

### Seitenextraktion

Endpunkt: `POST /transform/extract-pages`

Trennt eine hochgeladene Word- oder PDF-Datei in einzelne Seiten auf.  
Bestimmte Seiten werden entsprechend einer Konfiguration wieder in ein Dokument zusammengefasst.  
Das Ergebnis wird als PDF-Datei als Dateidownload zurückgegeben.

## Vorlagenverwaltung

Endpunkte: `GET /template/...`

Die Vorlagenverwaltung ist eine Gruppe von API-Endpunkten zum Auflisten, Löschen, Herunterladen, Hochladen und für die Inspektion von Word-Vorlagen.  
Zusätzlich zum direkten Verwenden von .docx-Dateien ist auch die Nutzung von Base64-Strings möglich.  
Alternativ können Vorlagen administrativ direkt auf dem QuickDocs Server im Vorlagenordner verwaltet werden.

Neben der normalen Verwaltung sind auch noch einige Export-Formate wie Bild und Syncfusion-DocumentEditor-Text vorhanden.

Die genauen Endpunkte und Dokumentation dazu sind im Swagger zu finden.

### Vorlage inspizieren

Endpunkt: `GET /template/inspect`

Gibt folgende Informationen zu einer Vorlage als Beispiel (mit Unterstützung von JobRouter Prefixen) zurück:

```json
{
    /* Gefundene Einzel-Marker.
     * Aktuell werden nur Marker in {{}} gefunden.
     * Gibt alle Marker an, die nicht in einer Gruppe vorkommen, bestimmt anhand dem Gruppenzusatz `Gruppe:`.
     */
    "marker": ["AdditionalInformation"],
    /* Gefundene Marker-Gruppen und deren Felder.
     */
    "markerGroups": {
        "articleList": ["name", "category", "quantity"]
    },
    /* Gefundene Serienfeld-Gruppen und deren Serienfelder.
     */
    "mergeGroups": {
        "articles": ["name", "category", "quantity"],
        "JR_IncidentHistory": ["Step", "Description", "Username", "InDate", "OutDate"]
    },
    /* Gefundene Einzel-Serienfelder.
     * Gibt alle Serienfelder an, die nicht in einer Gruppe vorkommen, bestimmt anhand dem gleichlautenden Namen.
     * Wird ein Einzel-Serienfeld mit gleichem Namen wie dem eines Gruppen-Serienfeld verwendet kann es zu Konflikten kommen.
     */
    "mergeFields": [
        "applicantName",
        "articleCostSum",
        "Bild",
        "highPriorityFlag",
        "Incident",
        "InDate",
        "supervisorName"
    ],
    /* Die Namen aller gefundenen Textmarken (Bookmarks) inkl. versteckte mit _ beginnend. */
    "bookmarks": ["B1", "B2"],
    /* Zusätzliche Informationen
     * Diese Angaben können unterschiedlich sein und sich später noch stark ändern.
     * Sie sind daher auch nicht für den allgemeinen Gebrauch gedacht und eher für Debugging/Analyse hilfreich.
     */
    "additionalDetails": {
        "supportJobRouterPrefixes": true,
        "markerStyle": "{{.*?}}",
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
        },
        /* Failsafe Angaben zu gefundenen/gesuchten MERGEFIELDs im Dokument für unterstütztes Debugging. */
        "propertyBasedMergeFieldCount": 29,
        "textSearchBasedMergeFieldCount": 29
    }
}
```
