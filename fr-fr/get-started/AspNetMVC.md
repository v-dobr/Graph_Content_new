# <a name="get-started-with-microsoft-graph-in-an-asp.net-4.6-mvc-app"></a>Prise en main de Microsoft Graph dans une application ASP.NET 4.6 MVC

Cet article décrit les tâches requises pour obtenir un jeton d’accès du point de terminaison Azure AD v2.0 et appeler Microsoft Graph. Il vous guidera dans la construction de l’[exemple de connexion Microsoft Graph pour ASP.NET 4.6](https://github.com/microsoftgraph/aspnet-connect-sample) et décrit les principaux concepts à mettre en œuvre pour utiliser Microsoft Graph. Cet article explique également comment accéder à Microsoft Graph à l’aide de la [bibliothèque cliente .NET de Microsoft Graph](https://github.com/microsoftgraph/msgraph-sdk-dotnet) ou des appels REST bruts.

L’image suivante présente l’application que vous allez créer. 

![Application web avec les boutons « Obtenir une adresse de messagerie » et « Envoyer un message électronique »](images/aspnet-connect-sample.png "Application web avec les boutons « Obtenir une adresse de messagerie » et « Envoyer un message électronique »")

Le [point de terminaison Azure AD v 2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-appmodel-v2-overview) permet aux utilisateurs de se connecter avec un compte Microsoft ou à l’aide d’un compte professionnel ou scolaire. L’application utilise l’[intergiciel OWIN OpenID Connect ASP.Net](https://www.nuget.org/packages/Microsoft.Owin.Security.OpenIdConnect/) et la [bibliothèque d’authentification de Microsoft (MSAL) pour .NET](https://www.nuget.org/packages/Microsoft.Identity.Client) pour gérer les connexions et les jetons.

>MSAL est actuellement en phase de version préliminaire et en tant que tel, il ne doit pas être utilisé dans le code de production. Il est utilisé ici à titre indicatif uniquement. 

**Pas envie de créer une application ?** Utilisez le [démarrage rapide de Microsoft Graph](https://graph.microsoft.io/en-us/getting-started) pour devenir opérationnel rapidement.

Pour télécharger une version de l’exemple de connexion qui utilise le point de terminaison Azure Active Directory et accède à Microsoft Graph à l’aide d’appels REST, voir l’[exemple de connexion d’ASP.NET MVC à Office 365](https://github.com/microsoftgraph/aspnet-connect-rest-sample).


## <a name="prerequisites"></a>Conditions préalables

Voici ce dont vous aurez besoin pour démarrer : 

- Un [compte Microsoft](https://www.outlook.com/) ou un [compte professionnel ou scolaire](http://dev.office.com/devprogram)
- Visual Studio 2015 
- L’[exemple de connexion de Microsoft Graph pour ASP.NET 4.6](https://github.com/microsoftgraph/aspnet-connect-sample) Vous utiliserez le dossier **starter-project** dans les fichiers d’exemple.


## <a name="register-the-application"></a>Inscription de l’application

À cette étape, vous allez inscrire une application sur le portail d’inscription des applications Microsoft. Cette action génère l’ID d’application et le mot de passe que vous utiliserez pour configurer l’application dans Visual Studio.

1. Connectez-vous au [portail d’inscription des applications Microsoft](https://apps.dev.microsoft.com/) en utilisant votre compte personnel, professionnel ou scolaire.

2. Choisissez **Ajouter une application**.

3. Entrez un nom pour l’application, puis choisissez **Créer une application**. 
    
    La page d’inscription s’affiche, répertoriant les propriétés de votre application.

4. Copiez l’ID de l’application. Il s’agit de l’identificateur unique de votre application. 

5. Sous **Secrets de l’application**, choisissez **Générer un nouveau mot de passe**. Copiez le mot de passe à partir de la boîte de dialogue **Nouveau mot de passe créé**.

    Vous utiliserez l’ID de l’application et le mot de passe pour configurer l’application. 

6. Sous **Plateformes**, choisissez **Ajouter une plateforme** > **Web**.

7. Assurez-vous que la case **Autoriser le flux implicite** est cochée, puis entrez *https://localhost:55065/* comme URI de redirection. 

    L’option **Autoriser le flux implicite** active le flux hybride OpenID Connect. Lors de l’authentification, cela permet à l’application de recevoir les informations de connexion (**id_token**) et les artefacts (dans ce cas, un code d’autorisation) qui servent à obtenir un jeton d’accès.

8. Cliquez sur **Enregistrer**.

### <a name="configure-the-project"></a>Configurer le projet

1. Ouvrez le fichier de solution pour le projet de lancement dans Visual Studio.

2. Ouvrez le fichier Web.config du projet.

3. Recherchez les clés de configuration de l’application dans l’élément **appSettings**. Remplacez les valeurs d’espace réservé ENTER_YOUR_CLIENT_ID et ENTER_YOUR_SECRET par les valeurs que vous venez de copier.

L’URI de redirection est l’URL du projet que vous avez enregistré. Les [étendues d’autorisation](https://graph.microsoft.io/en-us/docs/authorization/permission_scopes) demandées autorisent l’application à obtenir des informations de profil utilisateur et à envoyer un e-mail.


## <a name="authenticate-the-user-and-get-an-access-token"></a>Authentifier l’utilisateur et obtenir un jeton d’accès

Dans cette étape, vous ajouterez un code de gestion de jeton et de connexion. Mais tout d’abord, intéressons-nous de plus près au flux d’authentification.

Cette application utilise le flux d’octroi de code d’autorisation avec une identité utilisateur déléguée. Pour une application web, le flux requiert l’ID de l’application, le mot de passe et l’URI de redirection de l’application inscrite. 

Le flux d’authentification comprend les étapes de base suivantes :

1. Rediriger l’utilisateur pour l’authentification et le consentement
2. Obtention d’un code d’autorisation
3. Obtenir le code d’autorisation pour un jeton d’accès
4. Utilisez le jeton d’actualisation pour obtenir un nouveau jeton d’accès lorsque le jeton d’accès expire

L’application utilise l’[intergiciel OWIN OpenID Connect ASP.Net](https://www.nuget.org/packages/Microsoft.Owin.Security.OpenIdConnect/) et la [bibliothèque d’authentification de Microsoft (MSAL) pour .NET](https://www.nuget.org/packages/Microsoft.Identity.Client) pour gérer les connexions et les jetons. Ces ressources gèrent la plupart des tâches d’authentification pour vous.
    
Le projet de lancement déclare déjà les dépendances d’intergiciel et NuGet MSAL suivantes :

  - Microsoft.Owin.Security.OpenIdConnect
  - Microsoft.Owin.Security.Cookies
  - Microsoft.Owin.Host.SystemWeb
  - Microsoft.Identity.Client -Pre

Revenons à la création de l’application.

1. Dans le dossier **App_Start**, ouvrez Startup.Auth.cs. 

1. Remplacer la méthode **ConfigureAuth** par le code suivant. Ce code configure les coordonnées de communication avec Azure Active Directory et configure l’authentification par cookie utilisée par les intergiciels OpenID Connect.

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
  
  La classe de démarrage OWIN (définie dans Startup.cs) appelle la méthode **ConfigureAuth** lorsque l’application démarre, ce qui appelle **app.UseOpenIdConnectAuthentication** pour initialiser l’intergiciel pour la connexion la demande de jeton initiale. L’application demande les étendues d’autorisation suivantes :

  - **openid**, **email**, **profile** pour la connexion
  - **offline\_access** (requis pour obtenir les jetons d’actualisation), **User.Read**, **Mail.Send** pour l’acquisition de jeton
  
  L’objet MSAL **ConfidentialClientApplication** représente l’application et s’acquitte des tâches de gestion des jetons. Il est initialisé avec **SessionTokenCache** (l’exemple d’implémentation de cache de jetons défini dans TokenStorage/SessionTokenCache.cs) où il stocke des informations des jetons. Le cache enregistre des jetons de la session HTTP en cours en fonction de l’ID utilisateur, mais une application de production utilisera sans doute un stockage plus permanent.

Vous allez maintenant ajouter du code à l’exemple de fournisseur d’authentification, qui est conçu pour être facilement remplacé par votre propre fournisseur d’authentification personnalisé. La classe de fournisseur et l’interface ont déjà été ajoutées au projet.

1. Dans le dossier **Helpers**, ouvrez SampleAuthProvider.cs.

1. Remplacez la méthode **GetUserAccessTokenAsync** par l’implémentation suivante, qui utilise MSAL pour obtenir un jeton d’accès.

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

  MSAL recherche un jeton d’accès correspondant qui n’est pas expiré ou proche de le faire dans le cache. Si la bibliothèque ne peut pas trouver de jeton valide, elle utilise le jeton d’actualisation (s’il en existe un valide) pour obtenir un nouveau jeton d’accès. S’il est impossible d’obtenir un jeton d’accès en mode silencieux, MSAL génère une exception **MsalSilentTokenAcquisitionException** pour indiquer qu’une invite utilisateur est nécessaire. 

Vous devez maintenant ajouter du code pour gérer la connexion à l’interface utilisateur et la déconnexion.

1. Dans le dossier **Controllers**, ouvrez AccountController.cs.  

1. Ajoutez les méthodes suivantes à la classe **AccountController**. La méthode **SignIn** indique à l’intergiciel d’envoyer une demande d’autorisation à Azure Active Directory.

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

Vous êtes maintenant prêt à ajouter un code pour appeler Microsoft Graph. 

## <a name="call-microsoft-graph"></a>Appeler Microsoft Graph

Si vous utilisez la bibliothèque Microsoft Graph, poursuivez votre lecture. Si vous utilisez REST, reportez-vous à la section [À l’aide de l’API REST](#using-the-rest-api).

### <a name="using-the-library"></a>À l’aide de la bibliothèque
À cette étape, nous allons nous concentrer sur les classes **SDKHelper**, **GraphService** et **HomeController**. 

 - **SDKHelper** initialise une instance du client **GraphServiceClient** à partir de la bibliothèque avant chaque appel à Microsoft Graph. C’est à ce moment que le jeton d’accès est ajouté à la demande. 
 - **GraphService** crée et envoie des demandes à Microsoft Graph à l’aide de la bibliothèque et traite les réponses.
 - **HomeController** contient des actions qui déclenchent les appels à la bibliothèque en réponse à des événements de l’interface utilisateur.

Le projet de lancement déclare déjà une dépendance pour le package NuGet de la bibliothèque cliente .NET Microsoft Graph :  *Microsoft.Graph*.

1. Cliquez avec le bouton droit de la souris sur le dossier **Helpers** et sélectionnez **Ajouter** > **Classe**. 

1. Nommez la nouvelle classe *SDKHelper* et choisissez **Ajouter**.

1. Remplacez le contenu par le code suivant.

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

  Notez l’appel à **SampleAuthProvider** pour obtenir le jeton lors de l’initialisation du client.

1. Dans le dossier **Models**, ouvrez GraphService.cs. Le service utilise la bibliothèque pour interagir avec Microsoft Graph.

1. Ajoutez l’instruction **using** suivante.

        using Microsoft.Graph;

1. Remplacez *// GetMyEmailAddress* par la méthode suivante. Cela permet d’obtenir l’adresse de messagerie de l’utilisateur actuel. 

        // Get the current user's email address from their profile.
        public async Task<string> GetMyEmailAddress(GraphServiceClient graphClient)
        {

            // Get the current user. 
            // The app only needs the user's email address, so select the mail and userPrincipalName properties.
            // If the mail property isn't defined, userPrincipalName should map to the email for all account types. 
            User me = await graphClient.Me.Request().Select("mail,userPrincipalName").GetAsync();
            return me.Mail ?? me.UserPrincipalName;
        }

  Notez le segment **Select**, qui demande uniquement le renvoi des valeurs **mail** et **userPrinicipalName**. Vous pouvez utiliser **Sélectionner** et **Filtrer** pour réduire la charge des données de réponse.

1. Remplacez *// SendEmail* par les méthodes suivantes pour créer et envoyer le message.

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

1. Dans le dossier **Controllers**, ouvrez HomeController.cs.

1. Ajoutez l’instruction **using** suivante.

        using Microsoft.Graph;
  
1. Remplacez *// Controller actions* par les actions suivantes.

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

Ensuite, vous devez modifier l’exception que le fournisseur d’authentification génère lorsqu’une invite utilisateur est nécessaire.

1. Dans le dossier **Helpers**, ouvrez SampleAuthProvider.cs.

1. Ajoutez l’instruction **using** suivante.

        using Microsoft.Graph;
  
1. Dans le bloc **catch** **GetUserAccessTokenAsync**, modifiez l’exception générée comme suit :

        throw new ServiceException(
            new Error
            {
                Code = GraphErrorCode.AuthenticationFailure.ToString(),
                Message = Resource.Error_AuthChallengeNeeded,
            });

Enfin, vous devez ajouter un appel pour déconnecter le client. 

1. Dans le dossier **Controllers**, ouvrez AccountController.cs. 

1. Supprimez les marques de commentaires des lignes suivantes :

        SDKHelper.SignOutClient();

Vous êtes maintenant prêt à [exécuter l’application](#run-the-app).

### <a name="using-the-rest-api"></a>À l’aide de l’API REST
À cette étape, vous utiliserez les classes **GraphService**, **GraphResources** **HomeController**. 

 - **GraphService** crée et envoie des demandes HTTP à Microsoft Graph et traite les réponses. 
 - **GraphResources** représente les données que l’application envoie à Microsoft Graph et en reçoit. 
 - **HomeController** contient les actions qui déclenchent les appels à Microsoft Graph en réponse aux événements de l’interface utilisateur.

Commencez par définir votre couche d’accès aux données.

1. Dans le dossier **Models**, ouvrez GraphService.cs.

1. Ajoutez les instructions **using** suivantes.

        using Newtonsoft.Json;
        using Newtonsoft.Json.Linq;
        using System.Net;
        using System.Net.Http;
        using System.Net.Http.Headers;
        using System.Text;

1. Remplacez *// GetMyEmailAddress* par la méthode suivante. Cela permet d’obtenir l’adresse de messagerie de l’utilisateur actuel. 

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

1. Remplacez *// SendEmail* par les méthodes suivantes pour créer et envoyer le message.

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

1. Cliquez avec le bouton droit de la souris sur le dossier **Models** et sélectionnez **Ajouter** > **Classe**.

1. Nommez la classe *GraphResources* et choisissez **OK**.

1. Remplacez le contenu par le code suivant.
 
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

1. Dans le dossier **Controllers**, ouvrez HomeController.cs.
  
1. Remplacez *// Controller actions* par les actions suivantes.

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

## <a name="run-the-app"></a>Exécuter l’application
1. Appuyez sur F5 pour créer et exécuter l’application. 

2. Connectez-vous à votre compte personnel, professionnel ou scolaire et accordez les autorisations demandées.

3. Choisissez le bouton **Obtenir l’adresse de messagerie**. Une fois l’opération terminée, l’adresse de messagerie de l’utilisateur connecté s’affiche dans la page.

4. Vous pouvez également modifier la liste des destinataires et l’objet de l’e-mail, puis cliquer sur le bouton **Envoyer un message électronique**. Lorsque le message est envoyé, un message de réussite s’affiche sous le bouton.


## <a name="next-steps"></a>Étapes suivantes
- Testez l’API REST à l’aide de l’[Afficheur Graph](https://graph.microsoft.io/graph-explorer).
- Recherchez des exemples d’opérations courantes dans l’[exemple d’extraits de code Microsoft Graph pour ASP.NET 4.6](https://github.com/microsoftgraph/aspnet-snippets-sample) ou explorez nos autres [exemples ASP.NET](http://aka.ms/aspnetgraphsamples) sur GitHub.

## <a name="see-also"></a>Voir aussi
- [Bibliothèque cliente .NET Microsoft Graph](https://github.com/microsoftgraph/msgraph-sdk-dotnet)
- [Scénario d’authentification d’une application web auprès d’une API web](https://azure.microsoft.com/en-us/documentation/articles/active-directory-authentication-scenarios/#web-application-to-web-api)
- [Intégrer une identité Microsoft et Microsoft Graph dans une application web à l’aide d’OpenID Connect](https://azure.microsoft.com/en-us/documentation/samples/active-directory-dotnet-webapp-openidconnect-v2/)
- [Protocoles Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
- [Jetons Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)
