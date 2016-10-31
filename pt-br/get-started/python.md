# <a name="get-started-with-microsoft-graph-in-a-python-app"></a>Introdução ao Microsoft Graph em um aplicativo Python 

Este artigo descreve as tarefas obrigatórias para obter um token de acesso do Azure AD e chamar o Microsoft Graph. Ele orientará você em relação ao [Exemplo de conexão com Microsoft Graph para Python](https://github.com/microsoftgraph/python3-connect-rest-sample) e explica os principais conceitos que você implementa para utilizar a API do Microsoft Graph. O artigo descreve como acessar o Microsoft Graph usando chamadas diretas REST.

![Captura de tela do exemplo de conexão com o Office 365 para Python](./images/web-screenshot.png)

> **Observação** Esse passo a passo e o exemplo são baseados no uso do ponto de extremidade do Azure AD. Verifique regularmente se há versões atualizadas que usam o ponto de extremidade do Azure AD v2.0.

##  <a name="prerequisites"></a>Pré-requisitos

  * Uma conta do Office 365 para empresas. Inscreva-se em uma [Assinatura do Office 365 para Desenvolvedor](https://msdn.microsoft.com/en-us/office/office365/howto/setup-development-environment#bk_Office365Account) que inclua os recursos necessários para começar a criar aplicativos do Office 365.
  * O [Exemplo de conexão com o Microsoft Graph para Python](https://github.com/microsoftgraph/python3-connect-rest-sample)

## <a name="register-the-application-in-azure-active-directory"></a>Registrar o aplicativo no Azure Active Directory

Primeiro, você precisa registrar seu aplicativo e definir permissões para usar o Microsoft Graph. Isso permite que os usuários entrem no aplicativo com contas corporativas ou de estudante.

1. Entre no [Portal do Azure](https://portal.azure.com/).
2. Na barra superior, clique em sua conta; em seguida, na lista **Directory**, escolha o locatário do Azure Active Directory em que deseja registrar o aplicativo.
3. Clique em **Mais Serviços**, na barra de navegação à esquerda, e escolha **Azure Active Directory**.
4. Clique em **Registros de aplicativos** e escolha **Adicionar**.
5. Insira um nome amigável para o aplicativo, por exemplo 'MSGraphConnectPython' e selecione 'App Web/API' como o **Tipo de Aplicativo**. Para a URL de logon, insira ‘http://127.0.0.1:8000/connect/get_token/’. Clique em **Criar** para criar o aplicativo.
6. Ainda no Portal do Azure, escolha o aplicativo, clique em **Configurações** e escolha **Propriedades**.
7. Localize o valor da ID do Aplicativo e copie-o para a Área de Transferência.
8. Configure permissões para o seu aplicativo:
9. No menu **Configurações**, escolha a seção **Permissões obrigatórias**, clique em **Adicionar**, em **Selecionar uma API** e selecione **Microsoft Graph**.
10. Em seguida, clique em Selecionar Permissões e selecione **Entrar e ler o perfil de usuário** e **Enviar email como um usuário**. Clique em **Selecionar** e depois **Concluído**.
11. No menu **Configurações**, escolha a seção **Chaves**. Insira uma descrição e selecione uma duração para a chave. Clique em **Salvar**.
12. **Importante:** Copie o valor da chave. Você não conseguirá acessar esse valor novamente depois de sair desse painel. Você usará esse valor como o segredo do aplicativo.

## <a name="redirect-the-browser-to-the-sign-in-page"></a>Redirecionar o navegador para a página de entrada

O aplicativo precisa redirecionar o navegador para a página de entrada a fim de iniciar o fluxo OAuth e obter um código de autorização. 

No exemplo do Connect, o código a seguir (localizado em [*connect/auth_helper.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/auth_helper.py)) cria a URL que o aplicativo precisa para redirecionar o usuário e é direcionado ao modo de exibição em ele pode ser usado para o redirecionamento. 

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
## <a name="receive-an-authorization-code-in-your-reply-url-page"></a>Receber um código de autorização em sua página de URL de resposta

Depois que o usuário entra, o navegador é redirecionado para a URL de resposta, a função ```get_token``` em [*connect/views.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/views.py), com um código de autorização anexado à cadeia de caracteres de consulta como a variável ```code```. 

O exemplo do Connect obtém o código da cadeia de caracteres de consulta para poder trocá-lo por um token de acesso.

```python
auth_code = request.GET['code']
```

<!--<a name="accessToken"></a>-->
## <a name="request-an-access-token-from-the-token-issuing-endpoint"></a>Solicitar um token de acesso do ponto de extremidade de emissão do token

Quando tiver o código de autorização, você poderá usá-lo com os valores ID do cliente, chave e URL de resposta obtidos do Azure Active Directory para solicitar um token de acesso. 

> **Observação** A solicitação também tem que especificar um recurso que você esteja tentando consumir. No caso do Microsoft Graph, o valor do recurso é `https://graph.microsoft.com`.

O exemplo do Connect solicita um token na função ```get_token_from_code``` no arquivo [*connect/auth_helper.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/auth_helper.py).

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

> **Observação** A resposta fornece mais informações além do token de acesso. Por exemplo, o aplicativo pode obter um token de atualização para solicitar novos tokens de acesso sem ter que conectar o usuário novamente.

<!--<a name="request"></a>-->
## <a name="use-the-access-token-in-a-request-to-the-microsoft-graph-api"></a>Usar o token de acesso em uma solicitação para a API do Microsoft Graph

Com um token de acesso, o aplicativo pode fazer solicitações autenticadas à API do Microsoft Graph. Seu aplicativo deve anexar o token de acesso ao cabeçalho **Authorization** de cada solicitação.

O exemplo do Connect envia um email usando o ponto de extremidade ```me/microsoft.graph.sendMail``` na API do Microsoft Graph. O código está na função ```call_sendMail_endpoint``` no arquivo [*connect/graph_service.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/graph_service.py). Esse é o código que mostra como anexar o código de acesso ao cabeçalho Authorization.

```python
# Set request headers.
headers = { 
  'User-Agent' : 'python_tutorial/1.0',
  'Authorization' : 'Bearer {0}'.format(access_token),
  'Accept' : 'application/json',
  'Content-Type' : 'application/json'
}
```

> **Observação** A solicitação também deve enviar um cabeçalho **Content-Type** com um valor aceito pela API do Microsoft Graph, por exemplo, `application/json`.

A API do Microsoft Graph é uma API unificadora muito poderosa que pode ser usada para interagir com todos os tipos de dados da Microsoft. Confira a referência de API para explorar o que mais você pode fazer com o Microsoft Graph.