# <a name="authenticate-microsoft-graph-apps-with-the-azure-ad-v2.0-endpoint"></a>Authentification d’applications Microsoft Graph avec le point de terminaison Azure AD v2.0

> **Vous voulez créer des applications pour des entreprises ?** Il est possible que votre application ne fonctionne pas si l’entreprise a activé les fonctionnalités de sécurité pour la mobilité en entreprise comme l’<a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-conditional-access-device-policies/" target="_newtab">accès conditionnel des appareils</a>.  

> Pour prendre en charge **toutes les entreprises clientes** dans **tous les scénarios**, utilisez le point de terminaison Azure AD et gérez vos applications avec le [portail de gestion Azure](https://aka.ms/aadapplist). Pour plus d’informations, consultez les [fonctionnalités des points de terminaison Azure AD et Azure AD v2.0](auth_overview.md#deciding-between-azure-ad-and-the-v2-authentication-endpoint).


Le point de terminaison Azure AD v2.0 vous permet de créer des applications qui acceptent aussi bien les identités professionnelles et scolaires (Azure Active Directory) que les identités personnelles (compte Microsoft).

Dans le passé, pour développer une application prenant en charge les comptes Microsoft et Azure Active Directory, vous deviez vous intégrer à deux systèmes totalement distincts. Avec le point de terminaison Azure AD v2.0, vous pouvez désormais prendre en charge les deux types de comptes avec une intégration unique : un processus simple pour toucher une audience constituée de millions d’utilisateurs avec des comptes personnels et des comptes professionnels/scolaires.  

Une fois que vous intégrez vos applications avec le point de terminaison Azure AD v2.0, celles-ci peuvent immédiatement accéder aux points de terminaison Microsoft Graph disponibles pour les comptes à la fois personnels et professionnels ou scolaires, tels que : 

| Données              | Point de terminaison                                       |
|:------------------|:-----------------------------------------------|
| Profil utilisateur      | `https://graph.microsoft.com/v1.0/me`          |
| Messagerie Outlook      | `https://graph.microsoft.com/v1.0/me/messages` |
| Contacts Outlook  | `https://graph.microsoft.com/v1.0/me/contacts` |
| Calendriers Outlook | `https://graph.microsoft.com/v1.0/me/events`   |
| OneDrive          | `https://graph.microsoft.com/v1.0/me/drive`    |

 >**Remarque :** certains points de terminaison Microsoft Graph, tels que les groupes et les tâches, ne sont pas applicables aux comptes personnels.  

## <a name="microsoft-graph-authentication-scopes"></a>Étendues d’authentification de Microsoft Graph

Le point de terminaison Azure AD v2.0 prend en charge toutes les étendues d’autorisation répertoriées dans [Étendues d’autorisation de Microsoft Graph](permission_scopes.md). 

Pour plus d’informations sur l’utilisation d’étendues avec le point de terminaison Azure AD v2.0 et la façon dont elle diffère de l’utilisation de ressources dans Azure AD, voir <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/#scopes-not-resources" target="_newtab">Des étendues, pas des ressources</a>.

## <a name="see-it-in-action"></a>Voir En action

L’article relatif aux [exemples de connexion dans le référentiel Microsoft Graph](https://github.com/microsoftgraph?utf8=%E2%9C%93&query=connect) fournit des exemples simples décrivant comment authentifier les utilisateurs et se connecter à Microsoft Graph sur un large éventail de plateformes.

En outre, la section [Prise en main](http://graph.microsoft.io/en-us/docs/platform/get-started) contient des articles qui expliquent comment créer ces applications exemples, y compris les bibliothèques d’authentification utilisées sur chaque plateforme.

## <a name="see-also"></a>Voir aussi

- [Inscription d’une application avec le point de terminaison Azure AD v2.0](auth_register_app_v2.md)
- [Authentification d’application avec Microsoft Graph](auth_overview.md)
- <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare" target="_newtab">Nouveautés du modèle de Azure AD v2.0</a>
- <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/" target="_newtab">Dois-je utiliser le point de terminaison Azure AD v2.0 ?</a>
- <a href="https://azure.microsoft.com/en-us/documentation/articles/?product=active-directory&term=azure+ad+v2.0" target="_newtab">Documentation du point de terminaison Azure AD v2.0 sur Azure.com</a>
- <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-app-registration/#build-a-quick-start-app" target="_newtab">Démarrage rapide du code Azure AD v2.0 sur Azure.com</a>

