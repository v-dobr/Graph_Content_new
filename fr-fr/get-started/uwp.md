# <a name="get-started-with-microsoft-graph-in-a-universal-windows-10-app"></a>Prise en main de Microsoft Graph dans une application Windows 10 universelle

> **Vous voulez créer des applications pour des entreprises ?** Il est possible que votre application ne fonctionne pas si l’entreprise a activé les fonctionnalités de sécurité pour la mobilité en entreprise comme l’<a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-conditional-access-device-policies/" target="_newtab">accès conditionnel des appareils</a>. Dans ce cas, vos clients peuvent rencontrer des erreurs. 

> Pour prendre en charge **toutes les entreprises clientes** dans **tous les scénarios**, utilisez le point de terminaison Azure AD et gérez vos applications avec le [portail de gestion Azure](https://aka.ms/aadapplist). Pour plus d’informations, consultez les [fonctionnalités des points de terminaison Azure AD et Azure AD v2.0](../authorization/auth_overview.md#deciding-between-azure-ad-and-the-v2-authentication-endpoint).

Cet article décrit les tâches requises pour obtenir un jeton d’accès du [point de terminaison d’authentification v2.0](https://graph.microsoft.io/en-us/docs/authorization/converged_auth) et appeler Microsoft Graph. Il vous guide à travers le code de l’[exemple de connexion de Microsoft Graph pour UWP (REST)](https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample) et de l’[exemple de connexion de Microsoft Graph pour UWP (Library)](https://github.com/microsoftgraph/uwp-csharp-connect-sample) pour expliquer les concepts de base que vous devez implémenter dans une application qui utilise Microsoft Graph. Cet article explique également comment accéder à Microsoft Graph à l’aide des appels REST bruts et de la [bibliothèque cliente Microsoft Graph](http://www.nuget.org/packages/Microsoft.Graph/).

Vous pouvez télécharger les deux versions de l’application que vous créerez dans cette procédure pas à pas à partir de ces référentiels GitHub :

* [Exemple de connexion avec Microsoft Graph pour UWP (REST)](https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample)
* [Exemple de connexion avec Microsoft Graph pour UWP (Library)](https://github.com/microsoftgraph/uwp-csharp-connect-sample)

**Pas envie de créer une application ?** Utilisez le [démarrage rapide de Microsoft Graph](https://graph.microsoft.io/en-us/getting-started) pour devenir opérationnel rapidement.

## <a name="sample-user-interface"></a>Exemple d’interface utilisateur

L’exemple contient une interface utilisateur très simple, constituée d’une barre de commandes supérieure, d’un **bouton de connexion**, d’un bouton d’**envoi du message** et d’une zone de texte qui est renseignée automatiquement avec l’adresse e-mail de l’utilisateur connecté, mais qui peut être modifiée.

Le bouton d’**envoi du message** est désactivé lorsque l’utilisateur ne s’est pas connecté :

![Écran montrant le bouton de connexion activé et le bouton d’envoi du message désactivé](images/SignedOut.png)

La barre de commandes supérieure contient un bouton de déconnexion lorsque l’utilisateur s’est connecté :

![Écran montrant l’adresse e-mail de l’utilisateur connecté et le bouton d’envoi du message désactivé](images/SignedIn.png)

Toutes les chaînes de l’exemple d’interface utilisateur sont stockées dans le fichier Resources.resw du dossier Assets.

## <a name="prerequisites"></a>Conditions préalables

Voici ce dont vous aurez besoin pour démarrer : 

- Un [compte Microsoft](https://www.outlook.com/) ou un [compte professionnel ou scolaire](http://dev.office.com/devprogram)
- Visual Studio 2015 
- Soit le [projet de lancement de Microsoft Graph pour UWP (Library)](https://github.com/microsoftgraph/uwp-csharp-connect-sample/tree/master/starter), soit le [projet de lancement de Microsoft Graph pour UWP (REST)](https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample/tree/master/starter). Les deux modèles contiennent des classes vides auxquelles vous ajouterez un code. Ils contiennent également des chaînes de ressources. Pour obtenir l’un de ces projets ou les deux, clonez ou téléchargez l’[exemple de connexion avec Microsoft Graph pour UWP (Library)](https://github.com/microsoftgraph/uwp-csharp-connect-sample) et/ou l’[exemple de connexion avec Microsoft Graph pour UWP (REST)](https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample), et ouvrez la solution dans le dossier **starter**.


## <a name="register-the-app"></a>Inscription de l’application
 
1. Connectez-vous au [portail d’inscription des applications](https://apps.dev.microsoft.com/) en utilisant votre compte personnel, professionnel ou scolaire.
2. Sélectionnez **Ajouter une application**.
3. Entrez un nom pour l’application, puis sélectionnez **Créer une application**.
    
    La page d’inscription s’affiche, répertoriant les propriétés de votre application.
 
4. Sous **Plateformes**, sélectionnez **Ajouter une plateforme**.
5. Sélectionnez **Mobile Platform**.
6. Copiez les valeurs d’ID client (ID d’application) et d’URI de redirection dans le Presse-papiers. Vous devrez saisir ces valeurs dans l’exemple d’application.

    L’ID d’application est un identificateur unique pour votre application. L’URI de redirection est un URI unique fourni par Windows 10 pour chaque application afin de garantir que les messages envoyés à cet URI sont envoyés uniquement à cette application. 

7. Cliquez sur **Enregistrer**.

## <a name="configure-the-project"></a>Configurer le projet

1. Ouvrez le fichier de solution pour le projet de lancement dans Visual Studio.
2. Ouvrez le fichier **App.xaml** du projet et localisez le nœud `Application.Resources`. Remplacez l’ID de l’application et redirigez les espaces réservés d’URI avec les valeurs correspondantes de l’application que vous avez inscrite.


```xml
    <Application.Resources>
        <!-- Add your Client Id here. -->
        <x:String x:Key="ida:ClientID">ENTER_YOUR_CLIENT_ID</x:String>
        <!-- Add your Redirect URI here. -->
        <x:String x:Key="ida:ReturnUrl">ENTER_YOUR_REDIRECT_URI</x:String>
    </Application.Resources>
```

## <a name="install-the-microsoft-authentication-library-(msal)"></a>Installer la bibliothèque d’authentification de Microsoft (MSAL)

La [bibliothèque d’authentification de Microsoft](https://www.nuget.org/packages/Microsoft.Identity.Client) contient des classes et des méthodes qui vous permettent d’authentifier facilement les utilisateurs via le point de terminaison Azure AD v2.0.

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le nom du projet et sélectionnez **Gérer les packages NuGet...**
2. Cliquez sur Parcourir et recherchez Microsoft.Identity.Client.
3. Sélectionnez la dernière version de la bibliothèque d’authentification de Microsoft et cliquez sur **Installer**.

## <a name="install-the-microsoft-graph-client-library"></a>Installer la bibliothèque cliente de Microsoft Graph

> **Remarque** Si vous comptez utiliser des appels REST bruts pour accéder à Microsoft Graph, vous pouvez ignorer cette section.

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le nom du projet et sélectionnez **Gérer les packages NuGet...**
2. Cliquez sur Parcourir et recherchez Microsoft.Graph.
3. Sélectionnez la dernière version de la bibliothèque cliente de Microsoft Graph et cliquez sur **Installer**.

## <a name="create-the-authenticationhelper.cs-class"></a>Créer la classe AuthenticationHelper.cs

Ouvrez le fichier AuthenticationHelper.cs dans le projet de lancement. Ce fichier contient tout le code d’authentification, ainsi que la logique supplémentaire qui stocke les informations utilisateur et force l’authentification uniquement lorsque l’utilisateur s’est déconnecté de l’application. Cette classe contient au moins deux méthodes : `GetTokenForUserAsync` et `Signout`. Si vous utilisez la bibliothèque cliente de Microsoft Graph, vous devez ajouter une troisième méthode : `GetAuthenticatedClient`.

La méthode ``GetTokenHelperAsync`` s’exécute lorsque l’utilisateur s’authentifie et par la suite, chaque fois que l’application effectue un appel à Microsoft Graph.

**Utilisation de déclarations**

***Version de bibliothèque cliente***

Si vous utilisez la bibliothèque cliente de Microsoft Graph, les déclarations suivantes sont nécessaires :

```c#
using System;
using System.Diagnostics;
using System.Net.Http.Headers;
using System.Threading.Tasks;
using Microsoft.Graph;
using Microsoft.Identity.Client;
```

***Version REST***

Si vous utilisez des appels REST bruts pour accéder à Microsoft Graph, les déclarations `using` suivantes dans la classe AuthenticationHelper sont nécessaires :

```c#
using System;
using System.Threading.Tasks;
using Windows.Storage;
using Microsoft.Identity.Client;
```

**Champs de classe**

Les champs suivants sont nécessaires pour les deux versions de la classe AuthenticationHelper :

```c#
// The Client ID is used by the application to uniquely identify itself to the Azure AD v2.0 endpoint.
static string clientId = App.Current.Resources["ida:ClientID"].ToString();
public static string[] Scopes = { "User.Read", "Mail.Send" };
public static PublicClientApplication IdentityClientApp = new PublicClientApplication(clientId);
public static string TokenForUser = null;
public static DateTimeOffset Expiration;
```

Notez que les deux versions utilisent la classe `PublicClientApplication` MSAL pour authentifier l’utilisateur. Le champ `Scopes` stocke les étendues d’autorisation de Microsoft Graph que l’application devra demander lorsque l’utilisateur s’authentifiera. 

***Version de bibliothèque cliente***

Si vous utilisez la bibliothèque cliente, stockez le `GraphServicesClient` en tant que champ afin de le construire une seule fois :

```c#
private static GraphServiceClient graphClient = null;
```

***Version REST***

Si vous utilisez des appels REST, vous devrez stocker des valeurs dans les paramètres d’itinérance de l’application, car vous devrez stocker des informations sur l’utilisateur. (La bibliothèque cliente fournit ces informations dans l’autre version.)

```c#
public static ApplicationDataContainer _settings = ApplicationData.Current.RoamingSettings;
```

**GetTokenForUserAsync**

***Version de bibliothèque cliente***

La méthode `GetTokenForUserAsync` utilise PublicClientApplicationClass et le paramètre ClientId pour obtenir un jeton d’accès pour l’utilisateur. Si l’utilisateur ne s’est pas encore authentifié, il lance l’interface utilisateur d’authentification. Il s’agit de la version de la méthode à utiliser si vous utilisez la bibliothèque cliente.

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

***Version REST***

La version REST de la méthode `GetTokenForUserAsync` en fait un peu plus, car elle doit également stocker des informations sur l’utilisateur connecté.

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

**Signout**

La méthode `Signout` dans les deux versions déconnecte tous les utilisateurs connectés via le `PublicClientApplication` (un seul utilisateur, dans le cas présent) et annule la valeur `TokenForUser`. La version qui utilise la bibliothèque cliente annule également la valeur `GraphServicesClient` stockée, et la version REST annule les valeurs stockées dans les paramètres d’itinérance de l’application.

***Version de bibliothèque cliente***

Il s’agit de la version de bibliothèque cliente de la méthode `Signout`.

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

***Version REST***

Il s’agit de la version REST de la méthode `Signout`.

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

**GetAuthenticatedClient (bibliothèque cliente uniquement)**

Enfin, si vous utilisez la bibliothèque cliente, vous aurez besoin d’une méthode qui crée un `GraphServicesClient`. Cette méthode crée un client qui utilise la méthode `GetTokenForUserAsync` pour chaque appel via le client à Microsoft Graph.

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

## <a name="send-an-email-with-microsoft-graph"></a>Envoi d’un message électronique avec Microsoft Graph

Ouvrez le fichier MailHelper.cs dans votre projet de lancement. Ce fichier contient le code qui crée et envoie un e-mail. Il se compose d’une seule méthode (``ComposeAndSendMailAsync``) qui crée une demande POST et l’envoie au point de terminaison **https://graph.microsoft.com/v1.0/me/microsoft.graph.SendMail**. 

La méthode ``ComposeAndSendMailAsync`` prend trois valeurs de chaîne (``subject``, ``bodyContent`` et ``recipients``) qui lui sont transmises par le fichier MainPage.xaml.cs. Les chaînes ``subject`` et ``bodyContent`` sont stockées, ainsi que toutes les autres chaînes de l’interface utilisateur, dans le fichier Resources.resw. La chaîne ``recipients`` provient de la zone d’adresse dans l’interface de l’application. 

**Version REST**

La version REST de ce fichier requiert les déclarations `using` suivantes :

```c#
using System;
using System.Threading.Tasks;
```

Étant donné que l’utilisateur peut potentiellement transmettre plusieurs adresses, la première tâche consiste à diviser la chaîne ``recipients`` en un ensemble d’objets EmailAddress pouvant ensuite être transmis dans le corps POST de la requête :

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

La deuxième tâche consiste à construire un objet Message JSON valide et à l’envoyer au point de terminaison **me/microsoft.graph.SendMail** via une requête HTTP POST. Dans la mesure où la chaîne ``bodyContent`` est un document HTML, la requête définit la valeur **ContentType** sur HTML. Notez également l’appel vers ``AuthenticationHelper.GetTokenHelperAsync`` pour vous assurer que nous disposons d’un nouveau jeton d’accès à transmettre dans la requête.

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

La classe complète se présente comme suit :

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

**Version de bibliothèque cliente**

La version de bibliothèque cliente de ce fichier requiert les déclarations `using` suivantes :

```c#
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
```

Le code pour envoyer un message avec la bibliothèque cliente est très semblable au code qui utilise des appels REST. Au lieu de créer des objets JSON et de les transférer directement au point de terminaison **SendMail** dans une requête POST HTTP, vous créez des objets équivalents qui sont définis dans la bibliothèque cliente et transférez l’objet `Message` obtenu à la méthode `SendMail` du `GraphServiceClient`. Le client effectue la tâche d’extraction du jeton d’accès et de transfert de la requête au point de terminaison **SendMail**.

La classe complète se présente comme suit :

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

##<a name="create-the-user-interface-in-mainpage.xaml"></a>Créer l’interface utilisateur dans MainPage.xaml

Maintenant que vous avez écrit le code qui effectue la tâche d’authentification de l’utilisateur et d’envoi d’un message via Microsoft Graph, il vous suffit de créer l’interface simple décrite ci-dessus. 

Le fichier MainPage.xaml dans votre projet de lancement comprend déjà tous les XAML dont vous aurez besoin. Il vous suffit d’ajouter le code qui initie l’interface dans le fichier MainPage.xaml.cs. Recherchez ce fichier dans votre projet et ouvrez-le.

Ce fichier contient déjà toutes les déclarations `using` requises pour les versions REST et de bibliothèque cliente de l’exemple.

***Version de bibliothèque cliente***

La version de bibliothèque cliente de l’application crée un `GraphServiceClient` lorsque l’utilisateur s’authentifie. 

Ajoutez ce code dans votre espace de noms pour terminer la version de bibliothèque cliente de la classe MainPage dans MainPage.xaml.cs :

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

***Version REST***

La version REST de cette classe est très semblable à la version de bibliothèque cliente, excepté qu’elle appelle la méthode `GetTokenForUserAsync` directement lorsque l’utilisateur s’authentifie. Elle récupère également les valeurs de l’utilisateur des paramètres d’itinérance de l’application. 

Ajoutez ce code dans votre espace de noms pour terminer la version REST de la classe MainPage dans MainPage.xaml.cs :

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
 
Vous avez effectué les trois étapes nécessaires pour interagir avec Microsoft Graph : inscription de l’application, authentification de l’utilisateur et exécution d’une requête. 

## <a name="run-the-app"></a>Exécuter l’application
1. Appuyez sur F5 pour créer et exécuter l’application. 

2. Connectez-vous à votre compte personnel, professionnel ou scolaire et accordez les autorisations demandées.

3. Choisissez le bouton **Envoyer un message électronique**. Lorsque le message est envoyé, un message de réussite s’affiche sous le bouton.

## <a name="next-steps"></a>Étapes suivantes
- Testez l’API REST à l’aide de l’[Afficheur Graph](https://graph.microsoft.io/graph-explorer).
- Recherchez des exemples d’opérations courantes pour les opérations REST et SDK dans [Exemple d’extraits de code Microsoft Graph UWP (SDK)](https://github.com/microsoftgraph/uwp-csharp-snippets-sample) et l’[exemple d’extraits de code Microsoft Graph UWP (REST)](https://github.com/microsoftgraph/uwp-csharp-snippets-rest-sample), ou explorez nos autres [exemples UWP](https://github.com/microsoftgraph?utf8=%E2%9C%93&query=uwp) sur GitHub.

## <a name="see-also"></a>Voir aussi
- [Bibliothèque cliente .NET Microsoft Graph](https://github.com/microsoftgraph/msgraph-sdk-dotnet)
- [Protocoles Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
- [Jetons Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)

