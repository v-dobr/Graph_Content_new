# <a name="get-started-with-microsoft-graph-in-a-python-app"></a>Начало работы с Microsoft Graph в приложении Python 

В этой статье описываются задачи, которые необходимо выполнить, чтобы получить маркер доступа из Azure AD и вызвать Microsoft Graph. В ней описывается создание [примера Microsoft Graph Connect для Python](https://github.com/microsoftgraph/python3-connect-rest-sample), и рассматриваются основные понятия, которые необходимо реализовать для использования API Microsoft Graph. В этой статье рассказывается, как получить доступ к Microsoft Graph с помощью прямых вызовов REST.

![Снимок экрана приложения Python Connect для Office 365](./images/web-screenshot.png)

> **Примечание.** В этом пошаговом руководстве и описываемом в нем примере используется конечная точка Azure AD. Вскоре будут добавлены обновленные версии, использующие конечную точку Azure AD версии 2.0.

##  <a name="prerequisites"></a>Необходимые компоненты

  * Учетная запись Office 365 для бизнеса. Вы можете [подписаться на план Office 365 для разработчиков](https://msdn.microsoft.com/en-us/office/office365/howto/setup-development-environment#bk_Office365Account), который включает ресурсы, необходимые для создания приложений Office 365.
  * [Пример Microsoft Graph Connect для Python](https://github.com/microsoftgraph/python3-connect-rest-sample)

## <a name="register-the-application-in-azure-active-directory"></a>Регистрация приложения в Azure Active Directory

Для начала необходимо зарегистрировать приложение и задать разрешения на использование Microsoft Graph. Это позволяет пользователям входить в приложение с помощью рабочих или учебных учетных записей.

1. Войдите на [портал Azure](https://portal.azure.com/).
2. Выберите свою учетную запись в верхней панели, а затем в списке **Каталог** выберите клиент Active Directory, в котором необходимо зарегистрировать приложение.
3. На панели навигации слева нажмите **Другие службы** и выберите **Azure Active Directory**.
4. Выберите элементы **Регистрация приложений** **Добавить**.
5. Введите понятное имя приложения, например MSGraphConnectPython, и выберите значение "Веб-приложение/API" в поле **Тип приложения**. В поле "URL-адрес входа" введите адрес http://127.0.0.1:8000/connect/get_token/. Нажмите кнопку **Создать**, чтобы завершить создание приложения.
6. Оставаясь на портале Azure, выберите свое приложение, нажмите **Параметры** и выберите **Свойства**.
7. Найдите значение "Идентификатор приложения" и скопируйте его в буфер обмена.
8. Настройте разрешения для приложения:
9. В меню **Параметры** выберите раздел **Требуемые разрешения**, нажмите **Добавить**, **Выберите API** и выберите **Microsoft Graph**.
10. Затем нажмите элемент "Выберите разрешения" и выберите **Вход и чтение профиля пользователя** и **Отправка почты от имени пользователя**. Нажмите кнопку **Выбрать**, а затем — **Готово**.
11. В меню **Параметры** выберите раздел **Ключи**. Введите описание и выберите длительность для ключа. Нажмите кнопку **Сохранить**.
12. **Важно!** Скопируйте значение ключа. Когда вы закроете панель, снова получить доступ к этому значению будет невозможно. Это значение будет использоваться как секрет приложения.

## <a name="redirect-the-browser-to-the-sign-in-page"></a>Перенаправление браузера на страницу входа

Приложение должно перенаправить браузер на страницу входа, чтобы начать поток OAuth и получить код авторизации. 

В примере Connect приведенный ниже код (расположенный в файле [*connect/auth_helper.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/auth_helper.py)) создает URL-адрес, на который приложение должно перенаправлять пользователя, и передается в окно, в котором его можно использовать для перенаправления. 

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
## <a name="receive-an-authorization-code-in-your-reply-url-page"></a>Получение кода авторизации на странице URL-адреса ответа

После входа пользователя браузер перенаправляется на URL-адрес ответа, функцию ```get_token``` в файле [*connect/views.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/views.py), и к строке запроса добавляется код авторизации в виде переменной ```code```. 

Приложение Connect получает код из строки запроса, чтобы затем обменять его на токен доступа.

```python
auth_code = request.GET['code']
```

<!--<a name="accessToken"></a>-->
## <a name="request-an-access-token-from-the-token-issuing-endpoint"></a>Запрос токена доступа из конечной точки выдачи токенов

Используя код авторизации и значения идентификатора клиента, ключа и URL-адреса ответа, полученные из Azure Active Directory, вы можете запросить токен доступа. 

> **Примечание**. В запросе также необходимо указать ресурс. Значение ресурса для Microsoft Graph — `https://graph.microsoft.com`.

Приложение Connect запрашивает токен в функции ```get_token_from_code``` в файле [*connect/auth_helper.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/auth_helper.py).

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

> **Примечание**. Ответ содержит не только токен доступа, но и другие сведения. Например, приложение может получить токен обновления для запроса новых токенов доступа без повторного входа пользователя.

<!--<a name="request"></a>-->
## <a name="use-the-access-token-in-a-request-to-the-microsoft-graph-api"></a>Использование токена доступа в запросе к API Microsoft Graph

Используя токен доступа, приложение может отправлять проверенные запросы в API Microsoft Graph. Приложение должно добавлять токен доступа в заголовок **Authorization** каждого запроса.

Приложение Connect отправляет электронную почту, используя конечную точку ```me/microsoft.graph.sendMail``` в API Microsoft Graph. Код содержится в функции ```call_sendMail_endpoint``` в файле [*connect/graph_service.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/graph_service.py). Этот код показывает, как добавить код доступа в заголовок "Authorization".

```python
# Set request headers.
headers = { 
  'User-Agent' : 'python_tutorial/1.0',
  'Authorization' : 'Bearer {0}'.format(access_token),
  'Accept' : 'application/json',
  'Content-Type' : 'application/json'
}
```

> **Примечание**. В запросе необходимо также отправлять заголовок **Content-Type** со значением, принимаемым API Graph, например `application/json`.

API Microsoft Graph — это мощный единый API, с помощью которого можно взаимодействовать со всеми типами данных корпорации Майкрософт. Сведения о других возможностях API Microsoft Graph см. в справочнике по API.