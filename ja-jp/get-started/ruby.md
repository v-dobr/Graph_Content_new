# <a name="get-started-with-microsoft-graph-in-a-ruby-on-rails-app"></a>Ruby on Rails アプリで Microsoft Graph を使ってみる

この記事では、Azure AD v2.0 エンドポイントからアクセス トークンを取得し、Microsoft Graph を呼び出すために必要なタスクについて説明します。ここでは、[Microsoft Graph Ruby on Rails Connect のサンプル](https://github.com/microsoftgraph/ruby-connect-rest-sample)を構築する手順と、Microsoft Graph を使用するために実装する主要な概念について説明します。また、直接 REST 呼び出しを使用して Microsoft Graph にアクセスする方法についても説明します。

Azure AD エンドポイントを使用するバージョンの Connect サンプルをダウンロードするには、「[Microsoft Graph Ruby on Rails Connect サンプル](https://github.com/microsoftgraph/ruby-connect-rest-sample/tree/last_v1_auth)」を参照してください。

次の画像は、作成するアプリを示しています。 

![Microsoft Ruby on Rails Connect サンプルのスクリーンショット](./images/Microsoft-Graph-Ruby-Connect-UI.png)

**アプリを作成してみたくありませんか。**[Microsoft Graph クイック スタート](https://graph.microsoft.io/en-us/getting-started)を使用してすぐに使い始めるか、またはこの記事で取り扱っている [Ruby REST Connect サンプル](https://github.com/microsoftgraph/ruby-connect-rest-sample)をダウンロードしてください。


## <a name="prerequisites"></a>前提条件

開始するには、次のものが必要です。 

- 開発サーバー上でサンプルを実行する Ruby 2.1。
- Rails フレームワーク (このサンプルは Rails 4.2 でテストされています)。
- Bundler 依存関係マネージャー。
- Ruby 用の Rack Web サーバー インターフェイス。
- [Microsoft アカウント](https://www.outlook.com/)か[職場または学校アカウント](http://dev.office.com/devprogram)
- Ruby on Rails 用の Microsoft Graph Connect スターター プロジェクト。[Microsoft Graph Ruby on Rails Connect サンプル](https://github.com/microsoftgraph/ruby-connect-rest-sample)をダウンロードします。スタート プロジェクトは_スターター_ フォルダー内にあります。


## <a name="register-the-application"></a>アプリケーションの登録

Microsoft アプリケーション登録ポータルでアプリケーションを登録します。これにより、アプリの認証構成に使用するアプリ ID とシークレットが生成されます。

1. 個人用アカウント、あるいは職場または学校アカウントのいずれかを使用して、[Microsoft アプリ登録ポータル](https://apps.dev.microsoft.com/)にサインインします。

2. **[アプリの追加]** を選択します。

3. アプリの名前を入力して、**[アプリケーションの作成]** を選択します。

    登録ページが表示され、アプリのプロパティが一覧表示されます。

4. アプリケーション ID をコピーします。これは、アプリの一意識別子です。

5. **[アプリケーション シークレット]** で、**[新しいパスワードを生成する]** を選択します。**[新しいパスワードが生成されました]** ダイアログからアプリ シークレットをコピーします。

    アプリを構成するには、アプリケーション ID とアプリ シークレットを使用します。

6. **[プラットフォーム]** で、**[プラットフォームの追加]** > **[Web]** の順に選択します。

7. **[暗黙的フローを許可する]** のチェック ボックスが選択されていることを確認して、リダイレクト URI として「*http://localhost:3000/auth/microsoft_v2_auth/callback*」と入力します。

    [暗黙的フローを許可する] オプションにより、OpenID Connect ハイブリッド フローが有効になります。これにより、認証時にアプリはサインイン情報 (id_token) と成果物 (この場合は認証コード) の両方を受け取れるようになり、アプリはアクセス トークンを取得するときにこれらを使用できます。

    リダイレクト URI *http://localhost:3000/auth/microsoft_v2_auth/callback* は、認証要求が処理されたときに OmniAuth ミドルウェアが使用するように構成される値です。

8. **[保存]** を選びます。

## <a name="configure-the-project"></a>プロジェクトを構成する

1. [Microsoft Graph Ruby on Rails Connect サンプル](https://github.com/microsoftgraph/ruby-connect-rest-sample)をダウンロードするか、複製を作成します。任意のエディターで、_スターター_ フォルダーを開きます。
1. Bundler と Rack がない場合は、次のコマンドでインストールできます。

    ```
    gem install bundler rack
    ```
2. [config/environment.rb](config/environment.rb) ファイルで、次を実行します。
    - *ENTER_YOUR_CLIENT_ID* を登録済みのアプリケーションのクライアント ID と置き換えます。
    - *ENTER_YOUR_SECRET* を登録済みのアプリケーションのキーと置き換えます。

3. 次のコマンドを使って、Rails アプリケーションと依存関係をインストールします。

    ```
    bundle install
    ```

## <a name="authenticate-the-user-and-get-an-access-token"></a>ユーザーの認証とアクセス トークンの取得

このアプリは、委任されたユーザー ID で認証コードの付与フローを使用します。Web アプリケーションのフローでは、登録したアプリのアプリケーション ID、シークレット、およびリダイレクト URI が必要です。 

認証フローは、以下の基本的な手順に分けることができます。

1. 認証と同意のためにユーザーをリダイレクトする
2. 認証コードを取得する
3. 認証コードをアクセス トークンに交換する

>この認証フローの詳細については、Azure AD のドキュメントにある「[Web アプリケーション対 Web API](https://azure.microsoft.com/en-us/documentation/articles/active-directory-authentication-scenarios/#web-application-to-web-api)」および「[OpenID Connect を使用した Microsoft ID と Microsoft Graph の Web アプリケーションへの統合](https://azure.microsoft.com/en-us/documentation/samples/active-directory-dotnet-webapp-openidconnect-v2/)」を参照してください。

[Rack](http://rack.github.io/) ミドルウェアの積み重ねられた 3 つのピースを使用して、アプリが Microsoft Graph に対して認証できるようにします。

- [OmniAuth](https://rubygems.org/gems/omniauth): 複数プロバイダー認証用の汎用 Rack フレームワーク。
- [Omniauth oauth2](https://rubygems.org/gems/omniauth-oauth2): OmniAuth 用の抽象的な OAuth2 戦略。 
- omniauth-microsoft_v2_auth: 特に Azure AD v2.0 エンドポイントに対する認証を提供するために Omniauth-oauth2 をカスタマイズする OmniAuth 戦略。このプロジェクトにはコード サンプルが含まれます。

### <a name="specify-gem-dependencies-for-authentication"></a>認証に対して gem 依存関係を指定します。

Gemfile では、次の gem のコメントを解除して、依存関係として追加します。

    ```
    gem 'omniauth'
    gem 'omniauth-oauth2'
    gem 'omniauth-microsoft_v2_auth', path: './omniauth-microsoft_v2_auth'
    ```

`omniauth-microsoft_v2_auth` はアプリ プロジェクトに含まれ、指定したパスからインストールされることに注意してください。 

### <a name="configure-the-authentication-middleware"></a>アプリケーション ミドルウェアを構成する

`config/initializers/omniauth-microsoft_v2_auth.rb` で、次の行のコメントを解除します。

    ```
    Rails.application.config.middleware.use OmniAuth::Builder do
      provider :microsoft_v2_auth,
      ENV['CLIENT_ID'],
      ENV['CLIENT_SECRET'],
      :scope => ENV['SCOPE']
    end
    ```
これにより、使用するアプリ ID とアプリ シークレットの指定、またユーザーの要求範囲など、OmniAuth ミドルウェアが構成されます。これらは `config/environment.rb` で以前指定した値です。

### <a name="specify-routes-for-authentication"></a>認証のルートを指定する

ここで、認証フローに必要な 2 つのルートを指定する必要があります。最初のルートは認証要求を OmniAuth ミドルウェアに転送し、2 つめのルートは認証が実行されたときに OmniAuth がリダイレクトするアプリ内の場所を指定します。

`config/routes.rb` で、次のルート ディレクティブのコメントを解除します。

    get '/login', to: 'pages#login'

これにより、ログイン要求がページ コントローラーの `login` メソッドに転送され、さらに要求が omniauth-microsoft_v2_auth  ミドルウェアにリダイレクトされます。

    def login
        redirect_to '/auth/microsoft_v2_auth'
    end

次に、認証が実行されたときに OmniAuth がリダイレクトするアプリ内の場所を指定する必要があります。次のルートのコメントを解除します。

    match '/auth/:provider/callback', to: 'pages#callback', via: [:get, :post]

OmniAuth がユーザーの認証を完了すると、アプリ登録で指定したダイレクト URL を呼び出します。この例では、*http://localhost:3000/auth/microsoft_v2_auth/callback* です。上記のルート パターンはその URL に一致するので、要求はページ コントローラーの `callback` メソッドにルーティングされます。

### <a name="get-an-access-token"></a>アクセス トークンを取得する

次に、認証プロセスを実際に開始し、ユーザーが正常にサインインするとアクセス トークンを取得するコードを追加します。

サイト ルートのビューである、`app/views/pages/index.html.erb` を確認してください。ビューには、ユーザーがサインインできるようにする 1 つのボタンが含まれています。

    <button class="ms-Button" onclick="window.location.href = '/login'">
        <span class="ms-Button-label"><%= t('connect_button') %></span>
    </button>

前述のように、ログイン メソッドにより OmniAuth ミドルウェアにリダイレクトされます。このミドルウェアは、アプリ ID とアプリ シークレットに加えて、ユーザーの要求範囲で構成されています。ユーザーが正常に認証されると、OmniAuth は、アクセス トークンと他のユーザー情報と一緒にハッシュをアプリに返します。

それでは、OmniAuth コールバックを処理するコードを追加して、そのハッシュから情報を取得しましょう。 

`app/controllers/pages_controller.rb` で、空の `callback` メソッドを次のコードに置き換えます。

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

このメソッドは認証ハッシュを取得し、現在のセッションのアクセス トークン、ユーザー名、および電子メールを保存します。

> **注:**このプロジェクトでの簡易認証とトークンの処理は例示のみを目的として示されています。運用アプリでは、セキュリティで保護されたトークンの処理およびトークンの更新など、認証を処理するためのより信頼性の高い方法を構築する可能性があります。

## <a name="call-microsoft-graph"></a>Microsoft Graph を呼び出す

これで、Microsoft Graph を呼び出すためのコードを追加する準備が整いました。 

`callback` メソッド (`app/views/pages/callback.html.erb`) で表示されるビューには、ボタン 1 つの簡単なフォームが含まれています。フォームは、`send_mail` に投稿するものであり、対象の受信者の電子メール アドレスである、1 つのパラメーターが含まれています。
    
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

`app/controllers/pages_controller.rb` で、空の `send_mail` メソッドを次のコードに置き換えます。

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

このコードは HTTP 要求を作成して、電子メールの書式を設定し、電子メールを送信する Microsoft Graph を呼び出します。

電子メールを作成するために、コードはセッション トークンからユーザー名と、フォームから渡されたパラメーターから受信者の電子メール アドレスを取得します。次に、コードはプロジェクトに含まれるテンプレートから電子メールの本文を読み取り、ユーザー名と電子メール アドレスを補間し、HTTP 要求の本文として電子メールのテキストを添付します。

電子メールを送信するために、コードは HTTP 要求を作成し、認証ヘッダーとしてアクセス トークンを添付し、電子メール送信のエンドポイントに要求を投稿します。

最後に、コードは電子メールが成功したかどうかをユーザーに通知するために返される HTTP 応答コードを使用します。

## <a name="run-the-app"></a>アプリの実行

1. 次のコマンドを使って、Rails アプリケーションと依存関係をインストールします。

    ```
    bundle install
    ```
2. Rails アプリケーションを起動するには、次のコマンドを入力します。

    ```
    rackup -p 3000
    ```
3. Web ブラウザーで `http://localhost:3000` にアクセスします。

## <a name="see-also"></a>関連項目
- [Graph エクスプローラー](https://graph.microsoft.io/graph-explorer)を使用して REST API を試してみます。
- GitHub の他の [Microsoft Graph サンプル](https://github.com/microsoftgraph)を探索します。


