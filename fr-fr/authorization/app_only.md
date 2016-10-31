# <a name="call-microsoft-graph-in-a-service-or-daemon-app"></a>Appel Microsoft Graph dans une application de service ou de démon

Cet article décrit les tâches minimales requises pour connecter votre service à client unique ou votre application de démon à Office 365 et appeler l’API Microsoft Graph.

> Remarque : Cette rubrique décrit l’utilisation d’Azure Active Directory comme fournisseur d’authentification pour votre application. Pour implémenter une application de service ou de démon à l’aide du point de terminaison Azure AD v2.0, consultez la section <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols-oauth-client-creds/" target="_newtab">Protocoles v2.0 - flux des informations d’identification du client OAuth 2.0</a>.

## <a name="register-the-application-in-azure-active-directory"></a>Inscription de l’application dans Azure Active Directory

Avant de commencer à utiliser Office 365, vous devez inscrire votre application sur le [portail Azure](https://portal.azure.com). Gardez les détails suivants à l'esprit :

- Une fois que vous avez inscrit l’application, configurez les autorisations requises par votre application de service ou de démon.
- Prenez note des valeurs suivantes dans l’inscription de votre application Azure. Ces valeurs sont nécessaires pour configurer le flux OAuth dans votre application de service ou de démon.
    * ID d’application (unique pour votre application)
    * Clé d’application ou question secrète de l’application (unique pour votre application)
    * Point de terminaison du jeton OAuth 2.0 de votre application
      * Recherchez cette valeur en cliquant sur *Points de terminaison* en bas du portail de gestion Azure dans la page de votre application. Le point de terminaison ressemblera à `https://login.microsoftonline.com/<tenantId>/oauth2/token`.

## <a name="request-an-access-token-from-the-token-issuing-endpoint"></a>Demande de jeton d’accès à partir du point de terminaison émettant le jeton

Contrairement aux applications client, votre application de service ou de démon ne peut pas avoir un utilisateur qui se connecte et qui autorise votre application. À la place, votre application doit implémenter le flux d’octroi d’informations d’identification du client OAuth 2.0 qui lui permet d’utiliser ses propres informations d’identification, son ID client et une clé d’application pour l’authentification lors de l’appel de Microsoft Graph au lieu d’usurper l’identité d’un utilisateur. Pour plus d’informations sur le flux d’authentification, reportez-vous à [Appels de service à service à l’aide d’informations d’identification de client](https://msdn.microsoft.com/en-us/library/azure/dn645543.aspx).

Effectuez une demande HTTP POST au point de terminaison émettant le jeton avec les paramètres suivants, en remplaçant `<clientId>` et `<clientSecret>` par l’ID client et la clé de votre application, respectivement.

```http
POST https://login.microsoftonline.com/<tenantId>/oauth2/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials
&client_id=<clientId>
&client_secret=<clientSecret>
&resource=https://graph.microsoft.com
```

La réponse inclura un jeton d’accès et des informations sur l’expiration.

```json
{ 
  "token_type": "Bearer",
  "expires_in": "3599",
  "scope": "User.Read",
  "expires_on": "1449685363",
  "not_before": "1449681463",
  "resource": "https://graph.microsoft.com",
  "access_token": "<token>"
}
```

## <a name="use-the-access-token-in-a-request-to-the-microsoft-graph-api"></a>Utilisation du jeton d’accès dans une demande à l’API Microsoft Graph

Avec un jeton d’accès, votre application peut effectuer des demandes authentifiées à l’API Microsoft Graph. Votre application doit ajouter le jeton d’accès à l’en-tête **Authorization** de chaque demande.

Par exemple, une application de service ou de démon peut récupérer tous les utilisateurs d’un client si elle dispose de l’autorisation de *lecture des profils complets de tous les utilisateurs* sélectionnée dans le portail de gestion Azure. 

```http
GET https://graph.microsoft.com/v1.0/users
Authorization: Bearer <token>
```
