# <a name="get-started-with-microsoft-graph-in-a-ruby-on-rails-app"></a>Introducción a Microsoft Graph en una aplicación de Ruby on Rails

En este artículo, se describen las tareas necesarias para obtener un token de acceso desde el punto de conexión v2.0 de Azure AD y llamar a Microsoft Graph. Le muestra los pasos para la compilación del [Ejemplo Connect de Microsoft Graph para Ruby on Rails](https://github.com/microsoftgraph/ruby-connect-rest-sample) y explica los conceptos principales que implementará para usar Microsoft Graph. El artículo también describe cómo obtener acceso a Microsoft Graph mediante llamadas de REST directas.

Para descargar una versión del ejemplo Connect que usa el punto de conexión de Azure AD, consulte el [Ejemplo Connect de Microsoft Graph para Ruby on Rails](https://github.com/microsoftgraph/ruby-connect-rest-sample/tree/last_v1_auth).

En la imagen siguiente, se muestra la aplicación que va a crear. 

![Captura de pantalla del ejemplo Microsoft Ruby on Rails Connect ](./images/Microsoft-Graph-Ruby-Connect-UI.png)

**¿No desea compilar una aplicación?** Use el [inicio rápido de Microsoft Graph](https://graph.microsoft.io/en-us/getting-started) para ponerlo todo en funcionamiento de manera rápida o descargue el [Ejemplo Connect de REST de Ruby](https://github.com/microsoftgraph/ruby-connect-rest-sample) en el que se basa este artículo.


## <a name="prerequisites"></a>Requisitos previos

Para comenzar, necesitará: 

- Ruby 2.1 para ejecutar el ejemplo en un servidor de desarrollo.
- Marco Rails (el ejemplo se probó con Rails 4.2).
- Administrador de dependencias Bundler.
- Interfaz de servidor web Rack para Ruby.
- Una [cuenta Microsoft](https://www.outlook.com/) o una [cuenta profesional o educativa](http://dev.office.com/devprogram)
- Proyecto inicial de conexión de Microsoft Graph para Ruby on Rails. Descargue el [Ejemplo Connect de Microsoft Graph de Ruby on Rails](https://github.com/microsoftgraph/ruby-connect-rest-sample). El proyecto inicial se encuentra en la carpeta _starter_.


## <a name="register-the-application"></a>Registrar la aplicación

Registre una aplicación en el Portal de registro de aplicaciones de Microsoft. Esta acción generará el ID y el secreto de aplicación que usará para configurar la aplicación para la autenticación.

1. Inicie sesión en el [Portal de registro de aplicaciones de Microsoft](https://apps.dev.microsoft.com/) mediante su cuenta personal, profesional o educativa.

2. Seleccione **Agregar una aplicación**.

3. Escriba un nombre para la aplicación y seleccione **Crear aplicación**.

    Se muestra la página de registro, indicando las propiedades de la aplicación.

4. Copie el identificador de la aplicación. Se trata del identificador único para su aplicación.

5. En **Secretos de aplicación**, seleccione **Generar nueva contraseña**. Copie el secreto de aplicación del cuadro de diálogo **Nueva contraseña generada**.

    Deberá usar el ID y el secreto de aplicación para configurar la aplicación.

6. En **Plataformas**, elija **Agregar plataforma** > **Web**.

7. Asegúrese de que la casilla **Permitir flujo implícito** está seleccionada y escriba *http://localhost:3000/auth/microsoft_v2_auth/callback* como URI de redireccionamiento.

    La opción Permitir flujo implícito habilita el flujo híbrido de OpenID Connect. Durante la autenticación, esto permite que la aplicación reciba tanto la información de inicio de sesión (id_token) como los artefactos (en este caso, un código de autorización) que la aplicación usa para obtener un token de acceso.

    El URI de redireccionamiento *http://localhost:3000/auth/microsoft_v2_auth/callback* es el valor con el que el middleware de OmniAuth se ha configurado para usar una vez que haya procesado la solicitud de autenticación.

8. Elija **Guardar**.

## <a name="configure-the-project"></a>Configurar el proyecto

1. Descargue o clone el [Ejemplo Connect de Microsoft Graph de Ruby on Rails](https://github.com/microsoftgraph/ruby-connect-rest-sample). Abra la carpeta _starter_ en el editor que prefiera.
1. Si todavía no tiene Bundler ni Rack, puede instalarlos con el siguiente comando.

    ```
    gem install bundler rack
    ```
2. En el archivo [config/environment.rb](config/environment.rb) realice lo siguiente:
    - Reemplace *ENTER_YOUR_CLIENT_ID* por el identificador de cliente de su aplicación registrada.
    - Reemplace *ENTER_YOUR_SECRET* por la clave de su aplicación registrada.

3. Instale la aplicación Rails y las dependencias con el siguiente comando.

    ```
    bundle install
    ```

## <a name="authenticate-the-user-and-get-an-access-token"></a>Autenticar al usuario y obtener un token de acceso

Esta aplicación usa el flujo de concesión de códigos de autorización con una identidad de usuario delegado. Para una aplicación web, el flujo requiere el identificador de la aplicación, el secreto y el URI de redireccionamiento de la aplicación registrada. 

El flujo de autenticación puede desglosarse en los siguientes pasos básicos:

1. Redirigir al usuario para la autenticación y el consentimiento
2. Obtener un código de autorización
3. Canjear el código de autorización por un token de acceso

>Para obtener más información acerca de este flujo de autenticación, consulte [Aplicación web a API web](https://azure.microsoft.com/en-us/documentation/articles/active-directory-authentication-scenarios/#web-application-to-web-api) e [Integrar la identidad de Microsoft y Microsoft Graph en una aplicación web mediante OpenID Connect](https://azure.microsoft.com/en-us/documentation/samples/active-directory-dotnet-webapp-openidconnect-v2/) en la documentación de Azure AD.

Usaremos una pila de tres partes del middleware [Rack](http://rack.github.io/) para habilitar la aplicación para que se autentique en Microsoft Graph:

- [OmniAuth](https://rubygems.org/gems/omniauth), un marco generalizado de Rack para la autenticación de varios proveedores.
- [Omniauth-oauth2](https://rubygems.org/gems/omniauth-oauth2), una estrategia abstracta de OAuth2 para OmniAuth. 
- omniauth-microsoft_v2_auth, una estrategia de OmniAuth que personaliza Omniauth-oauth2 específicamente para proporcionar autenticación en el punto de conexión v2.0 de Azure AD. Este proyecto se incluye en el ejemplo de código.

### <a name="specify-gem-dependencies-for-authentication"></a>Especificar dependencias de gemas para la autenticación

En Gemfile, quite el comentario de las siguientes gemas para agregarlas como dependencias.

    ```
    gem 'omniauth'
    gem 'omniauth-oauth2'
    gem 'omniauth-microsoft_v2_auth', path: './omniauth-microsoft_v2_auth'
    ```

Tenga en cuenta que `omniauth-microsoft_v2_auth` se incluye en el proyecto de la aplicación y se instalará desde la ruta de acceso especificada. 

### <a name="configure-the-authentication-middleware"></a>Configurar el middleware de autenticación

En `config/initializers/omniauth-microsoft_v2_auth.rb`, quite la marca de comentario de las siguientes líneas.

    ```
    Rails.application.config.middleware.use OmniAuth::Builder do
      provider :microsoft_v2_auth,
      ENV['CLIENT_ID'],
      ENV['CLIENT_SECRET'],
      :scope => ENV['SCOPE']
    end
    ```
Esta acción configura el middleware de OmniAuth, incluida la especificación del ID y el secreto de aplicación que se usarán, así como los ámbitos que se solicitarán para el usuario. Estos son los valores que especificó anteriormente en `config/environment.rb`.

### <a name="specify-routes-for-authentication"></a>Especificar rutas para la autenticación

Ahora, debemos especificar dos rutas necesarias para el flujo de autenticación. La primera ruta reenvía la solicitud de autenticación al middleware de OmniAuth. La segunda ruta especifica la ubicación de la aplicación a la que OmniAuth deberá redirigir una vez que se haya realizado la autenticación.

En `config/routes.rb`, quite la marca de comentario de la siguiente directiva de ruta.

    get '/login', to: 'pages#login'

Esto dirige las solicitudes de inicio de sesión al método `login` del controlador de páginas, que a su vez redirige la solicitud al middleware omniauth-microsoft_v2_auth.

    def login
        redirect_to '/auth/microsoft_v2_auth'
    end

A continuación, deberemos especificar la ubicación de la aplicación a la que OmniAuth debería redirigir una vez que se haya realizado la autenticación. Quite la marca de comentario de la siguiente ruta.

    match '/auth/:provider/callback', to: 'pages#callback', via: [:get, :post]

Cuando OmniAuth termine de autenticar el usuario, llamará a la URL de redireccionamiento que se haya especificado en el registro de aplicaciones; en este caso, *http://localhost:3000/auth/microsoft_v2_auth/callback*. El patrón de ruta anterior coincide con esa URL y, en consecuencia, redirige la solicitud al método `callback` del controlador de páginas.

### <a name="get-an-access-token"></a>Obtener un token de acceso

A continuación, agregaremos el código que realmente inicia el proceso de autenticación y recupera el token de acceso una vez que el usuario ha iniciado sesión correctamente.

Eche un vistazo a `app/views/pages/index.html.erb`, la vista de la raíz del sitio. La vista incluye un solo botón, que permite a los usuarios iniciar sesión.

    <button class="ms-Button" onclick="window.location.href = '/login'">
        <span class="ms-Button-label"><%= t('connect_button') %></span>
    </button>

Como se muestra anteriormente, el método de inicio de sesión redirige al middleware de OmniAuth, que se ha configurado con el ID y el secreto de aplicación, así como con los ámbitos que se van a solicitar para el usuario. Una vez que el usuario se autentique correctamente, OmniAuth devolverá un hash con el token de acceso y otra información de usuario a la aplicación.

Ahora, agregaremos el código para realizar la devolución de llamada de OmniAuth y recuperar la información de ese hash. 

En `app/controllers/pages_controller.rb`, reemplace el método vacío `callback` por el código siguiente.

    ```
    def callback
        # Access the authentication hash for omniauth
        # and extract the auth token, user name, and email
        data = request.env['omniauth.auth']
    
        @email = data[:extra][:raw_info][:userPrincipalName]
        @name = data[:extra][:raw_info][:displayName]

        # Associate token/user values to the session
        session[:access_token] = data['credentials']['token']
        session[:name] = @name
        session[:email] = @email
        
        # Debug logging
        logger.info "Name: #{@name}"
        logger.info "Email: #{@email}"
        logger.info "[callback] - Access token: #{session[:access_token]}"
    end

    ```

Este método recupera el hash de autenticación y, a continuación, almacena el token de acceso, el nombre de usuario y el correo electrónico en la sesión actual.

> **Nota:** La autenticación simple y el uso de tokens en este proyecto se presentan solo con fines ilustrativos. En una aplicación de producción, deberá crear una forma más segura de usar la autenticación, incluidos el uso seguro de tokens y la actualización de tokens.

## <a name="call-microsoft-graph"></a>Llamar a Microsoft Graph

Ahora, ya puede agregar el código para llamar a Microsoft Graph. 

La vista que representa el método `callback` (`app/views/pages/callback.html.erb`) incluye un formulario simple con un solo botón. El formulario se publica en `send_mail` e incluye un solo parámetro: la dirección de correo electrónico del destinatario.
    
    ``` 
    <form action="../../send_mail" method="post">
      <div class="ms-Grid-col ms-u-mdPush1 ms-u-md9 ms-u-lgPush1 ms-u-lg6">
        ...
            <div class="ms-TextField">
               <input class="ms-TextField-field" name="specified_email" value="<%= @email %>">
            </div>
            <button class="ms-Button">
            <span class="ms-Button-label"><i class="ms-Icon ms-Icon--mail" aria-hidden="true"></i><%= t('send_mail_button') %></span>
            </button> 
        ...
    ```

En `app/controllers/pages_controller.rb`, reemplace el método vacío `send_mail` por el código siguiente.

    ```
    def send_mail
        logger.debug "[send_mail] - Access token: #{session[:access_token]}"
        
        # Used in the template
        @name = session[:name]
        @email = params[:specified_email]
        @recipient = params[:specified_email]
        @mail_sent = false
        
        send_mail_endpoint = URI("#{GRAPH_RESOURCE}#{SENDMAIL_ENDPOINT}")
        content_type = CONTENT_TYPE
        http = Net::HTTP.new(send_mail_endpoint.host, send_mail_endpoint.port)
        http.use_ssl = true
        
        # If you want to use a sniffer tool, like Fiddler, to see the request
        # you might need to add this line to tell the engine not to verify the
        # certificate or you might see a "certificate verify failed" error
        # http.verify_mode = OpenSSL::SSL::VERIFY_NONE
        
        email_body = File.read('app/assets/MailTemplate.html')
        email_body.sub! '{given_name}', @name
        email_subject = t('email_subject')
        
        logger.debug email_body
    
        email_message = "{
            Message: {
            Subject: '#{email_subject}',
            Body: {
                ContentType: 'HTML',
                Content: '#{email_body}'
            },
            ToRecipients: [
                {
                    EmailAddress: {
                        Address: '#{@recipient}'
                    }
                }
            ]
            },
            SaveToSentItems: true
            }"
            
        response = http.post(
            SENDMAIL_ENDPOINT,
            email_message,
            'Authorization' => "Bearer #{session[:access_token]}",
            'Content-Type' => content_type
        )
        
        logger.debug "Code: #{response.code}"
        logger.debug "Message: #{response.message}"
        
        # The send mail endpoint returns a 202 - Accepted code on success
        if response.code == '202'
            @mail_sent = true
        else
            @mail_sent = false
            flash[:httpError] = "#{response.code} - #{response.message}"
        end
        
        render 'callback'
    end
    ```

Este código crea la solicitud HTTP, aplica formato al correo electrónico y, a continuación, llama a Microsoft Graph para enviarlo.

Para crear el correo electrónico, el código extrae el nombre de usuario del token de sesión y la dirección de correo electrónico del destinatario de los parámetros que se han pasado desde el formulario. A continuación, el código lee el cuerpo del correo electrónico desde una plantilla que se incluye en el proyecto, interpola el nombre de usuario y la dirección de correo electrónico y asocia el texto del correo electrónico como cuerpo de la solicitud HTTP.

Para enviar el correo electrónico, el código crea la solicitud HTTP, adjunta el token de acceso como encabezado de autorización y, a continuación, publica la solicitud en el punto de conexión de envío de correo electrónico.

Por último, el código usa el código de respuesta HTTP que se ha devuelto para notificar al usuario si el correo electrónico se envió o no correctamente.

## <a name="run-the-app"></a>Ejecutar la aplicación

1. Instale la aplicación Rails y las dependencias con el siguiente comando.

    ```
    bundle install
    ```
2. Para iniciar la aplicación Rails, escriba el siguiente comando.

    ```
    rackup -p 3000
    ```
3. Vaya a `http://localhost:3000` en el explorador web.

## <a name="see-also"></a>Recursos adicionales
- Pruebe la API de REST mediante el [Probador de Graph](https://graph.microsoft.io/graph-explorer).
- Explore nuestros otros [ejemplos de Microsoft Graph](https://github.com/microsoftgraph) en GitHub.


