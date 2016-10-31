# <a name="get-started-with-microsoft-graph-and-rest"></a>Microsoft Graph および REST 入門

この記事では、Azure AD v2.0 エンドポイントからアクセス トークンを取得し、REST 呼び出しを使用して Microsoft Graph を呼び出すために必要なタスクについて説明します。電子メール メッセージの認証と取得のためにアプリが使用する要求と応答を、順を追って説明します。

最初に、Azure Active Directory (Azure AD) にアプリを登録する必要があります。 

## <a name="register-the-application"></a>アプリケーションの登録

現在、Azure AD にアプリを登録するには 2 つのオプションがあります。

  - アプリを登録して、個人用 (Microsoft) ID と職場および学校 (Azure AD) アカウントの両方で機能する Azure AD v2.0 エンドポイントを使用する。
  - アプリを登録して、職場および学校アカウントのみをサポートする Azure AD エンドポイントを使用する。

この記事では、v2.0 での登録を前提にしていますので、[アプリケーション登録ポータル](https://apps.dev.microsoft.com/)でアプリを登録します。「[Microsoft Graph アプリケーションを Azure AD v2.0 エンドポイントに登録する](../authorization/auth_register_app_v2.md)」の手順に従って、アプリを登録します。Azure AD エンドポイントの使用に関する詳細は、「[Azure AD を使用して認証する](../authorization/app_authorization.md)」を参照してください。

> v2.0 エンドポイントを使用する場合、いくつかの制限が適用されます。制限が適切であるかどうかを判断するには、「[v2.0 エンドポイントを使用する必要がありますか?](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/)」を参照してください。

## <a name="authenticate-the-user-and-get-an-access-token"></a>ユーザーの認証とアクセス トークンの取得

この記事で説明されているアプリは、Azure AD v2.0 エンドポイントからアクセス トークンを取得するために標準の [OAuth 2.0 プロトコル](http://tools.ietf.org/html/rfc6749)に従って[認証コードの付与フロー](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols-oauth-code/)を実装します。Azure AD v2.0 エンドポイントでサポートされているフローの完全なガイドは、「[v2.0 エンドポイントの種類](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-flows/)」を参照してください。

認証コードの付与フローを使用して、最初に認証コードを取得してから、コードをアクセス トークンと交換します。

### <a name="getting-an-authorization-code"></a>認証コードの取得

承認コードの付与フローでは、まず認証コードを取得します。ユーザーがサインインし、アプリが要求するアクセス許可に同意すると、コードが認証サーバーによってアプリに返されます。

認証コードを要求するには、Azure AD v2.0 エンドポイントに GET 要求を送信します。この URL は、ユーザーがサインインし、同意できるようにブラウザーで開く必要があります。

#### <a name="construct-the-request"></a>要求の構築

1- ベース URL から起動します。

```
https://login.microsoftonline.com/<tenant>/oauth2/v2.0/authorize
```

パス内の *tenant* セグメントは、アプリケーションにサインインできるユーザーを制御します。使用できる値は、*common*、*organizations*、*consumers*、および tenant の各識別子です。詳細については、「[プロトコル エンドポイント](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/#endpoints)」を参照してください。

2- ベース URL にクエリ パラメーターを追加します。これらは、アプリ、要求されたアクセス許可、および他の認証要求情報を識別するために使用されます。次の表に、いくつかの一般的なパラメーターを示します。

| パラメーター | 説明 |
|:------|:------|
| client_id | アプリを登録すると生成されるアプリ ID です。これにより、どのアプリがログオンを要求しているかが Azure AD に通知されます。 |
| redirect_uri | ユーザーがアプリに同意した後、Azure によってリダイレクトされる場所です。この値は、アプリを登録したときに使用された**リダイレクト URI** の値に対応している必要があります。 |
| response_type | アプリが想定している応答タイプです。この値は、認証コードの付与フローの `code` です。 |
| スコープ | アプリが要求している [Microsoft Graph のアクセス許可スコープ](../authorization/permission_scopes.md)をスペースで区切った一覧です。[シングル サインオン](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols-oidc/)に対して [OpenId Connect スコープ](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-scopes/#openid-connect-scopes)を指定することもできます。  |
| state | トークン応答でも返され、検証に使用される要求に含まれている値です。 |

たとえば、電子メールへの読み取りアクセスを要求するアプリケーションの要求 URL は次のようになります。

```
GET https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=<app ID>&redirect_uri=http%3A%2F%2Flocalhost/myapp%2F&response_type=code&state=1234&scope=mail.read
```

3- ユーザーをログオン URL にリダイレクトします。ユーザーには、アプリ名が表示されているサインイン画面が表示されます。サインインすると、アプリが必要とするアクセス許可の一覧が表示され、許可するか拒否するかの確認を求められます。同意すると、ブラウザーは認証コードでリダイレクト URL にリダイレクトされ、次の例に示すように、クエリ文字列に認証コードが表示されます。

```
http://localhost/myapp/?code=AwABAAAA...cZZ6IgAA&state=1234
```

次の手順では、返された認証コードをアクセス トークンと交換します。

### <a name="getting-an-access-token"></a>アクセス トークンの取得

アクセス トークンを取得するため、アプリはフォームでエンコードされたパラメーターに以下のパラメーターを付けてトークン要求 URL (例: `https://login.microsoftonline.com/common/oauth2/v2.0/token`) にポストします。

| パラメーター | 説明 |
|:------|:------|
| client_id | アプリを登録すると生成されるアプリ ID です。 |
| client_secret | アプリを登録するときに生成されるアプリ シークレットです。 |
| code | 前の手順で取得した認証コードです。 |
| redirect_uri | この値は、認証コード要求で使用した値と同じである必要があります。 |
| grant_type | アプリが使用している付与の種類です。この値は、認証コードの付与フローの `code` です。 |
| スコープ | アプリが要求している [Microsoft Graph のアクセス許可スコープ](../authorization/permission_scopes.md)をスペースで区切った一覧です。 |

前の手順のコードを使用する、アプリケーションの要求 URL は次のようになります。

```
POST https://login.microsoftonline.com/common/oauth2/v2.0/token
Content-Type: application/x-www-form-urlencoded

{
  grant_type=authorization_code
  &code=AwABAAAA...cZZ6IgAA
  &redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
  &resource=https%3A%2F%2Fgraph.microsoft.com%2F
  &scope=mail.read
  &client\_id=<app ID>
  &client\_secret=<app SECRET>
}
```

サーバーはアクセス トークンを含む JSON ペイロードで応答します。

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
  "token_type":"Bearer",
  "expires_in":"3600",
  "access_token":"eyJ0eXAi...b66LoPVA",
  "scope":"https://graph.microsoft.com/mail.read",
  ...
}
```

アクセス トークンは JSON ペイロードの `access_token` フィールドにあります。アプリが API に REST 呼び出しを行うときに、アプリはこの値を認証ヘッダーの設定で使用します。

## <a name="call-microsoft-graph"></a>Microsoft Graph を呼び出す

アクセス トークンを取得すると、Microsoft Graph を呼び出す準備ができます。このサンプル アプリはメッセージを取得するため、`https://graph.microsoft.com/v1.0/me/messages` エンドポイントに対する HTTP GET 要求を使用します。

### <a name="refine-the-request"></a>要求を絞り込む

アプリでは、OData クエリ パラメーターを使用して GET 要求の動作を制御できます。アプリがこれらのパラメーターを使用して返される結果の数と、各項目に返されるフィールドの数を制限することをお勧めします。 

サンプル アプリでは、表にメッセージの件名、送信者、メッセージを受信した日時が表示されます。表には最大 25 行が表示され、最新の受信メッセージが上位にくるように並べ替えられます。アプリでは、次のクエリ パラメーターを使用してこれらの結果を取得します。

- `$select` - `subject`、`sender`、および `dateTimeReceived` の各フィールドのみを指定します。
- `$top` -最大で 25 項目を指定します。
- `$orderby` - `dateTimeReceived` フィールドに基づいて結果を並べ替えます。

これにより、次の要求が行われます。

```
GET https://graph.microsoft.com/v1.0/me/messages?$select=subject,from,receivedDateTime&$top=25&$orderby=receivedDateTime%20DESC
Accept: application/json
Authorization: Bearer eyJ0eXAi...b66LoPVA
```

これで、Microsoft Graph を呼び出す方法が理解できたため、API リファレンスを使用して、アプリで必要なその他の呼び出しを作成することができます。ただしアプリでは、呼び出しを行うための適切なアクセス許可をアプリ登録で設定する必要がある点に注意してください。

## <a name="next-steps"></a>次の手順
- [Graph エクスプローラー](https://graph.microsoft.io/graph-explorer)を使用してさらに REST API を試してみる。

## <a name="see-also"></a>関連項目
- [Azure AD v2.0 のプロトコル](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
- [Azure AD v2.0 のトークン](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)
