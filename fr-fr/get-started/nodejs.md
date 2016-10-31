# <a name="get-started-with-microsoft-graph-in-a-node.js-app"></a>Prise en main de Microsoft Graph dans une application Node.js

Cet article décrit les tâches requises pour obtenir un jeton d’accès du point de terminaison Azure AD v2.0 et appeler Microsoft Graph. Il vous guidera dans la construction de l’[exemple de connexion pour Node.js](https://github.com/microsoftgraph/nodejs-connect-rest-sample) et décrit les principaux concepts à mettre en œuvre pour utiliser Microsoft Graph. Cet article explique comment accéder à l’API Microsoft Graph à l’aide d’appels REST bruts.

L’image suivante présente l’application que vous allez créer. 

![Après la connexion, l’application web affiche le bouton « Envoyer le message ».](./images/web-screenshot.png)


**Pas envie de créer une application ?** Utilisez le [démarrage rapide de Microsoft Graph](https://graph.microsoft.io/en-us/getting-started) pour devenir opérationnel rapidement.

Pour télécharger une version de l’exemple de connexion qui utilise le point de terminaison Azure AD, voir [Exemple de connexion Microsoft Graph pour Node.js](https://github.com/microsoftgraph/nodejs-connect-rest-sample/releases/tag/last_v1_auth).


## <a name="prerequisites"></a>Conditions préalables

Voici ce dont vous aurez besoin pour démarrer : 

- Un [compte Microsoft](https://www.outlook.com/) ou un [compte professionnel ou scolaire](http://dev.office.com/devprogram)
- [Node.js avec npm](https://nodejs.org/en/download/) 
- L’[exemple de connexion de Microsoft pour Node.js](https://github.com/microsoftgraph/nodejs-connect-rest-sample). Vous allez utiliser le dossier **starter-project** dans les fichiers d’exemple de cette procédure pas à pas.

## <a name="register-the-application"></a>Inscription de l’application
Inscrivez une application sur le portail d’inscription des applications Microsoft. Cette action génère l’ID d’application et le mot de passe que vous utiliserez pour configurer l’application dans Visual Studio.

1. Connectez-vous au [portail d’inscription des applications Microsoft](https://apps.dev.microsoft.com/) en utilisant votre compte personnel, professionnel ou scolaire.

2. Choisissez **Ajouter une application**.

3. Entrez un nom pour l’application, puis choisissez **Créer une application**. 
    
    La page d’inscription s’affiche, répertoriant les propriétés de votre application.

4. Copiez l’ID de l’application. Il s’agit de l’identificateur unique de votre application. 

5. Sous **Secrets de l'application**, choisissez **Générer un nouveau mot de passe**. Copiez le mot de passe à partir de la boîte de dialogue **Nouveau mot de passe créé**.

    Vous utiliserez l’ID de l’application et le mot de passe de l’application (clé secrète) pour configurer l’application. 

6. Sous **Plateformes**, choisissez **Ajouter une plateforme** > **Web**.

7. Entrez *http://localhost:3000/login* comme URI de redirection. 

8. Cliquez sur **Enregistrer**.


## <a name="configure-the-project"></a>Configurer le projet
1. Ouvrez le dossier **starter-project** dans les fichiers d’exemple.

1. Dans une invite de commandes, exécutez la commande suivante dans le répertoire racine du projet de lancement. Cette procédure installe les dépendances du projet.

        npm install

1. Dans les fichiers du projet de lancement, ouvrez authHelper.js.


1. Dans le champ **informations d’identification**, remplacez les valeurs d’espace réservé **ENTER\_YOUR\_CLIENT\_ID** et **ENTER\_YOUR\_SECRET**par les valeurs que vous venez de copier.

  
## <a name="authenticate-the-user-and-get-an-access-token"></a>Authentifier l’utilisateur et obtenir un jeton d’accès
Dans cette étape, vous ajouterez un code de gestion de jeton et de connexion. Mais tout d’abord, intéressons-nous de plus près au flux d’authentification.

Cette application utilise le flux d’octroi de code d’autorisation avec une identité utilisateur déléguée. Pour une application web, le flux requiert l’ID de l’application, la clé secrète et l’URI de redirection de l’application inscrite. 

Le flux d’authentification comprend les étapes de base suivantes :

1. Rediriger l’utilisateur pour l’authentification et le consentement
2. Obtention d’un code d’autorisation
3. Obtenir le code d’autorisation pour un jeton d’accès
4. Utilisez le jeton d’actualisation pour obtenir un nouveau jeton d’accès lorsque le jeton d’accès expire

L’application utilise l’intergiciel [oauth](https://www.npmjs.com/package/oauth) pour authentifier et obtenir des jetons. Elle utilise l’intergiciel [cookie-parser](https://www.npmjs.com/package/cookie-parser) pour mettre en cache des informations de jeton dans des cookies. Le code utilisé pour stocker les informations de jeton et y accéder se trouve dans le contrôleur index.js.
    
   >**Important** La gestion de jeton et d’authentification simple dans ce projet est fournie à titre d’exemple uniquement. Dans une application de production, vous devez mettre au point une méthode de gestion de l’authentification plus fiable (y compris la validation et la gestion de jetons sécurisée).

Revenons à la création de l’application.

1. Dans authHelper.js, remplacez la fonction *getTokenFromCode* par le code suivant. Cela permet d’obtenir un jeton d’accès à l’aide d’un code d’autorisation.

        function getTokenFromCode(code, callback) {
            var OAuth2 = OAuth.OAuth2;
            var oauth2 = new OAuth2(
                credentials.client_id,
                credentials.client_secret,
                credentials.authority,
                credentials.authorize_endpoint,
                credentials.token_endpoint
            );

            oauth2.getOAuthAccessToken(
                code,
                {
                    grant_type: 'authorization_code',
                    redirect_uri: credentials.redirect_uri,
                    response_mode: 'form_post',
                    nonce: uuid.v4(),
                    state: 'abcd'
                },
                function (e, accessToken, refreshToken) {
                    callback(e, accessToken, refreshToken);
                }
            );
        }

1. Remplacez la fonction **getTokenFromRefreshToken** par le code suivant. Cela permet d’obtenir un jeton d’accès à l’aide d’un jeton d’actualisation.

        function getTokenFromRefreshToken(refreshToken, callback) {
            var OAuth2 = OAuth.OAuth2;
            var oauth2 = new OAuth2(
                credentials.client_id,
                credentials.client_secret,
                credentials.authority,
                credentials.authorize_endpoint,
                credentials.token_endpoint
            );

            oauth2.getOAuthAccessToken(
                refreshToken,
                {
                    grant_type: 'refresh_token',
                    redirect_uri: credentials.redirect_uri,
                    response_mode: 'form_post',
                    nonce: uuid.v4(),
                    state: 'abcd'
                },
                function (e, accessToken) {
                    callback(e, accessToken);
                }
            );
        }

Vous êtes maintenant prêt à ajouter un code pour appeler Microsoft Graph. 

## <a name="call-microsoft-graph"></a>Appeler Microsoft Graph
L’application appelle Microsoft Graph pour obtenir les informations de l’utilisateur et envoyer un e-mail de la part de l’utilisateur. Ces appels sont lancés à partir du contrôleur index.js pour répondre aux événements de l’interface utilisateur.

1. Ouvrez requestUtil.js.

1. Remplacez la fonction **getUserData** par le code suivant. Cela permet de configurer et d’envoyer la requête GET au point de terminaison */me* et de traiter la réponse.

        function getUserData(accessToken, callback) {
            var options = {
                host: 'graph.microsoft.com',
                path: '/v1.0/me',
                method: 'GET',
                headers: {
                    'Content-Type': 'application/json',
                    Accept: 'application/json',
                    Authorization: 'Bearer ' + accessToken
                }
            };

            https.get(options, function (response) {
                var body = '';
                response.on('data', function (d) {
                    body += d;
                });
                response.on('end', function () {
                    var error;
                    if (response.statusCode === 200) {
                        callback(null, JSON.parse(body));
                    } else {
                        error = new Error();
                        error.code = response.statusCode;
                        error.message = response.statusMessage;
                        // The error body sometimes includes an empty space
                        // before the first character, remove it or it causes an error.
                        body = body.trim();
                        error.innerError = JSON.parse(body).error;
                        callback(error, null);
                    }
                });
            }).on('error', function (e) {
                callback(e, null);
            });
        }

1. Remplacez la fonction **postSendMail** par le code suivant. Cela permet de configurer et d’envoyer la requête POST au point de terminaison */me/sendMail* et de traiter la réponse.

        function postSendMail(accessToken, mailBody, callback) {
            var outHeaders = {
                'Content-Type': 'application/json',
                Authorization: 'Bearer ' + accessToken,
                'Content-Length': mailBody.length
            };
            var options = {
                host: 'graph.microsoft.com',
                path: '/v1.0/me/sendMail',
                method: 'POST',
                headers: outHeaders
            };

            // Set up the request
            var post = https.request(options, function (response) {
                var body = '';
                response.on('data', function (d) {
                    body += d;
                });
                response.on('end', function () {
                    var error;
                    if (response.statusCode === 202) {
                        callback(null);
                    } else {
                        error = new Error();
                        error.code = response.statusCode;
                        error.message = response.statusMessage;
                        // The error body sometimes includes an empty space
                        // before the first character, remove it or it causes an error.
                        body = body.trim();
                        error.innerError = JSON.parse(body).error;
                        // Note: If you receive a 500 - Internal Server Error
                        // while using a Microsoft account (outlook.com, hotmail.com or live.com),
                        // it's possible that your account has not been migrated to support this flow.
                        // Check the inner error object for code 'ErrorInternalServerTransientError'.
                        // You can try using a newly created Microsoft account or contact support.
                        callback(error);
                    }
                });
            });
            
            // write the outbound data to it
            post.write(mailBody);
            // we're done!
            post.end();

            post.on('error', function (e) {
                callback(e);
            });
        }
  
1. Ouvrez emailer.js.

1. Remplacez la fonction **wrapEmail** par le code suivant. Cela permet de créer la charge utile qui représente le message électronique à envoyer.

        function wrapEmail(content, recipient) {
            var emailAsPayload = {
                Message: {
                    Subject: 'Welcome to Office 365 development with Node.js and the Office 365 Connect sample',
                    Body: {
                        ContentType: 'HTML',
                        Content: content
                    },
                    ToRecipients: [
                        {
                            EmailAddress: {
                                Address: recipient
                            }
                        }
                    ]
                },
                SaveToSentItems: true
            };
            return emailAsPayload;
        }

## <a name="run-the-app"></a>Exécuter l’application

1. Dans une invite de commandes, exécutez la commande suivante dans le répertoire racine du projet de lancement.


        npm start

1. Dans un navigateur, accédez à *http://localhost: 3000* et choisissez le bouton **Se connecter à Office 365**.

1. Connectez-vous et octroyez les autorisations requises. 

1. Vous pouvez également modifier l’adresse e-mail du destinataire, puis cliquer sur le bouton **Envoyer un message électronique**. Lorsque le message est envoyé, un message de réussite s’affiche sous le bouton. 

## <a name="next-steps"></a>Étapes suivantes
- Testez l’API REST à l’aide de l’[Afficheur Graph](https://graph.microsoft.io/graph-explorer).
- Explorez nos autres [exemples Node.js](https://github.com/search?utf8=%E2%9C%93&q=node+sample+user%3Amicrosoftgraph&type=Repositories&ref=searchresults) sur GitHub.


## <a name="see-also"></a>Voir aussi
- [Protocoles Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
- [Jetons Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)