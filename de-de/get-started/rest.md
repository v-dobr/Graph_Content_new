# <a name="get-started-with-microsoft-graph-and-rest"></a>Erste Schritte mit Microsoft Graph und REST

Dieser Artikel beschreibt die erforderlichen Aufgaben zum Abrufen eines Zugriffstokens vom Azure AD v2.0-Endpunkt und zum Aufrufen von Microsoft Graph mithilfe von REST-Aufrufen. Er beschreibt die Abfolge der Anforderungen und Antworten, die eine App zum Authentifizieren und Abrufen von E-Mail-Nachrichten verwendet.

Zuerst müssen Sie Ihre App beim Azure Active Directory (Azure AD) registrieren. 

## <a name="register-the-application"></a>Registrieren der App

Es gibt derzeit zwei Optionen für die Registrierung einer App bei Azure AD.

  - Registrieren Sie eine App für die Verwendung des Azure AD v2.0-Endpunkts, der sowohl für persönliche (Microsoft) Identitäten als auch für Geschäfts- und Schulkonten (Azure AD) geeignet ist.
  - Registrieren Sie eine App für die Verwendung des Azure AD-Endpunkts, der nur Geschäfts- und Schulkonten unterstützt.

In diesem Artikel wird eine v2.0-Registrierung vorausgesetzt, weshalb Sie Ihre App im [App-Registrierungsportal](https://apps.dev.microsoft.com/) registrieren. Folgen Sie den Anweisungen in [Registrieren Ihrer Microsoft Graph-Anwendung mit dem Azure AD-Version 2.0-Endpunkt](../authorization/auth_register_app_v2.md), um Ihre App zu registrieren. Informationen zur Verwendung des Endpunkts Azure AD finden Sie unter [Authentifizieren mit Azure AD](../authorization/app_authorization.md).

> Einige Einschränkungen sind zu beachten, wenn Sie den Endpunkt Version 2.0 verwenden. Damit Sie die für Sie richtige Entscheidung treffen können, informieren Sie sich unter [Sollte ich den v2.0-Endpunkt verwenden?](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/)

## <a name="authenticate-the-user-and-get-an-access-token"></a>Authentifizierung des Benutzers und Abrufen eines Zugriffstokens

Die in diesem Artikel beschriebene App implementiert den [Authorization Code Grant-Datenfluss](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols-oauth-code/) zum Abrufen der Zugriffstoken von dem Azure AD v2.0-Endpunkt gemäß standardmäßiger[ OAuth 2.0-Protokolle](http://tools.ietf.org/html/rfc6749). Eine vollständige Anleitung zu den im Azure AD v2.0-Endpunkt unterstützten Datenflüssen finden Sie unter [v2.0-Endpunkt-Typen](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-flows/).

Mit dem Authorization Code Grant-Datenfluss rufen Sie zuerst einen Autorisierungscode ab und tauschen diesen Code dann gegen ein Zugangstoken aus.

### <a name="getting-an-authorization-code"></a>Abrufen eines Autorisierungscodes

Im ersten Schritt des Authorization Code Grant-Datenflusses wird der Autorisierungscode abgerufen. Der Code wird vom Autorisierungsserver an die App übergeben, wenn der Benutzer sich anmeldet und den von der App angeforderten Berechtigungen zustimmt.

Sie fordern einen Autorisierungscode an, indem Sie eine GET-Anforderung an den Azure AD v2.0-Endpunkt senden. Diese URL muss in einem Browser geöffnet werden, damit der Benutzer sich anmelden und seine Zustimmung erteilen kann.

#### <a name="construct-the-request"></a>Erstellen der Anforderung

1 – Beginnen Sie mit der Basis-URL:

```
https://login.microsoftonline.com/<tenant>/oauth2/v2.0/authorize
```

Das *Mandanten*-Segment im Pfad bestimmt, wer sich bei der Anwendung anmelden kann. Zulässige Werte sind *Allgemein*, *Organisationen*, *Consumer* und Mandantenbezeichner. Weitere Informationen finden Sie unter [Protokoll-Endpunkte](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/#endpoints).

2 – Hängen Sie Abfrageparameter auf der Basis-URL an. Diese werden verwendet, um die App, die erforderlichen Berechtigungen und andere Informationen zur Anforderung der Authentifizierung zu identifizieren. In der folgenden Tabelle werden einige allgemeine Parameter beschrieben:

| Parameter | Beschreibungen |
|:------|:------|
| client_id | Die beim Registrieren der App generierte Client-ID. Anhand dieser erkennt Azure AD, welche App die Anmeldung anfordert. |
| redirect_uri | Der Speicherort, zu dem Azure umleitet, nachdem der Benutzer seine Zustimmung für diese App erteilt hat. Dieser Wert muss mit dem Wert des beim Registrieren der App verwendeten **Umleitungs-URI** übereinstimmen. |
| response_type | Der Antworttyp, den die App erwartet. Dieser Wert ist `code` für den Authorization Code Grant-Datenfluss. |
| Bereich | Eine durch Leerzeichen getrennte Liste von [Microsoft Graph Berechtigungsbereichen](../authorization/permission_scopes.md), die die App anfordert. Sie können auch [OpenId Connect-Bereiche](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-scopes/#openid-connect-scopes) für [die einmalige Anmeldung](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols-oidc/) angeben.  |
| state | Ein Wert, der in der Anforderung enthalten ist, die in der Tokenantwort für die Validierung zurückgegeben wird. |

Die Anforderungs-URL für eine Anwendung, die den Lesezugriff auf E-Mails anfordert, könnte beispielsweise wie folgt aussehen.

```
GET https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=<app ID>&redirect_uri=http%3A%2F%2Flocalhost/myapp%2F&response_type=code&state=1234&scope=mail.read
```

3 – Leiten Sie dann den Benutzer zu der Anmelde-URL weiter. Es wird ein Anmeldefenster mit dem Namen der App angezeigt. Nach dem Anmelden wird eine Liste der Berechtigungen angezeigt, die die App anfordert, und der Benutzer wird aufgefordert, diesen zuzustimmen oder sie zu verweigern. Wenn der Benutzer zustimmt, veranlasst der Browser die Umleitung zu dem Umleitungs-URI mit dem Autorisierungscode und dem Status in der Abfragezeichenfolge, wie im folgenden Beispiel gezeigt.

```
http://localhost/myapp/?code=AwABAAAA...cZZ6IgAA&state=1234
```

Im nächsten Schritt wird der für ein Zugriffstoken zurückgegebene Autorisierungscode ausgetauscht.

### <a name="getting-an-access-token"></a>Abrufen eines Zugriffstokens

Zum Abrufen eines Zugriffstokens stellt die App fomularcodierte Parameter für die Tokenanforderungs-URL (beispielsweise `https://login.microsoftonline.com/common/oauth2/v2.0/token`) mit den folgenden Parametern bereit.

| Parameter | Beschreibungen |
|:------|:------|
| client_id | Die beim Registrieren der App generierte Client-ID. |
| client_secret | Der beim Registrieren der App generierte geheime Anwendungsschlüssel. |
| code | Der im vorherigen Schritt abgerufene Autorisierungscode. |
| redirect_uri | Dieser Wert muss mit dem in der Autorisierungscodeanforderung verwendeten Wert übereinstimmen. |
| grant_type | Der von der App verwendete Typ des Zugriffs. Dieser Wert ist `code` für den Authorization Code Grant-Datenfluss. |
| Bereich | Eine durch Leerzeichen getrennte Liste von [Microsoft Graph Berechtigungsbereichen](../authorization/permission_scopes.md), die die App anfordert. |

Die Anforderungs-URL für unsere Anwendung sieht unter Verwendung des Codes aus dem vorherigen Schritt wie folgt aus.

```
POST https://login.microsoftonline.com/common/oauth2/v2.0/token
Content-Type: application/x-www-form-urlencoded

{
  grant_type=authorization_code
  &code=AwABAAAA...cZZ6IgAA
  &redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
  &resource=https%3A%2F%2Fgraph.microsoft.com%2F
  &scope=mail.read
  &client\_id=<app ID>
  &client\_secret=<app SECRET>
}
```

Der Server antwortet mit einer JSON-Nutzlast, die das Zugriffstoken enthält.

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
  "token_type":"Bearer",
  "expires_in":"3600",
  "access_token":"eyJ0eXAi...b66LoPVA",
  "scope":"https://graph.microsoft.com/mail.read",
  ...
}
```

Das Zugriffstoken befindet sich im Feld `access_token` der JSON-Nutzlast. Die App verwendet diesen Wert zum Festlegen des Autorisierungsheaders bei REST-Aufrufen für die API.

## <a name="call-microsoft-graph"></a>Aufrufen von Microsoft Graph

Wenn die App über ein Zugriffstoken verfügt, kann Microsoft Graph aufgerufen werden. Da diese Beispiel-App Nachrichten abruft, verwendet sie eine HTTP-GET-Anforderung für den `https://graph.microsoft.com/v1.0/me/messages`-Endpunkt.

### <a name="refine-the-request"></a>Beschränken der Anforderung

Apps können das Verhalten der GET-Anforderungen mithilfe von OData-Abfrageparametern steuern. Diese Parameter sollten von den Apps verwendet werden, um die Anzahl der zurückgegebenen Ergebnisse und der für jedes Element zurückgegebenen Felder einzuschränken. 

In dieser Beispiel-App werden die Nachrichten in einer Tabelle angezeigt, in der Betreff, Absender und der Zeitpunkt enthalten sind, zu dem die Nachricht empfangen wurde. In der Tabelle werden maximal 25 Zeilen angezeigt, wobei diese so sortiert sind, dass zuletzt empfangene Nachrichten oben aufgeführt sind. Die App verwendet die folgenden Abfrageparameter, um diese Ergebnisse zu erzielen.

- `$select` - Gibt nur die Felder `subject`, `sender` und `dateTimeReceived` an.
- `$top` - Gibt maximal 25 Elemente an.
- `$orderby` - Sortiert die Ergebnisse nach dem `dateTimeReceived`-Feld.

Als Ergebnis erhält man die folgende Anforderung.

```
GET https://graph.microsoft.com/v1.0/me/messages?$select=subject,from,receivedDateTime&$top=25&$orderby=receivedDateTime%20DESC
Accept: application/json
Authorization: Bearer eyJ0eXAi...b66LoPVA
```

Da Sie nun gelernt haben, wie Aufrufe für Microsoft Graph getätigt werden, können Sie anhand der API-Referenz beliebige Arten von Aufrufen erstellen, die für Ihre App erforderlich sind. Denken Sie daran, dass entsprechende Berechtigungen für die App bei der App-Registrierung für die Aufrufe konfiguriert sein müssen.

## <a name="next-steps"></a>Nächste Schritte
- Testen Sie die Möglichkeiten der REST-API mithilfe des [Graph-Explorers](https://graph.microsoft.io/graph-explorer).

## <a name="see-also"></a>Siehe auch
- [Azure AD v2.0-Protokolle](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
- [Azure AD v2.0-Tokens](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)
