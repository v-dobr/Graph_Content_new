## <a name="register-your-microsoft-graph-application-with-the-azure-ad-v2.0-endpoint"></a>Registrar su aplicación de Microsoft Graph con el punto de conexión v2.0 de Azure AD

Para usar el punto de conexión v2.0 de Azure AD, deberá registrar la aplicación en el [Portal de registro de aplicaciones de Microsoft](https://apps.dev.microsoft.com) (https://apps.dev.microsoft.com). El registro de la aplicación establece su identidad con el proveedor de autenticación y permite que pruebe esa identidad al enviar solicitudes de autenticación del usuario. El registro generará el identificador y el secreto de la aplicación que usará al configurar la aplicación para la autenticación.

> **Nota: **Este artículo trata sobre el registro de aplicaciones mediante el punto de conexión v2.0 de Azure AD. Para [registrar la aplicación con Azure AD](app_authentication_azure_ad.md), use [Azure Portal](https://aka.ms/aadapplist).
> 
> Además, tenga en cuenta que si ya ha registrado aplicaciones en el Portal de administración de Microsoft Azure, esas aplicaciones no aparecerán en el Portal de registro de aplicaciones. Administre esas aplicaciones en el Portal de administración de Azure. 

1. Inicie sesión en el [Portal de registro de aplicaciones de Microsoft](https://apps.dev.microsoft.com/) mediante su cuenta personal, profesional o educativa.

2. Seleccione **Agregar una aplicación**.

3. Escriba un nombre para la aplicación y seleccione **Crear aplicación**.

    Se muestra la página de registro, indicando las propiedades de la aplicación.

4. Copie el identificador de la aplicación. Se trata del identificador único para su aplicación.

    Deberá usar el identificador de la aplicación para configurar la aplicación.

5. En **Plataformas**, elija **Agregar plataforma** y seleccione la plataforma adecuada para su aplicación:
    
    En el caso de las aplicaciones cliente:
    1. Seleccione **Plataforma móvil**.

    2. Copie tanto el identificador de cliente (identificador de la aplicación) como los valores del URI de redireccionamiento al Portapapeles. Deberá escribir estos valores en la aplicación de ejemplo.

        El identificador de la aplicación es un identificador único para su aplicación. El URI de redireccionamiento es un URI único que se proporciona para cada aplicación con el fin de garantizar que los mensajes enviados a ese URI solo se envían a esa aplicación. 

    Para aplicaciones web:
    1. Seleccione **Web**.
    2. Si usa el tipo de concesión implícita o el flujo híbrido de OpenID Connect, asegúrese de que esté seleccionada la casilla Permitir flujo implícito. 
        
        La opción Permitir flujo implícito habilita el flujo híbrido de OpenID Connect. Durante la autenticación, esto permite que la aplicación reciba tanto la información de inicio de sesión (id_token) como los artefactos (en este caso, un código de autorización) que la aplicación usa para obtener un token de acceso.


    3. Especifique un URI de redireccionamiento.
        
        El URI de redireccionamiento es la ubicación de la aplicación a la que el punto de conexión v2.0 de Azure AD llama una vez que ha procesado la solicitud de autenticación.
    4. En **Secretos de aplicación**, seleccione **Generar nueva contraseña**. Copie el secreto de aplicación del cuadro de diálogo **Nueva contraseña generada**.
        
        Usará el secreto de aplicación para configurar la aplicación.
    
6. Elija **Guardar**.

## <a name="see-also"></a>Recursos adicionales

[Autenticación de aplicaciones con Microsoft Graph](auth_overview.md)
