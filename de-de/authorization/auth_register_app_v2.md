## <a name="register-your-microsoft-graph-application-with-the-azure-ad-v2.0-endpoint"></a>Registrieren Ihrer Microsoft Graph-Anwendung mit dem Azure AD v2.0-Endpunkt

Um den Azure AD v2.0-Endpunkt zu verwenden, müssen Sie Ihre App im [Microsoft App-Registrierungsportal](https://apps.dev.microsoft.com) (https://apps.dev.microsoft.com) registrieren. Durch die Registrierung Ihrer App wird die Identität Ihrer App beim Authentifizierungsanbieter geschaffen. Außerdem kann die App so ihre Identität angeben, wenn sie Authentifizierungsanfragen vom Benutzer einreicht. Dadurch werden die ID und das Geheimnis der App generiert, mit der bzw. dem Sie die App für die Authentifizierung konfigurieren.

> **Hinweis: ** In diesem Artikel wird die App-Registrierung mit dem Azure AD v2.0-Endpunkt beschrieben. Verwenden Sie zum [Registrieren Ihrer App bei Azure AD](app_authentication_azure_ad.md) das [Azure-Portal](https://aka.ms/aadapplist).
> 
> Beachten Sie also: Wenn Sie bereits Apps im Microsoft Azure-Verwaltungsportal registriert haben, werden diese nicht im App-Registrierungsportal nicht aufgeführt. Verwalten Sie diese App im Azure Management-Portal. 

1. Melden Sie sich beim [Microsoft-App-Registrierungsportal](https://apps.dev.microsoft.com/) entweder mit Ihrem persönlichen oder geschäftlichen Konto oder mit Ihrem Schulkonto an.

2. Klicken Sie auf **App hinzufügen**.

3. Geben Sie einen Namen für die App ein, und wählen Sie **Anwendung erstellen** aus.

    Die Registrierungsseite wird angezeigt, und die Eigenschaften der App werden aufgeführt.

4. Kopieren Sie die Anwendungs-ID: Dies ist der eindeutige Bezeichner für Ihre App.

    Sie werden die Anwendungs-ID verwenden, um die App zu konfigurieren.

5. Wählen Sie unter **Plattformen** **Plattform hinzufügen** und wählen Sie die passende Plattform für Ihre App aus:
    
    Für Client-Apps:
    1. Klicken Sie auf **Mobile Plattform**.

    2. Kopieren Sie die Werte für die Client-ID (App-ID) und den Umleitungs-URI in die Zwischenablage. Sie müssen diese Werte in die Beispiel-App eingeben.

        Die App-ID ist ein eindeutiger Bezeichner für Ihre App. Der Umleitungs-URI ist ein eindeutiger URI, der sicherstellt, dass an diesen URI gesendete Nachrichten nur an diese Anwendung gesendet werden. 

    Für Web-Apps:
    1. Klicken Sie auf **Web**.
    2. Wenn Sie den impliziten Grant-Typ oder den OpenID Connect-Hybridfluss verwenden, stellen Sie sicher, dass das Kontrollkästchen „Impliziten Fluss zulassen“ aktiviert ist. 
        
        Die Option „Impliziten Fluss zulassen“ ermöglicht den OpenID Connect-Hybridfluss. Während der Authentifizierung ermöglicht dies der App, sowohl Anmeldeinformationen (das id_token) als auch Artefakte (in diesem Fall ein Autorisierungscode) abzurufen, den die App zum Abrufen eines Zugriffstokens verwendet.


    3. Festlegen einer Umleitungs-URI
        
        Die Umleitungs-URI ist der Standort in Ihrer App, den der Azure AD v2.0-Endpunkt aufruft, wenn er die Authentifizierungsanforderung verarbeitet hat.
    4. Wählen Sie unter **Anwendungsgeheimnisse** die Option **Neues Kennwort generieren** aus. Kopieren Sie das Anwendungsgeheimnis aus dem Dialogfeld **Neues Kennwort wurde generiert**.
        
        Sie werden das Anwendungsgeheimnis verwenden, um die App zu konfigurieren.
    
6. Wählen Sie **Speichern** aus.

## <a name="see-also"></a>Siehe auch

[App-Authentifizierung mit Microsoft Graph](auth_overview.md)
