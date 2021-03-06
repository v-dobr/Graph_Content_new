# <a name="microsoft-graph-optional-query-parameters"></a>Необязательные параметры запросов Microsoft Graph
В Microsoft Graph предусмотрено несколько дополнительных параметров запросов, с помощью которых можно указывать и регулировать объем возвращаемых данных. Microsoft Graph поддерживает следующие параметры запросов. 

|Имя|Значение|Описание|
|:---------------|:--------|:-------|
|$select|string|Список свойств, включаемых в ответ (разделитель — запятая).|
|$expand|string|Список отношений, развертываемых и включаемых в ответ (разделитель — запятая).  |
|$orderby|string|Список свойств, которые используются для сортировки элементов в коллекции ответов (разделитель — запятая).|
|$filter|string|Фильтрация ответа на основе набора условий.|
|$top|int|Количество элементов, возвращаемых в результирующем наборе.|
|$skip|int|Количество элементов, пропускаемых в результирующем наборе.|
|$skipToken|string|Токен разбиения по страницам, который используется для получения следующего набора результатов.|
|$count|none|Коллекция и количество элементов в ней.|

Эти параметры совместимы с [языком запросов OData версии 4](http://docs.oasis-open.org/odata/odata/v4.0/errata03/os/complete/part2-url-conventions/odata-v4.0-errata03-os-part2-url-conventions-complete.html#_Toc453752356).

>  **Примечание**. В **бета-версии** конечной точки Microsoft Graph для удобства можно пропускать префикс **$**. Например, вместо параметра **$expand** можно использовать **expand**. Дополнительные сведения и примеры см. в статье [Поддержка параметров без префиксов $ в Microsoft Graph](http://dev.office.com/queryparametersinMicrosoftGraph).  

**Кодирование параметров запроса**

- Если вы пользуетесь параметрами запроса в [Microsoft Graph Explorer](https://graph.microsoft.io/en-us/graph-explorer#), вы можете просто скопировать и вставить приведенные ниже примеры, не применяя кодировку URL к строке запроса. Следующий пример можно использовать _в Graph Explorer_, не кодируя пробелы и кавычки:
```http
GET https://graph.microsoft.com/v1.0/me/messages?$filter=from/emailAddress/address eq 'jon@contoso.com'
``` 
- При указании параметров запроса _в приложении_ убедитесь, что вы правильно кодируете [зарезервированные символы URI](https://tools.ietf.org/html/rfc3986#section-2.2). Например, кодируйте пробелы и кавычки в последнем примере следующим образом:
```http
GET https://graph.microsoft.com/v1.0/me/messages?$filter=from/emailAddress/address%20eq%20%27jon@contoso.com%27
```

### <a name="$select"></a>$select
Чтобы указать другой набор возвращаемых свойств, отличный от набора свойств, предоставляемого службой Graph, используйте параметр запроса **$select**. С помощью параметра **$select** можно выбрать подмножество или супермножество стандартного набора возвращаемых свойств. Например, при получении сообщений можно выбрать только свойства **from** и **subject**.

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

В запросах Microsoft Graph API пути к объекту или коллекции указанного элемента не разворачиваются автоматически. Эта особенность позволяет уменьшить сетевой трафик и время создания ответа службой. Однако в некоторых случаях эти результаты может потребоваться включить в ответ.

Чтобы API развернул дочерний объект или коллекцию и включил эти результаты, можно использовать параметр строки запроса **$expand**.

Например, с помощью параметра **$expand** можно получить сведения о корневом диске и дочерних элементах верхнего уровня на диске. Кроме того, в этом примере используется оператор **$select**, который возвращает только свойства _id_ и _name_ дочерних элементов.

```http
GET https://graph.microsoft.com/v1.0/me/drive/root?$expand=children($select=id,name)
```

>  **Примечание**. Максимальное количество развернутых объектов в запросе — 20. 

> Кроме того, запрашивая ресурс [user](http://graph.microsoft.io/en-us/docs/api-reference/v1.0/resources/user), можно использовать параметр **$expand**, чтобы получить свойства только одного дочернего объекта или коллекции за раз. 

В следующем примере возвращаются объекты **user**, каждый из которых содержит до 20 объектов **directReport** в развернутой коллекции **directReports**:
```http
GET https://graph.microsoft.com/v1.0/users?$expand=directReports
```
Другие ресурсы также могут иметь ограничения, поэтому всегда проверяйте наличие ошибок.


<!---The following shows a sample result that is returned in the response body.-->


## <a name="$orderby"></a>$orderby

Чтобы указать порядок сортировки элементов, возвращаемых из API Microsof Graph, используйте параметр запроса **$orderby**. 

Например, чтобы возвратить список пользователей в организации, упорядоченный по отображаемому имени, используйте следующий синтаксис:

```http
GET https://graph.microsoft.com/v1.0/users?$orderby=displayName
``` 

Вы также можете сортировать данные по объектам сложного типа. В приведенном ниже примере полученные сообщения сортируются по полю **address** свойства **from**, принадлежащего к сложному типу **emailAddress**.

```http
GET https://graph.microsoft.com/v1.0/me/messages?$orderby=from/emailAddress/address
``` 

Чтобы отсортировать результаты по возрастанию или убыванию, добавьте `asc` или `desc` к имени поля, разделив их пробелом, например: `?$orderby=name%20desc`.

 >  **Примечание**. В запросах к ресурсу [user] параметр **$orderby** невозможно совмещать с выражениями filter.

## <a name="$filter"></a>$filter
Чтобы отфильтровать данные ответа на основе набора условий, используйте параметр запроса **$filter**. Например, чтобы возвратить список пользователей в организации, отфильтрованный по отображаемому имени, которое начинается с "Garth", используйте следующий синтаксис:

```http
GET https://graph.microsoft.com/v1.0/users?$filter=startswith(displayName,'Garth')
```

Вы также можете фильтровать данные по объектам сложного типа. В следующем примере возвращаются сообщения, для которых в поле **address** свойства **from** задано значение "jon@contoso.com". **from** — это свойство сложного типа **emailAddress**.

```http
GET https://graph.microsoft.com/v1.0/me/messages?$filter=from/emailAddress/address eq 'jon@contoso.com'
``` 

## <a name="$top"></a>$top
Чтобы указать максимальное количество элементов, возвращаемых в результирующем наборе, используйте параметр запроса **$top**. Параметр запроса **$top** определяет подмножество в коллекции. В это подмножество включаются первые N элементов набора, где N — положительное целое число, указанное в этом параметре запроса. Например, чтобы возвратить первые пять сообщений в почтовом ящике пользователя, используйте следующий синтаксис:

```http
GET https://graph.microsoft.com/v1.0/me/messages?$top=5
```

## <a name="$skip"></a>$skip
Чтобы задать количество элементов, которое необходимо пропустить, используйте параметр запроса **$skip**. Например, чтобы возвратить список событий, начиная с 21-го, отсортированный по дате создания, используйте указанный ниже синтаксис.

```http
GET  https://graph.microsoft.com/v1.0/me/events?$orderby=createdDateTime&$skip=20
```

## <a name="$skiptoken"></a>$skipToken
Чтобы запросить вторую и последующие страницы данных Graph, используйте параметр запроса **$skipToken**. Параметр запроса **$skipToken** предоставляется в списке URL-адресов, возвращаемом службой Graph вместе с частичным подмножеством результатов (как правило, это происходит при постраничном просмотре на стороне сервера). Он указывает в коллекции момент завершения отправки результатов и передается обратно в Graph, чтобы указать, с какого момента необходимо возобновить отправку результатов. Например, значение параметра запроса **$skipToken** может определять десятый или двадцатый элемент в коллекции, содержащей 50 элементов, а также любую другую позицию в коллекции.

Некоторые ответы будут содержать значение `@odata.nextLink`. Другие ответы включают значение **$skipToken**. Значение **$skipToken** — это метка, которая сообщает службе, где необходимо возобновить работу для следующего набора результатов. Ниже приведен пример значения `@odata.nextLink` из ответа, где список пользователей упорядочен по свойству `displayName`: 

```
"@odata.nextLink": "https://graph.microsoft.com/v1.0/users?$orderby=displayName&$skiptoken=X%2783630372100000000000000000000%27"
```

Чтобы вернуть следующую страницу списка пользователей в организации, используйте приведенный ниже синтаксис.

```http
GET  https://graph.microsoft.com/v1.0/users?$orderby=displayName&$skiptoken=X%2783630372100000000000000000000%27
```

## <a name="$count"></a>$count
Используйте параметр запроса **$count**, чтобы включить общее количество элементов в коллекции вместе со страницей значений, возвращенных из Graph, как показано в следующем примере:
```http
GET  https://graph.microsoft.com/v1.0/me/contacts?$count=true
```
При этом возвращается коллекция **contacts**, а также количество элементов в коллекции **contacts** в свойстве `@odata.count`.

>**Примечание.** Эта возможность не поддерживается для коллекций [directoryObject](http://graph.microsoft.io/en-us/docs/api-reference/v1.0/resources/directoryobject).
