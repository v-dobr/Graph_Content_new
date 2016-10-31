# <a name="get-started-with-microsoft-graph-in-an-ios-app"></a>Introducción a Microsoft Graph en una aplicación de iOS

> **¿Desea compilar aplicaciones para clientes empresariales?** Es posible que la aplicación no funcione si su cliente empresarial activa características de seguridad de movilidad empresarial como el <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-conditional-access-device-policies/" target="_newtab">acceso condicional al dispositivo</a>. En casos así, es posible que no tenga constancia de esta activación y que sus clientes obtengan errores. 

> Para admitir **todos los clientes empresariales** en **todos los escenarios de empresa**, deberá usar el punto de conexión de Azure AD y administrar las aplicaciones mediante el [Portal de administración de Azure](https://aka.ms/aadapplist). Para obtener más información, consulte [Decidir entre los puntos de conexión de Azure AD y Azure AD v2.0 ](../authorization/auth_overview.md#deciding-between-azure-ad-and-the-v2-authentication-endpoint).

En este artículo se describen las tareas necesarias para obtener un token de acceso desde el punto de conexión de [Azure AD v2.0](https://graph.microsoft.io/en-us/docs/authorization/converged_auth) y llamar a Microsoft Graph. Le guiará por el código del [Ejemplo Connect de Office 365 para iOS (SDK)](https://github.com/microsoftgraph/ios-objectivec-connect-sample) para explicar los conceptos principales que se deben implementar en una aplicación que use Microsoft Graph. Describe cómo obtener acceso a Microsoft Graph mediante el [SDK de Microsoft Graph para iOS](https://github.com/microsoftgraph/msgraph-sdk-ios).

Puede descargar la versión de la aplicación que creará desde este repositorio de GitHub:

* [Ejemplo Connect de Office 365 para iOS con el SDK de Microsoft Graph](https://github.com/microsoftgraph/ios-objectivec-connect-sample)

En la imagen siguiente, se muestra la aplicación que va a crear.

![El tutorial de conexión de ejemplo, muestra cómo conectarse y enviar un correo en la aplicación.](./images/iOSConnectWalkthrough.png)


El flujo de trabajo será conectarse o autenticarse en Microsoft Graph, iniciar sesión con su cuenta personal o profesional y, por último, enviar un correo a un destinatario.

**¿No desea compilar una aplicación?** Use el [inicio rápido de Microsoft Graph](https://graph.microsoft.io/en-us/getting-started) para ponerlo todo en funcionamiento de manera rápida.

## <a name="prerequisites"></a>Requisitos previos

Para comenzar, necesitará: 

* [Xcode](https://developer.apple.com/xcode/downloads/) de Apple
* La instalación de [CocoaPods](https://guides.cocoapods.org/using/using-cocoapods.html) como administrador de dependencias.
* Una [cuenta Microsoft](https://www.outlook.com/) o una [cuenta profesional o educativa](http://dev.office.com/devprogram)
* [Proyecto inicial de Microsoft Graph para iOS](https://github.com/microsoftgraph/ios-objectivec-connect-sample). Esta plantilla contiene clases a las que agregará código. Para obtener este proyecto, clone o descargue el proyecto de ejemplo desde esta ubicación y trabajará con el área de trabajo de la carpeta **starter-project** (**O365-iOS-Microsoft-Graph-SDK.xcworkspace**).

## <a name="register-the-app"></a>Registrar la aplicación
 
1. Inicie sesión en el [Portal de registro de aplicaciones](https://apps.dev.microsoft.com/) mediante su cuenta personal, profesional o educativa.
2. Seleccione **Agregar una aplicación**.
3. Escriba un nombre para la aplicación y seleccione **Crear aplicación**.
    
    Se muestra la página de registro, indicando las propiedades de la aplicación.
 
4. En **Plataformas**, seleccione **Agregar plataforma**.
5. Seleccione **Plataforma móvil**.
6. Copie el identificador de cliente en el Portapapeles. Deberá escribir este valor en la aplicación de ejemplo.

    El id. de la aplicación es un identificador único para su aplicación. 

7. Seleccione **Guardar**.

## <a name="importing-the-project-dependencies"></a>Importar las dependencias del proyecto

1. Clone este repositorio ([Ejemplo Connect de Office 365 para iOS con el SDK de Microsoft Graph](https://github.com/microsoftgraph/ios-objectivec-connect-sample)). **Recuerde que usará el ejemplo de la carpeta starter-project y no el ejemplo de la raíz del proyecto.**
2. Use CocoaPods para importar el SDK de Microsoft Graph y las dependencias de autenticación. Esta aplicación de ejemplo ya contiene un podfile que recibirá los pods en el proyecto. Vaya a la carpeta **starter-project** de la aplicación **Terminal** y, desde **Terminal**, ejecute lo siguiente:

        pod install

   Recibirá la confirmación de la importación de los pods al proyecto. Para obtener más información, consulte [CocoaPods](https://guides.cocoapods.org/using/using-cocoapods.html).


## <a name="enable-keychain-sharing"></a>Habilitar el uso compartido de cadenas de claves
 
Para Xcode 8, deberá agregar el grupo de cadenas de claves o la aplicación no podrá obtener acceso a la cadena de claves. Para agregar el grupo de cadenas de claves:
 
1. Seleccione el proyecto en el panel de administración de proyectos de Xcode. (⌘ + 1).
 
2. Seleccione **O365-iOS-Microsoft-Graph-SDK**.
 
3. En la pestaña Funcionalidad, habilite **Uso compartido de cadenas de claves**.
 
4. Agregue **com.microsoft.O365-iOS-Microsoft-Graph-SDK** a los grupos de cadenas de claves.
 

## <a name="authenticating-with-microsoft-graph"></a>Autenticación con Microsoft Graph

Para volver a visitar el flujo de trabajo de la interfaz de usuario, la aplicación va a solicitar al usuario que se autentique y, a continuación, este podrá enviar un correo al usuario especificado. Para realizar solicitudes en el servicio Microsoft Graph, se debe proporcionar un proveedor de autenticación que sea capaz de autenticar solicitudes HTTPS con un token de portador OAuth 2.0 adecuado. En el proyecto de ejemplo, hay una clase de autenticación que ya tiene código auxiliar que se llama **AuthenticationProvider.m.** Agregaremos una función para solicitar (y adquirir) un token de acceso para llamar a la API de Microsoft Graph. 

1. Abra el área de trabajo del proyecto Xcode (**O365-iOS-Microsoft-Graph-SDK.xcworkspace**) de la carpeta **starter-project** y vaya a la carpeta **Authentication** para abrir el archivo **AuthenticationProvider.m.** Agregue el código siguiente a esa clase.

        -(void) connectToGraphWithClientId:(NSString *)clientId scopes:(NSArray *)scopes completion:(void (^)   (NSError *))completion{
            [NXOAuth2AuthenticationProvider setClientId:kClientId
                                              scopes:scopes];
    
    
            /**
            Obtains access token by performing login with UI, where viewController specifies the parent view controller.
            @param viewController The view controller to present the UI on.
             @param completionHandler The completion handler to be called when the authentication has completed.
            error should be non nil if there was no error, and should contain any error(s) that occurred.
             */

                if ([[NXOAuth2AuthenticationProvider sharedAuthProvider] loginSilent]) {
                completion(nil);
                }
                else {
                    [[NXOAuth2AuthenticationProvider sharedAuthProvider] loginWithViewController:nil completion:^(NSError *error) {
                    if (!error) {
                    NSLog(@"Authentication successful.");
                    completion(nil);
                    }
                 else {
                     NSLog(@"Authentication failed - %@", error.localizedDescription);
                    completion(error);
                    }
                }];
            }
    
        }

2. A continuación, agregue el método al archivo de encabezado. Abra el archivo **AuthenticationProvider.h** y agregue el código siguiente a esta clase.

        -(void) connectToGraphWithClientId:(NSString *)clientId
                            scopes:(NSArray *)scopes
                        completion:(void (^)(NSError *error))completion;



2. Por último, llamaremos a este método desde **ConnectViewController.m**. Este controlador es la vista predeterminada que carga la aplicación y hay un único botón denominado **Conectar** que el usuario pulsará para iniciar el proceso de autenticación. Este método toma dos parámetros, el **Id. de cliente** y los **ámbitos**, que trataremos con más detalle a continuación. Agregue la siguiente acción a **ConnectViewController.m**.

        - (IBAction)connectTapped:(id)sender {
            [self showLoadingUI:YES];   
            NSArray *scopes = [kScopes componentsSeparatedByString:@","];
            [self.authProvider connectToGraphWithClientId:kClientId scopes:scopes completion:^(NSError *error) {
                if (!error) {
                    [self performSegueWithIdentifier:@"showSendMail" sender:nil];
                    [self showLoadingUI:NO];
                    NSLog(@"Authentication successful.");
                    }
                else{
                    NSLog(NSLocalizedString(@"CHECK_LOG_ERROR", error.localizedDescription));
                    [self showLoadingUI:NO];
                    };
                }];
        }

## <a name="send-an-email-with-microsoft-graph"></a>Enviar un correo electrónico con Microsoft Graph

Después de configurar el proyecto para que pueda autenticar, las siguientes tareas corresponderán al envío de un correo a un usuario mediante la API de Microsoft Graph. De forma predeterminada, el usuario que inició sesión será el destinatario, pero tiene la posibilidad de cambiarlo a cualquier otro destinatario. El código con el que trabajaremos aquí está ubicado en la carpeta **Controllers** y en la clase **SendMailViewController.m.** Verá que aquí hay otro código representado para la interfaz de usuario y un método auxiliar para recuperar la información del perfil del usuario desde el servicio Microsoft Graph. Nos concentraremos en los métodos para crear un mensaje de correo y enviarlo.

1. Abra el archivo **SendMailViewController.m.** de la carpeta Controllers y agregue el siguiente método auxiliar a la clase:

        // Create a sample test message to send to specified user account
        -(MSGraphMessage*) getSampleMessage{
            MSGraphMessage *message = [[MSGraphMessage alloc]init];
            MSGraphRecipient *toRecipient = [[MSGraphRecipient alloc]init];
            MSGraphEmailAddress *email = [[MSGraphEmailAddress alloc]init];
    
            email.address = self.emailAddress;
            toRecipient.emailAddress = email;
    
            NSMutableArray *toRecipients = [[NSMutableArray alloc]init];
            [toRecipients addObject:toRecipient];
    
            message.subject = NSLocalizedString(@"MAIL_SUBJECT", comment: "");
    
            MSGraphItemBody *emailBody = [[MSGraphItemBody alloc]init];
            NSString *htmlContentPath = [[NSBundle mainBundle] pathForResource:@"EmailBody" ofType:@"html"];
            NSString *htmlContentString = [NSString stringWithContentsOfFile:htmlContentPath encoding:NSUTF8StringEncoding error:nil];
    
            emailBody.content = htmlContentString;
            emailBody.contentType = [MSGraphBodyType html];
            message.body = emailBody;
    
            message.toRecipients = toRecipients;
    
            return message;
    
        }


2. Abra el archivo **SendMailViewController.m.** Agregue a la clase el siguiente método para enviar correo.  

        //Send mail to the specified user in the email text field
        -(void) sendMail {   
            MSGraphMessage *message = [self getSampleMessage];
            MSGraphUserSendMailRequestBuilder *requestBuilder = [[self.graphClient me]sendMailWithMessage:message saveToSentItems:true];
            NSLog(@"%@", requestBuilder);
            MSGraphUserSendMailRequest *mailRequest = [requestBuilder request];
            [mailRequest executeWithCompletion:^(NSDictionary *response, NSError *error) {
                if(!error){
                    NSLog(@"response %@", response);
                    NSLog(NSLocalizedString(@"ERROR", ""), error.localizedDescription);
            
                    dispatch_async(dispatch_get_main_queue(), ^{
                        self.statusTextView.text = NSLocalizedString(@"SEND_SUCCESS", comment: "");
                });
            }
            else {
                NSLog(NSLocalizedString(@"ERROR", ""), error.localizedDescription);
                self.statusTextView.text = NSLocalizedString(@"SEND_FAILURE", comment: "");
                }
            }];
    
        }

Así, **getSampleMessage** creará un borrador HTML de ejemplo para usar con fines de demostración. A continuación, el siguiente método, **sendMail**, toma ese mensaje y ejecuta la solicitud de enviarlo. Una vez más, el destinatario predeterminado es el usuario que inició sesión.


## <a name="run-the-app"></a>Ejecutar la aplicación
1. Antes de ejecutar el ejemplo, deberá proporcionar el identificador de cliente que recibió del proceso de registro en la sección **Registrar la aplicación.** Abra el archivo **AuthenticationConstants.m** de la carpeta **Application**. Verá que el ClientID del proceso de registro se puede agregar a la parte superior del archivo:  

        // You will set your application's clientId
        NSString * const kClientId    = @"ENTER_CLIENT_ID_HERE";
        NSString * const kScopes = @"https://graph.microsoft.com/Mail.Send, https://graph.microsoft.com/User.Read, offline_access";
Nota: Observará que se han configurado los siguientes ámbitos de permiso para este proyecto: **"https://graph.microsoft.com/Mail.Send", "https://graph.microsoft.com/User.Read", "offline_access"**. Las llamadas de servicio usadas en este proyecto, el envío de un correo a su cuenta de correo y la recuperación de parte de la información de perfil (nombre para mostrar, dirección de correo electrónico) requieren estos permisos para que la aplicación se ejecute correctamente.

2. Ejecute el ejemplo, pulse **Conectar**, inicie sesión con su cuenta personal, profesional o educativa y conceda los permisos solicitados.

3. Elija el botón **Enviar correo electrónico**. Cuando se envíe el correo, se mostrará un mensaje de operación correcta debajo del botón.

## <a name="next-steps"></a>Pasos siguientes
- Pruebe la API de REST mediante el [Probador de Graph](https://graph.microsoft.io/graph-explorer).
- Busque ejemplos de operaciones comunes tanto para REST como para SDK en el [Ejemplo de fragmentos de código de Objective C para Microsoft Graph para iOS](https://github.com/microsoftgraph/ios-objectiveC-snippets-sample). 

## <a name="see-also"></a>Recursos adicionales
- [SDK de Microsoft Graph para iOS](https://github.com/microsoftgraph/msgraph-sdk-ios)
- [Protocolos de Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
- [Tokens de Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)
