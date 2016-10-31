# <a name="microsoft-graph-optional-query-parameters"></a>Parámetros de consulta opcionales de Microsoft Graph
Microsoft Graph proporciona varios parámetros de consulta opcionales que puede usar para especificar y controlar la cantidad de datos devueltos en una respuesta. Microsoft Graph es compatible con las siguientes opciones de consulta. 

|Nombre|Valor|Descripción|
|:---------------|:--------|:-------|
|$select|string|Lista separada por comas de las propiedades para incluir en la respuesta.|
|$expand|string|Lista separada por comas de las relaciones para expandir e incluir en la respuesta.  |
|$orderby|string|Lista separada por comas de las propiedades que se usan para cambiar el orden de los elementos de la colección de respuesta.|
|$filter|string|Filtre la respuesta basándose en un conjunto de criterios.|
|$top|int|El número de elementos a devolver en un conjunto de resultados.|
|$skip|int|El número de elementos a omitir en un conjunto de resultados.|
|$skipToken|string|Token de paginación que se usa para obtener el siguiente conjunto de resultados.|
|$count|ninguno|Una colección y el número de elementos de la colección.|

Estos parámetros son compatibles con el [lenguaje de consulta de OData V4](http://docs.oasis-open.org/odata/odata/v4.0/errata03/os/complete/part2-url-conventions/odata-v4.0-errata03-os-part2-url-conventions-complete.html#_Toc453752356).

>  **Nota**: En el punto de conexión **beta** de Microsoft Graph, puede omitir el prefijo **$** para obtener una experiencia más sencilla. Por ejemplo, en lugar de usar **$expand**, puede usar **expand**. Para obtener más detalles y ejemplos, consulte [Compatibilidad con los parámetros de consulta sin el prefijo '$' en Microsoft Graph](http://dev.office.com/queryparametersinMicrosoftGraph).  

**Codificación de los parámetros de consulta**

- Si prueba los parámetros de consulta en el [Explorador de Microsoft Graph](https://graph.microsoft.io/en-us/graph-explorer#), puede simplemente copiar y pegar los ejemplos siguientes sin aplicar ninguna codificación de direcciones URL a la cadena de consulta. El siguiente ejemplo funciona bien _en el explorador de Graph_ sin codificar los caracteres de espacio y comillas:
```http
GET https://graph.microsoft.com/v1.0/me/messages?$filter=from/emailAddress/address eq 'jon@contoso.com'
``` 
- En general, al especificar los parámetros de consulta _en su aplicación_, asegúrese de codificar correctamente los caracteres que estén [reservados para significados especiales en un URI](https://tools.ietf.org/html/rfc3986#section-2.2). Por ejemplo, codifique los caracteres de espacio y comillas del ejemplo anterior, tal y como se muestra a continuación:
```http
GET https://graph.microsoft.com/v1.0/me/messages?$filter=from/emailAddress/address%20eq%20%27jon@contoso.com%27
```

### <a name="$select"></a>$select
Para especificar un conjunto diferente de propiedades para devolver en lugar del conjunto de propiedades predeterminado que proporciona Graph, use la opción de consulta **$select**. La opción **$select** permite elegir un subconjunto o un superconjunto del conjunto predeterminado que se haya devuelto. Por ejemplo, al recuperar los mensajes, quizás quiera que solo se devuelvan las propiedades **from** y **subject** de los mensajes.

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

En las solicitudes de la API de Microsoft Graph, las navegaciones a un objeto o a una colección del elemento al que se hace referencia no se expanden automáticamente. Esto es así por diseño para reducir el tráfico de la red y el tiempo que se tarda en generar una respuesta desde el servicio. Sin embargo, en algunos casos puede que desee incluir esos resultados en una respuesta.

Puede usar el parámetro de cadena de consulta **$expand** para indicar a la API que expanda una colección u objeto secundarios y que incluya esos resultados.

Por ejemplo, para recuperar la información de la unidad de raíz y los elementos secundarios del nivel superior en una unidad, use el parámetro **$expand**. En este ejemplo, también se usa una instrucción **$select** para devolver solo las propiedades _id_ y _name_ de los elementos secundarios.

```http
GET https://graph.microsoft.com/v1.0/me/drive/root?$expand=children($select=id,name)
```

>  **Nota**: El número máximo de objetos ampliados para una solicitud es 20. 

> Asimismo, si realiza una consulta en el recurso [usuario](http://graph.microsoft.io/en-us/docs/api-reference/v1.0/resources/user), puede usar **$expand** para obtener las propiedades de un solo objeto secundario o de una colección a la vez. 

El ejemplo siguiente obtiene objetos **user**, cada uno con hasta 20 objetos **directReport** en la colección **directReports** ampliada:
```http
GET https://graph.microsoft.com/v1.0/users?$expand=directReports
```
Otros recursos también pueden tener un límite, así que siempre compruébelo para evitar posibles errores.


<!---The following shows a sample result that is returned in the response body.-->


## <a name="$orderby"></a>$orderby

Para especificar el orden de clasificación de los elementos devueltos de la API de Microsoft Graph, use la opción de consulta **$orderby**. 

Por ejemplo, para devolver los usuarios de la organización ordenados por su nombre para mostrar, la sintaxis será la siguiente:

```http
GET https://graph.microsoft.com/v1.0/users?$orderby=displayName
``` 

También puede ordenar por entidades de tipo complejo. El ejemplo siguiente obtiene los mensajes y los ordena por el campo **address** de la propiedad **from**, que es del tipo complejo **emailAddress**:

```http
GET https://graph.microsoft.com/v1.0/me/messages?$orderby=from/emailAddress/address
``` 

Para ordenar los resultados en orden ascendente o descendente, anexe `asc` o `desc` al nombre del campo, separado por un espacio. Por ejemplo, `?$orderby=name%20desc`.

 >  **Nota**: Si realiza una consulta en el recurso [user], **$orderby** no se podrá combinar con expresiones de filtro.

## <a name="$filter"></a>$filter
Para filtrar los datos de respuesta a partir de un conjunto de criterios, use la opción de consulta **$filter**. Por ejemplo, para devolver los usuarios en el filtro de la organización con un nombre para mostrar que comience por "Sergio", la sintaxis será la siguiente:

```http
GET https://graph.microsoft.com/v1.0/users?$filter=startswith(displayName,'Garth')
```

También puede filtrar por entidades de tipo complejo. En el ejemplo siguiente, se devuelven mensajes cuyo campo **address** de la propiedad **from** es igual a "alberto@contoso.com". La propiedad **from** es del tipo complejo **emailAddress**.

```http
GET https://graph.microsoft.com/v1.0/me/messages?$filter=from/emailAddress/address eq 'jon@contoso.com'
``` 

## <a name="$top"></a>$top
Para especificar el número máximo de elementos a devolver en un conjunto de resultados, use la opción de consulta **$top**. La opción de consulta **$top** identifica un subconjunto de la colección. Este subconjunto se forma al seleccionar solo los primeros N elementos del conjunto, donde N es un entero positivo especificado mediante esta opción de consulta. Por ejemplo, para devolver los cinco primeros mensajes en el buzón del usuario, la sintaxis es la siguiente:

```http
GET https://graph.microsoft.com/v1.0/me/messages?$top=5
```

## <a name="$skip"></a>$skip
Para establecer el número de elementos que se omitirán antes de recuperar elementos de una colección, use la opción de consulta **$skip**. Por ejemplo, para que se devuelvan los eventos ordenados por fecha de creación y comenzando por el evento 21, la sintaxis será la siguiente:

```http
GET  https://graph.microsoft.com/v1.0/me/events?$orderby=createdDateTime&$skip=20
```

## <a name="$skiptoken"></a>$skipToken
Para solicitar la segunda página y las posteriores de los datos de Graph, use la opción de consulta **$skipToken**. La opción de consulta **$skipToken** es una opción que se proporciona en las URL que se devuelven de Graph cuando este ha devuelto un subconjunto parcial de resultados, normalmente debido a la paginación del lado servidor. Identifica en una colección el punto en el que el servidor ha terminado de enviar los resultados y se devuelve a Graph para indicar el punto desde el cual debe reanudar el envío de los resultados. Por ejemplo, el valor de una opción de consulta **$skipToken** podría identificar el décimo elemento de una colección o el número 20 de una colección que contuviera 50, o bien cualquier otra posición de la colección.

En algunas respuestas, verá un valor `@odata.nextLink`. Algunas de ellas incluirán un valor **$skipToken**. El valor **$skipToken** es como un marcador que indica al servicio el punto en el que deberá reanudarse para obtener el siguiente conjunto de resultados. El siguiente es un ejemplo de un valor `@odata.nextLink` de una respuesta en la que se han solicitado los usuarios ordenados por `displayName`: 

```
"@odata.nextLink": "https://graph.microsoft.com/v1.0/users?$orderby=displayName&$skiptoken=X%2783630372100000000000000000000%27"
```

Para devolver la siguiente página de usuarios de su organización, la sintaxis será como la que se muestra a continuación.

```http
GET  https://graph.microsoft.com/v1.0/users?$orderby=displayName&$skiptoken=X%2783630372100000000000000000000%27
```

## <a name="$count"></a>$count
Use **$count** como consulta de parámetro para incluir un recuento del número total de elementos de una colección junto con la página de valores de datos que se devuelve de Graph, como en el ejemplo siguiente:
```http
GET  https://graph.microsoft.com/v1.0/me/contacts?$count=true
```
Esta acción devolvería tanto la colección de **contactos** como el número de elementos de la colección de **contactos** en la propiedad `@odata.count`.

>**Nota:** Esto no se admite para las colecciones [directoryObject](http://graph.microsoft.io/en-us/docs/api-reference/v1.0/resources/directoryobject).
