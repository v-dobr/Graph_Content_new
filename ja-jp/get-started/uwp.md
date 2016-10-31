# <a name="get-started-with-microsoft-graph-in-a-universal-windows-10-app"></a>ユニバーサル Windows 10 アプリで Microsoft Graph を使ってみる

> **エンタープライズのお客様向けにアプリを作成していますか?**エンタープライズのお客様が、<a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-conditional-access-device-policies/" target="_newtab">条件付きのデバイスへのアクセス</a>のようなエンタープライズ モビリティ セキュリティの機能をオンにしている場合、アプリが動作しない可能性があります。その場合、気がつかないまま、お客様の側でエラーが発生してしまう可能性があります。 

> **すべてのエンタープライズのお客様**の**すべてのエンタープライズ シナリオ**をサポートするには、Azure AD エンドポイントを使用し、[Azure 管理ポータル](https://aka.ms/aadapplist)でアプリを管理する必要があります。詳細については、「[Azure AD か Azure AD v2.0 エンドポイントかを決定する](../authorization/auth_overview.md#deciding-between-azure-ad-and-the-v2-authentication-endpoint)」を参照してください。

この記事では、[Azure AD v2.0 エンドポイント](https://graph.microsoft.io/en-us/docs/authorization/converged_auth)からアクセス トークンを取得し、Microsoft Graph を呼び出すために必要なタスクについて説明します。ここでは、[UWP (REST) 用 Microsoft Graph Connect サンプル](https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample)と [UWP (ライブラリ) 用 Microsoft Graph Connect サンプル](https://github.com/microsoftgraph/uwp-csharp-connect-sample)の各サンプル内のコードを説明し、Microsoft Graph を使用するアプリで実装する必要のある主要な概念について説明します。また、この記事では、未加工の REST 呼び出しと [Microsoft Graph クライアント ライブラリ](http://www.nuget.org/packages/Microsoft.Graph/)を使用して Microsoft Graph にアクセスする方法についても説明します。

これらの GitHub リポジトリのこのチュートリアルで作成するアプリの両方のバージョンをダウンロードすることができます。

* [UWP (REST) 用 Microsoft Graph Connect のサンプル](https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample)
* [UWP (ライブラリ) 用 Microsoft Graph Connect のサンプル](https://github.com/microsoftgraph/uwp-csharp-connect-sample)

**アプリを作成してみたくありませんか。**「[Microsoft Graph クイック スタート](https://graph.microsoft.io/en-us/getting-started)」を使用すれば、すばやく稼働させることができます。

## <a name="sample-user-interface"></a>サンプル ユーザー インターフェイス

サンプルには、上部のコマンド バー、**[接続]** ボタン､**[メールの送信]** ボタン、およびサインイン ユーザーの電子メールアドレスが自動入力された編集可能なテキスト ボックスで構成される非常にシンプルなユーザー インターフェイスが含まれています。

**[メールの送信]** ボタンは、ユーザーが接続されていないときは無効になります。

![[接続] ボタンが有効になっていて、[メールの送信] ボタンが無効になっていることを示す画面](images/SignedOut.png)

ユーザーが接続されている場合、上部のコマンド バーに [切断] ボタンが表示されます。

![接続されたユーザーの電子メール アドレスと [メールの送信] ボタンが有効になっていることを示す画面](images/SignedIn.png)

すべてのサンプルの UI 文字列は、Assets フォルダー内の Resources.resw ファイルに格納されています。

## <a name="prerequisites"></a>前提条件

開始するには、次のものが必要です。 

- [Microsoft アカウント](https://www.outlook.com/)か[職場または学校アカウント](http://dev.office.com/devprogram)
- Visual Studio 2015 
- [UWP (ライブラリ) 用 Microsoft Graph スターター プロジェクト](https://github.com/microsoftgraph/uwp-csharp-connect-sample/tree/master/starter)または [UWP (REST) 用 Microsoft Graph スターター プロジェクト](https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample/tree/master/starter)のいずれか。両方のテンプレートには、コードを追加する空のクラスが含まれています。また、リソース文字列も含まれています。これらのいずれかまたは両方のプロジェクトを取得するには、[UWP (ライブラリ) 用 Microsoft Graph Connect サンプル](https://github.com/microsoftgraph/uwp-csharp-connect-sample)と [UWP (REST) 用 Microsoft Graph Connect サンプル](https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample)の両方またはいずれかを複製するかダウンロードして、**スターター** フォルダー内でソリューションを開きます。


## <a name="register-the-app"></a>アプリを登録する
 
1. 個人用アカウントか職場または学校アカウントのいずれかを使用して、[アプリ登録ポータル](https://apps.dev.microsoft.com/)にサインインします。
2. **[アプリの追加]** を選択します。
3. アプリの名前を入力して、**[アプリケーションの作成]** を選択します。
    
    登録ページが表示され、アプリのプロパティが一覧表示されます。
 
4. **[プラットフォーム]** で、**[プラットフォームの追加]** を選びます。
5. **[Mobile プラットフォーム]** を選びます。
6. クライアント ID (アプリ ID) とリダイレクト URI の値の両方をクリップボードにコピーします。サンプル アプリにこれらの値を入力する必要があります。

    アプリ ID は、アプリの一意識別子です。リダイレクト URI は、各アプリケーションに対して Windows 10 で提供される一意の URI であり、その URI に送信されたメッセージだけがそのアプリケーションに送信されます。 

7. **[保存]** を選びます。

## <a name="configure-the-project"></a>プロジェクトを構成する

1. Visual Studio でスターター プロジェクトのソリューション ファイルを開きます。
2. プロジェクトの **App.xaml** ファイルを開き、`Application.Resources` ノードを見つけます。アプリケーション ID とリダイレクト URI のプレースホルダーを、登録したアプリの対応する値に置き換えます。


```xml
    <Application.Resources>
        <!-- Add your Client Id here. -->
        <x:String x:Key="ida:ClientID">ENTER_YOUR_CLIENT_ID</x:String>
        <!-- Add your Redirect URI here. -->
        <x:String x:Key="ida:ReturnUrl">ENTER_YOUR_REDIRECT_URI</x:String>
    </Application.Resources>
```

## <a name="install-the-microsoft-authentication-library-(msal)"></a>Microsoft 認証ライブラリ (MSAL) をインストールします

[Microsoft 認証ライブラリ](https://www.nuget.org/packages/Microsoft.Identity.Client)には、Azure AD v2.0 エンドポイントを使用したユーザーの認証を簡単にするクラスとメソッドが含まれています。

1. ソリューション エクスプローラーで、プロジェクト名を右クリックして、**[NuGet パッケージの管理...]** を選びます。
2. [参照] をクリックし、Microsoft.Identity.Client を検索します。
3. Microsoft 認証ライブラリの最新バージョンを選択して、**[インストール]** をクリックします。

## <a name="install-the-microsoft-graph-client-library"></a>Microsoft Graph クライアント ライブラリをインストールします

> **注**: 未加工の REST 呼び出しを使用して Microsoft Graph にアクセスする場合は、このセクションをスキップできます。

1. ソリューション エクスプローラーで、プロジェクト名を右クリックして、**[NuGet パッケージの管理...]** を選びます。
2. [参照] をクリックし、Microsoft.Graph を検索します。
3. Microsoft Graph クライアント ライブラリの最新バージョンを選択して、**[インストール]** をクリックします。

## <a name="create-the-authenticationhelper.cs-class"></a>AuthenticationHelper.cs クラスを作成する

スターター プロジェクトの AuthenticationHelper.cs ファイルを開きます。このファイルには、ユーザー情報を格納し、アプリからユーザーが切断されたときにのみ認証を強制する追加のロジックとともに、すべての認証コードが含まれます。このクラスには、少なくとも次の 2 つのメソッドが含まれています。`GetTokenForUserAsync` および `Signout`。Microsoft Graph クライアント ライブラリを使用している場合は 3 番目のメソッド `GetAuthenticatedClient` を追加する必要があります。

``GetTokenHelperAsync`` メソッドは、ユーザーを認証し、その後、Microsoft Graph を呼び出すたびに実行されます。

**宣言の使用**

***クライアント ライブラリ バージョン***

Microsoft Graph クライアント ライブラリを使用している場合は、これらの宣言が必要です。

```c#
using System;
using System.Diagnostics;
using System.Net.Http.Headers;
using System.Threading.Tasks;
using Microsoft.Graph;
using Microsoft.Identity.Client;
```

***REST バージョン***

未加工の REST 呼び出しを使用して Microsoft Graph にアクセスしている場合は、AuthenticationHelper クラスのこれらの `using` 宣言が必要です。

```c#
using System;
using System.Threading.Tasks;
using Windows.Storage;
using Microsoft.Identity.Client;
```

**クラス フィールド**

AuthenticationHelper クラスの両方のバージョンには、これらのフィールドが必要です。

```c#
// The Client ID is used by the application to uniquely identify itself to the Azure AD v2.0 endpoint.
static string clientId = App.Current.Resources["ida:ClientID"].ToString();
public static string[] Scopes = { "User.Read", "Mail.Send" };
public static PublicClientApplication IdentityClientApp = new PublicClientApplication(clientId);
public static string TokenForUser = null;
public static DateTimeOffset Expiration;
```

両方のバージョンで、ユーザーの認証に MSAL `PublicClientApplication` クラスが使用されることに注意してください。`Scopes` フィールドは、ユーザーが認証を行うときにアプリが要求する必要のある Microsoft Graph のアクセス許可スコープを保存します。 

***クライアント ライブラリ バージョン***

クライアント ライブラリを使用している場合は、1 回作成するだけで済むように `GraphServicesClient` をフィールドとして保存します。

```c#
private static GraphServiceClient graphClient = null;
```

***REST バージョン***

REST 呼び出しを使用している場合は、ユーザーに関する情報を保存する必要があるため、アプリのローミングの設定で複数の値を保存する必要があります。(クライアント ライブラリは、他のバージョンでこの情報を提供します)。

```c#
public static ApplicationDataContainer _settings = ApplicationData.Current.RoamingSettings;
```

**GetTokenForUserAsync**

***クライアント ライブラリ バージョン***

`GetTokenForUserAsync` メソッドは、PublicClientApplicationClass および ClientId 設定を使用して、ユーザーのアクセス トークンを取得します。ユーザーがまだ認証していない場合、認証 UI を起動します。これは、クライアント ライブラリを使用している場合に使用するメソッドのバージョンです。

```c#
        public static async Task<string> GetTokenForUserAsync()
        {
            AuthenticationResult authResult;
            try
            {
                authResult = await IdentityClientApp.AcquireTokenSilentAsync(Scopes);
                TokenForUser = authResult.Token;
            }

            catch (Exception)
            {
                if (TokenForUser == null || Expiration <= DateTimeOffset.UtcNow.AddMinutes(5))
                {
                    authResult = await IdentityClientApp.AcquireTokenAsync(Scopes);

                    TokenForUser = authResult.Token;
                    Expiration = authResult.ExpiresOn;
                }
            }

            return TokenForUser;
        }
```

***REST バージョン***

`GetTokenForUserAsync` メソッドの REST バージョンでは、サインインしているユーザーに関する情報も保存する必要があるため、実行する作業がもう少しあります。

```c#
        public static async Task<string> GetTokenForUserAsync()
        {
            AuthenticationResult authResult;
            try
            {
                authResult = await IdentityClientApp.AcquireTokenSilentAsync(Scopes);
                TokenForUser = authResult.Token;
                // save user ID in local storage
                _settings.Values["userID"] = authResult.User.UniqueId;
                _settings.Values["userEmail"] = authResult.User.DisplayableId;
                _settings.Values["userName"] = authResult.User.Name;
            }

            catch (Exception)
            {
                if (TokenForUser == null || Expiration <= DateTimeOffset.UtcNow.AddMinutes(5))
                {
                    authResult = await IdentityClientApp.AcquireTokenAsync(Scopes);

                    TokenForUser = authResult.Token;
                    Expiration = authResult.ExpiresOn;

                    // save user ID in local storage
                    _settings.Values["userID"] = authResult.User.UniqueId;
                    _settings.Values["userEmail"] = authResult.User.DisplayableId;
                    _settings.Values["userName"] = authResult.User.Name;
                }
            }

            return TokenForUser;
        }
```

**サインアウト**

両方のバージョンでの `Signout` メソッドは、`PublicClientApplication` 経由でログインしたすべてのユーザーをサインアウトし (この場合は 1 人のユーザーのみ)、`TokenForUser` 値を無効にします。クライアント ライブラリを使用するバージョンでは、保存した `GraphServicesClient` も無効になり、また REST バージョンではアプリのローミング設定に保存した値が無効になります。

***クライアント ライブラリ バージョン***

これは `Signout` メソッドのクライアント ライブラリ バージョンです。

```c#
        public static void SignOut()
        {
            foreach (var user in IdentityClientApp.Users)
            {
                user.SignOut();
            }
            graphClient = null;
            TokenForUser = null;

        }
``` 

***REST バージョン***

これは `Signout` メソッドの REST バージョンです。

```c#
        public static void SignOut()
        {
            foreach (var user in IdentityClientApp.Users)
            {
                user.SignOut();
            }

            TokenForUser = null;

            //Clear stored values from last authentication.
            _settings.Values["userID"] = null;
            _settings.Values["userEmail"] = null;
            _settings.Values["userName"] = null;

        }
```

**GetAuthenticatedClient (クライアント ライブラリのみ)**

最後に、クライアント ライブラリを使用している場合は、`GraphServicesClient` を作成するメソッドが必要です。このメソッドは、Microsoft Graph に対するクライアント経由でのすべての呼び出しに対して `GetTokenForUserAsync` を使用するクライアントを作成します。

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

``ComposeAndSendMailAsync`` メソッドは、MainPage.xaml.cs ファイルによって渡される 3 つの文字列値 (``subject``、``bodyContent``、``recipients``) を取得します。``subject`` および ``bodyContent`` の文字列は、その他のすべての UI 文字列とともに Resources.resw ファイルに格納されます。``recipients`` の文字列はアプリのインターフェイスにあるアドレス ボックスのものです。 

**REST バージョン**

このファイルの REST バージョンには次の `using` 宣言が必要です。

```c#
using System;
using System.Threading.Tasks;
```

アドレスは複数渡すことができるため、最初に、要求の POST 本文に入れて渡せるように、``recipients`` 文字列を一連の EmailAddress オブジェクトに分割します。

```c#
            // Prepare the recipient list
            string[] splitter = { ";" };
            var splitRecipientsString = recipients.Split(splitter, StringSplitOptions.RemoveEmptyEntries);
            string recipientsJSON = null;

            int n = 0;
            foreach (string recipient in splitRecipientsString)
            {
                if ( n==0)
                recipientsJSON += "{'EmailAddress':{'Address':'" + recipient.Trim() + "'}}";
                else
                {
                    recipientsJSON += ", {'EmailAddress':{'Address':'" + recipient.Trim() + "'}}";
                }
                n++;
            }
```

次に、有効な JSON メッセージ オブジェクトを作成し、HTTP POST 要求を使用して **me/microsoft.graph.SendMail** エンドポイントに送信します。``bodyContent`` の文字列は HTML 文書であるため、要求によって **ContentType** 値が HTML に設定されます。また、``AuthenticationHelper.GetTokenHelperAsync`` への呼び出しにより、要求に渡される新規のアクセス トークンがあることを確認します。

```c#
                HttpClient client = new HttpClient();
                var token = await AuthenticationHelper.GetTokenHelperAsync();
                client.DefaultRequestHeaders.Add("Authorization", "Bearer " + token);

                // Build contents of post body and convert to StringContent object.
                // Using line breaks for readability.
                string postBody = "{'Message':{" 
                    +  "'Body':{ " 
                    + "'Content': '" + bodyContent + "'," 
                    + "'ContentType':'HTML'}," 
                    + "'Subject':'" + subject + "'," 
                    + "'ToRecipients':[" + recipientsJSON +  "]}," 
                    + "'SaveToSentItems':true}";

                var emailBody = new StringContent(postBody, System.Text.Encoding.UTF8, "application/json");

                HttpResponseMessage response = await client.PostAsync(new Uri("https://graph.microsoft.com/v1.0/me/microsoft.graph.SendMail"), emailBody);

                if ( !response.IsSuccessStatusCode)
                {

                    throw new Exception("We could not send the message: " + response.StatusCode.ToString());
                }
```

完全なクラスは、次のようになります。

```c#
    class MailHelper
    {
        /// <summary>
        /// Compose and send a new email.
        /// </summary>
        /// <param name="subject">The subject line of the email.</param>
        /// <param name="bodyContent">The body of the email.</param>
        /// <param name="recipients">A semicolon-separated list of email addresses.</param>
        /// <returns></returns>
        internal async Task ComposeAndSendMailAsync(string subject,
                                                            string bodyContent,
                                                            string recipients)
        {

            // Prepare the recipient list
            string[] splitter = { ";" };
            var splitRecipientsString = recipients.Split(splitter, StringSplitOptions.RemoveEmptyEntries);
            string recipientsJSON = null;

            int n = 0;
            foreach (string recipient in splitRecipientsString)
            {
                if ( n==0)
                recipientsJSON += "{'EmailAddress':{'Address':'" + recipient.Trim() + "'}}";
                else
                {
                    recipientsJSON += ", {'EmailAddress':{'Address':'" + recipient.Trim() + "'}}";
                }
                n++;
            }

            try
            {

                HttpClient client = new HttpClient();
                var token = await AuthenticationHelper.GetTokenForUserAsync();
                client.DefaultRequestHeaders.Add("Authorization", "Bearer " + token);

                // Build contents of post body and convert to StringContent object.
                // Using line breaks for readability.
                string postBody = "{'Message':{" 
                    +  "'Body':{ " 
                    + "'Content': '" + bodyContent + "'," 
                    + "'ContentType':'HTML'}," 
                    + "'Subject':'" + subject + "'," 
                    + "'ToRecipients':[" + recipientsJSON +  "]}," 
                    + "'SaveToSentItems':true}";

                var emailBody = new StringContent(postBody, System.Text.Encoding.UTF8, "application/json");

                HttpResponseMessage response = await client.PostAsync(new Uri("https://graph.microsoft.com/v1.0/me/microsoft.graph.SendMail"), emailBody);

                if ( !response.IsSuccessStatusCode)
                {

                    throw new Exception("We could not send the message: " + response.StatusCode.ToString());
                }


            }

            catch (Exception e)
            {
                throw new Exception("We could not send the message: " + e.Message);
            }
        }
    }
```

**クライアント ライブラリ バージョン**

このファイルのクライアント バージョンには次の `using` 宣言が必要です。

```c#
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
```

クライアント ライブラリを使用してメッセージを送信するためのコードは、REST 呼び出しを使用するコードによく似ています。JSON オブジェクトを作成して、HTTP POST 要求の **SendMail** エンドポイントに直接渡す代わりに、クライアント ライブラリで定義される同等のオブジェクトを作成し、結果の `Message` オブジェクトを `GraphServiceClient` の `SendMail` メソッドに渡します。クライアントは、アクセス トークンを取得する作業と、要求を **SendMail** エンドポイントに渡す作業を行います。

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

##<a name="create-the-user-interface-in-mainpage.xaml"></a>MainPage.xaml でユーザー インターフェイスを作成する

これで Microsoft Graph 経由でユーザーを認証して、メッセージを送信するためすべての作業を実行するコードを記述できました。あとは、前述の簡単なインターフェイスを作成するだけです。 

スターター プロジェクトの MainPage.xaml ファイルには、必要なすべての XAML が既に含まれています。必要な作業は、インターフェイスを機能させるコードを MainPage.xaml.cs ファイルに追加するだけです。スターター プロジェクトでこのファイルを見つけて、開きます。

このファイルには、クライアント ライブラリと REST バージョンの両方のサンプルに必要なすべての `using` 宣言が既に含まれています。

***クライアント ライブラリ バージョン***

クライアント ライブラリ バージョンのアプリは、ユーザーが認証するときに `GraphServiceClient` を作成します。 

名前空間内にこのコードを追加して、MainPage.xaml.cs の MainPage クラスのクライアント ライブラリ バージョンを完了します。

```c#
    public sealed partial class MainPage : Page
    {
        private string _mailAddress;
        private string _displayName = null;
        private MailHelper _mailHelper = new MailHelper();

        public MainPage()
        {
            this.InitializeComponent();
        }

        protected override void OnNavigatedTo(NavigationEventArgs e)
        {
            // Developer code - if you haven't registered the app yet, we warn you. 
            if (!App.Current.Resources.ContainsKey("ida:ClientID"))
            {
                InfoText.Text = ResourceLoader.GetForCurrentView().GetString("NoClientIdMessage");
                ConnectButton.IsEnabled = false;
            }
            else
            {
                InfoText.Text = ResourceLoader.GetForCurrentView().GetString("ConnectPrompt");
                ConnectButton.IsEnabled = true;
            }
        }

        /// <summary>
        /// Signs in the current user.
        /// </summary>
        /// <returns></returns>
        public async Task<bool> SignInCurrentUserAsync()
        {
            var graphClient = AuthenticationHelper.GetAuthenticatedClient();

            if (graphClient != null)
            {
                var user = await graphClient.Me.Request().GetAsync();
                string userId = user.Id;
                _mailAddress = user.UserPrincipalName;
                _displayName = user.DisplayName;
                return true;
            }
            else
            {
                return false;
            }

        }


        private async void ConnectButton_Click(object sender, RoutedEventArgs e)
        {
            ProgressBar.Visibility = Visibility.Visible;
            if (await SignInCurrentUserAsync())
            { 
                InfoText.Text = "Hi " + _displayName + "," + Environment.NewLine + ResourceLoader.GetForCurrentView().GetString("SendMailPrompt");
                MailButton.IsEnabled = true;
                EmailAddressBox.IsEnabled = true;
                ConnectButton.Visibility = Visibility.Collapsed;
                DisconnectButton.Visibility = Visibility.Visible;
                EmailAddressBox.Text = _mailAddress;
            }
            else
            {
                InfoText.Text = ResourceLoader.GetForCurrentView().GetString("AuthenticationErrorMessage");
            }

            ProgressBar.Visibility = Visibility.Collapsed;
        }

        private async void MailButton_Click(object sender, RoutedEventArgs e)
        {
            _mailAddress = EmailAddressBox.Text;
            ProgressBar.Visibility = Visibility.Visible;
            MailStatus.Text = string.Empty;
            try
            {
                await _mailHelper.ComposeAndSendMailAsync(ResourceLoader.GetForCurrentView().GetString("MailSubject"), ComposePersonalizedMail(_displayName), _mailAddress);
                MailStatus.Text = string.Format(ResourceLoader.GetForCurrentView().GetString("SendMailSuccess"), _mailAddress);
            }
            catch (Exception)
            {
                MailStatus.Text = ResourceLoader.GetForCurrentView().GetString("MailErrorMessage");
            }
            finally
            {
                ProgressBar.Visibility = Visibility.Collapsed;
            }
            
        }

        // <summary>
        // Personalizes the email.
        // </summary>
        public static string ComposePersonalizedMail(string userName)
        {
            return String.Format(ResourceLoader.GetForCurrentView().GetString("MailContents"), userName);
        }

        private void Disconnect_Click(object sender, RoutedEventArgs e)
        {
            ProgressBar.Visibility = Visibility.Visible;
            AuthenticationHelper.SignOut();
            ProgressBar.Visibility = Visibility.Collapsed;
            MailButton.IsEnabled = false;
            EmailAddressBox.IsEnabled = false;
            ConnectButton.Visibility = Visibility.Visible;
            InfoText.Text = ResourceLoader.GetForCurrentView().GetString("ConnectPrompt");
            this._displayName = null;
            this._mailAddress = null;
        }
    }
```

***REST バージョン***

このクラスの REST バージョンは、ユーザーが認証するときに `GetTokenForUserAsync` メソッドを直接呼び出すことを除いて、クライアント バージョンとよく似ています。また、アプリのローミング設定からユーザーの値も取得します。 

名前空間内にこのコードを追加して、MainPage.xaml.cs ｄの MainPage クラスの REST バージョンを完了します。

```c#
    public sealed partial class MainPage : Page
    {
        private string _mailAddress;
        private string _displayName = null;
        private MailHelper _mailHelper = new MailHelper();
        public static ApplicationDataContainer _settings = ApplicationData.Current.RoamingSettings;

        public MainPage()
        {
            this.InitializeComponent();
        }

        protected override void OnNavigatedTo(NavigationEventArgs e)
        {
            // Developer code - if you haven't registered the app yet, we warn you. 
            if (!App.Current.Resources.ContainsKey("ida:ClientID"))
            {
                InfoText.Text = ResourceLoader.GetForCurrentView().GetString("NoClientIdMessage");
                ConnectButton.IsEnabled = false;
            }
            else
            {
                InfoText.Text = ResourceLoader.GetForCurrentView().GetString("ConnectPrompt");
                ConnectButton.IsEnabled = true;
            }
        }

        /// <summary>
        /// Signs in the current user.
        /// </summary>
        /// <returns></returns>
        public async Task<bool> SignInCurrentUserAsync()
        {
            var token = await AuthenticationHelper.GetTokenForUserAsync();

            if (token != null)
            {
                string userId = (string)_settings.Values["userID"];
                _mailAddress = (string)_settings.Values["userEmail"];
                _displayName = (string)_settings.Values["userName"];
                return true;
            }
            else
            {
                return false;
            }

        }


        private async void ConnectButton_Click(object sender, RoutedEventArgs e)
        {
            ProgressBar.Visibility = Visibility.Visible;
            if (await SignInCurrentUserAsync())
            { 
                InfoText.Text = "Hi " + _displayName + "," + Environment.NewLine + ResourceLoader.GetForCurrentView().GetString("SendMailPrompt");
                MailButton.IsEnabled = true;
                EmailAddressBox.IsEnabled = true;
                ConnectButton.Visibility = Visibility.Collapsed;
                DisconnectButton.Visibility = Visibility.Visible;
                EmailAddressBox.Text = _mailAddress;
            }
            else
            {
                InfoText.Text = ResourceLoader.GetForCurrentView().GetString("AuthenticationErrorMessage");
            }

            ProgressBar.Visibility = Visibility.Collapsed;
        }

        private async void MailButton_Click(object sender, RoutedEventArgs e)
        {
            _mailAddress = EmailAddressBox.Text;
            ProgressBar.Visibility = Visibility.Visible;
            MailStatus.Text = string.Empty;
            try
            {
                await _mailHelper.ComposeAndSendMailAsync(ResourceLoader.GetForCurrentView().GetString("MailSubject"), ComposePersonalizedMail(_displayName), _mailAddress);
                MailStatus.Text = string.Format(ResourceLoader.GetForCurrentView().GetString("SendMailSuccess"), _mailAddress);
            }
            catch (Exception)
            {
                MailStatus.Text = ResourceLoader.GetForCurrentView().GetString("MailErrorMessage");
            }
            finally
            {
                ProgressBar.Visibility = Visibility.Collapsed;
            }
            
        }

        // <summary>
        // Personalizes the email.
        // </summary>
        public static string ComposePersonalizedMail(string userName)
        {
            return String.Format(ResourceLoader.GetForCurrentView().GetString("MailContents"), userName);
        }

        private void Disconnect_Click(object sender, RoutedEventArgs e)
        {
            ProgressBar.Visibility = Visibility.Visible;
            AuthenticationHelper.SignOut();
            ProgressBar.Visibility = Visibility.Collapsed;
            MailButton.IsEnabled = false;
            EmailAddressBox.IsEnabled = false;
            ConnectButton.Visibility = Visibility.Visible;
            InfoText.Text = ResourceLoader.GetForCurrentView().GetString("ConnectPrompt");
            this._displayName = null;
            this._mailAddress = null;
        }
    }
```
 
これで、Microsoft Graph と対話するために必要な、アプリの登録、ユーザーの認証、および要求の作成の 3 つの手順が完了しました。 

## <a name="run-the-app"></a>アプリの実行
1. F5 キーを押して、アプリを構築して実行します。 

2. 個人用あるいは職場または学校アカウントでサインインし、要求されたアクセス許可を付与します。

3. **[メールの送信]** ボタンを選びます。メールが送信されると、ボタンの下に成功メッセージが表示されます。

## <a name="next-steps"></a>次の手順
- [Graph エクスプローラー](https://graph.microsoft.io/graph-explorer)を使用して REST API を試してみます。
- [Microsoft Graph UWP スニペット サンプル (SDK)](https://github.com/microsoftgraph/uwp-csharp-snippets-sample) と [Microsoft Graph UWP スニペット サンプル (REST)](https://github.com/microsoftgraph/uwp-csharp-snippets-rest-sample) での REST と SDK の両方の操作に共通する操作の例を検索するか、GitHub で他の [UWP サンプル](https://github.com/microsoftgraph?utf8=%E2%9C%93&query=uwp)を探索します。

## <a name="see-also"></a>関連項目
- [Microsoft Graph .NET クライアント ライブラリ](https://github.com/microsoftgraph/msgraph-sdk-dotnet)
- [Azure AD v2.0 のプロトコル](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
- [Azure AD v2.0 のトークン](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)

