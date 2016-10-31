# <a name="get-started-with-microsoft-graph-in-a-node.js-app"></a>Erste Schritte mit Microsoft Graph in einer Node.js-App

Dieser Artikel beschreibt die erforderlichen Aufgaben zum Abrufen eines Zugriffstokens vom Azure AD v2.0-Endpunkt und zum Aufrufen von Microsoft Graph. Sie werden durch die Erstellung des [Microsoft Connect-Beispiels für Node.js](https://github.com/microsoftgraph/nodejs-connect-rest-sample) geführt und erhalten Informationen zu den Hauptkonzepten, die Sie zur Verwendung von Microsoft Graph implementieren. In diesem Artikel wird beschrieben, wie Sie mithilfe von Raw-REST-Aufrufen auf die Microsoft Graph-API zugreifen.

In der folgenden Abbildung ist die App dargestellt, die Sie erstellen. 

![Die Web-App nach der Anmeldung mit der Schaltfläche „E-Mail senden“](./images/web-screenshot.png)


**Sie möchten keine App erstellen?** Verwenden Sie für einen schnellen Einstieg den [Schnellstart Microsoft Graph](https://graph.microsoft.io/en-us/getting-started).

Zum Herunterladen einer Version des Connect-Beispiels, das den Endpunkt Azure AD verwendet, siehe [Microsoft Graph Connect-Beispiel für Node.js](https://github.com/microsoftgraph/nodejs-connect-rest-sample/releases/tag/last_v1_auth).


## <a name="prerequisites"></a>Voraussetzungen

Für die ersten Schritte benötigen Sie: 

- Ein [Microsoft-Konto](https://www.outlook.com/) oder ein [Geschäfts- oder Schulkonto](http://dev.office.com/devprogram)
- [Node.js with npm](https://nodejs.org/en/download/) 
- Das [Microsoft Connect-Beispiel für Node.js](https://github.com/microsoftgraph/nodejs-connect-rest-sample). Für diese exemplarische Vorgehensweise verwenden Sie den Ordner **Startprojekt** in den Beispieldateien.

## <a name="register-the-application"></a>Registrieren der App
Registrieren Sie eine App im Microsoft App-Registrierungsportal. Dadurch werden die APP-ID und das Kennwort generiert, mit der bzw. dem Sie die App in Visual Studio konfigurieren.

1. Melden Sie sich beim [Microsoft-App-Registrierungsportal](https://apps.dev.microsoft.com/) entweder mit Ihrem persönlichen oder geschäftlichen Konto oder mit Ihrem Schulkonto an.

2. Klicken Sie auf **App hinzufügen**.

3. Geben Sie einen Namen für die App ein, und wählen Sie **Anwendung erstellen** aus. 
    
    Die Registrierungsseite wird angezeigt, und die Eigenschaften der App werden aufgeführt.

4. Kopieren Sie die Anwendungs-ID: Dies ist der eindeutige Bezeichner für Ihre App. 

5. Wählen Sie unter **Anwendungsgeheimnisse** die Option **Neues Kennwort generieren** aus. Kopieren Sie das Kennwort aus dem Dialogfeld **Neues Kennwort wurde generiert**.

    Sie werden die ID und das Kennwort (geheim) der Anwendung-verwenden, um die App zu konfigurieren. 

6. Wählen Sie unter **Plattformen** die Option **Plattform hinzufügen** > ** Web** aus.

7. Geben Sie *http://localhost:3000/login* als Umleitungs-URI ein. 

8. Wählen Sie **Speichern** aus.


## <a name="configure-the-project"></a>Konfigurieren des Projekts
1. Öffnen Sie den Ordner **Startprojekt** in den Beispieldateien.

1. Führen Sie den folgenden Befehl in einem Eingabeaufforderungsfenster im Stammverzeichnis des Startprojekts aus. Dadurch werden die Projekt-Abhängigkeiten installiert.

        npm install

1. Öffnen Sie in den Projektdateien authHelper.js.


1. Ersetzen Sie im Feld **Anmeldeinformationen** die Platzhalterwerte i **ENTER\_YOUR\_CLIENT\_ID** und **ENTER\_YOUR\_SECRET** durch die Werte, die Sie soeben kopiert haben.

  
## <a name="authenticate-the-user-and-get-an-access-token"></a>Authentifizierung des Benutzers und Abrufen eines Zugriffstokens
In diesem Schritt fügen Sie Code für die Anmeldung und die Token-Verwaltung ein. Doch zunächst werfen wir einen genaueren Blick auf den Ablauf der Authentifizierung.

Diese App verwendet den Authorization Code Grant-Datenfluss mit einer delegierten Benutzeridentität. Für eine Webanwendung erfordert der Ablauf die ID der Anwendung, das Geheimnis und den Umleitungs-URI aus der registrierten App. 

Der Authentifizierungsfluss kann in diese grundlegenden Schritte unterteilt werden:

1. Umleitung des Benutzers für die Authentifizierung und Zustimmung
2. Anfordern eines Autorisierungscodes
3. Einlösen des Autorisierungscodes für ein Zugriffstoken
4. Anfordern eines neuen Zugriffstokens mit dem Aktualisierungstoken, wenn das Zugriffstoken abläuft

Die App verwendet die Middleware [OAuth](https://www.npmjs.com/package/oauth) zum Authentifizieren und Beschaffen von Token. Sie verwendet die Middleware [Cookie Parser](https://www.npmjs.com/package/cookie-parser), um Tokeninformationen in Cookies zwischenzuspeichern. Der Code zum Speichern und Aufrufen von Tokeninformationen befindet sich im Controller index.js.
    
   >**Wichtig** Die einfache Handhabung von Authentifizierung und Token bei diesem Projekt dient lediglich zu Beispielzwecken. In einer Produktions-App sollten Sie die Authentifizierung robuster gestalten, unter anderem mit Validierung und sicherer Handhabung von Token.

Damit zurück zur App-Erstellung.

1. Ersetzen Sie in authHelper.js, die Funktion *GetTokenFromCode* durch den folgenden Code. Dadurch wird ein Zugriffstoken mithilfe eines Autorisierungscodes abgerufen.

        function getTokenFromCode(code, callback) {
            var OAuth2 = OAuth.OAuth2;
            var oauth2 = new OAuth2(
                credentials.client_id,
                credentials.client_secret,
                credentials.authority,
                credentials.authorize_endpoint,
                credentials.token_endpoint
            );

            oauth2.getOAuthAccessToken(
                code,
                {
                    grant_type: 'authorization_code',
                    redirect_uri: credentials.redirect_uri,
                    response_mode: 'form_post',
                    nonce: uuid.v4(),
                    state: 'abcd'
                },
                function (e, accessToken, refreshToken) {
                    callback(e, accessToken, refreshToken);
                }
            );
        }

1. Ersetzen Sie die Funktion **getTokenFromRefreshToken** durch den folgenden Code. Dadurch wird ein Zugriffstoken mithilfe eines Aktualisierungstokens abgerufen.

        function getTokenFromRefreshToken(refreshToken, callback) {
            var OAuth2 = OAuth.OAuth2;
            var oauth2 = new OAuth2(
                credentials.client_id,
                credentials.client_secret,
                credentials.authority,
                credentials.authorize_endpoint,
                credentials.token_endpoint
            );

            oauth2.getOAuthAccessToken(
                refreshToken,
                {
                    grant_type: 'refresh_token',
                    redirect_uri: credentials.redirect_uri,
                    response_mode: 'form_post',
                    nonce: uuid.v4(),
                    state: 'abcd'
                },
                function (e, accessToken) {
                    callback(e, accessToken);
                }
            );
        }

Sie können nun Code hinzufügen, um Microsoft Graph aufzurufen. 

## <a name="call-microsoft-graph"></a>Aufrufen von Microsoft Graph
Die App ruft Microsoft Graph auf, um Benutzerinformationen abzurufen und eine E-Mail-Nachricht im Auftrag des Benutzers zu senden. Diesen Aufrufe werden vom Controller index.js als Antwort auf Benutzeroberflächenereignissen initiiert.

1. Öffnen Sie requestUtil.js.

1. Ersetzen Sie die Funktion **getUserData** durch den folgenden Code. Dadurch wird die GET-Anforderung konfiguriert und an den Endpunkt */me* gesendet und die Antwort verarbeitet.

        function getUserData(accessToken, callback) {
            var options = {
                host: 'graph.microsoft.com',
                path: '/v1.0/me',
                method: 'GET',
                headers: {
                    'Content-Type': 'application/json',
                    Accept: 'application/json',
                    Authorization: 'Bearer ' + accessToken
                }
            };

            https.get(options, function (response) {
                var body = '';
                response.on('data', function (d) {
                    body += d;
                });
                response.on('end', function () {
                    var error;
                    if (response.statusCode === 200) {
                        callback(null, JSON.parse(body));
                    } else {
                        error = new Error();
                        error.code = response.statusCode;
                        error.message = response.statusMessage;
                        // The error body sometimes includes an empty space
                        // before the first character, remove it or it causes an error.
                        body = body.trim();
                        error.innerError = JSON.parse(body).error;
                        callback(error, null);
                    }
                });
            }).on('error', function (e) {
                callback(e, null);
            });
        }

1. Ersetzen Sie die Funktion **postSendMail** durch den folgenden Code. Dadurch wird die POST-Anforderung konfiguriert und an den Endpunkt */me/sendMail* gesendet und die Antwort verarbeitet.

        function postSendMail(accessToken, mailBody, callback) {
            var outHeaders = {
                'Content-Type': 'application/json',
                Authorization: 'Bearer ' + accessToken,
                'Content-Length': mailBody.length
            };
            var options = {
                host: 'graph.microsoft.com',
                path: '/v1.0/me/sendMail',
                method: 'POST',
                headers: outHeaders
            };

            // Set up the request
            var post = https.request(options, function (response) {
                var body = '';
                response.on('data', function (d) {
                    body += d;
                });
                response.on('end', function () {
                    var error;
                    if (response.statusCode === 202) {
                        callback(null);
                    } else {
                        error = new Error();
                        error.code = response.statusCode;
                        error.message = response.statusMessage;
                        // The error body sometimes includes an empty space
                        // before the first character, remove it or it causes an error.
                        body = body.trim();
                        error.innerError = JSON.parse(body).error;
                        // Note: If you receive a 500 - Internal Server Error
                        // while using a Microsoft account (outlook.com, hotmail.com or live.com),
                        // it's possible that your account has not been migrated to support this flow.
                        // Check the inner error object for code 'ErrorInternalServerTransientError'.
                        // You can try using a newly created Microsoft account or contact support.
                        callback(error);
                    }
                });
            });
            
            // write the outbound data to it
            post.write(mailBody);
            // we're done!
            post.end();

            post.on('error', function (e) {
                callback(e);
            });
        }
  
1. Öffnen Sie emailer.js.

1. Ersetzen Sie die Funktion **wrapEMail** durch den folgenden Code. Dadurch werden die Nutzdaten erstellt, die die zu sendende E-Mail-Nachricht darstellen.

        function wrapEmail(content, recipient) {
            var emailAsPayload = {
                Message: {
                    Subject: 'Welcome to Office 365 development with Node.js and the Office 365 Connect sample',
                    Body: {
                        ContentType: 'HTML',
                        Content: content
                    },
                    ToRecipients: [
                        {
                            EmailAddress: {
                                Address: recipient
                            }
                        }
                    ]
                },
                SaveToSentItems: true
            };
            return emailAsPayload;
        }

## <a name="run-the-app"></a>Ausführen der App

1. Führen Sie den folgenden Befehl in einem Eingabeaufforderungsfenster im Stammverzeichnis des Startprojekts aus.


        npm start

1. Navigieren Sie in einem Browser zu *http://localhost:3000*, und wählen Sie die Schaltfläche **Mit Office 365 verbinden** aus.

1. Melden Sie sich an, und erteilen Sie die erforderlichen Berechtigungen. 

1. Optional können Sie die E-Mail-Adresse des Empfängers bearbeiten. Klicken Sie dann auf die Schaltfläche **E-Mail senden**. Nachdem die E-Mail gesendet wurde, wird unter der Schaltfläche eine Erfolgsmeldung angezeigt. 

## <a name="next-steps"></a>Nächste Schritte
- Testen Sie die REST-API mithilfe des [Graph-Explorers](https://graph.microsoft.io/graph-explorer).
- Schauen Sie sich weitere [Node.js-Beispiele](https://github.com/search?utf8=%E2%9C%93&q=node+sample+user%3Amicrosoftgraph&type=Repositories&ref=searchresults) unter GitHub an.


## <a name="see-also"></a>Siehe auch
- [Azure AD v2.0-Protokolle](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
- [Azure AD v2.0-Tokens](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)