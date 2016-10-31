# <a name="get-started-with-microsoft-graph-in-a-universal-windows-10-app"></a>Erste Schritte mit Microsoft Graph in einer universellen Windows 10-App

> **Sie erstellen Apps für Unternehmenskunden?** Ihre App funktioniert möglicherweise nicht, wenn Ihr Unternehmenskunde Enterprise Mobility-Sicherheitsfunktionen wie <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-conditional-access-device-policies/" target="_newtab">bedingten Gerätezugriff</a> aktiviert. In diesem Fall treten bei Ihren Kunden möglicherweise Fehler auf. 

> Zur Unterstützung **aller Unternehmenskunden** über **alle Unternehmensszenarien** hinweg müssen Sie den Azure AD-Endpunkt verwenden und Ihr Apps mithilfe des [Azure-Verwaltungsportals](https://aka.ms/aadapplist) verwalten. Weitere Informationen finden Sie unter [Entscheiden zwischen dem Azure AD- und dem Azure AD v2.0-Endpunkt](../authorization/auth_overview.md#deciding-between-azure-ad-and-the-v2-authentication-endpoint).

Dieser Artikel beschreibt die erforderlichen Aufgaben zum Abrufen eines Zugriffstokens vom [Azure AD v2.0-Endpunkt](https://graph.microsoft.io/en-us/docs/authorization/converged_auth) und zum Aufrufen von Microsoft Graph. Er führt Sie durch den Code im [Microsoft Graph Connect-Beispiel für UWP (REST)](https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample) und im [Microsoft Graph Connect-Beispiel für UWP (Bibliothek)](https://github.com/microsoftgraph/uwp-csharp-connect-sample) und erläutert die wichtigsten Konzepte, die in einer App implementiert werden müssen, die Microsoft Graph verwendet. In diesem Artikel wird auch beschrieben, wie Sie mit Raw-REST-Aufrufen und der [Microsoft Graph Clientbibliothek](http://www.nuget.org/packages/Microsoft.Graph/) auf Microsoft Graph zugreifen.

Sie können beide Versionen der App, die Sie in dieser Anleitung erstellen, aus diesen GitHub Repositories herunterladen:

* [Microsoft Graph Connect-Beispiel für UWP (REST)](https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample)
* [Microsoft Graph Connect-Beispiel für UWP (Bibliothek)](https://github.com/microsoftgraph/uwp-csharp-connect-sample)

**Sie möchten keine App erstellen?** Verwenden Sie für einen schnellen Einstieg den [Schnellstart Microsoft Graph](https://graph.microsoft.io/en-us/getting-started).

## <a name="sample-user-interface"></a>Beispielbenutzeroberfläche

Das Beispiel enthält eine sehr einfache Benutzeroberfläche, die aus einer oberen Befehlsleiste, einer **Schaltfläche zum Herstellen einer Verbindung**, einer Schaltfläche zum **Senden von E-Mails** und aus einem Textfeld besteht, in das die E-Mail-Adresse des angemeldeten Benutzers automatisch eingetragen wird, die jedoch bearbeitet werden kann.

Die Schaltfläche zum **Senden von E-Mails** ist deaktiviert, wenn der Benutzer keine Verbindung hergestellt hat:

![Bildschirm mit aktivierter Schaltfläche zum Verbinden und deaktivierter Schaltfläche zum Senden von E-Mails](images/SignedOut.png)

Die obere Befehlsleiste enthält eine Schaltfläche zum Trennen der Verbindung, wenn der Benutzer eine Verbindung hergestellt hat:

![Bildschirm mit der E-Mail-Adresse des verbundenen Benutzers und aktivierter Schaltfläche zum Senden von E-Mails](images/SignedIn.png)

Die gesamten Zeichenfolgen der Beispielbenutzeroberfläche werden in der Datei „Resources.resw“ innerhalb des Objektordners gespeichert.

## <a name="prerequisites"></a>Voraussetzungen

Für die ersten Schritte benötigen Sie: 

- Ein [Microsoft-Konto](https://www.outlook.com/) oder ein [Geschäfts- oder Schulkonto](http://dev.office.com/devprogram)
- Visual Studio 2015 
- Entweder das [Microsoft Graph Startprojekt für UWP (Bibliothek)](https://github.com/microsoftgraph/uwp-csharp-connect-sample/tree/master/starter) oder das [Microsoft Graph Startprojekt für UWP (REST)](https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample/tree/master/starter). Beide Vorlagen enthalten leere Klassen, denen Sie Code hinzufügen müssen. Außerdem enthalten sie Ressourcenzeichenfolgen. Um eines oder beide der folgenden Projekte abzurufen, müssen Sie das [Microsoft Graph Connect-Beispiel für UWP (Bibliothek)](https://github.com/microsoftgraph/uwp-csharp-connect-sample) und/oder das [Microsoft Graph Connect-Beispiel für UWP (REST)](https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample) klonen oder herunterladen. Dann öffnen Sie die Projektmappe innerhalb des **Start**ordners.


## <a name="register-the-app"></a>Registrieren der App
 
1. Melden Sie sich beim [App-Registrierungsportal](https://apps.dev.microsoft.com/) entweder mit Ihrem persönlichen oder geschäftlichen Konto oder mit Ihrem Schulkonto an.
2. Klicken Sie auf **App hinzufügen**.
3. Geben Sie einen Namen für die App ein, und wählen Sie **Anwendung erstellen** aus.
    
    Die Registrierungsseite wird angezeigt, und die Eigenschaften der App werden aufgeführt.
 
4. Wählen Sie unter **Plattformen** die Option **Plattform hinzufügen** aus.
5. Klicken Sie auf **Mobile Plattform**.
6. Kopieren Sie die Werte für die Client-ID (App-ID) und den Umleitungs-URI in die Zwischenablage. Sie müssen diese Werte in die Beispiel-App eingeben.

    Die App-ID ist ein eindeutiger Bezeichner für Ihre App. Der Umleitungs-URI ist ein eindeutiger, von Windows 10 für jede Anwendung bereitgestellter URI, der sicherstellt, dass an diesen URI gesendete Nachrichten nur an diese Anwendung gesendet werden. 

7. Klicken Sie auf **Speichern**.

## <a name="configure-the-project"></a>Konfigurieren des Projekts

1. Öffnen Sie die Projektmappendatei für das Startprojekt in Visual Studio.
2. Öffnen Sie die Datei **App.xaml** des Projekts, und suchen Sie den Knoten `Application.Resources`. Ersetzen Sie die Platzhalter der Anwendungs-ID und des Umleitungs-URI durch die entsprechenden Werte der App, die Sie registriert haben.


```xml
    <Application.Resources>
        <!-- Add your Client Id here. -->
        <x:String x:Key="ida:ClientID">ENTER_YOUR_CLIENT_ID</x:String>
        <!-- Add your Redirect URI here. -->
        <x:String x:Key="ida:ReturnUrl">ENTER_YOUR_REDIRECT_URI</x:String>
    </Application.Resources>
```

## <a name="install-the-microsoft-authentication-library-(msal)"></a>Installieren der Microsoft Authentication Library (MSAL)

Die [Microsoft Authentifizierungsbibliothek](https://www.nuget.org/packages/Microsoft.Identity.Client) enthält Klassen und Methoden, die das Authentifizieren von Benutzern über den Azure AD v2.0-Endpunkt vereinfachen.

1. Klicken Sie im Projektmappen-Explorer mit der rechten Maustaste auf den Projektnamen, und wählen Sie **NuGet-Pakete verwalten...** aus.
2. Klicken Sie auf „Durchsuchen“, und suchen Sie nach Microsoft.Identity.Client.
3. Wählen Sie die neueste Version der Microsoft Authentifizierungsbibliothek, und klicken Sie auf **Installieren**.

## <a name="install-the-microsoft-graph-client-library"></a>Installieren der Microsoft Graph-Clientbibliothek

> **Hinweis** Wenn Sie Raw-REST-Aufrufe verwenden, um auf Microsoft Graph zuzugreifen, können Sie diesen Abschnitt überspringen.

1. Klicken Sie im Projektmappen-Explorer mit der rechten Maustaste auf den Projektnamen, und wählen Sie **NuGet-Pakete verwalten...** aus.
2. Klicken Sie auf „Durchsuchen“, und suchen Sie nach Microsoft.Graph.
3. Wählen Sie die neueste Version der Microsoft Graph Clientbibliothek, und klicken Sie auf **Installieren**.

## <a name="create-the-authenticationhelper.cs-class"></a>Erstellen der Klasse AuthenticationHelper.cs

Öffnen Sie die Datei AuthenticationHelper.cs im Startprojekt. Diese Datei wird den gesamten Authentifizierungscode sowie zusätzliche Logik enthalten, die Benutzerinformationen speichert und die Authentifizierung nur dann erzwingt, wenn der Benutzer die Verbindung zu der App getrennt hat. Diese Klasse enthält mindestens zwei Methoden: `GetTokenForUserAsync` und `Signout`. Wenn Sie die Microsoft Graph Clientbibliothek verwenden, müssen Sie eine dritte Methode hinzufügen: `GetAuthenticatedClient`.

Die Methode ``GetTokenHelperAsync`` wird ausgeführt, wenn der Benutzer sich authentifiziert, und anschließend jedes Mal, wenn die App Microsoft Graph aufruft.

**Verwenden von Deklarationen**

***Clientbibliothek-Version***

Wenn Sie die Microsoft Graph Clientbibliothek verwenden, benötigen Sie diese Deklarationen:

```c#
using System;
using System.Diagnostics;
using System.Net.Http.Headers;
using System.Threading.Tasks;
using Microsoft.Graph;
using Microsoft.Identity.Client;
```

***REST-Version***

Wenn Sie für den Microsoft Graph Zugriff Raw-REST-Anrufe verwenden, benötigen Sie diese `using`-Deklarationen in der Klasse AuthenticationHelper:

```c#
using System;
using System.Threading.Tasks;
using Windows.Storage;
using Microsoft.Identity.Client;
```

**Felder der Klasse**

Beide Versionen der AuthenticationHelper.Klasse benötigen diese Felder:

```c#
// The Client ID is used by the application to uniquely identify itself to the Azure AD v2.0 endpoint.
static string clientId = App.Current.Resources["ida:ClientID"].ToString();
public static string[] Scopes = { "User.Read", "Mail.Send" };
public static PublicClientApplication IdentityClientApp = new PublicClientApplication(clientId);
public static string TokenForUser = null;
public static DateTimeOffset Expiration;
```

Beachten Sie, dass beide Versionen die MSAL-Klasse `PublicClientApplication` zum Authentifizieren des Benutzers verwenden. Im Feld `Scopes` sind die Microsoft Graph Berechtigungsbereiche gespeichert, die die App anfordern muss, wenn der Benutzer sich authentifiziert. 

***Clientbibliothek-Version***

Wenn Sie die Clientbibliothek verwenden, speichern Sie den `GraphServicesClient` als ein Feld, damit Sie ihn nur einmal zu erstellen brauchen:

```c#
private static GraphServiceClient graphClient = null;
```

***REST-Version***

Wenn Sie REST-Aufrufe verwenden, müssen Sie einige Werte in den Roaming-Einstellungen der App speichern, da Sie Informationen über den Benutzer speichern müssen. (Die Clientbibliothek stellt diese Informationen in der anderen Version bereit.)

```c#
public static ApplicationDataContainer _settings = ApplicationData.Current.RoamingSettings;
```

**GetTokenForUserAsync**

***Clientbibliothek-Version***

Die Methode `GetTokenForUserAsync` verwendet die PublicClientApplicationClass und die ClientId-Einstellung, um ein Zugriffstoken für den Benutzer abzurufen. Wenn der Benutzer sich nicht bereits authentifiziert hat, wird die Authentifizierungsbenutzeroberfläche gestartet. Dies ist die zu verwendende Version der Methode, wenn Sie die Clientbibliothek verwenden.

```c#
        public static async Task<string> GetTokenForUserAsync()
        {
            AuthenticationResult authResult;
            try
            {
                authResult = await IdentityClientApp.AcquireTokenSilentAsync(Scopes);
                TokenForUser = authResult.Token;
            }

            catch (Exception)
            {
                if (TokenForUser == null || Expiration <= DateTimeOffset.UtcNow.AddMinutes(5))
                {
                    authResult = await IdentityClientApp.AcquireTokenAsync(Scopes);

                    TokenForUser = authResult.Token;
                    Expiration = authResult.ExpiresOn;
                }
            }

            return TokenForUser;
        }
```

***REST-Version***

Die REST-Version der Methode `GetTokenForUserAsync` veranlasst einiges mehr, da sie auch einige Informationen über den Benutzer speichern muss.

```c#
        public static async Task<string> GetTokenForUserAsync()
        {
            AuthenticationResult authResult;
            try
            {
                authResult = await IdentityClientApp.AcquireTokenSilentAsync(Scopes);
                TokenForUser = authResult.Token;
                // save user ID in local storage
                _settings.Values["userID"] = authResult.User.UniqueId;
                _settings.Values["userEmail"] = authResult.User.DisplayableId;
                _settings.Values["userName"] = authResult.User.Name;
            }

            catch (Exception)
            {
                if (TokenForUser == null || Expiration <= DateTimeOffset.UtcNow.AddMinutes(5))
                {
                    authResult = await IdentityClientApp.AcquireTokenAsync(Scopes);

                    TokenForUser = authResult.Token;
                    Expiration = authResult.ExpiresOn;

                    // save user ID in local storage
                    _settings.Values["userID"] = authResult.User.UniqueId;
                    _settings.Values["userEmail"] = authResult.User.DisplayableId;
                    _settings.Values["userName"] = authResult.User.Name;
                }
            }

            return TokenForUser;
        }
```

**Abmeldung**

Die Methode `Signout` in beiden Versionen meldet alle Benutzer ab, die über die `PublicClientApplication` angemeldet sind (nur einen Benutzer, in diesem Fall), und hebt den Wert `TokenForUser` auf. Die Version, die die Clientbibliothek verwendet, hebt den gespeicherten `GraphServicesClient` ebenfalls auf, und die REST-Version hebt die Werte auf, die in den Roaming-Einstellungen der App gespeichert sind.

***Clientbibliothek-Version***

Dies ist die Clientbibliothek-Version der Methode `Signout`.

```c#
        public static void SignOut()
        {
            foreach (var user in IdentityClientApp.Users)
            {
                user.SignOut();
            }
            graphClient = null;
            TokenForUser = null;

        }
``` 

***REST-Version***

Dies ist die REST-Version der Methode `Signout`.

```c#
        public static void SignOut()
        {
            foreach (var user in IdentityClientApp.Users)
            {
                user.SignOut();
            }

            TokenForUser = null;

            //Clear stored values from last authentication.
            _settings.Values["userID"] = null;
            _settings.Values["userEmail"] = null;
            _settings.Values["userName"] = null;

        }
```

**GetAuthenticatedClient (nur Clientbibliothek)**

Wenn Sie die Clientbibliothek verwenden, benötigen Sie eine Methode zum Erstellen eines `GraphServicesClient`. Diese Methode erstellt einen Client, der die Methode `GetTokenForUserAsync` bei jedem Aufruf von Microsoft Graph über den Client verwendet.

```c#
        public static GraphServiceClient GetAuthenticatedClient()
        {
            if (graphClient == null)
            {
                // Create Microsoft Graph client.
                try
                {
                    graphClient = new GraphServiceClient(
                        "https://graph.microsoft.com/v1.0",
                        new DelegateAuthenticationProvider(
                            async (requestMessage) =>
                            {
                                var token = await GetTokenForUserAsync();
                                requestMessage.Headers.Authorization = new AuthenticationHeaderValue("bearer", token);
                            }));
                    return graphClient;
                }

                catch (Exception ex)
                {
                    Debug.WriteLine("Could not create a graph client: " + ex.Message);
                }
            }

            return graphClient;
        }
```

## <a name="send-an-email-with-microsoft-graph"></a>Senden einer E-Mail mit Microsoft Graph

Öffnen Sie die Datei MailHelper.cs in Ihrem Startprojekt. Diese Datei enthält den Code, durch den eine E-Mail erstellt und gesendet wird. Er besteht aus einer einzigen Methode – ``ComposeAndSendMailAsync`` –, die eine POST-Anforderung konstruiert und an den Endpunkt **https://graph.microsoft.com/v1.0/me/microsoft.graph.SendMail** sendet. 

Die ``ComposeAndSendMailAsync``-Methode verwendet die drei Zeichenfolgenwerte ``subject``, ``bodyContent`` und ``recipients``, die von der Datei „MainPage.xaml.cs“ an sie übergeben werden. Die Zeichenfolgen ``subject`` und ``bodyContent`` werden zusammen mit allen anderen Benutzeroberflächenzeichenfolgen in der Datei „Resources.resw“ gespeichert. Die Zeichenfolge ``recipients`` stammt aus dem Adressfeld in der App-Schnittstelle. 

**REST-Version**

Die REST-Version dieser Datei benötigt  die folgenden `using`-Deklarationen:

```c#
using System;
using System.Threading.Tasks;
```

Da der Benutzer mehrere Adressen übergeben kann, besteht die erste Aufgabe darin, die Zeichenfolge ``recipients`` in einen Satz von EmailAddress-Objekten zu unterteilen, die dann im POST-Text der Anforderung übergeben werden können.

```c#
            // Prepare the recipient list
            string[] splitter = { ";" };
            var splitRecipientsString = recipients.Split(splitter, StringSplitOptions.RemoveEmptyEntries);
            string recipientsJSON = null;

            int n = 0;
            foreach (string recipient in splitRecipientsString)
            {
                if ( n==0)
                recipientsJSON += "{'EmailAddress':{'Address':'" + recipient.Trim() + "'}}";
                else
                {
                    recipientsJSON += ", {'EmailAddress':{'Address':'" + recipient.Trim() + "'}}";
                }
                n++;
            }
```

Die zweite Aufgabe besteht darin, ein gültiges JSON-Nachrichtenobjekt zu erstellen und dieses über eine HTTP-POST-Anforderung an den **me/microsoft.graph.SendMail**-Endpunkt zu senden. Da die Zeichenfolge  ``bodyContent`` ein HTML-Dokument ist, legt die Anforderung den Wert **ContentType** auf HTML fest. Beachten Sie auch den Aufruf von ``AuthenticationHelper.GetTokenHelperAsync``, womit sichergestellt wird, dass in der Anforderung ein neues Zugriffstoken zum Übergeben vorhanden ist.

```c#
                HttpClient client = new HttpClient();
                var token = await AuthenticationHelper.GetTokenHelperAsync();
                client.DefaultRequestHeaders.Add("Authorization", "Bearer " + token);

                // Build contents of post body and convert to StringContent object.
                // Using line breaks for readability.
                string postBody = "{'Message':{" 
                    +  "'Body':{ " 
                    + "'Content': '" + bodyContent + "'," 
                    + "'ContentType':'HTML'}," 
                    + "'Subject':'" + subject + "'," 
                    + "'ToRecipients':[" + recipientsJSON +  "]}," 
                    + "'SaveToSentItems':true}";

                var emailBody = new StringContent(postBody, System.Text.Encoding.UTF8, "application/json");

                HttpResponseMessage response = await client.PostAsync(new Uri("https://graph.microsoft.com/v1.0/me/microsoft.graph.SendMail"), emailBody);

                if ( !response.IsSuccessStatusCode)
                {

                    throw new Exception("We could not send the message: " + response.StatusCode.ToString());
                }
```

Die vollständige Klasse sieht wie folgt aus:

```c#
    class MailHelper
    {
        /// <summary>
        /// Compose and send a new email.
        /// </summary>
        /// <param name="subject">The subject line of the email.</param>
        /// <param name="bodyContent">The body of the email.</param>
        /// <param name="recipients">A semicolon-separated list of email addresses.</param>
        /// <returns></returns>
        internal async Task ComposeAndSendMailAsync(string subject,
                                                            string bodyContent,
                                                            string recipients)
        {

            // Prepare the recipient list
            string[] splitter = { ";" };
            var splitRecipientsString = recipients.Split(splitter, StringSplitOptions.RemoveEmptyEntries);
            string recipientsJSON = null;

            int n = 0;
            foreach (string recipient in splitRecipientsString)
            {
                if ( n==0)
                recipientsJSON += "{'EmailAddress':{'Address':'" + recipient.Trim() + "'}}";
                else
                {
                    recipientsJSON += ", {'EmailAddress':{'Address':'" + recipient.Trim() + "'}}";
                }
                n++;
            }

            try
            {

                HttpClient client = new HttpClient();
                var token = await AuthenticationHelper.GetTokenForUserAsync();
                client.DefaultRequestHeaders.Add("Authorization", "Bearer " + token);

                // Build contents of post body and convert to StringContent object.
                // Using line breaks for readability.
                string postBody = "{'Message':{" 
                    +  "'Body':{ " 
                    + "'Content': '" + bodyContent + "'," 
                    + "'ContentType':'HTML'}," 
                    + "'Subject':'" + subject + "'," 
                    + "'ToRecipients':[" + recipientsJSON +  "]}," 
                    + "'SaveToSentItems':true}";

                var emailBody = new StringContent(postBody, System.Text.Encoding.UTF8, "application/json");

                HttpResponseMessage response = await client.PostAsync(new Uri("https://graph.microsoft.com/v1.0/me/microsoft.graph.SendMail"), emailBody);

                if ( !response.IsSuccessStatusCode)
                {

                    throw new Exception("We could not send the message: " + response.StatusCode.ToString());
                }


            }

            catch (Exception e)
            {
                throw new Exception("We could not send the message: " + e.Message);
            }
        }
    }
```

**Clientbibliothek-Version**

Die Clientbibliothek-Version dieser Datei benötigt  die folgenden `using`-Deklarationen:

```c#
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
```

Der Code zum Senden einer Nachricht mit der Clientbibliothek sieht sehr ähnlich aus wie der Code, der REST-Aufrufe verwendet. Statt JSON-Objekte zu erstellen und diese direkt an den Endpunkt **SendMail** in einer HTTP-POST-Anforderung zu übergeben, erstellen Sie äquivalente Objekte, die in der Clientbibliothek definiert sind und das resultierende `Message`-Objekt an die Methode `SendMail` des `GraphServiceClient` übergeben. Der Client ruft das Zugriffstoken ab und übergibt die Anforderung an den **SendMail**-Endpunkt.

Die vollständige Klasse sieht wie folgt aus:

```c#
    public class MailHelper
    {
        /// <summary>
        /// Compose and send a new email.
        /// </summary>
        /// <param name="subject">The subject line of the email.</param>
        /// <param name="bodyContent">The body of the email.</param>
        /// <param name="recipients">A semicolon-separated list of email addresses.</param>
        /// <returns></returns>
        public async Task ComposeAndSendMailAsync(string subject,
                                                            string bodyContent,
                                                            string recipients)
        {

            // Prepare the recipient list
            string[] splitter = { ";" };
            var splitRecipientsString = recipients.Split(splitter, StringSplitOptions.RemoveEmptyEntries);
            List<Recipient> recipientList = new List<Recipient>();

            foreach (string recipient in splitRecipientsString)
            {
                recipientList.Add(new Recipient { EmailAddress = new EmailAddress { Address = recipient.Trim() } });
            }

            try
            {
                var graphClient = AuthenticationHelper.GetAuthenticatedClient();

                var email = new Message
                {
                    Body = new ItemBody
                    {
                        Content = bodyContent,
                        ContentType = BodyType.Html,
                    },
                    Subject = subject,
                    ToRecipients = recipientList,
                };

                try
                {
                    await graphClient.Me.SendMail(email, true).Request().PostAsync();
                }
                catch (ServiceException exception)
                {
                    throw new Exception("We could not send the message: " + exception.Error == null ? "No error message returned." : exception.Error.Message);
                }


            }

            catch (Exception e)
            {
                throw new Exception("We could not send the message: " + e.Message);
            }
        }
    }
``` 

##<a name="create-the-user-interface-in-mainpage.xaml"></a>Erstellen der Benutzeroberfläche in MainPage.xaml

Nun, da Sie den Code geschrieben haben, der die Authentifizierung des Benutzers und das Senden einer Nachricht über Microsoft Graph veranlasst, brauchen Sie nur noch die oben beschriebene einfache Schnittstelle zu erstellen. 

Die Datei MainPage.xaml in Ihrem Startprojekt enthält bereits die gesamte benötigte XAML. Sie müssen lediglich den Code hinzufügen, durch den die Benutzeroberfläche an die Datei MainPage.xaml.cs verwiesen wird. Suchen Sie diese Datei in Ihrem Projekt, und öffnen Sie sie.

Diese Datei enthält alle bereits alle `using`-Deklarationen für die Clientbibliothek und die REST-Versionen des Beispiels.

***Clientbibliothek-Version***

Die Clientbibliothek-Version der App erstellt einen `GraphServiceClient`, wenn der Benutzer sich authentifiziert. 

Fügen Sie diesen Code in Ihren Namespace ein, um die Clientbibliothek-Version der Klasse MainPage in MainPage.xaml.cs auszuführen:

```c#
    public sealed partial class MainPage : Page
    {
        private string _mailAddress;
        private string _displayName = null;
        private MailHelper _mailHelper = new MailHelper();

        public MainPage()
        {
            this.InitializeComponent();
        }

        protected override void OnNavigatedTo(NavigationEventArgs e)
        {
            // Developer code - if you haven't registered the app yet, we warn you. 
            if (!App.Current.Resources.ContainsKey("ida:ClientID"))
            {
                InfoText.Text = ResourceLoader.GetForCurrentView().GetString("NoClientIdMessage");
                ConnectButton.IsEnabled = false;
            }
            else
            {
                InfoText.Text = ResourceLoader.GetForCurrentView().GetString("ConnectPrompt");
                ConnectButton.IsEnabled = true;
            }
        }

        /// <summary>
        /// Signs in the current user.
        /// </summary>
        /// <returns></returns>
        public async Task<bool> SignInCurrentUserAsync()
        {
            var graphClient = AuthenticationHelper.GetAuthenticatedClient();

            if (graphClient != null)
            {
                var user = await graphClient.Me.Request().GetAsync();
                string userId = user.Id;
                _mailAddress = user.UserPrincipalName;
                _displayName = user.DisplayName;
                return true;
            }
            else
            {
                return false;
            }

        }


        private async void ConnectButton_Click(object sender, RoutedEventArgs e)
        {
            ProgressBar.Visibility = Visibility.Visible;
            if (await SignInCurrentUserAsync())
            { 
                InfoText.Text = "Hi " + _displayName + "," + Environment.NewLine + ResourceLoader.GetForCurrentView().GetString("SendMailPrompt");
                MailButton.IsEnabled = true;
                EmailAddressBox.IsEnabled = true;
                ConnectButton.Visibility = Visibility.Collapsed;
                DisconnectButton.Visibility = Visibility.Visible;
                EmailAddressBox.Text = _mailAddress;
            }
            else
            {
                InfoText.Text = ResourceLoader.GetForCurrentView().GetString("AuthenticationErrorMessage");
            }

            ProgressBar.Visibility = Visibility.Collapsed;
        }

        private async void MailButton_Click(object sender, RoutedEventArgs e)
        {
            _mailAddress = EmailAddressBox.Text;
            ProgressBar.Visibility = Visibility.Visible;
            MailStatus.Text = string.Empty;
            try
            {
                await _mailHelper.ComposeAndSendMailAsync(ResourceLoader.GetForCurrentView().GetString("MailSubject"), ComposePersonalizedMail(_displayName), _mailAddress);
                MailStatus.Text = string.Format(ResourceLoader.GetForCurrentView().GetString("SendMailSuccess"), _mailAddress);
            }
            catch (Exception)
            {
                MailStatus.Text = ResourceLoader.GetForCurrentView().GetString("MailErrorMessage");
            }
            finally
            {
                ProgressBar.Visibility = Visibility.Collapsed;
            }
            
        }

        // <summary>
        // Personalizes the email.
        // </summary>
        public static string ComposePersonalizedMail(string userName)
        {
            return String.Format(ResourceLoader.GetForCurrentView().GetString("MailContents"), userName);
        }

        private void Disconnect_Click(object sender, RoutedEventArgs e)
        {
            ProgressBar.Visibility = Visibility.Visible;
            AuthenticationHelper.SignOut();
            ProgressBar.Visibility = Visibility.Collapsed;
            MailButton.IsEnabled = false;
            EmailAddressBox.IsEnabled = false;
            ConnectButton.Visibility = Visibility.Visible;
            InfoText.Text = ResourceLoader.GetForCurrentView().GetString("ConnectPrompt");
            this._displayName = null;
            this._mailAddress = null;
        }
    }
```

***REST-Version***

Die REST-Version dieser Klasse ist der Clientbibliothek-Version sehr ähnlich, mit der Ausnahme, dass sie die Methode `GetTokenForUserAsync` direkt aufruft, wenn der Benutzer sich authentifiziert. Außerdem ruft sie Benutzerwerte aus den Roaming-Einstellungen der App ab. 

Fügen Sie diesen Code in Ihren Namespace ein, um die REST-Version der Klasse MainPage in MainPage.xaml.cs auszuführen:

```c#
    public sealed partial class MainPage : Page
    {
        private string _mailAddress;
        private string _displayName = null;
        private MailHelper _mailHelper = new MailHelper();
        public static ApplicationDataContainer _settings = ApplicationData.Current.RoamingSettings;

        public MainPage()
        {
            this.InitializeComponent();
        }

        protected override void OnNavigatedTo(NavigationEventArgs e)
        {
            // Developer code - if you haven't registered the app yet, we warn you. 
            if (!App.Current.Resources.ContainsKey("ida:ClientID"))
            {
                InfoText.Text = ResourceLoader.GetForCurrentView().GetString("NoClientIdMessage");
                ConnectButton.IsEnabled = false;
            }
            else
            {
                InfoText.Text = ResourceLoader.GetForCurrentView().GetString("ConnectPrompt");
                ConnectButton.IsEnabled = true;
            }
        }

        /// <summary>
        /// Signs in the current user.
        /// </summary>
        /// <returns></returns>
        public async Task<bool> SignInCurrentUserAsync()
        {
            var token = await AuthenticationHelper.GetTokenForUserAsync();

            if (token != null)
            {
                string userId = (string)_settings.Values["userID"];
                _mailAddress = (string)_settings.Values["userEmail"];
                _displayName = (string)_settings.Values["userName"];
                return true;
            }
            else
            {
                return false;
            }

        }


        private async void ConnectButton_Click(object sender, RoutedEventArgs e)
        {
            ProgressBar.Visibility = Visibility.Visible;
            if (await SignInCurrentUserAsync())
            { 
                InfoText.Text = "Hi " + _displayName + "," + Environment.NewLine + ResourceLoader.GetForCurrentView().GetString("SendMailPrompt");
                MailButton.IsEnabled = true;
                EmailAddressBox.IsEnabled = true;
                ConnectButton.Visibility = Visibility.Collapsed;
                DisconnectButton.Visibility = Visibility.Visible;
                EmailAddressBox.Text = _mailAddress;
            }
            else
            {
                InfoText.Text = ResourceLoader.GetForCurrentView().GetString("AuthenticationErrorMessage");
            }

            ProgressBar.Visibility = Visibility.Collapsed;
        }

        private async void MailButton_Click(object sender, RoutedEventArgs e)
        {
            _mailAddress = EmailAddressBox.Text;
            ProgressBar.Visibility = Visibility.Visible;
            MailStatus.Text = string.Empty;
            try
            {
                await _mailHelper.ComposeAndSendMailAsync(ResourceLoader.GetForCurrentView().GetString("MailSubject"), ComposePersonalizedMail(_displayName), _mailAddress);
                MailStatus.Text = string.Format(ResourceLoader.GetForCurrentView().GetString("SendMailSuccess"), _mailAddress);
            }
            catch (Exception)
            {
                MailStatus.Text = ResourceLoader.GetForCurrentView().GetString("MailErrorMessage");
            }
            finally
            {
                ProgressBar.Visibility = Visibility.Collapsed;
            }
            
        }

        // <summary>
        // Personalizes the email.
        // </summary>
        public static string ComposePersonalizedMail(string userName)
        {
            return String.Format(ResourceLoader.GetForCurrentView().GetString("MailContents"), userName);
        }

        private void Disconnect_Click(object sender, RoutedEventArgs e)
        {
            ProgressBar.Visibility = Visibility.Visible;
            AuthenticationHelper.SignOut();
            ProgressBar.Visibility = Visibility.Collapsed;
            MailButton.IsEnabled = false;
            EmailAddressBox.IsEnabled = false;
            ConnectButton.Visibility = Visibility.Visible;
            InfoText.Text = ResourceLoader.GetForCurrentView().GetString("ConnectPrompt");
            this._displayName = null;
            this._mailAddress = null;
        }
    }
```
 
Sie haben nun die drei erforderlichen Schritte für die Interaktion mit Microsoft Graph durchgeführt: App-Registrierung, Benutzerauthentifizierung und eine Anforderung. 

## <a name="run-the-app"></a>Ausführen der App
1. Drücken Sie zum Erstellen und Ausführen der App F5. 

2. Melden Sie sich mit Ihrem persönlichen Konto oder mit Ihrem Geschäfts- oder Schulkonto an, und gewähren Sie die erforderlichen Berechtigungen.

3. Klicken Sie auf die Schaltfläche **E-Mail senden**. Nachdem die E-Mail gesendet wurde, wird unter der Schaltfläche eine Erfolgsmeldung angezeigt.

## <a name="next-steps"></a>Nächste Schritte
- Testen Sie die REST-API mithilfe des [Graph-Explorers](https://graph.microsoft.io/graph-explorer).
- Beispiele allgemeiner Vorgänge sowohl für REST- als auch für SDK-Operationen finden Sie im [Microsoft Graph UWP Snippets-Beispiel (SDK)](https://github.com/microsoftgraph/uwp-csharp-snippets-sample) und im [Microsoft Graph UWP Snippets-Beispiel (REST)](https://github.com/microsoftgraph/uwp-csharp-snippets-rest-sample), oder erforschen Sie unsere [UWP-Bespiele](https://github.com/microsoftgraph?utf8=%E2%9C%93&query=uwp) auf GitHub.

## <a name="see-also"></a>Siehe auch
- [Microsoft Graph .NET Clientbibliothek](https://github.com/microsoftgraph/msgraph-sdk-dotnet)
- [Protokolle für Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
- [Azure AD v2.0-Tokens](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)

