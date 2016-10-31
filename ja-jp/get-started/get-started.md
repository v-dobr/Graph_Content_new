# <a name="getting-started-building-microsoft-graph-apps"></a>Microsoft Graph アプリの構築を開始する

このセクションの記事では、Microsoft Graph に接続するアプリをさまざまな言語および開発プラットフォームで構築する方法を詳しく説明します。各記事では、まず該当するプラットフォームのサンプル スターター プロジェクトを説明し、次いでユーザー認証の機能を追加する手順を説明し、そのアカウントから Microsoft Graph でメールを送信するためのサンプル要求を作成します。完成したプロジェクトは、そのプラットフォーム用の [Microsoft Graph リポジトリの Connect サンプル](https://github.com/microsoftgraph?utf8=%E2%9C%93&query=connect)とまったく同じものです。

好みの認証プロバイダーと開発プラットフォームについて説明した記事を選び、Microsoft Graph への接続を開始します。

選んだ開発プラットフォームについての記事に記載されている手順に従うことも、[クイック スタート](http://dev.office.com/getting-started/office365apis)機能を試してみて、動作するソリューションをすばやく稼働させることもできます。

完成した Connect サンプルを探すには、GitHub の [Microsoft Graph リポジトリ](https://github.com/microsoftgraph)にアクセスしてください。次の表は、認証プロバイダーとプラットフォームごとにサンプルをリストにしたもので、REST または Microsoft Graph クライアント ライブラリのどちらを使用して Microsoft Graph に接続するかを示しています。

<table>
  <tr>
    <th>プラットフォーム</th>
    <th>Azure AD エンドポイント</th> 
    <th>Azure AD v2.0 エンドポイント</th>
  </tr>
  <tr>
    <td>Android</td>
    <td>
        <a href="https://github.com/microsoftgraph/android-java-connect-rest-sample">REST サンプル</a>または <a href="https://github.com/microsoftgraph/android-java-connect-sample/tree/last_v1_auth">SDK サンプル</a>
    </td> 
    <td>
        <a href="https://github.com/microsoftgraph/android-java-connect-sample">SDK サンプル</a>
    </td> 
  </tr>
  <tr>
    <td>AngularJS</td>
    <td>
        <a href="https://github.com/microsoftgraph/angular-connect-rest-sample/tree/last_v1_auth">REST サンプル</a>
    </td> 
    <td>
        <a href="https://github.com/microsoftgraph/angular-connect-rest-sample">REST サンプル</a>または <a href="https://github.com/microsoftgraph/angular-connect-sample">SDK サンプル</a>
    </td> 
  </tr>
  <tr>
    <td>ASP.NET</td>
    <td>
        <a href="https://github.com/microsoftgraph/aspnet-connect-rest-sample">REST サンプル</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/aspnet-connect-sample">SDK サンプル</a>
    </td> 
  </tr>
  <tr>
    <td>iOS (Obj-C)</td>
    <td>
        <a href="https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample">REST サンプル</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/ios-objectivec-connect-sample">SDK サンプル</a>
    </td> 
  </tr>
  <tr>
    <td>iOS (Swift)</td>
    <td>
        <a href="https://github.com/microsoftgraph/ios-swift-connect-rest-sample">REST サンプル</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/ios-swift-connect-sample">SDK サンプル</a>
    </td> 
  </tr>
  <tr>
    <td>NodeJS</td>
    <td>
        <a href="https://github.com/microsoftgraph/nodejs-connect-rest-sample/tree/last_v1_auth">REST サンプル</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/nodejs-connect-rest-sample">REST サンプル</a>
    </td> 
  </tr>
  <tr>
    <td>PHP</td>
    <td>
        <a href="https://github.com/microsoftgraph/php-connect-rest-sample/tree/last_v1_auth">REST サンプル</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/php-connect-rest-sample">REST サンプル</a>
    </td> 
  </tr>
  <tr>
    <td>Python</td>
    <td>
        <a href="https://github.com/microsoftgraph/python3-connect-rest-sample">REST サンプル</a>
    </td>     
    <td>
    </td> 
  </tr>
  <tr>
    <td>Ruby</td>
    <td>
        <a href="https://github.com/microsoftgraph/ruby-connect-rest-sample/tree/last_v1_auth">REST サンプル</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/ruby-connect-rest-sample">REST サンプル</a>
    </td> 
  </tr>
  <tr>
    <td>UWP</td>
    <td>
        <a href="https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample/tree/last_v1_auth">REST サンプル</a>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample">REST サンプル</a>または <a href="https://github.com/microsoftgraph/uwp-csharp-connect-sample">SDK サンプル</a>
    </td> 
  </tr>
  <tr>
    <td>Xamarin</td>
    <td>
    </td>     
    <td>
        <a href="https://github.com/microsoftgraph/xamarin-csharp-connect-sample">SDK サンプル</a>
    </td> 
  </tr>
</table>

## <a name="see-also"></a>関連項目
- サンプルの REST 呼び出しを [API エクスプローラー](https://graph.microsoft.io/graph-explorer)で試してみてください。
- [Azure AD エンドポイントのドキュメント](https://azure.microsoft.com/en-us/documentation/services/active-directory/)
- [Azure AD v2.0 エンドポイントのドキュメント](https://azure.microsoft.com/en-us/documentation/articles/?service=active-directory&term=azure+ad+v2.0)
