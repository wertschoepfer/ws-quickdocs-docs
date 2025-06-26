# Integration von QuickDocs in JobRouter

QuickDocs stellt eine HTTP-REST-API zur schnellen und einfachen Verwaltung und Verarbeitung von Word- und PDF-Dokumenten bereit.

Designt ist QuickDocs als Erweiterung für JobRouter Prozesse, ist aber auch eigenständig nutzbar.  
Die Integration in Prozesse/Anwendungen ist über einfache HTTP-Anfragen, sowohl von JS (AJAX) als auch PHP (cURL) möglich.

Die REST-API ist eine ASP\.NET Anwendung und lässt sich auf Windows Servern mit IIS betreiben.  
Alternativ kann ein Linux Docker Container verwendet werden, ein Image ist vorhanden.
[📗 Wechseln zur Administration (Installation, Update)](./../admin/_toc.md)

**Navigation:**

-   [📃 Allgemeine Informationen zur API](api-infos.md)
-   [📃 Authentifizierung mit API-Tokens](api-auth.md)
-   [📄 Funktionen der QuickDocs-API](api-functions.md)
-   [📄 Integration der API in JobRouter-Prozesse](integration-jr.md)

**❓Knowledge Base:**

-   [📄 Wie lassen sich leere Felder bei der Verarbeitung entfernen?](kb-empty-fields.md)
-   [📄 Wie kann ich Abstand zwischen zwei Sätzen in einem Marker setzen?](kb-margin.md)
