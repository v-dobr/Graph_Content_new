# <a name="get-started-with-microsoft-graph-in-a-xamarin-forms-app"></a>Xamarin Forms アプリで Microsoft Graph を使ってみる

この記事では、[Azure AD v2.0 エンドポイント](https://graph.microsoft.io/en-us/docs/authorization/converged_auth)からアクセス トークンを取得し、Microsoft Graph を呼び出すために必要なタスクについて説明します。ここでは、[Xamarin Forms 用 Microsoft Graph Connect サンプル](https://github.com/microsoftgraph/xamarin-csharp-connect-sample) のサンプル内のコードを説明し、Microsoft Graph を使用するアプリで実装する必要のある主要な概念について説明します。また、この記事では、[Microsoft Graph クライアント ライブラリ](http://www.nuget.org/packages/Microsoft.Graph/)を使用して Microsoft Graph にアクセスする方法についても説明します。

次のアプリを作成します。

| UWP | Android | iOS |
| --- | ------- | ----|
| <img src="images/UWP.png" alt="Connect sample on UWP" width="100%" /> | <img src="images/Droid.png" alt="Connect sample on Android" width="100%" /> | <img src="images/iOS.png" alt="Connect sample on iOS" width="100%" /> |

**アプリを作成してみたくありませんか。**[Microsoft Graph クイック スタート](https://graph.microsoft.io/en-us/getting-started)を使用してすぐに使い始めるか、またはこの記事で取り扱っている [Xamarin Forms 用 Microsoft Graph Connect サンプル](https://github.com/microsoftgraph/xamarin-csharp-connect-sample)をダウンロードしてください。

## <a name="prerequisites"></a>前提条件

開始するには、次のものが必要です。 

- [Microsoft アカウント](https://www.outlook.com/)か[職場または学校アカウント](http://dev.office.com/devprogram)
- Visual Studio 2015 
- [Xamarin for Visual Studio](https://www.xamarin.com/visual-studio)
- Windows 10 ([開発モードが有効](https://msdn.microsoft.com/library/windows/apps/xaml/dn706236.aspx))
- [Xamarin Forms 用の Microsoft Graph Connect スターター プロジェクト](https://github.com/microsoftgraph/xamarin-csharp-connect-sample/tree/master/starter)。このテンプレートには、コード追加先のいくつかのクラスが含まれています。完全なビューおよびリソース文字列も含まれています。このプロジェクトを取得するには、[Xamarin Forms 用 Microsoft Graph Connect のサンプル](https://github.com/microsoftgraph/xamarin-csharp-connect-sample) を複製またはダウンロードして、**スターター** フォルダー内の **XamarinConnect** ソリューションを開きます。 

このサンプルで iOS プロジェクトを実行する場合は、以下のものが必要です:

- 最新の iOS SDK
- 最新バージョンの Xcode
- Mac OS X Yosemite(10.10) 以上 
- [Xamarin.iOS](https://developer.xamarin.com/guides/ios/getting_started/installation/mac/)
- [Visual Studio に接続されている Xamarin Mac エージェント](https://developer.xamarin.com/guides/ios/getting_started/installation/windows/connecting-to-mac/)


## <a name="register-the-app"></a>アプリを登録する
 
1. 個人用アカウントか職場または学校アカウントのいずれかを使用して、[アプリ登録ポータル](https://apps.dev.microsoft.com/)にサインインします。
2. **[アプリの追加]** を選択します。
3. アプリの名前を入力して、**[アプリケーションの作成]** を選択します。
    
    登録ページが表示され、アプリのプロパティが一覧表示されます。
 
4. **[プラットフォーム]** で、**[プラットフォームの追加]** を選びます。
5. **[Mobile プラットフォーム]** を選びます。
6. アプリケーション ID をコピーします。サンプル アプリにこの値を入力する必要があります。

    アプリケーション ID は、アプリの一意識別子です。リダイレクト URI は、各アプリケーションに対して Windows 10 で提供される一意の URI であり、その URI に送信されたメッセージだけがそのアプリケーションに送信されます。 

7. **[保存]** を選びます。

## <a name="configure-the-project"></a>プロジェクトを構成する

1. Visual Studio でスターター プロジェクトのソリューション ファイルを開きます。
2. **XamarinConnect (ポータブル)** プロジェクト内にある **App.cs** ファイルを開き、`ClientId` フィールドを検索します。アプリケーション ID のプレースホルダーを、登録したアプリのアプリケーション ID と置き換えます。

```c#
public static string ClientID = "ENTER_YOUR_CLIENT_ID";
public static string[] Scopes = { "User.Read", "Mail.Send" };
```
`Scopes` 値は、ユーザーが認証を行うときにアプリが要求する必要のある Microsoft Graph のアクセス許可スコープを保存します。`App` クラス コンストラクターが、ClientID 値を使用して MSAL `PublicClientApplication` クラスのインスタンスをインスタンス化する点に注意してください。ユーザーを認証するために後でこのクラスを使用します。

```c#
IdentityClientApp = new PublicClientApplication(ClientID);
```

## <a name="install-the-microsoft-authentication-library-(msal)"></a>Microsoft 認証ライブラリ (MSAL) をインストールします

[Microsoft 認証ライブラリ](https://www.nuget.org/packages/Microsoft.Identity.Client)には、v2.0 認証エンドポイントを使用したユーザーの認証を簡単にするクラスとメソッドが含まれています。

1. ソリューション エクスプローラーで、**XamarinConnect (ポータブル)** プロジェクトを右クリックして、**[NuGet パッケージの管理...]** を選びます。
2. [参照] をクリックし、Microsoft.Identity.Client を検索します。
3. Microsoft 認証ライブラリの最新バージョンを選択して、**[インストール]** をクリックします。

**XamarinConnect.Droid**、**XamarinConnect.iOS**、および **XamarinConnect.UWP** プロジェクトにおいて、これらと同じ手順を実行します。4 つのプロジェクトすべてに MSAL がインストールされていない場合、アプリはビルドされません。

## <a name="install-the-microsoft-graph-client-library"></a>Microsoft Graph クライアント ライブラリをインストールします

1. ソリューション エクスプローラーで、**XamarinConnect (ポータブル)** プロジェクトを右クリックして、**[NuGet パッケージの管理...]** を選びます。
2. [参照] をクリックし、Microsoft.Graph を検索します。
3. Microsoft Graph クライアント ライブラリの最新バージョンを選択して、**[インストール]** をクリックします。

## <a name="create-the-authenticationhelper.cs-class"></a>AuthenticationHelper.cs クラスを作成する

**XamarinConnect (ポータブル)** プロジェクト内にある AuthenticationHelper.cs ファイルを開きます。このファイルには、ユーザー情報を格納し、アプリからユーザーが切断されたときにのみ認証を強制する追加のロジックとともに、すべての認証コードが含まれます。このクラスには、少なくとも次の 3 つのメソッドが含まれています。`GetTokenForUserAsync`、`Signout`、および `GetAuthenticatedClient`。

`GetTokenHelperAsync` メソッドは、ユーザーを認証し、その後、Microsoft Graph を呼び出すたびに実行されます。

**宣言の使用**

ファイルの先頭にこれらの宣言があることを確認します。

```c#
using Microsoft.Graph;
using System;
using System.Diagnostics;
using System.Net.Http.Headers;
using System.Threading.Tasks;
using Microsoft.Identity.Client;
```

**クラス フィールド**

AuthenticationHelper クラス内にこれらのフィールドがあることを確認します。

```c#
public static string TokenForUser = null;
public static DateTimeOffset expiration;
private static GraphServiceClient graphClient = null;
```

サンプルでは、`GraphServicesClient` をフィールドに格納し、一度だけの作成で済むようにします。アクセス トークンの有効期限 `DateTimeOffset` を格納し、既存のトークンが期限切れになる直前まで新しいトークンを取得しないようにします。

**GetTokenForUserAsync**

`GetTokenForUserAsync` メソッドは **App.cs** ファイルでインスタンス化された `PublicClientApplicationClass` を使用して、ユーザーのアクセス トークンを取得します。ユーザーがまだ認証していない場合、認証 UI を起動します。

```c#
        public static async Task<string> GetTokenForUserAsync()
        {
            if (TokenForUser == null || expiration <= DateTimeOffset.UtcNow.AddMinutes(5))
            {
                AuthenticationResult authResult = await App.IdentityClientApp.AcquireTokenAsync(App.Scopes);

                TokenForUser = authResult.Token;
                expiration = authResult.ExpiresOn;
            }

            return TokenForUser;
        }
```

**サインアウト**

`Signout` メソッドは、`PublicClientApplication` 経由でログインしたすべてのユーザーをサインアウトし (この場合は 1 人のユーザーのみ)、`TokenForUser` 値を無効にします。それは、`GraphServicesClient` 値も無効にします。

```c#
        public static void SignOut()
        {
            foreach (var user in App.IdentityClientApp.Users)
            {
                user.SignOut();
            }
            graphClient = null;
            TokenForUser = null;

        }
``` 

**GetAuthenticatedClient**

最後に、`GraphServicesClient`·を作成するメソッドが必要になります。このメソッドは、Microsoft Graph に対するクライアント経由でのすべての呼び出しに対して `GetTokenForUserAsync` を使用するクライアントを作成します。

```c#
        public static GraphServiceClient GetAuthenticatedClient()
        {
            if (graphClient == null)
            {
                // Create Microsoft Graph client.
                try
                {
                    graphClient = new GraphServiceClient(
                        "https://graph.microsoft.com/v1.0",
                        new DelegateAuthenticationProvider(
                            async (requestMessage) =>
                            {
                                var token = await GetTokenForUserAsync();
                                requestMessage.Headers.Authorization = new AuthenticationHeaderValue("bearer", token);
                            }));
                    return graphClient;
                }

                catch (Exception ex)
                {
                    Debug.WriteLine("Could not create a graph client: " + ex.Message);
                }
            }

            return graphClient;
        }
```

## <a name="send-an-email-with-microsoft-graph"></a>Microsoft Graph を使用して電子メールを送信する

スターター プロジェクトの MailHelper.cs ファイルを開きます。このファイルには、電子メールを作成して送信するコードが含まれています。それは、POST 要求を作成して **https://graph.microsoft.com/v1.0/me/microsoft.graph.SendMail** エンドポイントに送信する単一のメソッド (``ComposeAndSendMailAsync``) で構成されています。 

``ComposeAndSendMailAsync`` メソッドは、MainPage.xaml.cs ファイルによって渡される``subject``、``bodyContent``、および``recipients`` の 3 つの文字列の値を取ります。``subject`` および ``bodyContent`` の文字列は、その他のすべての UI 文字列とともに AppResources.resx ファイルに格納されます。``recipients`` の文字列はアプリのインターフェイスにあるアドレス ボックスのものです。 

**宣言の使用**

ファイルの先頭にこれらの宣言を追加します。

```c#
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.Graph;
```

アドレスは複数渡すことができるため、最初に、``recipients`` 文字列を一連の `EmailAddress` オブジェクトに分割します。それは、`Recipients` オブジェクトのリストを作成するために使用され、そのオブジェクトは要求の POST 本文に入れられて渡されます。

```c#
            // Prepare the recipient list
            string[] splitter = { ";" };
            var splitRecipientsString = recipients.Split(splitter, StringSplitOptions.RemoveEmptyEntries);
            List<Recipient> recipientList = new List<Recipient>();

            foreach (string recipient in splitRecipientsString)
            {
                recipientList.Add(new Recipient { EmailAddress = new EmailAddress { Address = recipient.Trim() } });
            }
```

次に、`Message` オブジェクトを作成し、`GraphServiceClient` を使用して **me/microsoft.graph.SendMail** エンドポイントに送信します。``bodyContent`` の文字列は HTML 文書であるため、要求によって HTML に **ContentType** 値が設定されます。

```c#
            try
            {
                var graphClient = AuthenticationHelper.GetAuthenticatedClient();

                var email = new Message
                {
                    Body = new ItemBody
                    {
                        Content = bodyContent,
                        ContentType = BodyType.Html,
                    },
                    Subject = subject,
                    ToRecipients = recipientList,
                };

                try
                {
                    await graphClient.Me.SendMail(email, true).Request().PostAsync();
                }
                catch (ServiceException exception)
                {
                    throw new Exception("We could not send the message: " + exception.Error == null ? "No error message returned." : exception.Error.Message);
                }


            }

            catch (Exception e)
            {
                throw new Exception("We could not send the message: " + e.Message);
            }
```

完全なクラスは、次のようになります。

```c#
    public class MailHelper
    {
        /// <summary>
        /// Compose and send a new email.
        /// </summary>
        /// <param name="subject">The subject line of the email.</param>
        /// <param name="bodyContent">The body of the email.</param>
        /// <param name="recipients">A semicolon-separated list of email addresses.</param>
        /// <returns></returns>
        public async Task ComposeAndSendMailAsync(string subject,
                                                            string bodyContent,
                                                            string recipients)
        {

            // Prepare the recipient list
            string[] splitter = { ";" };
            var splitRecipientsString = recipients.Split(splitter, StringSplitOptions.RemoveEmptyEntries);
            List<Recipient> recipientList = new List<Recipient>();

            foreach (string recipient in splitRecipientsString)
            {
                recipientList.Add(new Recipient { EmailAddress = new EmailAddress { Address = recipient.Trim() } });
            }

            try
            {
                var graphClient = AuthenticationHelper.GetAuthenticatedClient();

                var email = new Message
                {
                    Body = new ItemBody
                    {
                        Content = bodyContent,
                        ContentType = BodyType.Html,
                    },
                    Subject = subject,
                    ToRecipients = recipientList,
                };

                try
                {
                    await graphClient.Me.SendMail(email, true).Request().PostAsync();
                }
                catch (ServiceException exception)
                {
                    throw new Exception("We could not send the message: " + exception.Error == null ? "No error message returned." : exception.Error.Message);
                }


            }

            catch (Exception e)
            {
                throw new Exception("We could not send the message: " + e.Message);
            }
        }
    }
``` 

これで、Microsoft Graph と対話するために必要な、アプリの登録、ユーザーの認証、および要求の作成の 3 つの手順が完了しました。 

## <a name="run-the-app"></a>アプリの実行
1. 実行するプロジェクトを選びます。ユニバーサル Windows プラットフォームのオプションを選択すると、ローカル コンピューターでサンプルを実行できます。iOS プロジェクトを実行する場合は、[Xamarin ツールがインストールされた Mac](https://developer.xamarin.com/guides/ios/getting_started/installation/windows/connecting-to-mac/) に接続する必要があります。(また、このソリューションを Mac 上の Xamarin Studio で開いて、そこからサンプルを直接実行することもできます。)Android プロジェクトを実行する場合は、[Visual Studio Emulator for Android](https://www.visualstudio.com/features/msft-android-emulator-vs.aspx) を使用できます。 

    ![](images/SelectProject.png "Select project in Visual Studio")

2. F5 キーを押して、ビルドとデバッグを実行します。　ソリューションを実行し、個人用アカウント、あるいは職場または学校のアカウントのいずれかでサインインします。
    > **注** ビルド構成マネージャーを開いて、ビルドと展開の手順が UWP プロジェクトに対して選択されていることを確認することが必要な場合があります。 

3. 個人用あるいは職場または学校アカウントでサインインし、要求されたアクセス許可を付与します。

4. **[メールの送信]** ボタンを選びます。メールが送信されると、成功メッセージが表示されます。

## <a name="next-steps"></a>次の手順
- [Graph エクスプローラー](https://graph.microsoft.io/graph-explorer)を使用して REST API を試してみます。
- [Xamarin.Forms 用 Microsoft Graph SDK スニペット ライブラリ](https://github.com/microsoftgraph/xamarin-csharp-snippets-sample) で一般的な操作の例を検索するか、または GitHub 上のその他の [Xamarin サンプル](https://github.com/microsoftgraph?utf8=%E2%9C%93&query=xamarin) を探してください。

## <a name="see-also"></a>関連項目
- [Microsoft Graph .NET クライアント ライブラリ](https://github.com/microsoftgraph/msgraph-sdk-dotnet)
- [Azure AD v2.0 のプロトコル](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
- [Azure AD v2.0 のトークン](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)
