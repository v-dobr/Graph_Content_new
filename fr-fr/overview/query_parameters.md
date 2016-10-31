# <a name="microsoft-graph-optional-query-parameters"></a>Paramètres de requête facultatifs Microsoft Graph
Microsoft Graph fournit plusieurs paramètres de requête facultatifs que vous pouvez utiliser pour spécifier et contrôler la quantité de données renvoyées dans une réponse. Microsoft Graph prend en charge les options de requête suivantes. 

|Nom|Valeur|Description|
|:---------------|:--------|:-------|
|$select|string|Liste de propriétés séparées par des virgules à inclure dans la réponse.|
|$expand|string|Liste de relations séparées par des virgules à développer et inclure dans la réponse.  |
|$orderby|string|Liste de propriétés séparées par des virgules qui sont utilisées pour trier l’ordre des éléments dans la collection de réponses.|
|$filter|string|Filtre la réponse en fonction d’un ensemble de critères.|
|$top|int|Le nombre d’éléments à renvoyer dans un ensemble de résultats.|
|$skip|int|Le nombre d’éléments à ignorer dans un ensemble de résultats.|
|$skipToken|string|Jeton de pagination qui est utilisé pour obtenir l’ensemble de résultats suivant.|
|$count|none|Une collection et le nombre d'éléments dans la collection.|

Ces paramètres sont compatibles avec la [langue de la requête OData V4](http://docs.oasis-open.org/odata/odata/v4.0/errata03/os/complete/part2-url-conventions/odata-v4.0-errata03-os-part2-url-conventions-complete.html#_Toc453752356).

>  **Remarque** : sur le point de terminaison **bêta** de Microsoft Graph, vous pouvez omettre le préfixe **$** pour une expérience plus simple. Par exemple, au lieu de **$expand**, vous pouvez utiliser **expand**. Pour plus d’informations et d’exemples, voir [Prise en charge des paramètres de requête sans préfixes $ dans Microsoft Graph](http://dev.office.com/queryparametersinMicrosoftGraph).  

**Codage des paramètres de requête**

- Si vous testez des paramètres de requête dans l’[explorateur Microsoft Graph](https://graph.microsoft.io/en-us/graph-explorer#), vous pouvez simplement copier et coller les exemples ci-dessous sans appliquer de codage des URL à la chaîne de requête. L'exemple suivant fonctionne bien _dans l'explorateur Graph_ sans codage des caractères d'espace et de guillemet :
```http
GET https://graph.microsoft.com/v1.0/me/messages?$filter=from/emailAddress/address eq 'jon@contoso.com'
``` 
- En règle générale, lorsque vous spécifiez des paramètres de requête _dans votre application_, veillez à encoder correctement les caractères qui sont [réservés pour des significations particulières dans un URI](https://tools.ietf.org/html/rfc3986#section-2.2). Par exemple, encodez les caractères d'espace et de guillemet dans le dernier exemple, comme indiqué :
```http
GET https://graph.microsoft.com/v1.0/me/messages?$filter=from/emailAddress/address%20eq%20%27jon@contoso.com%27
```

### <a name="$select"></a>$select
Pour spécifier un autre ensemble de propriétés à renvoyer que l’ensemble par défaut fourni par le Graph, utilisez l’option de requête **$select**. L’option **$select** permet de choisir un sous-ensemble ou un sur-ensemble de l’ensemble renvoyé par défaut. Par exemple, lorsque vous récupérez vos messages, vous pouvez indiquer que seules les propriétés **from** et **subject** des messages sont renvoyées.

```http
GET https://graph.microsoft.com/v1.0/me/messages?$select=from,subject
```

<!--For example, when retrieving the children of an item on a drive, you want to select that only the **name** and **size** properties of items are returned.

```http
GET https://graph.microsoft.com/v1.0/me/drive/root/children?$select=name,size
```

By submitting the request with the `$select=name,size` query string, the objects
in the response will only have those property values included. 


```json
{
  "value": [
    {
      "id": "13140a9sd9aba",
      "name": "Documents",
      "size": 1024
    },
    {
      "id": "123901909124a",
      "name": "Pictures",
      "size": 1012010210
    }
  ]
}
```--> 

## <a name="$expand"></a>$expand

Dans les demandes d’API Microsoft Graph, les navigations vers un objet ou une collection de l'élément référencé ne sont pas développées automatiquement. En effet, cela permet de réduire le trafic réseau et le temps nécessaire à la génération d’une réponse du service. Toutefois, dans certains cas, vous pouvez inclure ces résultats dans une réponse.

Vous pouvez utiliser le paramètre de chaîne de requête **$expand** pour indiquer à l’API de développer une collection ou un objet enfant et d’inclure ces résultats.

Par exemple, pour récupérer les informations du lecteur racine et les éléments enfants de niveau supérieur dans un lecteur, vous utilisez le paramètre **$expand**. Cet exemple utilise également une instruction **$select** pour renvoyer uniquement les propriétés _id_ et _name_ des éléments enfants.

```http
GET https://graph.microsoft.com/v1.0/me/drive/root?$expand=children($select=id,name)
```

>  **Remarque** : le nombre maximal d’objets développés pour une demande est de 20. 

> En outre, si vous effectuez votre requête sur la ressource [user](http://graph.microsoft.io/en-us/docs/api-reference/v1.0/resources/user), vous pouvez utiliser **$expand** pour obtenir les propriétés d’un seul objet enfant ou collection à la fois. 

L'exemple suivant obtient des objets **user**, chacun comportant jusqu'à 20 objets **directReport** dans la collection **directReports** développée :
```http
GET https://graph.microsoft.com/v1.0/users?$expand=directReports
```
Certaines ressources peuvent également avoir une limite, vérifiez donc toujours les erreurs possibles.


<!---The following shows a sample result that is returned in the response body.-->


## <a name="$orderby"></a>$orderby

Pour spécifier l’ordre de tri des éléments renvoyés de l’API Microsof Graph, utilisez l’option de requête **$orderby**. 

Par exemple, pour renvoyer les utilisateurs de l’organisation classés par nom d’affichage, la syntaxe est la suivante :

```http
GET https://graph.microsoft.com/v1.0/users?$orderby=displayName
``` 

Vous pouvez également trier par entités de type complexe. L’exemple suivant obtient les messages et les trie en fonction du champ **address** de la propriété **from**, qui est de type complexe **emailAddress** :

```http
GET https://graph.microsoft.com/v1.0/me/messages?$orderby=from/emailAddress/address
``` 

Pour trier les résultats dans l’ordre croissant ou décroissant, ajoutez `asc` ou `desc` au nom de champ, séparé par un espace, par exemple, `?$orderby=name%20desc`.

 >  **Remarque** : si vous effectuez une requête sur la ressource [user], **$orderby** ne peut pas être combiné à des expressions de filtre.

## <a name="$filter"></a>$filter
Pour filtrer les données de réponse basées sur un ensemble de critères, utilisez l’option de requête **$filter**. Par exemple, pour renvoyer les utilisateurs de l’organisation filtrés par nom d’affichage commençant par « Garth », la syntaxe est la suivante :

```http
GET https://graph.microsoft.com/v1.0/users?$filter=startswith(displayName,'Garth')
```

Vous pouvez également filtrer par entités de type complexe. L’exemple suivant renvoie les messages contenant le champ **address** de la propriété **from** égal à « jon@contoso.com ». La propriété **from** est du type complexe **emailAddress**.

```http
GET https://graph.microsoft.com/v1.0/me/messages?$filter=from/emailAddress/address eq 'jon@contoso.com'
``` 

## <a name="$top"></a>$top
Pour spécifier le nombre maximal d’éléments à renvoyer dans un ensemble de résultats, utilisez l’option de requête **$top**. L’option de requête **$top** identifie un sous-ensemble dans la collection. Ce sous-ensemble est formé en sélectionnant uniquement les N premiers éléments de l’ensemble, où N est un nombre entier positif spécifié par cette option de requête. Par exemple, pour renvoyer les cinq premiers messages de la boîte aux lettres de l’utilisateur, la syntaxe est la suivante :

```http
GET https://graph.microsoft.com/v1.0/me/messages?$top=5
```

## <a name="$skip"></a>$skip
Pour définir le nombre d’éléments à ignorer avant de les récupérer dans une collection, utilisez l’option de requête **$skip**. Par exemple, pour renvoyer des événements triés par date de création, en commençant par le 21e événement, la syntaxe est la suivante.

```http
GET  https://graph.microsoft.com/v1.0/me/events?$orderby=createdDateTime&$skip=20
```

## <a name="$skiptoken"></a>$skipToken
Pour demander la deuxième page et les pages suivantes des données de Graph, utilisez l’option de requête **$skipToken**. L’option de requête **$skipToken** est une option fournie dans les URL renvoyées par le Graph lorsque le Graph a renvoyé un sous-ensemble partiel des résultats, généralement en raison de la pagination côté serveur. Il identifie le point d’une collection où le serveur a terminé d’envoyer des résultats et est retransmis au Graph pour indiquer l’endroit où il doit reprendre l’envoi des résultats. Par exemple, la valeur d’une option de requête **$skipToken** peut identifier le dixième élément dans une collection ou le 20e élément dans une collection contenant 50 éléments, ou toute autre position dans la collection.

Dans certaines réponses, vous verrez une valeur `@odata.nextLink`. Certaines d’entre elles incluent une valeur **$skipToken**. La valeur **$skipToken** est semblable à un marqueur qui indique au service où reprendre pour l’ensemble de résultats suivant. Voici un exemple d’une valeur `@odata.nextLink` d’une réponse où il a été demandé aux utilisateurs un tri selon `displayName` : 

```
"@odata.nextLink": "https://graph.microsoft.com/v1.0/users?$orderby=displayName&$skiptoken=X%2783630372100000000000000000000%27"
```

Pour renvoyer la page suivante d’utilisateurs dans votre organisation, la syntaxe est la suivante.

```http
GET  https://graph.microsoft.com/v1.0/users?$orderby=displayName&$skiptoken=X%2783630372100000000000000000000%27
```

## <a name="$count"></a>$count
Utilisez **$count** comme paramètre de requête pour inclure le nombre total d’éléments dans une collection, ainsi que la page des valeurs de données renvoyées par le Graph, comme dans l’exemple suivant :
```http
GET  https://graph.microsoft.com/v1.0/me/contacts?$count=true
```
Ceci renverra à la fois la collection **contacts** et le nombre d'éléments dans la collection **contacts** dans la propriété `@odata.count`.

>**Remarque :** ceci n'est pas pris en charge pour les collections [directoryObject](http://graph.microsoft.io/en-us/docs/api-reference/v1.0/resources/directoryobject).
