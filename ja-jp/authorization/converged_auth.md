# <a name="authenticate-microsoft-graph-apps-with-the-azure-ad-v2.0-endpoint"></a>Microsoft Graph アプリを Azure AD v2.0 エンドポイントで認証する

> **エンタープライズのお客様向けにアプリを作成していますか?**エンタープライズのお客様が、<a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-conditional-access-device-policies/" target="_newtab">デバイスへの条件付きアクセス</a>のようなエンタープライズ モビリティ セキュリティ機能をオンにしている場合、アプリが動作しない可能性があります。  

> **すべてのエンタープライズのお客様**の**すべてのエンタープライズ シナリオ**をサポートするには、Azure AD エンドポイントを使用し、[Azure 管理ポータル](https://aka.ms/aadapplist)でアプリを管理する必要があります。詳細については、「[Azure AD か Azure AD v2.0 エンドポイントかを決定する](auth_overview.md#deciding-between-azure-ad-and-the-v2-authentication-endpoint)」を参照してください。


Azure AD v2.0 エンドポイントを使用すると、職場または学校 (Azure Active Directory) ID と、個人 (Microsoft アカウント) ID の両方を受け入れるアプリを作成できます。

以前は、Microsoft アカウントと Azure Active Directory の両方をサポートするアプリを開発する場合は、完全に独立した 2 つのシステムと統合する必要がありました。Azure AD v2.0 エンドポイントを使用すると、両方の種類のアカウントのサポートを 1 つに統合できます。1 つの簡単な手順で、個人のアカウントと、仕事/学校のアカウントの両方の数百万に及ぶユーザーを対象とすることができます。  

アプリを Azure AD v2.0 エンドポイントに統合すると、以下に示すような、個人アカウントと職場または学校アカウントの両方で利用できる Microsoft Graph エンドポイントに即座にアクセスできます。 

| データ              | エンドポイント                                       |
|:------------------|:-----------------------------------------------|
| ユーザー プロファイル      | `https://graph.microsoft.com/v1.0/me`          |
| Outlook のメール      | `https://graph.microsoft.com/v1.0/me/messages` |
| Outlook の連絡先  | `https://graph.microsoft.com/v1.0/me/contacts` |
| Outlook の予定表 | `https://graph.microsoft.com/v1.0/me/events`   |
| OneDrive          | `https://graph.microsoft.com/v1.0/me/drive`    |

 >**注:**グループやタスクなどのいくつかの Microsoft Graph エンドポイントは、個人用のアカウントを対象としていません。  

## <a name="microsoft-graph-authentication-scopes"></a>Microsoft Graph 認証のスコープ

Azure AD v2.0 エンドポイントは、「[Microsoft Graph のアクセス許可スコープ](permission_scopes.md)」に一覧表示されているすべてのアクセス許可スコープをサポートしています。 

Azure AD v2.0 エンドポイントでのスコープの使用に関する詳細、および Azure AD でのリソースの使用との違いについては、「<a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/#scopes-not-resources" target="_newtab">リソースではなく、スコープ</a>」をご覧ください。

## <a name="see-it-in-action"></a>実際に確認しましょう。

「[Microsoft Graph リポジトリの Connect サンプル](https://github.com/microsoftgraph?utf8=%E2%9C%93&query=connect)」には、さまざまなプラットフォームでユーザーを認証し、Microsoft Graph に接続する方法を示す簡単な例が記されています。

また、「[作業の開始](http://graph.microsoft.io/en-us/docs/platform/get-started)」セクションには、各プラットフォームで使用される認証ライブラリなど、サンプル アプリを作成する方法を説明した記事が収められています。

## <a name="see-also"></a>関連項目

- [アプリを Azure AD v2.0 エンドポイントに登録する](auth_register_app_v2.md)
- [Microsoft Graph でのアプリ認証](auth_overview.md)
- <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare" target="_newtab">Azure AD v2.0 モデルの新機能</a>
- <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/" target="_newtab">Azure AD v2.0 エンドポイントを使用したほうがいいでしょうか?</a>
- <a href="https://azure.microsoft.com/en-us/documentation/articles/?product=active-directory&term=azure+ad+v2.0" target="_newtab">Azure.com にある Azure AD v2.0 エンドポイントに関する文書</a>
- <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-app-registration/#build-a-quick-start-app" target="_newtab">Azure.com にある Azure AD v2.0 コード クイック スタート</a>

