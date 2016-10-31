# <a name="get-started-with-microsoft-graph-in-a-xamarin-forms-app"></a>Prise en main de Microsoft Graph dans une application de formulaires Xamarin

Cet article décrit les tâches requises pour obtenir un jeton d’accès du [point de terminaison d’authentification v2.0](https://graph.microsoft.io/en-us/docs/authorization/converged_auth) et appeler Microsoft Graph. Il vous guide à travers le code de l’[exemple de connexion de Microsoft Graph pour les formulaires Xamarin](https://github.com/microsoftgraph/xamarin-csharp-connect-sample) pour expliquer les concepts de base que vous devez implémenter dans une application qui utilise Microsoft Graph. Cet article décrit également comment accéder à Microsoft Graph à l’aide de la [bibliothèque cliente de Microsoft Graph](http://www.nuget.org/packages/Microsoft.Graph/).

Il s’agit de l’application que vous allez créer.

| UWP | Android | iOS |
| --- | ------- | ----|
| <img src="images/UWP.png" alt="Connect sample on UWP" width="100%" /> | <img src="images/Droid.png" alt="Connect sample on Android" width="100%" /> | <img src="images/iOS.png" alt="Connect sample on iOS" width="100%" /> |

**Pas envie de créer une application ?** Utilisez le [démarrage rapide de Microsoft Graph](https://graph.microsoft.io/en-us/getting-started) pour devenir opérationnel rapidement ou téléchargez l’[exemple de connexion de Microsoft Graph pour les formulaires Xamarin](https://github.com/microsoftgraph/xamarin-csharp-connect-sample) sur lequel repose cet article.

## <a name="prerequisites"></a>Conditions préalables

Voici ce dont vous aurez besoin pour démarrer : 

- Un [compte Microsoft](https://www.outlook.com/) ou un [compte professionnel ou scolaire](http://dev.office.com/devprogram)
- Visual Studio 2015 
- [Xamarin pour Visual Studio](https://www.xamarin.com/visual-studio)
- Windows 10 (avec [mode de développement](https://msdn.microsoft.com/library/windows/apps/xaml/dn706236.aspx))
- Le [projet de lancement de connexion avec Microsoft Graph pour les formulaires Xamarin](https://github.com/microsoftgraph/xamarin-csharp-connect-sample/tree/master/starter). Ce modèle contient plusieurs classes auxquelles vous ajouterez un code. Il contient également des vues complètes et des chaînes de ressources. Pour obtenir ce projet, clonez ou téléchargez l’[exemple de connexion de Microsoft Graph pour les formulaires Xamarin](https://github.com/microsoftgraph/xamarin-csharp-connect-sample) et ouvrez la solution **XamarinConnect** dans le dossier **starter**. 

Si vous souhaitez exécuter le projet iOS dans cet exemple, vous avez besoin des éléments suivants :

- Le dernier kit de développement logiciel iOS
- La dernière version de Xcode
- Mac OS X Yosemite (10.10) et versions supérieures 
- [Xamarin.iOS](https://developer.xamarin.com/guides/ios/getting_started/installation/mac/)
- Un [agent Xamarin Mac connecté à Visual Studio](https://developer.xamarin.com/guides/ios/getting_started/installation/windows/connecting-to-mac/)


## <a name="register-the-app"></a>Inscription de l’application
 
1. Connectez-vous au [portail d’inscription des applications](https://apps.dev.microsoft.com/) en utilisant votre compte personnel, professionnel ou scolaire.
2. Sélectionnez **Ajouter une application**.
3. Entrez un nom pour l’application, puis sélectionnez **Créer une application**.
    
    La page d’inscription s’affiche, répertoriant les propriétés de votre application.
 
4. Sous **Plateformes**, sélectionnez **Ajouter une plateforme**.
5. Sélectionnez **Mobile Platform**.
6. Copiez l’ID de l’application. Vous devrez saisir cette valeur dans l’exemple d’application.

    L’ID d’application est un identificateur unique pour votre application. L’URI de redirection est un URI unique fourni par Windows 10 pour chaque application afin de garantir que les messages envoyés à cet URI sont envoyés uniquement à cette application. 

7. Cliquez sur **Enregistrer**.

## <a name="configure-the-project"></a>Configurer le projet

1. Ouvrez le fichier de solution pour le projet de lancement dans Visual Studio.
2. Ouvrez le fichier **App.cs** à l’intérieur du projet **XamarinConnect (Portable)** et localisez le champ `ClientId`. Remplacez l’espace réservé de l’ID de l’application par l’ID de l’application que vous avez inscrite.

```c#
public static string ClientID = "ENTER_YOUR_CLIENT_ID";
public static string[] Scopes = { "User.Read", "Mail.Send" };
```
La valeur `Scopes` stocke les étendues d’autorisation de Microsoft Graph que l’application devra demander lorsque l’utilisateur s’authentifiera. Notez que le constructeur de classe `App` utilise la valeur ClientID pour instancier une instance de la classe `PublicClientApplication` MSAL. Vous utiliserez cette classe ultérieurement pour authentifier l’utilisateur.

```c#
IdentityClientApp = new PublicClientApplication(ClientID);
```

## <a name="install-the-microsoft-authentication-library-(msal)"></a>Installer la bibliothèque d’authentification de Microsoft (MSAL)

La [bibliothèque d’authentification de Microsoft](https://www.nuget.org/packages/Microsoft.Identity.Client) contient des classes et des méthodes qui vous permettent d’authentifier facilement les utilisateurs via le point de terminaison d’authentification v2.0.

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet **XamarinConnect (Portable)** et sélectionnez **Gérer les packages NuGet...**
2. Cliquez sur Parcourir et recherchez Microsoft.Identity.Client.
3. Sélectionnez la dernière version de la bibliothèque d’authentification de Microsoft et cliquez sur **Installer**.

Effectuez ces mêmes procédures pour les projets **XamarinConnect.Droid**, **XamarinConnect.iOS** et **XamarinConnect.UWP**. Votre application ne se générera pas si MSAL n’est pas installé dans les quatre projets.

## <a name="install-the-microsoft-graph-client-library"></a>Installer la bibliothèque cliente de Microsoft Graph

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet **XamarinConnect (Portable)** et sélectionnez **Gérer les packages NuGet...**
2. Cliquez sur Parcourir et recherchez Microsoft.Graph.
3. Sélectionnez la dernière version de la bibliothèque cliente de Microsoft Graph et cliquez sur **Installer**.

## <a name="create-the-authenticationhelper.cs-class"></a>Créer la classe AuthenticationHelper.cs

Ouvrez le fichier AuthenticationHelper.cs à l’intérieur du projet **XamarinConnect (Portable)**. Ce fichier contient tout le code d’authentification, ainsi que la logique supplémentaire qui stocke les informations utilisateur et force l’authentification uniquement lorsque l’utilisateur s’est déconnecté de l’application. Cette classe contient au moins trois méthodes : `GetTokenForUserAsync`, `Signout` et `GetAuthenticatedClient`.

La méthode `GetTokenHelperAsync` s’exécute lorsque l’utilisateur s’authentifie et par la suite, chaque fois que l’application effectue un appel à Microsoft Graph.

**Utilisation de déclarations**

Vérifiez que ces déclarations sont en haut du fichier :

```c#
using Microsoft.Graph;
using System;
using System.Diagnostics;
using System.Net.Http.Headers;
using System.Threading.Tasks;
using Microsoft.Identity.Client;
```

**Champs de classe**

Vérifiez que la classe AuthenticationHelper contient ces champs :

```c#
public static string TokenForUser = null;
public static DateTimeOffset expiration;
private static GraphServiceClient graphClient = null;
```

L’exemple stocke le `GraphServicesClient` dans un champ pour que vous ne le créiez qu’une seule fois. Il stocke la `DateTimeOffset` d’expiration du jeton d’accès afin de ne pas récupérer un nouveau jeton tant que le jeton existant ne va pas expirer.

**GetTokenForUserAsync**

La méthode `GetTokenForUserAsync` utilise la `PublicClientApplicationClass` instanciée dans le fichier **App.cs** pour obtenir un jeton d’accès pour l’utilisateur. Si l’utilisateur ne s’est pas encore authentifié, il lance l’interface utilisateur d’authentification.

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

**Signout**

La méthode `Signout` déconnecte tous les utilisateurs connectés via le `PublicClientApplication` (un seul utilisateur, dans le cas présent) et annule la valeur `TokenForUser`. Il annule également la valeur `GraphServicesClient`.

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

Pour finir, vous aurez besoin d’une méthode qui crée un `GraphServicesClient`. Cette méthode crée un client qui utilise la méthode `GetTokenForUserAsync` pour chaque appel via le client à Microsoft Graph.

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

La méthode ``ComposeAndSendMailAsync`` prend trois valeurs de chaîne (``subject``, ``bodyContent`` et ``recipients``) qui lui sont transmises par le fichier MainPage.xaml.cs. Les chaînes ``subject`` et ``bodyContent`` sont stockées, ainsi que toutes les autres chaînes de l’interface utilisateur, dans le fichier AppResources.resx. La chaîne ``recipients`` provient de la zone d’adresse dans l’interface de l’application. 

**Utilisation de déclarations**

Ajoutez ces déclarations en haut du fichier :

```c#
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.Graph;
```

Étant donné que l’utilisateur peut potentiellement transmettre plusieurs adresses, la première tâche consiste à diviser la chaîne ``recipients`` en un ensemble d’objets `EmailAddress` pouvant ensuite être utilisés pour construire la liste d’objets `Recipients` qui seront transmis dans le corps POST de la requête :

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

La deuxième tâche consiste à construire un objet `Message` et à l’envoyer au point de terminaison **me/microsoft.graph.SendMail** via le `GraphServiceClient`. Dans la mesure où la chaîne ``bodyContent`` est un document HTML, la requête définit la valeur **ContentType** sur HTML.

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

Vous avez effectué les trois étapes nécessaires pour interagir avec Microsoft Graph : inscription de l’application, authentification de l’utilisateur et exécution d’une requête. 

## <a name="run-the-app"></a>Exécuter l’application
1. Sélectionnez le projet à exécuter. Si vous sélectionnez l’option Plateforme Windows universelle, vous pouvez exécuter l’exemple sur l’ordinateur local. Si vous souhaitez exécuter le projet iOS, vous devez vous connecter à un [Mac sur lequel les outils de Xamarin](https://developer.xamarin.com/guides/ios/getting_started/installation/windows/connecting-to-mac/) ont été installés. (Vous pouvez également ouvrir cette solution dans Xamarin Studio sur un Mac et exécuter l’exemple directement à partir de là.) Vous pouvez utiliser l’[émulateur Visual Studio pour Android](https://www.visualstudio.com/features/msft-android-emulator-vs.aspx) si vous souhaitez exécuter le projet Android. 

    ![](images/SelectProject.png "Select project in Visual Studio")

2. Appuyez sur F5 pour créer et déboguer l’application. Exécutez la solution et connectez-vous avec votre compte personnel, professionnel ou scolaire.
    > **Remarque** Vous devrez ouvrir le gestionnaire de configurations de build pour vous assurer que les étapes de création et de déploiement sont sélectionnées pour le projet UWP. 

3. Connectez-vous à votre compte personnel, professionnel ou scolaire et accordez les autorisations demandées.

4. Choisissez le bouton **Envoyer un message**. Lorsque l’e-mail est envoyé, un message de réussite s’affiche.

## <a name="next-steps"></a>Étapes suivantes
- Testez l’API REST à l’aide de l’[Afficheur Graph](https://graph.microsoft.io/graph-explorer).
- Recherchez des exemples d’opérations courantes dans la [bibliothèque d’extraits de code du kit de développement Microsoft Graph pour Xamarin.Forms](https://github.com/microsoftgraph/xamarin-csharp-snippets-sample), ou explorez nos autres [exemples Xamarin](https://github.com/microsoftgraph?utf8=%E2%9C%93&query=xamarin) sur GitHub.

## <a name="see-also"></a>Voir aussi
- [Bibliothèque cliente .NET Microsoft Graph](https://github.com/microsoftgraph/msgraph-sdk-dotnet)
- [Protocoles Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
- [Jetons Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)
