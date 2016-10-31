# <a name="get-started-with-microsoft-graph-and-rest"></a>Introducción a Microsoft Graph y REST

En este artículo, se describen las tareas necesarias para obtener un token de acceso desde el punto de conexión v2.0 de Azure AD y llamar a Microsoft Graph mediante llamadas de REST. Describe la secuencia de solicitudes y respuestas que una aplicación usa para autenticar y recuperar mensajes de correo electrónico.

En primer lugar, necesitará registrar la aplicación con Azure Active Directory (Azure AD). 

## <a name="register-the-application"></a>Registrar la aplicación

Actualmente, hay dos opciones para registrar su aplicación con Azure AD.

  - Registre una aplicación para usar el punto de conexión v2.0 de Azure AD que funcione tanto con identidades personales (Microsoft) como con cuentas profesionales y educativas (Azure AD).
  - Registre una aplicación para usar el punto de conexión de Azure AD que admita solo cuentas profesionales y educativas.

Si está leyendo este artículo, se supone que ya ha realizado el registro de v2.0, así que deberá registrar su aplicación en el [Portal de registro de aplicaciones](https://apps.dev.microsoft.com/). Siga las instrucciones de [Registrar su aplicación de Microsoft Graph con el punto de conexión v2.0 de Azure AD](../authorization/auth_register_app_v2.md) para registrar su aplicación. Para obtener información acerca de cómo usar el punto de conexión de Azure AD, consulte [Realizar la autenticación mediante Azure AD](../authorization/app_authorization.md).

> Existen algunas limitaciones en el uso del punto de conexión v2.0. Para decidir si es o no la opción que más le conviene, consulte [¿Debería usar el punto de conexión v2.0?](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/)

## <a name="authenticate-the-user-and-get-an-access-token"></a>Autenticar al usuario y obtener un token de acceso

La aplicación que se describe en este artículo implementa el [flujo de concesión del código de autorización](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols-oauth-code/) para obtener los tokens de acceso desde el punto de conexión v2.0 de Azure AD, siguiendo los [protocolos de OAuth 2.0 estándares](http://tools.ietf.org/html/rfc6749). Para obtener una guía completa de los flujos que se admiten en el punto de conexión v2.0 de Azure AD, consulte [Tipos de punto de conexión v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-flows/).

Con el flujo de concesión del código de autorización, primero obtiene un código de autorización y, a continuación, lo intercambia por un token de acceso.

### <a name="getting-an-authorization-code"></a>Obtener un código de autorización

El primer paso en el flujo de concesión del código de autorización es obtener un código de autorización. El código se devuelve a la aplicación mediante el servidor de autorización cuando el usuario inicia sesión y da su consentimiento a los permisos que solicita la aplicación.

Para solicitar un código de autorización, deberá enviar una solicitud GET al punto de conexión v2.0 de Azure AD. Esta URL debe estar abierta en un explorador para que el usuario pueda iniciar sesión y proporcionar su consentimiento.

#### <a name="construct-the-request"></a>Crear la solicitud

1 - Empiece con la URL base:

```
https://login.microsoftonline.com/<tenant>/oauth2/v2.0/authorize
```

El segmento de *inquilino* en la ruta de acceso controla quién puede iniciar sesión en la aplicación. Los valores permitidos son los siguientes: *comunes*, *organizaciones*, *consumidores* e identificadores de inquilino. Para obtener más información, consulte [Puntos de conexión de protocolo](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/#endpoints).

2 - Anexe los parámetros de consulta a la URL base. Estos se usan para identificar la aplicación, los permisos solicitados y el resto de la información de la solicitud de autenticación. En la siguiente tabla, se describen algunos parámetros comunes:

| Parámetro | Descripciones |
|:------|:------|
| client_id | El identificador de la aplicación que se genera al registrar la aplicación. Esto permite que Azure AD conozca qué aplicación está solicitando el inicio de sesión. |
| redirect_uri | La ubicación a la que Azure redirigirá cuando el usuario haya dado su consentimiento a la aplicación. Este valor debe corresponder al valor del **URI de redireccionamiento** que se usó al registrar la aplicación. |
| response_type | El tipo de respuesta que espera la aplicación. Este valor es `code` para el flujo de concesión del código de autorización. |
| ámbito | Una lista de los ámbitos de permisos de [Microsoft Graph](../authorization/permission_scopes.md) que está solicitando la aplicación separados por espacios. También puede especificar [ámbitos de OpenId Connect](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-scopes/#openid-connect-scopes) para el [inicio de sesión único](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols-oidc/).   |
| estado | Valor que se incluye en la solicitud, que también se devolverá en la respuesta del token y que se usa para la validación. |

Por ejemplo, la URL de solicitud de una aplicación que requiriera acceso de lectura al correo sería similar a la que se muestra a continuación.

```
GET https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=<app ID>&redirect_uri=http%3A%2F%2Flocalhost/myapp%2F&response_type=code&state=1234&scope=mail.read
```

3- Redirija al usuario a la URL de inicio de sesión. Se le presentará al usuario una pantalla de inicio de sesión que mostrará el nombre de la aplicación. Después de que inicie sesión, al usuario se le presentará una lista de los permisos que requiere la aplicación y se le solicitará que los permita o que los deniegue. Si el usuario da su consentimiento, el explorador le redirigirá al URI de redireccionamiento con el código de autorización y el estado en la cadena de consulta, tal y como se muestra en el siguiente ejemplo.

```
http://localhost/myapp/?code=AwABAAAA...cZZ6IgAA&state=1234
```

El paso siguiente consiste en intercambiar el código de autorización devuelto por un token de acceso.

### <a name="getting-an-access-token"></a>Obtener un token de acceso

Para obtener un token de acceso, la aplicación publica los parámetros del formulario codificado a la URL de solicitud del token (por ejemplo, `https://login.microsoftonline.com/common/oauth2/v2.0/token`) con los siguientes parámetros.

| Parámetro | Descripciones |
|:------|:------|
| client_id | El identificador de la aplicación que se genera al registrar la aplicación. |
| client_secret | El secreto de aplicación que se genera al registrar la aplicación. |
| código | El código de autorización que se obtuvo en el paso anterior. |
| redirect_uri | Este valor debe ser el mismo que el valor usado en la solicitud del código de autorización. |
| grant_type | El tipo de concesión que está usando la aplicación. Este valor es `code` para el flujo de concesión del código de autorización. |
| ámbito | Una lista de los ámbitos de permisos de [Microsoft Graph](../authorization/permission_scopes.md) que está solicitando la aplicación separados por espacios. |

La URL de solicitud para nuestra aplicación, usando el código del paso anterior, tiene el aspecto que se muestra a continuación.

```
POST https://login.microsoftonline.com/common/oauth2/v2.0/token
Content-Type: application/x-www-form-urlencoded

{
  grant_type=authorization_code
  &code=AwABAAAA...cZZ6IgAA
  &redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
  &resource=https%3A%2F%2Fgraph.microsoft.com%2F
  &scope=mail.read
  &client\_id=<app ID>
  &client\_secret=<app SECRET>
}
```

El servidor responde con una carga JSON que incluye el token de acceso.

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
  "token_type":"Bearer",
  "expires_in":"3600",
  "access_token":"eyJ0eXAi...b66LoPVA",
  "scope":"https://graph.microsoft.com/mail.read",
  ...
}
```

El token de acceso se encuentra en el campo `access_token` de la carga JSON. La aplicación usa este valor para establecer el encabezado de autorización al realizar las llamadas REST a la API.

## <a name="call-microsoft-graph"></a>Llamar a Microsoft Graph

Cuando la aplicación tenga un token de acceso, estará lista para llamar a Microsoft Graph. Dado que esta aplicación de ejemplo está recuperando mensajes, usará una solicitud HTTP GET al punto de conexión `https://graph.microsoft.com/v1.0/me/messages`.

### <a name="refine-the-request"></a>Restringir la solicitud

Las aplicaciones pueden controlar el comportamiento de las solicitudes GET usando parámetros de consulta OData. Se recomienda que las aplicaciones usen estos parámetros para limitar el número de resultados que se devuelven y los campos que se devuelven para cada elemento. 

Esta aplicación de ejemplo mostrará los mensajes en una tabla que muestra el asunto, el remitente y la fecha y hora en que se recibió el mensaje. La tabla muestra un máximo de 25 filas y está ordenada de forma que los mensajes recibidos recientemente se encuentren en la parte superior. La aplicación usa los siguientes parámetros de consulta para obtener estos resultados.

- `$select` - Especifica solo los campos `subject`, `sender`, y `dateTimeReceived`.
- `$top` - Especifica un máximo de 25 elementos.
- `$orderby` - Ordena los resultados por el campo `dateTimeReceived`.

Esto da como resultado la siguiente solicitud.

```
GET https://graph.microsoft.com/v1.0/me/messages?$select=subject,from,receivedDateTime&$top=25&$orderby=receivedDateTime%20DESC
Accept: application/json
Authorization: Bearer eyJ0eXAi...b66LoPVA
```

Ahora que ha visto cómo realizar llamadas a Microsoft Graph, puede usar la referencia de la API para construir cualquier otro tipo de llamada que su aplicación necesite realizar. Sin embargo, tenga en cuenta que su aplicación debe tener los permisos adecuados configurados en el registro de aplicaciones para las llamadas que realice.

## <a name="next-steps"></a>Pasos siguientes
- Pruebe más características de la API de REST mediante el [Probador de Graph](https://graph.microsoft.io/graph-explorer).

## <a name="see-also"></a>Recursos adicionales
- [Protocolos de Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
- [Tokens de Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)
