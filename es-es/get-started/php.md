# <a name="get-started-with-microsoft-graph-in-a-php-app"></a>Introducción a Microsoft Graph en una aplicación PHP

En este artículo se describen las tareas necesarias para obtener un token de acceso desde el punto de conexión v2.0 de Azure AD y llamar a Microsoft Graph. Le muestra los pasos para la compilación del [Ejemplo Connect para PHP](https://github.com/microsoftgraph/php-connect-rest-sample) y explica los conceptos principales que implementará para usar Microsoft Graph. El artículo también describe cómo acceder a Microsoft Graph mediante llamadas de REST.

Para utilizar Microsoft Graph en la aplicación PHP, debe mostrar la página de inicio de sesión de Microsoft a los usuarios. La siguiente captura de pantalla muestra una página de inicio de sesión para cuentas de Microsoft.

![Página de inicio de sesión para cuentas Microsoft](images/MicrosoftSignIn.png)

**¿No desea compilar una aplicación?** Puede ponerlo todo en funcionamiento de manera rápida descargando el [Ejemplo Connect para PHP](https://github.com/microsoftgraph/php-connect-rest-sample) en el que se basa este artículo.


## <a name="prerequisites"></a>Requisitos previos

Para comenzar, necesitará: 

- Una [cuenta Microsoft](https://www.outlook.com/) o una [cuenta profesional o educativa](http://dev.office.com/devprogram)
- PHP (versión 5.5.9 o superior)
- [Composer](https://getcomposer.org/)


## <a name="register-the-application"></a>Registrar la aplicación
Registre una aplicación en el Portal de registro de aplicaciones de Microsoft. Este proceso generará el identificador de la aplicación y la contraseña que utilizará para configurar la aplicación.

1. Inicie sesión en el [Portal de registro de aplicaciones de Microsoft](https://apps.dev.microsoft.com/) mediante su cuenta personal, profesional o educativa.

2. Seleccione **Agregar una aplicación**.

3. Escriba un nombre para la aplicación y seleccione **Crear aplicación**. 
    
    Se muestra la página de registro, indicando las propiedades de la aplicación.

4. Elija **Generar nueva contraseña**.

5. Copie el id. de la aplicación y la contraseña.

6. Elija **Agregar plataforma** y **Web**.

7. En el campo **URI de redirección**, escriba `http://localhost:8000/oauth`.

8. Seleccione **Guardar**.


## <a name="configure-the-project"></a>Configurar el proyecto

Inicie un nuevo proyecto mediante Composer. Para crear un nuevo proyecto PHP mediante el marco Laravel, use el siguiente comando:

```bash
composer create-project --prefer-dist laravel/laravel getstarted
```
 
Esta acción creará una carpeta **getstarted** que podrá usar para este proyecto.

> Nota: También puede usar el [Proyecto inicial](https://github.com/microsoftgraph/php-connect-rest-sample/tree/master/starter-project) que se ocupa de la configuración del proyecto para que el usuario pueda centrarse en las secciones de codificación de este tutorial.

## <a name="authenticate-the-user-and-get-an-access-token"></a>Autenticar al usuario y obtener un token de acceso
Use una biblioteca de OAuth para simplificar el proceso de autenticación. [La liga de PHP](http://thephpleague.com/) proporciona una [biblioteca cliente de OAuth](https://github.com/thephpleague/oauth2-client) que podrá usar en este proyecto.

### <a name="add-the-dependency-to-composer"></a>Agregar la dependencia a Composer

Abra el archivo `composer.json` e incluya la siguiente dependencia en la sección **require**:

```json
"league/oauth2-client": "^1.4"
```

Actualice las dependencias ejecutando el siguiente comando:

```bash
composer update
```

### <a name="start-the-authentication-flow"></a>Inicie el flujo de autenticación

1. Abra el archivo **resources** > **views** > **welcome.blade.php**. Reemplace el elemento div **title** por el siguiente código.
    ```html
    <div class="title" onClick="window.location='/oauth'">Sign in to Microsoft</div>
    ```
    
2. Escriba una sugerencia de la clase `Illuminate\Http\Request` en el archivo **app** > **Http** > **routes.php**. Agregue la siguiente línea antes de cualquier declaración de ruta.
    ```php
    use Illuminate\Http\Request;
    ```
    
3. Agregue una ruta */oauth* al archivo **app** > **Http** > **routes.php**. Para agregar la ruta, copie el siguiente código después de la declaración de ruta predeterminada. Inserte el **Id. de la aplicación** y la **contraseña** de la aplicación en los marcadores de posición marcados como **\<YOUR_APPLICATION_ID\>** y **\<YOUR_PASSWORD\>**, respectivamente.
    ```php
    Route::get('/oauth', function () {
        $provider = new \League\OAuth2\Client\Provider\GenericProvider([
            'clientId'                => '<YOUR_APPLICATION_ID>',
            'clientSecret'            => '<YOUR_PASSWORD>',
            'redirectUri'             => 'http://localhost:8000/oauth',
            'urlAuthorize'            => 'https://login.microsoftonline.com/common/oauth2/v2.0/authorize',
            'urlAccessToken'          => 'https://login.microsoftonline.com/common/oauth2/v2.0/token',
            'urlResourceOwnerDetails' => '',
            'scopes'                  => 'openid mail.send'
        ]);

        if (!$request->has('code')) {
            return redirect($provider->getAuthorizationUrl());
        }
    });
    ```
    
Cuando llegue a este punto, deberá tener una aplicación PHP que muestre lo siguiente: *Iniciar sesión en Microsoft*. Si hace clic en el texto, la aplicación muestra la página de inicio de sesión de Microsoft. El siguiente paso es controlar el código que envía el servidor de autorización a la URI de redirección y cambiarlo por un token de acceso.

### <a name="exchange-the-authorization-code-for-an-access-token"></a>Cambiar el código de autorización por un token de acceso

Es necesario usar la respuesta del servidor de autorización, que contiene un código que puede intercambiar por un token de acceso.

Actualice la ruta */oauth* de manera que pueda obtener un token de acceso con el código de autorización. Para ello, abra el archivo **app** > **Http** > **routes.php** y agregue la siguiente cláusula condicional *else* a la instrucción *if* existente.

```php
if (!$request->has('code')) {
    ...
    // add the following lines
} else {
    $accessToken = $provider->getAccessToken('authorization_code', [
        'code'     => $request->input('code')
    ]);
    exit($accessToken->getToken());
}
```
    
Tenga en cuenta que hay un token de acceso en la línea siguiente: `exit($accessToken->getToken());`. Ahora, ya puede agregar el código para llamar a Microsoft Graph. 

## <a name="call-microsoft-graph-using-rest"></a>Llamar a Microsoft Graph con REST
Puede llamar a Microsoft Graph con REST. Reemplace la línea `exit($accessToken->getToken());` por el siguiente código. Escriba su dirección de correo electrónico en el marcador de posición marcado como **\<YOUR_EMAIL_ADDRESS\>**.

```php
$client = new \GuzzleHttp\Client();

$email = "{
    Message: {
    Subject: 'Sent using the Microsoft Graph REST API',
    Body: {
        ContentType: 'text',
        Content: 'This is the email body'
    },
    ToRecipients: [
        {
            EmailAddress: {
            Address: '<YOUR_EMAIL_ADDRESS>'
            }
        }
    ]
    }}";

$response = $client->request('POST', 'https://graph.microsoft.com/v1.0/me/sendmail', [
    'headers' => [
        'Authorization' => 'Bearer ' . $accessToken->getToken(),
        'Content-Type' => 'application/json;odata.metadata=minimal;odata.streaming=true'
    ],
    'body' => $email
]);
if($response.getStatusCode() === 201) {
    exit('Email sent, check your inbox');
} else {
    exit('There was an error sending the email. Status code: ' . $response.getStatusCode());
}
```

## <a name="run-the-app"></a>Ejecutar la aplicación
Ya puede probar la aplicación PHP.

1. En el shell escriba el siguiente comando:
    ```bash
    php artisan serve
    ```
    
2. Vaya a `http://localhost:8000` en el explorador web.
3. Elija **Iniciar sesión en Microsoft**.
4. Inicie sesión con su cuenta personal, profesional o educativa y conceda los permisos solicitados.

Compruebe la Bandeja de entrada de la dirección de correo electrónico que configuró en la sección [Llamar a Microsoft Graph con REST](#call-microsoft-graph-using-rest). Debe tener un correo electrónico de la cuenta que utilizó para iniciar sesión en la aplicación.

## <a name="next-steps"></a>Pasos siguientes
- Pruebe el [Probador de Microsoft Graph](https://graph.microsoft.io/graph-explorer).


## <a name="see-also"></a>Recursos adicionales
* [Protocolos de Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
* [Tokens de Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)
