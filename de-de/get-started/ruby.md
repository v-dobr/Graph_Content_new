# <a name="get-started-with-microsoft-graph-in-a-ruby-on-rails-app"></a>Erste Schritte mit Microsoft Graph in einer Ruby on Rails-App

Dieser Artikel beschreibt die erforderlichen Aufgaben zum Abrufen eines Zugriffstokens vom Azure AD v2.0-Endpunkt und zum Aufrufen von Microsoft Graph. Sie werden durch die Erstellung des [Microsoft Graph Ruby on Rails Connect-Beispiels](https://github.com/microsoftgraph/ruby-connect-rest-sample) geführt und erhalten Informationen zu den Hauptkonzepten, die Sie zur Verwendung von Microsoft Graph implementieren. In diesem Artikel wird auch beschrieben, wie Sie mithilfe von direkten REST-Aufrufen auf Microsoft Graph zugreifen.

Zum Herunterladen einer Version des Connect-Beispiels, das den Endpunkt Azure AD verwendet, siehe [Microsoft Graph Ruby on Rails Connect-Beispiel](https://github.com/microsoftgraph/ruby-connect-rest-sample/tree/last_v1_auth).

In der folgenden Abbildung ist die App dargestellt, die Sie erstellen. 

![Screenshot des Microsoft Ruby on Rails Connect-Beispiels](./images/Microsoft-Graph-Ruby-Connect-UI.png)

**Sie möchten keine App erstellen?** Verwenden Sie für einen schnellen Einstieg den [Schnellstart Microsoft Graph](https://graph.microsoft.io/en-us/getting-started), oder laden Sie das [Ruby REST Connect-Beispiel](https://github.com/microsoftgraph/ruby-connect-rest-sample) herunter, auf dem dieser Artikel basiert.


## <a name="prerequisites"></a>Voraussetzungen

Für die ersten Schritte benötigen Sie: 

- Zum Ausführen des Beispiels ist Ruby 2.1 auf einem Entwicklungsserver erforderlich.
- Rails-Framework (das Beispiel wurde auf Rails 4.2 getestet).
- Bundler Dependency Manager.
- Rack-Webserveroberfläche für Ruby.
- Ein [Microsoft-Konto](https://www.outlook.com/) oder ein [Geschäfts- oder Schulkonto](http://dev.office.com/devprogram)
- Das Microsoft Graph Connect-Startprojekt für Ruby on Rails. Laden Sie das [Microsoft Graph Ruby on Rails Connect-Beispiel](https://github.com/microsoftgraph/ruby-connect-rest-sample) herunter. Das Startprojekt befindet sich im _Start_ordner.


## <a name="register-the-application"></a>Registrieren der App

Registrieren Sie eine App im Microsoft App-Registrierungsportal. Dadurch werden die ID und das Geheimnis der App generiert, mit der bzw. dem Sie die App für die Authentifizierung konfigurieren.

1. Melden Sie sich beim [Microsoft-App-Registrierungsportal](https://apps.dev.microsoft.com/) entweder mit Ihrem persönlichen oder geschäftlichen Konto oder mit Ihrem Schulkonto an.

2. Klicken Sie auf **App hinzufügen**.

3. Geben Sie einen Namen für die App ein, und wählen Sie **Anwendung erstellen** aus.

    Die Registrierungsseite wird angezeigt, und die Eigenschaften der App werden aufgeführt.

4. Kopieren Sie die Anwendungs-ID: Dies ist der eindeutige Bezeichner für Ihre App.

5. Wählen Sie unter **Anwendungsgeheimnisse** die Option **Neues Kennwort generieren** aus. Kopieren Sie das Anwendungsgeheimnis aus dem Dialogfeld **Neues Kennwort wurde generiert**.

    Sie werden die ID und das Geheimnis der Anwendung verwenden, um die App zu konfigurieren.

6. Wählen Sie unter **Plattformen** die Option **Plattform hinzufügen** > ** Web** aus.

7. Stellen Sie sicher, dass das Kontrollkästchen **Impliziten Fluss zulassen** aktiviert ist, und geben Sie *http://localhost:3000/auth/microsoft_v2_auth/callback* als Umleitungs-URI ein.

    Die Option „Impliziten Fluss zulassen“ ermöglicht den OpenID Connect-Hybridfluss. Während der Authentifizierung ermöglicht dies der App, sowohl Anmeldeinformationen (das id_token) als auch Artefakte (in diesem Fall ein Autorisierungscode) abzurufen, den die App zum Abrufen eines Zugriffstokens verwendet.

    Der Umleitungs-URI *Http://localhost:3000/autorisierende/microsoft_v2_auth/Rückruf* ist der Wert, den die OmniAuth Middleware entsprechend der Konfiguration verwendet, wenn sie die Authentifizierungsanforderung verarbeitet hat.

8. Wählen Sie **Speichern** aus.

## <a name="configure-the-project"></a>Konfigurieren des Projekts

1. Laden Sie das [Microsoft Graph Ruby on Rails Connect-Beispiel](https://github.com/microsoftgraph/ruby-connect-rest-sample) herunter, oder klonen Sie es. Öffnen Sie den _Start_ordner im Editor Ihrer Wahl.
1. Wenn Sie weder über einen Bundler noch über ein Rack verfügen, können Sie sie mithilfe des folgenden Befehls installieren.

    ```
    gem install bundler rack
    ```
2. Nehmen Sie in der Datei [config/environment.rb](config/environment.rb) die folgende Aktion vor.
    - Ersetzen Sie *ENTER_YOUR_CLIENT_ID* durch die Client-ID Ihrer registrierten Anwendung.
    - Ersetzen Sie *ENTER_YOUR_SECRET* durch den Schlüssel Ihrer registrierten Anwendung.

3. Installieren Sie die Rails-Anwendung und -Abhängigkeiten mit dem folgenden Befehl.

    ```
    bundle install
    ```

## <a name="authenticate-the-user-and-get-an-access-token"></a>Authentifizierung des Benutzers und Abrufen eines Zugriffstokens

Diese App verwendet den Authorization Code Grant-Datenfluss mit einer delegierten Benutzeridentität. Für eine Webanwendung erfordert der Ablauf die ID und das Geheimnis der Anwendung, sowie den Umleitungs-URI aus der registrierten App. 

Der Authentifizierungsfluss kann in diese grundlegenden Schritte unterteilt werden:

1. Umleitung des Benutzers für die Authentifizierung und Zustimmung
2. Anfordern eines Autorisierungscodes
3. Einlösen des Autorisierungscodes für ein Zugriffstoken

>Weitere Informationen zu diesem Autorisierungsfluss finden Sie unter [Webanwendung zu Web-API](https://azure.microsoft.com/en-us/documentation/articles/active-directory-authentication-scenarios/#web-application-to-web-api) und [Integrieren einer Microsoft-Identität und von Microsoft Graph in eine Webanwendung mit OpenID Connect](https://azure.microsoft.com/en-us/documentation/samples/active-directory-dotnet-webapp-openidconnect-v2/) in der Dokumentation zu Azure AD.

Wir verwenden einen Dreierstack [Rack](http://rack.github.io/) Middleware, damit die App sich gegenüber Microsoft Graph authentifizieren kann:

- [OmniAuth](https://rubygems.org/gems/omniauth), ein allgemeines Rack-Framework für die Multi-Provider-Authentifizierung.
- [Omniauth-oauth2](https://rubygems.org/gems/omniauth-oauth2), eine abstrakte OAuth2-Strategie für OmniAuth. 
- Omniauth-microsoft_v2_auth, eine OmniAuth-Strategie, die Omniauth-oauth2 anpasst, um konkret Authentifizierung gegenüber dem Azure AD v2.0-Endpunkt bereitzustellen. Dieses Projekt ist im Codebeispiel enthalten.

### <a name="specify-gem-dependencies-for-authentication"></a>Festlegen von Gem-Abhängigkeiten für die Authentifizierung

Heben Sie in Gemfile die Auskommentierung folgender Gems auf, um sie als Abhängigkeiten hinzuzufügen.

    ```
    gem 'omniauth'
    gem 'omniauth-oauth2'
    gem 'omniauth-microsoft_v2_auth', path: './omniauth-microsoft_v2_auth'
    ```

Beachten Sie, dass `omniauth-microsoft_v2_auth` im App-Projekt enthalten ist und ausgehend von dem angegebenen Pfad installiert werden wird. 

### <a name="configure-the-authentication-middleware"></a>Konfigurieren der Authentifizierungs-Middleware

Entfernen Sie in `config/initializers/omniauth-microsoft_v2_auth.rb` die Auskommentierung aus folgenden Zeilen:

    ```
    Rails.application.config.middleware.use OmniAuth::Builder do
      provider :microsoft_v2_auth,
      ENV['CLIENT_ID'],
      ENV['CLIENT_SECRET'],
      :scope => ENV['SCOPE']
    end
    ```
Dadurch wird die OmniAuth Middleware konfiguriert, und unter anderem werden die zu verwendende ID und das zu verwendende Geheimnis der App sowie die Bereiche festgelegt, die für den Benutzer anzufordern sind. Dies sind die Werte, die Sie zuvor in `config/environment.rb` angegeben haben.

### <a name="specify-routes-for-authentication"></a>Festlegen von Routen für die Authentifizierung

Jetzt müssen wir zwei Routen für den Authentifizierungsfluss festlegen. Die erste Route leitet die Authentifizierungsanforderung an die OmniAuth Middleware weiter, und die zweite nennt den Speicherort in der App, zu dem OmniAuth umleiten soll, nachdem die Authentifizierung erfolgt ist.

Entfernen Sie in `config/routes.rb` die Auskommentierung aus folgender Routenanweisung:

    get '/login', to: 'pages#login'

Dadurch werden Anmeldeanfragen an die `login`-Methode des Pages-Controllers geleitet, die wiederum die Anforderung an die Omniauth-microsoft_v2_auth Middleware weiterleitet.

    def login
        redirect_to '/auth/microsoft_v2_auth'
    end

Als Nächstes müssen wir angeben, wohin in der App OmniAuth umleiten soll, nachdem die Authentifizierung erfolgt ist. Entfernen Sie die Auskommentierung folgender Route.

    match '/auth/:provider/callback', to: 'pages#callback', via: [:get, :post]

Wenn die OmniAuth-Authentifizierung des Benutzers abgeschlossen ist, wird die bei der App-Registrierung angegebene Umleitungs-URL aufgerufen, in diesem Fall *http://localhost:3000/auth/microsoft_v2_auth/callback*. Das oben genannte Routenmuster ermittelt diese URL und leitet die Anforderung an die `callback`-Methode des Page-Controllers weiter.

### <a name="get-an-access-token"></a>Abrufen eines Zugriffstokens

Als Nächstes fügen wir den Code hinzu, der den Authentifizierungsprozess startet und das Zugriffstoken abruft, sobald der Benutzer sich erfolgreich angemeldet hat.

Sehen Sie sich `app/views/pages/index.html.erb` an, die Ansicht für das Stammverzeichnis der Website. Die Ansicht enthält eine einzelne Schaltfläche, die Benutzern die Anmeldung ermöglicht.

    <button class="ms-Button" onclick="window.location.href = '/login'">
        <span class="ms-Button-label"><%= t('connect_button') %></span>
    </button>

Wie oben gezeigt veranlasst die Login-Methode die Weiterleitung an die OmniAuth Middleware, die mit der App-ID und dem App-Geheimnis sowie den Bereichen konfiguriert ist, die für den Benutzer anzufordern sind. Nachdem der Benutzer erfolgreich authentifiziert worden ist, übergibt OmniAuth ein Hash mit dem Zugriffstoken und andere Benutzerinformationen an die App.

Nun fügen wir Code zur Behandlung des OmniAuth-Rückrufs und zum Abrufen von Informationen von diesem Hash hinzu. 

Ersetzen Sie in `app/controllers/pages_controller.rb` die leere `callback`-Methode durch den folgenden Code.

    ```
    def callback
        # Access the authentication hash for omniauth
        # and extract the auth token, user name, and email
        data = request.env['omniauth.auth']
    
        @email = data[:extra][:raw_info][:userPrincipalName]
        @name = data[:extra][:raw_info][:displayName]

        # Associate token/user values to the session
        session[:access_token] = data['credentials']['token']
        session[:name] = @name
        session[:email] = @email
        
        # Debug logging
        logger.info "Name: #{@name}"
        logger.info "Email: #{@email}"
        logger.info "[callback] - Access token: #{session[:access_token]}"
    end

    ```

Diese Methode ruft den Authentifizierungs-Hash ab und speichert dann das Zugriffstoken, den Benutzernamen und die E-Mail-Daten in der aktuellen Sitzung.

> **Hinweis:** Die einfache Handhabung von Authentifizierung und Token bei diesem Projekt dient nur zur Veranschaulichung. In einer Produktions-App würden Sie die Authentifizierung wahrscheinlich robuster gestalten, unter anderem mit sicherer Handhabung von Token und Tokenaktualisierung.

## <a name="call-microsoft-graph"></a>Aufrufen von Microsoft Graph

Sie können nun Code hinzufügen, um Microsoft Graph aufzurufen. 

Die von der `callback`-Methode gerenderte Ansicht (`app/views/pages/callback.html.erb`) beinhaltet ein einfaches Formular mit einer einzigen Schaltfläche. Das Formular übergibt an `send_mail` und enthält einen einzelnen Parameter, die E-Mail-Adresse des Empfängers.
    
    ``` 
    <form action="../../send_mail" method="post">
      <div class="ms-Grid-col ms-u-mdPush1 ms-u-md9 ms-u-lgPush1 ms-u-lg6">
        ...
            <div class="ms-TextField">
               <input class="ms-TextField-field" name="specified_email" value="<%= @email %>">
            </div>
            <button class="ms-Button">
            <span class="ms-Button-label"><i class="ms-Icon ms-Icon--mail" aria-hidden="true"></i><%= t('send_mail_button') %></span>
            </button> 
        ...
    ```

Ersetzen Sie in `app/controllers/pages_controller.rb` die leere `send_mail`-Methode durch den folgenden Code.

    ```
    def send_mail
        logger.debug "[send_mail] - Access token: #{session[:access_token]}"
        
        # Used in the template
        @name = session[:name]
        @email = params[:specified_email]
        @recipient = params[:specified_email]
        @mail_sent = false
        
        send_mail_endpoint = URI("#{GRAPH_RESOURCE}#{SENDMAIL_ENDPOINT}")
        content_type = CONTENT_TYPE
        http = Net::HTTP.new(send_mail_endpoint.host, send_mail_endpoint.port)
        http.use_ssl = true
        
        # If you want to use a sniffer tool, like Fiddler, to see the request
        # you might need to add this line to tell the engine not to verify the
        # certificate or you might see a "certificate verify failed" error
        # http.verify_mode = OpenSSL::SSL::VERIFY_NONE
        
        email_body = File.read('app/assets/MailTemplate.html')
        email_body.sub! '{given_name}', @name
        email_subject = t('email_subject')
        
        logger.debug email_body
    
        email_message = "{
            Message: {
            Subject: '#{email_subject}',
            Body: {
                ContentType: 'HTML',
                Content: '#{email_body}'
            },
            ToRecipients: [
                {
                    EmailAddress: {
                        Address: '#{@recipient}'
                    }
                }
            ]
            },
            SaveToSentItems: true
            }"
            
        response = http.post(
            SENDMAIL_ENDPOINT,
            email_message,
            'Authorization' => "Bearer #{session[:access_token]}",
            'Content-Type' => content_type
        )
        
        logger.debug "Code: #{response.code}"
        logger.debug "Message: #{response.message}"
        
        # The send mail endpoint returns a 202 - Accepted code on success
        if response.code == '202'
            @mail_sent = true
        else
            @mail_sent = false
            flash[:httpError] = "#{response.code} - #{response.message}"
        end
        
        render 'callback'
    end
    ```

Dieser Code erstellt die HTTP-Anforderung, formatiert die E-Mail und ruft dann Microsoft Graph zum Senden der E-Mail auf.

Um die E-Mail zu erstellen, zieht der Code den Benutzernamen aus dem Sitzungstoken und die E-Mail-Adresse des Empfängers aus den Parametern des Formulars heraus. Der Code liest dann den Textkörper der E-Mail aus einer Vorlage, die im Projekt enthalten ist, interpoliert den Benutzernamen und die E-Mail-Adresse und fügt den Text der E-Mail-Nachricht als Textkörper der HTTP-Anforderung an.

Zum Senden der E-Mail erstellt der Code die HTTP-Anforderung, fügt das Zugriffstoken als Autorisierungsheader hinzu und sendet dann die Anfrage an den Sendmail-Endpunkt.

Schließlich verwendet der Code, den zurückgegebenen HTTP-Antwortcode, um den Benutzer zu informieren, ob die E-Mail erfolgreich war oder nicht.

## <a name="run-the-app"></a>Ausführen der App

1. Installieren Sie die Rails-Anwendung und -Abhängigkeiten mit dem folgenden Befehl.

    ```
    bundle install
    ```
2. Geben Sie zum Starten der Rails-Anwendung den folgenden Befehl ein.

    ```
    rackup -p 3000
    ```
3. Navigieren Sie im Webbrowser zu `http://localhost:3000`.

## <a name="see-also"></a>Siehe auch
- Testen Sie die REST-API mithilfe des [Graph-Explorers](https://graph.microsoft.io/graph-explorer).
- Schauen Sie sich weitere [Microsoft Graph-Beispiele](https://github.com/microsoftgraph) unter GitHub an.


