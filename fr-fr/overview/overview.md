# <a name="overview-of-microsoft-graph"></a>Présentation de Microsoft Graph

Microsoft Graph expose plusieurs API d’Office 365 et d’autres services de cloud Microsoft via un seul point de terminaison : (**https://graph.microsoft.com**). Microsoft Graph simplifie les requêtes qui seraient autrement plus complexes. 
 
Vous pouvez utiliser Microsoft Graph pour :

- Accéder aux données à partir de plusieurs services de cloud Microsoft, y compris Azure Active Directory, Exchange Online dans le cadre d’Office 365, SharePoint, OneDrive, OneNote et Planner.
- Naviguer entre les entités et les relations.
- Accéder aux analyses et aux renseignements du cloud Microsoft (pour les utilisateurs commerciaux).

**Pile de développement Microsoft Graph**

![Un diagramme qui montre les couches de la pile de développement Microsoft Graph. Dans la partie inférieure se trouve la couche de données, incluant utilisateurs, groupes, fichier, courrier, calendriers, contacts personnels, tâches, contacts organisationnels, personnes, Excel et notes. La couche suivante concerne l’authentification et l’autorisation. Vient ensuite l’environnement de développement de votre choix, notamment les kits de développement logiciel API Android, iOS et Visual Studio Microsoft Graph. La dernière couche représente votre solution qui utilise la technologie de votre choix, notamment .NET, JS, HTML et Ruby et qui est hébergée sur Microsoft Azure ou une autre plate-forme d’hébergement.](./images/MicrosoftGraph_DevStack.png)

<!--<a name="msg_queries"> </a>-->

##<a name="common-microsoft-graph-queries"></a>Requêtes Microsoft Graph courantes

Microsoft Graph expose deux points de terminaison : /v1.0 et /bêta. Le point de terminaison /v1.0 inclut les ressources auxquelles vous pouvez accéder dans votre application de production. Le point de terminaison [/bêta](http://graph.microsoft.io/en-us/docs/api-reference/beta/beta-overview) inclut des API qui sont en cours d’aperçu. Le tableau suivant répertorie certaines requêtes courantes que vous pouvez utiliser pour accéder à l’API de Microsoft Graph.

| **Opération** | **Point de terminaison de service** |
|:--------------------------|:----------------------------------------|
|   GET mon profil |    [`https://graph.microsoft.com/v1.0/me`](/graph-explorer/#?request=me&version=v1.0) |
|   GET mes fichiers | [`https://graph.microsoft.com/v1.0/me/drive/root/children`](/graph-explorer/#?request=me%2Fdrive%2Froot%2Froot%2Fchildren&version=v1.0) |
|   GET ma photo     | [`https://graph.microsoft.com/v1.0/me/photo/$value`](/graph-explorer/#?request=me%2Fphoto%2F%24value&version=v1.0) |
|   GET mon courrier |   [`https://graph.microsoft.com/v1.0/me/messages`](/graph-explorer/#?request=me%2Fmessages&version=v1.0) |
|   GET mon courrier Importance haute | [`https://graph.microsoft.com/v1.0/me/messages?$filter=importance%20eq%20'high'`](/graph-explorer/#?request=me%2Fmessages%3F%24filter%3Dimportance%2520eq%2520'high'&version=v1.0) |
|   GET mon calendrier |   [`https://graph.microsoft.com/v1.0/me/calendar`](/graph-explorer/#?request=me%2Fcalendar&version=v1.0) |
|   GET mon responsable  | [`https://graph.microsoft.com/v1.0/me/manager`](/graph-explorer/#?request=me%2Fmanager&version=v1.0) |
|   GET le dernier utilisateur pour modifier le fichier foo.txt |  [`https://graph.microsoft.com/v1.0/me/drive/root/children/foo.txt/lastModifiedByUser`](/graph-explorer/#?request=me%2Fdrive%2Froot%2Froot%2Fchildren%2Ffoo.txt%2FlastModifiedByUser&version=v1.0) |
|   GET des groupes unifiés dont je suis membre|   [`https://graph.microsoft.com/v1.0/me/memberOf/$/microsoft.graph.group?$filter=groupTypes/any(a:a%20eq%20'unified')`](/graph-explorer/#?request=me%2FmemberOf%2F%24%2Fmicrosoft.graph.group%3F%24filter%3DgroupTypes%2Fany(a%3Aa%2520eq%2520'unified'&version=v1.0)) |
|   GET des utilisateurs dans mon organisation     | [`https://graph.microsoft.com/v1.0/users`](/graph-explorer/#?request=users&version=v1.0) |
|   GET des conversations de groupe |   `https://graph.microsoft.com/v1.0/groups/<id>/conversations`|
|   GET des personnes en relation avec moi    | [`https://graph.microsoft.com/beta/me/people`](/graph-explorer/#?request=me%2Fpeople&version=beta)  |
|   GET les fichiers qui me sont suggérés |  [`https://graph.microsoft.com/beta/me/trendingAround`](/graph-explorer/#?request=me%2FtrendingAround&version=beta) |
|   GET les personnes avec qui je collabore     | [`https://graph.microsoft.com/beta/me/workingWith`](/graph-explorer/#?request=me%2FworkingWith&version=beta) |
|   GET mes tâches    | [`https://graph.microsoft.com/beta/me/tasks`](/graph-explorer/#?request=me%2Ftasks&version=beta) |
|   GET mes notes |  [`https://graph.microsoft.com/beta/me/notes/notebooks`](/graph-explorer/#?request=me%2Fnotes%2Fnotebooks&version=beta) |


>**Remarque :** les API dans le point de terminaison bêta peuvent être modifiées. Nous vous déconseillons de les utiliser dans vos applications de production. 

<!-- <a name="msg_roof"> </a> -->

## <a name="explore-microsoft-graph"></a>Explorer Microsoft Graph

- [Prise en main](../get-started/get-started.md) à l’aide de Microsoft Graph et de la plateforme de votre choix.
- Découvrez les ressources et les opérations que vous pouvez utiliser dans vos applications de production en parcourant la table des matières.
- Affichez un aperçu des nouvelles [API bêta](http://graph.microsoft.io/en-us/docs/api-reference/beta/beta-overview).
- Visitez l’[Explorateur Microsoft Graph](https://graph.microsoft.io/en-us/graph-explorer).

 >  Votre avis compte beaucoup pour nous. Communiquez avec nous sur [Stack Overflow](http://stackoverflow.com/questions/tagged/office365+or+microsoftgraph). Posez vos questions avec les tags [MicrosoftGraph] et [Office 365].



