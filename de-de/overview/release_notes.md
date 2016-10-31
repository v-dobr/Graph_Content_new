# <a name="known-issues-with-microsoft-graph"></a>Bekannte Probleme in Microsoft Graph

Dieser Artikel beschreibt bekannte Probleme in Microsoft Graph. Informationen zu den neuesten Updates finden Sie im [Microsoft Graph Changelog](http://graph.microsoft.io/en-us/changelog).

## <a name="users"></a>Benutzer
#### <a name="no-instant-access-after-creation"></a>Kein direkter Zugriff nach der Erstellung
Benutzer können direkt über POST in der Benutzerentität erstellt werden. Eine Office 365-Lizenz muss zunächst einem Benutzer zugewiesen werden, damit dieser Zugriff auf Office 365-Dienste erhält. Aufgrund der verteilten Art des Diensts kann es bis zu 15 Minuten dauern, bis Dateien-, Nachrichten- und Ereignisentitäten für diesen Benutzer über die Microsoft Graph-API zur Verfügung stehen. In dieser Zeit erhalten Apps als Antwort den HTTP-Fehler 404. 

#### <a name="photo-restrictions"></a>Fotoeinschränkungen
Benutzerprofilfotos können nur gelesen und aktualisiert werden, wenn der Benutzer über ein Postfach verfügt. Darüber hinaus kann auf Fotos, die *ggf.* zuvor mithilfe der **thumbnailPhoto**-Eigenschaft (unter Verwendung der Office 365 Unified-API-Vorschau oder Azure AD Graph oder über die AD Connect-Synchronisierung) gespeichert wurden, nicht mehr über die Microsoft Graph-Benutzerfotoeigenschaft zugegriffen werden. Beim Auftreten eines Fehlers beim Lesen oder Aktualisieren eines Fotos wird die folgende Meldung angezeigt:

```javascript
    {
      "error": {
        "code": "ErrorNonExistentMailbox",
        "message": "The SMTP address has no mailbox associated with it."
      }
    }
```

 > **Hinweis**:  Kurz nach der allgemeinen Verfügbarkeit wird das Speichern und Abrufen von Benutzerprofilfotos aktiviert, selbst wenn der Benutzer über kein Postfach verfügt. Dann sollte dieser Fehler nicht mehr auftreten.

#### <a name="default-contacts-folder"></a>Standardordner für Kontakte

In der `/v1.0`-Version enthält `GET /me/contactFolders` keinen Standardordner für Kontakte des Benutzers. 

Es wird ein Update zur Verfügung gestellt. In der Zwischenzeit können Sie die folgende [list contacts](http://graph.microsoft.io/docs/api-reference/v1.0/api/user_list_contacts)-Abfrage und die **parentFolderId**-Eigenschaft als Problemumgehung verwenden, um die ID des Standardordners für Kontakte abzurufen:

```
GET https://graph.microsoft.com/v1.0/me/contacts?$top=1&$select=parentFolderId
```
In der obigen Abfrage gilt Folgendes:
1. `/me/contacts?$top=1` ruft die Eigenschaften eines [Kontakts](http://graph.microsoft.io/docs/api-reference/v1.0/resources/contact) im Standardordner für Kontakte ab.
2. Das Anfügen von `&$select=parentFolderId` gibt nur die **parentFolderId**-Eigenschaft des Kontakts zurück, also die ID des Standardordners für Kontakte.

#### <a name="adding-and-accessing-ics-based-calendars-in-user's-mailbox"></a>Hinzufügen von und Zugreifen auf ICS-basierte Kalender im Postfach des Benutzers
Derzeit gibt es eine teilweise Unterstützung für einen Kalender, der auf einem Internet-Kalenderabonnement (ICS) basiert:
* Sie können einen ICS-basierten Kalender einem Benutzerpostfach über die Benutzeroberfläche, jedoch nicht über die Microsoft Graph-API hinzufügen. 
* [Das Auflisten der Kalender des Benutzers](http://graph.microsoft.io/docs/api-reference/v1.0/api/user_list_calendars) ermöglicht Ihnen, die Eigenschaften **Name**, **Farbe** und **ID** jedes [Kalenders](http://graph.microsoft.io/docs/api-reference/v1.0/resources/calendar) in der Standardkalendergruppe oder einer bestimmten Kalendergruppe des Benutzers abzurufen, einschließlich ICS-basierte Kalender. Sie können die ICS-URL in der Kalenderressource nicht speichern und nicht darauf zugreifen.
* Sie haben auch die Möglichkeit zum [Auflisten der Ereignisse](http://graph.microsoft.io/docs/api-reference/v1.0/api/calendar_list_events) eines ICS-basierten Kalenders.

## <a name="groups"></a>Gruppen
#### <a name="policy"></a>Richtlinie
Beim Erstellen und Benennen von einheitlichen Gruppen mit Microsoft Graph werden einheitliche Gruppenrichtlinien umgegangen, die über die Outlook Web App konfiguriert wurden. 

#### <a name="group-permission-scopes"></a>Berechtigungsbereiche für Gruppen
Microsoft Graph stellt zwei Berechtigungsbereiche (*Group.Read.All* und *Group.ReadWrite.All*) für den Zugriff auf Gruppen-APIs zur Verfügung.  Diesen Berechtigungsbereichen muss ein Administrator seine Zustimmung erteilen (Dies ist anders als in der Vorschauversion).  In Zukunft werden neue Bereiche für Gruppen hinzugefügt, für die Benutzer Ihre Zustimmung erteilen können.

#### <a name="adding-and-getting-attachments-of-group-posts"></a>Hinzufügen und Abrufen von Anlagen von Gruppenbeiträgen
Durch das [Hinzufügen](http://graph.microsoft.io/docs/api-reference/v1.0/api/post_post_attachments) von Anlagen zu Gruppenbeiträgen, das [Auflisten](http://graph.microsoft.io/docs/api-reference/v1.0/api/post_list_attachments) und das Abrufen von Anlagen von Gruppenbeiträgen wird aktuell die Fehlermeldung „Die OData-Anforderung wird nicht unterstützt“ zurückgegeben. Für die `/v1.0`- und `/beta`-Versionen wurde eine Problemlösung entwickelt, die bis Ende Januar 2016 allgemein verfügbar sein sollte.

## <a name="contacts"></a>Kontakte
* Derzeit werden nur persönliche Kontakten unterstützt. Organisationskontakte werden derzeit nicht in `/v1.0` unterstützt, sind jedoch in `/beta` vorhanden.
* Die Mobiltelefonnummer von persönlichen Kontakten wird für einen Kontakt nicht zurückgegeben. Diese wird in Kürze hinzugefügt. In der Zwischenzeit können sie über Outlook-APIs auf diese zugreifen.

### <a name="drives,-files-and-content-streaming"></a>Laufwerke, Dateien und Streamen von Inhalten
* Beim ersten Zugriff auf ein persönliches Benutzerlaufwerk über Microsoft Graph tritt ein 401-Fehler auf, wenn der Benutzer seine persönliche Website noch nicht in einem Browser aufgerufen hat.
* Es können Dateien (Dateien in Office-Gruppen, Laufwerke oder E-Mail-Dateianlagen) mit einer Größe von bis zu 4 MB hoch- und heruntergeladen werden.

## <a name="functionality-available-only-in-office-365-rest-apis"></a>In Office 365-REST-APIs verfügbare Funktionen

Einige Funktionen sind noch nicht verfügbar in Microsoft Graph. Wenn Sie die gesuchte Funktionalität nicht finden, können Sie die endpunkt-spezifischen [Office 365 REST-APIs](https://msdn.microsoft.com/en-us/office/office365/api/api-catalog) verwenden.

#### <a name="synchronization"></a>Synchronisierung
Outlook-, OneDrive- und Azure AD-Synchronisierungsfunktionen (in Azure AD auch differenzielle Abfrage genannt) stehen in `/v1.0` oder `/beta` nicht zur Verfügung.  Wenn für die Anwendung Synchronisierungsfunktionen erforderlich sind, verwenden Sie vorhandene Office 365- und Azure AD-REST-APIs oder testen Sie das neue Webhooks-Vorschaufeature über Microsoft Graph für Ereignisse, Nachrichten und Kontakte.

> **HINWEIS**: Unser Ziel ist, die Lücke zwischen vorhandenen APIs und Microsoft Graph einschließlich Synchronisierung so schnell wie möglich zu schließen.

#### <a name="batching"></a>Batchverarbeitung
Batchverarbeitung wird von Microsoft Graph nicht unterstützt. Sie können jedoch die Outlook-Betaendpunkte und [Outlook-REST-Batchaufrufe](https://msdn.microsoft.com/en-us/office/office365/api/batch-outlook-rest-requests) verwenden. 

#### <a name="availability-in-china"></a>Verfügbarkeit in China
Der Microsoft Graph-Dienst wird von21Vianet betrieben (und ist jetzt in China verfügbar). Lesen Sie [Unabhängige Microsoft Graph-Cloudbereitstellungen](http://graph.microsoft.io/docs/overview/deployments), um weitere Informationen zu erhalten und mehr über die Einschränkungen zu erfahren.

#### <a name="service-actions-and-functions"></a>Dienstaktionen und Funktionen
`isMemberOf` und `getObjectsById` sind in Microsoft Graph nicht verfügbar.

## <a name="microsoft-graph-permissions"></a>Microsoft Graph-Berechtigungen
Neueste Informationen zu den von Microsoft Graph unterstützten Anwendungs- und delegierten Berechtigungen finden Sie im Thema [Berechtigungsbereiche](http://graph.microsoft.io/docs/authorization/permission_scopes). Darüber hinaus gelten die folgenden Einschränkungen für `v1.0`:

|Berechtigung |   Berechtigungstyp | Einschränkung |  Alternative |
|-----------|-----------------|------------|--------------|
|_User.ReadWrite_| Delegiert    | Mobiltelefonnummer kann nicht aktualisiert werden.|    Wählen Sie auch `Directory.AccessAsUser.All`| 
|_User.ReadWrite.All_|  Delegiert|  Es können keine anderen CRUD-Vorgänge für `User` ausgeführt werden als das Aktualisieren des HD-Fotos vom Benutzer und der erweiterten Profileigenschaften.| Wählen Sie `Directory.ReadWrite.All` oder `Directory.AccessAsUser.All`, wenn Löschen von Benutzern erforderlich ist.|
|_User.Read.All_|   Anwendung |Es können keine Lesevorgänge für andere Benutzer ausgeführt werden.| Wählen Sie auch `Directory.Read.All`|
| _User.ReadWrite.All_ |    Anwendung |   Es können keine anderen CRUD-Vorgänge für `User` ausgeführt werden als das Aktualisieren des HD-Fotos vom Benutzer und der erweiterten Profileigenschaften. |    Wählen Sie auch `Directory.ReadWrite.All`. **HINWEIS**: Löschen von Benutzern ist nicht möglich.|
|_Group.Read.All_   | Anwendung | Gruppen oder Gruppenmitgliedschaften können nicht aufgelistet werden.  Gruppeninhalte können weiterhin für Office-Gruppen gelesen werden.	   | Wählen Sie auch `Directory.Read.All` |
|_Group.ReadWrite.All_  | Anwendung   | Auflisten von Gruppen oder Gruppenmitgliedschaften, Erstellen von Gruppen, Aktualisieren von Gruppenmitgliedschaften oder Löschen von Gruppen ist nicht möglich.  Gruppeninhalte können weiterhin für Office-Gruppen gelesen und aktualisiert werden.	   | Wählen Sie auch `Directory.ReadWrite.All`. **HINWEIS**:  Löschen von Gruppen ist nicht möglich.|

Darüber hinaus gelten die folgenden `/beta`-Einschränkungen:

|Berechtigung |   Berechtigungstyp | Einschränkung |  Alternative |
|-----------|-----------------|------------|--------------|
| _Group.ReadWrite.All_ | Delegiert | Planeraufgaben können in Office-Gruppen nicht gelesen oder aktualisiert werden.	  | Wählen Sie auch `Tasks.ReadWrite`|
|_Tasks.ReadWrite_  | Delegiert | Aufgaben von angemeldeten Benutzern können nicht gelesen oder aktualisiert werden.| Wählen Sie auch `Group.ReadWrite.All`|

## <a name="odata-related-limitations"></a>OData-bezogene Einschränkungen
* **$expand**-Einschränkungen: 
 * Keine Unterstützung für `nextLink`.
 * Keine Unterstützung für mehr als eine Erweiterungsebene.
 * Keine Unterstützung mit zusätzlichen Parametern (**$filter**, **$select**).
* Mehrere Namespaces werden nicht unterstützt.
* GET-Anforderungen für `$ref` und Umwandlung wird für Benutzer, Gruppen, Geräte, Dienstprinzipale und Anwendungen nicht unterstützt.
* `@odata.bind` wird nicht unterstützt.  Ein Entwickler kann daher `Accepted` oder `RejectedSenders` für eine Gruppe nicht ordnungsgemäß festlegen.
* `@odata.id` ist bei Nicht-Aufnahmenavigationen (z. B. Nachrichten) nicht vorhanden, wenn minimale Metadaten verwendet werden.
* Arbeitsauslastungsübergreifende Filter/Suche ist nicht verfügbar. 
* Volltextsuche (unter Verwendung von **$search**) ist nur für einige Entitäten wie Nachrichten verfügbar.

  >  Ihr Feedback ist uns wichtig. Nehmen Sie unter [Stack Overflow](http://stackoverflow.com/questions/tagged/office365) Kontakt mit uns auf. Taggen Sie Ihre Fragen mit [MicrosoftGraph] und [office365].

  
             .

