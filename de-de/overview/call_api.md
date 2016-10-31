# <a name="calling-the-microsoft-graph-api"></a>Aufrufen der Microsoft Graph-API

Wenn Sie auf eine Microsoft Graph-Ressource zugreifen oder diese bearbeiten möchten, müssen Sie die Ressourcen-URLs aufrufen und festlegen, indem Sie einen der folgenden Vorgänge verwenden.   

- GET
- POST
- PATCH
- PUT
- DELETE 

Alle Microsoft Graph-API-Anforderungen verwenden das folgende Basis-URL-Muster:

```
    https://graph.microsoft.com/{version}/{resource}?[query_parameters]
```

Für diese URL:

- `https://graph.microsoft.com` ist der Microsoft Graph-API-Endpunkt.
- `{version}` ist die Zieldienstversion, z. B. `v1.0` oder `beta`.
- `{resource}` ist das Ressourcensegment oder der Pfad, z. B
  - `users`, `groups`, `devices`, `organization`
  - Der Alias `me`, der den angemeldeten Benutzer auflöst.
   - Die mit einem Benutzer verknüpften Ressourcen, z. B. `me/events`, `me/drive` oder `me/messages`.
  - Der Alias `myOrganization`, der den Mandanten des bei der Organisation angemeldet Benutzers auflöst.
- `[query_parameters]` stellt zusätzliche Abfrageparameter dar, z. B. `$filter` und `$select`.

Optional können Sie auch den Mandanten als Teil der Anforderung angeben. Geben Sie bei Verwendung von `me` nicht den Mandanten an. Eine Liste allgemeiner Anforderungen finden Sie unter [Übersicht über Microsoft Graph](overview.md).

## <a name="microsoft-graph-api-metadata"></a>Microsoft Graph-API-Metadaten
Das Metadatendokument ($metadata) wird im Dienststamm veröffentlicht. Über die folgenden URLs können Sie das Dienstdokument für v1.0 und die Betaversionen anzeigen.

Metadaten für Microsoft Graph-API `v1.0`
```
    https://graph.microsoft.com/v1.0/$metadata
```
Metadaten für Microsoft Graph-API `beta`
```
    https://graph.microsoft.com/beta/$metadata
```

Mithilfe der Metadaten können Sie das Datenmodell von Microsoft Graph sehen und verstehen, einschließlich Entitätstypen und Entitätenmengen, komplexe Typen und Enumerationen, aus denen die Anforderungs- und Antwortpakete bestehen, die an und von Microsoft Graph gesendet werden. Die Metadaten können Sie verwenden, um die Beziehungen zwischen Entitäten in Microsoft Graph zu verstehen und URLs einzurichten, die zwischen Entitäten navigieren. Diese auf Navigation-basierende Vernetzung ist das charakteristische Merkmal von Microsoft Graph.

Bei den Pfad-URL-Ressourcennamen und Abfrageparametern sowie den Aktionsparametern und -werten wird nicht nach Groß-/Kleinschreibung unterschieden. Bei zugewiesenen Werten, Entitäts-IDs und anderen base64-codierte Werten wird nach Groß-/Kleinschreibung unterschieden.

In den folgenden Abschnitten sind einige grundlegende Programmierungsmusteraufrufe der Microsoft Graph-API dargestellt.

## <a name="navigate-from-a-set-to-a-member"></a>Navigation von einer Sammlung zu einem Element

Zum Anzeigen von Informationen zu einem Benutzer müssen Sie die `User`-Entität aus der `users`-Sammlung für den jeweiligen durch seinen Bezeichner angegebenen Benutzer abrufen, indem Sie eine HTTPS-GET-Anforderung verwenden. Für eine `User`-Entität kann entweder die `id`- oder `userPrincipalName`-Eigenschaft als Bezeichner verwendet werden. Im folgenden Anforderungsbeispiel wird der `userPrincipalName`-Wert als Benutzer-ID verwendet. 

```no-highlight 
GET https://graph.microsoft.com/v1.0/users/john.doe@contoso.onmicrosoft.com HTTP/1.1
Authorization : Bearer <access_token>
```

Wenn der Vorgang erfolgreich ist, erhalten Sie den Statuscode 200 OK mit der Benutzerressourcendarstellung in der Nutzlast, wie im Folgenden dargestellt:

```no-highlight 
HTTP/1.1 200 OK
content-type: application/json;odata.metadata=minimal
content-length: 982

{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users/$entity",
    "id": "c95e3b3a-c33b-48da-a6e9-eb101e8a4205",
    "city": "Redmond",
    "country": "USA",
    "department": "Help Center",
    "displayName": "John Doe",
    "givenName": "John",
    "userPrincipalName": "john.doe@contoso.onmicrosoft.com",

    ... 
}
```


## <a name="project-from-an-entity-to-properties"></a>Projektion auf eine Entität in Eigenschaften
Um nur biographische Daten des Benutzers abzurufen, z. B. die vom Benutzer bereitgestellte Beschreibung im Feld _Über mich_ und seine Qualifikationen, können Sie den _select_-Abfrageparameter zu der vorherigen Anforderung hinzufügen. Beispiel:

```no-highlight 
GET https://graph.microsoft.com/v1.0/users/john.doe@contoso.onmicrosoft.com?$select=displayName,aboutMe,skills HTTP/1.1
Authorization : Bearer <access_token>
```

Bei erfolgreicher Antwort wird der Statuscode 200 OK zurückgegeben und eine Nutzlast im folgenden Format:

```no-highlight 
HTTP/1.1 200 OK
content-type: application/json;odata.metadata=minimal
content-length: 169

{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users/$entity",
    "aboutMe": "A cool and nice guy.",
    "displayName": "John Doe",
    "skills": [
        "n-Lingual",
        "public speaking",
        "doodling"
    ]
}
```

Statt aller Eigenschaftensätze für die `user`-Entität werden hier nur die `aboutMe`-, `displayName`- und `skills`-Eigenschaften zurückgegeben.

## <a name="traverse-to-another-resource-via-relationship"></a>Durchlaufen einer anderen Ressource anhand der Beziehung
Ein Vorgesetzter enthält eine `directReports`-Beziehung zu anderen Benutzern, die diesem unterstellt sind. Zum Abfragen der Liste der direkten Mitarbeiter eines Benutzers können Sie die folgende HTTPS-GET-Anforderung zum Navigieren zum gewünschten Ziel über das Durchlaufen von Beziehungen verwenden. 

```no-highlight 
GET https://graph.microsoft.com/v1.0/users/john.doe@contoso.onmicrosoft.com/directReports HTTP/1.1
Authorization : Bearer <access_token>
```

Bei erfolgreicher Antwort wird der Statuscode 200 OK zurückgegeben und eine Nutzlast im folgenden Format:

```no-highlight 
HTTP/1.1 200 OK
content-type: application/json;odata.metadata=minimal
content-length: 152
    
{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#directoryObjects/$entity",
    "@odata.type": "#microsoft.graph.user",
    "id": "c37b074d-fe9d-4e68-83ad-b4401d3be174",
    "department": "Sales & Marketing",
    "displayName": "Bonnie Kearney",

    ...
}
```

Ebenso können Sie anhand einer Beziehung zu verwandten Ressourcen navigieren. Die `user => messages`-Beziehung ermöglicht beispielsweise eine Durchquerung von einem Azure AD-Benutzer zu einer Gruppe von Outlook-E-Mail-Nachrichten. Im nachstehenden Beispiel wird gezeigt, wie dies in einem REST-API-Aufruf funktioniert:


```no-highlight 
GET https://graph.microsoft.com/v1.0/me/messages HTTP/1.1
Authorization : Bearer <access_token>
```

    
Bei erfolgreicher Antwort wird der Statuscode 200 OK zurückgegeben und eine Nutzlast im folgenden Format:


```no-highlight 
HTTP/1.1 200 OK
content-type: application/json;odata.metadata=minimal
odata-version: 4.0
content-length: 147
    
{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users('john.doe%40contoso.onmicrosoft.com')/Messages",
  "@odata.nextLink": "https://graph.microsoft.com/v1.0/me/messages?$top=1&$skip=1",
  "value": [
    {
      "@odata.etag": "W/\"FwAAABYAAABMR67yw0CmT4x0kVgQUH/3AAJL+Kej\"",
      "id": "<id-value>",
      "createdDateTime": "2015-11-14T00:24:42Z",
      "lastModifiedDateTime": "2015-11-14T00:24:42Z",
      "changeKey": "FwAAABYAAABMR67yw0CmT4x0kVgQUH/3AAJL+Kej",
      "categories": [],
      "receivedDateTime": "2015-11-14T00:24:42Z",
      "sentDateTime": "2015-11-14T00:24:28Z",
      "hasAttachments": false,
      "subject": "Did you see last night's game?",
      "body": {
        "ContentType": "HTML",
        "Content": "<content>"
      },
      "BodyPreview": "it was great!",
      "Importance": "Normal",
            
       ...
    }
  ]
}
```

## <a name="project-from-entities-to-properties"></a>Projektion auf Entitäten in Eigenschaften
Neben der Projektion auf eine Entität in ihren Eigenschaften können Sie auch die ähnliche `select`-Abfrageoption auf eine Entitätssammlung anwenden, um eine Projektion auf eine Sammlung mit einigen Eigenschaften durchzuführen. Senden Sie zum Abfragen des Namens der Laufwerkelemente vom angemeldeten Benutzer die folgende HTTPS-GET-Anforderung:

```no-highlight 
GET https://graph.microsoft.com/v1.0/me/drive/root/children?$select=name HTTP/1.1
Authorization : Bearer <access_token>
```

Bei einer erfolgreichen Antwort werden der Statuscode 200 OK und eine Nutzlast mit den Namen und Typen der freigegebenen Dateien zurückgegeben, wie im folgenden Beispiel dargestellt:

```no-highlight 
{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users('john.doe%40contoso.onmicrosoft.com')/drive/root/children(name,type)",
  "value": [
    {
      "@odata.etag": "\"{896A8E4D-27BF-424B-A0DA-F073AE6570E2},2\"",
      "name": "Shared with Everyone"
    },
    {
      "@odata.etag": "\"{B39D5D2E-E968-491A-B0EB-D5D0431CB423},1\"",
      "name": "Documents"
    },
    {
      "@odata.etag": "\"{9B51EA38-3EE6-4DC1-96A6-230E325EF054},2\"",
      "name": "newFile.txt"
    }
  ]
}
```

## <a name="query-a-subset-of-users-with-the-filtering-query-option"></a>Abfragen einer Teilmenge von Benutzern mit der Filterabfrageoption
Wenn Sie nach Mitarbeitern mit einer bestimmten Position innerhalb Ihrer Organisation suchen möchten, können Sie aus der Benutzersammlung eine _filter_-Abfrageoption angeben. Nachfolgend ein Beispiel:

    
```no-highlight 
GET https://graph.microsoft.com/v1.0/users/?$filter=jobTitle+eq+%27Helper%27 HTTP/1.1
Authorization : Bearer <access_token>
```

Bei einer erfolgreichen Antwort werden der Statuscode 200 OK und eine Liste von Benutzern mit der angegebenen Position (`'Helper'`) zurückgegeben, wie im folgenden Beispiel dargestellt:

```no-highlight 
HTTP/1.1 200 OK
content-type: application/json;odata.metadata=minimal;odata.streaming=true;IEEE754Compatible=false;charset=utf-8
odata-version: 4.0
content-length: 986

{
    "@odata.context": "https://graph.microsoft.com/v1.0/contoso.onmicrosoft.com/$metadata#users",
    "value": [
        {
            "id": "c95e3b3a-c33b-48da-a6e9-eb101e8a4205",
            "city": "Redmond",
            "country": "USA",
            "department": "Help Center",
            "displayName": "Jane Doe",
            "givenName": "Jane",
            "jobTitle": "Helper",
            ......
            "mailNickname": "Jane",
            "mobile": null,
            "otherMails": [
                "jane.doe@contoso.onmicrosoft.com"
            ],
            ......
            "surname": "Doe",
            "usageLocation": "US",
            "userPrincipalName": "help@contoso.onmicrosoft.com",
            "userType": "Member"
        },
        
        ...
    ]
}
```

## <a name="call-actions-or-functions"></a>Aufrufen von Aktionen oder Funktionen
Microsoft Graph unterstützt auch _Aktionen_ und _Funktionen_ für die Manipulation von Ressourcen auf eine Art und Weise, wie sie mit Standard-HTTP-Methoden nicht einfach zu realisieren ist. Die folgende HTTPS-POST-Anforderung z. B. ermöglicht es dem angemeldeten Benutzer (`me`), eine E-Mail zu senden:
```no-highlight 
POST https://graph.microsoft.com/v1.0/me/sendMail HTTP/1.1
authorization: bearer <access_token>
content-type: application/json
content-length: 96

{
  "message": {
    "subject": "Meet for lunch?",
    "body": {
      "contentType": "Text",
      "content": "The new cafeteria is open."
    },
    "toRecipients": [
      {
        "emailAddress": {
          "address": "garthf@a830edad9050849NDA1.onmicrosoft.com"
        }
      }
    ],
    "attachments": [
      {
        "@odata.type": "#Microsoft.OutlookServices.FileAttachment",
        "name": "menu.txt",
        "contentBytes": "bWFjIGFuZCBjaGVlc2UgdG9kYXk="
      }
    ]
  },
  "saveToSentItems": "false"
}
```

Die Anforderungsnutzlast enthält die Eingabe für die `sendMail`-Aktion, welche auch in $metadata definiert ist.
