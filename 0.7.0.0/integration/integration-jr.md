# Integration von QuickDocs in JobRouter

[🔙 Zurück zur Übersicht](_toc.md)

Eine der Hauptaufgaben von QuickDocs ist eine gute Integration in JobRouter Prozesse.

Wir empfehlen, nicht direkt per AJAX im Browser die REST-API anzusprechen, sondern per `jr_execute_dialog_function` eine PHP Dialogfunktion für die Kommunikation zu QuickDocs zu verwenden. Dies hat 2 Vorteile:

-   Der Server, auf dem QuickDocs installiert ist, ist ggf. nicht vom Browser erreichbar.
-   Man hat oft eine bessere, zentralere Kontrolle über die Kommunikation mit QuickDocs und Zugriff auf alle Prozesstabellenfelder, etc.

Sofern der Server, auf dem QuickDocs läuft auch vom Browser aus erreichbar ist, kann QuickDocs auch direkt per JavaScript (fetch/xhr/ajax) im Browser aufgerufen werden.

Manche Endpunkte/Routen der API können auch direkt im Browser per URL (GET) aufgerufen werden, siehe dazu [📄 Verwendung der REST-API](api.md).

<!-- TODO: Dokumentation
Die Implementierung kann im Referenzprozess im Dialog //TODO// eingesehen werden.
 -->

## Füllen einer Wordvorlage im Dialog mit Anzeige in Integration rechts

Die Prozesse müssen Mapping-Informationen (JSON) bereitstellen, diese an die REST-API schicken und dann die Ergebnisdateien (binär oder base64) verarbeiten.

Normalerweise wird ein Dialog erzeugt und mit einer Integration (mit oder ohne JobViewer) auf ein Attachment/File-Dialogelement.

Per JavaScript wird dann die QuickDocs-PHP-Dialofunktion angestoßen und das Ergebnis in das Attachment/File-Dialogelement gespeichert. Dabei wird auch die Integration rechts geladen.

Dadurch ist eine direkte Vorschau des Ergebnisses innerhalb weniger Sekunden möglich, ein Abschicken oder Neuladen des Dialogs entfällt.

<!-- TODO: Dokumentation
Die Implementierung kann im Referenzprozess im Dialog //TODO// eingesehen werden.
 -->

### Echtzeitverarbeitung

Dies ist eine Erweiterung mit dem Vorteil, dass beim Laden der Vorschau bzw. beim Aufruf von QuickDocs kein Lade-Popup angezeigt wird.

Dies wird insbesondere dann empfohlen, wenn direkt auf Änderungen (on change) reagiert und QuickDocs angestoßen werden soll.

Für diesen Fall wird eine Alternative zu `jr_execute_dialog_function` verwendet.

<!-- TODO: Dokumentation
Die Implementierung kann im Referenzprozess im Dialog //TODO// eingesehen werden.
 -->
