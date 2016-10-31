# <a name="microsoft-graph-optional-query-parameters"></a>Optionale Abfrageparameter in Microsoft Graph
Microsoft Graph stellt mehrere optionale Abfrageparameter bereit, die Sie zum Festlegen und Steuern der in einer Antwort zurückgegebenen Datenmenge verwenden können. Microsoft Graph unterstützt die folgenden Abfrageoptionen. 

|Name|Wert|Beschreibung|
|:---------------|:--------|:-------|
|$select|string|Durch Trennzeichen getrennte Liste der Eigenschaften, die in der Antwort aufgenommen werden.|
|$expand|string|Durch Trennzeichen getrennte Liste der Beziehungen, die in der Antwort erweitert und aufgenommen werden.  |
|$orderby|string|Durch Trennzeichen getrennte Liste von Eigenschaften, die zum Sortieren der Elemente in der Antwortsammlung verwendet werden.|
|$filter|string|Filtert die Antwort basierend auf einer Reihe von Kriterien.|
|$top|int|Die Anzahl der Elemente, die in einem Resultset zurückgegeben werden.|
|$skip|int|Die Anzahl der Elemente, die in einem Resultset übersprungen werden.|
|$skipToken|string|Pagingtoken, das zum Abrufen des nächsten Resultsets verwendet wird.|
|$count|Keine|Eine Auflistung, und die Anzahl der Elemente in der Auflistung.|

Diese Parameter sind kompatibel mit der [Abfragesprache OData V4](http://docs.oasis-open.org/odata/odata/v4.0/errata03/os/complete/part2-url-conventions/odata-v4.0-errata03-os-part2-url-conventions-complete.html#_Toc453752356).

>  **Hinweis**: Auf dem Microsoft Graph **Beta**-Endpunkt, kann das Präfix **$** entfallen, um die Erfahrung zu vereinfachen. Anstelle von **$Erweitern**, können Sie z. B. **Erweitern** verwenden. Weitere Informationen und Beispiele finden Sie unter [Abfrageparameter ohne $ Präfixe in Microsoft Graph unterstützen](http://dev.office.com/queryparametersinMicrosoftGraph).  

**Codieren von Abfrageparametern**

- Wenn Sie Abfrageparameter in [Microsoft Graph Explorer](https://graph.microsoft.io/en-us/graph-explorer#) ausprobieren, können Sie einfach die unten aufgeführten Beispiele kopieren und einfügen, ohne der Abfragezeichenfolge eine URL-Codierung hinzufügen zu müssen. Das folgende Beispiel funktioniert einwandfrei _im Graph Explorer_, ohne die Leerzeichen und Anführungszeichen zu codieren:
```http
GET https://graph.microsoft.com/v1.0/me/messages?$filter=from/emailAddress/address eq 'jon@contoso.com'
``` 
- Stellen Sie beim Festlegen von Abfrageparametern [in Ihrer App](https://tools.ietf.org/html/rfc3986#section-2.2) allgemein sicher, dass Sie solche Zeichen angemessen codieren, für die _im URI eine besondere Bedeutung reserviert ist_. Codieren Sie zum Beispiel die Leerzeichen und Anführungszeichen im letzten Beispiel wie dargestellt:
```http
GET https://graph.microsoft.com/v1.0/me/messages?$filter=from/emailAddress/address%20eq%20%27jon@contoso.com%27
```

### <a name="$select"></a>$select
Um für die Rückgabe eine andere Gruppe von Eigenschaften anzugeben als die Standardgruppe von Graph, verwenden Sie die Abfrageoption **$select**. Die **$select**-Option ermöglicht die Auswahl einer Teilmenge oder Obermenge der zurückgegebenen Standardgruppe. Wenn Sie z. B. Ihre Nachrichten abrufen, möchten Sie ggf. festlegen, dass nur die Eigenschaften **Von** und **Betreff** der Nachrichten zurückgegeben werden.

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

In Microsoft Graph-API-Anforderungen wird die Navigation zu einem Objekt oder einer Auflistung des Elements, auf das verwiesen wird, nicht automatisch erweitert. Dies ist beabsichtigt, da der Netzwerkdatenverkehr und die Zeit zum Generieren einer Antwort vom Dienst auf diese Weise reduziert werden. Möglicherweise möchten Sie jedoch in einigen Fällen diese Ergebnisse in eine Antwort einbeziehen.

Sie können den **$expand**-Abfragezeichenfolgen-Parameter verwenden, um die API zum Erweitern eines untergeordneten Objekts oder einer Auflistung und zum Einbeziehen dieser Ergebnisse anzuweisen.

Verwenden Sie beispielsweise zum Abrufen von Stammlaufwerkinformationen und der untergeordneten Elemente auf oberster Ebene in einem Laufwerk den **$expand**-Parameter. In diesem Beispiel wird auch eine **$select**-Anweisung verwendet, um nur die Eigenschaften_ID_ und _Name_ der untergeordneten Elemente zurückzugeben.

```http
GET https://graph.microsoft.com/v1.0/me/drive/root?$expand=children($select=id,name)
```

>  **Hinweis**: Die maximale Anzahl der erweiterten Objekte für eine Anforderung beträgt 20. 

> Wenn Sie außerdem eine Abfrage für die [Benutzer](http://graph.microsoft.io/en-us/docs/api-reference/v1.0/resources/user)-Ressource durchführen, können Sie **$expand** zum Abrufen der Eigenschaften von jeweils nur einem untergeordneten Objekt oder einer Auflistung verwenden. 

Das folgende Beispiel ruft **Benutzer**-Objekte ab, jeweils mit bis zu 20 erweiterten **directReport**-Objekten in der **directReports**-Auflistung:
```http
GET https://graph.microsoft.com/v1.0/users?$expand=directReports
```
Einige andere Ressourcen haben möglicherweise ebenfalls einen Höchstwert, führen Sie daher immer eine Überprüfung auf mögliche Fehler durch.


<!---The following shows a sample result that is returned in the response body.-->


## <a name="$orderby"></a>$orderby

Verwenden Sie zum Festlegen der Sortierreihenfolge der aus der Microsoft Graph-API zurückgegebenen Elemente die **$orderby**-Abfrageoption. 

Wenn die zurückgegebenen Benutzer in der Organisation nach dem Anzeigenamen sortiert werden sollen, lautet die Syntax hierfür wie folgt:

```http
GET https://graph.microsoft.com/v1.0/users?$orderby=displayName
``` 

Sie können auch nach komplexen Typentitäten sortieren. Im folgenden Beispiel werden Nachrichten abgerufen und nach dem **Adresse**-Feld der **from**-Eigenschaft sortiert, die vom komplexen Typ **emailAddress** ist:

```http
GET https://graph.microsoft.com/v1.0/me/messages?$orderby=from/emailAddress/address
``` 

Wenn Sie die Ergebnisse in aufsteigender oder absteigender Reihenfolge sortieren möchten, fügen Sie entweder `asc` oder `desc` an den Namen des Felds getrennt durch ein Leerzeichen an, z. B. `?$orderby=name%20desc`.

 >  **Hinweis**: Bei Abfragen zur [user]-Ressource kann **$orderby** nicht zusammen mit filter-Ausdrücken verwendet werden.

## <a name="$filter"></a>$filter
Wenn Sie die zurückgegebenen Daten anhand bestimmter Kriterien filtern möchten, verwenden Sie die **$filter**-Abfrageoption. Wenn die zurückgegebenen Benutzer in der Organisation anhand des Anzeigenamens gefiltert werden sollen, der mit „Garth“ beginnt, lautet die Syntax hierfür wie folgt:

```http
GET https://graph.microsoft.com/v1.0/users?$filter=startswith(displayName,'Garth')
```

Sie können auch nach komplexen Typentitäten filtern. Das folgende Beispiel gibt Nachrichten zurück, bei denen das Feld **address** der **from**-Eigenschaft gleich „jon@contoso.com“ ist. Die **from**-Eigenschaft hat den komplexen Typ **emailAddress**.

```http
GET https://graph.microsoft.com/v1.0/me/messages?$filter=from/emailAddress/address eq 'jon@contoso.com'
``` 

## <a name="$top"></a>$top
Verwenden Sie zum Festlegen der maximalen Anzahl der in einem Resultset zurückzugebenden Elemente die **$top**-Abfrageoption. Die **$top**-Abfrageoption ermittelt eine Teilmenge in der Sammlung. Diese Teilmenge setzt sich aus den festgelegten ersten N Elementen zusammen, wobei N eine positive ganze Zahl ist, die durch diese Abfrageoption angegeben ist. Wenn beispielsweise die ersten fünf Nachrichten im Postfach des Benutzers zurückgegeben werden sollen, lautet die Syntax wie folgt:

```http
GET https://graph.microsoft.com/v1.0/me/messages?$top=5
```

## <a name="$skip"></a>$skip
Verwenden Sie zum Festlegen der Anzahl der Elemente, die vor dem Abrufen von Elementen in einer Sammlung übersprungen werden sollen, die **$skip**-Abfrageoption. Wenn die zurückgegebenen Ereignisse nach Erstellungsdatum sortiert werden, wobei mit dem 21. Ereignis begonnen wird, lautet die Syntax wie folgt.

```http
GET  https://graph.microsoft.com/v1.0/me/events?$orderby=createdDateTime&$skip=20
```

## <a name="$skiptoken"></a>$skipToken
Verwenden Sie zur Anforderung der zweiten und nachfolgender Seiten mit Graph-Daten die Abfrageoption **$skipToken**. Die **$skipToken**-Abfrageoption ist eine Option, die in URLs bereitgestellt wird, die von Graph übergeben werden, wenn Graph, in der Regel aufgrund von serverseitigem Paging, einen Teil einer Teilmenge der Ergebnisse zurückgibt. Sie nennt den Punkt in einer Sammlung, an dem der Server das Senden der Ergebnisse beendet hat, und wird zu Graph zurückgegeben, um anzugeben, von wo aus das Senden der Ergebnisse wieder aufgenommen werden sollte. Der Wert einer **$skipToken**-Abfrageoption könnte z. B. das zehnte Element in einer Sammlung oder das 20. Element in einer Sammlung mit 50 Elementen oder eine andere Position in der Sammlung angeben.

In einigen Fällen wird ein `@odata.nextLink`-Wert angezeigt. Manchmal ist ein **$skipToken**-Wert enthalten. Der **$skipToken**-Wert stellt eine Markierung für den Dienst dar, die angibt, an welcher Stelle das nächste Resultset beginnen soll. Nachfolgend ein Beispiel für einen `@odata.nextLink`-Wert aus einer Antwort, in der Benutzer sortiert nach `displayName` angefordert werden: 

```
"@odata.nextLink": "https://graph.microsoft.com/v1.0/users?$orderby=displayName&$skiptoken=X%2783630372100000000000000000000%27"
```

Um die nächste Seite der Benutzer in Ihrer Organisation zurückzugeben, lautet die Syntax wie folgt.

```http
GET  https://graph.microsoft.com/v1.0/users?$orderby=displayName&$skiptoken=X%2783630372100000000000000000000%27
```

## <a name="$count"></a>$count
Verwenden Sie **$count** als Abfrageparameter, um die Gesamtzahl der Elemente in einer Sammlung zusammen mit der Seite der Datenwerte anzugeben, die von Graph zurückgegeben werden (siehe folgendes Beispiel):
```http
GET  https://graph.microsoft.com/v1.0/me/contacts?$count=true
```
Zurückgegeben werden sowohl die **Kontakte**-Sammlung als auch die Anzahl der Elemente in der **Kontakte**-Sammlung in der Eigenschaft `@odata.count`.

>**Hinweis:** Dies wird für [directoryObject](http://graph.microsoft.io/en-us/docs/api-reference/v1.0/resources/directoryobject)-Sammlungen nicht unterstützt.
