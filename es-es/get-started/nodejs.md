# <a name="get-started-with-microsoft-graph-in-a-node.js-app"></a>Introducción a Microsoft Graph en una aplicación de Node.js

En este artículo, se describen las tareas necesarias para obtener un token de acceso desde el punto de conexión v2.0 de Azure AD y llamar a Microsoft Graph. Le muestra los pasos para la compilación del [Ejemplo Connect de Microsoft para Node.js](https://github.com/microsoftgraph/nodejs-connect-rest-sample) y explica los conceptos principales que implementará para usar Microsoft Graph. El artículo describe cómo obtener acceso a la API de Microsoft Graph mediante llamadas de REST sin procesar.

En la imagen siguiente, se muestra la aplicación que va a crear. 

![Aplicación web después del inicio de sesión que muestra el botón “Enviar correo”](./images/web-screenshot.png)


**¿No desea compilar una aplicación?** Use el [inicio rápido de Microsoft Graph](https://graph.microsoft.io/en-us/getting-started) para ponerlo todo en funcionamiento de manera rápida.

Para descargar una versión del ejemplo Connect que usa el punto de conexión de Azure AD, consulte el [Ejemplo Connect de Microsoft Graph para Node.js](https://github.com/microsoftgraph/nodejs-connect-rest-sample/releases/tag/last_v1_auth).


## <a name="prerequisites"></a>Requisitos previos

Para comenzar, necesitará: 

- Una [cuenta Microsoft](https://www.outlook.com/) o una [cuenta profesional o educativa](http://dev.office.com/devprogram)
- [Node.js con npm](https://nodejs.org/en/download/) 
- [Ejemplo Connect de Microsoft Graph para Node.js](https://github.com/microsoftgraph/nodejs-connect-rest-sample). Usará la carpeta **starter-project** en los archivos de ejemplo para este tutorial.

## <a name="register-the-application"></a>Registrar la aplicación
Registre una aplicación en el Portal de registro de aplicaciones de Microsoft. Esta acción generará el identificador de la aplicación y la contraseña que usará para configurar la aplicación en Visual Studio.

1. Inicie sesión en el [Portal de registro de aplicaciones de Microsoft](https://apps.dev.microsoft.com/) mediante su cuenta personal, profesional o educativa.

2. Seleccione **Agregar una aplicación**.

3. Escriba un nombre para la aplicación y seleccione **Crear aplicación**. 
    
    Se muestra la página de registro, indicando las propiedades de la aplicación.

4. Copie el identificador de la aplicación. Se trata del identificador único para su aplicación. 

5. En **Secretos de aplicación**, seleccione **Generar nueva contraseña**. Copie la contraseña del cuadro de diálogo **Nueva contraseña generada**.

    Deberá usar el identificador de la aplicación y la contraseña (secreto) para configurar la aplicación. 

6. En **Plataformas**, elija **Agregar plataforma** > **Web**.

7. Escriba *http://localhost:3000/login* como URI de redireccionamiento. 

8. Elija **Guardar**.


## <a name="configure-the-project"></a>Configurar el proyecto
1. Abra la carpeta **starter-project** en los archivos de ejemplo.

1. En un símbolo del sistema, ejecute el siguiente comando en el directorio raíz del proyecto inicial. Esta acción instalará las dependencias del proyecto.

        npm install

1. En los archivos del proyecto inicial, abra el archivo authHelper.js.


1. En el campo **Credenciales**, remplace los valores de los marcadores de posición **ENTER\_YOUR\_CLIENT\_ID** y **ENTER\_YOUR\_SECRET** por los valores que acaba de copiar.

  
## <a name="authenticate-the-user-and-get-an-access-token"></a>Autenticar al usuario y obtener un token de acceso
En este paso, agregará el código de administración del token y de inicio de sesión. Pero, primero, echemos un vistazo más de cerca al flujo de autenticación.

Esta aplicación usa el flujo de concesión de códigos de autorización con una identidad de usuario delegado. Para una aplicación web, el flujo requiere el identificador de la aplicación, el secreto y el URI de redireccionamiento de la aplicación registrada. 

El flujo de autenticación puede desglosarse en los siguientes pasos básicos:

1. Redirigir al usuario para la autenticación y el consentimiento
2. Obtener un código de autorización
3. Canjear el código de autorización por un token de acceso
4. Use el token de actualización para obtener un token de acceso nuevo cuando expire el actual.

La aplicación usa el middleware [oauth](https://www.npmjs.com/package/oauth) para autenticar y obtener tokens. Usa el middleware [cookie-parser](https://www.npmjs.com/package/cookie-parser) para almacenar en caché la información del token en las cookies. El código que se usa para almacenar y obtener acceso a la información de token se encuentra en el controlador index.js.
    
   >**Importante**: La autenticación simple y el uso de tokens en este proyecto se realizan solo con fines de ejemplo. En una aplicación de producción, deberá crear una forma más segura de usar la autenticación, incluidos el uso seguro de tokens y la validación.

Ahora, volvamos a la compilación de la aplicación.

1. En authHelper.js, reemplace la función *getTokenFromCode* por el código siguiente. Esta acción obtiene un token de acceso mediante un código de autorización.

        function getTokenFromCode(code, callback) {
            var OAuth2 = OAuth.OAuth2;
            var oauth2 = new OAuth2(
                credentials.client_id,
                credentials.client_secret,
                credentials.authority,
                credentials.authorize_endpoint,
                credentials.token_endpoint
            );

            oauth2.getOAuthAccessToken(
                code,
                {
                    grant_type: 'authorization_code',
                    redirect_uri: credentials.redirect_uri,
                    response_mode: 'form_post',
                    nonce: uuid.v4(),
                    state: 'abcd'
                },
                function (e, accessToken, refreshToken) {
                    callback(e, accessToken, refreshToken);
                }
            );
        }

1. Reemplace la función **getTokenFromRefreshToken** por el siguiente código. Esta acción obtiene un token de acceso mediante un token de actualización.

        function getTokenFromRefreshToken(refreshToken, callback) {
            var OAuth2 = OAuth.OAuth2;
            var oauth2 = new OAuth2(
                credentials.client_id,
                credentials.client_secret,
                credentials.authority,
                credentials.authorize_endpoint,
                credentials.token_endpoint
            );

            oauth2.getOAuthAccessToken(
                refreshToken,
                {
                    grant_type: 'refresh_token',
                    redirect_uri: credentials.redirect_uri,
                    response_mode: 'form_post',
                    nonce: uuid.v4(),
                    state: 'abcd'
                },
                function (e, accessToken) {
                    callback(e, accessToken);
                }
            );
        }

Ahora, ya puede agregar el código para llamar a Microsoft Graph. 

## <a name="call-microsoft-graph"></a>Llamar a Microsoft Graph
La aplicación llama a Microsoft Graph para obtener información del usuario y para enviar un correo electrónico en nombre de este. Estas llamadas se inician desde el controlador index.js en respuesta a los eventos de la interfaz de usuario.

1. Abra el archivo requestUtil.js.

1. Reemplace la función **getUserData** por el siguiente código. Esta acción configura y envía la solicitud GET al punto de conexión */me* y procesa la respuesta.

        function getUserData(accessToken, callback) {
            var options = {
                host: 'graph.microsoft.com',
                path: '/v1.0/me',
                method: 'GET',
                headers: {
                    'Content-Type': 'application/json',
                    Accept: 'application/json',
                    Authorization: 'Bearer ' + accessToken
                }
            };

            https.get(options, function (response) {
                var body = '';
                response.on('data', function (d) {
                    body += d;
                });
                response.on('end', function () {
                    var error;
                    if (response.statusCode === 200) {
                        callback(null, JSON.parse(body));
                    } else {
                        error = new Error();
                        error.code = response.statusCode;
                        error.message = response.statusMessage;
                        // The error body sometimes includes an empty space
                        // before the first character, remove it or it causes an error.
                        body = body.trim();
                        error.innerError = JSON.parse(body).error;
                        callback(error, null);
                    }
                });
            }).on('error', function (e) {
                callback(e, null);
            });
        }

1. Reemplace la función **postSendMail** por el siguiente código. Esta acción configura y envía la solicitud POST al punto de conexión */me/sendMail* y procesa la respuesta.

        function postSendMail(accessToken, mailBody, callback) {
            var outHeaders = {
                'Content-Type': 'application/json',
                Authorization: 'Bearer ' + accessToken,
                'Content-Length': mailBody.length
            };
            var options = {
                host: 'graph.microsoft.com',
                path: '/v1.0/me/sendMail',
                method: 'POST',
                headers: outHeaders
            };

            // Set up the request
            var post = https.request(options, function (response) {
                var body = '';
                response.on('data', function (d) {
                    body += d;
                });
                response.on('end', function () {
                    var error;
                    if (response.statusCode === 202) {
                        callback(null);
                    } else {
                        error = new Error();
                        error.code = response.statusCode;
                        error.message = response.statusMessage;
                        // The error body sometimes includes an empty space
                        // before the first character, remove it or it causes an error.
                        body = body.trim();
                        error.innerError = JSON.parse(body).error;
                        // Note: If you receive a 500 - Internal Server Error
                        // while using a Microsoft account (outlook.com, hotmail.com or live.com),
                        // it's possible that your account has not been migrated to support this flow.
                        // Check the inner error object for code 'ErrorInternalServerTransientError'.
                        // You can try using a newly created Microsoft account or contact support.
                        callback(error);
                    }
                });
            });
            
            // write the outbound data to it
            post.write(mailBody);
            // we're done!
            post.end();

            post.on('error', function (e) {
                callback(e);
            });
        }
  
1. Abra el archivo emailer.js.

1. Reemplace la función **wrapEmail** por el siguiente código. Esta acción compila la carga que representa el mensaje de correo electrónico que se enviará.

        function wrapEmail(content, recipient) {
            var emailAsPayload = {
                Message: {
                    Subject: 'Welcome to Office 365 development with Node.js and the Office 365 Connect sample',
                    Body: {
                        ContentType: 'HTML',
                        Content: content
                    },
                    ToRecipients: [
                        {
                            EmailAddress: {
                                Address: recipient
                            }
                        }
                    ]
                },
                SaveToSentItems: true
            };
            return emailAsPayload;
        }

## <a name="run-the-app"></a>Ejecutar la aplicación

1. En un símbolo del sistema, ejecute el siguiente comando en el directorio raíz del proyecto inicial.


        npm start

1. En un explorador, vaya a *http://localhost:3000* y elija el botón **Conectarse a Office 365**.

1. Inicie sesión y conceda los permisos solicitados. 

1. De forma opcional, edite la dirección de correo electrónico del destinatario y elija el botón **Enviar correo**. Cuando se envíe el correo, se mostrará un mensaje de Operación correcta debajo del botón. 

## <a name="next-steps"></a>Pasos siguientes
- Pruebe la API de REST mediante el [Probador de Graph](https://graph.microsoft.io/graph-explorer).
- Explore el resto de nuestros [ejemplos de Node.js](https://github.com/search?utf8=%E2%9C%93&q=node+sample+user%3Amicrosoftgraph&type=Repositories&ref=searchresults) en GitHub.


## <a name="see-also"></a>Recursos adicionales
- [Protocolos de Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
- [Tokens de Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)