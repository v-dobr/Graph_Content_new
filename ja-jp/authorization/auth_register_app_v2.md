## <a name="register-your-microsoft-graph-application-with-the-azure-ad-v2.0-endpoint"></a>Microsoft Graph アプリケーションを Azure AD v2.0 エンドポイントに登録する

Azure AD v2.0 エンドポイントを使用するには、アプリを [Microsoft アプリ登録ポータル](https://apps.dev.microsoft.com) (https://apps.dev.microsoft.com) に登録する必要があります。アプリを登録すると、認証プロバイダーによってそのアプリの ID が確立され、ユーザーからの認証要求を送信するとき、アプリは自身の ID を証明できるようになります。登録すると、アプリで認証を構成する際に使用するアプリ ID とアプリケーション シークレットが生成されます。

> **注:**この記事では、Azure AD v2.0 エンドポイントへのアプリ登録について説明します。[Azure AD にアプリを登録する](app_authentication_azure_ad.md)には、[Azure ポータル](https://aka.ms/aadapplist)を使用します。
> 
> 既に Microsoft Azure 管理ポータルに登録済みのアプリはアプリ登録ポータルには表示されないので、ご注意ください。それらのアプリは Azure 管理ポータルで管理します。 

1. 個人用アカウント、あるいは職場または学校アカウントのいずれかを使用して、[Microsoft アプリ登録ポータル](https://apps.dev.microsoft.com/)にサインインします。

2. **[アプリの追加]** を選択します。

3. アプリの名前を入力して、**[アプリケーションの作成]** を選択します。

    登録ページが表示され、アプリのプロパティが一覧表示されます。

4. アプリケーション ID をコピーします。これは、アプリの一意識別子です。

    アプリを構成するには、アプリケーション ID を使用します。

5. **[プラットフォーム]** で、**[プラットフォームの追加]** を選び、アプリに適したプラットフォームを選びます。
    
    クライアント アプリの場合:
    1. **[Mobile プラットフォーム]** を選びます。

    2. クライアント ID (アプリ ID) とリダイレクト URI の値の両方をクリップボードにコピーします。サンプル アプリにこれらの値を入力する必要があります。

        アプリ ID は、アプリの一意識別子です。リダイレクト URI は、各アプリケーションに提供される一意の URI であり、その URI に送信されたメッセージはそのアプリケーションだけに送信されます。 

    Web アプリの場合:
    1. **[Web]** を選びます。
    2. 暗黙的な許可の種類を使用している場合、または OpenID Connect ハイブリッド フローを使用している場合は、必ず [暗黙的フローを許可する] のチェック ボックスを選びます。 
        
        [暗黙的フローを許可する] オプションにより、OpenID Connect ハイブリッド フローが有効になります。これにより、認証時にアプリはサインイン情報 (id_token) と成果物 (この場合は認証コード) の両方を受け取れるようになり、アプリはアクセス トークンを取得するときにこれらを使用できます。


    3. リダイレクト URI を指定します。
        
        リダイレクト URI は、Azure AD v2.0 エンドポイントが認証要求処理を完了した際に呼び出す、アプリ内の場所です。
    4. **[アプリケーション シークレット]** で、**[新しいパスワードを生成する]** を選びます。**[新しいパスワードが生成されました]** ダイアログ ボックスからアプリケーション シークレットをコピーします。
        
        アプリを構成するために、アプリケーション シークレットを使用します。
    
6. **[保存]** を選びます。

## <a name="see-also"></a>関連項目

[Microsoft Graph でのアプリ認証](auth_overview.md)