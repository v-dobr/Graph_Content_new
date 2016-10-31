# <a name="known-issues-with-microsoft-graph"></a>Problemas conocidos de Microsoft Graph

En este artículo, se describen los problemas conocidos de Microsoft Graph. Para obtener información acerca de las actualizaciones más recientes, consulte el [Registro de cambios de Microsoft Graph](http://graph.microsoft.io/en-us/changelog).

## <a name="users"></a>Usuarios
#### <a name="no-instant-access-after-creation"></a>No hay acceso instantáneo después de la creación
Los usuarios pueden crearse inmediatamente a través de un POST en la entidad del usuario. Una licencia de Office 365 primero se debe asignar a un usuario, con el fin de obtener acceso a los servicios de Office 365. Incluso entonces, debido a la naturaleza distribuida del servicio, puede llevar 15 minutos que los archivos, mensajes y entidades de eventos estén disponibles para su uso para este usuario, a través de la API de Microsoft Graph. Durante este tiempo, las aplicaciones recibirán una respuesta de error HTTP 404. 

#### <a name="photo-restrictions"></a>Restricciones de la foto
Leer y actualizar la foto de perfil de un usuario solo es posible si el usuario tiene un buzón. Además, cualquier foto que *pueda* haberse almacenado previamente mediante la propiedad **thumbnailPhoto** (usando la versión preliminar de la API unificada de Office 365 o Azure AD Graph, o bien mediante la sincronización de AD Connect) ya no será accesible mediante la propiedad de foto del usuario de Microsoft Graph. En este caso, no poder leer o actualizar una foto provocaría el error siguiente:

```javascript
    {
      "error": {
        "code": "ErrorNonExistentMailbox",
        "message": "The SMTP address has no mailbox associated with it."
      }
    }
```

 > **NOTA**:  Poco después de GA, se habilitará el almacenamiento y la recuperación de las fotos de perfil del usuario, incluso cuando el usuario no tenga un buzón, y este error debería desaparecer.

#### <a name="default-contacts-folder"></a>Carpeta de contactos predeterminada

En la versión `/v1.0`, `GET /me/contactFolders` no incluye la carpeta de contactos predeterminada del usuario. 

Estará disponible una corrección. Mientras tanto, puede usar la siguiente consulta [enumerar contactos](http://graph.microsoft.io/docs/api-reference/v1.0/api/user_list_contacts) y la propiedad **parentFolderId** como una solución alternativa para obtener el identificador de la carpeta de contactos predeterminada:

```
GET https://graph.microsoft.com/v1.0/me/contacts?$top=1&$select=parentFolderId
```
En la consulta anterior:
1. `/me/contacts?$top=1` obtiene las propiedades de un elemento [contact](http://graph.microsoft.io/docs/api-reference/v1.0/resources/contact) de la carpeta de contactos predeterminada.
2. Al anexar `&$select=parentFolderId` se devuelve solo la propiedad **parentFolderId** del contacto, que es el identificador de la carpeta de contactos predeterminada.

#### <a name="adding-and-accessing-ics-based-calendars-in-user's-mailbox"></a>Agregar y acceder a calendarios basados en archivos ICS en el buzón del usuario
Actualmente, existe una compatibilidad parcial con un calendario basado en una suscripción a calendarios de Internet (ICS):
* Puede agregar un calendario basado en ICS a un buzón de usuario mediante la interfaz de usuario, pero no mediante la API de Microsoft Graph. 
* [Enumerar los calendarios del usuario](http://graph.microsoft.io/docs/api-reference/v1.0/api/user_list_calendars) le permite obtener las propiedades **name**, **color** e **id** de cada [calendario](http://graph.microsoft.io/docs/api-reference/v1.0/resources/calendar) en el grupo de calendarios predeterminado del usuario, o en un grupo de calendarios especificado, incluidos los calendarios basados en ICS. No puede almacenar ni acceder a la dirección URL de una ICS en el recurso del calendario.
* También puede [enumerar los eventos](http://graph.microsoft.io/docs/api-reference/v1.0/api/calendar_list_events) de un calendario basado en ICS.

## <a name="groups"></a>Grupos
#### <a name="policy"></a>Directiva
Usar Microsoft Graph para crear y nombrar un grupo unificado omite cualquier directiva de grupo unificada que esté configurada a través de Outlook Web App. 

#### <a name="group-permission-scopes"></a>Ámbitos de permiso de grupo
Microsoft Graph expone dos ámbitos de permisos (*Group.Read.All* y *Group.ReadWrite.All*) para obtener acceso a las API de grupos.  Estos ámbitos de permisos deben ser aceptados por un administrador (lo que supone un cambio respecto a la versión preliminar).  En el futuro, planeamos agregar nuevos ámbitos para grupos que pueden ser aceptados por los usuarios.

#### <a name="adding-and-getting-attachments-of-group-posts"></a>Agregar y obtener los datos adjuntos de las publicaciones de grupo
Actualmente, al [agregar](http://graph.microsoft.io/docs/api-reference/v1.0/api/post_post_attachments) datos adjuntos a las publicaciones de grupo, así como al [enumerar](http://graph.microsoft.io/docs/api-reference/v1.0/api/post_list_attachments) y obtener los datos adjuntos de las publicaciones de grupo, se devuelve el mensaje de error "La solicitud de OData no es compatible". Se ha desarrollado una corrección tanto para la versión `/v1.0` como para la `/beta` y se espera que esté disponible a finales de enero de 2016.

## <a name="contacts"></a>Contactos
* Solo los contactos personales son compatibles actualmente. Actualmente no se admiten contactos de la organización en `/v1.0`, pero pueden encontrarse en `/beta`.
* No se está devolviendo el teléfono móvil del contacto personal a un contacto. Se agregará en breve. Mientras tanto, puede tener acceso a este a través de las API de Outlook.

### <a name="drives,-files-and-content-streaming"></a>Unidades, archivos y streaming de contenido
* La primera vez que accede a una unidad personal del usuario a través de Microsoft Graph antes de que el usuario acceda a su sitio personal a través del explorador, se produce una respuesta 401.
* La carga y descarga de archivos (archivos de grupos de Office, unidades o datos adjuntos de correo) está limitada a 4 MB.

## <a name="functionality-available-only-in-office-365-rest-apis"></a>Esta funcionalidad solo está disponible en las API de REST de Office 365

Algunas funciones todavía no están disponibles en Microsoft Graph. Si no ve la funcionalidad que busca, puede usar las [API de REST de Office 365](https://msdn.microsoft.com/en-us/office/office365/api/api-catalog) específicas del punto de conexión.

#### <a name="synchronization"></a>Sincronización
Las funcionalidades de sincronización de Outlook, OneDrive y Azure AD (en Azure AD también se conoce como consulta diferencial) no están disponibles en `/v1.0` ni en `/beta`.  Si su aplicación requiere funcionalidades de sincronización, continúe usando las API de REST de Azure AD y Office 365 existentes o explore la nueva versión preliminar de la característica Webhooks que se ofrece a través de Microsoft Graph para eventos, mensajes y contactos.

> **NOTA**: Nuestro objetivo es contrarrestar la diferencia entre las API existentes y Microsoft Graph lo más rápidamente posible, incluida la sincronización.

#### <a name="batching"></a>Procesamiento por lotes
El procesamiento por lotes no se admite en Microsoft Graph. Sin embargo, puede usar el punto de conexión en versión beta de Outlook y [las llamadas REST por lotes de Outlook](https://msdn.microsoft.com/en-us/office/office365/api/batch-outlook-rest-requests). 

#### <a name="availability-in-china"></a>Disponibilidad en China
El servicio de Microsoft Graph está operado por 21Vianet (y ahora disponible en China). Consulte [Implementaciones de nube soberana de Microsoft Graph](http://graph.microsoft.io/docs/overview/deployments) para obtener más información, incluidas las restricciones.

#### <a name="service-actions-and-functions"></a>Funciones y acciones del servicio
`isMemberOf` y `getObjectsById` no están disponibles en Microsoft Graph

## <a name="microsoft-graph-permissions"></a>Permisos de Microsoft Graph
Consulte [Ámbitos de permiso](http://graph.microsoft.io/docs/authorization/permission_scopes) para obtener la información más reciente acerca de los permisos delegados y de aplicación que admite Microsoft Graph. Además, se aplican a `v1.0` las siguientes limitaciones:

|Permiso |   Tipo de permiso | Limitación |  Alternativa |
|-----------|-----------------|------------|--------------|
|_User.ReadWrite_| Delegado    | No se puede actualizar el número de teléfono móvil|    Seleccionar también `Directory.AccessAsUser.All`| 
|_User.ReadWrite.All_|  Delegado|  No se puede realizar ninguna operación CRUD en `User` que no sea la actualización de la foto HD del usuario y las propiedades de perfil extendidas| Seleccione también `Directory.ReadWrite.All` o `Directory.AccessAsUser.All` si se requiere la eliminación del usuario.|
|_User.Read.All_|   Aplicación |No se puede realizar ninguna operación de lectura en otros usuarios| Seleccionar también `Directory.Read.All`|
| _User.ReadWrite.All_ |    Aplicación |   No se puede realizar ninguna operación CRUD en `User` que no sea la actualización de la foto HD del usuario y las propiedades de perfil extendidas |    Seleccionar también`Directory.ReadWrite.All`. **NOTA**: La eliminación del usuario no será posible.|
|_Group.Read.All_   | Aplicación | No se pueden enumerar los grupos ni las pertenencias a estos.  Todavía se puede leer el contenido del grupo para los grupos de Office   | Seleccionar también `Directory.Read.All` |
|_Group.ReadWrite.All_  | Aplicación   | No se pueden enumerar grupos ni pertenencias a grupos, crear grupos, actualizar pertenencias a grupos ni eliminar grupos.  Todavía se puede leer y actualizar el contenido del grupo para los grupos de Office.   | Seleccionar también `Directory.ReadWrite.All`. **NOTA**:  La eliminación del grupo no será posible.|

Además, existen las siguientes `/beta` limitaciones:

|Permiso |   Tipo de permiso | Limitación |  Alternativa |
|-----------|-----------------|------------|--------------|
| _Group.ReadWrite.All_ | Delegado | No se puede leer ni actualizar las tareas del planificador de los grupos de Office  | Seleccionar también `Tasks.ReadWrite`|
|_Tasks.ReadWrite_  | Delegado | No se puede leer ni actualizar las tareas del usuario que inició sesión| Seleccionar también `Group.ReadWrite.All`|

## <a name="odata-related-limitations"></a>Limitaciones relacionadas con OData
* Limitaciones de **$expand**: 
 * No se admite para `nextLink`
 * No se admite para más de un nivel de expansión
 * No se admite con parámetros adicionales (**$filter**, **$select**)
* No se admiten varios espacios de nombres
* Las solicitudes GET en `$ref` y la conversión no se admiten en usuarios, grupos, dispositivos, entidades de servicio y aplicaciones.
* `@odata.bind` no se admite.  Esto significa que los desarrolladores no podrán establecer correctamente `Accepted` o `RejectedSenders` en un grupo.
* `@odata.id` no está presente en las navegaciones de no contención (como los mensajes) cuando se usan metadatos mínimos
* No está disponible el filtrado o la búsqueda de carga de trabajo cruzada. 
* La búsqueda de texto completo (con **$search**) solo está disponible para algunas entidades, como los mensajes.

  >  Su opinión es importante para nosotros. Conecte con nosotros en [Stack Overflow](http://stackoverflow.com/questions/tagged/office365). Etiquete sus preguntas con [MicrosoftGraph] y [office365].

  
             .

