# <a name="use-rest-apis-to-access-mailboxes-in-exchange-hybrid-deployments-(preview)"></a>Usar las API de REST para acceder a los buzones en las implementaciones híbridas de Exchange (versión preliminar)

Microsoft Graph siempre ha proporcionado acceso a buzones de cliente en la nube en Exchange Online como parte de Office 365. La actualización acumulativa 3 (CU3) de Exchange 2016, que se publicó en setiembre de 2016 para los servidores locales de Exchange, permite la integración de la API de REST con Office 365. Si su aplicación usa la versión 1.0 de la API de [Correo](http://graph.microsoft.io/en-us/docs/api-reference/v1.0/resources/message), [Calendario](http://graph.microsoft.io/en-us/docs/api-reference/v1.0/resources/calendar), o [Contactos](http://graph.microsoft.io/en-us/docs/api-reference/v1.0/resources/contact), podrá disfrutar de una experiencia de autenticación y de la aplicación sin interrupciones en implementaciones _híbridas_, independientemente de si el buzón es local o está en la nube, siempre que dicha implementación cumpla con los [](#requirements-for-rest-api-to-work-in-hybrid-deployments) especificados. 


Cuando Microsoft Graph identifica que la llamada de una API de REST está intentando obtener acceso a un buzón local de una implementación híbrida, pasa la solicitud de REST a un extremo de REST local que procede a procesar la solicitud. Este descubrimiento permite acceder a la API de REST.

>**Nota:** La capacidad para usar estas API de REST en implementaciones híbridas actualmente se encuentra en versión preliminar.

>Solo la versión 1.0 de la API de Correo, Calendario y Contactos está disponible para los buzones de implementaciones híbridas. Otros conjuntos de versiones 1.0 de API, como la API de [Grupos](http://graph.microsoft.io/en-us/docs/api-reference/v1.0/resources/group) u otras versiones de las API, no están disponibles. Si intenta usar una API que no forma parte del conjunto admitido en una implementación híbrida, obtendrá el siguiente mensaje de error:

>"Las API de REST para este buzón están actualmente en versión preliminar. Encontrará más información acerca de las API de REST en versión preliminar en https://dev.outlook.com".

## <a name="requirements-for-the-rest-api-to-work-in-hybrid-deployments"></a>Requisitos para que la API de REST funcione en implementaciones híbridas

Microsoft Graph proporciona amplitud (cuenta con una amplia compatibilidad de estándares, como JSON, OAUTH y ODATA, que se conectan desde las plataformas más populares) y flexibilidad (incluye permisos con ámbitos específicos y precisos para obtener acceso a los datos del usuario). Si su organización está interesada en permitir el desarrollo de la aplicación de Microsoft Graph y está actualmente en una implementación híbrida o barajando la posibilidad de estarlo, tenga en cuenta los requisitos de implementación siguientes:

- Requisitos de buzón

  - Todos los buzones locales que usarán las API de REST deben estar ubicados en las bases de datos que se encuentran en los servidores de la actualización acumulativa 3 de Exchange 2016. 

- Requisitos previos de infraestructura

  - Todos los servidores de Exchange 2016 se deben actualizar a la actualización acumulativa 3 o a una posterior.  
  - La versión de Active Directory local debe sincronizarse con Azure Active Directory.
  - Los servidores de Exchange 2013 que coexistan en la misma matriz de carga equilibrada con servidores de Exchange 2016 deben quitarse de la matriz.

- Requisitos de red

  - Desde una perspectiva de DNS, el espacio de nombres Detección automática y el espacio de nombres del cliente local deben tener registros DNS de Internet. 
  - Si tiene un firewall o una puerta de enlace de aplicaciones que inspecciona y restringe el acceso, actualice la configuración apropiada para permitir la detección y el acceso.


Los administradores de TI pueden encontrar más información en los siguientes recursos:

- 
  [Implementaciones híbridas de Exchange Server](https://technet.microsoft.com/en-us/library/jj200581(v=exchg.150).aspx)
- [Lanzamiento de la actualización acumulativa de septiembre de 2016](https://blogs.technet.microsoft.com/exchange/2016/09/20/released-september-2016-quarterly-exchange-updates/) 
- [Requisitos de la arquitectura local de la API de REST](https://blogs.technet.microsoft.com/exchange/2016/09/26/on-premises-architectural-requirements-for-the-rest-api/)
