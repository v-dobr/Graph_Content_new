# <a name="call-microsoft-graph-in-a-service-or-daemon-app"></a>Aufrufen von Microsoft Graph in einem Dienst oder einer Dämon-App

In diesem Artikel werden Aufgaben beschrieben, die zum Herstellen einer Verbindung zwischen Ihrem Einzelinstanzdienst oder Dämon-App und Office 365 und zum Aufrufen der Microsoft Graph-API mindestens erforderlich sind.

> Hinweis: Dieses Thema befasst sich mit der Verwendung von Azure Active Directory als Authentifizierungsanbieter für Ihre App. Weitere Informationen zum Implementieren einer Dienst- oder Daemon-App mithilfe des Azure AD v2.0-Endpunkts finden Sie unter <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols-oauth-client-creds/" target="_newtab">v2.0-Protokolle - OAuth 2.0 Client-Anmeldeinformationen-Workflow</a>.

## <a name="register-the-application-in-azure-active-directory"></a>Registrieren der Anwendung in Azure Active Directory

Bevor Sie mit Office 365 arbeiten können, müssen Sie Ihre App im [Azure-Portal](https://portal.azure.com) registrieren. Bedenken Sie dabei Folgendes:

- Konfigurieren Sie nach der Registrierung der Anwendung die Anwendungsberechtigungen, die für Ihren Dienst oder Ihre Dämon-App erforderlich sind.
- Notieren Sie sich die folgenden Werte bei der Registrierung Ihrer Azure-App. Sie benötigen diese Werte, um den OAuth-Fluss in Ihrem Dienst oder Ihrer Dämon-App zu konfigurieren:
    * App-ID (eindeutig für Ihre Anwendung)
    * App-Schlüssel oder Secret (eindeutig für Ihre App)
    * OAuth 2.0-Tokenendpunkt Ihrer App
      * Klicken Sie für diesen Wert auf *Endpunkte anzeigen* am unteren Rand des Azure-Verwaltungsportals auf der App-Seite. Der Endpunkt sieht wie `https://login.microsoftonline.com/<tenantId>/oauth2/token` aus.

## <a name="request-an-access-token-from-the-token-issuing-endpoint"></a>Anfordern eines Zugriffstokens vom Endpunkt, der ein Token ausgibt

Im Gegensatz zu Client-Apps kann sich ein Benutzer nicht bei Ihrem Dienst oder Ihrer Dämon-App anmelden und die Anwendung autorisieren. In diesem Fall muss die Anwendung den Zugriffsfluss für die OAuth 2.0-Clientanmeldeinformationen implementieren, damit die Anwendung eigene Anmeldeinformationen, die Client-ID und einen Anwendungsschlüssel für die Authentifzierung zum Aufrufen von Microsoft Graph verwenden kann, statt die Identität eines Benutzers anzunehmen. Detail zum Authentifizierungsfluss finden Sie unter [Dienst-zu-Dienst-Anrufe über Clientanmeldeinformationen](https://msdn.microsoft.com/en-us/library/azure/dn645543.aspx).

Führen Sie eine HTTP POST-Anforderung an den Endpunkt, der ein Token ausgibt, mit den folgenden Parametern durch. Ersetzen Sie dabei `<clientId>` und `<clientSecret>` durch die Client-ID Ihrer App bzw. den Anwendungsschlüssel.

```http
POST https://login.microsoftonline.com/<tenantId>/oauth2/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials
&client_id=<clientId>
&client_secret=<clientSecret>
&resource=https://graph.microsoft.com
```

Als Antwort werden ein Zugriffstoken und Ablauf Access-Token und Gültigkeitsinformationen zurückgegeben.

```json
{ 
  "token_type": "Bearer",
  "expires_in": "3599",
  "scope": "User.Read",
  "expires_on": "1449685363",
  "not_before": "1449681463",
  "resource": "https://graph.microsoft.com",
  "access_token": "<token>"
}
```

## <a name="use-the-access-token-in-a-request-to-the-microsoft-graph-api"></a>Verwenden des Zugriffstokens in einer Anforderung an die Microsoft Graph-API

Mit einem Zugriffstoken kann Ihre App authentifizierte Anforderungen an die Microsoft Graph-API senden. Ihre App muss das Zugriffstoken an den **Autorisierungs**-Header jeder Anforderung anfügen.

Ein Dienst oder eine Dämon-App kann z. B. alle Benutzer in einem Mandanten abrufen, wenn für diesen bzw. diese die Berechtigung *Vollständige Profile aller Benutzer lesen* im Azure-Verwaltungsportal ausgewählt ist. 

```http
GET https://graph.microsoft.com/v1.0/users
Authorization: Bearer <token>
```
