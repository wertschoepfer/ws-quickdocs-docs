# ❓ Wie schicke ich das Dokument (die Vorlage) direkt an QuickDocs mit?

[🔙 Zurück zur Übersicht](_toc.md)

**Frage:**
In QuickDocs liegen i.d.R. die Vorlagen auf dem QuickDocs-Server ab.  
Für manche Prozesse möchte man aber die Vorlagen-Dokumente andersweitig (Archiv, Prozess etc.) pflegen.  
Diese Dokumente muss man dann für die Verarbeitung mitschicken - aber wie?

**Antwort:**
Bei der Verarbeitung von Dokumenten kann man im Mapping noch sog. "Options" mitschicken. Hier gibt es eine Option namnes "ReplacmeentTemplate". Hier kann man einfach den Dateiinhalt in Base64 kodiert eintragen und mitschicken.

Wichtig: Diese Vorgehensweise wird nicht auf Dauer empfohlen, da hierdurch der Request wesentlich größer ist (die komplette Vorlage wird jedes Mal mitgeschickt). Stattdessen sollten die Vorlagen auf dem Server liegen.

Wichtig: Obwohl eine Vorlage mitgeschickt wird, muss der `templateName` Parameter beachtet werden und muss eine valide Vorlage auf dem QuickDocs-Server sein. Die Vorlage wird nicht verwendet, muss jedoch zur Sicherheit mit angegeben werden.

```json
{
    "Options": {
        "ReplacementTemplate": "<Base64-Dateiinhalt-der-Vorlage>"
    }
}
```

Ein gängiger Ansatz im JobRouter ist die Verwendung von **Prozesskonfigurationen** für die Pflege der Vorlagen.  
Will man diese verwenden, kann das Dokument im PHP abrufen und dann mitschicken.

**Für JR Dialogfunktion**

```php
private function executeProcessMapping(QuickDocsClient $client, QuickDocsDialogBackend $backend) {
    $templateName = $backend->getTemplateName();
    $mappingJson = $backend->getMappingJson();

    // Laden des absoluten Dateipfads auf die Vorlage.
    // Prozesskonfiguration für die im Frontend definierte Vorlage wird gesucht.
    // Wichtig: Prozesskonfiguration muss Typ "Word" sein, der Wert von `getConfiguration` ist der Dateiname inkl. Dateiendung.
    $processConfigurationValue = $this->getConfiguration($templateName);
    $processConfigurationFileRelativeDataPath = "_configurations/templates/word/$templateName/$processConfigurationValue";
    $processConfigurationFileAbsolutePath = $this->getFullDataPath(
        $processConfigurationFileRelativeDataPath, $this->getProcessName(), $this->getVersion());

    // Danach wird der Inhalt der Vorlage (vom Dokument) geladen und in Base64 umgewandelt.
    $fileContent = file_get_contents($processConfigurationFileAbsolutePath);
    $fileContentBase64 = base64_encode($fileContent);

    // Wir verwenden den QuickDocs-Mapper um das vom Dialog übergeben Mapping noch zu ergänzen.
    $mapping = (new QuickDocsDialogMapper($this))
        ->addMappingJson($mappingJson)
         // Erweiterung des Mappings für das ReplacemenTemplate.
        ->addMapping([
            "options" => [
                "replacementTemplate" => $fileContentBase64
            ]
        ])
        ->getMapping();

    // Rest ist gleich wie immer.
    // Handbuch: https://github.com/wertschoepfer/ws-quickdocs-docs/blob/main/latest/integration/api-functions.md#dokumentenverarbeitung
    $mappingJson = json_encode($mapping, JSON_PRETTY_PRINT);
    $result = $client->process($templateName, $mappingJson, QuickDocsResultType::PDF_BASE_64);
    // ...
}
```

**Für JR Regelausführungsfunktion**

```php
private function executeProcessMapping(QuickDocsClient $client, ?array $actionConfig = null): void {
    $templateName = $this->getTableValue("usedTemplateName") ?? throw new Exception("Fehlender Wert von Prozesstabellenfeld 'usedTemplateName'.");
    $mappingJson = $this->getTableValue("usedMappingJson") ?? throw new Exception("Fehlender Wert von Prozesstabellenfeld 'usedMappingJson'.");

    // Laden des absoluten Dateipfads auf die Vorlage.
    // Prozesskonfiguration für die im Frontend definierte Vorlage wird gesucht.
    // Wichtig: Prozesskonfiguration muss Typ "Word" sein, der Wert von `getConfiguration` ist der Dateiname inkl. Dateiendung.
    $processConfigurationValue = $this->getConfiguration($templateName);
    $processConfigurationFileRelativeDataPath = "_configurations/templates/word/$templateName/$processConfigurationValue";
    $processConfigurationFileAbsolutePath = $this->getFullDataPath(
        $processConfigurationFileRelativeDataPath, $this->getProcessName(), $this->getVersion());

    // Danach wird der Inhalt der Vorlage (vom Dokument) geladen und in Base64 umgewandelt.
    $fileContent = file_get_contents($processConfigurationFileAbsolutePath);
    $fileContentBase64 = base64_encode($fileContent);

    // Erweiterung des Mappings für das ReplacemenTemplate.
    $mapping = json_decode($mappingJson, associative: true);
    $mapping['options'] = $mapping['options'] ?? [];
    $mapping['options']['replacementTemplate'] = $fileContentBase64;

    // Rest ist gleich wie immer.
    // Handbuch: https://github.com/wertschoepfer/ws-quickdocs-docs/blob/main/latest/integration/api-functions.md#dokumentenverarbeitung
    $mappingJson = json_encode($mapping, JSON_PRETTY_PRINT);
    $result = $client->process($templateName, $mappingJson, QuickDocsResultType::WORD);
    // ...
}
```
