## <a name="register-your-microsoft-graph-application-with-the-azure-ad-v2.0-endpoint"></a>Inscription de votre application Microsoft Graph avec le point de terminaison Azure AD v2.0

Pour utiliser le point de terminaison Azure AD v2.0, vous devez inscrire votre application sur le [portail d’inscription des applications Microsoft](https://apps.dev.microsoft.com) (https://apps.dev.microsoft.com). L’inscription de votre application établit l’identité de votre application avec le fournisseur d’authentification et permet à votre application de prouver son identité lors de l’envoi de demandes d’authentification de l’utilisateur. L’inscription génère l’ID d’application et la question secrète de l’application que vous utiliserez pour configurer l’application pour l’authentification.

> **Remarque : **cet article décrit l’inscription de l’application avec le point de terminaison Azure AD v2.0. Pour [inscrire votre application avec Azure AD](app_authentication_azure_ad.md), utilisez le [portail Azure](https://aka.ms/aadapplist).
> 
> En outre, n’oubliez pas que si vous avez déjà inscrit des applications dans le portail de gestion Microsoft Azure, ces applications ne sont pas répertoriées dans le portail d’inscription des applications. Gérez ces applications dans le portail de gestion Azure. 

1. Connectez-vous au [portail d’inscription des applications Microsoft](https://apps.dev.microsoft.com/) en utilisant votre compte personnel, professionnel ou scolaire.

2. Choisissez **Ajouter une application**.

3. Entrez un nom pour l’application, puis choisissez **Créer une application**.

    La page d’inscription s’affiche, répertoriant les propriétés de votre application.

4. Copiez l’ID de l’application. Il s’agit de l’identificateur unique de votre application.

    Vous utiliserez l’ID d’application pour configurer l’application.

5. Sous **Plateformes**, choisissez **Ajouter une plateforme** et sélectionnez la plateforme appropriée pour votre application :
    
    Pour les applications clientes :
    1. Sélectionnez **Mobile Platform**.

    2. Copiez les valeurs d’ID client (ID d’application) et d’URI de redirection dans le Presse-papiers. Vous devrez saisir ces valeurs dans l’exemple d’application.

        L’ID d’application est un identificateur unique pour votre application. L’URI de redirection est un URI unique fourni pour chaque application afin de garantir que les messages envoyés à cet URI sont envoyés uniquement à cette application. 

    Pour les applications web :
    1. Sélectionnez **Web**.
    2. Si vous utilisez le type d’octroi Implicite ou si vous utilisez le flux hybride OpenID Connect, vérifiez que la case Autoriser le flux implicite est cochée. 
        
        L’option Autoriser le flux implicite active le flux hybride OpenID Connect. Lors de l’authentification, cela permet à l’application de recevoir les informations de connexion (id_token) et les artefacts (dans ce cas, un code d’autorisation) qui servent à obtenir un jeton d’accès.


    3. Spécifiez un URI de redirection.
        
        L’URI de redirection est l’emplacement dans votre application que le point de terminaison Azure AD v2.0 appelle lorsqu’il a traité la demande d’authentification.
    4. Sous **Secrets de l'application**, choisissez **Générer un nouveau mot de passe**. Copiez la question secrète de l’application à partir de la boîte de dialogue **Nouveau mot de passe créé**.
        
        Vous utiliserez la question secrète de l’application pour configurer l’application.
    
6. Cliquez sur **Enregistrer**.

## <a name="see-also"></a>Voir aussi

[Authentification d’application avec Microsoft Graph](auth_overview.md)
