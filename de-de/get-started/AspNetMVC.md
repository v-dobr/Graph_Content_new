# <a name="get-started-with-microsoft-graph-in-an-asp.net-4.6-mvc-app"></a>Erste Schritte mit Microsoft Graph in einer ASP.NET 4.6 MVC-App

Dieser Artikel beschreibt die erforderlichen Aufgaben zum Abrufen eines Zugriffstoken vom Azure AD v2.0-Endpunkt und zum Aufrufen von Microsoft Graph. Sie werden durch die Erstellung des [Microsoft Graph Connect-Beispiels für ASP.NET 4.6](https://github.com/microsoftgraph/aspnet-connect-sample) geführt und erhalten Informationen zu den Hauptkonzepten, die Sie zur Verwendung von Microsoft Graph implementieren. In diesem Artikel wird auch beschrieben, wie Sie mithilfe des [Microsoft Graph-SDKs für Android](https://github.com/microsoftgraph/msgraph-sdk-dotnet) oder reinen REST-Aufrufen auf Microsoft Graph zugreifen.

In der folgenden Abbildung ist die App dargestellt, die Sie erstellen. 

![Die Web-App mit den Schaltfläche "E-Mail-Adresse abrufen" und "E-Mail senden" ](images/aspnet-connect-sample.png "Die Web-App mit den Schaltfläche 'E-Mail-Adresse abrufen' und 'E-Mail senden'")

Der [Azure AD Version 2.0-Endpunkt](https://azure.microsoft.com/en-us/documentation/articles/active-directory-appmodel-v2-overview) ermöglicht Benutzern, sich mit einem Microsoft-Konto (MSA) oder einem Geschäfts-, Schul- oder Unikonto anzumelden. Die App verwendet die [ASP.Net OpenID Connect OWIN Middleware](https://www.nuget.org/packages/Microsoft.Owin.Security.OpenIdConnect/) und die [Microsoft Authentication Library (MSAL) für .NET](https://www.nuget.org/packages/Microsoft.Identity.Client) für die Anmeldung und die Token-Verwaltung.

>MSAL befindet sich derzeit in der Vorabversion und sollte daher nicht in Produktionscode verwendet werden. Es dient hier nur zur Veranschaulichung 

**Sie möchten keine App erstellen?** Verwenden Sie für einen schnellen Einstieg den [Schnellstart Microsoft Graph](https://graph.microsoft.io/en-us/getting-started).

Um eine Version des Connect-Beispiels herunterzuladen, die den Azure AD-Endpunkt  verwendet und über REST-Anrufe auf Microsoft Graph zugreift, finden Sie unter [Office 365 Connect ASP.NET MVC-Beispiel](https://github.com/microsoftgraph/aspnet-connect-rest-sample).


## <a name="prerequisites"></a>Voraussetzungen

Für die ersten Schritte benötigen Sie: 

- Ein [Microsoft-Konto](https://www.outlook.com/) oder ein [Geschäfts- oder Schulkonto](http://dev.office.com/devprogram)
- Visual Studio 2015 
- [Microsoft Graph Connect-Beispiel für ASP.NET 4.6](https://github.com/microsoftgraph/aspnet-connect-sample). Sie verwenden den Ordner **Startprojekt** in den Beispieldateien.


## <a name="register-the-application"></a>Registrieren der App

In diesem Schritt registrieren Sie eine App im Microsoft App-Registrierungsportal. Dadurch werden die APP-ID und das Kennwort generiert, mit der bzw. dem Sie die App in Visual Studio konfigurieren.

1. Melden Sie sich beim [Microsoft-App-Registrierungsportal](https://apps.dev.microsoft.com/) entweder mit Ihrem persönlichen oder geschäftlichen Konto oder mit Ihrem Schulkonto an.

2. Klicken Sie auf **App hinzufügen**.

3. Geben Sie einen Namen für die App ein, und wählen Sie **Anwendung erstellen** aus. 
    
    Die Registrierungsseite wird angezeigt, und die Eigenschaften der App werden aufgeführt.

4. Kopieren Sie die Anwendungs-ID: Dies ist der eindeutige Bezeichner für Ihre App. 

5. Wählen Sie unter **Anwendungsgeheimnisse** die Option **Neues Kennwort generieren** aus. Kopieren Sie das Kennwort aus dem Dialogfeld **Neues Kennwort wurde generiert**.

    Sie werden die Anwendungs-ID und das Kennwort verwenden, um die App zu konfigurieren. 

6. Wählen Sie unter **Plattformen** die Option **Plattform hinzufügen** > ** Web** aus.

7. Stellen Sie sicher, dass das Kontrollkästchen **Impliziten Fluss zulassen** aktiviert ist, und geben Sie *http://localhost:55065/* als Umleitungs-URI ein. 

    Die Option **Impliziten Fluss zulassen** ermöglicht den OpenID Connect-Hybridfluss. Während der Authentifizierung ermöglicht dies der App, sowohl Anmeldeinformationen (das **id_token**) als auch Artefakte (in diesem Fall ein Autorisierungscode) abzurufen, den die App zum Abrufen eines Zugriffstokens verwendet.

8. Wählen Sie **Speichern** aus.

### <a name="configure-the-project"></a>Konfigurieren des Projekts

1. Öffnen Sie die Projektmappendatei für das Startprojekt in Visual Studio.

2. Öffnen Sie die Datei Web.config des Projekts.

3. Suchen Sie die App-Konfigurationsschlüssel im Element **AppSettings**. Ersetzen Sie die Platzhalterwerte ENTER_YOUR_CLIENT_ID und ENTER_YOUR_SECRET durch die Werte, die Sie soeben kopiert haben.

Der Umleitung-URI ist die URL des Projekts, das Sie registriert haben. Die angeforderten [Berechtigungsbereiche](https://graph.microsoft.io/en-us/docs/authorization/permission_scopes) ermöglichen der App das Abrufen der Benutzerprofilinformationen und das Senden einer E-Mail.


## <a name="authenticate-the-user-and-get-an-access-token"></a>Authentifizierung des Benutzers und Abrufen eines Zugriffstokens

In diesem Schritt fügen Sie Code für die Anmeldung und die Token-Verwaltung ein. Doch zunächst werfen wir einen genaueren Blick auf den Ablauf der Authentifizierung.

Diese App verwendet den Authorization Code Grant-Datenfluss mit einer delegierten Benutzeridentität. Für eine Webanwendung erfordert der Ablauf die Anwendungs-ID, das Kennwort und den Umleitungs-URI aus der registrierten App. 

Der Authentifizierungsfluss kann in diese grundlegenden Schritte unterteilt werden:

1. Umleitung des Benutzers für die Authentifizierung und Zustimmung
2. Anfordern eines Autorisierungscodes
3. Einlösen des Autorisierungscodes für ein Zugriffstoken
4. Anfordern eines neuen Zugriffstokens mit dem Aktualisierungstoken, wenn das Zugriffstoken abläuft

Die App verwendet die [ASP.Net OpenID Connect OWIN Middleware](https://www.nuget.org/packages/Microsoft.Owin.Security.OpenIdConnect/) und die [Microsoft Authentication Library (MSAL) für .NET](https://www.nuget.org/packages/Microsoft.Identity.Client) für die Anmeldung und die Token-Verwaltung. Sie verarbeitet die meisten Authentifizierungsaufgaben für Sie.
    
Das Startprojekt deklariert bereits die folgenden Middleware- und MSAL NuGet-Abhängigkeiten:

  - Microsoft.Owin.Security.OpenIdConnect
  - Microsoft.Owin.Security.Cookies
  - Microsoft.Owin.Host.SystemWeb
  - Microsoft.Identity.Client -Pre

Damit zurück zur App-Erstellung.

1. Im Ordner **App_Start** öffnen Sie Startup.Auth.cs. 

1. Ersetzen Sie die **ConfigureAuth**-Methode durch den folgenden Code. Dadurch werden die Koordinaten für die Kommunikation mit Azure AD und die Cookie-Authentifizierung eingerichtet, von der Middleware OpenID Connect verwendet werden.

        public void ConfigureAuth(IAppBuilder app)
        {
            app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);

            app.UseCookieAuthentication(new CookieAuthenticationOptions());

            app.UseOpenIdConnectAuthentication(
                new OpenIdConnectAuthenticationOptions
                {

                    // The `Authority` represents the Microsoft v2.0 authentication and authorization service.
                    // The `Scope` describes the permissions that your app will need. See https://azure.microsoft.com/documentation/articles/active-directory-v2-scopes/                    
                    ClientId = appId,
                    Authority = "https://login.microsoftonline.com/common/v2.0",
                    PostLogoutRedirectUri = redirectUri,
                    RedirectUri = redirectUri,
                    Scope = "openid email profile offline_access " + graphScopes,
                    TokenValidationParameters = new TokenValidationParameters
                    {
                        ValidateIssuer = false,
                        // In a real application you would use IssuerValidator for additional checks, 
                        // like making sure the user's organization has signed up for your app.
                        //     IssuerValidator = (issuer, token, tvp) =>
                        //     {
                        //         if (MyCustomTenantValidation(issuer)) 
                        //             return issuer;
                        //         else
                        //             throw new SecurityTokenInvalidIssuerException("Invalid issuer");
                        //     },
                    },
                    Notifications = new OpenIdConnectAuthenticationNotifications
                    {
                        AuthorizationCodeReceived = async (context) =>
                        {
                            var code = context.Code;
                            string signedInUserID = context.AuthenticationTicket.Identity.FindFirst(ClaimTypes.NameIdentifier).Value;
                            ConfidentialClientApplication cca = new ConfidentialClientApplication(
                                appId, 
                                redirectUri,
                                new ClientCredential(appSecret),
                                new SessionTokenCache(signedInUserID, context.OwinContext.Environment["System.Web.HttpContextBase"] as HttpContextBase));
                                string[] scopes = graphScopes.Split(new char[] { ' ' });

                            AuthenticationResult result = await cca.AcquireTokenByAuthorizationCodeAsync(scopes, code);
                        },
                        AuthenticationFailed = (context) =>
                        {
                            context.HandleResponse();
                            context.Response.Redirect("/Error?message=" + context.Exception.Message);
                            return Task.FromResult(0);
                        }
                    }
                });
        }
  
  Die OWIN-Startklasse (in Startup.cs definiert) ruft die **ConfigureAuth**-Methode auf, wenn die App gestartet wird. Dadurch wiederum wird **app. UseOpenIdConnectAuthentication** aufgerufen, um die Middleware für die Anmeldung und die ursprüngliche Token-Anforderung zu initialisierieren. Die App fordert die folgenden Berechtigungsbereiche an:

  - **openid**, **email**, **profile** für die Anmeldung
  - **offline\_access** (zum Abrufen von Aktualsierungstoken erforderlich), **User.Read**, **Mail.Send** für die Token-Erfassung
  
  Das MSAL-Objekt **ConfidentialClientApplication** stellt die App dar und übernimmt Token-Verwaltungsaufgaben. Es wird mit **SessionTokenCache** (die Beispiel-Token-Cache-Implementierung, die in TokenStorage/SessionTokenCache.cs definiert ist) initialisiert. Dort werden die Token-Informationen gespeichert. Der Cache speichert Token in der aktuellen HTTP-Sitzung basierend auf der Benutzer-ID, aber eine Produktionsanwendung wird wahrscheinlich einen beständigeren Speicher verwenden.

Jetzt fügen Sie dem Beispiel-Authentifizierungsanbieter Code hinzu, der von Ihrem eigenen Authentifizierungsanbieter problemlos ersetzt werden kann. Die Benutzeroberfläche und Anbieterklasse wurden dem Projekt bereits hinzugefügt.

1. Im Ordner **Hilfsprogramme** öffnen Sie SampleAuthProvider.cs.

1. Ersetzen Sie die **GetUserAccessTokenAsync**-Methode mit der folgenden Implementierung, die MSAL zum Abrufen eines Zugriffstokens verwendet.

        // Get an access token. First tries to get the token from the token cache.
        public async Task<string> GetUserAccessTokenAsync()
        {
            string signedInUserID = ClaimsPrincipal.Current.FindFirst(ClaimTypes.NameIdentifier).Value;
            tokenCache = new SessionTokenCache(
                signedInUserID, 
                HttpContext.Current.GetOwinContext().Environment["System.Web.HttpContextBase"] as HttpContextBase);
            //var cachedItems = tokenCache.ReadItems(appId); // see what's in the cache

            ConfidentialClientApplication cca = new ConfidentialClientApplication(
                appId, 
                redirectUri,
                new ClientCredential(appSecret), 
                tokenCache);

            try
            {
                AuthenticationResult result = await cca.AcquireTokenSilentAsync(scopes.Split(new char[] { ' ' }));
                return result.Token;
            }

            // Unable to retrieve the access token silently.
            catch (MsalSilentTokenAcquisitionException)
            {
                HttpContext.Current.Request.GetOwinContext().Authentication.Challenge(
                    new AuthenticationProperties() { RedirectUri = "/" },
                    OpenIdConnectAuthenticationDefaults.AuthenticationType);

                throw new Exception(Resource.Error_AuthChallengeNeeded);
            }
        }

  MSAL prüft den Cache für einen passenden Zugriffstoken, der nicht abgelaufen ist oder demnächst abläuft. Wenn Sie keinen gültigen finden, wird der Aktualisierungstoken (sofern ein gültiger vorhanden ist) zum Abrufen eines neuen Zugriffstoken verwendet. Wenn kein neuer Zugriffstoken im Hintergrund abgerufen werden kann, gibt MSAL eine **MsalSilentTokenAcquisitionException**-Ausnahme aus, die angibt, dass eine Eingabeaufforderung durch den Benutzer erforderlich ist. 

Als Nächstes fügen Sie Code hinzu, um die An- und Abmeldung von der Benutzeroberfläche zu ermöglichen.

1. Im Ordner **Controller** öffnen Sie AccountController.cs.  

1. Fügen Sie der **AccountController**-Klasse die folgenden Methoden hinzu. Die **Anmeldung**-Methode signalisiert der Middleware, eine Autorisierungsanforderung an Azure AD zu senden.

        public void SignIn()
        {
            if (!Request.IsAuthenticated)
            {
                // Signal OWIN to send an authorization request to Azure.
                HttpContext.GetOwinContext().Authentication.Challenge(
                    new AuthenticationProperties { RedirectUri = "/" },
                    OpenIdConnectAuthenticationDefaults.AuthenticationType);
            }
        }

        // Here we just clear the token cache, sign out the GraphServiceClient, and end the session with the web app.  
        public void SignOut()
        {
            if (Request.IsAuthenticated)
            {
                // Get the user's token cache and clear it.
                string userObjectId = ClaimsPrincipal.Current.FindFirst(ClaimTypes.NameIdentifier).Value;

                SessionTokenCache tokenCache = new SessionTokenCache(userObjectId, HttpContext);
                tokenCache.Clear(userObjectId);
            }

            //SDKHelper.SignOutClient();

            // Send an OpenID Connect sign-out request. 
            HttpContext.GetOwinContext().Authentication.SignOut(
            CookieAuthenticationDefaults.AuthenticationType);
            Response.Redirect("/");
        }

Sie können nun Code hinzufügen, um Microsoft Graph aufzurufen. 

## <a name="call-microsoft-graph"></a>Aufrufen von Microsoft Graph

Wenn Sie die Microsoft Graph-Bibliothek verwenden, lesen Sie weiter. Wenn Sie REST verwenden, lesen Sie den Abschnitt [Verwenden der REST-API](#using-the-rest-api).

### <a name="using-the-library"></a>Verwenden der Bibliothek
In diesem Schritt liegt der Schwerpunkt auf den Klassen **SDKHelper**, **GraphService** und **HomeController**. 

 - **SDKHelper** Initialisiert eine Instanz von **GraphServiceClient** aus der Bibliothek vor jedem Aufruf von Microsoft Graph. Zu diesem Zeitpunkt wir der Zugriffstoken der Anforderung hinzugefügt. 
 - **GraphService** erstellt und sendet mithilfe der Bibliothek Anfragen an Microsoft Graph und verarbeitet die Antworten.
 - **HomeController** enthält Aktionen, die Bibliotheksaufrufe als Antwort auf Benutzeroberflächenereignisse auslösen.

Das Startprojekt deklariert bereits eine Abhängigkeit für das Microsoft Graph .NET Client Library NuGet-Paket:  *Microsoft.Graph*.

1. Klicken Sie mit der rechten Maustaste in den Ordner **Hilfsprogramme** Ordner, und wählen Sie **Hinzufügen** > **Klasse**. 

1. Nennen Sie die neue Klasse *SDKHelper*, und wählen Sie **Hinzufügen**.

1. Ersetzen Sie den Inhalt durch den folgenden Code.

        using System.Net.Http.Headers;
        using Microsoft.Graph;

        namespace Microsoft_Graph_SDK_ASPNET_Connect.Helpers
        {
            public class SDKHelper
            {   
                private static GraphServiceClient graphClient = null;

                // Get an authenticated Microsoft Graph Service client.
                public static GraphServiceClient GetAuthenticatedClient()
                {
                    GraphServiceClient graphClient = new GraphServiceClient(
                        new DelegateAuthenticationProvider(
                            async (requestMessage) =>
                            {
                                string accessToken = await SampleAuthProvider.Instance.GetUserAccessTokenAsync();

                                // Append the access token to the request.
                                requestMessage.Headers.Authorization = new AuthenticationHeaderValue("bearer", accessToken);
                            }));
                    return graphClient;
                }

                public static void SignOutClient()
                {
                    graphClient = null;
                }
            }
        }

  Beachten Sie den Aufruf von **SampleAuthProvider**, um den Token abzurufen, wenn der Client initialisiert wird.

1. Im Ordner **Modelle** öffnen Sie GraphService.cs. Der Dienst verwendet die Bibliothek zur Interaktion mit dem Microsoft Graph.

1. Fügen Sie die folgende **using**-Anweisung hinzu.

        using Microsoft.Graph;

1. Ersetzen Sie */ / GetMyEmailAddress* durch die folgende Methode. Dadurch wird die E-Mail-Adresse des aktuellen Benutzers abgerufen. 

        // Get the current user's email address from their profile.
        public async Task<string> GetMyEmailAddress(GraphServiceClient graphClient)
        {

            // Get the current user. 
            // The app only needs the user's email address, so select the mail and userPrincipalName properties.
            // If the mail property isn't defined, userPrincipalName should map to the email for all account types. 
            User me = await graphClient.Me.Request().Select("mail,userPrincipalName").GetAsync();
            return me.Mail ?? me.UserPrincipalName;
        }

  Beachten Sie das **Select**-Segment, mit dem nur **mail** und **UserPrinicipalName** zurückgegeben werden. Sie können mit **Select** und **Filter** die Größe der Antwortdaten-Nutzlast verringern.

1. Ersetzen Sie */ / SendEmail* mit den folgenden Methoden zum Erstellen und Senden der E-Mail.

        // Send an email message from the current user.
        public async Task SendEmail(GraphServiceClient graphClient, Message message)
        {
            await graphClient.Me.SendMail(message, true).Request().PostAsync();
        }

        // Create the email message.
        public Message BuildEmailMessage(string recipients, string subject)
        {

            // Prepare the recipient list.
            string[] splitter = { ";" };
            string[] splitRecipientsString = recipients.Split(splitter, StringSplitOptions.RemoveEmptyEntries);
            List<Recipient> recipientList = new List<Recipient>();
            foreach (string recipient in splitRecipientsString)
            {
                recipientList.Add(new Recipient
                {
                    EmailAddress = new EmailAddress
                    {
                        Address = recipient.Trim()
                    }
                });
            }

            // Build the email message.
            Message email = new Message
            {
                Body = new ItemBody
                {
                    Content = Resource.Graph_SendMail_Body_Content,
                    ContentType = BodyType.Html,
                },
                Subject = subject,
                ToRecipients = recipientList
            };
            return email;
        }

1. Im Ordner **Controller** öffnen Sie HomeController.cs.

1. Fügen Sie die folgende **using**-Anweisung hinzu.

        using Microsoft.Graph;
  
1. Ersetzen Sie *// Controller actions* durch die folgenden Aktionen.

        [Authorize]
        // Get the current user's email address from their profile.
        public async Task<ActionResult> GetMyEmailAddress()
        {
            try
            {

                // Initialize the GraphServiceClient.
                GraphServiceClient graphClient = SDKHelper.GetAuthenticatedClient();

                // Get the current user's email address. 
                ViewBag.Email = await graphService.GetMyEmailAddress(graphClient);
                return View("Graph");
            }
            catch (ServiceException se)
            {
                if (se.Error.Message == Resource.Error_AuthChallengeNeeded) return new EmptyResult();
                return RedirectToAction("Index", "Error", new { message = Resource.Error_Message + Request.RawUrl + ": " + se.Error.Message });
            }
        }

        [Authorize]
        // Send mail on behalf of the current user.
        public async Task<ActionResult> SendEmail()
        {
            if (string.IsNullOrEmpty(Request.Form["email-address"]))
            {
                ViewBag.Message = Resource.Graph_SendMail_Message_GetEmailFirst;
                return View("Graph");
            }

            // Build the email message.
            Message message = graphService.BuildEmailMessage(Request.Form["recipients"], Request.Form["subject"]);
            try
            {

                // Initialize the GraphServiceClient.
                GraphServiceClient graphClient = SDKHelper.GetAuthenticatedClient();

                // Send the email.
                await graphService.SendEmail(graphClient, message);

                // Reset the current user's email address and the status to display when the page reloads.
                ViewBag.Email = Request.Form["email-address"];
                ViewBag.Message = Resource.Graph_SendMail_Success_Result;
                return View("Graph");
            }
            catch (ServiceException se)
            {
                if (se.Error.Message == Resource.Error_AuthChallengeNeeded) return new EmptyResult();
                return RedirectToAction("Index", "Error", new { message = Resource.Error_Message + Request.RawUrl + ": " + se.Error.Message });
           }
        }

Als Nächstes ändern Sie die Ausnahme, die der Authentifizierungsanbieter auslöst, wenn eine Benutzereingabeaufforderung erforderlich ist.

1. Im Ordner **Hilfsprogramme** öffnen Sie SampleAuthProvider.cs.

1. Fügen Sie die folgende **using**-Anweisung hinzu.

        using Microsoft.Graph;
  
1. Ändern Sie im **Catch**-Block der **GetUserAccessTokenAsync**-Methode die ausgelöste Ausnahme wie folgt:

        throw new ServiceException(
            new Error
            {
                Code = GraphErrorCode.AuthenticationFailure.ToString(),
                Message = Resource.Error_AuthChallengeNeeded,
            });

Abschließend fügen Sie einen Anruf zum Abmelden des Client ein. 

1. Im Ordner **Controller** öffnen Sie AccountController.cs. 

1. Entfernen Sie die Auskommentierung aus der folgenden Zeile:

        SDKHelper.SignOutClient();

Jetzt können [die App ausführen](#run-the-app).

### <a name="using-the-rest-api"></a>Verwenden der REST-API
In diesem Schritt arbeiten Sie mit den Klassen **GraphService**, **GraphResources** und **HomeController**. 

 - **GraphService** erstellt und sendet HTTP-Anfragen an Microsoft Graph und verarbeitet die Antworten. 
 - **GraphResources** stellt Daten dar, welche die App an Microsoft Graph sendet und von Microsoft Graph empfängt. 
 - **HomeController** enthält Aktionen, die Aufrufe von Microsoft Graph als Antwort auf Benutzeroberflächenereignisse auslösen.

Beginnen Sie mit der Definition Ihrer Datenzugriffebene.

1. Im Ordner **Modelle** öffnen Sie GraphService.cs.

1. Fügen Sie die folgenden **using**-Anweisungen hinzu.

        using Newtonsoft.Json;
        using Newtonsoft.Json.Linq;
        using System.Net;
        using System.Net.Http;
        using System.Net.Http.Headers;
        using System.Text;

1. Ersetzen Sie */ / GetMyEmailAddress* durch die folgende Methode. Dadurch wird die E-Mail-Adresse des aktuellen Benutzers abgerufen. 

        // Get the current user's email address from their profile.
        public async Task<string> GetMyEmailAddress(string accessToken)
        {

            // Get the current user. 
            // The app only needs the user's email address, so select the mail and userPrincipalName properties.
            // If the mail property isn't defined, userPrincipalName should map to the email for all account types. 
            string endpoint = "https://graph.microsoft.com/v1.0/me";
            string queryParameter = "?$select=mail,userPrincipalName";
            UserInfo me = new UserInfo();

            using (var client = new HttpClient())
            {
                using (var request = new HttpRequestMessage(HttpMethod.Get, endpoint + queryParameter))
                {
                    request.Headers.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
                    request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);
                    using (HttpResponseMessage response = await client.SendAsync(request))
                    {
                        if (response.StatusCode == HttpStatusCode.OK)
                        {
                            var json = JObject.Parse(await response.Content.ReadAsStringAsync());
                            me.Address = !string.IsNullOrEmpty(json.GetValue("mail").ToString())?json.GetValue("mail").ToString():json.GetValue("userPrincipalName").ToString();
                        }
                        return me.Address?.Trim();
                    }
                }
            }
        }

1. Ersetzen Sie */ / SendEmail* mit den folgenden Methoden zum Erstellen und Senden der E-Mail.

        // Send an email message from the current user.
        public async Task<string> SendEmail(string accessToken, MessageRequest email)
        {
            string endpoint = "https://graph.microsoft.com/v1.0/me/sendMail";
            using (var client = new HttpClient())
            {
                using (var request = new HttpRequestMessage(HttpMethod.Post, endpoint))
                {
                    request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);
                    request.Content = new StringContent(JsonConvert.SerializeObject(email), Encoding.UTF8, "application/json");
                    using (HttpResponseMessage response = await client.SendAsync(request))
                    {
                        if (response.IsSuccessStatusCode)
                        {
                            return Resource.Graph_SendMail_Success_Result;
                        }
                        return response.ReasonPhrase;
                    }
                }
            }
        }

        // Create the email message.
        public MessageRequest BuildEmailMessage(string recipients, string subject)
        {

            // Prepare the recipient list.
            string[] splitter = { ";" };
            string[] splitRecipientsString = recipients.Split(splitter, StringSplitOptions.RemoveEmptyEntries);
            List<Recipient> recipientList = new List<Recipient>();
            foreach (string recipient in splitRecipientsString)
            {
                recipientList.Add(new Recipient
                {
                    EmailAddress = new UserInfo
                    {
                        Address = recipient.Trim()
                    }
                });
            }

            // Build the email message.
            Message message = new Message
            {
                Body = new ItemBody
                {
                    Content = Resource.Graph_SendMail_Body_Content,
                    ContentType = "HTML"
                },
                Subject = subject,
                ToRecipients = recipientList
            };

            return new MessageRequest
            {
                Message = message,
                SaveToSentItems = true
            };
        }

1. Klicken Sie mit der rechten Maustaste in den Ordner **Modelle**, und wählen Sie **Hinzufügen** > **Klasse**.

1. Nennen Sie die Klasse *GraphResources*, und wählen Sie **OK**.

1. Ersetzen Sie den Inhalt durch den folgenden Code.
 
        using System;
        using System.Collections.Generic;

        namespace Microsoft_Graph_SDK_ASPNET_Connect.Models
        {
            public class UserInfo
            {
                public string Address { get; set; }
            }

            public class Message
            {
                public string Subject { get; set; }
                public ItemBody Body { get; set; }
                public List<Recipient> ToRecipients { get; set; }
            }

          public class Recipient
            {
                public UserInfo EmailAddress { get; set; }
            }

            public class ItemBody
            {
                public string ContentType { get; set; }
                public string Content { get; set; }
            }

            public class MessageRequest
            {
                public Message Message { get; set; }
                public bool SaveToSentItems { get; set; }
            }
        }

1. Im Ordner **Controller** öffnen Sie HomeController.cs.
  
1. Ersetzen Sie *// Controller actions* durch die folgenden Aktionen.

        [Authorize]
        // Get the current user's email address from their profile.
        public async Task<ActionResult> GetMyEmailAddress()
        {
            try
            {

                // Get an access token.
                string accessToken = await SampleAuthProvider.Instance.GetUserAccessTokenAsync();

                // Get the current user's email address. 
                ViewBag.Email = await graphService.GetMyEmailAddress(accessToken);
                return View("Graph");
            }
            catch (Exception e)
            {
                if (e.Message == Resource.Error_AuthChallengeNeeded) return new EmptyResult();
                return RedirectToAction("Index", "Error", new { message = Resource.Error_Message + Request.RawUrl + ": " + e.Message });
            }
        }

        [Authorize]
        // Send mail on behalf of the current user.
        public async Task<ActionResult> SendEmail()
        {
            if (string.IsNullOrEmpty(Request.Form["email-address"]))
            {
                ViewBag.Message = Resource.Graph_SendMail_Message_GetEmailFirst;
                return View("Graph");
            }

            // Build the email message.
            MessageRequest email = graphService.BuildEmailMessage(Request.Form["recipients"], Request.Form["subject"]);
            try
            {

                // Get an access token.
                string accessToken = await SampleAuthProvider.Instance.GetUserAccessTokenAsync();

                // Send the email.
                ViewBag.Message = await graphService.SendEmail(accessToken, email);

                // Reset the current user's email address and the status to display when the page reloads.
                ViewBag.Email = Request.Form["email-address"];
                return View("Graph");
            }
            catch (Exception e)
            {
                if (e.Message == Resource.Error_AuthChallengeNeeded) return new EmptyResult();
                return RedirectToAction("Index", "Error", new { message = Resource.Error_Message + Request.RawUrl + ": " + e.Message });
            }
        }

## <a name="run-the-app"></a>Ausführen der App
1. Drücken Sie zum Erstellen und Ausführen der App F5. 

2. Melden Sie sich mit Ihrem persönlichen Konto oder mit Ihrem Geschäfts- oder Schulkonto an, und gewähren Sie die erforderlichen Berechtigungen.

3. Klicken Sie auf die Schaltfläche **E-Mail-Adresse abrufen**. Wenn der Vorgang abgeschlossen ist, wird die E-Mail-Adresse des angemeldeten Benutzers auf der Seite angezeigt.

4. Optional können Sie die Empfängerliste und den Betreff der E-Mail bearbeiten. Klicken Sie dann auf die Schaltfläche **E-Mail senden**. Nachdem die E-Mail gesendet wurde, wird unter der Schaltfläche eine Erfolgsmeldung angezeigt.


## <a name="next-steps"></a>Nächste Schritte
- Testen Sie die REST-API mithilfe des [Graph-Explorers](https://graph.microsoft.io/graph-explorer).
- Suchen Sie nach Beispielen für allgemeine Vorgänge im [Microsoft Graph-Codeausschnittbeispiel für ASP.NET 4.6](https://github.com/microsoftgraph/aspnet-snippets-sample). Sie können auch unsere anderen [ASP.NET-Beispiele](http://aka.ms/aspnetgraphsamples) auf GitHub erkunden.

## <a name="see-also"></a>Siehe auch
- [Microsoft Graph .NET Clientbibliothek](https://github.com/microsoftgraph/msgraph-sdk-dotnet)
- [Webanwendung mit Web-API-Authentifizierungsszenario](https://azure.microsoft.com/en-us/documentation/articles/active-directory-authentication-scenarios/#web-application-to-web-api)
- [Integrieren einer Microsoft-Identität und von Microsoft Graph in eine Webanwendung mithilfe von OpenID Connect](https://azure.microsoft.com/en-us/documentation/samples/active-directory-dotnet-webapp-openidconnect-v2/)
- [Azure AD v2.0-Protokolle](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
- [Azure AD v2.0-Tokens](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)
