# <a name="get-started-with-microsoft-graph-in-a-python-app"></a>Prise en main de Microsoft Graph dans une application Python 

Cet article décrit les tâches requises pour obtenir un jeton d’accès d’Azure AD et appeler Microsoft Graph. Il vous guidera dans l’[exemple de connexion Microsoft Graph pour Python](https://github.com/microsoftgraph/python3-connect-rest-sample) et décrit les principaux concepts à mettre en œuvre pour utiliser l’API Microsoft Graph. Cet article explique comment accéder à Microsoft Graph à l’aide d’appels REST directs.

![Capture d’écran d’un exemple de connexion d’une application Python à Office 365](./images/web-screenshot.png)

> **Remarque** Cette procédure pas à pas et l’exemple sur lequel elle est basée utilisent le point de terminaison Azure AD. Vérifiez prochainement les versions mises à jour qui utilisent le point de terminaison Azure AD v2.0.

##  <a name="prerequisites"></a>Conditions préalables

  * Un compte professionnel Office 365. Vous pouvez vous inscrire à [Office 365 Developer](https://msdn.microsoft.com/en-us/office/office365/howto/setup-development-environment#bk_Office365Account) pour accéder aux ressources dont vous avez besoin pour commencer à créer des applications Office 365.
  * L’[exemple de connexion de Microsoft Graph pour Python](https://github.com/microsoftgraph/python3-connect-rest-sample)

## <a name="register-the-application-in-azure-active-directory"></a>Inscription de l’application dans Azure Active Directory

Tout d’abord, vous devez inscrire votre application et définir des autorisations pour utiliser Microsoft Graph. Cela permet aux utilisateurs de se connecter à l’application avec des comptes professionnels ou scolaires.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Dans la barre supérieure, cliquez sur votre compte et dans la liste **Répertoire**, choisissez le client Active Directory où vous souhaitez enregistrer votre application.
3. Cliquez sur **Plus de services** dans le volet de navigation gauche, puis choisissez **Azure Active Directory**.
4. Cliquez sur **Inscriptions de l’application** et choisissez **Ajouter**.
5. Entrez un nom convivial pour l’application, par exemple « MSGraphConnectPython » et sélectionnez « Web app/API » comme **Type d’application**. Comme URL de connexion, saisissez « http://127.0.0.1:8000/connecter/get_token/ ». Cliquez sur **Créer** pour créer l’application.
6. Tout en restant dans le portail Azure, choisissez votre application, cliquez sur **Paramètres** et choisissez **Propriétés**.
7. Recherchez la valeur ID de l’application et copiez-la dans le Presse-papiers.
8. Configurez les autorisations pour votre application :
9. Dans le menu **Paramètres**, choisissez la section **Autorisations requises**, cliquez sur **Ajouter** et **Sélectionner une API**, puis sélectionnez **Microsoft Graph**.
10. Ensuite, cliquez sur Sélectionner des autorisations et sélectionnez **Activer la connexion et lire le profil utilisateur** et **Envoyer un e-mail en tant qu'utilisateur**. Cliquez sur **Sélectionner**, puis sur **Terminé**.
11. Dans le menu **Paramètres**, choisissez la section **Clés**. Entrez une description et sélectionnez une durée pour la clé. Cliquez sur **Enregistrer**.
12. **Important** : copiez la valeur de la clé. Vous ne pourrez pas accéder de nouveau à cette valeur une fois que vous aurez quitté ce volet. Vous utiliserez cette valeur comme question secrète de votre application.

## <a name="redirect-the-browser-to-the-sign-in-page"></a>Redirection du navigateur vers la page de connexion

Votre application doit rediriger le navigateur vers la page de connexion pour démarrer le flux OAuth et obtenir un code d’autorisation. 

Dans l’exemple de connexion, le code suivant (situé dans [*connect/auth_helper.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/auth_helper.py)) génère l’URL vers laquelle l’application redirige l’utilisateur et est transmis à la vue où il peut être utilisé pour la redirection. 

```python
# This function creates the signin URL that the app will
# direct the user to in order to sign in to Office 365 and
# give the app consent.
def get_signin_url(redirect_uri):
  # Build the query parameters for the signin URL.
  params = { 'client_id': client_id,
             'redirect_uri': redirect_uri,
             'response_type': 'code'
           }

  authorize_url = '{0}{1}'.format(authority, '/common/oauth2/authorize?{0}')
  signin_url = authorize_url.format(urlencode(params))
  return signin_url
```

<!--<a name="authCode"></a>-->
## <a name="receive-an-authorization-code-in-your-reply-url-page"></a>Réception d’un code d’autorisation dans la page de votre URL de réponse

Une fois que l’utilisateur se connecte, le navigateur est redirigé vers votre URL de réponse, la fonction ```get_token``` dans [*connect/views.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/views.py), avec un code d’autorisation ajouté à la chaîne de requête comme variable ```code```. 

L’exemple de connexion obtient le code de la chaîne de requête, afin de pouvoir ensuite l’échanger contre un jeton d’accès.

```python
auth_code = request.GET['code']
```

<!--<a name="accessToken"></a>-->
## <a name="request-an-access-token-from-the-token-issuing-endpoint"></a>Demande de jeton d’accès à partir du point de terminaison émettant le jeton

Une fois que vous avez le code d’autorisation, vous pouvez l’utiliser avec l’ID client, la clé et les valeurs de l’URL de réponse obtenues d’Azure Active Directory pour demander un jeton d’accès. 

> **Remarque** La demande doit également spécifier une ressource que vous essayez d’utiliser. Dans le cas de Microsoft Graph, la valeur de la ressource est `https://graph.microsoft.com`.

L’exemple de connexion demande un jeton dans la fonction ```get_token_from_code``` du fichier [*connect/auth_helper.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/auth_helper.py).

```python
# This function passes the authorization code to the token
# issuing endpoint, gets the token, and then returns it.
def get_token_from_code(auth_code, redirect_uri):
  # Build the post form for the token request
  post_data = { 'grant_type': 'authorization_code',
                'code': auth_code,
                'redirect_uri': redirect_uri,
                'client_id': client_id,
                'client_secret': client_secret,
                'resource': 'https://graph.microsoft.com'
              }
              
  r = requests.post(token_url, data = post_data)
  
  try:
    return r.json()
  except:
    return 'Error retrieving token: {0} - {1}'.format(r.status_code, r.text)
```

> **Remarque** La réponse fournit davantage d’informations que le jeton d’accès. Par exemple, votre application peut obtenir un jeton d’actualisation pour demander de nouveaux jetons d’accès sans que l’utilisateur se connecte de nouveau explicitement.

<!--<a name="request"></a>-->
## <a name="use-the-access-token-in-a-request-to-the-microsoft-graph-api"></a>Utilisation du jeton d’accès dans une demande à l’API Microsoft Graph

Avec un jeton d’accès, votre application peut effectuer des demandes authentifiées à l’API Microsoft Graph. Votre application doit ajouter le jeton d’accès à l’en-tête **Authorization** de chaque demande.

L’exemple de connexion envoie un message électronique à l’aide du point de terminaison ```me/microsoft.graph.sendMail``` dans l’API Microsoft Graph. Le code se trouve dans la fonction ```call_sendMail_endpoint``` du fichier [*connect/graph_service.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/graph_service.py). Il s’agit du code qui montre comment ajouter le code d’accès dans l’en-tête Authorization.

```python
# Set request headers.
headers = { 
  'User-Agent' : 'python_tutorial/1.0',
  'Authorization' : 'Bearer {0}'.format(access_token),
  'Accept' : 'application/json',
  'Content-Type' : 'application/json'
}
```

> **Remarque** La demande doit également envoyer un en-tête **Content-Type** avec une valeur acceptée par l’API Graph, par exemple `application/json`.

L’API Microsoft Graph est une API d’unification très puissante, qui peut être utilisée pour interagir avec tous les types de données Microsoft. Consultez la référence API pour découvrir les autres tâches que vous pouvez exécuter avec Microsoft Graph.