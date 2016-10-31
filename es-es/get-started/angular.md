# <a name="get-started-with-microsoft-graph-in-an-angularjs-app"></a>Introducción a Microsoft Graph en una aplicación de AngularJS

En este artículo, se describen las tareas necesarias para obtener un token de acceso desde el punto de conexión v2.0 de Azure AD y llamar a Microsoft Graph. Le muestra los pasos para la compilación del [Ejemplo Connect de Microsoft para AngularJS](https://github.com/microsoftgraph/angular-connect-rest-sample) y explica los conceptos principales que implementará para usar Microsoft Graph. El artículo describe cómo obtener acceso a la API de Microsoft Graph mediante llamadas de REST sin procesar.

En la imagen siguiente, se muestra la aplicación que va a crear. 

![Aplicación web después del inicio de sesión que muestra el botón “Enviar correo”](./images/angular-connect-sample.png)


**¿No desea compilar una aplicación?** Use el [inicio rápido de Microsoft Graph](https://graph.microsoft.io/en-us/getting-started) para ponerlo todo en funcionamiento de manera rápida.

Para descargar una versión del ejemplo Connect que usa el punto de conexión de Azure AD, consulte el [Ejemplo Connect de Microsoft Graph para AngularJS](https://github.com/microsoftgraph/angular-connect-rest-sample/releases/tag/last_v1_auth).


## <a name="prerequisites"></a>Requisitos previos

Para comenzar, necesitará: 

- Una [cuenta Microsoft](https://www.outlook.com/) o una [cuenta profesional o educativa](http://dev.office.com/devprogram)
- [Node.js con npm](https://nodejs.org/en/download/)
- [Bower](https://bower.io)
- [Ejemplo Connect de Microsoft para AngularJS](https://github.com/microsoftgraph/angular-connect-rest-sample). Usará la carpeta **starter-project** en los archivos de ejemplo para este tutorial.

## <a name="register-the-application"></a>Registrar la aplicación
Registre una aplicación en el Portal de registro de aplicaciones de Microsoft. Esta acción generará el identificador de la aplicación y la contraseña que usará para configurar la aplicación en Visual Studio.

1. Inicie sesión en el [Portal de registro de aplicaciones de Microsoft](https://apps.dev.microsoft.com/) mediante su cuenta personal, profesional o educativa.

2. Seleccione **Agregar una aplicación**.

3. Escriba un nombre para la aplicación y seleccione **Crear aplicación**. 
    
    Se muestra la página de registro, indicando las propiedades de la aplicación.

4. Copie el identificador de la aplicación. Este es el identificador único de la aplicación que usará para configurarla.

5. En **Plataformas**, elija **Agregar plataforma** > **Web**.

7. Asegúrese de que la casilla **Permitir flujo implícito** está seleccionada y escriba *http://localhost:8080/login* como URI de redireccionamiento. 

8. Elija **Guardar**.


## <a name="configure-the-project"></a>Configurar el proyecto
1. Abra la carpeta **starter-project** en los archivos de ejemplo.
2. En un símbolo del sistema, ejecute los siguientes comandos en el directorio raíz del proyecto inicial. Esta acción instalará las dependencias del proyecto.

    npm install  bower install hello

3. En los archivos del proyecto inicial, en la carpeta **public/scripts**, abra el archivo config.js.
4. En el campo **clientId**, reemplace el valor del marcador de posición **ENTER_YOUR_CLIENT_ID** por el identificador de la aplicación que acaba de copiar.

  
## <a name="authenticate-the-user-and-get-an-access-token"></a>Autenticar al usuario y obtener un token de acceso
En este paso, agregará el código de recuperación del token y de inicio de sesión. Pero, primero, echemos un vistazo más de cerca al flujo de autenticación.

Esta aplicación de página única usa una implementación muy básica del flujo de concesión implícita, que requiere el identificador de la aplicación y el URI de redireccionamiento de la aplicación registrada. 

El flujo de autenticación puede desglosarse en los siguientes pasos básicos:

1. Redirigir al usuario para la autenticación y el consentimiento.
2. Obtener un token de acceso.

La aplicación usa la biblioteca del lado cliente [HelloJS](https://adodson.com/hello.js) para autenticar y obtener tokens. La aplicación almacena el token de acceso en el almacenamiento local.
    
   >**Importante**: La autenticación simple y el uso de tokens en este proyecto se realizan solo con fines de ejemplo. En una aplicación de producción, deberá crear una forma más segura de usar la autenticación, incluidos el uso seguro de tokens y la validación.

Ahora, volvamos a la compilación de la aplicación.

1 - Abra el archivo aad.js y agregue el código siguiente. Esta acción configura la comunicación con el proveedor de autenticación de Azure AD y agrega un agente de escucha que almacena la respuesta de autenticación que contiene el token de acceso. (Las referencias del script a HelloJS ya se han agregado a la vista index.html).

```
hello.init({
      
  aad: {
    name: 'Azure Active Directory', 
    oauth: {
      version: 2,
      auth: 'https://login.microsoftonline.com/common/oauth2/v2.0/authorize',
      grant: 'https://login.microsoftonline.com/common/oauth2/v2.0/token'
    },
    scope_delim: ' ',

    // Don't even try submitting via form.
    // This means no POST operations in <=IE9
    form: false
  }
});

hello.on('auth.login', function (auth) {

    // save the auth info into localStorage
    localStorage.auth = angular.toJson(auth.authResponse);
});
```

2 - En graphHelper.js, reemplace *// Initialize the auth request* por el código siguiente. Esto establece los parámetros de la solicitud de autenticación.

```
// Initialize the auth request.
hello.init( {
  aad: clientId // from public/scripts/config.js
  }, {
  redirect_uri: redirectUrl,
  scope: graphScopes
});
```

3 - Reemplace *// Sign in and sign out the user* por el código siguiente. La función **login** usa HelloJS para obtener la información del token. El agente de escucha en aad.js almacena esta información (incluido el token de acceso) en el almacenamiento local.

```
// Sign in and sign out the user.
login: function login() {
  hello('aad').login({
    display: 'page',
    state: 'abcd'
  });
},
logout: function logout() {
  hello('aad').logout();
  delete localStorage.auth;
  delete localStorage.user;
},
```

Ahora, ya puede agregar el código para llamar a Microsoft Graph. 

## <a name="call-microsoft-graph"></a>Llamar a Microsoft Graph
La aplicación llama a Microsoft Graph para obtener información del usuario y para enviar un correo electrónico en nombre de este. Estas llamadas se inician desde MainController en respuesta a los eventos de la interfaz de usuario.

1 - En el archivo graphHelper.js, reemplace *// Get the profile of the current user* por el código siguiente. Esta acción configura y envía la solicitud GET al punto de conexión */me* y procesa la respuesta.

```
// Get the profile of the current user.
  me: function me() {
    return $http.get('https://graph.microsoft.com/v1.0/me');
},
```
  
2- Reemplace *// Send an email on behalf of the current user* por el código siguiente. Esta acción configura y envía la solicitud POST al punto de conexión */me/sendMail* y procesa la respuesta.

```
// Send an email on behalf of the current user.
sendMail: function sendMail(email) {
  return $http.post('https://graph.microsoft.com/v1.0/me/sendMail', { 'message' : email, 'saveToSentItems': true });        
}
```

3- En la carpeta **public/controllers**, abra el archivo mainController.js.

4- Reemplace *// Set the default headers and user properties* por el código siguiente. Esta acción agrega el token de acceso a la solicitud HTTP, llama a **GraphHelper.me** para obtener el perfil del usuario actual y procesa la respuesta.

```
// Set the default headers and user properties.
function processAuth() {
  let auth = angular.fromJson(localStorage.auth); 

  // Check token expiry. If the token is valid for another 5 minutes, we'll use it.       
  let expiration = new Date();
  expiration.setTime((auth.expires - 300) * 1000); 
  if (expiration > new Date()) {

    // Add the required Authorization header with bearer token.
    $http.defaults.headers.common.Authorization = 'Bearer ' + auth.access_token;
        
    // This header has been added to identify our sample in the Microsoft Graph service. If extracting this code for your project please remove.
    $http.defaults.headers.common.SampleID = 'angular-connect-rest-starter';
        
    if (localStorage.getItem('user') === null) {

      // Get the profile of the current user.
      GraphHelper.me().then(function(response) {

        // Save the user to localStorage.
        let user =response.data;
        localStorage.setItem('user', angular.toJson(user));

        vm.displayName = user.displayName;
        vm.emailAddress = user.mail || user.userPrincipalName;
      });
   } else {
     let user = angular.fromJson(localStorage.user);

     vm.displayName = user.displayName;
     vm.emailAddress = user.mail || user.userPrincipalName;
    }
  }
} 
```

5- Reemplace *// Send an email on behalf of the current user* por el código siguiente. Esta acción compila el mensaje de correo electrónico, llama a **GraphHelper.sendMail** y procesa la respuesta.

```
// Send an email on behalf of the current user.
function sendMail() {

  // Check token expiry. If the token is valid for another 5 minutes, we'll use it.
  let auth = angular.fromJson(localStorage.auth);
  let expiration = new Date();
  expiration.setTime((auth.expires - 300) * 1000);
  if (expiration > new Date()) {
  
    // Build the HTTP request payload (the Message object).
    var email = {
        Subject: 'Welcome to Microsoft Graph development with AngularJS and the Microsoft Graph Connect sample',
        Body: {
            ContentType: 'HTML',
            Content: getEmailContent()
        },
        ToRecipients: [
            {
                EmailAddress: {
                    Address: vm.emailAddress
                }
            }
        ]
    };

    // Save email address so it doesn't get lost with two way data binding.
    vm.emailAddressSent = vm.emailAddress;

    GraphHelper.sendMail(email)
        .then(function (response) {
            $log.debug('HTTP request to the Microsoft Graph API returned successfully.', response);
            response.status === 202 ? vm.requestSuccess = true : vm.requestSuccess = false;
            vm.requestFinished = true;
        }, function (error) {
            $log.error('HTTP request to the Microsoft Graph API failed.');
            vm.requestSuccess = false;
            vm.requestFinished = true;
        });
    } else {

    // If the token is expired, this sample just redirects the user to sign in.
    GraphHelper.login();
    }
};

// Get the HTMl for the email to send.
function getEmailContent() {
  return "<html><head> <meta http-equiv=\'Content-Type\' content=\'text/html; charset=us-ascii\'> <title></title> </head><body style=\'font-family:calibri\'> <p>Congratulations " + vm.displayName + ",</p> <p>This is a message from the Microsoft Graph Connect sample. You are well on your way to incorporating Microsoft Graph endpoints in your apps. </p> <h3>What&#8217;s next?</h3><ul><li>Check out <a href='https://graph.microsoft.io' target='_blank'>graph.microsoft.io</a> to start building Microsoft Graph apps today with all the latest tools, templates, and guidance to get started quickly.</li><li>Use the <a href='https://graph.microsoft.io/graph-explorer' target='_blank'>Graph explorer</a> to explore the rest of the APIs and start your testing.</li><li>Browse other <a href='https://github.com/microsoftgraph/' target='_blank'>samples on GitHub</a> to see more of the APIs in action.</li></ul> <h3>Give us feedback</h3> <ul><li>If you have any trouble running this sample, please <a href='https://github.com/microsoftgraph/angular-connect-rest-sample/issues' target='_blank'>log an issue</a>.</li><li>For general questions about the Microsoft Graph API, post to <a href='https://stackoverflow.com/questions/tagged/microsoftgraph?sort=newest' target='blank'>Stack Overflow</a>. Make sure that your questions or comments are tagged with [microsoftgraph].</li></ul><p>Thanks and happy coding!<br>Your Microsoft Graph samples development team</p> <div style=\'text-align:center; font-family:calibri\'> <table style=\'width:100%; font-family:calibri\'> <tbody> <tr> <td><a href=\'https://github.com/microsoftgraph/angular-connect-rest-sample\'>See on GitHub</a> </td> <td><a href=\'https://officespdev.uservoice.com/\'>Suggest on UserVoice</a> </td> <td><a href=\'https://twitter.com/share?text=I%20just%20started%20developing%20%23Angular%20apps%20using%20the%20%23MicrosoftGraph%20Connect%20sample!%20&url=https://github.com/microsoftgraph/angular-connect-rest-sample\'>Share on Twitter</a> </td> </tr> </tbody> </table> </div>  </body> </html>";
};
```

6- Guarde todos los cambios.

## <a name="run-the-app"></a>Ejecutar la aplicación

1- En un símbolo del sistema, ejecute el siguiente comando en el directorio raíz del proyecto inicial.

```
npm start
```

2 - En un explorador, vaya a *http://localhost:8080* y elija el botón **Conectar**.

3 - Inicie sesión y conceda los permisos solicitados. 

4- De forma opcional, edite la dirección de correo electrónico del destinatario y elija el botón **Enviar correo**. Cuando se envíe el correo, se mostrará un mensaje de Operación correcta debajo del botón. 

## <a name="next-steps"></a>Pasos siguientes
- Pruebe la API de REST mediante el [Probador de Graph](https://graph.microsoft.io/graph-explorer).
- Explore el resto de nuestros [ejemplos de AngularJS](https://github.com/search?utf8=%E2%9C%93&q=angular+sample+user%3Amicrosoftgraph&type=Repositories&ref=searchresults) en GitHub.


## <a name="see-also"></a>Recursos adicionales
- [Protocolos de Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
- [Tokens de Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)