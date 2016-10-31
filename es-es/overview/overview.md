# <a name="overview-of-microsoft-graph"></a>Información general de Microsoft Graph

Microsoft Graph expone varias API de Office 365 y de otros servicios en la nube de Microsoft a través de un único punto de conexión: **https://graph.microsoft.com**. Microsoft Graph simplifica las consultas que, de lo contrario, serían más complejas. 
 
Puede usar Microsoft Graph para:

- Acceder a los datos de varios servicios en la nube de Microsoft, incluidos Azure Active Directory, Exchange Online como parte de Office 365, SharePoint, OneDrive, OneNote y Planner
- Explorar las entidades y las relaciones
- Acceder a los datos y a la información de la nube de Microsoft (para usuarios comerciales)

**Pila de desarrollo de Microsoft Graph**

![Un diagrama en el que se muestran los niveles de la pila de desarrollo de Microsoft Graph. En la parte inferior, encontrará el nivel de datos, que incluye los usuarios, los grupos, los archivos, los calendarios, los contactos personales, el correo electrónico, las tareas, los contactos de la organización, Excel y las notas. La siguiente capa es la de autenticación y autorización. A continuación se encuentra el entorno de desarrollo que elija, como Android, iOS o los SDK de la API de Microsoft Graph para Visual Studio. La capa final es su solución, que usa la tecnología que elija, como .NET, JS, HTML o Ruby, y que se aloja en Microsoft Azure o en cualquier otra plataforma.](./images/MicrosoftGraph_DevStack.png)

<!--<a name="msg_queries"> </a>-->

##<a name="common-microsoft-graph-queries"></a>Consultas comunes de Microsoft Graph

Microsoft Graph expone dos puntos de conexión: /v1.0 y /beta. El punto de conexión /v1.0 incluye los recursos a los que puede obtener acceso en su aplicación de producción. El punto de conexión [/beta](http://graph.microsoft.io/en-us/docs/api-reference/beta/beta-overview) incluye las API que todavía están en versión preliminar. La siguiente tabla enumera algunas consultas comunes que puede usar para obtener acceso a la API de Microsoft Graph.

| **Operación** | **Punto de conexión de servicio** |
|:--------------------------|:----------------------------------------|
|   OBTENER mi perfil |    [`https://graph.microsoft.com/v1.0/me`](/graph-explorer/#?request=me&version=v1.0) |
|   OBTENER mis archivos | [`https://graph.microsoft.com/v1.0/me/drive/root/children`](/graph-explorer/#?request=me%2Fdrive%2Froot%2Froot%2Fchildren&version=v1.0) |
|   OBTENER mi foto     | [`https://graph.microsoft.com/v1.0/me/photo/$value`](/graph-explorer/#?request=me%2Fphoto%2F%24value&version=v1.0) |
|   OBTENER mi correo |   [`https://graph.microsoft.com/v1.0/me/messages`](/graph-explorer/#?request=me%2Fmessages&version=v1.0) |
|   OBTENER mi correo electrónico de importancia alta | [`https://graph.microsoft.com/v1.0/me/messages?$filter=importance%20eq%20'high'`](/graph-explorer/#?request=me%2Fmessages%3F%24filter%3Dimportance%2520eq%2520'high'&version=v1.0) |
|   OBTENER mi calendario |   [`https://graph.microsoft.com/v1.0/me/calendar`](/graph-explorer/#?request=me%2Fcalendar&version=v1.0) |
|   OBTENER mi administrador  | [`https://graph.microsoft.com/v1.0/me/manager`](/graph-explorer/#?request=me%2Fmanager&version=v1.0) |
|   OBTENER el último usuario que modificó el archivo foo.txt |  [`https://graph.microsoft.com/v1.0/me/drive/root/children/foo.txt/lastModifiedByUser`](/graph-explorer/#?request=me%2Fdrive%2Froot%2Froot%2Fchildren%2Ffoo.txt%2FlastModifiedByUser&version=v1.0) |
|   OBTENER grupos unificados de los que soy miembro|   [`https://graph.microsoft.com/v1.0/me/memberOf/$/microsoft.graph.group?$filter=groupTypes/any(a:a%20eq%20'unified')`](/graph-explorer/#?request=me%2FmemberOf%2F%24%2Fmicrosoft.graph.group%3F%24filter%3DgroupTypes%2Fany(a%3Aa%2520eq%2520'unified'&version=v1.0)) |
|   OBTENER usuarios de mi organización     | [`https://graph.microsoft.com/v1.0/users`](/graph-explorer/#?request=users&version=v1.0) |
|   OBTENER conversaciones de grupo |   `https://graph.microsoft.com/v1.0/groups/<id>/conversations`|
|   OBTENER usuarios relacionados conmigo    | [`https://graph.microsoft.com/beta/me/people`](/graph-explorer/#?request=me%2Fpeople&version=beta)  |
|   OBTENER archivos de tendencias a mi alrededor |  [`https://graph.microsoft.com/beta/me/trendingAround`](/graph-explorer/#?request=me%2FtrendingAround&version=beta) |
|   OBTENER usuarios con las que estoy trabajando     | [`https://graph.microsoft.com/beta/me/workingWith`](/graph-explorer/#?request=me%2FworkingWith&version=beta) |
|   OBTENER mis tareas    | [`https://graph.microsoft.com/beta/me/tasks`](/graph-explorer/#?request=me%2Ftasks&version=beta) |
|   OBTENER mis notas |  [`https://graph.microsoft.com/beta/me/notes/notebooks`](/graph-explorer/#?request=me%2Fnotes%2Fnotebooks&version=beta) |


>**Nota:** Las API del punto de conexión beta están sujetos a cambios. No le recomendamos que las use en sus aplicaciones de producción. 

<!-- <a name="msg_roof"> </a> -->

## <a name="explore-microsoft-graph"></a>Explorar Microsoft Graph

- [Empiece](../get-started/get-started.md) a usar Microsoft Graph y la plataforma que desee.
- Descubra los recursos y las operaciones que puede usar en sus aplicaciones de producción examinando la tabla de contenido.
- Descubra las nuevas [/API en fase beta](http://graph.microsoft.io/en-us/docs/api-reference/beta/beta-overview).
- Visite el [Explorador de Microsoft Graph](https://graph.microsoft.io/en-us/graph-explorer).

 >  Su opinión es importante para nosotros. Conecte con nosotros en [Stack Overflow](http://stackoverflow.com/questions/tagged/office365+or+microsoftgraph). Etiquete sus preguntas con [MicrosoftGraph] y [office365].



