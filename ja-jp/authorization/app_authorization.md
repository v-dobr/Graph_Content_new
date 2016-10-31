
# <a name="microsoft-graph-app-authentication-using-azure-ad"></a>Azure AD による Microsoft Graph アプリの認証

この記事では、Microsoft Graph アプリの認証と承認フローの例を詳しく説明します。この例では Azure Active Directory (Azure AD) および認証プロバイダーを使用し、承認フローとして<a href="https://msdn.microsoft.com/en-us/library/azure/dn645542.aspx" target="_newtab">認証コード付与フロー</a>を使用します。この例では Microsoft Graph アプリで Azure AD を使用してユーザーを認証する方法、アクセス トークンを取得する方法、更新トークンを使用してアクセス トークンを更新する方法について説明します。

コード付与フローについては、認証プロセスは 2 つの基本的な手順に分割できます。

1. 認証コードを要求する
2. 認証コードを使用して、アクセス トークンと更新トークンを要求する 

現在のアクセス トークンの有効期限が切れたときに、更新トークンを使用して新しいアクセス トークンを取得することができます。
 
##<a name="authenticate-a-user-and-get-app-authorized"></a>ユーザーを認証してアプリの承認を得る

アプリの承認を得るには、まずユーザーの認証を得る必要があります。これを行うには、アプリ情報とともにユーザーを Azure Active Directory (Azure AD) 承認エンドポイントにリダイレクトして、職場または学校アカウントにサインインしてもらいます。ユーザーがサインインして、アプリが要求するアクセス許可に同意すると (ユーザーがまだ同意していなかった場合)、アプリは OAuth アクセス トークンを取得するために必要な認証コードを受け取ります。

> 
  **注**:[Azure AD Authentication Library (ADAL)](https://msdn.microsoft.com/en-us/library/azure/jj573266.aspx) でメソッドを呼び出してこれを行うことができます。承認フローの詳細については、「[認証コードの付与フロー](https://msdn.microsoft.com/en-us/library/azure/dn645542.aspx)」を参照してください。

アプリの承認は、次の URL を使用して HTTPS GET 要求を送信することから始まります。
 
```GET https://login.microsoftonline.com/common/oauth2/authorize?response_type=code&redirect_uri=<uri>&client_id=<id>```

**必須のクエリ文字列パラメーター**

| パラメーター名  | 値  | 説明                                                                                            |
|:----------------|:-------|:-------------------------------------------------------------------------------------------------------|
| *client_id*     | string | アプリ用に作成されたクライアント ID。これは、Azure テナントのアプリケーション レジストリに設定されている、アプリの**クライアント ID** 値です。                                                                  |
| *response_type* | string | 要求された応答の種類を指定します。認証コード付与の要求では、値は code でなければなりません。 |
| *redirect_uri*  | string | 認証完了時にブラウザーの送信先となるリダイレクト URL。この値は、アプリの構成済み**返信 URL** 値と一致する必要があります                        |
 


実行中のアプリケーションで実装されているそのような要求の例を次に示します。


```GET https://login.microsoftonline.com/common/oauth2/authorize?response_type=code&redirect_uri=http%3a%2f%2flocalhost:1339/auth/azureoauth/callback&client_id=8b8539cd-7b75-427f-bef1-4a6264fd4940``` 

この要求は `200 OK` 応答を返し、Azure AD アカウントのログイン ページを提示します。 

ユーザーが有効な認証情報でサインインして、アプリに付与されたアクセス許可に同意すると、ログイン ページは `POST` 要求を Azure に送信します。その要求が成功すると、Azure は呼び出しをアプリのリダイレクト URI に転送する `302 Found` メッセージを返し、アプリが必要なアクセス トークンを受信できるようにします。応答の `Location` ヘッダーで指定された転送 URI は、アプリの応答 URL に `code=...` と `session_state=...` の 2 つのクエリ パラメーターを付加したものに相当します。次の例は、このような応答の抜粋を示しています。 

```no-highlight 
HTTP/1.1 302 Found
Cache-Control: no-cache, no-store
Pragma: no-cache
Content-Type: text/html; charset=utf-8
Expires: -1
Location: http://localhost:1339/auth/azureoauth/callback?code=AAABAAAAvPM...&session_state=a9556cd3-cae6-4bc9-bf51-672f7b79b7c6
Server: Microsoft-IIS/8.5
P3P: CP="DSP CUR OTPi IND OTRi ONL FIN"

..... 
```

この例では、アプリの応答 URL は `http://localhost:1339/auth/azureoauth/callback` です。この応答の処理中に、`code` パラメーター値を抽出して、それを使って初期 OAuth 2.0 アクセス権を取得し、トークンを更新する必要があります (次のセクションを参照)。

> 上記の `302 Found` 応答は、`https://login.windows.net/<tenantId>/oauth2/authorize?...` URL に対してログイン処理を開始した場合に取得する `302 Found` 応答とは異なります。後者の場合、`302 Found` 応答は要求を `login.microsoftonline.com` に転送します。
 
<!---<a name="msg_get_app_authenticated"> </a> -->

##<a name="acquire-an-access-token"></a>アクセス トークンを取得する
Microsoft Graph リソースにアクセスするには、アプリはすべての HTTP 要求に OAuth 2.0 の有効なアクセス トークンを含める必要があります。次の POST 要求を使用してアクセス トークンを取得できます。

```no-highlight 
POST https://login.microsoftonline.com/common/oauth2/token HTTP/1.1
content-type : application/x-www-form-urlencoded
content-length : 144
```
 
この要求には、次の形式の URL エンコードされたペイロードが必要です。
 
```no-highlight 
grant_type=authorization_code
&redirect_uri=<uri>
&client_id=<id>
&client_secret=<secret_key>
&code=<code>
&resource=https%3A%2F%2Fgraph.microsoft.com%2F
```

**必須のクエリ文字列パラメーター**

| パラメーター名  | 値  | 説明                                                                                            |
|:----------------|:-------|:-------------------------------------------------------------------------------------------------------|
| *client_id*     | string | アプリ用に作成されたクライアント ID。  |
| *client_secret*  | string | アプリ用に作成されたキー。この値は、Azure 管理ポータルのアプリの構成ページにある **[キー]** セクションの値と同じです|
| *redirect_uri*  | string | 認証完了時にブラウザーの送信先となるリダイレクト URL。  |
| *code*  | string | 認証コード承認要求に対する応答から返される `code` クエリ パラメーターの値。 |
| *resource*   | string | アクセスするリソース。Microsoft Graph API を呼び出すには、このパラメーターの値を "https://graph.microsoft.com/" に設定します。|

次のスニペットは、初期 OAuth 2.0 アクセス トークンを取得するために使用される要求ペイロードの例を示しています。

```no-highlight  
grant_type=authorization_code&
redirect_uri=http%3A%2F%2Flocalhost%3A1339%2Fauth%2Fazureoauth%2Fcallback&
client_id=8b8539cd-7b75-427f-bef1-4a6264fd4940&client_secret=PJW3dznGfyNSm3rM9aHeXWGKsTMepKXT1Eqy45XXdU4%3D&
code=AAABAAAAvPM1KaPlrEqdFSBzjqfTGBLRVQc6BtQmJ_9DQZUg8Tb7TJgTmbTE7AHM93qB5EKc4Bf-bOgzt3mebAywK-09X1uBHwOluuqSWfd9LU2HHgZtxcZFIYI5UL7L1UEvhrJRvX2iHhfz9ZSRMZMVL55n_K79gCOxtSATeCUw52zPk5ZaQ87Y42SCLsRZN4Y_zddhD3mMpkObiHVT8HzfhBUiT0oX0e-Q439vkbZoKiq1HaqMR3IPHiCXDbPPH5u7a4NTe5xAhh-o2MUIe6s4Xqql86sv1-IwyroOJJMueGUarkfbgwqmYL9Tm-jWab8o-BAK_plVsN73GU8cXO8ts30wa2YmNR5ZxSkw8oiB4mSZwGzGQlch55DRnucDs0SZBgj5etGi3SeXv5jhKlDU2S0bAPnGxF3QAH0N_zBpfakETVlcsHKi714u-tn9da6aTPQsE0IYKTAYgxjTMei6zfRFvCZi-tKdFR6X9TvvmF2iPdGQGWKeLw8CMWUzU8VmOhiHc0aBKG6RaXAOTM067J_9WKYPxMopcytD2z8HVkL1QhggAA&
resource=https%3A%2F%2Fgraph.microsoft.com%2F
```

この要求が成功すると、`200 OK` 応答が返されます。次に、例を示します。

```no-highlight  
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Expires: -1
Content-Length: 2978
Access-Control-Allow-Origin: *

{
    "token_type":"Bearer",
    "expires_in":"3599",
    "expires_on":"1426551729",
    "not_before":"1426547829",
    "resource":"https://graph.microsoft.com/",
    "access_token":"eyJ0eXAiOiJKV1QiLCJhb...",
    "refresh_token":"AAABAAAAvPM1KaPlrEqd...",
    "scope": "Calendar.ReadWrite Directory.Read.All Files.ReadWrite Group.ReadWrite.All Mail.ReadWrite Mail.Send User.ReadBasic.All",
    "id_token":"eyJ0eXAiOiJKV1QiLCJhbGci..."
}
```

 
応答本文は、アクセス トークン (`access_token`) を含む JSON 形式の文字列です。Microsoft Graph リソースへのアクセスを求める後続のあらゆる HTTP 要求に、このトークンを含める必要があります。 

`scope` プロパティの値は、アプリの登録時にアプリに付与されるアクセス許可と一致する必要があります。

アクセス トークンは、`3599` プロパティで指定された発行時刻から、指定された時間間隔 (上の例では `expires_in`) の秒数 (または 1 時間) だけ有効になります。結果には、期限切れ間近のまたは期限切れのアクセス トークンの更新に使用する必要のある更新トークン (`refresh_token`) も含まれています。 

プロダクション コードでは、アプリがこれらのトークンの有効期限を監視して、更新トークンが期限切れになる前に期限切れ間近のアクセス トークンを更新する必要があります。 
-->

<!---<a name="msg_renew_access_token using refresh token"> </a> -->

##<a name="renew-expiring-access-token-using-refresh-token"></a>更新トークンを使用して期限切れ間近のアクセス トークンを更新する
期限切れのアクセス トークンを更新するには、(更新トークンの有効期限が切れる前に) 次の例と同様の POST 要求を使用します。

```no-highlight  
POST https://login.microsoftonline.com/common/oauth2/token HTTP/1.1
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 897


grant_type=refresh_token
&redirect_uri=http%3A%2F%2Flocalhost%3A1339%2Fauth%2Fazureoauth%2Fcallback
&client_id=8b8539cd-7b75-427f-bef1-4a6264fd4940
&client_secret=PJW3dznGfyNSm3rM9aHeXWGKsTMepKXT1Eqy45XXdU4%3D
&refresh_token=AAABAAAAvPM1KaPlrEqdFSBzjqfTGM74--...
&resource=https%3A%2F%2Fgraph.microsoft.com%2F
```

**必須のクエリ文字列パラメーター**

| パラメーター名  | 値  | 説明                                                                                                                                         |
|:----------------|:-------|:----------------------------------------------------------------------------------------------------------------------------------------------------|
| *client_id*     | string | アプリケーション用に作成されたクライアント ID。  |
| *redirect_uri*  | string | 認証完了時にブラウザーの送信先となるリダイレクト URL。これは、最初の要求で使用される *redirect_uri* 値と一致する必要があります。 |
| *client_secret* | string | アプリケーション用に作成されたキー値のいずれか。                                                                                                     |
| *refresh_token* | string | 以前に受信した更新トークン。    |
| *resource*      | string | アクセスするリソース。|

この要求は初期トークン取得要求とほぼ同じであることに注意してください。要求ペイロード内に 2 つの違いがあります。具体的には、`grant_type` パラメーターの値が (`refresh_token` ではなく) `code` になっています。
 
正常な応答は、次の出力のような JSON 文字列のペイロードを返します。

```no-highlight 
{
    "token_type":"Bearer",
    "expires_in":"3600",
    "expires_on":"1426561346",
    "not_before":"1426557446",
    "resource":"https://graph.microsoft.com/",
    "access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOi...", 
    "refresh_token":"AAABAAAAvPM1KaPlrEqdFSBzj...",
   "scope":"Graph.Read",
    "pwd_exp":"6553342",
    "pwd_url":"https://portal.microsoftonline.com/ChangePassword.aspx"
}
```
 
`id_token` プロパティがないことを除いて、この応答本文の構文とセマンティクスは初期トークン取得応答と同じです。新しく返された `access_token` と `refresh_token` の値の有効期間は延長されています。アクセス トークンの新しい有効期間は、トークン更新要求が正常に送信された時刻から、`expires_in` の値で指定された秒数が経過するまでです。 
 
更新トークンが期限切れになると、前述の POST 要求を使用して期限切れのアクセス トークンを更新できなくなります。代わりに、アプリ承認および認証プロセスを再起動する必要があります。



