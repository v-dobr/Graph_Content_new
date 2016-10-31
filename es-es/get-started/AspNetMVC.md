# <a name="get-started-with-microsoft-graph-in-an-asp.net-4.6-mvc-app"></a>Introducción a Microsoft Graph en una aplicación de ASP.NET 4.6 MVC

En este artículo, se describen las tareas necesarias para obtener un token de acceso desde el punto de conexión v2.0 de Azure AD y llamar a Microsoft Graph. Le muestra los pasos para la compilación del [Ejemplo Connect de Microsoft Graph para ASP.NET 4.6](https://github.com/microsoftgraph/aspnet-connect-sample) y explica los conceptos principales que implementará para usar Microsoft Graph. En el artículo, también se describe cómo obtener acceso a Microsoft Graph usando la [Biblioteca cliente .NET de Microsoft Graph](https://github.com/microsoftgraph/msgraph-sdk-dotnet) o llamadas de REST sin procesar.

En la imagen siguiente, se muestra la aplicación que va a crear. 

![La aplicación web con los botones "Obtener la dirección de correo electrónico" y "Enviar correo electrónico"](images/aspnet-connect-sample.png "La aplicación web con los botones "Obtener la dirección de correo electrónico" y "Enviar correo electrónico"")

El [punto de conexión v2.0 de Azure AD](https://azure.microsoft.com/en-us/documentation/articles/active-directory-appmodel-v2-overview) permite a los usuarios iniciar sesión con una cuenta Microsoft (MSA) o con una cuenta profesional o educativa. La aplicación usa el [middleware de OWIN de OpenID Connect ASP.Net](https://www.nuget.org/packages/Microsoft.Owin.Security.OpenIdConnect/) y la [Biblioteca de autenticación de Microsoft (MSAL) para .NET](https://www.nuget.org/packages/Microsoft.Identity.Client) para el inicio de sesión y la administración de tokens.

>La Biblioteca de autenticación de Microsoft está actualmente en versión preliminar y, por tanto, no debe usarse en código de producción. Aquí se usa únicamente con fines ilustrativos. 

**¿No desea compilar una aplicación?** Use el [inicio rápido de Microsoft Graph](https://graph.microsoft.io/en-us/getting-started) para ponerlo todo en funcionamiento de manera rápida.

Para descargar una versión del ejemplo de Connect que usa el punto de conexión de Azure AD y que obtiene acceso a Microsoft Graph mediante llamadas de REST sin procesar, consulte [Ejemplo de Connect de Office 365 para ASP.NET MVC](https://github.com/microsoftgraph/aspnet-connect-rest-sample).


## <a name="prerequisites"></a>Requisitos previos

Para comenzar, necesitará: 

- Una [cuenta Microsoft](https://www.outlook.com/) o una [cuenta profesional o educativa](http://dev.office.com/devprogram)
- Visual Studio 2015 
- [Ejemplo de Connect de Microsoft Graph para ASP.NET 4.6](https://github.com/microsoftgraph/aspnet-connect-sample). Usará la carpeta **starter-project** en los archivos de ejemplo.


## <a name="register-the-application"></a>Registrar la aplicación

En este paso, registrará una aplicación en el Portal de registro de aplicaciones de Microsoft. Esta acción generará el identificador de la aplicación y la contraseña que usará para configurar la aplicación en Visual Studio.

1. Inicie sesión en el [Portal de registro de aplicaciones de Microsoft](https://apps.dev.microsoft.com/) mediante su cuenta personal, profesional o educativa.

2. Seleccione **Agregar una aplicación**.

3. Escriba un nombre para la aplicación y seleccione **Crear aplicación**. 
    
    Se muestra la página de registro, indicando las propiedades de la aplicación.

4. Copie el identificador de la aplicación. Se trata del identificador único para su aplicación. 

5. En **Secretos de aplicación**, elija **Generar nueva contraseña**. Copie la contraseña del cuadro de diálogo **Nueva contraseña generada**.

    Deberá usar el Id. y el secreto de aplicación para configurar la aplicación. 

6. En **Plataformas**, elija **Agregar plataforma** > **Web**.

7. Asegúrese de que la casilla **Permitir flujo implícito** esté seleccionada y escriba *http://localhost:55065/* como URI de redireccionamiento. 

    La opción **Permitir flujo implícito**habilita el flujo híbrido de OpenID Connect. Durante la autenticación, esto permite que la aplicación reciba tanto la información de inicio de sesión (el **id_token**) como los artefactos (en este caso, un código de autorización) que la aplicación usa para obtener un token de acceso.

8. Elija **Guardar**.

### <a name="configure-the-project"></a>Configurar el proyecto

1. Abra el archivo de la solución para el proyecto inicial en Visual Studio.

2. Abra el archivo Web.config del proyecto.

3. Busque las claves de configuración de la aplicación en el elemento **appSettings**. Reemplace los valores de los marcadores de posición ENTER_YOUR_CLIENT_ID y ENTER_YOUR_SECRET por los valores que acaba de copiar.

El URI de redireccionamiento es la URL del proyecto que ha registrado. Los [ámbitos de permisos](https://graph.microsoft.io/en-us/docs/authorization/permission_scopes) solicitados permiten a la aplicación obtener la información del perfil de usuario y enviar un correo electrónico.


## <a name="authenticate-the-user-and-get-an-access-token"></a>Autenticar al usuario y obtener un token de acceso

En este paso, agregará el código de administración del token y de inicio de sesión. Pero, primero, echemos un vistazo más de cerca al flujo de autenticación.

Esta aplicación usa el flujo de concesión de códigos de autorización con una identidad de usuario delegado. Para una aplicación web, el flujo requiere el Id. de aplicación, la contraseña y el URI de redireccionamiento de la aplicación registrada. 

El flujo de autenticación puede desglosarse en los siguientes pasos básicos:

1. Redirigir al usuario para la autenticación y el consentimiento
2. Obtener un código de autorización
3. Canjear el código de autorización por un token de acceso
4. Use el token de actualización para obtener un token de acceso nuevo cuando expire el actual.

La aplicación usa el [middleware de OWIN de OpenID Connect ASP.Net](https://www.nuget.org/packages/Microsoft.Owin.Security.OpenIdConnect/) y la [Biblioteca de autenticación de Microsoft (MSAL) para .NET](https://www.nuget.org/packages/Microsoft.Identity.Client) para el inicio de sesión y la administración de tokens. Estos elementos controlan la mayoría de las tareas de autenticación en su lugar.
    
El proyecto inicial ya declara el siguiente middleware y las dependencias de NuGet de la Biblioteca de autenticación de Microsoft:

  - Microsoft.Owin.Security.OpenIdConnect
  - Microsoft.Owin.Security.Cookies
  - Microsoft.Owin.Host.SystemWeb
  - Microsoft.Identity.Client -Pre

Ahora, volvamos a la compilación de la aplicación.

1. En la carpeta **App_Start** carpeta, abra el archivo Startup.Auth.cs. 

1. Reemplace el método **ConfigureAuth** por el siguiente código. Esta acción configura las coordenadas para la comunicación con Azure AD y configura la autenticación con cookies que usa el middleware de OpenID Connect.

        public void ConfigureAuth(IAppBuilder app)
        {
            app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);

            app.UseCookieAuthentication(new CookieAuthenticationOptions());

            app.UseOpenIdConnectAuthentication(
                new OpenIdConnectAuthenticationOptions
                {

                    // The `Authority` represents the Microsoft v2.0 authentication and authorization service.
                    // The `Scope` describes the permissions that your app will need. See https://azure.microsoft.com/documentation/articles/active-directory-v2-scopes/                    
                    ClientId = appId,
                    Authority = "https://login.microsoftonline.com/common/v2.0",
                    PostLogoutRedirectUri = redirectUri,
                    RedirectUri = redirectUri,
                    Scope = "openid email profile offline_access " + graphScopes,
                    TokenValidationParameters = new TokenValidationParameters
                    {
                        ValidateIssuer = false,
                        // In a real application you would use IssuerValidator for additional checks, 
                        // like making sure the user's organization has signed up for your app.
                        //     IssuerValidator = (issuer, token, tvp) =>
                        //     {
                        //         if (MyCustomTenantValidation(issuer)) 
                        //             return issuer;
                        //         else
                        //             throw new SecurityTokenInvalidIssuerException("Invalid issuer");
                        //     },
                    },
                    Notifications = new OpenIdConnectAuthenticationNotifications
                    {
                        AuthorizationCodeReceived = async (context) =>
                        {
                            var code = context.Code;
                            string signedInUserID = context.AuthenticationTicket.Identity.FindFirst(ClaimTypes.NameIdentifier).Value;
                            ConfidentialClientApplication cca = new ConfidentialClientApplication(
                                appId, 
                                redirectUri,
                                new ClientCredential(appSecret),
                                new SessionTokenCache(signedInUserID, context.OwinContext.Environment["System.Web.HttpContextBase"] as HttpContextBase));
                                string[] scopes = graphScopes.Split(new char[] { ' ' });

                            AuthenticationResult result = await cca.AcquireTokenByAuthorizationCodeAsync(scopes, code);
                        },
                        AuthenticationFailed = (context) =>
                        {
                            context.HandleResponse();
                            context.Response.Redirect("/Error?message=" + context.Exception.Message);
                            return Task.FromResult(0);
                        }
                    }
                });
        }
  
  La clase de inicio de OWIN (definida en Startup.cs) invoca al método **ConfigureAuth** cuando se inicia la aplicación, que a su vez llama a **app.UseOpenIdConnectAuthentication** para inicializar el middleware para el inicio de sesión y la solicitud de token inicial. La aplicación solicita los siguientes ámbitos de permisos:

  - **openid**, **email**, **profile** para el inicio de sesión
  - **offline\_access** (se requiere para obtener los tokens de actualización), **User.Read** y **Mail.Send** para la adquisición de tokens
  
  El objeto **ConfidentialClientApplication** de la Biblioteca de autenticación de Microsoft representa la aplicación y controla las tareas de administración de tokens. Se inicializa con **SessionTokenCache** (la implementación de caché de tokens de ejemplo que se define en TokenStorage/SessionTokenCache.cs), donde se almacena la información de token. La caché guarda los tokens en la sesión HTTP actual en función del Id. de usuario, pero una aplicación de producción probablemente usará un almacenamiento más persistente.

Ahora, deberá agregar el código al proveedor de autenticación de ejemplo, que está diseñado para reemplazarse fácilmente por su propio proveedor de autenticación personalizado. La clase de interfaz y la de proveedor ya se han agregado al proyecto.

1. En la carpeta **Helpers**, abra el archivo SampleAuthProvider.cs.

1. Reemplace el método **GetUserAccessTokenAsync** por la siguiente implementación que usa la Biblioteca de autenticación de Microsoft para obtener un token de acceso.

        // Get an access token. First tries to get the token from the token cache.
        public async Task<string> GetUserAccessTokenAsync()
        {
            string signedInUserID = ClaimsPrincipal.Current.FindFirst(ClaimTypes.NameIdentifier).Value;
            tokenCache = new SessionTokenCache(
                signedInUserID, 
                HttpContext.Current.GetOwinContext().Environment["System.Web.HttpContextBase"] as HttpContextBase);
            //var cachedItems = tokenCache.ReadItems(appId); // see what's in the cache

            ConfidentialClientApplication cca = new ConfidentialClientApplication(
                appId, 
                redirectUri,
                new ClientCredential(appSecret), 
                tokenCache);

            try
            {
                AuthenticationResult result = await cca.AcquireTokenSilentAsync(scopes.Split(new char[] { ' ' }));
                return result.Token;
            }

            // Unable to retrieve the access token silently.
            catch (MsalSilentTokenAcquisitionException)
            {
                HttpContext.Current.Request.GetOwinContext().Authentication.Challenge(
                    new AuthenticationProperties() { RedirectUri = "/" },
                    OpenIdConnectAuthenticationDefaults.AuthenticationType);

                throw new Exception(Resource.Error_AuthChallengeNeeded);
            }
        }

  La Biblioteca de autenticación de Microsoft comprueba la caché de un token de acceso coincidente que no haya expirado o esté a punto de expirar. Si no encuentra ninguno válido, usa el token de actualización (si existe uno válido) para obtener un nuevo token de acceso. Si no puede obtener un nuevo token de acceso de forma automática, la Biblioteca de autenticación de Microsoft iniciará la excepción **MsalSilentTokenAcquisitionException** para indicar que se necesita un mensaje de usuario. 

A continuación, deberá agregar el código para controlar el inicio y el cierre de sesión desde la interfaz de usuario.

1. En la carpeta **Controllers**, abra el archivo AccountController.cs.  

1. Agregue los siguientes métodos a la clase **AccountController**. El método **SignIn** envía una señal al middleware para que envíe una solicitud de autorización a Azure AD.

        public void SignIn()
        {
            if (!Request.IsAuthenticated)
            {
                // Signal OWIN to send an authorization request to Azure.
                HttpContext.GetOwinContext().Authentication.Challenge(
                    new AuthenticationProperties { RedirectUri = "/" },
                    OpenIdConnectAuthenticationDefaults.AuthenticationType);
            }
        }

        // Here we just clear the token cache, sign out the GraphServiceClient, and end the session with the web app.  
        public void SignOut()
        {
            if (Request.IsAuthenticated)
            {
                // Get the user's token cache and clear it.
                string userObjectId = ClaimsPrincipal.Current.FindFirst(ClaimTypes.NameIdentifier).Value;

                SessionTokenCache tokenCache = new SessionTokenCache(userObjectId, HttpContext);
                tokenCache.Clear(userObjectId);
            }

            //SDKHelper.SignOutClient();

            // Send an OpenID Connect sign-out request. 
            HttpContext.GetOwinContext().Authentication.SignOut(
            CookieAuthenticationDefaults.AuthenticationType);
            Response.Redirect("/");
        }

Ahora, ya puede agregar el código para llamar a Microsoft Graph. 

## <a name="call-microsoft-graph"></a>Llamar a Microsoft Graph

Si usa la biblioteca de Microsoft Graph, siga leyendo. Si usa REST, consulte la sección [Usar la API de REST](#using-the-rest-api).

### <a name="using-the-library"></a>Uso de la biblioteca
En este paso, deberá centrarse en las clases **SDKHelper**, **GraphService** y **HomeController**. 

 - **SDKHelper** inicializa una instancia de **GraphServiceClient** desde la biblioteca antes de cada llamada a Microsoft Graph. Este es el momento en el que el token de acceso se agrega a la solicitud. 
 - **GraphService** compila las solicitudes HTTP y las envía a Microsoft Graph mediante la biblioteca y procesa las respuestas.
 - **HomeController** contiene las acciones que inician las llamadas a la biblioteca en respuesta a los eventos de interfaz de usuario.

El proyecto inicial ya declara una dependencia para el paquete de NuGet de la Biblioteca cliente .NET de Microsoft Graph:  *Microsoft.Graph*.

1. Haga clic con el botón derecho en la carpeta **Helpers** y elija **Agregar** > **Clase**. 

1. Asigne a la nueva clase el nombre *SDKHelper* y elija **Agregar**.

1. Reemplace el contenido por el siguiente código.

        using System.Net.Http.Headers;
        using Microsoft.Graph;

        namespace Microsoft_Graph_SDK_ASPNET_Connect.Helpers
        {
            public class SDKHelper
            {   
                private static GraphServiceClient graphClient = null;

                // Get an authenticated Microsoft Graph Service client.
                public static GraphServiceClient GetAuthenticatedClient()
                {
                    GraphServiceClient graphClient = new GraphServiceClient(
                        new DelegateAuthenticationProvider(
                            async (requestMessage) =>
                            {
                                string accessToken = await SampleAuthProvider.Instance.GetUserAccessTokenAsync();

                                // Append the access token to the request.
                                requestMessage.Headers.Authorization = new AuthenticationHeaderValue("bearer", accessToken);
                            }));
                    return graphClient;
                }

                public static void SignOutClient()
                {
                    graphClient = null;
                }
            }
        }

  Fíjese en la llamada a **SampleAuthProvider** para obtener el token cuando se inicialice el cliente.

1. En la carpeta **Modelos** abra el archivo GraphService.cs. El servicio usa la biblioteca para interactuar con Microsoft Graph.

1. Agregue la siguiente instrucción **Using**.

        using Microsoft.Graph;

1. Reemplace *// GetMyEmailAddress* por el siguiente método. Esta acción obtiene la dirección de correo electrónico del usuario actual. 

        // Get the current user's email address from their profile.
        public async Task<string> GetMyEmailAddress(GraphServiceClient graphClient)
        {

            // Get the current user. 
            // The app only needs the user's email address, so select the mail and userPrincipalName properties.
            // If the mail property isn't defined, userPrincipalName should map to the email for all account types. 
            User me = await graphClient.Me.Request().Select("mail,userPrincipalName").GetAsync();
            return me.Mail ?? me.UserPrincipalName;
        }

  Fíjese en el segmento **Select**, que solo solicita que se devuelvan los elementos **mail** y **userPrinicipalName**. Puede usar los segmentos **Select** y **Filter** para reducir el tamaño de la carga de datos de respuesta.

1. Reemplace *// SendEmail* por los siguientes métodos para crear y enviar el correo electrónico.

        // Send an email message from the current user.
        public async Task SendEmail(GraphServiceClient graphClient, Message message)
        {
            await graphClient.Me.SendMail(message, true).Request().PostAsync();
        }

        // Create the email message.
        public Message BuildEmailMessage(string recipients, string subject)
        {

            // Prepare the recipient list.
            string[] splitter = { ";" };
            string[] splitRecipientsString = recipients.Split(splitter, StringSplitOptions.RemoveEmptyEntries);
            List<Recipient> recipientList = new List<Recipient>();
            foreach (string recipient in splitRecipientsString)
            {
                recipientList.Add(new Recipient
                {
                    EmailAddress = new EmailAddress
                    {
                        Address = recipient.Trim()
                    }
                });
            }

            // Build the email message.
            Message email = new Message
            {
                Body = new ItemBody
                {
                    Content = Resource.Graph_SendMail_Body_Content,
                    ContentType = BodyType.Html,
                },
                Subject = subject,
                ToRecipients = recipientList
            };
            return email;
        }

1. En la carpeta **Controllers**, abra el archivo HomeController.cs.

1. Agregue la siguiente instrucción **Using**.

        using Microsoft.Graph;
  
1. Reemplace *// Controller actions* por las siguientes acciones.

        [Authorize]
        // Get the current user's email address from their profile.
        public async Task<ActionResult> GetMyEmailAddress()
        {
            try
            {

                // Initialize the GraphServiceClient.
                GraphServiceClient graphClient = SDKHelper.GetAuthenticatedClient();

                // Get the current user's email address. 
                ViewBag.Email = await graphService.GetMyEmailAddress(graphClient);
                return View("Graph");
            }
            catch (ServiceException se)
            {
                if (se.Error.Message == Resource.Error_AuthChallengeNeeded) return new EmptyResult();
                return RedirectToAction("Index", "Error", new { message = Resource.Error_Message + Request.RawUrl + ": " + se.Error.Message });
            }
        }

        [Authorize]
        // Send mail on behalf of the current user.
        public async Task<ActionResult> SendEmail()
        {
            if (string.IsNullOrEmpty(Request.Form["email-address"]))
            {
                ViewBag.Message = Resource.Graph_SendMail_Message_GetEmailFirst;
                return View("Graph");
            }

            // Build the email message.
            Message message = graphService.BuildEmailMessage(Request.Form["recipients"], Request.Form["subject"]);
            try
            {

                // Initialize the GraphServiceClient.
                GraphServiceClient graphClient = SDKHelper.GetAuthenticatedClient();

                // Send the email.
                await graphService.SendEmail(graphClient, message);

                // Reset the current user's email address and the status to display when the page reloads.
                ViewBag.Email = Request.Form["email-address"];
                ViewBag.Message = Resource.Graph_SendMail_Success_Result;
                return View("Graph");
            }
            catch (ServiceException se)
            {
                if (se.Error.Message == Resource.Error_AuthChallengeNeeded) return new EmptyResult();
                return RedirectToAction("Index", "Error", new { message = Resource.Error_Message + Request.RawUrl + ": " + se.Error.Message });
           }
        }

A continuación, deberá cambiar la excepción que inicia el proveedor de autenticación cuando se necesita un mensaje de usuario.

1. En la carpeta **Helpers**, abra el archivo SampleAuthProvider.cs.

1. Agregue la siguiente instrucción **Using**.

        using Microsoft.Graph;
  
1. En el bloque **catch** del método **GetUserAccessTokenAsync**, cambie la excepción que se ha iniciado como se indica a continuación:

        throw new ServiceException(
            new Error
            {
                Code = GraphErrorCode.AuthenticationFailure.ToString(),
                Message = Resource.Error_AuthChallengeNeeded,
            });

Por último, agregará una llamada a cerrar la sesión del cliente. 

1. En la carpeta **Controllers**, abra el archivo AccountController.cs. 

1. Quite la marca de comentario de la siguiente línea:

        SDKHelper.SignOutClient();

Ahora, ya puede [ejecutar la aplicación](#run-the-app).

### <a name="using-the-rest-api"></a>Uso de la API de REST
En este paso, trabajará con las clases **GraphService**, **GraphResources** y **HomeController**. 

 - **GraphService** compila las solicitudes HTTP y las envía a Microsoft Graph y procesa las respuestas. 
 - **GraphResources** representa los datos que la aplicación envía a Microsoft Graph y los que recibe de esta herramienta. 
 - **HomeController** contiene las acciones que inician las llamadas a Microsoft Graph en respuesta a los eventos de interfaz de usuario.

Empiece por definir el nivel de acceso a los datos.

1. En la carpeta **Modelos** abra el archivo GraphService.cs.

1. Agregue las siguientes instrucciones **Using**.

        using Newtonsoft.Json;
        using Newtonsoft.Json.Linq;
        using System.Net;
        using System.Net.Http;
        using System.Net.Http.Headers;
        using System.Text;

1. Reemplace *// GetMyEmailAddress* por el siguiente método. Esta acción obtiene la dirección de correo electrónico del usuario actual. 

        // Get the current user's email address from their profile.
        public async Task<string> GetMyEmailAddress(string accessToken)
        {

            // Get the current user. 
            // The app only needs the user's email address, so select the mail and userPrincipalName properties.
            // If the mail property isn't defined, userPrincipalName should map to the email for all account types. 
            string endpoint = "https://graph.microsoft.com/v1.0/me";
            string queryParameter = "?$select=mail,userPrincipalName";
            UserInfo me = new UserInfo();

            using (var client = new HttpClient())
            {
                using (var request = new HttpRequestMessage(HttpMethod.Get, endpoint + queryParameter))
                {
                    request.Headers.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
                    request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);
                    using (HttpResponseMessage response = await client.SendAsync(request))
                    {
                        if (response.StatusCode == HttpStatusCode.OK)
                        {
                            var json = JObject.Parse(await response.Content.ReadAsStringAsync());
                            me.Address = !string.IsNullOrEmpty(json.GetValue("mail").ToString())?json.GetValue("mail").ToString():json.GetValue("userPrincipalName").ToString();
                        }
                        return me.Address?.Trim();
                    }
                }
            }
        }

1. Reemplace *// SendEmail* por los siguientes métodos para crear y enviar el correo electrónico.

        // Send an email message from the current user.
        public async Task<string> SendEmail(string accessToken, MessageRequest email)
        {
            string endpoint = "https://graph.microsoft.com/v1.0/me/sendMail";
            using (var client = new HttpClient())
            {
                using (var request = new HttpRequestMessage(HttpMethod.Post, endpoint))
                {
                    request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);
                    request.Content = new StringContent(JsonConvert.SerializeObject(email), Encoding.UTF8, "application/json");
                    using (HttpResponseMessage response = await client.SendAsync(request))
                    {
                        if (response.IsSuccessStatusCode)
                        {
                            return Resource.Graph_SendMail_Success_Result;
                        }
                        return response.ReasonPhrase;
                    }
                }
            }
        }

        // Create the email message.
        public MessageRequest BuildEmailMessage(string recipients, string subject)
        {

            // Prepare the recipient list.
            string[] splitter = { ";" };
            string[] splitRecipientsString = recipients.Split(splitter, StringSplitOptions.RemoveEmptyEntries);
            List<Recipient> recipientList = new List<Recipient>();
            foreach (string recipient in splitRecipientsString)
            {
                recipientList.Add(new Recipient
                {
                    EmailAddress = new UserInfo
                    {
                        Address = recipient.Trim()
                    }
                });
            }

            // Build the email message.
            Message message = new Message
            {
                Body = new ItemBody
                {
                    Content = Resource.Graph_SendMail_Body_Content,
                    ContentType = "HTML"
                },
                Subject = subject,
                ToRecipients = recipientList
            };

            return new MessageRequest
            {
                Message = message,
                SaveToSentItems = true
            };
        }

1. Haga clic con el botón derecho en la carpeta **Modelos** y elija **Agregar** > **Clase**.

1. Asigne un nombre a la clase *GraphResources* y elija **Aceptar**.

1. Reemplace el contenido por el siguiente código.
 
        using System;
        using System.Collections.Generic;

        namespace Microsoft_Graph_SDK_ASPNET_Connect.Models
        {
            public class UserInfo
            {
                public string Address { get; set; }
            }

            public class Message
            {
                public string Subject { get; set; }
                public ItemBody Body { get; set; }
                public List<Recipient> ToRecipients { get; set; }
            }

          public class Recipient
            {
                public UserInfo EmailAddress { get; set; }
            }

            public class ItemBody
            {
                public string ContentType { get; set; }
                public string Content { get; set; }
            }

            public class MessageRequest
            {
                public Message Message { get; set; }
                public bool SaveToSentItems { get; set; }
            }
        }

1. En la carpeta **Controllers**, abra el archivo HomeController.cs.
  
1. Reemplace *// Controller actions* por las siguientes acciones.

        [Authorize]
        // Get the current user's email address from their profile.
        public async Task<ActionResult> GetMyEmailAddress()
        {
            try
            {

                // Get an access token.
                string accessToken = await SampleAuthProvider.Instance.GetUserAccessTokenAsync();

                // Get the current user's email address. 
                ViewBag.Email = await graphService.GetMyEmailAddress(accessToken);
                return View("Graph");
            }
            catch (Exception e)
            {
                if (e.Message == Resource.Error_AuthChallengeNeeded) return new EmptyResult();
                return RedirectToAction("Index", "Error", new { message = Resource.Error_Message + Request.RawUrl + ": " + e.Message });
            }
        }

        [Authorize]
        // Send mail on behalf of the current user.
        public async Task<ActionResult> SendEmail()
        {
            if (string.IsNullOrEmpty(Request.Form["email-address"]))
            {
                ViewBag.Message = Resource.Graph_SendMail_Message_GetEmailFirst;
                return View("Graph");
            }

            // Build the email message.
            MessageRequest email = graphService.BuildEmailMessage(Request.Form["recipients"], Request.Form["subject"]);
            try
            {

                // Get an access token.
                string accessToken = await SampleAuthProvider.Instance.GetUserAccessTokenAsync();

                // Send the email.
                ViewBag.Message = await graphService.SendEmail(accessToken, email);

                // Reset the current user's email address and the status to display when the page reloads.
                ViewBag.Email = Request.Form["email-address"];
                return View("Graph");
            }
            catch (Exception e)
            {
                if (e.Message == Resource.Error_AuthChallengeNeeded) return new EmptyResult();
                return RedirectToAction("Index", "Error", new { message = Resource.Error_Message + Request.RawUrl + ": " + e.Message });
            }
        }

## <a name="run-the-app"></a>Ejecutar la aplicación
1. Pulse F5 para compilar y ejecutar la aplicación. 

2. Inicie sesión con su cuenta personal, profesional o educativa y conceda los permisos solicitados.

3. Seleccione el botón **Obtener la dirección de correo electrónico**. Cuando finaliza la operación, la dirección de correo electrónico del usuario con sesión iniciada se muestra en la página.

4. De forma opcional, modifique la lista de destinatarios y el asunto del correo electrónico y, después, seleccione el botón **Enviar correo electrónico**. Cuando se envía el correo, se muestra un mensaje de Operación correcta debajo del botón.


## <a name="next-steps"></a>Pasos siguientes
- Pruebe la API de REST mediante el [Probador de Graph](https://graph.microsoft.io/graph-explorer).
- Busque ejemplos de operaciones comunes en [Fragmentos de código de ejemplo para ASP.NET 4.6](https://github.com/microsoftgraph/aspnet-snippets-sample) o explore nuestros otros [ejemplos para ASP.NET](http://aka.ms/aspnetgraphsamples) en GitHub.

## <a name="see-also"></a>Recursos adicionales
- [Biblioteca cliente .NET de Microsoft Graph](https://github.com/microsoftgraph/msgraph-sdk-dotnet)
- [Aplicación web para el escenario de autenticación de la API web](https://azure.microsoft.com/en-us/documentation/articles/active-directory-authentication-scenarios/#web-application-to-web-api)
- [Integrar la identidad de Microsoft y Microsoft Graph en una aplicación web usando OpenID Connect](https://azure.microsoft.com/en-us/documentation/samples/active-directory-dotnet-webapp-openidconnect-v2/)
- [Protocolos de Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
- [Tokens de Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)
