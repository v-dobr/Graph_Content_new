# <a name="get-started-with-microsoft-graph-in-an-angularjs-app"></a>Prise en main de Microsoft Graph dans une application AngularJS

Cet article décrit les tâches requises pour obtenir un jeton d’accès du point de terminaison Azure AD v2.0 et appeler Microsoft Graph. Il vous guidera dans la construction de l’[exemple de connexion Microsoft pour AngularJS](https://github.com/microsoftgraph/angular-connect-rest-sample) et décrit les principaux concepts à mettre en œuvre pour utiliser Microsoft Graph. Cet article explique comment accéder à l’API Microsoft Graph à l’aide d’appels REST bruts.

L’image suivante présente l’application que vous allez créer. 

![Après la connexion, l’application web affiche le bouton « Envoyer le message ».](./images/angular-connect-sample.png)


**Pas envie de créer une application ?** Utilisez le [démarrage rapide de Microsoft Graph](https://graph.microsoft.io/en-us/getting-started) pour devenir opérationnel rapidement.

Pour télécharger une version de l’exemple de connexion qui utilise le point de terminaison Azure AD, voir [Exemple de connexion Microsoft Graph pour AngularJS](https://github.com/microsoftgraph/angular-connect-rest-sample/releases/tag/last_v1_auth).


## <a name="prerequisites"></a>Conditions préalables

Voici ce dont vous aurez besoin pour démarrer : 

- Un [compte Microsoft](https://www.outlook.com/) ou un [compte professionnel ou scolaire](http://dev.office.com/devprogram)
- [Node.js avec npm](https://nodejs.org/en/download/)
- [Bower](https://bower.io)
- L’[exemple de connexion de Microsoft pour AngularJS](https://github.com/microsoftgraph/angular-connect-rest-sample). Vous allez utiliser le dossier **starter-project** dans les fichiers d’exemple de cette procédure pas à pas.

## <a name="register-the-application"></a>Inscription de l’application
Inscrivez une application sur le portail d’inscription des applications Microsoft. Cette action génère l’ID d’application et le mot de passe que vous utiliserez pour configurer l’application dans Visual Studio.

1. Connectez-vous au [portail d’inscription des applications Microsoft](https://apps.dev.microsoft.com/) en utilisant votre compte personnel, professionnel ou scolaire.

2. Choisissez **Ajouter une application**.

3. Entrez un nom pour l’application, puis choisissez **Créer une application**. 
    
    La page d’inscription s’affiche, répertoriant les propriétés de votre application.

4. Copiez l’ID de l’application. Il s’agit de l’identificateur unique pour votre application que vous utiliserez pour configurer l’application.

5. Sous **Plateformes**, choisissez **Ajouter une plateforme** > **Web**.

7. Assurez-vous que la case **Autoriser le flux implicite** est cochée, puis entrez *https://localhost:8080/login* comme URI de redirection. 

8. Cliquez sur **Enregistrer**.


## <a name="configure-the-project"></a>Configurer le projet
1. Ouvrez le dossier **starter-project** dans les fichiers d’exemple.
2. Dans une invite de commandes, exécutez les commandes suivantes dans le répertoire racine du projet de lancement. Cette procédure installe les dépendances du projet.

    npm install  bower install hello

3. Dans les fichiers du projet de lancement, dans le dossier **public/scripts**, ouvrez config.js.
4. Dans le champ **clientID**, remplacez la valeur de l’espace réservé **ENTER_YOUR_CLIENT_ID** par l’ID d’application que vous venez de copier.

  
## <a name="authenticate-the-user-and-get-an-access-token"></a>Authentifier l’utilisateur et obtenir un jeton d’accès
Dans cette étape, vous ajouterez un code de récupération de jeton et de connexion. Mais tout d’abord, intéressons-nous de plus près au flux d’authentification.

Cette application à une page utilise une implémentation très simple du flux d’octroi implicite, qui requiert l’ID de l’application et l’URI de redirection de l’application inscrite. 

Le flux d’authentification comprend les étapes de base suivantes :

1. Rediriger l’utilisateur pour l’authentification et le consentement.
2. Obtenir un jeton d’accès.

L’application utilise la bibliothèque côté client [HelloJS](https://adodson.com/hello.js) pour authentifier et obtenir des jetons. L’application stocke le jeton d’accès dans le stockage local.
    
   >**Important** La gestion de jeton et d’authentification simple dans ce projet est fournie à titre d’exemple uniquement. Dans une application de production, vous devez mettre au point une méthode de gestion de l’authentification plus fiable (y compris la validation et la gestion de jetons sécurisée).

Revenons à la création de l’application.

1 - Ouvrez aad.js et ajoutez le code suivant. Cela permet de configurer la communication avec le fournisseur d’authentification Azure AD et d’ajouter un détecteur qui stocke la réponse d’authentification contenant le jeton d’accès. (Les références de script à HelloJS sont déjà ajoutées à la vue index.html).

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

2 - Dans graphHelper.js, remplacez *// Initialize the auth request* par le code suivant. Ceci permet de définir les paramètres pour la demande d’authentification.

```
// Initialize the auth request.
hello.init( {
  aad: clientId // from public/scripts/config.js
  }, {
  redirect_uri: redirectUrl,
  scope: graphScopes
});
```

3 - Remplacez *// Sign in and sign out the user* par le code suivant. La fonction de **connexion** utilise HelloJS pour obtenir les informations du jeton. Le détecteur dans aad.js stocke ces informations (y compris le jeton d’accès) dans un stockage local.

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

Vous êtes maintenant prêt à ajouter un code pour appeler Microsoft Graph. 

## <a name="call-microsoft-graph"></a>Appeler Microsoft Graph
L’application appelle Microsoft Graph pour obtenir les informations de l’utilisateur et envoyer un e-mail de la part de l’utilisateur. Ces appels sont lancés à partir du MainController pour répondre aux événements de l’interface utilisateur.

1 - Dans graphHelper.js, remplacez *// Get the profile of the current user* par le code suivant. Cela permet de configurer et d’envoyer la requête GET au point de terminaison */me* et de traiter la réponse.

```
// Get the profile of the current user.
  me: function me() {
    return $http.get('https://graph.microsoft.com/v1.0/me');
},
```
  
2 - Remplacez *// Send an email on behalf of the current user* par le code suivant. Cela permet de configurer et d’envoyer la requête POST au point de terminaison */me/sendMail* et de traiter la réponse.

```
// Send an email on behalf of the current user.
sendMail: function sendMail(email) {
  return $http.post('https://graph.microsoft.com/v1.0/me/sendMail', { 'message' : email, 'saveToSentItems': true });        
}
```

3 - Dans le dossier **public/controllers**, ouvrez mainController.js.

4 - Remplacez *// Set the default headers and user properties* par le code suivant. Ceci permet d’ajouter le jeton d’accès à la requête HTTP, d’appeler **GraphHelper.me** pour obtenir le profil de l’utilisateur actuel et de traiter la réponse.

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

5 - Remplacez *// Send an email on behalf of the current user* par le code suivant. Cela permet de créer le message électronique, d’appeler **GraphHelper.sendMail**, puis de traiter la réponse.

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

6 - Enregistrez toutes vos modifications.

## <a name="run-the-app"></a>Exécuter l’application

1 - Dans une invite de commandes, exécutez la commande suivante dans le répertoire racine du projet de lancement.

```
npm start
```

2 - Dans un navigateur, accédez à *http://localhost: 8080* et choisissez le bouton **Se connecter**.

3 - Connectez-vous et octroyer les autorisations requises. 

4 - Vous pouvez également modifier l’adresse e-mail du destinataire, puis cliquer sur le bouton **Envoyer un message électronique**. Lorsque le message est envoyé, un message de réussite s’affiche sous le bouton. 

## <a name="next-steps"></a>Étapes suivantes
- Testez l’API REST à l’aide de l’[Afficheur Graph](https://graph.microsoft.io/graph-explorer).
- Explorez nos autres [exemples AngularJS](https://github.com/search?utf8=%E2%9C%93&q=angular+sample+user%3Amicrosoftgraph&type=Repositories&ref=searchresults) sur GitHub.


## <a name="see-also"></a>Voir aussi
- [Protocoles Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
- [Jetons Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)