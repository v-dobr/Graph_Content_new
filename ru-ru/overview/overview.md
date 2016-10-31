# <a name="overview-of-microsoft-graph"></a>Обзор Microsoft Graph

Microsoft Graph предоставляет несколько API из Office 365 и других облачных служб Майкрософт через одну конечную точку: **https://graph.microsoft.com**. Microsoft Graph упрощает запросы, которые были бы намного сложнее при использовании другой службы. 
 
С помощью Microsoft Graph вы можете:

- получать доступ к данным из нескольких облачных служб Майкрософт, в том числе Azure Active Directory, Exchange Online в составе Office 365, SharePoint, OneDrive, OneNote и Планировщик;
- переходить между объектами и связями;
- получать доступ к аналитическим и статистическим данным из облака Майкрософт (для коммерческих пользователей).

**Стек разработки Microsoft Graph**

![Схема, на которой показаны уровни стека разработки Microsoft Graph. Внизу представлен уровень данных, включающий в себя пользователей, группы, файлы, почту, календари, личные контакты, задачи, контакты организации, людей, Excel и заметки. Следующий уровень — проверка подлинности и авторизация. Далее — выбранная вами среда разработки, в том числе Android, iOS и пакеты SDK с API Microsoft Graph для Visual Studio. И наконец — ваше решение с выбранной технологией, например .NET, JS, HTML или Ruby, размещенное в службе Microsoft Azure или на другой платформе.](./images/MicrosoftGraph_DevStack.png)

<!--<a name="msg_queries"> </a>-->

##<a name="common-microsoft-graph-queries"></a>Распространенные запросы Microsoft Graph

Microsoft Graph предоставляет две конечные точки: /v1.0 и /beta. Конечная точка /v1.0 включает ресурсы, доступные из рабочего приложения. Конечная точка [/beta](http://graph.microsoft.io/en-us/docs/api-reference/beta/beta-overview) включает API-интерфейсы, которые в настоящий момент находятся в предварительной версии. В приведенной ниже таблице перечислены распространенные запросы, с помощью которых можно получить доступ к API Microsoft Graph.

| **Операция** | **Конечная точка службы** |
|:--------------------------|:----------------------------------------|
|   Получение своего профиля |    [`https://graph.microsoft.com/v1.0/me`](/graph-explorer/#?request=me&version=v1.0) |
|   Получение своих файлов | [`https://graph.microsoft.com/v1.0/me/drive/root/children`](/graph-explorer/#?request=me%2Fdrive%2Froot%2Froot%2Fchildren&version=v1.0) |
|   Получение своей фотографии     | [`https://graph.microsoft.com/v1.0/me/photo/$value`](/graph-explorer/#?request=me%2Fphoto%2F%24value&version=v1.0) |
|   Получение своей почты |   [`https://graph.microsoft.com/v1.0/me/messages`](/graph-explorer/#?request=me%2Fmessages&version=v1.0) |
|   Получение электронной почты высокой важности | [`https://graph.microsoft.com/v1.0/me/messages?$filter=importance%20eq%20'high'`](/graph-explorer/#?request=me%2Fmessages%3F%24filter%3Dimportance%2520eq%2520'high'&version=v1.0) |
|   Получение своего календаря |   [`https://graph.microsoft.com/v1.0/me/calendar`](/graph-explorer/#?request=me%2Fcalendar&version=v1.0) |
|   Получение сведений о руководителе  | [`https://graph.microsoft.com/v1.0/me/manager`](/graph-explorer/#?request=me%2Fmanager&version=v1.0) |
|   Получение сведений о последнем пользователе, изменившем файл foo.txt |  [`https://graph.microsoft.com/v1.0/me/drive/root/children/foo.txt/lastModifiedByUser`](/graph-explorer/#?request=me%2Fdrive%2Froot%2Froot%2Fchildren%2Ffoo.txt%2FlastModifiedByUser&version=v1.0) |
|   Получение списка единых групп, в которые вы входите|   [`https://graph.microsoft.com/v1.0/me/memberOf/$/microsoft.graph.group?$filter=groupTypes/any(a:a%20eq%20'unified')`](/graph-explorer/#?request=me%2FmemberOf%2F%24%2Fmicrosoft.graph.group%3F%24filter%3DgroupTypes%2Fany(a%3Aa%2520eq%2520'unified'&version=v1.0)) |
|   Получение списка пользователей в организации     | [`https://graph.microsoft.com/v1.0/users`](/graph-explorer/#?request=users&version=v1.0) |
|   Получение групповых бесед |   `https://graph.microsoft.com/v1.0/groups/<id>/conversations`|
|   Получение списка связанных пользователей    | [`https://graph.microsoft.com/beta/me/people`](/graph-explorer/#?request=me%2Fpeople&version=beta)  |
|   Получение файлов, популярных в вашей компании |  [`https://graph.microsoft.com/beta/me/trendingAround`](/graph-explorer/#?request=me%2FtrendingAround&version=beta) |
|   Получение списка коллег     | [`https://graph.microsoft.com/beta/me/workingWith`](/graph-explorer/#?request=me%2FworkingWith&version=beta) |
|   Получение своих задач    | [`https://graph.microsoft.com/beta/me/tasks`](/graph-explorer/#?request=me%2Ftasks&version=beta) |
|   Получение своих заметок |  [`https://graph.microsoft.com/beta/me/notes/notebooks`](/graph-explorer/#?request=me%2Fnotes%2Fnotebooks&version=beta) |


>**Примечание.** API-интерфейсы в конечной точке бета-версии могут быть изменены. Не рекомендуем использовать их в рабочих приложениях. 

<!-- <a name="msg_roof"> </a> -->

## <a name="explore-microsoft-graph"></a>Изучение Microsoft Graph

- [Приступите к работе](../get-started/get-started.md) с Microsoft Graph на выбранной вами платформе.
- В оглавлении вы найдете ресурсы и операции, которые можно использовать в рабочих приложениях.
- Опробуйте [бета-версии новых API](http://graph.microsoft.io/en-us/docs/api-reference/beta/beta-overview).
- Посетите [песочницу Microsoft Graph](https://graph.microsoft.io/en-us/graph-explorer).

 >  Ваше мнение важно для нас. Для связи с нами используйте сайт [Stack Overflow](http://stackoverflow.com/questions/tagged/office365+or+microsoftgraph). Помечайте свои вопросы тегами [MicrosoftGraph] и [office365].



