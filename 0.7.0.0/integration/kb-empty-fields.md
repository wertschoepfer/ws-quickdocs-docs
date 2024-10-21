# ❓ Wie lassen sich leere Felder bei der Verarbeitung entfernen?

[🔙 Zurück zur Übersicht](_toc.md)

**Frage:**
Die Dokumentvorlage hat viele Mergefelder definiert. Der Prozess stellt aber nicht für alle Felder Daten zur Verfügung. Schickt man QuickDocs nun ein Mapping mit nur den ausgefüllten Feldern, dann bleiben im Ergebnis die Mergefelder einfach stehen. Wie können diese entfernt werden?

**Antwort:**
Bei der Verarbeitung von Dokumenten kann man im Mapping noch sog. "Options" mitschicken. Hier gibt es eine Option namnes "RemoveEmptyMergeFields". Gibt man diese mit "true" an, dann werden alle Felder entfern, die nicht übermittelt wurden.

```json
{
    "Options": {
        /* Alle Felder, die nicht in den MergeFields unten stehen aber in der Dokumentvorlage existierten, werden entfernt und bleiben nicht bestehen. */
        "RemoveEmptyMergeFields": true
    }
}
```

Will man nur einzelne Felder entfernen und nicht alle, dann genügt es, diese einfach mit einem Leerzeichen zu füllen.  
Leerer Text ("") wird als nicht angegeben interpretiert.

```json
{
    "MergeFields": {
        "FeldOhneWert": " "
    }
}
```
