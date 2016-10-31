# <a name="calling-the-microsoft-graph-api"></a>Microsoft Graph API を呼び出す

Microsoft Graph リソースにアクセスして操作するには、次の操作のいずれを使用して、リソース URL を呼び出し指定します。   

- GET
- POST
- PATCH
- PUT
- DELETE 

Microsoft Graph API のすべての要求は、次の基本的な URL パターンを使用します。

```
    https://graph.microsoft.com/{version}/{resource}?[query_parameters]
```

この URL で、

- `https://graph.microsoft.com` は、Microsoft Graph API エンドポイントです。
- `{version}` は、ターゲットのサービス バージョンです。たとえば、`v1.0` または `beta` です。
- `{resource}` は、以下のようなリソースのセグメントまたはパスです。
  - `users`, `groups`, `devices`, `organization`
  - 別名 `me`、サインインしているユーザーに解決されます
   - ユーザーに属するリソース (`me/events`、`me/drive`、`me/messages` など)
  - 別名 `myOrganization`、サインインしているユーザーの組織のテナントに解決されます
- `[query_parameters]` は、追加のクエリ パラメーター (`$filter` や `$select` など) を表します。

必要に応じて、テナントを要求の一部としても指定できます。`me` を使用する場合は、テナントを指定しません。一般的な要求の一覧については、「[Microsoft Graph の概要](overview.md)」を参照してください。

## <a name="microsoft-graph-api-metadata"></a>Microsoft Graph API メタデータ
メタデータ ドキュメント ($metadata) が、サービス ルートに公開されます。たとえば、v1.0 およびベータ版のサービス ドキュメントは、次の URL で表示できます。

Microsoft Graph API `v1.0` メタデータ。
```
    https://graph.microsoft.com/v1.0/$metadata
```
Microsoft Graph API `beta` メタデータ。
```
    https://graph.microsoft.com/beta/$metadata
```

メタデータにより、エンティティの種類およびセット、複合型、ならびに Microsoft Graph が送受信する要求および応答パケットを構成する列挙型を含む、Microsoft Graph のデータ モデルを参照し、理解することができます。メタデータを使用して、Microsoft Graph のエンティティ間のリレーションシップを理解することができ、また、エンティティ間を移動する URL を確立することができます。このナビゲーション ベースの相互関連性は、Microsoft Graph 固有の特徴となっています。

パス URL リソース名、クエリ パラメーター、アクション パラメーターと値は、大文字と小文字が区別されません。ただし、割り当てた値、エンティティ ID、その他の base64 でエンコードされた値では大文字と小文字を区別します。

次のセクションでは、Microsoft Graph API を呼び出す基本的なプログラミング パターンをいくつか示します。

## <a name="navigate-from-a-set-to-a-member"></a>セットからメンバーへの移動

ユーザーに関する情報を表示するには、HTTPS GET 要求を使用して、`users` コレクションから識別子によって識別される特定ユーザーの `User` エンティティを取得します。`User` エンティティでは、`id` プロパティと `userPrincipalName` プロパティのいずれかを識別子として使用できます。次の要求の例では、ユーザーの ID として `userPrincipalName` の値が使用されています。 

```no-highlight 
GET https://graph.microsoft.com/v1.0/users/john.doe@contoso.onmicrosoft.com HTTP/1.1
Authorization : Bearer <access_token>
```

成功した場合は、次のように、ペイロードにユーザー リソース表現を含む 200 OK 応答が返されるはずです。

```no-highlight 
HTTP/1.1 200 OK
content-type: application/json;odata.metadata=minimal
content-length: 982

{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users/$entity",
    "id": "c95e3b3a-c33b-48da-a6e9-eb101e8a4205",
    "city": "Redmond",
    "country": "USA",
    "department": "Help Center",
    "displayName": "John Doe",
    "givenName": "John",
    "userPrincipalName": "john.doe@contoso.onmicrosoft.com",

    ... 
}
```


## <a name="project-from-an-entity-to-properties"></a>1 つのエンティティから複数のプロパティへのプロジェクション
ユーザーから提供された_自己紹介_の記述やスキル セットなどのユーザーの個人データのみを取得するには、以前の要求に _select_ クエリ パラメーターを追加することができます。たとえば、

```no-highlight 
GET https://graph.microsoft.com/v1.0/users/john.doe@contoso.onmicrosoft.com?$select=displayName,aboutMe,skills HTTP/1.1
Authorization : Bearer <access_token>
```

成功した場合の応答は、200 OK 状態と次の形式のペイロードを返します。

```no-highlight 
HTTP/1.1 200 OK
content-type: application/json;odata.metadata=minimal
content-length: 169

{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users/$entity",
    "aboutMe": "A cool and nice guy.",
    "displayName": "John Doe",
    "skills": [
        "n-Lingual",
        "public speaking",
        "doodling"
    ]
}
```

ここでは、`user` エンティティ上のプロパティ セット全体ではなく、`aboutMe` プロパティと `displayName` プロパティと `skills` プロパティのみが返されます。

## <a name="traverse-to-another-resource-via-relationship"></a>リレーションシップ経由の別のリソースへの走査
上司は、直属の部下である他のユーザーとの `directReports` リレーションシップを保持します。ユーザーの直属の部下の一覧を問い合わせるために、次の HTTPS GET 要求を使用して、リレーションシップ走査経由で指定されたターゲットに移動できます。 

```no-highlight 
GET https://graph.microsoft.com/v1.0/users/john.doe@contoso.onmicrosoft.com/directReports HTTP/1.1
Authorization : Bearer <access_token>
```

成功した場合の応答は、200 OK 状態と次の形式のペイロードを返します。

```no-highlight 
HTTP/1.1 200 OK
content-type: application/json;odata.metadata=minimal
content-length: 152
    
{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#directoryObjects/$entity",
    "@odata.type": "#microsoft.graph.user",
    "id": "c37b074d-fe9d-4e68-83ad-b4401d3be174",
    "department": "Sales & Marketing",
    "displayName": "Bonnie Kearney",

    ...
}
```

同様に、リレーションシップをフォローすると関連リソースに移動できます。たとえば、`user => messages` リレーションシップは、Azure AD ユーザーから Outlook メール メッセージのセットへのグラフ走査を可能にします。次の例は、REST API 呼び出しでこれを行う方法を示しています。


```no-highlight 
GET https://graph.microsoft.com/v1.0/me/messages HTTP/1.1
Authorization : Bearer <access_token>
```

    
成功した場合の応答は、200 OK 状態と次の形式のペイロードを返します。


```no-highlight 
HTTP/1.1 200 OK
content-type: application/json;odata.metadata=minimal
odata-version: 4.0
content-length: 147
    
{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users('john.doe%40contoso.onmicrosoft.com')/Messages",
  "@odata.nextLink": "https://graph.microsoft.com/v1.0/me/messages?$top=1&$skip=1",
  "value": [
    {
      "@odata.etag": "W/\"FwAAABYAAABMR67yw0CmT4x0kVgQUH/3AAJL+Kej\"",
      "id": "<id-value>",
      "createdDateTime": "2015-11-14T00:24:42Z",
      "lastModifiedDateTime": "2015-11-14T00:24:42Z",
      "changeKey": "FwAAABYAAABMR67yw0CmT4x0kVgQUH/3AAJL+Kej",
      "categories": [],
      "receivedDateTime": "2015-11-14T00:24:42Z",
      "sentDateTime": "2015-11-14T00:24:28Z",
      "hasAttachments": false,
      "subject": "Did you see last night's game?",
      "body": {
        "ContentType": "HTML",
        "Content": "<content>"
      },
      "BodyPreview": "it was great!",
      "Importance": "Normal",
            
       ...
    }
  ]
}
```

## <a name="project-from-entities-to-properties"></a>複数のエンティティから複数のプロパティへのプロジェクション
単一のエンティティからそのプロパティへのプロジェクションに加えて、同様の `select` クエリ オプションをエンティティ コレクションに適用して、それらのプロパティのいずれかのコレクションにプロジェクションさせることもできます。たとえば、サインインしているユーザーのドライブにある項目を問い合わせるには、次の HTTPS GET 要求を送信します。

```no-highlight 
GET https://graph.microsoft.com/v1.0/me/drive/root/children?$select=name HTTP/1.1
Authorization : Bearer <access_token>
```

成功した場合の応答は、次の例に示すように、200 OK 状態コードと、共有ファイルの名前と種類を含むペイロードを返します。

```no-highlight 
{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users('john.doe%40contoso.onmicrosoft.com')/drive/root/children(name,type)",
  "value": [
    {
      "@odata.etag": "\"{896A8E4D-27BF-424B-A0DA-F073AE6570E2},2\"",
      "name": "Shared with Everyone"
    },
    {
      "@odata.etag": "\"{B39D5D2E-E968-491A-B0EB-D5D0431CB423},1\"",
      "name": "Documents"
    },
    {
      "@odata.etag": "\"{9B51EA38-3EE6-4DC1-96A6-230E325EF054},2\"",
      "name": "newFile.txt"
    }
  ]
}
```

## <a name="query-a-subset-of-users-with-the-filtering-query-option"></a>フィルター処理クエリ オプションを使用してユーザーのサブセットを問い合わせる
組織内の特定の役職の従業員を検索するには、ユーザー コレクションから移動してから、_filter_ クエリ オプションを指定することができます。次に、例を示します。

    
```no-highlight 
GET https://graph.microsoft.com/v1.0/users/?$filter=jobTitle+eq+%27Helper%27 HTTP/1.1
Authorization : Bearer <access_token>
```

成功した場合の応答は、次の例に示すように、200 OK 状態コードと指定された役職 (`'Helper'`) を持つユーザーの一覧を返します。

```no-highlight 
HTTP/1.1 200 OK
content-type: application/json;odata.metadata=minimal;odata.streaming=true;IEEE754Compatible=false;charset=utf-8
odata-version: 4.0
content-length: 986

{
    "@odata.context": "https://graph.microsoft.com/v1.0/contoso.onmicrosoft.com/$metadata#users",
    "value": [
        {
            "id": "c95e3b3a-c33b-48da-a6e9-eb101e8a4205",
            "city": "Redmond",
            "country": "USA",
            "department": "Help Center",
            "displayName": "Jane Doe",
            "givenName": "Jane",
            "jobTitle": "Helper",
            ......
            "mailNickname": "Jane",
            "mobile": null,
            "otherMails": [
                "jane.doe@contoso.onmicrosoft.com"
            ],
            ......
            "surname": "Doe",
            "usageLocation": "US",
            "userPrincipalName": "help@contoso.onmicrosoft.com",
            "userType": "Member"
        },
        
        ...
    ]
}
```

## <a name="call-actions-or-functions"></a>アクションまたは関数を呼び出す
Microsoft Graph は、_アクション_および_関数_もサポートし、標準の HTTP メソッドとは単純に一致しない方法でリソースを操作します。たとえば、次の HTTPS POST 要求は、サインインしているユーザー (`me`) に電子メール メッセージを送信させます。
```no-highlight 
POST https://graph.microsoft.com/v1.0/me/sendMail HTTP/1.1
authorization: bearer <access_token>
content-type: application/json
content-length: 96

{
  "message": {
    "subject": "Meet for lunch?",
    "body": {
      "contentType": "Text",
      "content": "The new cafeteria is open."
    },
    "toRecipients": [
      {
        "emailAddress": {
          "address": "garthf@a830edad9050849NDA1.onmicrosoft.com"
        }
      }
    ],
    "attachments": [
      {
        "@odata.type": "#Microsoft.OutlookServices.FileAttachment",
        "name": "menu.txt",
        "contentBytes": "bWFjIGFuZCBjaGVlc2UgdG9kYXk="
      }
    ]
  },
  "saveToSentItems": "false"
}
```

要求ペイロードには、`sendMail` アクションへの入力が含まれています。これは、$metadata でも定義されています。
