# Allgemeine Informationen

<!-- TODO: Dokumentation klären. -->

[🔙 Zurück zur Übersicht](_toc.md)

## QuickDocs API

QuickDocs stellt eine REST-API zur schnellen und einfachen Verwaltung und Verarbeitung von Word-Vorlagen/Dokumenten bereit.

Designt ist QuickDocs als Erweiterung für JobRouter Prozesse, ist aber auch eigenständig nutzbar.

Die REST-API ist eine ASP\.NET Anwendung und lässt sich auf Windows Servern mit IIS betreiben. Alternativ kann ein Linux Docker Container verwendet werden, ein Image ist vorhanden.  
Die Integration in Prozesse ist über einfache HTTP-Anfragen, sowohl von JS (AJAX) als auch PHP (cURL) möglich.

## Dokumentenverarbeitung

Bei der Verarbeitung von den Dokumenten braucht es auf dem Server eine vorbereitete Word-Datei (.docx), welche als Vorlage verwendet wird.

Diese Vorlage kann dann per API verarbeitet und in verschiedenen Ausgabeformaten (PDF/Word/etc.) ausgegeben werden.  
Es ist auch möglich (für speziellere Use-Cases) die Vorlage direkt mitzuschicken.

Über das sog. **Mapping** gibt der API-Aufruf Informationen an QuickDocs, wie die Vorlage bearbeitet werden soll.

### Felder & Gruppen

QuickDocs erlaubt die Befüllung von sog. Mail-Merge-Word-`MERGEFIELD`-Feldern (**MergeFields**) und versteht auch Gruppen (**MergeGroups**) anhand Prefixen (für Tabellen zum Beispiel). Auch Bilder lassen sich dynamisch einsetzen.

### XHTML-Marker

Darüber hinaus - und der größte Vorteil von QuickDocs - kann man auch mit einfachen Text-Markern (**Marker**) Inhalt in die Vorlage füllen. Dieser Inhalt ist dann beliebiger XHTML-Inhalt, welcher in Word umgewandelt wird. Darüber können einfache Sätze, ganze Absätze, Tabellen, Überschriften, Bilder aber auch ganze Seiten oder der gesamte Dokumentinhalt vom Prozess designt/generiert und dann an QuickDocs für die Befüllung übergeben werden. Über diese XHTML-Marker ist sehr viel umsetzbar, was mit den "normalen" Feldern nicht möglich ist.

Textmarker sind einfache, selbst definierte Wörter in der Vorlage und somit ganz normaler Text. Wir empfehlen die Nutzung von Sonderzeichen wie zum Beispiel `{{MeinTextMarker}}`, um versehentliche Ersetzung von anderen Vorkommen im Dokument zu vermeiden.

Ein gutes Beispiel ist eine dynamische Aufzählung (Punkt 1, Punkt 2, Punkt 3). Klassischerweise hätte man eine fixe aufzählung mit zum Beispiel 3 Felder. Der Prozess füllt diese dann entweder mit Text oder mit `(Leer)`. Dabei stört jedoch, dass "leere" Punkte auch als Aufzählungsinhalt zählen und somit (je nach Formatierung) zum Beispiel das Aufzählungszeichen oder ein nicht gewünschter Abstand angezeigt wird.

Bei der Verwendung von Markern kann stattdessen die komplette Aufzählung als 1 XHTML-Paket übergeben werden und somit beliebig viele Aufzählungspunkte enthalten.

Auch dynamische Tabellen-Spalten sind damit gut umsetzbar und Bilder und andere Elemente sind ebenso einfügbar.

Das zu schickende XHTML kann dabei sehr komplex und mitunter verbos sein, das Format muss explizit (inline) mitgegeben werden.  
Für die Erstellung bieten wir jederzeit unsere Unterstützung an.

### Textmarken

Da **Felder und Gruppen** nur sehr simple Szenarien abdecken, die Verwendung der **XHTML-Marker** jedoch aufgrund der komplexen, sehr granularen Steuerung etwas umständlich sein kann, gibt es zusätzlich noch einen Mittelweg: Textmarken (**Bookmarks**).

Bei diesen hat der Ersteller der Word-Vorlage die Kontrolle, da die Textmarken direkt in Word gesetzt werden. Ein Bereich (Word, Satz, Absatz, oder auch mehr) wird dabei in Word markiert und mit einer benannten Textmarke versehen.

QuickDocs kann dann mitgegeben werden, wie mit den Textmarken umgegangen wird. Aktuell kann man den Inhalt der Texmtarken löschen.

Dadurch lässt sich sehr leicht ein Szenario abdecken, bei welchem verschiedene Passagen einer Vorlage bedingt (dynamisch) angezeigt werden sollen.
