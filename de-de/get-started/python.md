# <a name="get-started-with-microsoft-graph-in-a-python-app"></a>Erste Schritte mit Microsoft Graph in einer Python-App 

Dieser Artikel beschreibt die erforderlichen Aufgaben zum Abrufen eines Zugriffstokens aus dem Azure AD und zum Aufrufen von Microsoft Graph. Sie werden durch das [Microsoft Connect-Beispiel für Python](https://github.com/microsoftgraph/python3-connect-rest-sample) geführt und erhalten Informationen zu den Hauptkonzepten, die Sie zur Verwendung der Microsoft Graph-API implementieren. In diesem Artikel wird beschrieben, wie Sie mithilfe von direkten REST-Aufrufen auf Microsoft Graph zugreifen.

![Screenshot des Office 365 Phyton Connect-Beispiels](./images/web-screenshot.png)

> **Hinweis** Diese exemplarische Vorgehensweise und das Beispiel, auf der sie basiert, verwenden den Endpunkt Azure AD. Halten Sie immer mal wieder Ausschau nach aktualisierten Versionen, die den Endpunkt Azure AD v2.0 verwenden.

##  <a name="prerequisites"></a>Voraussetzungen

  * Ein Office 365 for Business-Konto Sie können sich für ein [Office 365-Entwicklerabonnement](https://msdn.microsoft.com/en-us/office/office365/howto/setup-development-environment#bk_Office365Account) registrieren. Dieses umfasst die Ressourcen, die Sie zum Erstellen von Office 365-Apps benötigen.
  * [Microsoft Graph Connect-Beispiel für Python](https://github.com/microsoftgraph/python3-connect-rest-sample)

## <a name="register-the-application-in-azure-active-directory"></a>Registrieren der Anwendung in Azure Active Directory

Sie müssen Ihre Anwendung zuerst registrieren und die Berechtigungen für die Verwendung von Microsoft Graph festlegen. Dann können Benutzer sich bei der Anwendung mit Geschäfts- oder Schulkonten anmelden.

1. Melden Sie sich beim [Azure-Portal](https://portal.azure.com/) an.
2. Klicken Sie in der oberen Leiste auf Ihr Konto, und wählen Sie unter der Liste **Verzeichnis** den Active Directory-Mandanten aus, bei dem Sie Ihre Anwendung registrieren möchten.
3. Klicken Sie im linken Navigationsbereich auf **Weitere Dienste**, und wählen Sie **Azure Active Directory** aus.
4. Klicken Sie auf **App-Registrierungen**, und wählen Sie **Hinzufügen** aus.
5. Geben Sie einen Anzeigenamen für die Anwendung ein, z. B. 'MSGraphConnectPython', und wählen Sie „Web-app/API“ als **Anwendungstyp** aus. Geben Sie als URL für die Anmeldung „http://127.0.0.1:8000/connect/get_token/“ ein. Klicken Sie auf **Erstellen**, um die Anwendung zu erstellen.
6. Während Sie sich noch im Azure-Portal befinden, wählen Sie die Anwendung aus, und klicken Sie auf **Einstellungen** und dann auf **Eigenschaften**.
7. Suchen Sie den Wert der Anwendungs-ID, und kopieren Sie ihn in die Zwischenablage.
8. Konfigurieren von Berechtigungen für Ihre Anwendung:
9. Wählen Sie im Menü **Einstellungen** den Abschnitt **Erforderliche Berechtigungen** aus, klicken Sie auf **Hinzufügen** und dann auf **Eine API auswählen**, und wählen Sie **Microsoft Graph** aus.
10. Klicken Sie dann auf „Berechtigungen auswählen“, und wählen Sie **Anmelden und Lesen des Benutzerprofils aktivieren** sowie**E-Mails als Benutzer senden** aus. Klicken Sie auf **Auswählen** und dann auf **Fertig**.
11. Wählen Sie im Menü **Einstellungen** den Abschnitt **Schlüssel** aus. Geben Sie eine Beschreibung ein, und wählen Sie eine Dauer für den Schlüssel aus. Klicken Sie auf **Speichern**.
12. **Wichtig**: Kopieren Sie den Wert des Schlüssels. Sie können auf diesen Wert nicht erneut zugreifen, nachdem Sie in diesen Bereich verlassen haben. Sie verwenden diesen Wert als Ihr App-Geheimnis.

## <a name="redirect-the-browser-to-the-sign-in-page"></a>Weiterleiten des Browsers zur Anmeldeseite

Die App muss den Browser an die Anmeldeseite weiterleiten, um den OAuth-Fluss zu beginnen und einen Autorisierungscode zu erhalten. 

Im Connect-Beispiel erstellt der folgende Code (befindet sich in [*connect/auth_helper.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/auth_helper.py)) die URL, die die App an den Benutzer weiterleiten muss und die an die Ansicht weitergeleitet wird, in der sie für die Umleitung verwendet werden kann. 

```python
# This function creates the signin URL that the app will
# direct the user to in order to sign in to Office 365 and
# give the app consent.
def get_signin_url(redirect_uri):
  # Build the query parameters for the signin URL.
  params = { 'client_id': client_id,
             'redirect_uri': redirect_uri,
             'response_type': 'code'
           }

  authorize_url = '{0}{1}'.format(authority, '/common/oauth2/authorize?{0}')
  signin_url = authorize_url.format(urlencode(params))
  return signin_url
```

<!--<a name="authCode"></a>-->
## <a name="receive-an-authorization-code-in-your-reply-url-page"></a>Erhalten eines Autorisierungscodes auf Ihrer Antwort-URL-Seite

Nachdem der Benutzer sich angemeldet hat, wird der Browser an Ihre Antwort URL, die ```get_token```-Funktion in [*connect/views.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/views.py), mit einem Autorisierungscode weitergeleitet, der der Abfragezeichenfolge als ```code```-Variable angefügt ist. 

Das Connect-Beispiel ruft den Code aus der Abfragezeichenfolge ab, sodass es diesen anschließend gegen ein Zugriffstoken austauschen kann.

```python
auth_code = request.GET['code']
```

<!--<a name="accessToken"></a>-->
## <a name="request-an-access-token-from-the-token-issuing-endpoint"></a>Anfordern eines Zugriffstokens vom Endpunkt, der ein Token ausgibt

Nachdem Sie den Autorisierungscode erhalten haben, können Sie diesen zusammen mit den Werten für die Client-ID, den Schlüssel und die Antwort-URL verwenden, die Sie von Azure Active Directory zum Anfordern eines Zugriffstokens erhalten haben. 

> **Hinweis** Die Anforderung muss auch eine Ressource angeben, die Sie nutzen möchten. Im Falle von Microsoft Graph lautet der Ressourcenwert `https://graph.microsoft.com`.

Das Connect-Beispiel fordert ein Token in der ```get_token_from_code```-Funktion in der Datei [*connect/auth_helper.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/auth_helper.py) an.

```python
# This function passes the authorization code to the token
# issuing endpoint, gets the token, and then returns it.
def get_token_from_code(auth_code, redirect_uri):
  # Build the post form for the token request
  post_data = { 'grant_type': 'authorization_code',
                'code': auth_code,
                'redirect_uri': redirect_uri,
                'client_id': client_id,
                'client_secret': client_secret,
                'resource': 'https://graph.microsoft.com'
              }
              
  r = requests.post(token_url, data = post_data)
  
  try:
    return r.json()
  except:
    return 'Error retrieving token: {0} - {1}'.format(r.status_code, r.text)
```

> **Hinweis** Die Antwort bietet mehr Informationen als nur das Zugriffstoken. Beispielsweise kann Ihre App ein Aktualisierungstoken abrufen, um neue Zugriffstoken anzufordern, ohne dass sich der Benutzer erneut explizit anmelden muss.

<!--<a name="request"></a>-->
## <a name="use-the-access-token-in-a-request-to-the-microsoft-graph-api"></a>Verwenden des Zugriffstokens in einer Anforderung an die Microsoft Graph-API

Mit einem Zugriffstoken kann Ihre App authentifizierte Anforderungen an die Microsoft Graph-API senden. Ihre App muss das Zugriffstoken an den **Autorisierungs**-Header jeder Anforderung anfügen.

Im Connect-Beispiel wird eine E-Mail mithilfe des ```me/microsoft.graph.sendMail```-Endpunkts in der Microsoft Graph-API gesendet. Der Code ist in der ```call_sendMail_endpoint```-Funktion in der Datei [*connect/graph_service.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/graph_service.py) vorhanden. Dies ist der Code, der zeigt, wie dem Header „Autorisierung“ der Zugriffscode angehängt wird.

```python
# Set request headers.
headers = { 
  'User-Agent' : 'python_tutorial/1.0',
  'Authorization' : 'Bearer {0}'.format(access_token),
  'Accept' : 'application/json',
  'Content-Type' : 'application/json'
}
```

> **Hinweis** Die Anforderung muss auch einen Header des Typs **Content-Type** mit einem Wert senden, der von der Graph-API, z. B. `application/json`, akzeptiert wird.

Die Microsoft Graph-API ist eine leistungsfähige einheitliche API, die für die Interaktion mit beliebigen Microsoft-Daten verwendet werden kann. Informationen zu weiteren Möglichkeiten mit Microsoft Graph finden Sie in der API-Referenz.