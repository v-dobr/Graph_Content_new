# <a name="get-started-with-microsoft-graph-in-a-python-app"></a>Python アプリで Microsoft Graph を使ってみる 

この記事では、Azure AD からアクセス トークンを取得し、Microsoft Graph を呼び出すために必要なタスクについて説明します。ここでは、[Python 用 Microsoft Graph Connect のサンプル](https://github.com/microsoftgraph/python3-connect-rest-sample)の手順と、Microsoft Graph API を使用するために実装する主要な概念について説明します。直接 REST 呼び出しを使用して Microsoft Graph にアクセスする方法について説明します。

![Office 365 Python 接続サンプルのスクリーンショット](./images/web-screenshot.png)

> **注**: このチュートリアルとこれに基づくサンプルでは、Azure AD エンドポイントを使用します。Azure AD v2.0 エンドポイントを使用する更新バージョンはまもなく利用できるようになります。後日、もう一度ご確認ください。

##  <a name="prerequisites"></a>前提条件

  * ビジネス向けの Office 365 アカウント。Office 365 アプリのビルドを開始するために必要なリソースを含む、[Office 365 Developer サブスクリプション](https://msdn.microsoft.com/en-us/office/office365/howto/setup-development-environment#bk_Office365Account)にサインアップできます。
  * [Python 用 Microsoft Graph Connect のサンプル](https://github.com/microsoftgraph/python3-connect-rest-sample)

## <a name="register-the-application-in-azure-active-directory"></a>Azure Active Directory にアプリケーションを登録する

最初に、アプリケーションを登録し、Microsoft Graph のサービスを使用するアクセス許可を設定する必要があります。これにより、ユーザーが職場または学校アカウントでアプリケーションにサインインできるようになります。

1. [Azure Portal](https://portal.azure.com/) にサインインします。
2. 上部のバーで、お客様のアカウントをクリックして、**ディレクトリ** リストの下の、アプリケーションを登録する Active Directory のテナントを選びます。
3. 左側のナビゲーションにある **[その他のサービス]** をクリックして、**Azure Active Directory** を選びます。
4. **[アプリの登録]** をクリックし、**[追加]** を選びます。
5. アプリケーションのフレンドリ名 (例: MSGraphConnectPython) を入力して、**アプリケーションの種類**として [Web アプリ/API] を選びます。サインオン URL には、http://127.0.0.1:8000/connect/get_token/ を入力します。**[作成]** をクリックして、アプリケーションを作成します。
6. Azure Portal でアプリケーションを選択し、**[設定]** をクリックして、**[プロパティ]** を選びます。
7. アプリケーション ID の値を検索し、クリップボードにコピーします。
8. アプリケーションのアクセス許可を構成します。
9. **[設定]** メニューで、「**必要なアクセス許可**」セクションを選択し、**[追加]**、**[API の選択]** の順にクリックして、**[Microsoft Graph]** を選びます。
10. 次に、[アクセス許可の選択] をクリックし、**[サインインおよびユーザー プロファイルの読み取り]** と **[ユーザーとしてメールを送信]** を選びます。**[選択]**、**[完了]** の順にクリックします。
11. **[設定]** メニューで、**[キー]** セクションを選びます。説明を入力し、キーの期間を選びます。**[保存]** をクリックします。
12. **重要**:キーの値をコピーしてください。このウィンドウを離れると、この値に再度アクセスできなくなります。この値はアプリ シークレットとして使用します。

## <a name="redirect-the-browser-to-the-sign-in-page"></a>ブラウザーをサインイン ページにリダイレクトする

アプリはブラウザーをサインイン ページにリダイレクトして OAuth フローを開始し、認証コードを取得する必要があります。 

接続サンプルの次のコード ([*connect/auth_helper.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/auth_helper.py) を参照) では、アプリユーザーをリダイレクトするため、またリダイレクトに使用するビューにパイプされるためにアプリが必要とする URL を構築します。 

```python
# This function creates the signin URL that the app will
# direct the user to in order to sign in to Office 365 and
# give the app consent.
def get_signin_url(redirect_uri):
  # Build the query parameters for the signin URL.
  params = { 'client_id': client_id,
             'redirect_uri': redirect_uri,
             'response_type': 'code'
           }

  authorize_url = '{0}{1}'.format(authority, '/common/oauth2/authorize?{0}')
  signin_url = authorize_url.format(urlencode(params))
  return signin_url
```

<!--<a name="authCode"></a>-->
## <a name="receive-an-authorization-code-in-your-reply-url-page"></a>応答 URL ページで認証コードを受け取る

ユーザーがサインインした後、ブラウザーは応答 URL にリダイレクトされ、[*connect/views.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/views.py) 内の ```get_token``` 関数と認証コードがクエリ文字列に ```code``` 変数として付加されます。 

接続サンプルは、クエリ文字列からコードを取得し、アクセス トークン用に交換することができます。

```python
auth_code = request.GET['code']
```

<!--<a name="accessToken"></a>-->
## <a name="request-an-access-token-from-the-token-issuing-endpoint"></a>トークンを発行するエンドポイントからアクセス トークンを要求する

認証コードを作成したら、Azure Active Directory から取得したクライアント ID、キー、応答 URL の値に沿ってこれを使用して、アクセス トークンを要求できます。 

> **注**: 要求には、使用する必要のあるリソースも指定してください。Microsoft Graph の場合、リソースの値は `https://graph.microsoft.com` です。

接続サンプルでは、[*connect/auth_helper.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/auth_helper.py) ファイルの ```get_token_from_code``` 関数を要求しています。

```python
# This function passes the authorization code to the token
# issuing endpoint, gets the token, and then returns it.
def get_token_from_code(auth_code, redirect_uri):
  # Build the post form for the token request
  post_data = { 'grant_type': 'authorization_code',
                'code': auth_code,
                'redirect_uri': redirect_uri,
                'client_id': client_id,
                'client_secret': client_secret,
                'resource': 'https://graph.microsoft.com'
              }
              
  r = requests.post(token_url, data = post_data)
  
  try:
    return r.json()
  except:
    return 'Error retrieving token: {0} - {1}'.format(r.status_code, r.text)
```

> **注**: 応答では、アクセス トークン以外の情報も提供されます。たとえば、アプリは更新トークンを取得することにより、ユーザーが再度明示的にサインインしなくても、新しいアクセス トークンを要求できます。

<!--<a name="request"></a>-->
## <a name="use-the-access-token-in-a-request-to-the-microsoft-graph-api"></a>Microsoft Graph API への要求にアクセス トークンを使用する

アクセス トークンを使用することにより、あなたのアプリは Microsoft Graph API へ認証された要求を行うことができます。アプリで各要求の **Authorization** ヘッダーにアクセス トークンを追加する必要があります。

接続サンプルは、Microsoft Graph API で ```me/microsoft.graph.sendMail``` エンドポイントを使用してメールを送信します。コードは [*connect/graph_service.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/graph_service.py) ファイル内の ```call_sendMail_endpoint``` 関数に記述されています。これは、Authorization ヘッダーにアクセス コードを付加する方法を示すコードです。

```python
# Set request headers.
headers = { 
  'User-Agent' : 'python_tutorial/1.0',
  'Authorization' : 'Bearer {0}'.format(access_token),
  'Accept' : 'application/json',
  'Content-Type' : 'application/json'
}
```

> **注**: 要求は **Content-Type** ヘッダーに `application/json` など、Graph API に受け入れられる値も指定して送信します。

Microsoft Graph API は、あらゆる種類の Microsoft データとの対話に使用できる、非常に強力な統合 API です。API リファレンスを参照し、Microsoft Graph で何を行うことができるかを調べてください。