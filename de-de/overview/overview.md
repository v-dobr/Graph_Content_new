# <a name="overview-of-microsoft-graph"></a>Übersicht über Microsoft Graph

Microsoft Graph stellt mehrere APIs aus Office 365 und anderen Microsoft-Clouddiensten über einen einzelnen Endpunkt (**https://graph.microsoft.com**) zur Verfügung. Microsoft Graph vereinfacht Abfragen, die andernfalls komplexer ausfallen würden. 
 
Mit Microsoft Graph können Sie Folgendes machen:

- Zugriff auf Daten aus mehreren Microsoft-Clouddiensten, u. a. Azure Active Directory, Exchange Online als Teil von Office 365, SharePoint, OneDrive, OneNote und Planner.
- Navigieren zwischen Entitäten und Beziehungen.
- Zugriff auf Informationen und Einblicke aus der Microsoft-Cloud (für kommerzielle Benutzer).

**Microsoft Graph-Entwicklerstack**

![Ein Diagramm, das die Ebenen des Microsoft Graph-Entwicklerstacks darstellt. Unten befindet sich die Datenebene, die Benutzer, Gruppen, Dateien, E-Mails, Kalender, persönliche Kontakte, Aufgaben, Organisationskontakte, Personen, Excel und Notizen enthält. Die nächste Ebene dient zur Authentifizierung und Autorisierung. Als Nächstes kommt die Entwicklungsumgebung Ihrer Auswahl, einschließlich Android-, iOS- und Visual Studio-Microsoft Graph-API-SDKs. Die letzte Ebene ist Ihre Lösung, die die Technologie Ihrer Wahl verwendet, darunter .NET, JS, HTML und Ruby, und sie wird auf Microsoft Azure oder einer anderen Hostingplattform gehostet.](./images/MicrosoftGraph_DevStack.png)

<!--<a name="msg_queries"> </a>-->

##<a name="common-microsoft-graph-queries"></a>Allgemeine Microsoft Graph-Abfragen

Microsoft Graph stellt zwei Endpunkte bereit: /v1.0 und /beta. Der Endpunkt /v1.0 umfasst die Ressourcen, auf die Sie in Ihrer Produktions-App zugreifen können. Der Endpunkt [/beta](http://graph.microsoft.io/en-us/docs/api-reference/beta/beta-overview)enthält APIs, die sich derzeit in der Vorschau befinden. Die folgende Tabelle listet einige allgemeine Abfragen auf, die Sie verwenden können, um auf die Microsoft Graph-API zuzugreifen.

| **Vorgang** | **Dienstendpunkt** |
|:--------------------------|:----------------------------------------|
|   Eigenes Profil mit GET abrufen |    [`https://graph.microsoft.com/v1.0/me`](/graph-explorer/#?request=me&version=v1.0) |
|   Eigene Dateien mit GET abrufen | [`https://graph.microsoft.com/v1.0/me/drive/root/children`](/graph-explorer/#?request=me%2Fdrive%2Froot%2Froot%2Fchildren&version=v1.0) |
|   Eigenes Foto mit GET abrufen	     | [`https://graph.microsoft.com/v1.0/me/photo/$value`](/graph-explorer/#?request=me%2Fphoto%2F%24value&version=v1.0) |
|   Eigene E-Mail mit GET abrufen |   [`https://graph.microsoft.com/v1.0/me/messages`](/graph-explorer/#?request=me%2Fmessages&version=v1.0) |
|   Eigene E-Mails mit Wichtigkeit „Hoch“ mit GET abrufen | [`https://graph.microsoft.com/v1.0/me/messages?$filter=importance%20eq%20'high'`](/graph-explorer/#?request=me%2Fmessages%3F%24filter%3Dimportance%2520eq%2520'high'&version=v1.0) |
|   Eigenen Kalender mit GET abrufen |   [`https://graph.microsoft.com/v1.0/me/calendar`](/graph-explorer/#?request=me%2Fcalendar&version=v1.0) |
|   Eigenen Vorgesetzten mit GET abrufen	  | [`https://graph.microsoft.com/v1.0/me/manager`](/graph-explorer/#?request=me%2Fmanager&version=v1.0) |
|   Letzten Benutzer zum Ändern der Datei „foo.txt“ mit GET abrufen |  [`https://graph.microsoft.com/v1.0/me/drive/root/children/foo.txt/lastModifiedByUser`](/graph-explorer/#?request=me%2Fdrive%2Froot%2Froot%2Fchildren%2Ffoo.txt%2FlastModifiedByUser&version=v1.0) |
|   Einheitliche Gruppen mit GET abrufen, bei denen ich Mitglied bin|   [`https://graph.microsoft.com/v1.0/me/memberOf/$/microsoft.graph.group?$filter=groupTypes/any(a:a%20eq%20'unified')`](/graph-explorer/#?request=me%2FmemberOf%2F%24%2Fmicrosoft.graph.group%3F%24filter%3DgroupTypes%2Fany(a%3Aa%2520eq%2520'unified'&version=v1.0)) |
|   Benutzer in meiner Organisation mit GET abrufen	     | [`https://graph.microsoft.com/v1.0/users`](/graph-explorer/#?request=users&version=v1.0) |
|   Gruppenunterhaltungen mit GET abrufen |   `https://graph.microsoft.com/v1.0/groups/<id>/conversations`|
|   Verwandte Personen mit GET abrufen    | [`https://graph.microsoft.com/beta/me/people`](/graph-explorer/#?request=me%2Fpeople&version=beta)  |
|   Häufig verwendete Dateien mit GET abrufen |  [`https://graph.microsoft.com/beta/me/trendingAround`](/graph-explorer/#?request=me%2FtrendingAround&version=beta) |
|   Personen mit GET abrufen, mit denen ich zusammenarbeite     | [`https://graph.microsoft.com/beta/me/workingWith`](/graph-explorer/#?request=me%2FworkingWith&version=beta) |
|   Eigene Aufgaben mit GET abrufen    | [`https://graph.microsoft.com/beta/me/tasks`](/graph-explorer/#?request=me%2Ftasks&version=beta) |
|   Eigene Notizen mit GET abrufen |  [`https://graph.microsoft.com/beta/me/notes/notebooks`](/graph-explorer/#?request=me%2Fnotes%2Fnotebooks&version=beta) |


>**Hinweis:** Die APIs im Beta-Endpunkt können geändert werden. Es wird davon abgeraten, sie in Produktions-Apps zu verwenden. 

<!-- <a name="msg_roof"> </a> -->

## <a name="explore-microsoft-graph"></a>Microsoft Graph erforschen

- [Erste Schritte](../get-started/get-started.md) mit Microsoft Graph und der Plattform Ihrer Wahl.
- Entdecken Sie die Ressourcen und Vorgänge, die Sie in den Produktions-Apps verwenden können, indem Sie das Inhaltsverzeichnis durchsuchen.
- Anzeigen der neuen [Beta-APIs](http://graph.microsoft.io/en-us/docs/api-reference/beta/beta-overview).
- Testen Sie den [Microsoft Graph-Explorer](https://graph.microsoft.io/en-us/graph-explorer).

 >  Ihr Feedback ist uns wichtig. Nehmen Sie unter [Stack Overflow](http://stackoverflow.com/questions/tagged/office365+or+microsoftgraph) Kontakt mit uns auf. Taggen Sie Ihre Fragen mit [MicrosoftGraph] und [office365].



