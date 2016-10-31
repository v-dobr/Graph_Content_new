# <a name="get-started-with-microsoft-graph-in-a-node.js-app"></a>Node.js アプリで Microsoft Graph を使ってみる

この記事では、Azure AD v2.0 エンドポイントからアクセス トークンを取得し、Microsoft Graph を呼び出すために必要なタスクについて説明します。ここでは、[Node.js 用の Microsoft Connect サンプル](https://github.com/microsoftgraph/nodejs-connect-rest-sample)の構築手順と、Microsoft Graph を使用するために実装する主要な概念について説明します。この記事では、生の REST 呼び出しを使用して Microsoft Graph API にアクセスする方法について説明します。

次の画像は、作成するアプリを示しています。 

![ログイン後に [メールの送信] ボタンを表示する Web アプリ](./images/web-screenshot.png)


**アプリを作成してみたくありませんか。**「[Microsoft Graph クイック スタート](https://graph.microsoft.io/en-us/getting-started)」を使用すれば、すばやく稼働させることができます。

Azure AD エンドポイントを使用するバージョンの Connect サンプルをダウンロードするには、「[Node.js 用の Microsoft Graph Connect サンプル](https://github.com/microsoftgraph/nodejs-connect-rest-sample/releases/tag/last_v1_auth)」をご覧ください。


## <a name="prerequisites"></a>前提条件

開始するには、次のものが必要です。 

- [Microsoft アカウント](https://www.outlook.com/)か[職場または学校アカウント](http://dev.office.com/devprogram)
- [npm 付きの Node.js](https://nodejs.org/en/download/) 
- [Node.js 用 Microsoft Connect のサンプル](https://github.com/microsoftgraph/nodejs-connect-rest-sample)このチュートリアルには、サンプル ファイル内の **starter-project** フォルダーを使用します。

## <a name="register-the-application"></a>アプリケーションの登録
Microsoft アプリケーション登録ポータルでアプリケーションを登録します。これにより、アプリを Visual Studio で構成する際に使用するアプリ ID とパスワードが生成されます。

1. 個人用アカウント、あるいは職場または学校アカウントのいずれかを使用して、[Microsoft アプリ登録ポータル](https://apps.dev.microsoft.com/)にサインインします。

2. **[アプリの追加]** を選択します。

3. アプリの名前を入力して、**[アプリケーションの作成]** を選択します。 
    
    登録ページが表示され、アプリのプロパティが一覧表示されます。

4. アプリケーション ID をコピーします。これは、アプリの一意識別子です。 

5. **[アプリケーション シークレット]** で、**[新しいパスワードを生成する]** を選びます。**[新しいパスワードが生成されました]** ダイアログからパスワードをコピーします。

    アプリを構成するには、アプリケーション ID とアプリケーション パスワード (シークレット) を使用します。 

6. **[プラットフォーム]** で、**[プラットフォームの追加]** > **[Web]** の順に選びます。

7. リダイレクト URI には、*http://localhost:3000/login* と入力します。 

8. **[保存]** を選びます。


## <a name="configure-the-project"></a>プロジェクトを構成する
1. サンプル ファイル内の **starter-project** フォルダーを開きます。

1. コマンド プロンプトで、スターター プロジェクトのルート ディレクトリから次のコマンドを実行します。これにより、プロジェクトの依存関係がインストールされます。

        npm install

1. スターター プロジェクト ファイル内の authHelper.js を開きます。


1. **credentials** フィールド内の **ENTER\_YOUR\_CLIENT\_ID** プレースホルダーと **ENTER\_YOUR\_SECRET** プレースホルダーの値を、直前にコピーした値に置き換えます。

  
## <a name="authenticate-the-user-and-get-an-access-token"></a>ユーザーの認証とアクセス トークンの取得
この手順では、サインインとトークンの管理コードを追加します。しかし、まず認証フローをさらに詳しく見てみましょう。

このアプリは、委任されたユーザー ID で認証コードの付与フローを使用します。Web アプリケーションのフローでは、登録したアプリのアプリケーション ID、シークレット、およびリダイレクト URI が必要です。 

認証フローは、以下の基本的な手順に分けることができます。

1. 認証と同意のためにユーザーをリダイレクトする
2. 認証コードを取得する
3. 認証コードをアクセス トークンに交換する
4. アクセス トークンの有効期限が切れたら、更新トークンを使用して新しいアクセス トークンを取得する

アプリは [oauth](https://www.npmjs.com/package/oauth) ミドルウェアを使用して認証し、トークンを取得します。アプリは [cookie-parser](https://www.npmjs.com/package/cookie-parser) ミドルウェアを使用して、トークン情報をクッキーにキャッシュします。トークン情報の格納とアクセスに使用されるコードは、index.js コントローラーにあります。
    
   >**重要** このプロジェクトでは、サンプルの目的で単純な認証とトークンの処理を使用しているに過ぎません。運用アプリでは、検証やセキュリティで保護されたトークンの処理など、認証を処理するためのより信頼性の高い方法を構築する必要があります。

ここでアプリの作成に戻ります。

1. authHelper.js 内の *getTokenFromCode* 関数を次のコードに置き換えます。これにより、認証コードを使用してアクセス トークンを取得します。

        function getTokenFromCode(code, callback) {
            var OAuth2 = OAuth.OAuth2;
            var oauth2 = new OAuth2(
                credentials.client_id,
                credentials.client_secret,
                credentials.authority,
                credentials.authorize_endpoint,
                credentials.token_endpoint
            );

            oauth2.getOAuthAccessToken(
                code,
                {
                    grant_type: 'authorization_code',
                    redirect_uri: credentials.redirect_uri,
                    response_mode: 'form_post',
                    nonce: uuid.v4(),
                    state: 'abcd'
                },
                function (e, accessToken, refreshToken) {
                    callback(e, accessToken, refreshToken);
                }
            );
        }

1. **getTokenFromRefreshToken** 関数を次のコードに置き換えます。これにより、更新トークンを使用してアクセス トークンを取得します。

        function getTokenFromRefreshToken(refreshToken, callback) {
            var OAuth2 = OAuth.OAuth2;
            var oauth2 = new OAuth2(
                credentials.client_id,
                credentials.client_secret,
                credentials.authority,
                credentials.authorize_endpoint,
                credentials.token_endpoint
            );

            oauth2.getOAuthAccessToken(
                refreshToken,
                {
                    grant_type: 'refresh_token',
                    redirect_uri: credentials.redirect_uri,
                    response_mode: 'form_post',
                    nonce: uuid.v4(),
                    state: 'abcd'
                },
                function (e, accessToken) {
                    callback(e, accessToken);
                }
            );
        }

これで、Microsoft Graph を呼び出すためのコードを追加する準備が整いました。 

## <a name="call-microsoft-graph"></a>Microsoft Graph を呼び出す
アプリは Microsoft Graph を呼び出してユーザー情報を取得し、ユーザーの代わりにメールを送信します。これらの呼び出しは UI イベントに応答して index.js コントローラーから始まります。

1. requestUtil.js を開きます。

1. **getUserData** 関数を、以下のコードに置き換えます。これにより GET 要求を構成して */me* エンドポイントに送信し、応答を処理します。

        function getUserData(accessToken, callback) {
            var options = {
                host: 'graph.microsoft.com',
                path: '/v1.0/me',
                method: 'GET',
                headers: {
                    'Content-Type': 'application/json',
                    Accept: 'application/json',
                    Authorization: 'Bearer ' + accessToken
                }
            };

            https.get(options, function (response) {
                var body = '';
                response.on('data', function (d) {
                    body += d;
                });
                response.on('end', function () {
                    var error;
                    if (response.statusCode === 200) {
                        callback(null, JSON.parse(body));
                    } else {
                        error = new Error();
                        error.code = response.statusCode;
                        error.message = response.statusMessage;
                        // The error body sometimes includes an empty space
                        // before the first character, remove it or it causes an error.
                        body = body.trim();
                        error.innerError = JSON.parse(body).error;
                        callback(error, null);
                    }
                });
            }).on('error', function (e) {
                callback(e, null);
            });
        }

1. **postSendMail** 関数を次のコードに置き換えます。これにより POST 要求を構成して */me/sendMail* エンドポイントに送信し、応答を処理します。

        function postSendMail(accessToken, mailBody, callback) {
            var outHeaders = {
                'Content-Type': 'application/json',
                Authorization: 'Bearer ' + accessToken,
                'Content-Length': mailBody.length
            };
            var options = {
                host: 'graph.microsoft.com',
                path: '/v1.0/me/sendMail',
                method: 'POST',
                headers: outHeaders
            };

            // Set up the request
            var post = https.request(options, function (response) {
                var body = '';
                response.on('data', function (d) {
                    body += d;
                });
                response.on('end', function () {
                    var error;
                    if (response.statusCode === 202) {
                        callback(null);
                    } else {
                        error = new Error();
                        error.code = response.statusCode;
                        error.message = response.statusMessage;
                        // The error body sometimes includes an empty space
                        // before the first character, remove it or it causes an error.
                        body = body.trim();
                        error.innerError = JSON.parse(body).error;
                        // Note: If you receive a 500 - Internal Server Error
                        // while using a Microsoft account (outlook.com, hotmail.com or live.com),
                        // it's possible that your account has not been migrated to support this flow.
                        // Check the inner error object for code 'ErrorInternalServerTransientError'.
                        // You can try using a newly created Microsoft account or contact support.
                        callback(error);
                    }
                });
            });
            
            // write the outbound data to it
            post.write(mailBody);
            // we're done!
            post.end();

            post.on('error', function (e) {
                callback(e);
            });
        }
  
1. emailer.js を開きます。

1. **wrapEmail** 関数を次のコードに置き換えます。これにより、送信するメール メッセージを表すペイロードを構築します。

        function wrapEmail(content, recipient) {
            var emailAsPayload = {
                Message: {
                    Subject: 'Welcome to Office 365 development with Node.js and the Office 365 Connect sample',
                    Body: {
                        ContentType: 'HTML',
                        Content: content
                    },
                    ToRecipients: [
                        {
                            EmailAddress: {
                                Address: recipient
                            }
                        }
                    ]
                },
                SaveToSentItems: true
            };
            return emailAsPayload;
        }

## <a name="run-the-app"></a>アプリの実行

1. コマンド プロンプトで、スターター プロジェクトのルート ディレクトリから次のコマンドを実行します。


        npm start

1. ブラウザーで *http://localhost:3000* にナビゲートし、**[Office 365 に接続する]** ボタンを選びます。

1. サインインして要求されたアクセス許可を付与します。 

1. 必要に応じて、受信者のメール アドレスを編集してから、**[メールの送信]** ボタンを選びます。メールが送信されると、ボタンの下に成功メッセージが表示されます。 

## <a name="next-steps"></a>次の手順
- [Graph エクスプローラー](https://graph.microsoft.io/graph-explorer)を使用して REST API を試してみます。
- GitHub で他の [Node.js サンプル](https://github.com/search?utf8=%E2%9C%93&q=node+sample+user%3Amicrosoftgraph&type=Repositories&ref=searchresults)を探します。


## <a name="see-also"></a>関連項目
- [Azure AD v2.0 のプロトコル](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
- [Azure AD v2.0 のトークン](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)