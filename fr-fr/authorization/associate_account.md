# <a name="associate-your-office-365-account-with-azure-ad-to-create-and-manage-apps"></a>Association de votre compte Office 365 à Azure AD pour créer et gérer des applications

Pour authentifier vos applications en utilisant Microsoft Azure Active Directory (Azure AD), vous devez les inscrire dans Azure AD. C’est l’endroit où les informations de compte et d’application utilisateur Office 365 sont stockées. Pour gérer Azure AD via le portail de gestion Azure, vous devez disposer d’un abonnement Microsoft Azure. Vous pouvez utiliser le portail de gestion dans Microsoft Azure pour gérer les utilisateurs, les rôles et les applications. 

Cet article décrit comment associer votre compte Office 365 à Azure AD pour créer et gérer des applications.

 >**Remarque :** Cet article utilise Azure AD comme fournisseur d’authentification pour votre application. Si vous utilisez le point de terminaison Azure AD v2.0, vous pouvez ignorer cette étape. Pour plus d’informations, voir [Authentification d’application avec Microsoft Graph](../auth_overview.md).

## <a name="prerequisites"></a>Conditions préalables

**Compte Office 365 Business**

Si vous n’avez pas de compte Office 365 Business, vous pouvez :

- vous inscrire à un [plan Office 365 Business](http://products.office.com/en-us/business/compare-office-365-for-business-plans) énuméré ci-dessus, ou
- [articiper au programme pour les développeurs Office 365 et obtenir un abonnement gratuit d’un an à Office 365](https://aka.ms/devprogramsignup).

**Abonnement Microsoft Azure** 

- Si vous avez un abonnement Microsoft Azure, vous pouvez lui associer votre abonnement Office 365 Business. 

- Autrement, vous devrez créer un nouvel abonnement Azure et l’associer à votre compte Office 365, afin de pouvoir inscrire et gérer des applications.


<!---<a name="bk_AssociateExistingAzureSubscription"> </a>-->

## <a name="to-associate-an-existing-azure-subscription-with-your-office-365-account"></a>Association d’un abonnement Azure existant à votre compte Office 365


1. Connectez-vous au [Portail de gestion Microsoft Azure](https://manage.windowsazure.com) avec vos informations d’identification Azure existantes (par exemple, votre ID Microsoft comme utilisateur@live.com).
        
2. Sélectionnez le nœud **Active Directory**, puis sélectionnez l’onglet **Annuaire** et, en bas de l’écran, sélectionnez **Nouveau**. 
     
4. Dans le menu **Nouveau**, sélectionnez **Active Directory**  >  **Annuaire**  >  **Création personnalisée**.
    
5. Dans **Ajout d’un annuaire**, dans la zone de liste déroulante **Annuaire**, sélectionnez **Utiliser un annuaire existant**. Cochez la case **Je suis prêt à me déconnecter**, puis sélectionnez la case à cocher dans le coin inférieur droit. 
    
    Vous revenez au Portail de gestion Azure.
        
3. Connectez-vous avec vos informations de compte Office 365. 
    
    Vous devrez indiquer si vous souhaitez utiliser ou non votre répertoire avec Azure. 
    
    >**Important :** Pour associer votre compte Office 365 à Azure AD, vous aurez besoin d’un compte Office 365 Business avec des privilèges d’administrateur général. 
    
        
4. Sélectionnez **Continuer**, puis **Se déconnecter**.
        
5. Fermez le navigateur et ouvrez de nouveau le [portail](https://manage.windowsazure.com). Autrement, vous obtiendrez une erreur d’accès refusé.
    
        
6. Reconnectez-vous avec vos informations d’identification Azure existantes (par exemple, votre ID Microsoft comme utilisateur@live.com). Accédez au nœud **Active Directory** ; votre compte Office 365 devrait maintenant être répertorié sous **Annuaire**.
    

<!--<a name="bk_AssociateNewAzureSubscription"> </a>-->

## <a name="to-create-a-new-azure-subscription-and-associate-it-with-your-office-365-account"></a>Création d’un abonnement Azure et association à votre compte Office 365


1. Connectez-vous à Office 365. À partir de la page **Accueil**, sélectionnez l’icône **Administrateur** pour ouvrir le centre d’administration Office 365.
2. Dans la page de menu située du côté gauche de la page, faites défiler vers le bas jusqu’à **Administrateur** et sélectionnez **Azure AD**.

    >**Important :** Pour ouvrir le centre d’administration Office 365 et accéder à Azure AD, vous aurez besoin d’un compte Office 365 Business avec des privilèges d’administrateur général. 
    
3. Créez un abonnement.
        
    Si vous utilisez une version d’évaluation d’Office 365, vous verrez un message vous indiquant qu’Azure AD est limité aux clients avec les services payants. Vous pouvez toujours créer un abonnement Azure de 30 jours d’évaluation gratuite, mais vous devrez effectuer quelques étapes supplémentaires :
    
    1. Sélectionnez votre pays ou région, puis choisissez **Abonnement Azure**.
    2. Saisissez vos informations personnelles. À des fins de vérification, saisissez un numéro de téléphone auquel vous pouvez être contacté et indiquez si vous souhaitez recevoir un SMS ou être appelé.
    3. Lorsque vous recevez votre code de vérification, saisissez-le et choisissez **Vérifier le code**.
    4. Saisissez les informations de paiement, acceptez les termes du contrat, puis sélectionnez **S’inscrire**.
        
        Votre carte de crédit ne sera pas débitée.
        
        Ne fermez pas et n’actualisez pas votre navigateur pendant la création de votre abonnement Azure.
            
4. Une fois que votre abonnement Azure est créé, choisissez **Portail**.
        
5. La présentation d’Azure apparaît. Vous pouvez la visualiser ou choisir **X** pour la fermer.
        
    Vous devez maintenant voir tous les éléments dans votre abonnement Azure. Il indique un annuaire avec le nom de votre client Office 365.
    
## <a name="see-also"></a>Voir aussi
- [Informations de base relatives à l’inscription d’une application dans Azure AD](https://azure.microsoft.com/en-us/documentation/articles/active-directory-authentication-scenarios/#basics-of-registering-an-application-in-azure-ad)
- [Ajouter, mettre à jour ou supprimer une application dans Azure AD](https://azure.microsoft.com/en-us/documentation/articles/active-directory-integrating-applications/)