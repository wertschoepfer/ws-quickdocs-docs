# ❓ Wie kann ich Abstand zwischen zwei Sätzen in einem Marker setzen?

[🔙 Zurück zur Übersicht](_toc.md)

**Frage:**
Die Dokumentvorlage hat einen Marker, der im Mapping ausgefüllt wird. Dieser soll einen Wert einfügen, mit Abstand zum übrigen Text. `<br>` scheint der Marker jedoch zu ignorieren.

```html
<div>Wert<br /></div>
```

**Antwort:**
Will man lediglich Abstand zu bestehendem Text schaffen, dann kann man sich mit `margin` aushelfen.

```html
<div style="margin-bottom: 20px">Wert</div>
```

`<br />` ist grundsätzlich möglich, jedoch nur in der Mitte vom Inhalt, nicht am Ende.
