# <a name="call-microsoft-graph-in-a-service-or-daemon-app"></a>サービスまたはデーモン アプリで Microsoft Graph を呼び出す

この記事では、シングル テナントのサービスまたはデーモン アプリを Office 365 に接続し、Microsoft Graph API を呼び出すために最小限必要なタスクについて説明します。

> 注:このトピックでは、アプリの認証プロバイダーとして Azure Active Directory を使用する場合について説明します。Azure AD v2.0 エンドポイントを使用してサービスまたはデーモン アプリを実装する場合は、「<a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols-oauth-client-creds/" target="_newtab">v2.0 プロトコル - OAuth 2.0 クライアント資格情報のフロー</a>」をご覧ください。

## <a name="register-the-application-in-azure-active-directory"></a>Azure Active Directory にアプリケーションを登録する

Office 365 の使用を開始する前に、[Azure ポータル](https://portal.azure.com)でアプリケーションを登録する必要があります。以下の詳細にご注意ください。

- アプリケーションを登録した後、サービスまたはデーモン アプリで必要なアプリケーションのアクセス許可を設定します。
- Azure アプリケーション登録の次の値をメモしてください。サービスまたはデーモンアプリ内に OAuth フローを構成するにはこれらの値が必要です。
    * アプリケーション ID (アプリケーションで一意)
    * アプリ キー、またはシークレット (アプリケーションで一意)
    * アプリの OAuth 2.0 のトークン エンドポイント
      * この値は、Azure 管理ポータルでアプリのページの下部にある *[エンドポイントの表示]* をクリックすると表示されます。エンドポイントは `https://login.microsoftonline.com/<tenantId>/oauth2/token` のようになります。

## <a name="request-an-access-token-from-the-token-issuing-endpoint"></a>トークンを発行するエンドポイントからアクセス トークンを要求する

クライアント アプリとは異なり、サービスやデーモン アプリでは、ユーザーにサインインしてあなたのアプリケーションを承認してもらうことはできません。代わりに、アプリケーションに OAuth 2.0 クライアント資格情報付与フローが実装されている必要があります。これにより、Microsoft Graph を呼び出すときに、ユーザーを偽装するのではなく、独自の資格情報、そのクライアント ID、アプリケーション キーを使用して認証を行えるようになります。認証フローの詳細については、「[クライアント資格情報を使用したサービス間の呼び出し](https://msdn.microsoft.com/en-us/library/azure/dn645543.aspx)」をご覧ください。

以下のパラメーターを使用してエンドポイントを発行するトークンに HTTP POST 要求を行います。その際、`<clientId>` と `<clientSecret>` をそれぞれ、自分のアプリのクライアント ID とアプリケーション キーに置き換えます。

```http
POST https://login.microsoftonline.com/<tenantId>/oauth2/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials
&client_id=<clientId>
&client_secret=<clientSecret>
&resource=https://graph.microsoft.com
```

応答には、アクセス トークンと有効期限の情報が含まれます。

```json
{ 
  "token_type": "Bearer",
  "expires_in": "3599",
  "scope": "User.Read",
  "expires_on": "1449685363",
  "not_before": "1449681463",
  "resource": "https://graph.microsoft.com",
  "access_token": "<token>"
}
```

## <a name="use-the-access-token-in-a-request-to-the-microsoft-graph-api"></a>Microsoft Graph API への要求にアクセス トークンを使用する

アクセス トークンを使用することにより、あなたのアプリは Microsoft Graph API へ認証された要求を行うことができます。アプリで各要求の **Authorization** ヘッダーにアクセス トークンを追加する必要があります。

たとえば、Azure 管理ポータルで*すべてのユーザーの完全なプロファイルの読み取り*アクセス許可が選択されている場合、サービスまたはデーモン アプリはテナント内のすべてのユーザーを取得することができます。 

```http
GET https://graph.microsoft.com/v1.0/users
Authorization: Bearer <token>
```
