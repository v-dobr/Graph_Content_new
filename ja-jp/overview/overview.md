# <a name="overview-of-microsoft-graph"></a>Microsoft Graph の概要

Microsoft Graph は 1 つのエンドポイントを介して複数の API を Office 365 およびその他の Microsoft クラウド サービスから公開するものです (**https://graph.microsoft.com**)。Microsoft Graph は複雑なクエリを簡略化します。 
 
Microsoft Graph を使用して次のことができます。

- Azure Active Directory、Office 365 の一部としての Exchange Online、SharePoint、OneDrive、OneNote、Planner を含む、複数の Microsoft クラウド サービスからデータにアクセスします。
- エンティティ間およびリレーションシップ間を移動します。
- Microsoft クラウドからのインテリジェンスとインサイトにアクセスします (ビジネス ユーザー向け)。

**Microsoft Graph の開発のスタック**

![Microsoft Graph 開発のスタックの層を示す図。最下部はデータ層で、ユーザー、グループ、ファイル、メール、予定表、個人用連絡先、タスク、組織の連絡先、人、Excel、メモが含まれています。次の層は認証と承認です。次は自分で選ぶ開発環境で、Android、iOS、Visual Studio の Microsoft Graph API SDK が含まれています。最上部の層はソリューションで、.NET、JS、HTML、Ruby など、選んだテクノロジを使用し、Microsoft Azure か別のホスト プラットフォーム上でホストされます。](./images/MicrosoftGraph_DevStack.png)

<!--<a name="msg_queries"> </a>-->

##<a name="common-microsoft-graph-queries"></a>一般的な Microsoft Graph のクエリ

Microsoft Graph では、2 つのエンドポイント (/v1.0 および /beta) を公開します。/V1.0 エンドポイントには、運用アプリ内でアクセスできるリソースが含まれます。[/beta](http://graph.microsoft.io/en-us/docs/api-reference/beta/beta-overview) エンドポイントには、現在プレビュー段階の API が含まれます。次の表は、Microsoft Graph API にアクセスするために使用できるいくつかの一般的なクエリの一覧を示します。

| **操作** | **サービス エンドポイント** |
|:--------------------------|:----------------------------------------|
|   自分のプロファイルの取得 |    [`https://graph.microsoft.com/v1.0/me`](/graph-explorer/#?request=me&version=v1.0) |
|   自分のファイルの取得 | [`https://graph.microsoft.com/v1.0/me/drive/root/children`](/graph-explorer/#?request=me%2Fdrive%2Froot%2Froot%2Fchildren&version=v1.0) |
|   自分の写真の取得	     | [`https://graph.microsoft.com/v1.0/me/photo/$value`](/graph-explorer/#?request=me%2Fphoto%2F%24value&version=v1.0) |
|   自分のメールの取得 |   [`https://graph.microsoft.com/v1.0/me/messages`](/graph-explorer/#?request=me%2Fmessages&version=v1.0) |
|   自分にとって重要度の高いメールの取得 | [`https://graph.microsoft.com/v1.0/me/messages?$filter=importance%20eq%20'high'`](/graph-explorer/#?request=me%2Fmessages%3F%24filter%3Dimportance%2520eq%2520'high'&version=v1.0) |
|   自分の予定表の取得 |   [`https://graph.microsoft.com/v1.0/me/calendar`](/graph-explorer/#?request=me%2Fcalendar&version=v1.0) |
|   自分の上司の取得	  | [`https://graph.microsoft.com/v1.0/me/manager`](/graph-explorer/#?request=me%2Fmanager&version=v1.0) |
|   foo.txt ファイルを最後に変更したユーザーの取得 |  [`https://graph.microsoft.com/v1.0/me/drive/root/children/foo.txt/lastModifiedByUser`](/graph-explorer/#?request=me%2Fdrive%2Froot%2Froot%2Fchildren%2Ffoo.txt%2FlastModifiedByUser&version=v1.0) |
|   自分がメンバーになっている統合グループの取得|   [`https://graph.microsoft.com/v1.0/me/memberOf/$/microsoft.graph.group?$filter=groupTypes/any(a:a%20eq%20'unified')`](/graph-explorer/#?request=me%2FmemberOf%2F%24%2Fmicrosoft.graph.group%3F%24filter%3DgroupTypes%2Fany(a%3Aa%2520eq%2520'unified'&version=v1.0)) |
|   自分の所属組織のユーザーの取得	     | [`https://graph.microsoft.com/v1.0/users`](/graph-explorer/#?request=users&version=v1.0) |
|   グループ会話の取得 |   `https://graph.microsoft.com/v1.0/groups/<id>/conversations`|
|   自分に関連付けられたユーザーの取得    | [`https://graph.microsoft.com/beta/me/people`](/graph-explorer/#?request=me%2Fpeople&version=beta)  |
|   自分の周りで人気上昇中のファイルの取得 |  [`https://graph.microsoft.com/beta/me/trendingAround`](/graph-explorer/#?request=me%2FtrendingAround&version=beta) |
|   一緒に作業しているユーザーの取得     | [`https://graph.microsoft.com/beta/me/workingWith`](/graph-explorer/#?request=me%2FworkingWith&version=beta) |
|   自分のタスクの取得    | [`https://graph.microsoft.com/beta/me/tasks`](/graph-explorer/#?request=me%2Ftasks&version=beta) |
|   自分のノートの取得 |  [`https://graph.microsoft.com/beta/me/notes/notebooks`](/graph-explorer/#?request=me%2Fnotes%2Fnotebooks&version=beta) |


>**注:**ベータ版のエンドポイントの API は予告なしに変更されることがあります。運用アプリ内で使用することはお勧めしません。 

<!-- <a name="msg_roof"> </a> -->

## <a name="explore-microsoft-graph"></a>Microsoft Graph の使い方

- Microsoft Graph と選択したプラットフォームの使用を[開始](../get-started/get-started.md)します。
- TOC を参照して、運用アプリ内で使用できるリソースおよび操作を探索します。
- 新しい [/beta API](http://graph.microsoft.io/en-us/docs/api-reference/beta/beta-overview) をプレビューします。
- [Microsoft Graph エクスプローラー](https://graph.microsoft.io/en-us/graph-explorer)にアクセスします。

 >  お客様からのフィードバックを重視しています。[Stack Overflow](http://stackoverflow.com/questions/tagged/office365+or+microsoftgraph)でご連絡いただけます。質問には [MicrosoftGraph] と [office365] でタグ付けしてください。



