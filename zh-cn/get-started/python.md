# <a name="get-started-with-microsoft-graph-in-a-python-app"></a>在 Python 应用中开始使用 Microsoft Graph 

本文介绍了从 Azure AD 获取访问令牌和调用 Microsoft Graph 所需的任务。本文演示了 [适用于 Python 的 Microsoft Graph Connect 示例](https://github.com/microsoftgraph/python3-connect-rest-sample)，并说明使用 Microsoft Graph API 要实现的主要概念。本文介绍了如何使用 REST 直接调用来访问 Microsoft Graph。

![Office 365 Python Connect 示例屏幕截图](./images/web-screenshot.png)

> **注意** 截图基于的演练和示例使用了 Azure AD 终结点。请稍后再查阅使用 Azure AD v2.0 终结点的更新版本。

##  <a name="prerequisites"></a>先决条件

  * Office 365 商业版帐户。可以注册 [Office 365 开发人员订阅](https://msdn.microsoft.com/en-us/office/office365/howto/setup-development-environment#bk_Office365Account)，其中包含开始生成 Office 365 应用所需的资源。
  * [适用于 Python 的 Microsoft Graph Connect 示例](https://github.com/microsoftgraph/python3-connect-rest-sample)

## <a name="register-the-application-in-azure-active-directory"></a>在 Azure Active Directory 中注册应用程序

首先，需要注册应用程序并设置 Microsoft Graph 的使用权限。此操作允许用户使用工作或学校帐户登录应用程序。

1. 登录到 [Azure 门户](https://portal.azure.com/)。
2. 在顶栏中，单击帐户并在**目录**列表下，选择希望在其中注册应用的 Active Directory 租户。
3. 单击左侧导航中的“**更多服务**”，并选择 **Azure Active Directory**。
4. 单击“**应用注册**”，然后选择“**添加**”。
5. 为该应用程序输入一个友好名称，例如“MSGraphConnectPython”，然后选择“Web 应用/API”作为“**应用程序类型**”。至于登录 URL，输入“http://127.0.0.1:8000/connect/get_token/”。单击“**创建**”来创建应用程序。
6. 仍在 Azure 门户中，选择应用程序，单击“**设置**”，然后选择“**属性**”。
7. 查找“应用程序 ID”值并将其复制到剪贴板。
8. 配置应用程序的权限：
9. 在“**设置**”菜单中，选择“**所需的权限**”部分，单击“**添加**”，然后单击“**选择 API**”，并选择“**Microsoft Graph**。
10. 然后，单击“**选择权限**”，选择“登录并读取用户个人资料”和“**以用户身份发送邮件**”。单击“**选择**”，然后单击“**完成**”。
11. 在“**设置**”菜单中，选择“**密钥**”部分。输入说明并选择该密钥的有效期。单击“**保存**”。
12. **重要提示**：复制密钥值。离开此窗格后将无法再次访问该值。将该值用作应用密钥。

## <a name="redirect-the-browser-to-the-sign-in-page"></a>将浏览器重定向到登录页

你的应用程序需要将浏览器重定向到登录页开始 OAuth 流程并获得授权代码。 

在 Connect 示例中，以下代码（位于 [*connect/auth_helper.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/auth_helper.py)）将生成应用需要将用户重定向到的 URL 并发送到它可用于重定向的视图。 

```python
# This function creates the signin URL that the app will
# direct the user to in order to sign in to Office 365 and
# give the app consent.
def get_signin_url(redirect_uri):
  # Build the query parameters for the signin URL.
  params = { 'client_id': client_id,
             'redirect_uri': redirect_uri,
             'response_type': 'code'
           }

  authorize_url = '{0}{1}'.format(authority, '/common/oauth2/authorize?{0}')
  signin_url = authorize_url.format(urlencode(params))
  return signin_url
```

<!--<a name="authCode"></a>-->
## <a name="receive-an-authorization-code-in-your-reply-url-page"></a>在回复 URL 页面中接收授权代码

用户登录后，浏览器将重定向到你的回复 URL，[*connect/views.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/views.py) 中的 ```get_token``` 函数和授权代码附加到查询字符串作为 ```code``` 变量。 

Connect 示例从查询字符串获取代码，以便可以用其交换访问令牌。

```python
auth_code = request.GET['code']
```

<!--<a name="accessToken"></a>-->
## <a name="request-an-access-token-from-the-token-issuing-endpoint"></a>从令牌颁发终结点请求获取访问令牌

获取授权代码之后，你可以与从 Azure Active Directory 获取的客户端 ID、密钥以及回复 URL 值一起使用，以请求访问令牌。 

> **注意** 请求还必须指定你尝试要使用的资源。在 Microsoft Graph 情况中，资源值为 `https://graph.microsoft.com`。

Connect 示例请求 [*connect/auth_helper.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/auth_helper.py) 文件中的 ```get_token_from_code``` 函数中的令牌。

```python
# This function passes the authorization code to the token
# issuing endpoint, gets the token, and then returns it.
def get_token_from_code(auth_code, redirect_uri):
  # Build the post form for the token request
  post_data = { 'grant_type': 'authorization_code',
                'code': auth_code,
                'redirect_uri': redirect_uri,
                'client_id': client_id,
                'client_secret': client_secret,
                'resource': 'https://graph.microsoft.com'
              }
              
  r = requests.post(token_url, data = post_data)
  
  try:
    return r.json()
  except:
    return 'Error retrieving token: {0} - {1}'.format(r.status_code, r.text)
```

> **注意** 响应提供了更多信息，不仅仅是访问令牌。例如，你的应用将获取一个刷新令牌来请求新的访问令牌，无需用户再次显式登录。

<!--<a name="request"></a>-->
## <a name="use-the-access-token-in-a-request-to-the-microsoft-graph-api"></a>在对 Microsoft Graph API 提出的请求中使用访问令牌

使用访问令牌，您的应用可以对 Microsoft Graph API 提出身份验证请求。您的应用必须将访问令牌附加到各个请求的**授权**头中。

Connect 示例使用 Microsoft Graph API 中的 ```me/microsoft.graph.sendMail``` 终结点发送电子邮件。代码位于 [*connect/graph_service.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/graph_service.py) 文件的 ```call_sendMail_endpoint``` 函数中。这是显示如何将访问代码附加到授权标头中的代码。

```python
# Set request headers.
headers = { 
  'User-Agent' : 'python_tutorial/1.0',
  'Authorization' : 'Bearer {0}'.format(access_token),
  'Accept' : 'application/json',
  'Content-Type' : 'application/json'
}
```

> **注意** 该请求还必须发送包含 Graph API 接受的值的 **Content-Type** 标头，例如 `application/json`。

Microsoft Graph API 的功能非常强大，统一了可用于与各种 Microsoft 数据进行交互的 API。请查看 API 参考，了解还可以使用 Microsoft Graph 完成什么任务。