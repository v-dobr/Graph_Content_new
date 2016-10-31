# <a name="get-started-with-microsoft-graph-in-an-angularjs-app"></a>Erste Schritte mit Microsoft Graph in einer Angular JS-App

Dieser Artikel beschreibt die erforderlichen Aufgaben zum Abrufen eines Zugriffstokens vom Azure AD v2.0-Endpunkt und zum Aufrufen von Microsoft Graph. Sie werden durch die Erstellung des [Microsoft Connect-Beispiels für Angular JS](https://github.com/microsoftgraph/angular-connect-rest-sample) geführt und erhalten Informationen zu den Hauptkonzepten, die Sie zur Verwendung von Microsoft Graph implementieren. In diesem Artikel wird beschrieben, wie Sie mithilfe von Raw-REST-Aufrufen auf die Microsoft Graph-API zugreifen.

In der folgenden Abbildung ist die App dargestellt, die Sie erstellen. 

![Die Web-App nach der Anmeldung mit der Schaltfläche „E-Mail senden“](./images/angular-connect-sample.png)


**Sie möchten keine App erstellen?** Verwenden Sie für einen schnellen Einstieg den [Schnellstart Microsoft Graph](https://graph.microsoft.io/en-us/getting-started).

Zum Herunterladen einer Version des Connect-Beispiels, das den Endpunkt Azure AD verwendet, siehe [Microsoft Graph Connect-Beispiel für Angular JS](https://github.com/microsoftgraph/angular-connect-rest-sample/releases/tag/last_v1_auth).


## <a name="prerequisites"></a>Voraussetzungen

Für die ersten Schritte benötigen Sie: 

- Ein [Microsoft-Konto](https://www.outlook.com/) oder ein [Geschäfts- oder Schulkonto](http://dev.office.com/devprogram)
- [Node.js with npm](https://nodejs.org/en/download/)
- [Bower](https://bower.io)
- Das [Microsoft Connect-Beispiel für AngularJS](https://github.com/microsoftgraph/angular-connect-rest-sample). Für diese exemplarische Vorgehensweise verwenden Sie den Ordner **Startprojekt** in den Beispieldateien.

## <a name="register-the-application"></a>Registrieren der App
Registrieren Sie eine App im Microsoft App-Registrierungsportal. Dadurch werden die APP-ID und das Kennwort generiert, mit der bzw. dem Sie die App in Visual Studio konfigurieren.

1. Melden Sie sich beim [Microsoft-App-Registrierungsportal](https://apps.dev.microsoft.com/) entweder mit Ihrem persönlichen oder geschäftlichen Konto oder mit Ihrem Schulkonto an.

2. Klicken Sie auf **App hinzufügen**.

3. Geben Sie einen Namen für die App ein, und wählen Sie **Anwendung erstellen** aus. 
    
    Die Registrierungsseite wird angezeigt, und die Eigenschaften der App werden aufgeführt.

4. Kopieren Sie die Anwendungs-ID: Hierbei handelt es sich um einen eindeutigen Bezeichner, die Sie für die Konfiguration der App verwenden werden.

5. Wählen Sie unter **Plattformen** die Option **Plattform hinzufügen** > **Web** aus.

7. Stellen Sie sicher, dass das Kontrollkästchen **Impliziten Fluss zulassen** aktiviert ist, und geben Sie *http://localhost:8080/login* als Umleitungs-URI ein. 

8. Wählen Sie **Speichern** aus.


## <a name="configure-the-project"></a>Konfigurieren des Projekts
1. Öffnen Sie den Ordner **Startprojekt** in den Beispieldateien.
2. Führen Sie die folgenden Befehle in einem Eingabeaufforderungsfenster im Stammverzeichnis des Startprojekts aus. Dadurch werden die Projekt-Abhängigkeiten installiert.

    npm install  bower install hello

3. Öffnen Sie in den Startprojektdateien im Ordner **Öffentlich/Skripts** config.js.
4. Ersetzen Sie im Feld **Client-ID** den Platzhalterwert **ENTER_YOUR_CLIENT_ID** durch die Anwendungs-ID, die Sie gerade kopiert haben..

  
## <a name="authenticate-the-user-and-get-an-access-token"></a>Authentifizierung des Benutzers und Abrufen eines Zugriffstokens
In diesem Schritt fügen Sie Code für die Anmeldung und den Token-Abruf ein. Doch zunächst werfen wir einen genaueren Blick auf den Ablauf der Authentifizierung.

Diese Einzelseiten-Anwendung verwendet eine sehr grundlegende Implementierung des impliziten Grant-Flusses, für die Anwendungs-ID und Umleitungs-URI der registrierten App erforderlich sind. 

Der Authentifizierungsfluss kann in diese grundlegenden Schritte unterteilt werden:

1. Umleiten des Benutzers zur Authentifizierung und Zustimmung.
2. Abrufen eines Zugriffstokens

Die App verwendet die clientseitige Bibliothek [HelloJS](https://adodson.com/hello.js), um Tokens zu authentifizieren und abzurufen. Die App speichert das Zugriffstoken im lokalen Speicher.
    
   >**Wichtig** Die einfache Handhabung von Authentifizierung und Token bei diesem Projekt dient lediglich zu Beispielzwecken. In einer Produktions-App sollten Sie die Authentifizierung robuster gestalten, unter anderem mit Validierung und sicherer Handhabung von Token.

Damit zurück zur App-Erstellung.

1. Öffnen Sie aad.js und fügen Sie den folgenden Code hinzu. Dadurch wird die Kommunikation mit dem Azure AD-Authentifizierungsanbieter konfiguriert und ein Hörer hinzugefügt, der die Authentifizierungsantwort speicher, die das Zugriffstoken enthält. (Die Skriptverweise zu Hellojs wurden bereits zur Ansicht index.html hinzugefügt.)

```
hello.init({
      
  aad: {
    name: 'Azure Active Directory', 
    oauth: {
      version: 2,
      auth: 'https://login.microsoftonline.com/common/oauth2/v2.0/authorize',
      grant: 'https://login.microsoftonline.com/common/oauth2/v2.0/token'
    },
    scope_delim: ' ',

    // Don't even try submitting via form.
    // This means no POST operations in <=IE9
    form: false
  }
});

hello.on('auth.login', function (auth) {

    // save the auth info into localStorage
    localStorage.auth = angular.toJson(auth.authResponse);
});
```

2. Ersetzen Sie in graphHelper.js *//Authentifizierungsanfrage initialisieren* durch den folgenden Code. Dadurch werden Parameter für die Authentifizierungsanfrage festgelegt.

```
// Initialize the auth request.
hello.init( {
  aad: clientId // from public/scripts/config.js
  }, {
  redirect_uri: redirectUrl,
  scope: graphScopes
});
```

3. Ersetzen Sie *//Den Benutzer anmelden und abmelden* durch den folgenden Code. Die Funktion **Anmelden** verwendet HelloJS, um Tokeninformationen zu erhalten. Der Zuhörer in aad.js speicher diese Informationen, einschließlich dem Zugriffstoken, im lokalen Speicher.

```
// Sign in and sign out the user.
login: function login() {
  hello('aad').login({
    display: 'page',
    state: 'abcd'
  });
},
logout: function logout() {
  hello('aad').logout();
  delete localStorage.auth;
  delete localStorage.user;
},
```

Sie können nun Code hinzufügen, um Microsoft Graph aufzurufen. 

## <a name="call-microsoft-graph"></a>Aufrufen von Microsoft Graph
Die App ruft Microsoft Graph auf, um Benutzerinformationen abzurufen und eine E-Mail-Nachricht im Auftrag des Benutzers zu senden. Diese Anrufe werden vom Maincontroller als Antwort auf UI-Ereignisse initiiert.

1. Ersetzen Sie in graphHelper.js *//Profil des aktuellen Benutzers abrufen* durch folgenden Code. Dadurch wird die GET-Anforderung konfiguriert und an den Endpunkt */me* gesendet und die Antwort verarbeitet.

```
// Get the profile of the current user.
  me: function me() {
    return $http.get('https://graph.microsoft.com/v1.0/me');
},
```
  
2. Ersetzen Sie *Eine E-Mail im Namen des folgenden Benutzers senden* durch folgenden Code. Dadurch wird die POST-Anforderung konfiguriert und an den Endpunkt */me/sendMail* gesendet und die Antwort verarbeitet.

```
// Send an email on behalf of the current user.
sendMail: function sendMail(email) {
  return $http.post('https://graph.microsoft.com/v1.0/me/sendMail', { 'message' : email, 'saveToSentItems': true });        
}
```

3. Öffnen Sie im Ordner **Öffentlich/Controller** mainController.js.

4. Ersetzen Sie *//Standard-Header und Benutzereigenschaften festlegen* durch folgenden Code. Dadurch wird das Zugriffstoken zu der HTTP-Anforderung hinzugefügt, das Profil des aktuellen Benutzers von **GraphHelper.me** abgerufen und die Antwort verarbeitet.

```
// Set the default headers and user properties.
function processAuth() {
  let auth = angular.fromJson(localStorage.auth); 

  // Check token expiry. If the token is valid for another 5 minutes, we'll use it.       
  let expiration = new Date();
  expiration.setTime((auth.expires - 300) * 1000); 
  if (expiration > new Date()) {

    // Add the required Authorization header with bearer token.
    $http.defaults.headers.common.Authorization = 'Bearer ' + auth.access_token;
        
    // This header has been added to identify our sample in the Microsoft Graph service. If extracting this code for your project please remove.
    $http.defaults.headers.common.SampleID = 'angular-connect-rest-starter';
        
    if (localStorage.getItem('user') === null) {

      // Get the profile of the current user.
      GraphHelper.me().then(function(response) {

        // Save the user to localStorage.
        let user =response.data;
        localStorage.setItem('user', angular.toJson(user));

        vm.displayName = user.displayName;
        vm.emailAddress = user.mail || user.userPrincipalName;
      });
   } else {
     let user = angular.fromJson(localStorage.user);

     vm.displayName = user.displayName;
     vm.emailAddress = user.mail || user.userPrincipalName;
    }
  }
} 
```

5. Ersetzen Sie *//Eine E-Mail im Namen des aktuellen Benutzers* senden durch folgenden Code. Dadurch wird die E-Mail-Nachricht erstellt , **GraphHelper.sendMail** aufgerufen und die Antwort verarbeitet.

```
// Send an email on behalf of the current user.
function sendMail() {

  // Check token expiry. If the token is valid for another 5 minutes, we'll use it.
  let auth = angular.fromJson(localStorage.auth);
  let expiration = new Date();
  expiration.setTime((auth.expires - 300) * 1000);
  if (expiration > new Date()) {
  
    // Build the HTTP request payload (the Message object).
    var email = {
        Subject: 'Welcome to Microsoft Graph development with AngularJS and the Microsoft Graph Connect sample',
        Body: {
            ContentType: 'HTML',
            Content: getEmailContent()
        },
        ToRecipients: [
            {
                EmailAddress: {
                    Address: vm.emailAddress
                }
            }
        ]
    };

    // Save email address so it doesn't get lost with two way data binding.
    vm.emailAddressSent = vm.emailAddress;

    GraphHelper.sendMail(email)
        .then(function (response) {
            $log.debug('HTTP request to the Microsoft Graph API returned successfully.', response);
            response.status === 202 ? vm.requestSuccess = true : vm.requestSuccess = false;
            vm.requestFinished = true;
        }, function (error) {
            $log.error('HTTP request to the Microsoft Graph API failed.');
            vm.requestSuccess = false;
            vm.requestFinished = true;
        });
    } else {

    // If the token is expired, this sample just redirects the user to sign in.
    GraphHelper.login();
    }
};

// Get the HTMl for the email to send.
function getEmailContent() {
  return "<html><head> <meta http-equiv=\'Content-Type\' content=\'text/html; charset=us-ascii\'> <title></title> </head><body style=\'font-family:calibri\'> <p>Congratulations " + vm.displayName + ",</p> <p>This is a message from the Microsoft Graph Connect sample. You are well on your way to incorporating Microsoft Graph endpoints in your apps. </p> <h3>What&#8217;s next?</h3><ul><li>Check out <a href='https://graph.microsoft.io' target='_blank'>graph.microsoft.io</a> to start building Microsoft Graph apps today with all the latest tools, templates, and guidance to get started quickly.</li><li>Use the <a href='https://graph.microsoft.io/graph-explorer' target='_blank'>Graph explorer</a> to explore the rest of the APIs and start your testing.</li><li>Browse other <a href='https://github.com/microsoftgraph/' target='_blank'>samples on GitHub</a> to see more of the APIs in action.</li></ul> <h3>Give us feedback</h3> <ul><li>If you have any trouble running this sample, please <a href='https://github.com/microsoftgraph/angular-connect-rest-sample/issues' target='_blank'>log an issue</a>.</li><li>For general questions about the Microsoft Graph API, post to <a href='https://stackoverflow.com/questions/tagged/microsoftgraph?sort=newest' target='blank'>Stack Overflow</a>. Make sure that your questions or comments are tagged with [microsoftgraph].</li></ul><p>Thanks and happy coding!<br>Your Microsoft Graph samples development team</p> <div style=\'text-align:center; font-family:calibri\'> <table style=\'width:100%; font-family:calibri\'> <tbody> <tr> <td><a href=\'https://github.com/microsoftgraph/angular-connect-rest-sample\'>See on GitHub</a> </td> <td><a href=\'https://officespdev.uservoice.com/\'>Suggest on UserVoice</a> </td> <td><a href=\'https://twitter.com/share?text=I%20just%20started%20developing%20%23Angular%20apps%20using%20the%20%23MicrosoftGraph%20Connect%20sample!%20&url=https://github.com/microsoftgraph/angular-connect-rest-sample\'>Share on Twitter</a> </td> </tr> </tbody> </table> </div>  </body> </html>";
};
```

6. Speichern Sie all Ihre Änderungen.

## <a name="run-the-app"></a>Ausführen der App

1. Führen Sie den folgenden Befehl in einem Eingabeaufforderungsfenster im Stammverzeichnis des Startprojekts aus.

```
npm start
```

2. Navigieren Sie in einem Browser zu *http://localhost:8080* und wählen Sie die Schaltfläche **Verbinden** aus.

3. Melden Sie sich an und erteilen Sie die erforderlichen Berechtigungen. 

4. Optional können Sie die E-Mail-Adresse des Empfängers bearbeiten. Klicken Sie dann auf die Schaltfläche **E-Mail senden**. Nachdem die E-Mail gesendet wurde, wird unter der Schaltfläche eine Erfolgsmeldung angezeigt. 

## <a name="next-steps"></a>Nächste Schritte
- Testen Sie die REST-API mithilfe des [Graph-Explorers](https://graph.microsoft.io/graph-explorer).
- Weitere [AngularJS-Proben](https://github.com/search?utf8=%E2%9C%93&q=angular+sample+user%3Amicrosoftgraph&type=Repositories&ref=searchresults) finden Sie auf GitHub.


## <a name="see-also"></a>Siehe auch
- [Azure AD v2.0-Protokolle](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
- [Azure AD v2.0-Tokens](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)