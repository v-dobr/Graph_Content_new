# <a name="get-started-with-microsoft-graph-in-a-xamarin-forms-app"></a>Introducción a Microsoft Graph en una aplicación de Xamarin Forms

En este artículo se describen las tareas necesarias para obtener un token de acceso desde el punto de conexión de [Azure AD v2.0](https://graph.microsoft.io/en-us/docs/authorization/converged_auth) y llamar a Microsoft Graph. Le guiará por el código del [Ejemplo Connect de Microsoft Graph para Xamarin Forms](https://github.com/microsoftgraph/xamarin-csharp-connect-sample) para explicar los conceptos principales que se deben implementar en una aplicación que use Microsoft Graph. El artículo también describe cómo obtener acceso a Microsoft Graph mediante la [Biblioteca cliente de Microsoft Graph](http://www.nuget.org/packages/Microsoft.Graph/).

Esta es la aplicación que va a crear.

| UWP | Android | iOS |
| --- | ------- | ----|
| <img src="images/UWP.png" alt="Connect sample on UWP" width="100%" /> | <img src="images/Droid.png" alt="Connect sample on Android" width="100%" /> | <img src="images/iOS.png" alt="Connect sample on iOS" width="100%" /> |

**¿No desea compilar una aplicación?** Use el [inicio rápido de Microsoft Graph](https://graph.microsoft.io/en-us/getting-started) para ponerlo todo en funcionamiento de manera rápida o descargue el [Ejemplo Connect de Microsoft Graph para Xamarin Forms](https://github.com/microsoftgraph/xamarin-csharp-connect-sample) en el que se basa este artículo.

## <a name="prerequisites"></a>Requisitos previos

Para comenzar, necesitará: 

- Una [cuenta Microsoft](https://www.outlook.com/) o una [cuenta profesional o educativa](http://dev.office.com/devprogram)
- Visual Studio 2015 
- [Xamarin para Visual Studio](https://www.xamarin.com/visual-studio)
- Windows 10 ([modo de desarrollo habilitado](https://msdn.microsoft.com/library/windows/apps/xaml/dn706236.aspx))
- [Proyecto inicial de conexión de Microsoft Graph para Xamarin Forms](https://github.com/microsoftgraph/xamarin-csharp-connect-sample/tree/master/starter). Esta plantilla contiene varias clases a las que agregará código. También contiene vistas completas y cadenas de recursos. Para obtener este proyecto, clone o descargue el [Ejemplo Connect de Microsoft Graph para Xamarin Forms](https://github.com/microsoftgraph/xamarin-csharp-connect-sample) y abra la solución **XamarinConnect** de la carpeta **starter**. 

Si desea ejecutar el proyecto de iOS en este ejemplo, necesitará lo siguiente:

- El SDK de iOS más reciente
- La versión de Xcode más reciente
- Mac OS X Yosemite (10.10) o superior 
- [Xamarin.iOS](https://developer.xamarin.com/guides/ios/getting_started/installation/mac/)
- Un [agente Xamarin Mac conectado a Visual Studio](https://developer.xamarin.com/guides/ios/getting_started/installation/windows/connecting-to-mac/)


## <a name="register-the-app"></a>Registrar la aplicación
 
1. Inicie sesión en el [Portal de registro de aplicaciones](https://apps.dev.microsoft.com/) mediante su cuenta personal, profesional o educativa.
2. Seleccione **Agregar una aplicación**.
3. Escriba un nombre para la aplicación y seleccione **Crear aplicación**.
    
    Se muestra la página de registro, indicando las propiedades de la aplicación.
 
4. En **Plataformas**, seleccione **Agregar plataforma**.
5. Seleccione **Plataforma móvil**.
6. Copie el identificador de la aplicación. Deberá escribir estos valores en la aplicación de ejemplo.

    El identificador de la aplicación es un identificador único para su aplicación. El URI de redireccionamiento es un URI único que proporciona Windows 10 para cada aplicación con el fin de garantizar que los mensajes enviados a ese URI solo se envían a esa aplicación. 

7. Seleccione **Guardar**.

## <a name="configure-the-project"></a>Configurar el proyecto

1. Abra el archivo de la solución para el proyecto inicial en Visual Studio.
2. Abra el archivo **App.cs** del proyecto **XamarinConnect (Portable)** y localice el campo `ClientId`. Reemplace el marcador de posición identificador de la aplicación por el identificador de la aplicación que haya registrado.

```c#
public static string ClientID = "ENTER_YOUR_CLIENT_ID";
public static string[] Scopes = { "User.Read", "Mail.Send" };
```
El valor `Scopes` almacena los ámbitos de permisos de Microsoft Graph que la aplicación deberá solicitar cuando el usuario se autentique. Tenga en cuenta que el constructor de clases `App` usa el valor ClientID para crear una instancia de la clase `PublicClientApplication` de MSAL. Más adelante, usará esta clase para autenticar el usuario.

```c#
IdentityClientApp = new PublicClientApplication(ClientID);
```

## <a name="install-the-microsoft-authentication-library-(msal)"></a>Instalar la Biblioteca de autenticación de Microsoft (MSAL)

La [Biblioteca de autenticación de Microsoft](https://www.nuget.org/packages/Microsoft.Identity.Client) contiene clases y métodos que facilitan la autenticación de los usuarios a través del punto de conexión de autenticación v2.0.

1. En el Explorador de soluciones, haga clic con el botón derecho en el proyecto **XamarinConnect (Portable)** y seleccione **Administrar paquetes de NuGet...**
2. Haga clic en Examinar y busque Microsoft.Identity.Client.
3. Seleccione la versión más reciente de la Biblioteca de autenticación de Microsoft y haga clic en **Instalar**.

Realice estos mismos pasos para los proyectos **XamarinConnect.Droid**, **XamarinConnect.iOS** y **XamarinConnect.UWP**. La aplicación no se compilará si no se ha instalado MSAL en los cuatro proyectos.

## <a name="install-the-microsoft-graph-client-library"></a>Instalar la biblioteca cliente de Microsoft Graph

1. En el Explorador de soluciones, haga clic con el botón derecho en el proyecto **XamarinConnect (Portable)** y seleccione **Administrar paquetes de NuGet...**
2. Haga clic en Examinar y busque Microsoft.Graph.
3. Seleccione la versión más reciente de la Biblioteca cliente de Microsoft y haga clic en **Instalar**.

## <a name="create-the-authenticationhelper.cs-class"></a>Crear la clase AuthenticationHelper.cs

Abra el archivo AuthenticationHelper.cs del proyecto **XamarinConnect (Portable)**. El archivo contiene el código de autenticación completo, junto con la lógica adicional que almacena la información del usuario, y exige la autenticación solo cuando el usuario se ha desconectado de la aplicación. Esta clase contiene al menos tres métodos: `GetTokenForUserAsync`, `Signout` y `GetAuthenticatedClient`.

El método `GetTokenHelperAsync` se ejecuta cuando el usuario se autentica y, posteriormente, cada vez que la aplicación llama a Microsoft Graph.

**Usar declaraciones**

Asegúrese de que tiene esas declaraciones al principio del archivo:

```c#
using Microsoft.Graph;
using System;
using System.Diagnostics;
using System.Net.Http.Headers;
using System.Threading.Tasks;
using Microsoft.Identity.Client;
```

**Campos de clase**

Asegúrese de que tiene esos campos en la clase AuthenticationHelper:

```c#
public static string TokenForUser = null;
public static DateTimeOffset expiration;
private static GraphServiceClient graphClient = null;
```

En el ejemplo, se almacena el `GraphServicesClient` en un campo de modo que solo deba crearlo una sola vez. Almacena la `DateTimeOffset` de expiración del token de acceso para no recuperar un nuevo token hasta que el existente esté a punto de expirar.

**GetTokenForUserAsync**

El método `GetTokenForUserAsync` usa la `PublicClientApplicationClass` con instancias en el archivo **App.cs** para obtener un token de acceso para el usuario. Si el usuario todavía no se ha autenticado, inicia la interfaz de usuario de autenticación.

```c#
        public static async Task<string> GetTokenForUserAsync()
        {
            if (TokenForUser == null || expiration <= DateTimeOffset.UtcNow.AddMinutes(5))
            {
                AuthenticationResult authResult = await App.IdentityClientApp.AcquireTokenAsync(App.Scopes);

                TokenForUser = authResult.Token;
                expiration = authResult.ExpiresOn;
            }

            return TokenForUser;
        }
```

**Cerrar sesión**

El método `Signout` cierra la sesión de todos los usuarios que la hayan iniciado a través de la `PublicClientApplication` (en este caso, solo hay un usuario) y anula el valor `TokenForUser`. También anulan el valor `GraphServicesClient`.

```c#
        public static void SignOut()
        {
            foreach (var user in App.IdentityClientApp.Users)
            {
                user.SignOut();
            }
            graphClient = null;
            TokenForUser = null;

        }
``` 

**GetAuthenticatedClient**

Por último, necesitará un método que cree un `GraphServicesClient`. Este método crea un cliente que usa el método `GetTokenForUserAsync` para cada llamada a Microsoft Graph que realiza a través del cliente.

```c#
        public static GraphServiceClient GetAuthenticatedClient()
        {
            if (graphClient == null)
            {
                // Create Microsoft Graph client.
                try
                {
                    graphClient = new GraphServiceClient(
                        "https://graph.microsoft.com/v1.0",
                        new DelegateAuthenticationProvider(
                            async (requestMessage) =>
                            {
                                var token = await GetTokenForUserAsync();
                                requestMessage.Headers.Authorization = new AuthenticationHeaderValue("bearer", token);
                            }));
                    return graphClient;
                }

                catch (Exception ex)
                {
                    Debug.WriteLine("Could not create a graph client: " + ex.Message);
                }
            }

            return graphClient;
        }
```

## <a name="send-an-email-with-microsoft-graph"></a>Enviar un correo electrónico con Microsoft Graph

Abra el archivo MailHelper.cs en su proyecto inicial. Este archivo contiene el código que crea y envía correos electrónicos. Consta de un único método (``ComposeAndSendMailAsync``) que crea y envía una solicitud POST al punto de conexión **https://graph.microsoft.com/v1.0/me/microsoft.graph.SendMail**. 

El método ``ComposeAndSendMailAsync`` toma tres valores de cadena (``subject``, ``bodyContent`` y ``recipients``) que se le pasan mediante el archivo MainPage.xaml.cs. Las cadenas ``subject`` y ``bodyContent`` se almacenan, junto con todas las demás cadenas de la interfaz de usuario, en el archivo AppResources.resx. La cadena ``recipients`` proviene del cuadro de dirección en la interfaz de la aplicación. 

**Usar declaraciones**

Agregue esas declaraciones al principio del archivo:

```c#
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.Graph;
```

Ya que el usuario puede pasar potencialmente más de una dirección, la primera tarea es dividir la cadena ``recipients`` en un conjunto de objetos `EmailAddress` que pueden usarse a continuación para crear la lista de objetos `Recipients` que se pasará en el cuerpo POST de la solicitud:

```c#
            // Prepare the recipient list
            string[] splitter = { ";" };
            var splitRecipientsString = recipients.Split(splitter, StringSplitOptions.RemoveEmptyEntries);
            List<Recipient> recipientList = new List<Recipient>();

            foreach (string recipient in splitRecipientsString)
            {
                recipientList.Add(new Recipient { EmailAddress = new EmailAddress { Address = recipient.Trim() } });
            }
```

La segunda tarea es crear un objeto `Message` y enviarlo al punto de conexión **me/microsoft.graph.SendMail** a través del `GraphServiceClient`. Ya que la cadena ``bodyContent`` es un documento HTML, la solicitud establece el valor **ContentType** a HTML.

```c#
            try
            {
                var graphClient = AuthenticationHelper.GetAuthenticatedClient();

                var email = new Message
                {
                    Body = new ItemBody
                    {
                        Content = bodyContent,
                        ContentType = BodyType.Html,
                    },
                    Subject = subject,
                    ToRecipients = recipientList,
                };

                try
                {
                    await graphClient.Me.SendMail(email, true).Request().PostAsync();
                }
                catch (ServiceException exception)
                {
                    throw new Exception("We could not send the message: " + exception.Error == null ? "No error message returned." : exception.Error.Message);
                }


            }

            catch (Exception e)
            {
                throw new Exception("We could not send the message: " + e.Message);
            }
```

La clase completa tendrá este aspecto:

```c#
    public class MailHelper
    {
        /// <summary>
        /// Compose and send a new email.
        /// </summary>
        /// <param name="subject">The subject line of the email.</param>
        /// <param name="bodyContent">The body of the email.</param>
        /// <param name="recipients">A semicolon-separated list of email addresses.</param>
        /// <returns></returns>
        public async Task ComposeAndSendMailAsync(string subject,
                                                            string bodyContent,
                                                            string recipients)
        {

            // Prepare the recipient list
            string[] splitter = { ";" };
            var splitRecipientsString = recipients.Split(splitter, StringSplitOptions.RemoveEmptyEntries);
            List<Recipient> recipientList = new List<Recipient>();

            foreach (string recipient in splitRecipientsString)
            {
                recipientList.Add(new Recipient { EmailAddress = new EmailAddress { Address = recipient.Trim() } });
            }

            try
            {
                var graphClient = AuthenticationHelper.GetAuthenticatedClient();

                var email = new Message
                {
                    Body = new ItemBody
                    {
                        Content = bodyContent,
                        ContentType = BodyType.Html,
                    },
                    Subject = subject,
                    ToRecipients = recipientList,
                };

                try
                {
                    await graphClient.Me.SendMail(email, true).Request().PostAsync();
                }
                catch (ServiceException exception)
                {
                    throw new Exception("We could not send the message: " + exception.Error == null ? "No error message returned." : exception.Error.Message);
                }


            }

            catch (Exception e)
            {
                throw new Exception("We could not send the message: " + e.Message);
            }
        }
    }
``` 

Ya ha llevado a cabo los tres pasos requeridos para interactuar con Microsoft Graph: el registro de la aplicación, la autenticación del usuario y la creación de una solicitud. 

## <a name="run-the-app"></a>Ejecutar la aplicación
1. Seleccione el proyecto que desee ejecutar. Si selecciona la opción de plataforma universal de Windows, puede ejecutar el ejemplo en el equipo local. Si desea ejecutar el proyecto iOS, necesitará conectarse a un [Mac que tenga las herramientas Xamarin](https://developer.xamarin.com/guides/ios/getting_started/installation/windows/connecting-to-mac/) instaladas. (También puede abrir esta solución en Xamarin Studio en un Mac y ejecutar el ejemplo directamente desde allí). Puede usar el [Emulador de Visual Studio para Android](https://www.visualstudio.com/features/msft-android-emulator-vs.aspx) si desea ejecutar el proyecto de Android. 

    ![](images/SelectProject.png "Select project in Visual Studio")

2. Pulse F5 para compilar y depurar. Ejecute la solución e inicie sesión con su cuenta personal, profesional o educativa.
    > **Nota** Es posible que tenga que abrir el administrador de configuración de compilación para asegurarse de que los pasos de compilación e implementación están seleccionados para el proyecto UWP. 

3. Inicie sesión con su cuenta personal, profesional o educativa y conceda los permisos solicitados.

4. Elija el botón **Enviar correo**. Cuando se envía el correo, se muestra un mensaje de Operación correcta.

## <a name="next-steps"></a>Pasos siguientes
- Pruebe la API de REST mediante el [Probador de Graph](https://graph.microsoft.io/graph-explorer).
- Busque ejemplos de operaciones comunes en la [Biblioteca de fragmentos de código del SDK de Microsoft Graph para Xamarin.Forms](https://github.com/microsoftgraph/xamarin-csharp-snippets-sample) o explore el resto de nuestros [ejemplos de Xamarin](https://github.com/microsoftgraph?utf8=%E2%9C%93&query=xamarin) en GitHub.

## <a name="see-also"></a>Recursos adicionales
- [Biblioteca cliente .NET de Microsoft Graph](https://github.com/microsoftgraph/msgraph-sdk-dotnet)
- [Protocolos de Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
- [Tokens de Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)
