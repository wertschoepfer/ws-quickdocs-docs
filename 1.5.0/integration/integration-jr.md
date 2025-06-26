# Integration von QuickDocs in JobRouter

[🔙 Zurück zur Übersicht](_toc.md)

Eine der Hauptaufgaben von QuickDocs ist eine gute Integration in JobRouter Prozesse.  
Diese soll hier mit Beispielen und konkreten Anweisungen beschrieben werden.

Wir empfehlen für die Nutzung im Dialog, nicht direkt per AJAX im Browser die REST-API anzusprechen,  
sondern per `jr_execute_dialog_function` eine PHP Dialogfunktion für die Kommunikation zu QuickDocs zu verwenden. Dies hat 2 Vorteile:

-   Der Server, auf dem QuickDocs installiert ist, ist ggf. nicht vom Browser erreichbar.
-   Man hat eine bessere, zentralere Kontrolle über die Kommunikation mit QuickDocs und Zugriff auf alle Prozesstabellenfelder, Prozesskonfigurationen, etc.

Sofern der Server, auf dem QuickDocs läuft auch vom Browser aus erreichbar ist, kann QuickDocs auch direkt per JavaScript (`fetch/xhr/ajax`) im Browser aufgerufen werden.  
Manche Endpunkte/Routen der API können zudem auch direkt im Browser per URL (GET) aufgerufen werden.

## Einrichtung und Vorbereitung

In einen bestehenden Prozess soll QuickDocs eingebunden werden.

Wir bieten einen Referenzprozess für die genauen Implementierungsdetails samt eigener JavaScript- und PHP-Komponenten an.  
Alle Beispiele hier beziehen sich auf diesen Referenzprozess.

### Einrichtung Prozess

<!-- TODO: Überarbeitung: MD-Tabelle -->

Damit der Prozess bei der Kommunikation weiß, wie der Server lautet, wird eine Prozesskonfiguration angelegt mit Namen `quickDocsServerUrl`.  
Der Wert ist die URL des QuickDocs-Servers, zum Beispiel: `https://www.example.com/quickdocs`.

### Einrichtung PHP-Dialogfunktion

<!-- TODO: Überarbeitung: Auslagerung in data -->

Eine PHP-Dialog-Funktion wird angelegt mit Namen `quickDocs` und dem Inhalt der gleichnamigen PHP-Datei im Referenzprozess.  
Will man den Zwischenschritt der PHP nicht und arbeitet direkt im Dialog mit einer Verbindung zum QuickDocs, dann entfällt diese.

Die PHP-Funktion enthält oben eine empfohlenen Vorschlag für `execute` und unten eine Klasse, die in `execute` verwendet wird.  
Empfohlen wird, den unteren Bereich nicht zu ändern, da dieser bei Updates dann einfach ausgetauscht werden kann.  
Der obere Bereich kann und sollte nach den Anforderungen des Prozesses entsprechend angepasst werden,  
beispielsweise kann hier das vom Dialog übergegeben Mapping beim Verarbeiten von Dokumenten angepasst werden.

Hat man vor, mehrere PHP-Funktionen für quickDocs zu nutzen, dann kann man auch die Klasse auslagern.  
Wir empfehlen dafür einen separaten Hilfsprozess.

Ein Beispiel dazu ist im Referenzprozess die PHP-Dialogfunktion `quickDocs`.

### Einrichtung PHP-Regelausführungsfunktion

Natürlich ist neben einer PHP-Dialog-Funktion auch der Einsatz anderer Funktionen möglich.  
Der Vorgang ist der gleiche wie oben, nur dass die `execute`-Funktion entsprechend geändert werden muss.

Bei einer Regelausführungsfunktion werden keine Parameter vom Dialog erwartet, das Mapping wird somit im PHP erstellt.  
Ein Beispiel dazu ist im Referenzprozess die PHP-Regelausführungsfunktion `quickDocsRule`.

### Einrichtung JavaScript

Dem Prozess wird ein globales JavaScript hinzugefügt mit Namen `QuickDocs` und dem Inhalt der gleichnamigen JS-Datei im Referenzprozess.

Aus dem JavaScript `App` (bzw. einem Dialog-Skript) kann QuickDocs dann über `QuickDocs.*` verwendet werden.  
Das `QuickDocs`-Objekt bietet direkte Implementierungen für die Verwendung von QuickDocs an, wie es für die meisten Fälle empfohlen wird.  
An geeigneter Stelle gibt es Möglichkeiten einzelne Details zu überschreiben, um das Verhalten anzupassen.

Hat man vor, QuickDocs in mehreren Prozessen Prozessen gleich zu verwenden, dann kann man auch dieses Objekt auslagern.  
Wir empfehlen dafür einen separaten Hilfsprozess.

## Füllen einer Word-Vorlage im Dialog mit Anzeige in Integration rechts

Siehe [📄 Verwendung und Funktionen der REST-API > Dokumentenverarbeitung](api-functions.md#dokumentenverarbeitung).

Die Prozesse müssen Mapping-Informationen (JSON) bereitstellen, diese an die REST-API schicken und dann die Ergebnisdateien (binär oder base64) verarbeiten.

In einem bestehenden Dialog wird ein FILE-Dialogelement erstellt.  
Dieses Dialogelement wird dann für die rechte Inteagration (mit oder ohne JobViewer) verwendet.

Per JavaScript wird dann die QuickDocs-PHP-Dialogfunktion angestoßen und das Ergebnis in das Attachment/File-Dialogelement gespeichert.  
Dabei wird auch die Integration rechts geladen.

Beispiel:

```js
QuickDocs.process(
    'orderPdf', // Name des FILE-Elements im Dialog, welches das Dokument abspeichert
    'Auftrag.pdf', // Name der Ergebnis-Datei
    'Auftragsvorlage', // Name der Vorlage auf dem Server
    mapping // Mapping-Objekt
);
```

Zusätzlich dazu kann auch der `successCallback`, der `errorCallback` und die zu verwendende `dialogFunction` übergeben werden,  
falls diese Angaben vom Standard abweichen sollen.

Das Ergebnis ist eine direkte Vorschau der PDF innerhalb weniger Sekunden, ein Abschicken oder Neuladen des Dialogs entfällt.  
Während der Ausführung wird das standard JobRouter Lade-Popup angezeigt.

### Echtzeitverarbeitung

Dabei handelt es sich um eine Erweiterung mit dem Vorteil, dass beim Laden der Vorschau bzw. beim Aufruf von QuickDocs kein Lade-Popup angezeigt wird.  
Dies wird insbesondere dann empfohlen, wenn direkt auf Änderungen (on change) reagiert und QuickDocs angestoßen werden soll.

Für diesen Fall wird eine selbstentwickelte Alternative zu `jr_execute_dialog_function` verwendet.

Beispiel:

```js
QuickDocs.processWithoutLoadingScreen(
    'orderPdf', // Name des FILE-Elements im Dialog, welches das Dokument abspeichert
    'Auftrag.pdf', // Name der Ergebnis-Datei
    'Auftragsvorlage', // Name der Vorlage auf dem Server
    mapping // Mapping-Objekt
);
```

Auch hier muss das Mapping-Objekt entsprechend der [📄 Dokumentenverarbeitung der API](api-functions.md#dokumentenverarbeitung) sein.

Das Ergebnis ist gleich wie bei der normalen Verarbeitung.

Da kein Lade-Popup angezeigt wird, sollte der Prozess/Dialog in diesem Fall selbst sicher stellen,  
dass die parallelen, asynchronen Anfragen an QuickDocs zum gewünschten Ergebnis führen.

## Vorlagenverwaltung

Siehe [📄 Verwendung und Funktionen der REST-API > Vorlagenverwaltung](api-functions.md#vorlagenverwaltung).

Per REST-API können Vorlagen aus Prozessen aufgelistet, gelöscht, heruntergeladen, hochgeladen und inspiziert werden.
Zusätzlich zum direkten Verwenden von .docx-Dateien ist auch die Nutzung von Base64-Strings (für `jr_execute_dialog_function`) möglich.

Besondere Voraussetzungen gibt es keine, an beliebiger Stelle in Prozessen wird die die QuickDocs-PHP-Dialogfunktion angestoßen und das Ergebnis verarbeitet.

Beispiele:

```js
QuickDocs.downloadTemplateNames(
    (response) => console.log(response.result.names) // Success-Handler mit den Namen aller Vorlagen als String-Array
);

QuickDocs.downloadTemplate(
    'orderPdf', // Name des FILE-Elements im Dialog, welches das Dokument abspeichert
    'Auftrag.pdf', // Name der Ergebnis-Datei
    'Auftragsvorlage' // Name der Vorlage auf dem Server
);

QuickDocs.uploadTemplate(
    'orderPdf', // Name des FILE-Elements im Dialog, welches das Dokument enthält
    'Auftragsvorlage', // Name der (neuen) Vorlage auf dem Server
    false // False (Standard) = Überschreibt bestehende Vorlagen nicht | True = Überschreibt bestehende Vorlagen.
);

QuickDocs.deleteTemplate(
    'Auftragsvorlage' // Name der Vorlage auf dem Server
);

QuickDocs.inspectTemplate(
    'Auftragsvorlage', // Name der Vorlage auf dem Server
    (response) => console.log(response.result.data) // Success-Handler, mit dem Ergebnis der Inspektion als (JSON) Objekt
);
```

Zusätzlich kann auch der jeweilige `successCallback`, der `errorCallback` und die zu verwendende `dialogFunction` übergeben werden,  
falls diese Angaben vom Standard abweichen sollen.

<!-- TODO: Dokumenttransformation ggf. hinzufügen, aktuell nur SA -->
