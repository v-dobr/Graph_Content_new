# <a name="get-started-with-microsoft-graph-in-a-python-app"></a>Introducción a Microsoft Graph en una aplicación de Python 

En este artículo, se describen las tareas necesarias para obtener un token de acceso desde Azure AD y llamar a Microsoft Graph. Le muestra los pasos para la compilación del [Ejemplo Connect de Microsoft Graph para Python](https://github.com/microsoftgraph/python3-connect-rest-sample) y explica los conceptos principales que implementará para usar la API de Microsoft Graph. El artículo describe cómo obtener acceso a Microsoft Graph mediante llamadas de REST directas.

![Captura de pantalla del ejemplo Connect de Python de Office 365](./images/web-screenshot.png)

> **Nota**: Este tutorial y el ejemplo en el que se basa usan el punto de conexión de Azure AD. Compruébelo de nuevo pronto para obtener versiones actualizadas que usen el punto de conexión v2.0 de Azure AD.

##  <a name="prerequisites"></a>Requisitos previos

  * Una cuenta de Office 365 para empresas. Puede registrarse para obtener [una suscripción a Office 365 Developer](https://msdn.microsoft.com/en-us/office/office365/howto/setup-development-environment#bk_Office365Account), que incluirá los recursos que necesita para comenzar a compilar aplicaciones de Office 365.
  * [Ejemplo Connect de Microsoft Graph para Python](https://github.com/microsoftgraph/python3-connect-rest-sample)

## <a name="register-the-application-in-azure-active-directory"></a>Registrar la aplicación en Azure Active Directory

En primer lugar, deberá registrar la aplicación y establecer los permisos para usar Microsoft Graph. Esto permite a los usuarios iniciar sesión en la aplicación con cuentas profesionales o educativas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. En la barra superior, haga clic en su cuenta y, en la lista **Directorio**, elija el espacio empresarial de Active Directory donde desee registrar la aplicación.
3. Haga clic en **Más servicios**, en la barra de navegación izquierda y elija **Azure Active Directory**.
4. Haga clic en **Registros de aplicaciones** y elija **Agregar**.
5. Escriba un nombre descriptivo para la aplicación, por ejemplo, "MSGraphConnexiónPython" y seleccione "Aplicación web o API" como **Tipo de aplicación**. Como URL de inicio de sesión, escriba "http://127.0.0.1:8000/connect/get_token/". Haga clic en **Crear** para crear la aplicación.
6. Sin salir del portal de Azure, elija la aplicación, haga clic en **Configuración** y seleccione **Propiedades**.
7. Busque el valor del identificador de la aplicación y cópielo en el Portapapeles.
8. Configurar los permisos de la aplicación:
9. En el menú **Configuración**, elija la sección **Permisos requeridos**, haga clic en **Agregar** y, a continuación, en **Seleccionar API**. Posteriormente, seleccione **Microsoft Graph**.
10. A continuación, haga clic en Seleccionar permisos y seleccione **Iniciar sesión y leer el perfil del usuario** y **Enviar correo como un usuario**. Haga clic en **Seleccionar** y después en **Hecho**.
11. En el menú **Configuración**, elija la sección **Claves**. Escriba una descripción y seleccione una duración para la clave. Haga clic en **Guardar**.
12. **Importante**: Copie el valor de clave. No podrá volver a obtener acceso a este valor una vez que abandone este panel. Usará este valor como secreto de aplicación.

## <a name="redirect-the-browser-to-the-sign-in-page"></a>Redirigir al explorador a la página de inicio de sesión

Su aplicación necesita redirigir al explorador a la página de inicio de sesión para iniciar el flujo de OAuth y obtener un código de autorización. 

En el ejemplo Connect, el siguiente código (ubicado en [*connect/auth_helper.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/auth_helper.py)) genera la dirección URL que la aplicación necesita para redirigir al usuario y se canaliza a la vista en la que puede usarse para el redireccionamiento. 

```python
# This function creates the signin URL that the app will
# direct the user to in order to sign in to Office 365 and
# give the app consent.
def get_signin_url(redirect_uri):
  # Build the query parameters for the signin URL.
  params = { 'client_id': client_id,
             'redirect_uri': redirect_uri,
             'response_type': 'code'
           }

  authorize_url = '{0}{1}'.format(authority, '/common/oauth2/authorize?{0}')
  signin_url = authorize_url.format(urlencode(params))
  return signin_url
```

<!--<a name="authCode"></a>-->
## <a name="receive-an-authorization-code-in-your-reply-url-page"></a>Recibir un código de autorización en la página de la dirección URL de respuesta

Después de que el usuario inicie sesión, el explorador se redirige a su dirección URL de respuesta, la función ```get_token``` en [*connect/views.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/views.py), con un código de autorización anexado a la cadena de consulta como la variable ```code```. 

En el ejemplo Connect, se obtiene el código de la cadena de consulta para que, posteriormente, se pueda intercambiar por un token de acceso.

```python
auth_code = request.GET['code']
```

<!--<a name="accessToken"></a>-->
## <a name="request-an-access-token-from-the-token-issuing-endpoint"></a>Solicitar un token de acceso desde el token que emite un punto de conexión

Cuando tenga el código de autorización, podrá usarlo junto con los valores de los parámetros ID de cliente, Clave y Dirección URL de respuesta que obtuvo en Azure Active Directory para solicitar un token de acceso. 

> **Nota**: La solicitud debe especificar también un recurso que esté intentando usar. En el caso de Microsoft Graph, el valor de recurso es `https://graph.microsoft.com`.

En el ejemplo Connect, se solicita un token en la función ```get_token_from_code``` en el archivo [*connect/auth_helper.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/auth_helper.py).

```python
# This function passes the authorization code to the token
# issuing endpoint, gets the token, and then returns it.
def get_token_from_code(auth_code, redirect_uri):
  # Build the post form for the token request
  post_data = { 'grant_type': 'authorization_code',
                'code': auth_code,
                'redirect_uri': redirect_uri,
                'client_id': client_id,
                'client_secret': client_secret,
                'resource': 'https://graph.microsoft.com'
              }
              
  r = requests.post(token_url, data = post_data)
  
  try:
    return r.json()
  except:
    return 'Error retrieving token: {0} - {1}'.format(r.status_code, r.text)
```

> **Nota**: La respuesta proporciona más información aparte del token de acceso. Por ejemplo, su aplicación puede obtener un token de actualización para solicitar nuevos tokens de acceso sin que el usuario tenga que iniciar sesión de nuevo explícitamente.

<!--<a name="request"></a>-->
## <a name="use-the-access-token-in-a-request-to-the-microsoft-graph-api"></a>Usar el token de acceso en una solicitud a la API de Microsoft Graph

El token de acceso permite que su aplicación cree solicitudes autenticadas en la API de Microsoft Graph. Su aplicación debe anexar el token de acceso al encabezado de **autorización** de cada solicitud.

En el ejemplo Connect, se envía un correo electrónico usando el punto de conexión ```me/microsoft.graph.sendMail``` en la API de Microsoft Graph. El código está en la función ```call_sendMail_endpoint``` del archivo [*connect/graph_service.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/graph_service.py). Este es el código que muestra cómo anexar el código de acceso al encabezado de autorización.

```python
# Set request headers.
headers = { 
  'User-Agent' : 'python_tutorial/1.0',
  'Authorization' : 'Bearer {0}'.format(access_token),
  'Accept' : 'application/json',
  'Content-Type' : 'application/json'
}
```

> **Nota**: La solicitud también debe enviar un encabezado **Content-Type** con un valor que acepte la API de Graph. Por ejemplo, `application/json`.

La API de Microsoft Graph es una interfaz unificadora y muy avanzada que permite trabajar con todos los tipos de datos de Microsoft. Consulte la referencia de la API para ver todas las posibilidades que ofrece Microsoft Graph.