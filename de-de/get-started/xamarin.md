# <a name="get-started-with-microsoft-graph-in-a-xamarin-forms-app"></a>Erste Schritte mit Microsoft Graph in einer Xamarin Forms-App

Dieser Artikel beschreibt die erforderlichen Aufgaben zum Abrufen eines Zugriffstokens vom [Azure AD v2.0-Endpunkt](https://graph.microsoft.io/en-us/docs/authorization/converged_auth) und zum Aufrufen von Microsoft Graph. Er führt Sie durch den Code im [Microsoft Graph Connect-Beispiel für Xamarin Forms](https://github.com/microsoftgraph/xamarin-csharp-connect-sample) und erläutert die wichtigsten Konzepte, die in einer App implementiert werden müssen, die Microsoft Graph verwendet. In diesem Artikel wird auch beschrieben, wie Sie mit der [Microsoft Graph Clientbibliothek](http://www.nuget.org/packages/Microsoft.Graph/) auf Microsoft Graph zugreifen.

Dies ist die App, die Sie erstellen.

| UWP | Android | iOS |
| --- | ------- | ----|
| <img src="images/UWP.png" alt="Connect sample on UWP" width="100%" /> | <img src="images/Droid.png" alt="Connect sample on Android" width="100%" /> | <img src="images/iOS.png" alt="Connect sample on iOS" width="100%" /> |

**Sie möchten keine App erstellen?** Verwenden Sie für einen schnellen Einstieg den [Schnellstart Microsoft Graph](https://graph.microsoft.io/en-us/getting-started), oder laden Sie das [Microsoft Graph Connect-Beispiel für Xamarin Forms](https://github.com/microsoftgraph/xamarin-csharp-connect-sample) herunter, auf dem dieser Artikel basiert.

## <a name="prerequisites"></a>Voraussetzungen

Für die ersten Schritte benötigen Sie: 

- Ein [Microsoft-Konto](https://www.outlook.com/) oder ein [Geschäfts- oder Schulkonto](http://dev.office.com/devprogram)
- Visual Studio 2015 
- [Xamarin für Visual Studio](https://www.xamarin.com/visual-studio)
- Windows 10 (mit [aktiviertem Entwicklungsmodus](https://msdn.microsoft.com/library/windows/apps/xaml/dn706236.aspx))
- Das [Microsoft Graph Connect-Startprojekt für Xamarin Forms](https://github.com/microsoftgraph/xamarin-csharp-connect-sample/tree/master/starter). Diese Vorlage enthält verschiedene Klassen, denen Sie Code hinzufügen müssen. Darüber hinaus enthält sie vollständige Ansichten und Ressourcenzeichenfolgen. Um dieses Projekt abzurufen, müssen Sie das [Microsoft Graph Connect-Beispiel für Xamarin Forms](https://github.com/microsoftgraph/xamarin-csharp-connect-sample) klonen oder herunterladen und dann die **XamarinConnect**-Projektmappe im **Start**ordner öffnen. 

Wenn Sie das iOS-Projekt in diesem Beispiel ausführen möchten, benötigen Sie Folgendes:

- Das neueste iOS-SDK
- Die neueste Version von Xcode
- Mac OS X Yosemite (10.10) und höher 
- [Xamarin.iOS](https://developer.xamarin.com/guides/ios/getting_started/installation/mac/)
- Einen mit [Visual Studio verbundenen Xamarin Mac-Agent](https://developer.xamarin.com/guides/ios/getting_started/installation/windows/connecting-to-mac/)


## <a name="register-the-app"></a>Registrieren der App
 
1. Melden Sie sich beim [App-Registrierungsportal](https://apps.dev.microsoft.com/) entweder mit Ihrem persönlichen oder geschäftlichen Konto oder mit Ihrem Schulkonto an.
2. Klicken Sie auf **App hinzufügen**.
3. Geben Sie einen Namen für die App ein, und wählen Sie **Anwendung erstellen** aus.
    
    Die Registrierungsseite wird angezeigt, und die Eigenschaften der App werden aufgeführt.
 
4. Wählen Sie unter **Plattformen** die Option **Plattform hinzufügen** aus.
5. Klicken Sie auf **Mobile Plattform**.
6. Kopieren Sie die Anwendungs-ID. Sie müssen diesen Wert in die Beispiel-App eingeben.

    Die Anwendungs-ID ist ein eindeutiger Bezeichner für Ihre App. Der Umleitungs-URI ist ein eindeutiger, von Windows 10 für jede Anwendung bereitgestellter URI, der sicherstellt, dass an diesen URI gesendete Nachrichten nur an diese Anwendung gesendet werden. 

7. Klicken Sie auf **Speichern**.

## <a name="configure-the-project"></a>Konfigurieren des Projekts

1. Öffnen Sie die Projektmappendatei für das Startprojekt in Visual Studio.
2. Öffnen Sie die Datei **App.cs** innerhalb des Projekts **XamarinConnect (Portable)**, und suchen Sie das Feld `ClientId`. Ersetzen Sie den Platzhalter-der Anwendungs-ID durch die Anwendungs-ID der App, die Sie registriert haben.

```c#
public static string ClientID = "ENTER_YOUR_CLIENT_ID";
public static string[] Scopes = { "User.Read", "Mail.Send" };
```
Im Wert `Scopes` sind die Microsoft Graph Berechtigungsbereiche gespeichert, die die App anfordern muss, wenn der Benutzer sich authentifiziert. Beachten Sie, dass der Klassenkonstruktor `App` den ClientID-Wert verwendet, um eine Instanz der MSAL-Klasse `PublicClientApplication` zu instanziieren. Sie verwenden diese Klasse später zum Authentifizieren des Benutzers.

```c#
IdentityClientApp = new PublicClientApplication(ClientID);
```

## <a name="install-the-microsoft-authentication-library-(msal)"></a>Installieren der Microsoft Authentication Library (MSAL)

Die [Microsoft Authentifizierungsbibliothek](https://www.nuget.org/packages/Microsoft.Identity.Client) enthält Klassen und Methoden, die das Authentifizieren von Benutzern über den v2.0-Authentifizierungsendpunkt vereinfachen.

1. Klicken Sie im Projektmappen-Explorer mit der rechten Maustaste auf das Projekt **XamarinConnect (Portable)**, und wählen Sie **NuGet-Pakete verwalten...** aus.
2. Klicken Sie auf „Durchsuchen“, und suchen Sie nach Microsoft.Identity.Client.
3. Wählen Sie die neueste Version der Microsoft Authentifizierungsbibliothek, und klicken Sie auf **Installieren**.

Führen Sie dieselben Schritte für die Projekte **XamarinConnect.Droid**, **XamarinConnect.iOS** und **XamarinConnect.UWP** aus. Die App wird nicht erstellt werden, wenn MSAL nicht in allen vier Projekten installiert ist.

## <a name="install-the-microsoft-graph-client-library"></a>Installieren der Microsoft Graph-Clientbibliothek

1. Klicken Sie im Projektmappen-Explorer mit der rechten Maustaste auf das Projekt **XamarinConnect (Portable)**, und wählen Sie **NuGet-Pakete verwalten...** aus.
2. Klicken Sie auf „Durchsuchen“, und suchen Sie nach Microsoft.Graph.
3. Wählen Sie die neueste Version der Microsoft Graph Clientbibliothek, und klicken Sie auf **Installieren**.

## <a name="create-the-authenticationhelper.cs-class"></a>Erstellen der Klasse AuthenticationHelper.cs

Öffnen Sie die Datei AuthenticationHelper.cs im Projekt **XamarinConnect (Portable)**. Diese Datei wird den gesamten Authentifizierungscode sowie zusätzliche Logik enthalten, die Benutzerinformationen speichert und die Authentifizierung nur dann erzwingt, wenn der Benutzer die Verbindung zu der App getrennt hat. Diese Klasse enthält mindestens drei Methoden: `GetTokenForUserAsync`, `Signout` und `GetAuthenticatedClient`.

Die Methode `GetTokenHelperAsync` wird ausgeführt, wenn der Benutzer sich authentifiziert, und anschließend jedes Mal, wenn die App Microsoft Graph aufruft.

**Verwenden von Deklarationen**

Stellen Sie sicher, dass diese Deklarationen am Anfang der Datei vorhanden sind:

```c#
using Microsoft.Graph;
using System;
using System.Diagnostics;
using System.Net.Http.Headers;
using System.Threading.Tasks;
using Microsoft.Identity.Client;
```

**Felder der Klasse**

Stellen Sie sicher, dass Sie diese Felder in der Klasse AuthenticationHelper haben:

```c#
public static string TokenForUser = null;
public static DateTimeOffset expiration;
private static GraphServiceClient graphClient = null;
```

In diesem Beispiel wird der `GraphServicesClient` in einem Feld gespeichert, damit Sie ihn nur einmal zu erstellen brauchen. Es speichert das Ablaufen von `DateTimeOffset` des Zugriffstokens, sodass ein neues Token erst beschafft wird, wenn das vorhandene Token bald abläuft.

**GetTokenForUserAsync**

Die Methode `GetTokenForUserAsync` verwendet die `PublicClientApplicationClass`, die in der Datei **App.cs** instanziiert ist, um ein Zugriffstoken für den Benutzer abzurufen. Wenn der Benutzer sich nicht bereits authentifiziert hat, wird die Authentifizierungsbenutzeroberfläche gestartet.

```c#
        public static async Task<string> GetTokenForUserAsync()
        {
            if (TokenForUser == null || expiration <= DateTimeOffset.UtcNow.AddMinutes(5))
            {
                AuthenticationResult authResult = await App.IdentityClientApp.AcquireTokenAsync(App.Scopes);

                TokenForUser = authResult.Token;
                expiration = authResult.ExpiresOn;
            }

            return TokenForUser;
        }
```

**Abmeldung**

Die Methode `Signout` meldet alle Benutzer ab, die über die `PublicClientApplication` angemeldet sind (nur einen Benutzer, in diesem Fall), und hebt den Wert `TokenForUser` auf. Sie hebt auch den Wert `GraphServicesClient` auf.

```c#
        public static void SignOut()
        {
            foreach (var user in App.IdentityClientApp.Users)
            {
                user.SignOut();
            }
            graphClient = null;
            TokenForUser = null;

        }
``` 

**GetAuthenticatedClient**

Abschließend benötigen Sie eine Methode zum Erstellen einer `GraphServicesClient`. Diese Methode erstellt einen Client, der die Methode `GetTokenForUserAsync` bei jedem Aufruf von Microsoft Graph über den Client verwendet.

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

Die Methode ``ComposeAndSendMailAsync`` verwendet drei Zeichenfolgenwerte – ``subject``, ``bodyContent`` und ``recipients`` –, die von der Datei "MainPage.xaml.cs" an sie übergeben werden. Die Zeichenfolgen ``subject`` und ``bodyContent`` werden zusammen mit allen anderen Benutzeroberflächenzeichenfolgen in der Datei „AppResources.resx“ gespeichert. Die Zeichenfolge ``recipients`` stammt aus dem Adressfeld in der App-Schnittstelle. 

**Verwenden von Deklarationen**

Fügen Sie diese Deklarationen am Anfang der Datei hinzu:

```c#
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.Graph;
```

Da der Benutzer mehrere Adressen übergeben kann, besteht die erste Aufgabe darin, die Zeichenfolge ``recipients`` in einen Satz von `EmailAddress`-Objekten zu unterteilen, die dann verwendet werden können, um die Liste der `Recipients`-Objekte zu erstellen, die dann im POST-Text der Anforderung übergeben werden.

```c#
            // Prepare the recipient list
            string[] splitter = { ";" };
            var splitRecipientsString = recipients.Split(splitter, StringSplitOptions.RemoveEmptyEntries);
            List<Recipient> recipientList = new List<Recipient>();

            foreach (string recipient in splitRecipientsString)
            {
                recipientList.Add(new Recipient { EmailAddress = new EmailAddress { Address = recipient.Trim() } });
            }
```

Die zweite Aufgabe besteht darin, ein gültiges `Message`-Objekt zu erstellen und dieses an den **me/microsoft.graph.SendMail**-Endpunkt zu senden, und zwar über den `GraphServiceClient`. Da die Zeichenfolge ``bodyContent`` ein HTML-Dokument ist, legt die Anforderung den Wert **ContentType** auf HTML fest.

```c#
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
```

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

Sie haben nun die drei erforderlichen Schritte für die Interaktion mit Microsoft Graph durchgeführt: App-Registrierung, Benutzerauthentifizierung und eine Anforderung. 

## <a name="run-the-app"></a>Ausführen der App
1. Wählen Sie das auszuführende Projekt aus. Wenn Sie die Option für die universelle Windows-Plattform auswählen, können Sie das Beispiel auf dem lokalen Computer ausführen. Wenn Sie das iOS-Projekt ausführen möchten, müssen Sie eine Verbindung zu einem [Mac herstellen, auf dem die Xamarin-Tools installiert](https://developer.xamarin.com/guides/ios/getting_started/installation/windows/connecting-to-mac/) sind. (Sie können diese Projektmappe auch in Xamarin Studio auf einem Mac öffnen und das Beispiel direkt dort ausführen.) Sie können den [Visual Studio-Emulator für Android](https://www.visualstudio.com/features/msft-android-emulator-vs.aspx) verwenden, wenn Sie das Android-Projekt ausführen möchten. 

    ![](images/SelectProject.png "Select project in Visual Studio")

2. Drücken Sie zum Erstellen und Debuggen F5. Führen Sie die Lösung aus, und melden Sie sich entweder mit Ihrem persönlichen Konto oder mit Ihrem Geschäfts- oder Schulkonto an.
    > **Hinweis** Möglicherweise müssen Sie den Buildkonfigurations-Manager öffnen, um sicherzustellen, dass die Build- und Bereitstellungsschritte für das UWP-Projekt ausgewählt sind. 

3. Melden Sie sich mit Ihrem persönlichen Konto oder mit Ihrem Geschäfts- oder Schulkonto an, und gewähren Sie die erforderlichen Berechtigungen.

4. Klicken Sie auf die Schaltfläche **E-Mail senden**. Nachdem die E-Mail gesendet wurde, wird eine Erfolgsmeldung angezeigt.

## <a name="next-steps"></a>Nächste Schritte
- Testen Sie die REST-API mithilfe des [Graph-Explorers](https://graph.microsoft.io/graph-explorer).
- Beispiele für allgemeine Vorgänge finden Sie in der [Microsoft Graph SDK Snippets-Bibliothek für Xamarin.Forms](https://github.com/microsoftgraph/xamarin-csharp-snippets-sample), oder erforschen Sie unsere anderen [Xamarin-Beispiele](https://github.com/microsoftgraph?utf8=%E2%9C%93&query=xamarin) auf GitHub.

## <a name="see-also"></a>Siehe auch
- [Microsoft Graph .NET Clientbibliothek](https://github.com/microsoftgraph/msgraph-sdk-dotnet)
- [Protokolle für Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
- [Azure AD v2.0-Tokens](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)
