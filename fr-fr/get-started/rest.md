# <a name="get-started-with-microsoft-graph-and-rest"></a>Prise en main de Microsoft Graph et REST

Cet article décrit les tâches requises pour obtenir un jeton d’accès du point de terminaison Azure AD v2.0 et appeler Microsoft Graph avec des appels REST. Il décrit la séquence des demandes et réponses qu’une application utilise pour authentifier et récupérer des messages électroniques.

Tout d’abord, vous devez inscrire votre application avec Azure Active Directory (Azure AD). 

## <a name="register-the-application"></a>Inscription de l’application

Il existe actuellement deux options pour inscrire votre application avec Azure AD :

  - Inscrire une application pour utiliser le point de terminaison Azure AD v2.0 qui fonctionne à la fois pour les identités personnelles (Microsoft) et les comptes professionnels et scolaires (Azure AD).
  - Inscrire une application pour utiliser le point de terminaison Azure AD qui prend en charge les comptes professionnels et scolaires uniquement.

Cet article part du principe qu’une inscription v2.0 existe, donc vous inscrirez votre application sur le [portail d’inscription des applications](https://apps.dev.microsoft.com/). Suivez les instructions fournies dans la section [Inscription de votre application Microsoft Graph avec le point de terminaison Azure AD v2.0 ](../authorization/auth_register_app_v2.md) pour inscrire votre application. Pour plus d’informations sur l’utilisation du point de terminaison Azure AD, voir [Authentification avec Azure AD](../authorization/app_authorization.md).

> Certaines restrictions s’appliquent lorsque vous utilisez le point de terminaison v2.0. Pour savoir s’il vous convient, voir [Dois-je utiliser le point de terminaison v2.0 ?](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/)

## <a name="authenticate-the-user-and-get-an-access-token"></a>Authentifier l’utilisateur et obtenir un jeton d’accès

L’application décrite dans cet article implémente le [flux d’octroi de code d’autorisation](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols-oauth-code/) pour obtenir les jetons d’accès du point de terminaison Azure AD v2.0, en suivant les [protocoles OAuth 2.0](http://tools.ietf.org/html/rfc6749) standard. Pour des instructions complètes sur les flux pris en charge dans le point de terminaison Azure AD v2.0, voir [Types de point de terminaison v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-flows/).

Avec le flux d’octroi de code d’autorisation, vous obtenez d’abord un code d’autorisation, puis vous échangez le code contre un jeton d’accès.

### <a name="getting-an-authorization-code"></a>Obtention d’un code d’autorisation

La première étape dans le flux d’octroi de code d’autorisation consiste à obtenir un code d’autorisation. Ce code est renvoyé à l’application par le serveur d’autorisation lorsque l’utilisateur se connecte et accepte les autorisations requises par l’application.

Vous demandez un code d’autorisation en envoyant une requête GET au point de terminaison Azure AD v2.0. Cette URL doit être ouverte dans un navigateur pour que l’utilisateur puisse se connecter et fournir son consentement.

#### <a name="construct-the-request"></a>Construire la demande

1 - Commencez par l’URL de base :

```
https://login.microsoftonline.com/<tenant>/oauth2/v2.0/authorize
```

Le segment *client* dans le chemin d’accès contrôle qui peut se connecter à l’application. Les valeurs autorisées sont : *courant*, *organisations*, *consommateurs*et les identificateurs client. Pour plus d’informations, voir [Points de terminaison de protocole](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/#endpoints).

2 - Ajoutez des paramètres de requête à l’URL de base. Ils sont utilisés pour identifier l’application, les autorisations requises et d’autres informations de demande d’authentification. Le tableau suivant décrit certains paramètres courants.

| Paramètre | Descriptions |
|:------|:------|
| client_id | L’ID de l’application généré en inscrivant l’application. Il permet à Azure AD de savoir quelle application demande la connexion. |
| redirect_uri | L’emplacement vers lequel Azure redirige l’utilisateur une fois qu’il a accordé son consentement à l’application. Cette valeur doit correspondre à la valeur de l’**URI de redirection** utilisé lors de l’inscription de l’application. |
| response_type | Le type de réponse attendu par l’application. Cette valeur est `code` pour le flux d’octroi de code d’autorisation. |
| portée | Une liste des [étendues d’autorisation de Microsoft Graph](../authorization/permission_scopes.md) demandées par l’application, séparées par des espaces. Vous pouvez également spécifier des [étendues OpenId Connect](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-scopes/#openid-connect-scopes) pour [l’authentification unique](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols-oidc/).  |
| state | Une valeur incluse dans la demande qui est également renvoyée dans la réponse du jeton, utilisée pour la validation. |

Par exemple, l’URL de requête pour une application nécessitant un accès en lecture au courrier électronique se présente comme suit.

```
GET https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=<app ID>&redirect_uri=http%3A%2F%2Flocalhost/myapp%2F&response_type=code&state=1234&scope=mail.read
```

3 - Redirigez l’utilisateur vers l’URL de connexion. L’utilisateur voit apparaître un écran de connexion qui affiche le nom de l’application. Une fois connecté, l’utilisateur voit apparaître une liste des autorisations requises par l’application. Il est invité à l’autoriser ou la refuser. Si l’utilisateur accepte, le navigateur redirige vers l’URI de redirection avec le code d’autorisation et l’état de la chaîne de requête, comme indiqué dans l’exemple suivant.

```
http://localhost/myapp/?code=AwABAAAA...cZZ6IgAA&state=1234
```

L’étape suivante consiste à échanger le code d’autorisation renvoyé pour un jeton d’accès.

### <a name="getting-an-access-token"></a>Obtention d’un jeton d’accès

Pour obtenir un jeton d’accès, l’application publie des paramètres encodés dans l’URL de demande de jeton (`https://login.microsoftonline.com/common/oauth2/v2.0/token`, par exemple) avec les paramètres suivants.

| Paramètre | Descriptions |
|:------|:------|
| client_id | L’ID de l’application généré en inscrivant l’application. |
| client_secret | La question secrète de l’application générée en inscrivant l’application. |
| code | Le code d’autorisation obtenu à l’étape précédente. |
| redirect_uri | Cette valeur doit être identique à la valeur utilisée dans la demande de code d’autorisation. |
| grant_type | Le type d’octroi utilisé par l’application. Cette valeur est `code` pour le flux d’octroi de code d’autorisation. |
| portée | Une liste des [étendues d’autorisation de Microsoft Graph](../authorization/permission_scopes.md) demandées par l’application, séparées par des espaces. |

L’URL de requête pour notre application, en utilisant le code de l’étape précédente, se présente comme suit.

```
POST https://login.microsoftonline.com/common/oauth2/v2.0/token
Content-Type: application/x-www-form-urlencoded

{
  grant_type=authorization_code
  &code=AwABAAAA...cZZ6IgAA
  &redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
  &resource=https%3A%2F%2Fgraph.microsoft.com%2F
  &scope=mail.read
  &client\_id=<app ID>
  &client\_secret=<app SECRET>
}
```

Le serveur répond avec une charge JSON qui inclut le jeton d’accès.

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
  "token_type":"Bearer",
  "expires_in":"3600",
  "access_token":"eyJ0eXAi...b66LoPVA",
  "scope":"https://graph.microsoft.com/mail.read",
  ...
}
```

Le jeton d’accès se trouve dans le champ `access_token` de la charge JSON. L’application utilise cette valeur pour définir l’en-tête Authorization lors de l’exécution d’appels REST à l’API.

## <a name="call-microsoft-graph"></a>Appeler Microsoft Graph

Une fois que l’application a un jeton d’accès, elle peut appeler Microsoft Graph. Étant donné que cet exemple d’application récupère des messages, elle utilisera une requête HTTP GET au point de terminaison `https://graph.microsoft.com/v1.0/me/messages`.

### <a name="refine-the-request"></a>Améliorer la requête

Les applications peuvent contrôler le comportement des requêtes GET à l’aide de paramètres de requête OData. Il est recommandé que les applications utilisent ces paramètres afin de limiter le nombre de résultats renvoyés et les champs renvoyés pour chaque élément. 

Cet exemple d’application affichera les messages dans un tableau qui indiquera l’objet, l’expéditeur et la date et l’heure de réception du message. Le tableau affiche un maximum de 25 lignes et est trié de sorte que le message reçu en dernier apparaît en haut. L’application utilise les paramètres de requête suivant pour obtenir ces résultats.

- `$select` - Ne spécifie que les champs `subject`, `sender`, et `dateTimeReceived`.
- `$top` - Spécifie un maximum de 25 éléments.
- `$orderby` - Trie les résultats en fonction du champ `dateTimeReceived`.

Cela entraîne la demande suivante.

```
GET https://graph.microsoft.com/v1.0/me/messages?$select=subject,from,receivedDateTime&$top=25&$orderby=receivedDateTime%20DESC
Accept: application/json
Authorization: Bearer eyJ0eXAi...b66LoPVA
```

Maintenant que vous avez vu comment effectuer des appels à Microsoft Graph, vous pouvez utiliser la référence d’API pour créer tous les autres types d’appels que votre application doit effectuer. Toutefois, gardez à l’esprit que votre application doit disposer des autorisations appropriées configurées lors de l’inscription de l’application pour les appels qu’elle effectue.

## <a name="next-steps"></a>Étapes suivantes
- Testez l’API REST plus en détails à l’aide de l’[Explorateur Graph](https://graph.microsoft.io/graph-explorer).

## <a name="see-also"></a>Voir aussi
- [Protocoles Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
- [Jetons Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)
