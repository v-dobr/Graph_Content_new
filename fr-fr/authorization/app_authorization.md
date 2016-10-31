
# <a name="microsoft-graph-app-authentication-using-azure-ad"></a>Authentification de l’application Microsoft Graph en utilisant Azure AD

Cet article fournit une description détaillée d’un exemple de flux d’autorisation et d’authentification pour une application Microsoft Graph. Cet exemple utilise Azure Active Directory (Azure AD) et le fournisseur d’authentification, et le <a href="https://msdn.microsoft.com/en-us/library/azure/dn645542.aspx" target="_newtab">flux d’octroi de code d’autorisation</a> comme flux d’authentification. Cet exemple montre comment utiliser Azure AD dans une application Microsoft Graph pour authentifier un utilisateur, obtenir un jeton d’accès et renouveler un jeton d’accès à l’aide d’un jeton d’actualisation.

Pour les flux d’octroi de code, le processus d’authentification peut être divisé en deux étapes de base :

1. Demande d’un code d’autorisation
2. Utilisation d’un code d’autorisation pour demander un jeton d’accès et un jeton d’actualisation. 

vous pouvez utiliser le jeton d’actualisation pour acquérir un nouveau jeton d’accès lorsque le jeton d’accès en cours expire.
 
##<a name="authenticate-a-user-and-get-app-authorized"></a>Authentification d’un utilisateur et obtention d’une autorisation pour l’application

Pour obtenir l’autorisation de votre application, vous devez d’abord authentifier l’utilisateur. Pour cela, vous redirigez l’utilisateur vers le point de terminaison d’autorisation Azure Active Directory (AD Azure), ainsi que les informations relatives à votre application, pour vous connecter à son compte professionnel ou scolaire. Une fois que l’utilisateur est connecté et accepte les autorisations requises par votre application (s’il ne l’a pas déjà fait), votre application reçoit un code d’autorisation nécessaire pour obtenir un jeton d’accès OAuth.

> 
  **Remarque** :  vous pouvez effectuer cette opération en appelant des méthodes sur la bibliothèque [Azure AD Authentication Library (ADAL)](https://msdn.microsoft.com/en-us/library/azure/jj573266.aspx). Pour plus d’informations sur le flux d’autorisation, reportez-vous à la rubrique relative au [flux d’octroi de code d’autorisation](https://msdn.microsoft.com/en-us/library/azure/dn645542.aspx).

L’autorisation d’une application commence par l’envoi d’une demande GET HTTPS à l’aide de l’URL suivante :
 
```GET https://login.microsoftonline.com/common/oauth2/authorize?response_type=code&redirect_uri=<uri>&client_id=<id>```

**Paramètres de chaîne de requête requis**

| Nom du paramètre  | Valeur  | Description                                                                                            |
|:----------------|:-------|:-------------------------------------------------------------------------------------------------------|
| *client_id*     | string | ID client créé pour votre application. Il s’agit de la valeur **ID CLIENT** de votre application, définie dans le registre d’application du client Azure.                                                                  |
| *response_type* | string | Spécifie le type de réponse demandé. Dans une demande d’octroi de code d’autorisation, la valeur doit être un code. |
| *redirect_uri*  | string | URL de redirection vers laquelle est renvoyé le navigateur à la fin de l’authentification.  Cette valeur doit correspondre à la valeur **URL DE RÉPONSE** préconfigurée de l’application.                        |
 


Voici un exemple de demande de ce type telle qu’elle est implémentée dans une application en cours d’exécution :


```GET https://login.microsoftonline.com/common/oauth2/authorize?response_type=code&redirect_uri=http%3a%2f%2flocalhost:1339/auth/azureoauth/callback&client_id=8b8539cd-7b75-427f-bef1-4a6264fd4940``` 

Cette demande renvoie une réponse `200 OK` et présente la page de connexion au compte Azure AD. 

Une fois que l’utilisateur s’est connecté avec des informations d’identification valides et accepte les autorisations accordées pour l’application, la page de connexion envoie une demande `POST` à Azure. Si cette demande réussit, Azure répond avec un message `302 Found` pour transférer l’appel vers l’URI de redirection de l’application, pour que cette dernière reçoive le jeton d’accès requis. L’URI transféré, spécifié dans l’en-tête `Location` de la réponse, correspond à la valeur URL DE RÉPONSE de l’application, avec deux paramètres de requête,`code=...` et `session_state=...`, qui lui sont ajoutés. L’exemple suivant montre un extrait de réponse : 

```no-highlight 
HTTP/1.1 302 Found
Cache-Control: no-cache, no-store
Pragma: no-cache
Content-Type: text/html; charset=utf-8
Expires: -1
Location: http://localhost:1339/auth/azureoauth/callback?code=AAABAAAAvPM...&session_state=a9556cd3-cae6-4bc9-bf51-672f7b79b7c6
Server: Microsoft-IIS/8.5
P3P: CP="DSP CUR OTPi IND OTRi ONL FIN"

..... 
```

Dans cet exemple, l’URL DE RÉPONSE de l’application est `http://localhost:1339/auth/azureoauth/callback`. Lors du traitement de cette réponse, vous devez extraire la valeur de paramètre `code` et l’utiliser pour acquérir les jetons d’actualisation et d’accès OAuth 2.0 initiaux (voir la section suivante).

> La réponse `302 Found` ci-dessus est différente de la réponse `302 Found` que vous obtiendriez si vous commenciez le processus de connexion par rapport à l’URL `https://login.windows.net/<tenantId>/oauth2/authorize?...`. Dans ce dernier cas, la réponse `302 Found` transfère votre demande à `login.microsoftonline.com`.
 
<!---<a name="msg_get_app_authenticated"> </a> -->

##<a name="acquire-an-access-token"></a>Acquisition d’un jeton d’accès
Pour accéder aux ressources de Microsoft Graph, votre application doit inclure un jeton d’accès OAuth 2.0 valide dans toutes les demandes HTTP. Vous pouvez obtenir le jeton d’accès à l’aide de la demande POST suivante :

```no-highlight 
POST https://login.microsoftonline.com/common/oauth2/token HTTP/1.1
content-type : application/x-www-form-urlencoded
content-length : 144
```
 
Cette demande requiert une charge codée dans l’URL au format suivant :
 
```no-highlight 
grant_type=authorization_code
&redirect_uri=<uri>
&client_id=<id>
&client_secret=<secret_key>
&code=<code>
&resource=https%3A%2F%2Fgraph.microsoft.com%2F
```

**Paramètres de chaîne de requête requis**

| Nom du paramètre  | Valeur  | Description                                                                                            |
|:----------------|:-------|:-------------------------------------------------------------------------------------------------------|
| *client_id*     | string | ID client créé pour votre application.  |
| *client_secret*  | string | Clé créée pour votre application. Cette valeur est identique à la valeur de la section **Clés** de la page de configuration de l’application sur le portail de gestion Azure.|
| *redirect_uri*  | string | URL de redirection vers laquelle est renvoyé le navigateur à la fin de l’authentification.  |
| *code*  | string | Code d’autorisation. Valeur du paramètre de requête `code` renvoyée depuis la réponse à la demande d’autorisation. |
| *ressource*   | string | Ressource à laquelle vous souhaitez accéder. Pour appeler l’API Microsoft Graph, définissez cette valeur de paramètre sur « https://graph.microsoft.com/ ».|

L’extrait de code suivant montre un exemple de la charge de demande utilisée pour acquérir le jeton d’accès OAuth 2.0 initial :

```no-highlight  
grant_type=authorization_code&
redirect_uri=http%3A%2F%2Flocalhost%3A1339%2Fauth%2Fazureoauth%2Fcallback&
client_id=8b8539cd-7b75-427f-bef1-4a6264fd4940&client_secret=PJW3dznGfyNSm3rM9aHeXWGKsTMepKXT1Eqy45XXdU4%3D&
code=AAABAAAAvPM1KaPlrEqdFSBzjqfTGBLRVQc6BtQmJ_9DQZUg8Tb7TJgTmbTE7AHM93qB5EKc4Bf-bOgzt3mebAywK-09X1uBHwOluuqSWfd9LU2HHgZtxcZFIYI5UL7L1UEvhrJRvX2iHhfz9ZSRMZMVL55n_K79gCOxtSATeCUw52zPk5ZaQ87Y42SCLsRZN4Y_zddhD3mMpkObiHVT8HzfhBUiT0oX0e-Q439vkbZoKiq1HaqMR3IPHiCXDbPPH5u7a4NTe5xAhh-o2MUIe6s4Xqql86sv1-IwyroOJJMueGUarkfbgwqmYL9Tm-jWab8o-BAK_plVsN73GU8cXO8ts30wa2YmNR5ZxSkw8oiB4mSZwGzGQlch55DRnucDs0SZBgj5etGi3SeXv5jhKlDU2S0bAPnGxF3QAH0N_zBpfakETVlcsHKi714u-tn9da6aTPQsE0IYKTAYgxjTMei6zfRFvCZi-tKdFR6X9TvvmF2iPdGQGWKeLw8CMWUzU8VmOhiHc0aBKG6RaXAOTM067J_9WKYPxMopcytD2z8HVkL1QhggAA&
resource=https%3A%2F%2Fgraph.microsoft.com%2F
```

Lorsque cette demande aboutit, une réponse `200 OK` est renvoyée. Un exemple est indiqué comme suit :

```no-highlight  
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Expires: -1
Content-Length: 2978
Access-Control-Allow-Origin: *

{
    "token_type":"Bearer",
    "expires_in":"3599",
    "expires_on":"1426551729",
    "not_before":"1426547829",
    "resource":"https://graph.microsoft.com/",
    "access_token":"eyJ0eXAiOiJKV1QiLCJhb...",
    "refresh_token":"AAABAAAAvPM1KaPlrEqd...",
    "scope": "Calendar.ReadWrite Directory.Read.All Files.ReadWrite Group.ReadWrite.All Mail.ReadWrite Mail.Send User.ReadBasic.All",
    "id_token":"eyJ0eXAiOiJKV1QiLCJhbGci..."
}
```

 
Le corps de la réponse est une chaîne au format JSON contenant le jeton d’accès (`access_token`). Vous devez fournir ce jeton aux demandes HTTP résultantes pour accéder aux ressources de Microsoft Graph. 

La valeur de la propriété `scope` doit correspondre aux autorisations accordées pour l’application pendant l’inscription de l’application.

Le jeton d’accès reste valide pendant l’intervalle de temps spécifié (`3599` secondes dans l’exemple ci-dessus ou 1 heure) à partir de la date d’émission, comme spécifié dans la propriété `expires_in`. Le résultat contient également un jeton d’actualisation (`refresh_token`) qui doit être utilisé pour renouveler un jeton d’accès arrivant à expiration ou ayant expiré. 

Dans n’importe quel code de production, votre application doit vérifier l’expiration de ces jetons et renouveler le jeton d’accès arrivant à expiration avant l’expiration du jeton d’actualisation. 
-->

<!---<a name="msg_renew_access_token using refresh token"> </a> -->

##<a name="renew-expiring-access-token-using-refresh-token"></a>Renouvellement du jeton d’accès arrivant à expiration à l’aide du jeton d’actualisation
Pour actualiser un jeton d’accès qui a expiré, utilisez une demande POST semblable à l’exemple suivant (à condition que le jeton d’actualisation n’ait pas expiré) :

```no-highlight  
POST https://login.microsoftonline.com/common/oauth2/token HTTP/1.1
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 897


grant_type=refresh_token
&redirect_uri=http%3A%2F%2Flocalhost%3A1339%2Fauth%2Fazureoauth%2Fcallback
&client_id=8b8539cd-7b75-427f-bef1-4a6264fd4940
&client_secret=PJW3dznGfyNSm3rM9aHeXWGKsTMepKXT1Eqy45XXdU4%3D
&refresh_token=AAABAAAAvPM1KaPlrEqdFSBzjqfTGM74--...
&resource=https%3A%2F%2Fgraph.microsoft.com%2F
```

**Paramètres de chaîne de requête requis**

| Nom du paramètre  | Valeur  | Description                                                                                                                                         |
|:----------------|:-------|:----------------------------------------------------------------------------------------------------------------------------------------------------|
| *client_id*     | string | ID client créé pour votre application.  |
| *redirect_uri*  | string | URL de redirection vers laquelle est renvoyé le navigateur à la fin de l’authentification. Celle-ci doit correspondre à la valeur *redirect_uri* utilisée dans la première demande. |
| *client_secret* | string | Une des valeurs clés créées pour votre application.                                                                                                     |
| *refresh_token* | string | Jeton d’actualisation que vous avez reçu précédemment.    |
| *ressource*      | string | Ressource à laquelle vous souhaitez accéder.|

Notez que cette demande est presque identique à la demande d’acquisition de jeton initiale. Il existe deux différences dans la charge de la demande : le paramètre `grant_type` a maintenant la valeur de `refresh_token` (au lieu de `code`).
 
La réponse réussie renvoie la charge d’une chaîne JSON semblable à la sortie suivante :

```no-highlight 
{
    "token_type":"Bearer",
    "expires_in":"3600",
    "expires_on":"1426561346",
    "not_before":"1426557446",
    "resource":"https://graph.microsoft.com/",
    "access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOi...", 
    "refresh_token":"AAABAAAAvPM1KaPlrEqdFSBzj...",
   "scope":"Graph.Read",
    "pwd_exp":"6553342",
    "pwd_url":"https://portal.microsoftonline.com/ChangePassword.aspx"
}
```
 
Mis à part la propriété `id_token` manquante, le corps de cette réponse comporte une sémantique et une syntaxe identiques à celles de la réponse d’acquisition de jeton initiale. Les durées de vie des nouvelles valeurs `access_token` et `refresh_token` renvoyées sont prolongées. Le nouveau délai d’expiration pour le jeton d’accès est le nombre de secondes, spécifié dans la valeur `expires_in`, à partir du moment où la demande d’actualisation du jeton a été soumise. 
 
Lorsque le jeton d’actualisation arrive à expiration, vous ne pouvez pas renouveler n’importe quel jeton d’accès ayant expiré à l’aide de la demande POST qui vient d’être décrite. Au lieu de cela, vous devez redémarrer le processus d’autorisation et d’authentification de l’application.



