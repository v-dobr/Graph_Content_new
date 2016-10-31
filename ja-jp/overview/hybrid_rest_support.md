# <a name="use-rest-apis-to-access-mailboxes-in-exchange-hybrid-deployments-(preview)"></a>REST API を使用して Exchange ハイブリッド展開のメールボックスにアクセスする (プレビュー)

Microsoft Graph では、Office 365 の一部としての Exchange Online で、クラウドにある顧客メールボックスへのアクセスをずっと提供してきました。2016 年 9 月にリリースされた Exchange On-Premises サーバー用の Exchange 2016 Cumulative Update 3 (CU3) では、Office 365 との REST API の統合のサポートが追加されています。アプリで v1.0 の[メール](http://graph.microsoft.io/en-us/docs/api-reference/v1.0/resources/message)、[予定表](http://graph.microsoft.io/en-us/docs/api-reference/v1.0/resources/calendar)、または[連絡先](http://graph.microsoft.io/en-us/docs/api-reference/v1.0/resources/contact) API を使用している場合、_ハイブリッド_展開で認証やアプリケーションのエクスペリエンスがシームレスとなり、展開が特定の[要件](#requirements-for-rest-api-to-work-in-hybrid-deployments)を満たしていれば、メールボックスがオンプレミスとクラウドのどちらにあろうと関係なくなりました。 


舞台裏では、ハイブリッド展開のオンプレミスのメールボックスに REST API 呼び出しがアクセスしようとしていることを Microsoft Graph が識別すると、Microsoft Graph は REST 要求をオンプレミスの REST エンドポイントに送り、エンドポイントがその要求を処理します。この検出により、REST API へのアクセスが可能になります。

>**注:**ハイブリッド展開でこれらの REST API を使用する機能は、現在プレビュー段階です。

>ハイブリッド展開のメールボックスでは、v1.0 のメール、予定表、連絡先 API のみを使用できます。[グループ](http://graph.microsoft.io/en-us/docs/api-reference/v1.0/resources/group) API など、他の v1.0 API セットや、他のバージョンの API は使用できません。ハイブリッド展開でサポートされているセットに属さない API を使用しようとすると、次のエラー メッセージが表示されます。

>"このメールボックスの REST API は現在プレビュー段階です。REST API (プレビュー) の詳細については、https://dev.outlook.com をご覧ください。"

## <a name="requirements-for-the-rest-api-to-work-in-hybrid-deployments"></a>ハイブリッド展開で REST API が作動する要件

Microsoft Graph はオープンネス (JSON、OAUTH、ODATA といったオープンな標準をサポートし、最も人気のあるプラットフォームから接続可能)、および柔軟性 (ユーザー データにアクセスするための段階的で、厳格に範囲指定されたアクセス許可) を提供します。組織が Microsoft Graph アプリ開発の有効化に興味があり、現在、ハイブリッド展開を実装している、または実装を検討している場合、次の展開要件に注意してください。

- メールボックスの要件

  - REST API を使用するオンプレミスのメールボックスはすべて Exchange 2016 CU3 サーバー上のデータベースに配置される必要があります。 

- インフラストラクチャの要件

  - すべての Exchange 2016 サーバーを CU3 以降にアップグレードする必要があります。  
  - オンプレミスの Active Directory を Azure Active Directory と同期させる必要があります。
  - Exchange 2016 サーバーと同じ負荷分散アレイにおいて共存するすべての Exchange 2013 サーバーを、アレイから削除する必要があります。

- ネットワークの要件

  - DNS の観点から、自動検出の名前空間およびオンプレミスのクライアント名前空間には、インターネットの DNS レコードが含まれている必要があります。 
  - アクセスを検査および制限するファイアウォールまたはアプリケーション ゲートウェイがある場合は、適切な設定を更新して、検出およびアクセスを許可します。


IT 管理者は、次のリソースで詳細情報を確認できます。

- 
  [Exchange Server のハイブリッド展開](https://technet.microsoft.com/en-us/library/jj200581(v=exchg.150).aspx)
- [2016 年 9 月の Cumulative Update リリース](https://blogs.technet.microsoft.com/exchange/2016/09/20/released-september-2016-quarterly-exchange-updates/) 
- [REST API のオンプレミス アーキテクチャ要件](https://blogs.technet.microsoft.com/exchange/2016/09/26/on-premises-architectural-requirements-for-the-rest-api/)
