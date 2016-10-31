# <a name="authenticate-microsoft-graph-apps-with-the-azure-ad-v2.0-endpoint"></a>Autenticar aplicaciones de Microsoft Graph con el punto de conexión v2.0 de Azure AD

> **¿Desea compilar aplicaciones para clientes empresariales?** Es posible que la aplicación no funcione si su cliente empresarial activa características de seguridad de movilidad empresarial como el <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-conditional-access-device-policies/" target="_newtab">acceso condicional al dispositivo</a>.  

> Para admitir **todos los clientes empresariales** en **todos los escenarios de empresa**, deberá usar el punto de conexión de Azure AD y administrar las aplicaciones mediante el [Portal de administración de Azure](https://aka.ms/aadapplist). Para obtener más información, consulte [Decidir entre los puntos de conexión de Azure AD y Azure AD v2.0 ](auth_overview.md#deciding-between-azure-ad-and-the-v2-authentication-endpoint).


Use el punto de conexión v2.0 de Azure AD para crear aplicaciones que acepten tanto identidades profesionales y educativas (Azure Active Directory) como personales (cuenta Microsoft).

Antes, si deseaba desarrollar una aplicación para que admitiera tanto las cuentas Microsoft como las de Azure Active Directory, necesitaba integrarlas en dos sistemas completamente independientes. Mediante el punto de conexión v2.0 de Azure AD, ya puede admitir ambos tipos de cuentas con una integración única (un proceso simple para llegar a una audiencia que abarca millones de usuarios con cuentas personales y profesionales o educativas).  

Una vez que hayan integrado sus aplicaciones con el punto de conexión v2.0 de Azure AD, podrán obtener acceso de forma instantánea a los puntos de conexión disponibles de Microsoft Graph tanto para las cuentas personales como para las profesionales o educativas. Por ejemplo: 

| Datos              | Punto de conexión                                       |
|:------------------|:-----------------------------------------------|
| Perfil de usuario      | `https://graph.microsoft.com/v1.0/me`          |
| Correo de Outlook      | `https://graph.microsoft.com/v1.0/me/messages` |
| Contactos de Outlook  | `https://graph.microsoft.com/v1.0/me/contacts` |
| Calendarios de Outlook | `https://graph.microsoft.com/v1.0/me/events`   |
| OneDrive          | `https://graph.microsoft.com/v1.0/me/drive`    |

 >**Nota:** Algunos puntos de conexión de Microsoft Graph, como los grupos y las tareas, no se aplican a las cuentas personales.  

## <a name="microsoft-graph-authentication-scopes"></a>Ámbitos de autenticación de Microsoft Graph

El punto de conexión v2.0 de Azure AD admite todos los ámbitos de permisos que se enumeran en [Ámbitos de permisos de Microsoft Graph](permission_scopes.md). 

Para obtener más información acerca del uso de ámbitos con el punto de conexión v2.0 de Azure AD y de cómo se diferencia del uso de recursos en Azure AD, consulte <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/#scopes-not-resources" target="_newtab">Ámbitos, no recursos</a>.

## <a name="see-it-in-action"></a>Ver el funcionamiento

Los [ejemplos de conexión del repositorio de Microsoft Graph](https://github.com/microsoftgraph?utf8=%E2%9C%93&query=connect) proporcionan casos sencillos de cómo autenticar los usuarios y conectarse a Microsoft Graph en una amplia gama de plataformas.

Además, la sección [Introducción](http://graph.microsoft.io/en-us/docs/platform/get-started) contiene artículos que describen cómo crear estas aplicaciones de ejemplo, incluidas las bibliotecas de autenticación que se usan en cada plataforma.

## <a name="see-also"></a>Recursos adicionales

- [Registrar una aplicación con el punto de conexión v2.0 de Azure AD](auth_register_app_v2.md)
- [Autenticación de aplicaciones con Microsoft Graph](auth_overview.md)
- <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare" target="_newtab">Novedades acerca del modelo v2.0 de Azure AD</a>
- <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/" target="_newtab">¿Debería usar el punto de conexión v2.0 de Azure AD?</a>
- <a href="https://azure.microsoft.com/en-us/documentation/articles/?product=active-directory&term=azure+ad+v2.0" target="_newtab">Documentación acerca del punto de conexión v2.0 de Azure AD en Azure.com</a>
- <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-app-registration/#build-a-quick-start-app" target="_newtab">Inicios rápidos del código v2.0 de Azure AD en Azure.com</a>

