# <a name="microsoft-graph-permission-scopes"></a>Étendues d’autorisation de Microsoft Graph

Microsoft Graph expose les étendues d’autorisations OAuth 2.0 qui sont utilisées pour contrôler l’accès aux ressources d’une application. En tant que développeur, vous spécifiez les étendues d’autorisation appropriées pour l’accès requis par votre application. (Si vous utilisez l’authentification Azure AD, vous effectuez généralement cette action via le portail de gestion Azure. Si vous utilisez le point de terminaison Azure AD v2.0, vous demandez des autorisations dynamiquement pendant l’exécution.)

Après la connexion, les utilisateurs ou administrateurs ont la possibilité de consentir à autoriser votre application à accéder à leurs ressources avec les étendues d’autorisation que vous avez configurées. Pour cette raison, vous devez choisir les étendues d’autorisation qui fournissent le niveau de privilège le plus bas requis par votre application. Pour plus d’informations sur la façon de configurer les autorisations pour votre application et sur le processus de consentement, voir <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-integrating-applications/" target="_newtab">Intégration d’applications dans Azure Active Directory</a>.

>**Remarque :** certaines autorisations Microsoft Graph, telles que celles appartenant à des groupes et des tâches, ne sont pas applicables aux comptes personnels.  

##<a name="app-only-vs.-delegated-scopes"></a>Étendues déléguées et d’application uniquement
Les étendues d’autorisation peuvent être soit des étendues déléguées, soit des étendues d’application uniquement. Les étendues d’application uniquement (également appelées rôles d’application) accordent à l’application l’ensemble complet de privilèges offerts par l’étendue. Les étendues d’application uniquement sont généralement utilisées par des applications qui s’exécutent en tant que service sans utilisateur connecté. 


Les étendues d’autorisation déléguées concernent les applications qui agissent pour le compte d’un utilisateur. Ces étendues délèguent les privilèges de l’utilisateur connecté, ce qui permet à l’application d’agir comme l’utilisateur. Les droits réels accordés à l’application seront la combinaison de privilèges minimaux (l’intersection) des privilèges accordés par l’étendue et de ceux de l’utilisateur connecté. Par exemple, si l’étendue d’autorisation accorde des privilèges délégués pour écrire tous les objets d’annuaire, mais que l’utilisateur connecté possède des privilèges uniquement pour mettre à jour son propre profil utilisateur, l’application pourra uniquement écrire le profil de l’utilisateur connecté, mais pas d’autres objets.

## <a name="full-and-basic-profiles-for-users-and-groups"></a>Profils complets et de base pour des utilisateurs et des groupes

Le profil complet (ou profil) d’un utilisateur ou d’un groupe comprend toutes les propriétés déclarées de l’entité. Étant donné que le profil peut contenir des informations d’annuaire sensibles ou des informations d’identification personnelle (PII), plusieurs étendues restreignent l’accès de l’application à un ensemble limité de propriétés appelé profil de base. Pour les utilisateurs, le profil de base inclut uniquement les propriétés suivantes : 

- Nom complet
- Prénom et nom
- Photo
- Adresse de messagerie

Pour les groupes, le profil de base contient uniquement le nom d’affichage. 

<!---   <a name="msg_perm_details"> </a>  -->

## <a name="permission-scope-details"></a>Détails de l’étendue d’autorisation

Vous devez configurer votre application pour avoir les autorisations nécessaires pour accéder aux ressources de Microsoft Graph. Les autorisations sont étendues aux ressources individuelles pour les droits en lecture, écriture ou les deux à la fois. 

Les tableaux suivants répertorient les étendues d’autorisation de Microsoft Graph et expliquent l’accès accordé par chacune d’elles. 

- La colonne **Étendue** indique le nom de l’étendue. Les noms d’étendue sont au format resource.operation.constraint ; par exemple, Group.ReadWrite.All. Si la contrainte est « All », l’étendue accorde à l’application la possibilité d’exécuter l’opération (ReadWrite) sur toutes les ressources spécifiées (Group) dans l’annuaire ; dans le cas contraire, l’étendue permet uniquement l’opération sur le profil de l’utilisateur connecté. Les étendues peuvent accorder des privilèges limités pour l’opération spécifiée. Consultez la colonne **Description** pour des informations détaillées.
- La colonne **Autorisation** indique comment l’étendue est affichée dans le portail d’Azure. 
- La colonne **Description** décrit l’ensemble des privilèges accordés par l’étendue. Pour les étendues déléguées, l’accès réel accordé à l’application sera la combinaison de privilèges minimaux (intersection) de l’accès accordé par l’étendue et des privilèges de l’utilisateur connecté. 
- Les étendues sont regroupées selon que les autorisations exigent ou non le consentement d’un administrateur.

  > **Remarque** : voir [Problèmes connus](../overview/release_notes) pour les limites d’étendue d’autorisation `v1.0` et `beta`.
  
###<a name="permissions-requiring-administrator's-consent"></a>Autorisations nécessitant le consentement de l’administrateur

|   **Étendue**                  |  **Autorisation**                          |  **Description** |
|:-----------------------------|:-----------------------------------------|:-----------------|
| _User.Read.All_                |     Lire les profils complets de tous les utilisateurs           | Identique à User.ReadBasic.All, sauf qu’elle permet à l’application de lire le profil complet de tous les utilisateurs de l’organisation et lors de la lecture des propriétés de navigation comme les rapports de responsable et de collaborateurs. Le profil complet inclut toutes les propriétés déclarées de l’entité **Utilisateur**. Pour lire les groupes dont un utilisateur est membre, l’application exigera également soit Group.Read.All soit Group.ReadWrite.All. |
| _User.ReadWrite.All_           |     Lire et écrire les profils complets de tous les utilisateurs | Permet à l’application de lire et d’écrire l’ensemble complet des propriétés de profil, les rapports et les responsables d’autres utilisateurs de votre organisation, au nom de l’utilisateur connecté. |
| _Directory.Read.All_           |     Lire les données de l’annuaire                     | Permet à l’application de lire les données dans l’annuaire de votre organisation, comme les utilisateurs, les groupes et les applications. |
| _Directory.ReadWrite.All_      |     Lire et écrire les données de l’annuaire           | Permet à l’application de lire et écrire des données dans l’annuaire de votre organisation, telles que les utilisateurs et les groupes.  N’autorise pas la suppression d’un utilisateur ou d’un groupe. Elle n’autorise pas l’application à supprimer des utilisateurs ou des groupes ou à réinitialiser les mots de passe de l’utilisateur. |
| _Directory.AccessAsUser.All_   |     Accéder à l’annuaire en tant qu’utilisateur connecté  | Permet à l’application de disposer du même accès aux informations dans l’annuaire que l’utilisateur connecté.|
| _Group.Read.All_ |    Lire tous les groupes | Permet à l’application de répertorier les groupes et de lire leurs propriétés et toutes les appartenances au groupe, au nom de l’utilisateur connecté.  Permet également à l’application de lire le calendrier, les conversations, les fichiers et les autres types de contenu de groupe pour tous les groupes auxquels l’utilisateur connecté peut accéder. |
| _Group.ReadWrite.All_ |    Lire et écrire tous les groupes| Permet à l’application de créer des groupes et de lire toutes les propriétés et les appartenances du groupe, au nom de l’utilisateur connecté.  En outre, elle permet aux propriétaires de groupes de gérer leurs groupes et aux membres de groupes de mettre à jour le contenu des groupes. |


###<a name="permissions-not-requiring-administrator's-consent"></a>Autorisations ne nécessitant pas le consentement de l’administrateur

|   **Étendue**    |  **Autorisation**   |  **Description** |
|:-----------------------------|:-----------------------------------------|:-----------------|
| _User.Read_       |    Activer la connexion et lire le profil utilisateur | Permet aux utilisateurs de se connecter à l’application et permet à l’application de lire le profil des utilisateurs connectés. Le profil complet inclut toutes les propriétés déclarées de l’entité Utilisateur. L’application ne peut pas lire les propriétés de navigation, telles que les rapports de responsables ou de collaborateurs. Permet également à l’application de lire les informations de base suivantes sur la société de l’utilisateur connecté (via l’objet **TenantDetail**) : ID client, nom d’affichage du client et domaines vérifiés.|
| _User.ReadWrite_ |    Accéder en lecture et en écriture au profil utilisateur | Permet à l’application de lire votre profil. Elle permet également à l’application de mettre à jour vos informations de profil à votre place. |
| _User.ReadBasic.All_ |    Lire les profils de base de tous les utilisateurs | Permet à l’application de lire le profil de base de tous les utilisateurs de l’organisation au nom de l’utilisateur connecté. Les propriétés suivantes comprennent le profil de base d’un utilisateur : nom d’affichage, prénom et nom, photo et adresse électronique. Pour lire les groupes dont un utilisateur est membre, l’application exigera également Group.Read.All ou Group.ReadWrite.All.| 
| _Mail.Read_ |    Accéder en lecture aux courriers électroniques utilisateur | Permet à l’application de lire les courriers électroniques dans des boîtes aux lettres utilisateur.  |
| _Mail.ReadWrite_ |    Accéder en lecture et en écriture aux courriers électroniques utilisateur | Permet à l’application de créer, lire, mettre à jour et supprimer des messages électroniques dans des boîtes aux lettres utilisateur. N’inclut pas l’autorisation d’envoyer des messages électroniques.|
| _Mail.Send_ |    Envoyer un courrier électronique en tant qu’utilisateur | Permet à l’application d’envoyer des messages électroniques en tant qu’utilisateurs dans l’organisation.  |
| _Calendars.Read_ |    Accéder en lecture aux calendriers utilisateur  | Permet à l’application de lire les événements dans les calendriers utilisateur.|
| _Calendars.ReadWrite_ |    Avoir un accès total à des calendriers utilisateur  | Permet à l’application de créer, lire, mettre à jour et supprimer des événements dans des calendriers utilisateur. |
| _Contacts.Read_ |    Accéder en lecture aux contacts utilisateur  | Permet à l’application de lire les contacts de l’utilisateur. |
| _Contacts.ReadWrite_ |    Avoir un accès total à des contacts utilisateur  | Permet à l’application de créer, lire, mettre à jour et supprimer des contacts de l’utilisateur. |
| _Files.Read_ |    Lire les fichiers de l’utilisateur et les fichiers partagés avec l’utilisateur | Permet à l’application de lire les fichiers de l’utilisateur connecté et les fichiers partagés avec l’utilisateur.| 
| _Files.ReadWrite_ |   Ont un accès complet aux fichiers de l’utilisateur et aux fichiers partagés avec l’utilisateur | Permet à l’application de lire, créer, mettre à jour et supprimer les fichiers de l’utilisateur connecté et les fichiers partagés avec l’utilisateur. |
| _Files.ReadWrite.Selected_ |    Lire et écrire des fichiers sélectionnés par l’utilisateur | Permet à l’application de lire et d’écrire des fichiers sélectionnés par l’utilisateur. L’application a accès pendant plusieurs heures une fois que l’utilisateur sélectionne un fichier. |
| _Files.Read.Selected_ |    Lire des fichiers sélectionnés par l’utilisateur  | Permet à l’application de lire des fichiers sélectionnés par l’utilisateur. L’application a accès pendant plusieurs heures une fois que l’utilisateur sélectionne un fichier. |
| _Sites.Read.All_ |    Lire des éléments dans toutes les collections de sites | Permet à l’application de lire des documents et des éléments de liste dans toutes les collections de sites au nom de l’utilisateur connecté. |
| _openid_ |    Connecter des utilisateurs (aperçu) | Permet aux utilisateurs de se connecter à l’application avec leurs comptes professionnels ou scolaires et permet à l’application d’afficher les informations de profil utilisateur de base.|
| _offline_access_ |    Accéder aux données de l’utilisateur à tout moment (aperçu) | Permet à l’application de lire et de mettre à jour les données de l’utilisateur, même si elles n’utilisent pas l’application actuellement.|

###<a name="app-only-permissions-requiring-administrator's-consent"></a>Autorisations d’application uniquement nécessitant le consentement de l’administrateur

|   **Étendue**    |  **Autorisation**   |  **Description** |
|:---------------|:------------------|:-----------------|
| _Mail.Read_       |    Lire les messages dans toutes les boîtes aux lettres | Permet à l’application de lire les messages dans toutes les boîtes aux lettres sans utilisateur connecté.|
| _Mail.ReadWrite_ |    Lire et écrire les messages dans toutes les boîtes aux lettres | Permet à l’application de créer, lire, mettre à jour et supprimer les messages dans toutes les boîtes aux lettres sans utilisateur connecté. N’inclut pas l’autorisation d’envoyer des messages électroniques. |
| _Mail.Send_ |    Envoyer des messages électroniques en tant qu’utilisateur | Permet à l’application d’envoyer des messages électroniques en tant qu’utilisateur sans utilisateur connecté. | 
| _Calendars.Read_ |    Lire les calendriers dans toutes les boîtes aux lettres | Permet à l’application de lire les événements de tous les calendriers sans utilisateur connecté. |
| _Calendars.ReadWrite_ |    Lire et écrire les calendriers dans toutes les boîtes aux lettres | Permet à l’application de créer, lire, mettre à jour et supprimer des événements de tous les calendriers sans utilisateur connecté.|
| _Contacts.Read_ |    Lire les contacts dans toutes les boîtes aux lettres | Permet à l’application de lire tous les contacts dans toutes les boîtes aux lettres sans utilisateur connecté. |
| _Contacts.ReadWrite_ |    Lire et écrire les contacts dans toutes les boîtes aux lettres  |Permet à l’application de créer, lire, mettre à jour et supprimer tous les contacts dans toutes les boîtes aux lettres sans utilisateur connecté.|
| _User.ReadBasic.All_ |    Lire les profils de base de tous les utilisateurs  | Permet à l’application de lire un ensemble de base de propriétés de profil d’autres utilisateurs de votre organisation sans utilisateur connecté. Inclut le nom d’affichage, prénom et nom, photo et message d’absence du bureau.|
| _User.Read.All_ |    Lire les profils complets de tous les utilisateurs | Permet à l’application de lire l’ensemble complet des propriétés de profil, l’appartenance à un groupe, les rapports et les responsables d’autres utilisateurs de votre organisation, sans utilisateur connecté.| 
| _User.ReadWrite.All_ |   Lire et écrire les profils complets de tous les utilisateurs | Permet à l’application de lire et d’écrire l’ensemble complet des propriétés de profil, l’appartenance à un groupe, les rapports et les responsables d’autres utilisateurs de votre organisation, sans utilisateur connecté.|


##<a name="permission-scopes-in-preview"></a>Étendues d’autorisation dans l’aperçu
###<a name="permissions-not-requiring-administrator's-consent-(preview)"></a>Autorisations ne nécessitant pas le consentement de l’administrateur (fonctionnalité d’évaluation)

|   **Étendue**    |  **Autorisation**   |  **Description** |
|:---------------|:------------------|:-----------------|
| _Tasks.ReadWrite_ |    Créer, lire, mettre à jour et supprimer des tâches et des plans de l’utilisateur (aperçu) | Permet à l’application de créer, lire, mettre à jour et supprimer des tâches et des plans (et des tâches qu’ils contiennent), qui sont affectés ou partagés avec l’utilisateur connecté.|
| _People.Read_ |    Lire les listes des personnes appropriées des utilisateurs (aperçu) | Permet à l’application de lire une liste de classement des personnes appropriées de l’utilisateur connecté. La liste comprend les contacts locaux, contacts de réseaux sociaux, annuaire de votre organisation et les personnes issues de communications récentes (messagerie électronique et Skype, par exemple).|
| _People.ReadWrite_ |    Lire et écrire les listes des personnes appropriées des utilisateurs (aperçu) | Permet à l’application de créer, lire et écrire une liste de classement des personnes appropriées de l’utilisateur connecté. La liste comprend les contacts locaux, contacts de réseaux sociaux, annuaire de votre organisation et les personnes issues de communications récentes (messagerie électronique et Skype, par exemple).|
| _Notes.Create_ |    Créer des pages dans les blocs-notes des utilisateurs (aperçu) | Permet à l’application de lire les titres de blocs-notes et de sections et de créer des pages, des blocs-notes et des sections au nom de l’utilisateur connecté.|
| _Notes.ReadWrite.CreatedByApp_ |    Accès limité au bloc-notes (aperçu) | Permet à l’application de lire les titres de blocs-notes et de sections, de créer des pages au nom de l’utilisateur connecté. Permet également à l’application de lire et de mettre à jour des pages créées par l’application. |
| _Notes.Read_ |    Lire les blocs-notes de l’utilisateur (aperçu) | Permet à l’application d’afficher les titres de blocs-notes et de sections OneNote et de lire toutes les pages au nom de l’utilisateur connecté. Elle ne peut pas afficher les sections protégées par mot de passe. |
| _Notes.ReadWrite_ |    Lire et écrire les blocs-notes de l’utilisateur (aperçu) | Permet à l’application de lire les titres de blocs-notes et de sections, lire toutes les pages, écrire toutes les pages et créer des pages, au nom de l’utilisateur connecté.  Elle ne peut pas accéder aux sections protégées par mot de passe. |
| _Notes.Read.All_ |    Lire tous les blocs-notes auxquels l’utilisateur peut accéder (aperçu) | Permet à l’application de lire le contenu de tous les blocs-notes et sections auxquels l’utilisateur connecté peut accéder.   Elle ne peut pas lire les sections protégées par mot de passe. |
| _Notes.ReadWrite.All_ |    Lire et écrire les blocs-notes auxquels l’utilisateur peut accéder (aperçu) | Permet à l’application de lire et d’écrire le contenu de tous les blocs-notes et sections auxquels l’utilisateur connecté peut accéder.  Elle ne peut pas accéder aux sections protégées par mot de passe.|

##<a name="permission-scope-scenarios"></a>Scénarios d’étendue d’autorisation
Voici des scénarios d’application utilisant les ressources `User` et `Group`, et leurs étendues requises correspondantes. Le tableau suivant indique les étendues d’autorisations nécessaires pour qu’une application puisse effectuer des opérations spécifiques. Notez que dans certains cas, la capacité de l’application à effectuer certaines opérations dépendra du fait que l’étendue d’autorisation est d’application uniquement ou déléguée et, dans le cas des étendues d’autorisation déléguées, des privilèges de l’utilisateur connecté. 

###<a name="access-scenarios-using-the-user-resource-and-the-required-scopes"></a>Scénarios d’accès utilisant la ressource utilisateur et les étendues requises

| **Tâches de l’application impliquant l’utilisateur**   |  **Étendues requises** | **Autorisations** |
|:-------------------------------|:---------------------|:---------------|
| L’application souhaite lire les informations de base d’autres utilisateurs (nom d’affichage et image uniquement), par exemple pour les afficher dans une expérience de choix de personnes   | _User.ReadBasic.All_  |  Lire les profils de base de tous les utilisateurs |
| L’application souhaite lire le profil utilisateur complet pour l’utilisateur connecté (voir rapports de collaborateur, responsable, etc.)  | _User.Read_ | Activer la connexion et lire le profil utilisateur|
| L’application souhaite lire le profil utilisateur complet de tous les utilisateurs  | _User.Read.All_ |  Lire les profils complets de tous les utilisateurs   |
| L’application souhaite lire les fichiers, les informations de messagerie et de calendrier de l’utilisateur connecté  | _User.Read_, _Files.Read_, _Mail.Read_, _Calendar.Read_ | Activer la connexion et lire le profil utilisateur, Accéder en lecture aux fichiers utilisateur, Accéder en lecture aux courriers électroniques utilisateur, Accéder en lecture aux calendriers utilisateur |
| L’application souhaite lire les fichiers (my) de l’utilisateur connecté et les fichiers que d’autres utilisateurs ont partagés avec l’utilisateur connecté (me). | _User.Read_, _Files.Read_, _Sites.Read.All_ | Activer la connexion et lire le profil utilisateur, Accéder en lecture aux fichiers utilisateur, Lire des éléments dans toutes les collections de sites |
| L’application souhaite lire et écrire le profil complet de l’utilisateur connecté   | _User.ReadWrite_ | Accéder en lecture et en écriture au profil utilisateur |
| L’application souhaite lire et écrire le profil complet de tous les utilisateurs    | _User.ReadWrite.All_ | Lire et écrire les profils complets de tous les utilisateurs |
| L’application souhaite lire et écrire les fichiers, les informations de messagerie et de calendrier de l’utilisateur connecté    | _User.ReadWrite_, _Files.ReadWrite_, _Mail.ReadWrite_, _Calendar.ReadWrite_  |  Accéder en lecture et en écriture au profil utilisateur, Accéder en lecture et en écriture au profil utilisateur, Accéder en lecture et en écriture aux courriers électroniques utilisateur, Avoir un accès total à des calendriers utilisateur |
   

###<a name="access-scenarios-using-the-group-resource-and-the-required-scopes"></a>Scénarios d’accès utilisant la ressource groupe et les étendues requises
    
| **Tâches de l’application impliquant le groupe**  |  **Étendues requises** |  **Autorisations** |
|:-------------------------------|:---------------------|:---------------|
| L’application souhaite lire les informations du groupe de base (nom d’affichage et image uniquement), par exemple pour les afficher dans une expérience de choix de groupes  | _Group.Read.All_  | Lire tous les groupes|
| L’application souhaite lire tout le contenu de tous les groupes unifiés, y compris les fichiers, les conversations.  Elle doit également afficher les appartenances de groupe, pouvoir mettre à jour les membres du groupe, (si propriétaire).  |  _Group.Read.All_ | Lire des éléments dans toutes les collections de sites, Lire tous les groupes|
| L’application souhaite lire et écrire tout le contenu dans tous les groupes unifiés, y compris les fichiers, les conversations.  Elle doit également afficher les appartenances de groupe, pouvoir mettre à jour les membres du groupe, (si propriétaire).  |  _Group.ReadWrite.All_, _Sites.ReadWrite.All_ |  Lire et écrire tous les groupes, Modifier ou supprimer des éléments dans toutes les collections de sites |
| L’application souhaite découvrir (trouver) un groupe unifié. Cela permet à l’utilisateur de rechercher un groupe particulier et d’en choisir un dans la liste répertoriée pour autoriser l’utilisateur à rejoindre le groupe.     | _Group.ReadWrite.All_ | Lire et écrire tous les groupes|
| L’application souhaite créer un groupe via AAD Graph |   _Group.ReadWrite.All_ | Lire et écrire tous les groupes|
 


