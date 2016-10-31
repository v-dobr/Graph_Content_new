# <a name="use-rest-apis-to-access-mailboxes-in-exchange-hybrid-deployments-(preview)"></a>Utiliser des API REST pour accéder aux boîtes aux lettres dans les déploiements hybrides Exchange (aperçu)

Microsoft Graph a toujours fourni l’accès aux boîtes aux lettres client dans le cloud sur Exchange Online dans le cadre d’Office 365. La mise à jour cumulative 3 (CU3) d’Exchange 2016, publiée en septembre 2016 pour les serveurs locaux Exchange, prend en charge l’intégration de l’API REST avec Office 365. Si votre application utilise la version 1.0 de l’API [Courrier](http://graph.microsoft.io/en-us/docs/api-reference/v1.0/resources/message), [Calendrier](http://graph.microsoft.io/en-us/docs/api-reference/v1.0/resources/calendar) ou [Contacts](http://graph.microsoft.io/en-us/docs/api-reference/v1.0/resources/contact), vous allez maintenant également découvrir une authentification transparente et une expérience d’application dans les déploiements _hybrides_, que la boîte aux lettres soit locale ou dans le cloud, à condition que le déploiement réponde aux [exigences](#requirements-for-rest-api-to-work-in-hybrid-deployments) spécifiques. 


En coulisses, lorsque Microsoft Graph identifie qu’un appel API REST tente d’accéder à une boîte aux lettres locale dans un déploiement hybride, il transmet par proxy la demande REST à un point de terminaison REST local qui traite ensuite la demande. Cette découverte permet d’accéder à l’API REST.

>**Remarque :** La capacité à utiliser ces API REST dans les déploiements hybrides est actuellement en version d’évaluation.

>Seule la version 1.0 des API Courrier, Calendrier et Contacts est disponible pour les boîtes aux lettres dans les déploiements hybrides. Les autres ensembles d’API v1.0, tels que l’API [Groupes](http://graph.microsoft.io/en-us/docs/api-reference/v1.0/resources/group) ou les API dans d’autres versions, ne sont pas disponibles. Si vous essayez d’utiliser une API qui ne fait pas partie du jeu pris en charge dans un déploiement hybride, vous obtenez le message d’erreur suivant :

>« REST APIs for this mailbox are currently in preview. You can find more information about the preview REST APIs at https://dev.outlook.com. »

## <a name="requirements-for-the-rest-api-to-work-in-hybrid-deployments"></a>Configuration requise pour que l’API REST fonctionne dans les déploiements hybrides

Microsoft Graph assure ouverture (prise en charge des normes ouvertes, telles que JSON OAUTH et ODATA, via la connexion à partir de la plupart des plateformes connues) et flexibilité (autorisations précises, à étendue étroite pour accéder aux données utilisateur). Si votre organisation souhaite activer le développement d’applications Microsoft Graph et est actuellement dans un déploiement hybride ou l’envisage, notez les exigences de déploiement suivantes :

- Configuration de boîte aux lettres requise

  - Toutes les boîtes aux lettres locales qui utiliseront les API REST doivent être hébergées sur des bases de données situées sur des serveurs Exchange 2016 CU3. 

- Conditions requises en matière d’infrastructure

  - Tous les serveurs Exchange 2016 doivent être mis à niveau vers la CU3 ou version ultérieure.  
  - Active Directory local doit se synchroniser avec Azure Active Directory.
  - Tous les serveurs Exchange 2013 coexistant dans le même groupe à charge équilibrée avec les serveurs Exchange 2016 doivent être supprimés du groupe.

- Configuration de réseau requise

  - Du point de vue DNS, les espaces de noms de découverte automatique et l’espace de noms du client local doivent avoir des enregistrements DNS Internet. 
  - Si vous disposez d’une passerelle d’application ou d’un pare-feu qui inspecte et limite l’accès, mettez à jour les paramètres appropriés pour autoriser l’accès et la découverte.


Les administrateurs informatiques peuvent trouver plus d’informations dans les ressources suivantes :

- 
  [Déploiements hybrides Exchange Server](https://technet.microsoft.com/en-us/library/jj200581(v=exchg.150).aspx)
- [Version de mise à jour cumulative septembre 2016](https://blogs.technet.microsoft.com/exchange/2016/09/20/released-september-2016-quarterly-exchange-updates/) 
- [Exigences en matière d’architecture locale pour l’API REST](https://blogs.technet.microsoft.com/exchange/2016/09/26/on-premises-architectural-requirements-for-the-rest-api/)
