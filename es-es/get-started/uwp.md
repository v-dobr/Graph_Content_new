# <a name="get-started-with-microsoft-graph-in-a-universal-windows-10-app"></a>Introducción a Microsoft Graph en una aplicación de Windows 10 universal

> **¿Desea compilar aplicaciones para clientes empresariales?** Es posible que la aplicación no funcione si su cliente empresarial activa características de seguridad de movilidad empresarial como el <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-conditional-access-device-policies/" target="_newtab">acceso condicional al dispositivo</a>. En casos así, es posible que no tenga constancia de esta activación y que sus clientes obtengan errores. 

> Para admitir **todos los clientes empresariales** en **todos los escenarios de empresa**, deberá usar el punto de conexión de Azure AD y administrar las aplicaciones mediante el [Portal de administración de Azure](https://aka.ms/aadapplist). Para obtener más información, consulte [Decidir entre los puntos de conexión de Azure AD y Azure AD v2.0 ](../authorization/auth_overview.md#deciding-between-azure-ad-and-the-v2-authentication-endpoint).

En este artículo se describen las tareas necesarias para obtener un token de acceso desde el punto de conexión de [Azure AD v2.0](https://graph.microsoft.io/en-us/docs/authorization/converged_auth) y llamar a Microsoft Graph. Le guiará por el código del [Ejemplo Connect de Microsoft Graph para UWP (REST)](https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample) y del [Ejemplo Connect de Microsoft Graph para UWP (biblioteca)](https://github.com/microsoftgraph/uwp-csharp-connect-sample) para explicar los conceptos principales que se deben implementar en una aplicación que use Microsoft Graph. En el artículo, también se describe cómo obtener acceso a Microsoft Graph mediante llamadas de REST sin procesar y la [Biblioteca cliente de Microsoft Graph](http://www.nuget.org/packages/Microsoft.Graph/).

Puede descargar ambas versiones de la aplicación que creará en este tutorial desde los siguientes repositorios de GitHub:

* [Ejemplo Connect de Microsoft Graph para UWP (REST)](https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample)
* [Ejemplo Connect de Microsoft Graph para UWP (biblioteca)](https://github.com/microsoftgraph/uwp-csharp-connect-sample)

**¿No desea compilar una aplicación?** Use el [inicio rápido de Microsoft Graph](https://graph.microsoft.io/en-us/getting-started) para ponerlo todo en funcionamiento de manera rápida.

## <a name="sample-user-interface"></a>Interfaz de usuario de ejemplo

El ejemplo contiene una interfaz de usuario muy sencilla, que consta de una barra de comandos superior, el **botón Conectar**, el botón **Enviar correo** y un cuadro de texto que se rellena automáticamente con la dirección de correo electrónico del usuario que ha iniciado sesión pero puede modificarse.

El botón **Enviar correo** está deshabilitado cuando el usuario no está conectado:

![Pantalla que muestra el botón Conectar habilitado y el botón Enviar correo deshabilitado](images/SignedOut.png)

En la barra de comandos superior, aparece un botón de desconexión cuando el usuario está conectado:

![Pantalla que muestra la dirección de correo electrónico del usuario conectado y el botón Enviar correo habilitado](images/SignedIn.png)

Todas las cadenas de la interfaz de usuario del ejemplo se almacenan en el archivo Resources.resw dentro de la carpeta Assets.

## <a name="prerequisites"></a>Requisitos previos

Para comenzar, necesitará: 

- Una [cuenta Microsoft](https://www.outlook.com/) o una [cuenta profesional o educativa](http://dev.office.com/devprogram)
- Visual Studio 2015 
- Ya sea el [Proyecto inicial de Microsoft Graph para UWP (biblioteca)](https://github.com/microsoftgraph/uwp-csharp-connect-sample/tree/master/starter) o el [Proyecto inicial de Microsoft Graph para UWP (REST)](https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample/tree/master/starter). Ambas plantillas contienen clases a las que agregará código. También contienen cadenas de recursos. Para obtener uno de estos proyectos o ambos, clone o descargue el [Ejemplo Connect de Microsoft Graph para UWP (biblioteca)](https://github.com/microsoftgraph/uwp-csharp-connect-sample) o el [Ejemplo Connect de Microsoft Graph para UWP (REST)](https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample). A continuación, abra la solución en la carpeta **starter**.


## <a name="register-the-app"></a>Registrar la aplicación
 
1. Inicie sesión en el [Portal de registro de aplicaciones](https://apps.dev.microsoft.com/) mediante su cuenta personal, profesional o educativa.
2. Seleccione **Agregar una aplicación**.
3. Escriba un nombre para la aplicación y seleccione **Crear aplicación**.
    
    Se muestra la página de registro, indicando las propiedades de la aplicación.
 
4. En **Plataformas**, seleccione **Agregar plataforma**.
5. Seleccione **Plataforma móvil**.
6. Copie tanto el identificador de cliente (identificador de la aplicación) como los valores del URI de redireccionamiento al Portapapeles. Deberá escribir estos valores en la aplicación de ejemplo.

    El identificador de la aplicación es un identificador único para su aplicación. El URI de redireccionamiento es un URI único que proporciona Windows 10 para cada aplicación con el fin de garantizar que los mensajes enviados a ese URI solo se envían a esa aplicación. 

7. Seleccione **Guardar**.

## <a name="configure-the-project"></a>Configurar el proyecto

1. Abra el archivo de la solución para el proyecto inicial en Visual Studio.
2. Abra el archivo **App.xaml** del proyecto y busque el nodo `Application.Resources`. Reemplace los marcadores de posición identificador de la aplicación y URI de redireccionamiento por los valores correspondientes de la aplicación que haya registrado.


```xml
    <Application.Resources>
        <!-- Add your Client Id here. -->
        <x:String x:Key="ida:ClientID">ENTER_YOUR_CLIENT_ID</x:String>
        <!-- Add your Redirect URI here. -->
        <x:String x:Key="ida:ReturnUrl">ENTER_YOUR_REDIRECT_URI</x:String>
    </Application.Resources>
```

## <a name="install-the-microsoft-authentication-library-(msal)"></a>Instalar la Biblioteca de autenticación de Microsoft (MSAL)

La [Biblioteca de autenticación de Microsoft](https://www.nuget.org/packages/Microsoft.Identity.Client) contiene clases y métodos que facilitan la autenticación de los usuarios a través del punto de conexión v2.0 de Azure AD.

1. En el Explorador de soluciones, haga clic con el botón derecho en el nombre del proyecto y seleccione **Administrar paquetes de NuGet...**
2. Haga clic en Examinar y busque Microsoft.Identity.Client.
3. Seleccione la versión más reciente de la Biblioteca de autenticación de Microsoft y haga clic en **Instalar**.

## <a name="install-the-microsoft-graph-client-library"></a>Instalar la biblioteca cliente de Microsoft Graph

> **Nota**: Si va a usar llamadas de REST sin procesar para obtener acceso a Microsoft Graph, puede omitir esta sección.

1. En el Explorador de soluciones, haga clic con el botón derecho en el nombre del proyecto y seleccione **Administrar paquetes de NuGet...**
2. Haga clic en Examinar y busque Microsoft.Graph.
3. Seleccione la versión más reciente de la Biblioteca cliente de Microsoft y haga clic en **Instalar**.

## <a name="create-the-authenticationhelper.cs-class"></a>Crear la clase AuthenticationHelper.cs

Abra el archivo AuthenticationHelper.cs en el proyecto inicial. El archivo contiene el código de autenticación completo, junto con la lógica adicional que almacena la información del usuario, y exige la autenticación solo cuando el usuario se ha desconectado de la aplicación. Esta clase contiene al menos dos métodos: `GetTokenForUserAsync` y `Signout`. Si usa la Biblioteca cliente de Microsoft Graph, necesitará agregar un tercer método: `GetAuthenticatedClient`.

El método ``GetTokenHelperAsync`` se ejecuta cuando el usuario se autentica y, posteriormente, cada vez que la aplicación llama a Microsoft Graph.

**Usar declaraciones**

***Versión de la biblioteca cliente***

Si usa la Biblioteca cliente de Microsoft Graph, necesitará las siguientes declaraciones:

```c#
using System;
using System.Diagnostics;
using System.Net.Http.Headers;
using System.Threading.Tasks;
using Microsoft.Graph;
using Microsoft.Identity.Client;
```

***Versión de REST***

Si está usando llamadas de REST sin procesar para obtener acceso a Microsoft Graph, necesitará las siguientes declaraciones `using` en la clase AuthenticationHelper:

```c#
using System;
using System.Threading.Tasks;
using Windows.Storage;
using Microsoft.Identity.Client;
```

**Campos de clase**

Ambas versiones de la clase AuthenticationHelper necesitarán estos campos:

```c#
// The Client ID is used by the application to uniquely identify itself to the Azure AD v2.0 endpoint.
static string clientId = App.Current.Resources["ida:ClientID"].ToString();
public static string[] Scopes = { "User.Read", "Mail.Send" };
public static PublicClientApplication IdentityClientApp = new PublicClientApplication(clientId);
public static string TokenForUser = null;
public static DateTimeOffset Expiration;
```

Fíjese en que las dos versiones usan la clase `PublicClientApplication` de MSAL para autenticar el usuario. El campo `Scopes` almacena los ámbitos de permisos de Microsoft Graph que la aplicación deberá solicitar cuando el usuario se autentique. 

***Versión de la biblioteca cliente***

Si usa la biblioteca cliente, almacene el `GraphServicesClient` como campo de modo que solo tenga que crearlo una vez:

```c#
private static GraphServiceClient graphClient = null;
```

***Versión de REST***

Si usa llamadas de REST, deberá almacenar algunos valores en la configuración de itinerancia de la aplicación, ya que necesitará almacenar información acerca del usuario. (La biblioteca cliente proporciona esta información en la otra versión).

```c#
public static ApplicationDataContainer _settings = ApplicationData.Current.RoamingSettings;
```

**GetTokenForUserAsync**

***Versión de la biblioteca cliente***

El método `GetTokenForUserAsync` usa los valores de configuración PublicClientApplicationClass y ClientId para obtener un token de acceso para el usuario. Si el usuario todavía no se ha autenticado, inicia la interfaz de usuario de autenticación. Esta es la versión del método que se usará si está usando la biblioteca cliente.

```c#
        public static async Task<string> GetTokenForUserAsync()
        {
            AuthenticationResult authResult;
            try
            {
                authResult = await IdentityClientApp.AcquireTokenSilentAsync(Scopes);
                TokenForUser = authResult.Token;
            }

            catch (Exception)
            {
                if (TokenForUser == null || Expiration <= DateTimeOffset.UtcNow.AddMinutes(5))
                {
                    authResult = await IdentityClientApp.AcquireTokenAsync(Scopes);

                    TokenForUser = authResult.Token;
                    Expiration = authResult.ExpiresOn;
                }
            }

            return TokenForUser;
        }
```

***Versión de REST***

La versión de REST del método `GetTokenForUserAsync` requiere algo más de trabajo, ya que también necesita almacenar algunos datos acerca del usuario que haya iniciado sesión.

```c#
        public static async Task<string> GetTokenForUserAsync()
        {
            AuthenticationResult authResult;
            try
            {
                authResult = await IdentityClientApp.AcquireTokenSilentAsync(Scopes);
                TokenForUser = authResult.Token;
                // save user ID in local storage
                _settings.Values["userID"] = authResult.User.UniqueId;
                _settings.Values["userEmail"] = authResult.User.DisplayableId;
                _settings.Values["userName"] = authResult.User.Name;
            }

            catch (Exception)
            {
                if (TokenForUser == null || Expiration <= DateTimeOffset.UtcNow.AddMinutes(5))
                {
                    authResult = await IdentityClientApp.AcquireTokenAsync(Scopes);

                    TokenForUser = authResult.Token;
                    Expiration = authResult.ExpiresOn;

                    // save user ID in local storage
                    _settings.Values["userID"] = authResult.User.UniqueId;
                    _settings.Values["userEmail"] = authResult.User.DisplayableId;
                    _settings.Values["userName"] = authResult.User.Name;
                }
            }

            return TokenForUser;
        }
```

**Cerrar sesión**

El método `Signout` de ambas versiones cierra la sesión de todos los usuarios que la hayan iniciado a través de la `PublicClientApplication` (en este caso, solo hay un usuario) y anula el valor `TokenForUser`. La versión que usa la biblioteca cliente también anula el `GraphServicesClient` almacenado y la versión de REST anula los valores almacenados en la configuración de itinerancia de la aplicación.

***Versión de la biblioteca cliente***

Esta es la versión de la biblioteca cliente del método `Signout`.

```c#
        public static void SignOut()
        {
            foreach (var user in IdentityClientApp.Users)
            {
                user.SignOut();
            }
            graphClient = null;
            TokenForUser = null;

        }
``` 

***Versión de REST***

Esta es la versión de REST del método `Signout`.

```c#
        public static void SignOut()
        {
            foreach (var user in IdentityClientApp.Users)
            {
                user.SignOut();
            }

            TokenForUser = null;

            //Clear stored values from last authentication.
            _settings.Values["userID"] = null;
            _settings.Values["userEmail"] = null;
            _settings.Values["userName"] = null;

        }
```

**GetAuthenticatedClient (solo para la biblioteca cliente)**

Por último, si usa la biblioteca cliente, necesitará un método que cree un `GraphServicesClient`. Este método crea un cliente que usa el método `GetTokenForUserAsync` para cada llamada a Microsoft Graph que realiza a través del cliente.

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

El método ``ComposeAndSendMailAsync`` adquiere tres valores de cadena (``subject``, ``bodyContent`` y ``recipients``) que le pasa el archivo MainPage.xaml.cs. Las cadenas ``subject`` y ``bodyContent`` se almacenan, junto con todas las demás cadenas de la interfaz de usuario, en el archivo Resources.resw. La cadena ``recipients`` proviene del cuadro de dirección en la interfaz de la aplicación. 

**Versión de REST**

La versión REST de este archivo requiere las siguientes declaraciones `using`:

```c#
using System;
using System.Threading.Tasks;
```

Ya que el usuario puede pasar potencialmente más de una dirección, la primera tarea es dividir la cadena ``recipients`` en un conjunto de objetos EmailAddress que pueden pasarse en el cuerpo POST de la solicitud:

```c#
            // Prepare the recipient list
            string[] splitter = { ";" };
            var splitRecipientsString = recipients.Split(splitter, StringSplitOptions.RemoveEmptyEntries);
            string recipientsJSON = null;

            int n = 0;
            foreach (string recipient in splitRecipientsString)
            {
                if ( n==0)
                recipientsJSON += "{'EmailAddress':{'Address':'" + recipient.Trim() + "'}}";
                else
                {
                    recipientsJSON += ", {'EmailAddress':{'Address':'" + recipient.Trim() + "'}}";
                }
                n++;
            }
```

La segunda tarea es construir un objeto de mensaje JSON válido y enviarlo al punto de conexión **me/microsoft.graph.SendMail** a través de una solicitud HTTP POST. Ya que la cadena ``bodyContent`` es un documento HTML, la solicitud establece el valor del parámetro **ContentType** en HTML. Tenga en cuenta también la llamada a ``AuthenticationHelper.GetTokenHelperAsync`` para garantizar que tenemos un token de acceso reciente para pasar en la solicitud.

```c#
                HttpClient client = new HttpClient();
                var token = await AuthenticationHelper.GetTokenHelperAsync();
                client.DefaultRequestHeaders.Add("Authorization", "Bearer " + token);

                // Build contents of post body and convert to StringContent object.
                // Using line breaks for readability.
                string postBody = "{'Message':{" 
                    +  "'Body':{ " 
                    + "'Content': '" + bodyContent + "'," 
                    + "'ContentType':'HTML'}," 
                    + "'Subject':'" + subject + "'," 
                    + "'ToRecipients':[" + recipientsJSON +  "]}," 
                    + "'SaveToSentItems':true}";

                var emailBody = new StringContent(postBody, System.Text.Encoding.UTF8, "application/json");

                HttpResponseMessage response = await client.PostAsync(new Uri("https://graph.microsoft.com/v1.0/me/microsoft.graph.SendMail"), emailBody);

                if ( !response.IsSuccessStatusCode)
                {

                    throw new Exception("We could not send the message: " + response.StatusCode.ToString());
                }
```

La clase completa tendrá este aspecto:

```c#
    class MailHelper
    {
        /// <summary>
        /// Compose and send a new email.
        /// </summary>
        /// <param name="subject">The subject line of the email.</param>
        /// <param name="bodyContent">The body of the email.</param>
        /// <param name="recipients">A semicolon-separated list of email addresses.</param>
        /// <returns></returns>
        internal async Task ComposeAndSendMailAsync(string subject,
                                                            string bodyContent,
                                                            string recipients)
        {

            // Prepare the recipient list
            string[] splitter = { ";" };
            var splitRecipientsString = recipients.Split(splitter, StringSplitOptions.RemoveEmptyEntries);
            string recipientsJSON = null;

            int n = 0;
            foreach (string recipient in splitRecipientsString)
            {
                if ( n==0)
                recipientsJSON += "{'EmailAddress':{'Address':'" + recipient.Trim() + "'}}";
                else
                {
                    recipientsJSON += ", {'EmailAddress':{'Address':'" + recipient.Trim() + "'}}";
                }
                n++;
            }

            try
            {

                HttpClient client = new HttpClient();
                var token = await AuthenticationHelper.GetTokenForUserAsync();
                client.DefaultRequestHeaders.Add("Authorization", "Bearer " + token);

                // Build contents of post body and convert to StringContent object.
                // Using line breaks for readability.
                string postBody = "{'Message':{" 
                    +  "'Body':{ " 
                    + "'Content': '" + bodyContent + "'," 
                    + "'ContentType':'HTML'}," 
                    + "'Subject':'" + subject + "'," 
                    + "'ToRecipients':[" + recipientsJSON +  "]}," 
                    + "'SaveToSentItems':true}";

                var emailBody = new StringContent(postBody, System.Text.Encoding.UTF8, "application/json");

                HttpResponseMessage response = await client.PostAsync(new Uri("https://graph.microsoft.com/v1.0/me/microsoft.graph.SendMail"), emailBody);

                if ( !response.IsSuccessStatusCode)
                {

                    throw new Exception("We could not send the message: " + response.StatusCode.ToString());
                }


            }

            catch (Exception e)
            {
                throw new Exception("We could not send the message: " + e.Message);
            }
        }
    }
```

**Versión de la biblioteca cliente**

La versión de la biblioteca cliente de este archivo requiere las siguientes declaraciones `using`:

```c#
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
```

El código para enviar un mensaje mediante la biblioteca cliente es muy similar al código que se usa para las llamadas de REST. En lugar de crear objetos JSON y pasarlos directamente al punto de conexión **SendMail** en una solicitud HTTP POST, deberá crear objetos equivalentes que se definan en la biblioteca cliente y pasar el objeto `Message` resultante al método `SendMail` del `GraphServiceClient`. El cliente realiza el trabajo de recuperar el token de acceso y pasar la solicitud al punto de conexión **SendMail**.

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

##<a name="create-the-user-interface-in-mainpage.xaml"></a>Crear la interfaz de usuario en MainPage.xaml

Ahora que ya ha escrito el código que realiza todo el trabajo de autenticación del usuario y del envío de un mensaje a través de Microsoft Graph, solo deberá crear la interfaz simple que se describe anteriormente. 

El archivo MainPage.xaml de su proyecto inicial ya incluye todo el lenguaje XAML que necesitará. Solo deberá agregar el código que dirige la interfaz al archivo MainPage.xaml.cs. Busque este archivo en el proyecto y ábralo.

Este archivo ya contiene todas las declaraciones `using` que se requieren tanto para la versión de la biblioteca cliente como para las versiones de REST del ejemplo.

***Versión de la biblioteca cliente***

La versión de la biblioteca cliente de la aplicación crea un `GraphServiceClient` cuando el usuario se autentica. 

Agregue este código en su espacio de nombres para completar la versión de la biblioteca cliente de la clase MainPage en MainPage.xaml.cs:

```c#
    public sealed partial class MainPage : Page
    {
        private string _mailAddress;
        private string _displayName = null;
        private MailHelper _mailHelper = new MailHelper();

        public MainPage()
        {
            this.InitializeComponent();
        }

        protected override void OnNavigatedTo(NavigationEventArgs e)
        {
            // Developer code - if you haven't registered the app yet, we warn you. 
            if (!App.Current.Resources.ContainsKey("ida:ClientID"))
            {
                InfoText.Text = ResourceLoader.GetForCurrentView().GetString("NoClientIdMessage");
                ConnectButton.IsEnabled = false;
            }
            else
            {
                InfoText.Text = ResourceLoader.GetForCurrentView().GetString("ConnectPrompt");
                ConnectButton.IsEnabled = true;
            }
        }

        /// <summary>
        /// Signs in the current user.
        /// </summary>
        /// <returns></returns>
        public async Task<bool> SignInCurrentUserAsync()
        {
            var graphClient = AuthenticationHelper.GetAuthenticatedClient();

            if (graphClient != null)
            {
                var user = await graphClient.Me.Request().GetAsync();
                string userId = user.Id;
                _mailAddress = user.UserPrincipalName;
                _displayName = user.DisplayName;
                return true;
            }
            else
            {
                return false;
            }

        }


        private async void ConnectButton_Click(object sender, RoutedEventArgs e)
        {
            ProgressBar.Visibility = Visibility.Visible;
            if (await SignInCurrentUserAsync())
            { 
                InfoText.Text = "Hi " + _displayName + "," + Environment.NewLine + ResourceLoader.GetForCurrentView().GetString("SendMailPrompt");
                MailButton.IsEnabled = true;
                EmailAddressBox.IsEnabled = true;
                ConnectButton.Visibility = Visibility.Collapsed;
                DisconnectButton.Visibility = Visibility.Visible;
                EmailAddressBox.Text = _mailAddress;
            }
            else
            {
                InfoText.Text = ResourceLoader.GetForCurrentView().GetString("AuthenticationErrorMessage");
            }

            ProgressBar.Visibility = Visibility.Collapsed;
        }

        private async void MailButton_Click(object sender, RoutedEventArgs e)
        {
            _mailAddress = EmailAddressBox.Text;
            ProgressBar.Visibility = Visibility.Visible;
            MailStatus.Text = string.Empty;
            try
            {
                await _mailHelper.ComposeAndSendMailAsync(ResourceLoader.GetForCurrentView().GetString("MailSubject"), ComposePersonalizedMail(_displayName), _mailAddress);
                MailStatus.Text = string.Format(ResourceLoader.GetForCurrentView().GetString("SendMailSuccess"), _mailAddress);
            }
            catch (Exception)
            {
                MailStatus.Text = ResourceLoader.GetForCurrentView().GetString("MailErrorMessage");
            }
            finally
            {
                ProgressBar.Visibility = Visibility.Collapsed;
            }
            
        }

        // <summary>
        // Personalizes the email.
        // </summary>
        public static string ComposePersonalizedMail(string userName)
        {
            return String.Format(ResourceLoader.GetForCurrentView().GetString("MailContents"), userName);
        }

        private void Disconnect_Click(object sender, RoutedEventArgs e)
        {
            ProgressBar.Visibility = Visibility.Visible;
            AuthenticationHelper.SignOut();
            ProgressBar.Visibility = Visibility.Collapsed;
            MailButton.IsEnabled = false;
            EmailAddressBox.IsEnabled = false;
            ConnectButton.Visibility = Visibility.Visible;
            InfoText.Text = ResourceLoader.GetForCurrentView().GetString("ConnectPrompt");
            this._displayName = null;
            this._mailAddress = null;
        }
    }
```

***Versión de REST***

La versión de REST de esta clase es muy similar a la versión de la biblioteca cliente, a excepción de que llama al método `GetTokenForUserAsync` directamente cuando el usuario se autentica. También recupera los valores del usuario de la configuración de itinerancia de la aplicación. 

Agregue este código en su espacio de nombres para completar la versión de REST de la clase MainPage en MainPage.xaml.cs:

```c#
    public sealed partial class MainPage : Page
    {
        private string _mailAddress;
        private string _displayName = null;
        private MailHelper _mailHelper = new MailHelper();
        public static ApplicationDataContainer _settings = ApplicationData.Current.RoamingSettings;

        public MainPage()
        {
            this.InitializeComponent();
        }

        protected override void OnNavigatedTo(NavigationEventArgs e)
        {
            // Developer code - if you haven't registered the app yet, we warn you. 
            if (!App.Current.Resources.ContainsKey("ida:ClientID"))
            {
                InfoText.Text = ResourceLoader.GetForCurrentView().GetString("NoClientIdMessage");
                ConnectButton.IsEnabled = false;
            }
            else
            {
                InfoText.Text = ResourceLoader.GetForCurrentView().GetString("ConnectPrompt");
                ConnectButton.IsEnabled = true;
            }
        }

        /// <summary>
        /// Signs in the current user.
        /// </summary>
        /// <returns></returns>
        public async Task<bool> SignInCurrentUserAsync()
        {
            var token = await AuthenticationHelper.GetTokenForUserAsync();

            if (token != null)
            {
                string userId = (string)_settings.Values["userID"];
                _mailAddress = (string)_settings.Values["userEmail"];
                _displayName = (string)_settings.Values["userName"];
                return true;
            }
            else
            {
                return false;
            }

        }


        private async void ConnectButton_Click(object sender, RoutedEventArgs e)
        {
            ProgressBar.Visibility = Visibility.Visible;
            if (await SignInCurrentUserAsync())
            { 
                InfoText.Text = "Hi " + _displayName + "," + Environment.NewLine + ResourceLoader.GetForCurrentView().GetString("SendMailPrompt");
                MailButton.IsEnabled = true;
                EmailAddressBox.IsEnabled = true;
                ConnectButton.Visibility = Visibility.Collapsed;
                DisconnectButton.Visibility = Visibility.Visible;
                EmailAddressBox.Text = _mailAddress;
            }
            else
            {
                InfoText.Text = ResourceLoader.GetForCurrentView().GetString("AuthenticationErrorMessage");
            }

            ProgressBar.Visibility = Visibility.Collapsed;
        }

        private async void MailButton_Click(object sender, RoutedEventArgs e)
        {
            _mailAddress = EmailAddressBox.Text;
            ProgressBar.Visibility = Visibility.Visible;
            MailStatus.Text = string.Empty;
            try
            {
                await _mailHelper.ComposeAndSendMailAsync(ResourceLoader.GetForCurrentView().GetString("MailSubject"), ComposePersonalizedMail(_displayName), _mailAddress);
                MailStatus.Text = string.Format(ResourceLoader.GetForCurrentView().GetString("SendMailSuccess"), _mailAddress);
            }
            catch (Exception)
            {
                MailStatus.Text = ResourceLoader.GetForCurrentView().GetString("MailErrorMessage");
            }
            finally
            {
                ProgressBar.Visibility = Visibility.Collapsed;
            }
            
        }

        // <summary>
        // Personalizes the email.
        // </summary>
        public static string ComposePersonalizedMail(string userName)
        {
            return String.Format(ResourceLoader.GetForCurrentView().GetString("MailContents"), userName);
        }

        private void Disconnect_Click(object sender, RoutedEventArgs e)
        {
            ProgressBar.Visibility = Visibility.Visible;
            AuthenticationHelper.SignOut();
            ProgressBar.Visibility = Visibility.Collapsed;
            MailButton.IsEnabled = false;
            EmailAddressBox.IsEnabled = false;
            ConnectButton.Visibility = Visibility.Visible;
            InfoText.Text = ResourceLoader.GetForCurrentView().GetString("ConnectPrompt");
            this._displayName = null;
            this._mailAddress = null;
        }
    }
```
 
Ya ha llevado a cabo los tres pasos requeridos para interactuar con Microsoft Graph: el registro de la aplicación, la autenticación del usuario y la creación de una solicitud. 

## <a name="run-the-app"></a>Ejecutar la aplicación
1. Pulse F5 para compilar y ejecutar la aplicación. 

2. Inicie sesión con su cuenta personal, profesional o educativa y conceda los permisos solicitados.

3. Elija el botón **Enviar correo electrónico**. Cuando se envíe el correo, se mostrará un mensaje de Operación correcta debajo del botón.

## <a name="next-steps"></a>Pasos siguientes
- Pruebe la API de REST mediante el [Probador de Graph](https://graph.microsoft.io/graph-explorer).
- Busque ejemplos de operaciones comunes tanto para REST como para SDK en el [Ejemplo de fragmentos de código de UWP para Microsoft Graph (SDK)](https://github.com/microsoftgraph/uwp-csharp-snippets-sample) y en el [Ejemplo de fragmentos de código de UWP para Microsoft Graph (REST)](https://github.com/microsoftgraph/uwp-csharp-snippets-rest-sample) o explore el resto de nuestros [ejemplos de UWP](https://github.com/microsoftgraph?utf8=%E2%9C%93&query=uwp) en GitHub.

## <a name="see-also"></a>Recursos adicionales
- [Biblioteca cliente .NET de Microsoft Graph](https://github.com/microsoftgraph/msgraph-sdk-dotnet)
- [Protocolos de Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
- [Tokens de Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)

