# <a name="call-microsoft-graph-in-a-service-or-daemon-app"></a>Llamar a Microsoft Graph en una aplicación de servicio o demonio

En este artículo se describen las tareas mínimas necesarias para conectar una aplicación de servicio o demonio de espacio empresarial único a Office 365 y llamar a la API de Microsoft Graph.

> Nota: En este tema, se trata el uso de Azure Active Directory como proveedor de autenticación para su aplicación. Para implementar un servicio o una aplicación Daemon con la versión 2.0 del extremo de Azure AD, consulte <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols-oauth-client-creds/" target="_newtab">Protocolos de versión 2.0: flujo de credenciales del cliente de OAuth 2.0</a>.

## <a name="register-the-application-in-azure-active-directory"></a>Registrar la aplicación en Azure Active Directory

Antes de empezar a trabajar con Office 365, necesitará registrar la aplicación en [Azure Portal](https://portal.azure.com). Tenga en cuenta lo siguiente:

- Después de registrar la aplicación, deberá configurar los permisos de la aplicación que requiere la aplicación de servicio o demonio.
- Anote los siguientes valores en su registro de aplicaciones de Azure ya que los necesitará para configurar el flujo de OAuth en su aplicación de servicio o demonio:
    * Identificador de la aplicación (exclusivo de la aplicación)
    * Clave de aplicación o secreto (exclusivo de la aplicación)
    * El punto de conexión del token de OAuth 2.0 de su aplicación
      * Para ver este valor, haga clic en *Ver puntos de conexión* en la parte inferior del Portal de administración de Azure, en la página de su aplicación. El punto de conexión será algo similar a `https://login.microsoftonline.com/<tenantId>/oauth2/token`.

## <a name="request-an-access-token-from-the-token-issuing-endpoint"></a>Solicitar un token de acceso desde el token que emite un punto de conexión

A diferencia de las aplicaciones cliente, la aplicación de servicio o demonio no puede tener un inicio de sesión de usuario ni puede autorizar su aplicación. En lugar de suplantar a un usuario, la aplicación tiene que implementar el flujo de concesión de credenciales de cliente de OAuth 2.0; este flujo le permitirá usar sus propias credenciales, su Identificador de cliente y una clave de aplicación con los que se autenticará al llamar a Microsoft Graph. Para obtener detalles acerca del flujo de autenticación, consulte [Llamadas entre servicios mediante las credenciales del cliente](https://msdn.microsoft.com/en-us/library/azure/dn645543.aspx).

Realice una solicitud POST de HTTP en el punto de conexión que emite los tokens con los siguientes parámetros, reemplazando `<clientId>` y `<clientSecret>` con el identificador de cliente de la aplicación y la clave de la aplicación, respectivamente.

```http
POST https://login.microsoftonline.com/<tenantId>/oauth2/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials
&client_id=<clientId>
&client_secret=<clientSecret>
&resource=https://graph.microsoft.com
```

La respuesta incluirá un acceso token e información de expiración.

```json
{ 
  "token_type": "Bearer",
  "expires_in": "3599",
  "scope": "User.Read",
  "expires_on": "1449685363",
  "not_before": "1449681463",
  "resource": "https://graph.microsoft.com",
  "access_token": "<token>"
}
```

## <a name="use-the-access-token-in-a-request-to-the-microsoft-graph-api"></a>Usar el token de acceso en una solicitud a la API de Microsoft Graph

El token de acceso permite que su aplicación cree solicitudes autenticadas en la API de Microsoft Graph. Su aplicación debe anexar el token de acceso al encabezado de **autorización** de cada solicitud.

Por ejemplo, una aplicación de servicio o demonio puede recuperar todos los usuarios de un inquilino si tiene el permiso *Leer los perfiles completos de todos los usuarios* seleccionado en el Portal de administración de Azure. 

```http
GET https://graph.microsoft.com/v1.0/users
Authorization: Bearer <token>
```
