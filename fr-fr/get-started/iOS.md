# <a name="get-started-with-microsoft-graph-in-an-ios-app"></a>Prise en main de Microsoft Graph dans une application iOS

> **Vous voulez créer des applications pour des entreprises ?** Il est possible que votre application ne fonctionne pas si l’entreprise a activé les fonctionnalités de sécurité pour la mobilité en entreprise comme l’<a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-conditional-access-device-policies/" target="_newtab">accès conditionnel des appareils</a>. Dans ce cas, vos clients peuvent rencontrer des erreurs. 

> Pour prendre en charge **toutes les entreprises clientes** dans **tous les scénarios**, utilisez le point de terminaison Azure AD et gérez vos applications avec le [portail de gestion Azure](https://aka.ms/aadapplist). Pour plus d’informations, consultez les [fonctionnalités des points de terminaison Azure AD et Azure AD v2.0](../authorization/auth_overview.md#deciding-between-azure-ad-and-the-v2-authentication-endpoint).

Cet article décrit les tâches requises pour obtenir un jeton d’accès du [point de terminaison d’authentification v2.0](https://graph.microsoft.io/en-us/docs/authorization/converged_auth) et appeler Microsoft Graph. Il vous guide à travers le code de l’[Exemple d’Office Connect 365 pour iOS (SDK)](https://github.com/microsoftgraph/ios-objectivec-connect-sample) pour expliquer les concepts de base que vous devez implémenter dans une application qui utilise Microsoft Graph. Il décrit comment accéder à Microsoft Graph à l’aide du [Kit de développement logiciel (SDK) Microsoft Graph pour iOS](https://github.com/microsoftgraph/msgraph-sdk-ios).

Vous pouvez télécharger la version de l’application que vous allez créer à partir de ce référentiel GitHub :

* [Exemple de connexion d’Office 365 pour iOS avec le kit de développement logiciel Microsoft Graph](https://github.com/microsoftgraph/ios-objectivec-connect-sample)

L’image suivante présente l’application que vous allez créer.

![Procédure pas à pas de l’exemple de connexion qui indique comment se connecter et envoyer un e-mail dans l’application](./images/iOSConnectWalkthrough.png)


Le flux de travail consistera à se connecter/s’authentifier dans Microsoft Graph, se connecter avec votre compte professionnel ou personnel, et enfin à envoyer un e-mail à un destinataire.

**Pas envie de créer une application ?** Utilisez le [démarrage rapide de Microsoft Graph](https://graph.microsoft.io/en-us/getting-started) pour devenir opérationnel rapidement.

## <a name="prerequisites"></a>Conditions préalables

Voici ce dont vous aurez besoin pour démarrer : 

* [Xcode](https://developer.apple.com/xcode/downloads/) d’Apple
* Installation de [CocoaPods](https://guides.cocoapods.org/using/using-cocoapods.html) comme gestionnaire de dépendances
* Un [compte Microsoft](https://www.outlook.com/) ou un [compte professionnel ou scolaire](http://dev.office.com/devprogram)
* Le [Projet de lancement Microsoft Graph pour iOS](https://github.com/microsoftgraph/ios-objectivec-connect-sample). Ce modèle contient des classes auxquelles vous ajouterez un code. Pour obtenir ce projet, clonez ou téléchargez l’exemple de projet à partir de cet emplacement, et vous utiliserez l’espace de travail dans le dossier **starter-project** (**O365-iOS-Microsoft-Graph-SDK.xcworkspace**).

## <a name="register-the-app"></a>Inscription de l’application
 
1. Connectez-vous au [portail d’inscription des applications](https://apps.dev.microsoft.com/) en utilisant votre compte personnel, professionnel ou scolaire.
2. Sélectionnez **Ajouter une application**.
3. Entrez un nom pour l’application, puis sélectionnez **Créer une application**.
    
    La page d’inscription s’affiche, répertoriant les propriétés de votre application.
 
4. Sous **Plateformes**, sélectionnez **Ajouter une plateforme**.
5. Sélectionnez **Mobile Platform**.
6. Copiez la valeur d’ID client dans le Presse-papiers. Vous devrez saisir cette valeur dans l’exemple d’application.

    L’ID d’application est un identificateur unique pour votre application. 

7. Cliquez sur **Enregistrer**.

## <a name="importing-the-project-dependencies"></a>Importation des dépendances du projet

1. Clonez ce référentiel, [Exemple de connexion d’Office 365 Connect pour iOS avec le kit de développement logiciel Microsoft Graph](https://github.com/microsoftgraph/ios-objectivec-connect-sample). **N’oubliez pas que vous utiliserez l’exemple dans le dossier starter-project et non l’exemple à la racine du projet.**
2. Utilisez CocoaPods pour importer les dépendances d’authentification et le kit de développement logiciel Microsoft Graph. Cet exemple d’application contient déjà un podfile qui recevra les pods dans le projet. Accédez au dossier **starter-project** dans l’application **Terminal** et depuis **Terminal**, exécutez :

        pod install

   Vous recevrez une confirmation que les pods ont été importés dans le projet. Pour plus d’informations, voir [CocoaPods](https://guides.cocoapods.org/using/using-cocoapods.html)


## <a name="enable-keychain-sharing"></a>Activation du partage du trousseau
 
Pour Xcode 8, vous devez ajouter le groupe de trousseau, sinon votre application ne pourra pas y accéder. Pour ajouter le groupe de trousseau :
 
1. Sélectionnez le projet dans le volet du responsable de projet dans Xcode (⌘ + 1).
 
2. Sélectionnez **O365-iOS-Microsoft-Graph-SDK**.
 
3. Sous l’onglet Fonctionnalités, activez **Partage du trousseau**.
 
4. Ajoutez **com.microsoft.O365-iOS-Microsoft-Graph-SDK** aux groupes de trousseau.
 

## <a name="authenticating-with-microsoft-graph"></a>Authentification avec Microsoft Graph

Pour renvoyer le flux de travail de l’interface utilisateur, l’utilisateur de l’application va s’authentifier, puis il pourra envoyer un e-mail à un utilisateur spécifié. Pour effectuer des requêtes auprès du service Microsoft Graph, vous devez fournir un fournisseur d’authentification capable d’authentifier les requêtes HTTPS avec un jeton de support OAuth 2.0 approprié. Dans l’exemple de projet, il y a une classe d’authentification déjà générée appelée **AuthenticationProvider.m.** Nous ajouterons une fonction pour demander et acquérir un jeton d’accès pour appeler l’API Microsoft Graph. 

1. Ouvrez l’espace de travail du projet Xcode (**O365-iOS-Microsoft-Graph-SDK.xcworkspace**) dans le dossier **starter-project**, puis accédez au dossier **Authentification** et ouvrez le fichier **AuthenticationProvider.m.** Ajoutez le code suivant à cette classe.

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

2. Ensuite, ajoutez la méthode au fichier d’en-tête. Ouvrez le fichier **AuthenticationProvider.h** et ajoutez le code suivant à cette classe.

        -(void) connectToGraphWithClientId:(NSString *)clientId
                            scopes:(NSArray *)scopes
                        completion:(void (^)(NSError *error))completion;



2. Pour finir, nous allons appeler cette méthode depuis **ConnectViewController.m**. Ce contrôleur est l’affichage par défaut chargé par l’application, et il existe un seul bouton nommé **Se connecter** que l’utilisateur utilisera pour lancer le processus d’authentification. Cette méthode prend deux paramètres, l’**ID client** et les **étendues**, que nous allons aborder plus en détails ci-dessous. Ajoutez l’action suivante à **ConnectViewController.m**.

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

## <a name="send-an-email-with-microsoft-graph"></a>Envoi d’un message électronique avec Microsoft Graph

Une fois que vous avez configuré le projet pour pouvoir authentifier, les tâches suivantes consistent à envoyer un e-mail à un utilisateur à l’aide de l’API Microsoft Graph. Par défaut, l’utilisateur connecté sera le destinataire, mais vous pouvez le modifier et choisir un autre destinataire. Le code que nous allons utiliser ici se trouve dans le dossier **Controllers** et dans la classe **SendMailViewController.m.** Vous verrez qu’il y a un autre code représenté ici pour l’interface utilisateur et une méthode d’assistance pour récupérer les informations de profil utilisateur à partir du service Microsoft Graph. Nous allons nous concentrer sur les méthodes de création d’un message électronique et d’envoi de ce message.

1. Ouvrez **SendMailViewController.m.** dans le dossier Controllers et ajoutez la méthode d’assistance suivante à la classe :

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


2. Ouvrez **SendMailViewController.m.** Ajoutez la méthode d’envoi d’e-mail suivante à la classe.  

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

Par conséquent, **getSampleMessage** crée un exemple d’e-mail HTML brouillon à utiliser à des fins de démonstration. La méthode suivante, **sendMail**, prend ensuite ce message et exécute la demande pour l’envoyer. Encore une fois, le destinataire par défaut est l’utilisateur connecté.


## <a name="run-the-app"></a>Exécuter l’application
1. Avant d’exécuter l’exemple, vous devrez fournir l’ID client que vous avez reçu du processus d’inscription dans la section **Inscription de l’application.** Ouvrez **AuthenticationConstants.m** sous le dossier **Application**. Vous verrez que l’ID client du processus d’inscription peut être ajouté à la partie supérieure du fichier :  

        // You will set your application's clientId
        NSString * const kClientId    = @"ENTER_CLIENT_ID_HERE";
        NSString * const kScopes = @"https://graph.microsoft.com/Mail.Send, https://graph.microsoft.com/User.Read, offline_access";
Remarque : Vous remarquerez que les étendues d’autorisations suivantes ont été configurées pour ce projet : **« https://graph.microsoft.com/Mail.Send », « https://graph.microsoft.com/User.Read », « offline_access »**. Les appels de service utilisés dans ce projet, l’envoi d’un courrier électronique à votre compte de messagerie et la récupération des informations de profil (nom d’affichage, adresse e-mail) ont besoin de ces autorisations pour que l’application s’exécute correctement.

2. Exécutez l’exemple, appuyez sur **Se connecter**, connectez-vous à votre compte personnel, professionnel ou scolaire et accordez les autorisations demandées.

3. Choisissez le bouton **Envoyer un message électronique**. Lorsque le message est envoyé, un message de réussite s’affiche sous le bouton.

## <a name="next-steps"></a>Étapes suivantes
- Testez l’API REST à l’aide de l’[Afficheur Graph](https://graph.microsoft.io/graph-explorer).
- Recherchez des exemples d’opérations courantes pour les opérations REST et SDK dans l’[xemple d’extraits de code Microsoft Graph iOS Objective C](https://github.com/microsoftgraph/ios-objectiveC-snippets-sample).

## <a name="see-also"></a>Voir aussi
- [Kit de développement logiciel (SDK) Microsoft Graph pour iOS](https://github.com/microsoftgraph/msgraph-sdk-ios)
- [Protocoles Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
- [Jetons Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)
