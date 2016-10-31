# <a name="get-started-with-microsoft-graph-in-an-angularjs-app"></a>AngularJS アプリで Microsoft Graph を使ってみる

この記事では、Azure AD v2.0 エンドポイントからアクセス トークンを取得し、Microsoft Graph を呼び出すために必要なタスクについて説明します。ここでは、[AngularJS 用の Microsoft Connect サンプル](https://github.com/microsoftgraph/angular-connect-rest-sample)の構築手順と、Microsoft Graph を使用するために実装する主要な概念について説明します。この記事では、生の REST 呼び出しを使用して Microsoft Graph API にアクセスする方法について説明します。

次の画像は、作成するアプリを示しています。 

![ログイン後に [メールの送信] ボタンを表示する Web アプリ](./images/angular-connect-sample.png)


**アプリを作成してみたくありませんか。**「[Microsoft Graph クイック スタート](https://graph.microsoft.io/en-us/getting-started)」を使用すれば、すばやく稼働させることができます。

Azure AD エンドポイントを使用するバージョンの Connect サンプルをダウンロードするには、「[AngularJS 用の Microsoft Graph Connect サンプル](https://github.com/microsoftgraph/angular-connect-rest-sample/releases/tag/last_v1_auth)」をご覧ください。


## <a name="prerequisites"></a>前提条件

開始するには、次のものが必要です。 

- [Microsoft アカウント](https://www.outlook.com/)か[職場または学校アカウント](http://dev.office.com/devprogram)
- [npm 付きの Node.js](https://nodejs.org/en/download/)
- [Bower](https://bower.io)
- [AngularJS 用 Microsoft Connect のサンプル](https://github.com/microsoftgraph/angular-connect-rest-sample)このチュートリアルには、サンプル ファイル内の **starter-project** フォルダーを使用します。

## <a name="register-the-application"></a>アプリケーションの登録
Microsoft アプリケーション登録ポータルでアプリケーションを登録します。これにより、アプリを Visual Studio で構成する際に使用するアプリ ID とパスワードが生成されます。

1. 個人用アカウント、あるいは職場または学校アカウントのいずれかを使用して、[Microsoft アプリ登録ポータル](https://apps.dev.microsoft.com/)にサインインします。

2. **[アプリの追加]** を選択します。

3. アプリの名前を入力して、**[アプリケーションの作成]** を選択します。 
    
    登録ページが表示され、アプリのプロパティが一覧表示されます。

4. アプリケーション ID をコピーします。これは、アプリの構成に使用するアプリの一意識別子です。

5. **[プラットフォーム]** で、**[プラットフォームの追加]** > **[Web]** の順に選びます。

7. **[暗黙的フローを許可する]** チェック ボックスが選ばれていることを確認して、リダイレクト URI として *http://localhost:8080/login* を入力します。 

8. **[保存]** を選びます。


## <a name="configure-the-project"></a>プロジェクトを構成する
1. サンプル ファイル内の **starter-project** フォルダーを開きます。
2. コマンド プロンプトで、スターター プロジェクトのルート ディレクトリから次のコマンドを実行します。これにより、プロジェクトの依存関係がインストールされます。

    npm install  bower install hello

3. スターター プロジェクトのファイル内の **public/scripts** フォルダーにある、config.js を開きます。
4. **clientId** フィールド内の **ENTER_YOUR_CLIENT_ID** プレースホルダーの値を、直前にコピーしたアプリケーション ID に置き換えます。

  
## <a name="authenticate-the-user-and-get-an-access-token"></a>ユーザーの認証とアクセス トークンの取得
この手順では、サインインとトークンの取得コードを追加します。しかし、まず認証フローをさらに詳しく見てみましょう。

このシングル ページ アプリケーションでは暗黙的な付与フローのごく基本的な実装を使用します。これには登録したアプリのアプリケーション ID とリダイレクト URI が必要です。 

認証フローは、以下の基本的な手順に分けることができます。

1. 認証と同意のためにユーザーをリダイレクトします。
2. アクセス トークンを取得します。

アプリは [HelloJS](https://adodson.com/hello.js) クライアント側ライブラリを使用して、トークンを認証、取得します。アプリはローカル ストレージにアクセス トークンを格納します。
    
   >**重要** このプロジェクトでは、サンプルの目的で単純な認証とトークンの処理を使用しているに過ぎません。運用アプリでは、検証やセキュリティで保護されたトークンの処理など、認証を処理するためのより信頼性の高い方法を構築する必要があります。

ここでアプリの作成に戻ります。

1- aad.js を開き、次のコードを追加します。これにより、Azure AD 認証プロバイダーとの通信が構成され、アクセス トークンを含む認証応答を格納するためのリスナーが追加されます。(HelloJS のスクリプト参照は index.html ビューに追加済みです。)

```
hello.init({
      
  aad: {
    name: 'Azure Active Directory', 
    oauth: {
      version: 2,
      auth: 'https://login.microsoftonline.com/common/oauth2/v2.0/authorize',
      grant: 'https://login.microsoftonline.com/common/oauth2/v2.0/token'
    },
    scope_delim: ' ',

    // Don't even try submitting via form.
    // This means no POST operations in <=IE9
    form: false
  }
});

hello.on('auth.login', function (auth) {

    // save the auth info into localStorage
    localStorage.auth = angular.toJson(auth.authResponse);
});
```

2- graphHelper.js で、*// Initialize the auth request* を次のコードに置き換えます。これにより、承認要求用のパラメーターが設定されます。

```
// Initialize the auth request.
hello.init( {
  aad: clientId // from public/scripts/config.js
  }, {
  redirect_uri: redirectUrl,
  scope: graphScopes
});
```

3- *// Sign in and sign out the user* を次のコードに置き換えます。**login** 関数では、HelloJS を使用して、トークンの情報を取得します。aad.js のリスナーは、アクセス トークンを含む、この情報をローカル ストレージに格納します。

```
// Sign in and sign out the user.
login: function login() {
  hello('aad').login({
    display: 'page',
    state: 'abcd'
  });
},
logout: function logout() {
  hello('aad').logout();
  delete localStorage.auth;
  delete localStorage.user;
},
```

これで、Microsoft Graph を呼び出すためのコードを追加する準備が整いました。 

## <a name="call-microsoft-graph"></a>Microsoft Graph を呼び出す
アプリは Microsoft Graph を呼び出してユーザー情報を取得し、ユーザーの代わりにメールを送信します。これらの呼び出しは UI イベントに応答して MainController から始まります。

1- graphHelper.js で、*// Get the profile of the current user* を次のコードに置き換えます。これにより GET 要求を構成して */me* エンドポイントに送信し、応答を処理します。

```
// Get the profile of the current user.
  me: function me() {
    return $http.get('https://graph.microsoft.com/v1.0/me');
},
```
  
2- *// Send an email on behalf of the current user* を次のコードに置き換えます。これにより POST 要求を構成して */me/sendMail* エンドポイントに送信し、応答を処理します。

```
// Send an email on behalf of the current user.
sendMail: function sendMail(email) {
  return $http.post('https://graph.microsoft.com/v1.0/me/sendMail', { 'message' : email, 'saveToSentItems': true });        
}
```

3- **public/controllers** フォルダーで、mainController.js を開きます。

4- *// Set the default headers and user properties* を次のコードに置き換えます。これにより HTTP 要求にアクセス トークンを追加し、**GraphHelper.me** を呼び出して現在のユーザーのプロファイルを取得し、応答を処理します。

```
// Set the default headers and user properties.
function processAuth() {
  let auth = angular.fromJson(localStorage.auth); 

  // Check token expiry. If the token is valid for another 5 minutes, we'll use it.       
  let expiration = new Date();
  expiration.setTime((auth.expires - 300) * 1000); 
  if (expiration > new Date()) {

    // Add the required Authorization header with bearer token.
    $http.defaults.headers.common.Authorization = 'Bearer ' + auth.access_token;
        
    // This header has been added to identify our sample in the Microsoft Graph service. If extracting this code for your project please remove.
    $http.defaults.headers.common.SampleID = 'angular-connect-rest-starter';
        
    if (localStorage.getItem('user') === null) {

      // Get the profile of the current user.
      GraphHelper.me().then(function(response) {

        // Save the user to localStorage.
        let user =response.data;
        localStorage.setItem('user', angular.toJson(user));

        vm.displayName = user.displayName;
        vm.emailAddress = user.mail || user.userPrincipalName;
      });
   } else {
     let user = angular.fromJson(localStorage.user);

     vm.displayName = user.displayName;
     vm.emailAddress = user.mail || user.userPrincipalName;
    }
  }
} 
```

5- *// Send an email on behalf of the current user* を次のコードに置き換えます。これにより、メール メッセージを作成し、**GraphHelper.sendMail** を呼び出して、応答を処理します。

```
// Send an email on behalf of the current user.
function sendMail() {

  // Check token expiry. If the token is valid for another 5 minutes, we'll use it.
  let auth = angular.fromJson(localStorage.auth);
  let expiration = new Date();
  expiration.setTime((auth.expires - 300) * 1000);
  if (expiration > new Date()) {
  
    // Build the HTTP request payload (the Message object).
    var email = {
        Subject: 'Welcome to Microsoft Graph development with AngularJS and the Microsoft Graph Connect sample',
        Body: {
            ContentType: 'HTML',
            Content: getEmailContent()
        },
        ToRecipients: [
            {
                EmailAddress: {
                    Address: vm.emailAddress
                }
            }
        ]
    };

    // Save email address so it doesn't get lost with two way data binding.
    vm.emailAddressSent = vm.emailAddress;

    GraphHelper.sendMail(email)
        .then(function (response) {
            $log.debug('HTTP request to the Microsoft Graph API returned successfully.', response);
            response.status === 202 ? vm.requestSuccess = true : vm.requestSuccess = false;
            vm.requestFinished = true;
        }, function (error) {
            $log.error('HTTP request to the Microsoft Graph API failed.');
            vm.requestSuccess = false;
            vm.requestFinished = true;
        });
    } else {

    // If the token is expired, this sample just redirects the user to sign in.
    GraphHelper.login();
    }
};

// Get the HTMl for the email to send.
function getEmailContent() {
  return "<html><head> <meta http-equiv=\'Content-Type\' content=\'text/html; charset=us-ascii\'> <title></title> </head><body style=\'font-family:calibri\'> <p>Congratulations " + vm.displayName + ",</p> <p>This is a message from the Microsoft Graph Connect sample. You are well on your way to incorporating Microsoft Graph endpoints in your apps. </p> <h3>What&#8217;s next?</h3><ul><li>Check out <a href='https://graph.microsoft.io' target='_blank'>graph.microsoft.io</a> to start building Microsoft Graph apps today with all the latest tools, templates, and guidance to get started quickly.</li><li>Use the <a href='https://graph.microsoft.io/graph-explorer' target='_blank'>Graph explorer</a> to explore the rest of the APIs and start your testing.</li><li>Browse other <a href='https://github.com/microsoftgraph/' target='_blank'>samples on GitHub</a> to see more of the APIs in action.</li></ul> <h3>Give us feedback</h3> <ul><li>If you have any trouble running this sample, please <a href='https://github.com/microsoftgraph/angular-connect-rest-sample/issues' target='_blank'>log an issue</a>.</li><li>For general questions about the Microsoft Graph API, post to <a href='https://stackoverflow.com/questions/tagged/microsoftgraph?sort=newest' target='blank'>Stack Overflow</a>. Make sure that your questions or comments are tagged with [microsoftgraph].</li></ul><p>Thanks and happy coding!<br>Your Microsoft Graph samples development team</p> <div style=\'text-align:center; font-family:calibri\'> <table style=\'width:100%; font-family:calibri\'> <tbody> <tr> <td><a href=\'https://github.com/microsoftgraph/angular-connect-rest-sample\'>See on GitHub</a> </td> <td><a href=\'https://officespdev.uservoice.com/\'>Suggest on UserVoice</a> </td> <td><a href=\'https://twitter.com/share?text=I%20just%20started%20developing%20%23Angular%20apps%20using%20the%20%23MicrosoftGraph%20Connect%20sample!%20&url=https://github.com/microsoftgraph/angular-connect-rest-sample\'>Share on Twitter</a> </td> </tr> </tbody> </table> </div>  </body> </html>";
};
```

6- 変更をすべて保存します。

## <a name="run-the-app"></a>アプリの実行

1- コマンド プロンプトで、スターター プロジェクトのルート ディレクトリから次のコマンドを実行します。

```
npm start
```

2- ブラウザーで *http://localhost:8080* にナビゲートし **[Connect]** ボタンを選びます。

3- サインインして要求されたアクセス許可を付与します。 

4-.必要に応じて、受信者のメール アドレスを編集してから、**[メールの送信]** ボタンを選びます。メールが送信されると、ボタンの下に成功メッセージが表示されます。 

## <a name="next-steps"></a>次の手順
- [Graph エクスプローラー](https://graph.microsoft.io/graph-explorer)を使用して REST API を試してみます。
- GitHub で他の [AngularJS サンプル](https://github.com/search?utf8=%E2%9C%93&q=angular+sample+user%3Amicrosoftgraph&type=Repositories&ref=searchresults)を探します。


## <a name="see-also"></a>関連項目
- [Azure AD v2.0 のプロトコル](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
- [Azure AD v2.0 のトークン](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)