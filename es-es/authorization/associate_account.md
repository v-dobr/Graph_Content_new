# <a name="associate-your-office-365-account-with-azure-ad-to-create-and-manage-apps"></a>Asociar la cuenta de Office 365 a Azure AD para crear y administrar aplicaciones

Para autenticar las aplicaciones mediante Microsoft Azure Active Directory (Azure AD), debe registrarlas en Azure AD. Aquí es donde se almacena la información de las aplicaciones y de la cuenta de usuario de Office 365. Para administrar Azure AD a través del Portal de administración de Azure, necesita una suscripción a Microsoft Azure. Puede usar el Portal de administración de Microsoft Azure para administrar usuarios, roles y aplicaciones. 

En este artículo, se le muestra cómo asociar su cuenta de Office 365 a Azure AD para crear y administrar aplicaciones.

 >**Nota:** En este artículo, se usa Azure AD como proveedor de autenticación de su aplicación. Si usa el punto de conexión v2.0 de Azure AD, no será necesario realizar este paso. Para obtener más información, consulte [Autenticación de aplicaciones con Microsoft Graph](../auth_overview.md).

## <a name="prerequisites"></a>Requisitos previos

**Cuenta de Office 365 para empresas**

Si no tiene una cuenta de Office 365 para empresas, realice una de las siguientes acciones:

- Regístrese en uno de los [planes de Office 365 para empresas](http://products.office.com/en-us/business/compare-office-365-for-business-plans) enumerados anteriormente, o
- [Únase al programa Office 365 Developer y consiga una suscripción gratuita de 1 año a Office 365](https://aka.ms/devprogramsignup).

**Suscripción a Microsoft Azure** 

- Si tiene una suscripción de Microsoft Azure, puede asociarla con su suscripción de Office 365 para empresas. 

- De lo contrario, deberá crear una suscripción de Azure y asociarla con la cuenta de Office 365 para así poder registrar y administrar aplicaciones.


<!---<a name="bk_AssociateExistingAzureSubscription"> </a>-->

## <a name="to-associate-an-existing-azure-subscription-with-your-office-365-account"></a>Para asociar una suscripción de Azure existente con su cuenta de Office 365


1. Inicie sesión en el [Portal de administración de Microsoft Azure](https://manage.windowsazure.com) con sus credenciales de Azure existentes (por ejemplo, un identificador de Microsoft como user@live.com).
        
2. Seleccione el nodo **Active Directory**. A continuación, seleccione la pestaña **Directorio** y, en la parte inferior de la pantalla, seleccione **Nuevo**. 
     
4. En el menú **Nuevo**, seleccione **Active Directory**  >  **Directorio**  >  **Creación personalizada**.
    
5. En **Agregar directorio**, en el cuadro desplegable **Directorio**, seleccione **Usar directorio existente**. Active **Todo listo para cerrar sesión** y seleccione la casilla en la esquina inferior derecha. 
    
    Regresará al Portal de administración de Azure.
        
3. Inicie sesión con su cuenta de Office 365. 
    
    Se le preguntará si desea usar su directorio con Azure. 
    
    >**Importante:** Para poder asociar su cuenta de Office 365 a Azure AD, necesitará una cuenta de Office 365 para empresas con privilegios de administrador global. 
    
        
4. Seleccione **Continuar** y luego **Cerrar sesión ahora**.
        
5. Cierre el explorador y vuelva a abrir el [portal](https://manage.windowsazure.com). Si no lo hace, obtendrá un mensaje de acceso denegado.
    
        
6. Vuelva a iniciar sesión con sus credenciales de Azure existentes (por ejemplo, un identificador de Microsoft como user@live.com). Vaya al nodo de **Active Directory**; su cuenta de Office 365 debería aparecer ahora en **Directorio**.
    

<!--<a name="bk_AssociateNewAzureSubscription"> </a>-->

## <a name="to-create-a-new-azure-subscription-and-associate-it-with-your-office-365-account"></a>Cómo crear una nueva suscripción a Azure y asociarla a su cuenta de Office 365


1. Inicie sesión en Office 365. Desde la página **Inicio**, seleccione el icono **Administrador** para abrir el Centro de administración de Office 365.
2. En el lado izquierdo de la página de menú, desplácese hacia abajo hasta **Administrador** y seleccione **Azure AD**.

    >**Importante:** Para abrir el Centro de administración de Office 365 y obtener acceso a Azure AD, necesitará una cuenta de Office 365 para empresas con privilegios de administrador global. 
    
3. Cree una suscripción.
        
    Si está usando una versión de prueba de Office 365, verá un mensaje en el que se le indica que Azure AD está limitado a clientes con servicios de pago. Puede crear una suscripción de prueba de 30 días gratuita, pero deberá realizar algunos pasos adicionales:
    
    1. Seleccione su país o región y elija **Suscripción a Azure**.
    2. Escriba su información personal. Con fines de verificación, escriba un número de teléfono en el que podamos contactar con usted y especifique si prefiere recibir un mensaje de texto o una llamada.
    3. Una vez que reciba el código de verificación, escríbalo y elija **Comprobar código**.
    4. Escriba la información de pago, revise el contrato y seleccione **Suscribirse**.
        
        No se efectuará ningún cargo en su tarjeta de crédito.
        
        No cierre ni actualice el explorador mientras se crea la suscripción a Azure.
            
4. Una vez creada la suscripción a Azure, elija **Portal**.
        
5. Se iniciará el paseo por Azure. Puede verlo o elegir **X** para cerrarlo.
        
    Ahora, debería ver todos los elementos de su suscripción a Azure. El directorio que se muestra incluye el nombre de su espacio empresarial de Office 365.
    
## <a name="see-also"></a>Recursos adicionales
- [Conceptos básicos del registro de aplicaciones en Azure AD](https://azure.microsoft.com/en-us/documentation/articles/active-directory-authentication-scenarios/#basics-of-registering-an-application-in-azure-ad)
- [Agregar, actualizar o quitar una aplicación en Azure AD](https://azure.microsoft.com/en-us/documentation/articles/active-directory-integrating-applications/)