# <a name="get-started-with-microsoft-graph-in-a-ruby-on-rails-app"></a>Prise en main de Microsoft Graph dans une application Ruby on Rails

Cet article décrit les tâches requises pour obtenir un jeton d’accès du point de terminaison Azure AD v2.0 et appeler Microsoft Graph. Il vous guidera dans la construction de l’[exemple de connexion Microsoft Graph pour Ruby on Rails](https://github.com/microsoftgraph/ruby-connect-rest-sample) et décrit les principaux concepts à mettre en œuvre pour utiliser Microsoft Graph. Cet article explique également comment accéder à Microsoft Graph à l’aide d’appels REST directs.

Pour télécharger une version de l’exemple de connexion qui utilise le point de terminaison Azure AD, voir [Exemple de connexion Microsoft Graph pour Ruby on Rails](https://github.com/microsoftgraph/ruby-connect-rest-sample/tree/last_v1_auth).

L’image suivante présente l’application que vous allez créer. 

![Capture d’écran de l’exemple de connexion Microsoft Ruby on Rails](./images/Microsoft-Graph-Ruby-Connect-UI.png)

**Pas envie de créer une application ?** Utilisez le [démarrage rapide de Microsoft Graph](https://graph.microsoft.io/en-us/getting-started) pour devenir opérationnel rapidement ou téléchargez l’[exemple de connexion REST Ruby](https://github.com/microsoftgraph/ruby-connect-rest-sample) sur lequel repose cet article.


## <a name="prerequisites"></a>Conditions préalables

Voici ce dont vous aurez besoin pour démarrer : 

- Ruby 2.1 pour exécuter l’exemple sur un serveur de développement.
- La structure Rails (l’exemple a été testé sur Rails 4.2).
- Le gestionnaire de dépendances Bundler.
- L’interface de serveur web Rack pour Ruby.
- Un [compte Microsoft](https://www.outlook.com/) ou un [compte professionnel ou scolaire](http://dev.office.com/devprogram)
- Le projet de lancement de connexion avec Microsoft Graph pour Ruby on Rails. Téléchargez l’[exemple de connexion avec Microsoft Graph pour Ruby on Rails](https://github.com/microsoftgraph/ruby-connect-rest-sample). Le projet de lancement se trouve dans le dossier _starter_.


## <a name="register-the-application"></a>Inscription de l’application

Inscrivez une application sur le portail d’inscription des applications Microsoft. L’inscription génère l’ID d’application et la question secrète de l’application que vous utiliserez pour configurer l’application pour l’authentification.

1. Connectez-vous au [portail d’inscription des applications Microsoft](https://apps.dev.microsoft.com/) en utilisant votre compte personnel, professionnel ou scolaire.

2. Choisissez **Ajouter une application**.

3. Entrez un nom pour l’application, puis choisissez **Créer une application**.

    La page d’inscription s’affiche, répertoriant les propriétés de votre application.

4. Copiez l’ID de l’application. Il s’agit de l’identificateur unique de votre application.

5. Sous **Secrets de l'application**, choisissez **Générer un nouveau mot de passe**. Copiez la question secrète de l’application à partir de la boîte de dialogue **Nouveau mot de passe créé**.

    Vous utiliserez l’ID de l’application et la question secrète de l’application pour configurer l’application.

6. Sous **Plateformes**, choisissez **Ajouter une plateforme** > **Web**.

7. Assurez-vous que la case **Autoriser le flux implicite** est cochée, puis entrez *http://localhost:3000/auth/microsoft_v2_auth/callback* comme URI de redirection.

    L’option Autoriser le flux implicite active le flux hybride OpenID Connect. Lors de l’authentification, cela permet à l’application de recevoir les informations de connexion (id_token) et les artefacts (dans ce cas, un code d’autorisation) qui servent à obtenir un jeton d’accès.

    L’URI de redirection *http://localhost:3000/auth/microsoft_v2_auth/callback* est la valeur que doit utiliser l’intergiciel OmniAuth une fois qu’il a traité la demande d’authentification.

8. Cliquez sur **Enregistrer**.

## <a name="configure-the-project"></a>Configurer le projet

1. Téléchargez ou clonez l’[exemple de connexion Microsoft Graph pour Ruby on Rails](https://github.com/microsoftgraph/ruby-connect-rest-sample). Ouvrez le dossier _starter_ dans l’éditeur de votre choix.
1. Si vous n’avez pas encore Bundler et Rack sur votre ordinateur, vous pouvez les installer avec la commande suivante.

    ```
    gem install bundler rack
    ```
2. Dans le fichier [config/environment.rb](config/environment.rb), suivez les étapes ci-dessous :
    - Remplacez *ENTER_YOUR_CLIENT_ID* par l’ID client de votre application inscrite.
    - Remplacez *ENTER_YOUR_SECRET* par la clé de votre application inscrite.

3. Installez l’application Rails et les dépendances avec la commande suivante.

    ```
    bundle install
    ```

## <a name="authenticate-the-user-and-get-an-access-token"></a>Authentifier l’utilisateur et obtenir un jeton d’accès

Cette application utilise le flux d’octroi de code d’autorisation avec une identité utilisateur déléguée. Pour une application web, le flux requiert l’ID de l’application, la clé secrète et l’URI de redirection de l’application inscrite. 

Le flux d’authentification comprend les étapes de base suivantes :

1. Rediriger l’utilisateur pour l’authentification et le consentement
2. Obtention d’un code d’autorisation
3. Obtenir le code d’autorisation pour un jeton d’accès

>Pour plus d’informations sur ce flux d’authentification, voir [Application web vers API web](https://azure.microsoft.com/en-us/documentation/articles/active-directory-authentication-scenarios/#web-application-to-web-api) et [Intégrer l’identité Microsoft et Microsoft Graph dans une application web avec OpenID Connect](https://azure.microsoft.com/en-us/documentation/samples/active-directory-dotnet-webapp-openidconnect-v2/) dans la documentation Azure AD.

Nous utilisons une pile de trois éléments d’intergiciel [Rack](http://rack.github.io/) pour permettre à l’application de s’authentifier sur Microsoft Graph :

- [OmniAuth](https://rubygems.org/gems/omniauth), un cadre Rack généralisé pour l’authentification de fournisseurs multiples.
- [Omniauth-oauth2](https://rubygems.org/gems/omniauth-oauth2), une stratégie OAuth2 abstraite pour OmniAuth. 
- omniauth microsoft_v2_auth, une stratégie OmniAuth qui personnalise Omniauth-oauth2 pour fournir une authentification spécifiquement sur le point de terminaison Azure AD v2.0. Ce projet est inclus dans l’exemple de code.

### <a name="specify-gem-dependencies-for-authentication"></a>Spécifier des dépendances gem pour l’authentification

Dans Gemfile, supprimez les marques de commentaires des gem suivants pour les ajouter en tant que dépendances.

    ```
    gem 'omniauth'
    gem 'omniauth-oauth2'
    gem 'omniauth-microsoft_v2_auth', path: './omniauth-microsoft_v2_auth'
    ```

Notez que `omniauth-microsoft_v2_auth` est inclus dans le projet d’application et sera installé à partir du chemin d’accès spécifié. 

### <a name="configure-the-authentication-middleware"></a>Configurer l’intergiciel d’authentification

Dans `config/initializers/omniauth-microsoft_v2_auth.rb`, supprimez les marques de commentaires des lignes suivantes.

    ```
    Rails.application.config.middleware.use OmniAuth::Builder do
      provider :microsoft_v2_auth,
      ENV['CLIENT_ID'],
      ENV['CLIENT_SECRET'],
      :scope => ENV['SCOPE']
    end
    ```
Cela permet de configurer l’intergiciel OmniAuth et de spécifier l’ID d’application et la question secrète de l’application à utiliser, ainsi que les étendues à demander pour l’utilisateur. Ce sont les valeurs que vous avez spécifiées précédemment dans `config/environment.rb`.

### <a name="specify-routes-for-authentication"></a>Spécifier des itinéraires pour l’authentification

Nous devons maintenant spécifier deux itinéraires nécessaires pour le flux d’authentification. Le premier itinéraire transfère la demande d’authentification à l’intergiciel OmniAuth et le second indique l’emplacement dans l’application auquel OmniAuth doit rediriger une fois que l’authentification a eu lieu.

Dans `config/routes.rb`, supprimez les marques de commentaires de la directive d’itinéraire suivante.

    get '/login', to: 'pages#login'

Ceci dirige les demandes de connexion à la méthode `login` du contrôleur des pages, qui à son tour redirige la demande à l’intergiciel omniauth-microsoft_v2_auth.

    def login
        redirect_to '/auth/microsoft_v2_auth'
    end

Ensuite, nous devons indiquer l’emplacement de redirection d’OmniAuth dans l’application une fois que l’authentification a eu lieu. Supprimez les marques de commentaires de l’itinéraire suivant.

    match '/auth/:provider/callback', to: 'pages#callback', via: [:get, :post]

Une fois qu’OmniAuth a terminé d’authentifier l’utilisateur, il appelle l’URL de redirection spécifiée dans l’inscription de l’application ; dans ce cas, *http://localhost:3000/auth/microsoft_v2_auth/callback*. Le modèle d’itinéraire ci-dessus correspond à cette URL et par conséquent achemine la demande vers la méthode `callback` du contrôleur de page.

### <a name="get-an-access-token"></a>Obtenir un jeton d’accès

Ensuite, nous allons ajouter le code qui démarre réellement le processus d’authentification et récupère le jeton d’accès une fois que l’utilisateur s’est correctement connecté.

Examinons `app/views/pages/index.html.erb`, l’affichage de la racine du site. L’affichage inclut un bouton unique, qui permet aux utilisateurs de se connecter.

    <button class="ms-Button" onclick="window.location.href = '/login'">
        <span class="ms-Button-label"><%= t('connect_button') %></span>
    </button>

Comme indiqué précédemment, la méthode de connexion redirige vers l’intergiciel OmniAuth, qui a été configuré avec l’ID d’application et la question secrète de l’application, ainsi que les étendues à demander pour l’utilisateur. Une fois que l’utilisateur est authentifié, OmniAuth renvoie un code de hachage avec le jeton d’accès et d’autres informations utilisateur à l’application.

Nous allons maintenant ajouter le code pour gérer le callback OmniAuth et récupérer les informations de ce code de hachage. 

Dans `app/controllers/pages_controller.rb`, remplacez la méthode `callback` vide par le code suivant.

    ```
    def callback
        # Access the authentication hash for omniauth
        # and extract the auth token, user name, and email
        data = request.env['omniauth.auth']
    
        @email = data[:extra][:raw_info][:userPrincipalName]
        @name = data[:extra][:raw_info][:displayName]

        # Associate token/user values to the session
        session[:access_token] = data['credentials']['token']
        session[:name] = @name
        session[:email] = @email
        
        # Debug logging
        logger.info "Name: #{@name}"
        logger.info "Email: #{@email}"
        logger.info "[callback] - Access token: #{session[:access_token]}"
    end

    ```

Cette méthode récupère le code de hachage d’authentification, puis stocke le jeton d’accès, le nom d’utilisateur et l’e-mail dans la session en cours.

> **Remarque :** la gestion de jeton et d’authentification simple dans ce projet est fournie à titre d’exemple uniquement. Dans une application de production, vous devez probablement mettre au point une méthode de gestion de l’authentification plus fiable (y compris l’actualisation et la gestion de jetons sécurisée).

## <a name="call-microsoft-graph"></a>Appeler Microsoft Graph

Vous êtes maintenant prêt à ajouter un code pour appeler Microsoft Graph. 

L’affichage rendu par la méthode `callback` (`app/views/pages/callback.html.erb`) contient un formulaire simple avec un seul bouton. Le formulaire publie dans `send_mail` et inclut un seul paramètre, l’adresse e-mail du destinataire souhaité.
    
    ``` 
    <form action="../../send_mail" method="post">
      <div class="ms-Grid-col ms-u-mdPush1 ms-u-md9 ms-u-lgPush1 ms-u-lg6">
        ...
            <div class="ms-TextField">
               <input class="ms-TextField-field" name="specified_email" value="<%= @email %>">
            </div>
            <button class="ms-Button">
            <span class="ms-Button-label"><i class="ms-Icon ms-Icon--mail" aria-hidden="true"></i><%= t('send_mail_button') %></span>
            </button> 
        ...
    ```

Dans `app/controllers/pages_controller.rb`, remplacez la méthode `send_mail` vide par le code suivant.

    ```
    def send_mail
        logger.debug "[send_mail] - Access token: #{session[:access_token]}"
        
        # Used in the template
        @name = session[:name]
        @email = params[:specified_email]
        @recipient = params[:specified_email]
        @mail_sent = false
        
        send_mail_endpoint = URI("#{GRAPH_RESOURCE}#{SENDMAIL_ENDPOINT}")
        content_type = CONTENT_TYPE
        http = Net::HTTP.new(send_mail_endpoint.host, send_mail_endpoint.port)
        http.use_ssl = true
        
        # If you want to use a sniffer tool, like Fiddler, to see the request
        # you might need to add this line to tell the engine not to verify the
        # certificate or you might see a "certificate verify failed" error
        # http.verify_mode = OpenSSL::SSL::VERIFY_NONE
        
        email_body = File.read('app/assets/MailTemplate.html')
        email_body.sub! '{given_name}', @name
        email_subject = t('email_subject')
        
        logger.debug email_body
    
        email_message = "{
            Message: {
            Subject: '#{email_subject}',
            Body: {
                ContentType: 'HTML',
                Content: '#{email_body}'
            },
            ToRecipients: [
                {
                    EmailAddress: {
                        Address: '#{@recipient}'
                    }
                }
            ]
            },
            SaveToSentItems: true
            }"
            
        response = http.post(
            SENDMAIL_ENDPOINT,
            email_message,
            'Authorization' => "Bearer #{session[:access_token]}",
            'Content-Type' => content_type
        )
        
        logger.debug "Code: #{response.code}"
        logger.debug "Message: #{response.message}"
        
        # The send mail endpoint returns a 202 - Accepted code on success
        if response.code == '202'
            @mail_sent = true
        else
            @mail_sent = false
            flash[:httpError] = "#{response.code} - #{response.message}"
        end
        
        render 'callback'
    end
    ```

Ce code construit la requête HTTP, met en forme l’e-mail, puis appelle Microsoft Graph pour envoyer l’e-mail.

Pour créer l’e-mail, le code extrait le nom d’utilisateur du jeton de la session et l’adresse e-mail du destinataire des paramètres transférés du formulaire. Le code lit ensuite le corps de l’e-mail à partir d’un modèle inclus dans le projet, interpole l’adresse e-mail et le nom d’utilisateur puis joint le texte de l’e-mail en tant que corps de la requête HTTP.

Pour envoyer l’e-mail, le code construit la requête HTTP, joint le jeton d’accès sous la forme d’un en-tête d’autorisation puis publie la requête au point de terminaison Envoyer un message électronique.

Pour finir, le code utilise le code de réponse HTTP renvoyé pour indiquer à l’utilisateur si l’e-mail a réussi ou non.

## <a name="run-the-app"></a>Exécuter l’application

1. Installez l’application Rails et les dépendances avec la commande suivante.

    ```
    bundle install
    ```
2. Pour démarrer l’application Rails, entrez la commande suivante.

    ```
    rackup -p 3000
    ```
3. Accédez à `http://localhost:3000` dans votre navigateur web.

## <a name="see-also"></a>Voir aussi
- Testez l’API REST à l’aide de l’[Afficheur Graph](https://graph.microsoft.io/graph-explorer).
- Explorez nos autres [exemples Microsoft Graph](https://github.com/microsoftgraph) sur GitHub.


