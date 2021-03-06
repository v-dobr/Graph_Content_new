# <a name="microsoft-graph-permission-scopes"></a>Области разрешений Microsoft Graph

Microsoft Graph предоставляет разрешения OAuth 2.0, которые используются для управления доступом приложения к ресурсам. Разработчик может задать области разрешений, обеспечивающие необходимый уровень доступа для приложения. (При использовании проверки подлинности Azure AD это обычно делают на портале управления Azure. Если же используется конечная точка Azure AD версии 2.0, разрешения запрашиваются динамически во время работы приложения.)

После входа пользователям или администраторам дается возможность разрешить приложению доступ к своим ресурсам с настроенными разрешениям. По этой причине следует выбирать области разрешений, обеспечивающие минимальный уровень привилегий, необходимый приложению. Дополнительные сведения о настройке разрешений для приложения и процессе согласия см. в статье <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-integrating-applications/" target="_newtab">Интеграция приложений с Azure Active Directory</a>.

>**Примечание.** Некоторые разрешения Microsoft Graph, например разрешения для групп и задач, не применимы к личным учетным записям.  

##<a name="app-only-vs.-delegated-scopes"></a>Разрешения только для приложений и делегированные разрешения
Есть два типа разрешений: только для приложений и делегированные. Имея разрешение только для приложений (т. н. роль приложений), приложение имеет полный набор прав, предусмотренных этим разрешением. Разрешения только для приложений обычно используются приложениями, которые работают как служба без входа пользователей. 


Делегированные разрешения предназначены для приложений, действующих от имени пользователя. Приложение получает права вошедшего пользователя и может действовать от его имени. Фактические права, предоставляемые приложению, определяются пересечением прав, предусмотренных разрешением, и прав вошедшего пользователя. Например, если разрешение предусматривает делегированные права на запись всех объектов каталога, но вошедший пользователь может только обновлять собственный профиль, приложение сможет записывать только профиль этого пользователя.

## <a name="full-and-basic-profiles-for-users-and-groups"></a>Полный и базовый профили пользователей и групп

Полный профиль (или просто профиль) пользователя или группы включает все объявленные свойства объекта. Так как профиль может содержать конфиденциальную информацию о каталоге или личные сведения, доступ приложений к определенному набору свойств, который называется базовым профилем, ограничен. Базовый профиль пользователя включает только следующие свойства: 

- Отображаемое имя
- Имя и фамилия
- Фотография
- Адрес электронной почты

Базовый профиль группы содержит только отображаемое имя. 

<!---   <a name="msg_perm_details"> </a>  -->

## <a name="permission-scope-details"></a>Сведения о разрешениях

Чтобы получить необходимые разрешения на доступ к ресурсам Microsoft Graph, нужно настроить приложение. Разрешения распространяются на отдельные ресурсы и представляют собой права на чтение, запись или обе эти операции. 

В приведенных ниже таблицах перечислены разрешения Microsoft Graph, а также описываются предусмотренные ими права доступа. 

- В столбце **Область** указано название области разрешений. Названия разрешений представляются в формате "ресурс.операция.ограничение" (например, Group.ReadWrite.All). Если задано ограничение All, приложение может выполнять операцию (ReadWrite) со всеми указанными ресурсами (Group) в каталоге. В противном случае приложение может выполнять эту операцию только в профиле вошедшего пользователя. Разрешения могут предоставлять ограниченные привилегии для определенной операции. Подробные сведения представлены в столбце **Описание**.
- В столбце **Разрешение** показано, как разрешение отображается на портале Azure. 
- В столбце **Описание** представлено описание всех привилегий, предоставляемых разрешением. В случае делегированных разрешений фактические права доступа, предоставляемые приложению, определяются пересечением прав, предусмотренных разрешением, и привилегий вошедшего пользователя. 
- Разрешения группируются в соответствии с тем, требуется ли для них согласие администратора.

  > **Примечание**. Сведения об ограничениях для разрешений см. в разделе [Известные проблемы](../overview/release_notes) для `v1.0` и `beta`.
  
###<a name="permissions-requiring-administrator's-consent"></a>Разрешения, требующие согласия администратора

|   **Область**                  |  **Разрешение**                          |  **Описание** |
|:-----------------------------|:-----------------------------------------|:-----------------|
| _User.Read.All_                |     Чтение полных профилей всех пользователей           | Не отличается от разрешения User.ReadBasic.All за исключением того, что приложение сможет просматривать полные профили всех пользователей в организации, а также свойства иерархической организации, например руководителя и подчиненных. Полный профиль включает все объявленные свойства объекта **User**. Чтобы просматривать группы, в которых состоит пользователь, приложению также потребуется разрешение Group.Read.All или Group.ReadWrite.All. |
| _User.ReadWrite.All_           |     Чтение и запись полных профилей всех пользователей | Приложение сможет просматривать и записывать полный набор свойств профиля, подчиненных и руководителей других пользователей в организации от имени вошедшего пользователя. |
| _Directory.Read.All_           |     Чтение данных каталога                     | Позволяет приложению читать данные в каталоге вашей организации, такие как группы пользователей и приложения. |
| _Directory.ReadWrite.All_      |     Чтение и запись данных каталога           | Приложение сможет просматривать и записывать в каталоге организации данные, например сведения о пользователях и группах.  Не разрешается удалять пользователей или группы. Приложение не сможет удалять пользователей или группы, а также сбрасывать пароли пользователей. |
| _Directory.AccessAsUser.All_   |     Доступ к каталогу, аналогичный доступу вошедшего пользователя  | Приложение получит те же права доступа к данным в каталоге, что и вошедший пользователь.|
| _Group.Read.All_ |    Чтение всех групп | Приложение сможет выводить список групп, а также просматривать их свойства и все данные о членстве в группах от имени вошедшего пользователя.  Оно также сможет просматривать календарь, беседы, файлы и другое содержимое всех групп, к которым у вошедшего пользователя есть доступ. |
| _Group.ReadWrite.All_ |    Чтение и запись всех групп| Приложение сможет создавать группы, а также просматривать все их свойства и данные о членстве от имени вошедшего пользователя.  Кроме того, владельцы групп смогут управлять своими группами, а члены групп — обновлять содержимое групп. |


###<a name="permissions-not-requiring-administrator's-consent"></a>Разрешения, не требующие согласия администратора

|   **Область**    |  **Разрешение**   |  **Описание** |
|:-----------------------------|:-----------------------------------------|:-----------------|
| _User.Read_       |    Вход и чтение профиля пользователя | Пользователи смогут входить в приложение, а оно сможет просматривать профили вошедших пользователей. Полный профиль включает все объявленные свойства объекта User. Приложение не сможет просматривать свойства иерархической организации, например руководителя или подчиненных. Приложение также сможет просматривать следующие базовые сведения о компании вошедшего пользователя (через объект **TenantDetail**): идентификатор клиента, отображаемое имя клиента и проверенные домены.|
| _User.ReadWrite_ |    Доступ для чтения и записи к профилю пользователя | Приложение сможет просматривать ваш профиль. Кроме того, оно сможет обновлять данные вашего профиля от вашего имени. |
| _User.ReadBasic.All_ |    Чтение базовых профилей всех пользователей | Приложение сможет просматривать базовые профили всех пользователей в организации от имени вошедшего пользователя. Базовый профиль пользователя включает следующие свойства: отображаемое имя, имя и фамилия, фотография и адрес электронной почты. Чтобы просматривать группы, в которых состоит пользователь, приложению также потребуется разрешение Group.Read.All или Group.ReadWrite.All.| 
| _Mail.Read_ |    Чтение почты пользователя | Позволяет приложению читать электронную почту в почтовых ящиках пользователя. |
| _Mail.ReadWrite_ |    Доступ для чтения и записи к почте пользователя | Приложение сможет создавать, просматривать, обновлять и удалять сообщения в почтовых ящиках пользователей. Не включает разрешение на отправку почты.|
| _Mail.Send_ |    Отправка почты от имени пользователя | Позволяет приложению отправлять почту от имени пользователей в организации. |
| _Calendars.Read_ |    Чтение пользовательских календарей  | Позволяет приложению читать события в пользовательских календарях.|
| _Calendars.ReadWrite_ |    Полный доступ к пользовательским календарям  | Позволяет приложению создавать, читать, обновлять и удалять события в пользовательских календарях. |
| _Contacts.Read_ |    Чтение контактов пользователя  | Позволяет приложению читать контакты пользователя. |
| _Contacts.ReadWrite_ |    Полный доступ к контактам пользователя  | Позволяет приложению создавать, читать, обновлять и удалять контакты пользователя. |
| _Files.Read_ |    Чтение файлов пользователя, а также файлов, которыми с ним поделились | Позволяет приложению читать файлы вошедшего пользователя, а также файлы, которыми с ним поделились.| 
| _Files.ReadWrite_ |   Полный доступ к файлам пользователя и к файлам, которыми с ним поделились | Позволяет приложению читать, создавать, обновлять и удалять файлы вошедшего пользователя, а также файлы, которыми с ним поделились. |
| _Files.ReadWrite.Selected_ |    Чтение и запись файлов, выбранных пользователем | Приложение сможет просматривать и записывать файлы, выбранные пользователем. У приложения будет доступ в течение нескольких часов после выбора файла пользователем. |
| _Files.Read.Selected_ |    Чтение файлов, которые выбирает пользователь  | Приложение сможет просматривать файлы, выбранные пользователем. У приложения будет доступ в течение нескольких часов после выбора файла пользователем. |
| _Sites.Read.All_ |    Чтение элементов во всех семействах веб-сайтов | Позволяет приложению читать документы и элементы списка во всех семействах веб-сайтов от имени вошедшего пользователя. |
| _openid_ |    Вход пользователей (предварительная версия) | Пользователи смогут входить в приложение с помощью своей рабочей или учебной учетной записи, а приложение сможет просматривать основные данные профилей пользователей.|
| _offline_access_ |    Доступ к данным пользователя в любое время (предварительная версия) | Позволяет приложению читать и обновлять данные пользователя, даже если он в настоящее время не использует приложение.|

###<a name="app-only-permissions-requiring-administrator's-consent"></a>Разрешения только для приложений, требующие согласия администратора

|   **Область**    |  **Разрешение**   |  **Описание** |
|:---------------|:------------------|:-----------------|
| _Mail.Read_       |    Чтение почты во всех почтовых ящиках | Позволяет приложению читать почту во всех почтовых ящиках без вошедшего пользователя.|
| _Mail.ReadWrite_ |    Чтение и запись почты во всех почтовых ящиках | Приложение сможет создавать, просматривать, обновлять и удалять сообщения во всех почтовых ящиках без входа пользователя. Не включает разрешение на отправку почты. |
| _Mail.Send_ |    Отправка почты от имени любого пользователя | Позволяет приложению отправлять почту от имени любого пользователя без вошедшего пользователя. | 
| _Calendars.Read_ |    Чтение календарей во всех почтовых ящиках | Позволяет приложению читать события во всех календарях без вошедшего пользователя. |
| _Calendars.ReadWrite_ |    Чтение и запись календарей во всех почтовых ящиках | Позволяет приложению создавать, читать, обновлять и удалять события во всех календарях без вошедшего пользователя.|
| _Contacts.Read_ |    Чтение контактов во всех почтовых ящиках | Позволяет приложению читать все контакты во всех почтовых ящиках без вошедшего пользователя. |
| _Contacts.ReadWrite_ |    Чтение и запись контактов во всех почтовых ящиках  |Позволяет приложению создавать, читать, обновлять и удалять все контакты во всех почтовых ящиках без вошедшего пользователя.|
| _User.ReadBasic.All_ |    Чтение базовых профилей всех пользователей  | Приложение сможет просматривать базовый набор свойств профилей других пользователей в организации без входа пользователя. Включает имя и фамилию, отображаемое имя, фотографию и сообщение об отсутствии на рабочем месте.|
| _User.Read.All_ |    Чтение полных профилей всех пользователей | Приложение сможет просматривать полный набор свойств профиля, данные о членстве в группах, сведения о подчиненных и руководителях других пользователей в организации без входа пользователя.| 
| _User.ReadWrite.All_ |   Чтение и запись полных профилей всех пользователей | Приложение сможет просматривать и записывать полный набор свойств профиля, данные о членстве в группах, сведения о подчиненных и руководителях других пользователей в организации без входа пользователя.|


##<a name="permission-scopes-in-preview"></a>Разрешения в предварительной версии
###<a name="permissions-not-requiring-administrator's-consent-(preview)"></a>Разрешения, не требующие согласия администратора (предварительная версия)

|   **Область**    |  **Разрешение**   |  **Описание** |
|:---------------|:------------------|:-----------------|
| _Tasks.ReadWrite_ |    Создание, чтение, обновление и удаление задач и проектов пользователя (предварительная версия) | Приложение сможет создавать, просматривать, обновлять и удалять задачи и планы (а также задачи в них), назначенные или предоставленные вошедшему пользователю.|
| _People.Read_ |    Чтение списков людей, релевантных для пользователя (предварительная версия) | Приложение сможет просматривать ранжированный список контактов вошедшего пользователя. Список включает локальные контакты, контакты из социальных сетей и каталога вашей организации, а также пользователей, с которыми он недавно общался (например, с помощью электронной почты и Skype).|
| _People.ReadWrite_ |    Чтение и запись списков людей, релевантных для пользователя (предварительная версия) | Приложение сможет создавать, просматривать и записывать ранжированный список контактов вошедшего пользователя. Список включает локальные контакты, контакты из социальных сетей и каталога вашей организации, а также пользователей, с которыми он недавно общался (например, с помощью электронной почты и Skype).|
| _Notes.Create_ |    Создание страниц в записных книжках пользователей (предварительная версия) | Приложение сможет просматривать названия записных книжек и разделов, а также создавать страницы, записные книжки и разделы от имени вошедшего пользователя.|
| _Notes.ReadWrite.CreatedByApp_ |    Ограниченный доступ к записным книжкам (предварительная версия) | Приложение сможет просматривать названия записных книжек и разделов, а также создавать страницы от имени вошедшего пользователя. Кроме того, приложение сможет просматривать и обновлять созданные им страницы. |
| _Notes.Read_ |    Чтение записных книжек пользователя (предварительная версия) | Приложение сможет просматривать названия записных книжек и разделов OneNote, а также просматривать все страницы от имени вошедшего пользователя. Оно не сможет просматривать разделы, защищенные паролем. |
| _Notes.ReadWrite_ |    Чтение и запись записных книжек пользователя (предварительная версия) | Приложение сможет просматривать названия записных книжек и разделов, все страницы, а также записывать и создавать страницы от имени вошедшего пользователя.  Оно не получит доступ к разделам, защищенным паролем. |
| _Notes.Read.All_ |    Чтение всех записных книжек, к которым есть доступ у пользователя (предварительная версия) | Приложение сможет просматривать содержимое всех записных книжек и разделов, к которым есть доступ у вошедшего пользователя.   Оно не сможет просматривать разделы, защищенные паролем. |
| _Notes.ReadWrite.All_ |    Чтение и запись всех записных книжек, к которым есть доступ у пользователя (предварительная версия) | Приложение сможет просматривать и записывать содержимое всех записных книжек и разделов, доступных вошедшему пользователю.  Оно не получит доступ к разделам, защищенным паролем.|

##<a name="permission-scope-scenarios"></a>Сценарии использования разрешений
Ниже приведено несколько сценариев с использованием ресурсов `User` и `Group`, а также соответствующие необходимые разрешения. В таблице ниже приведены разрешения, необходимые приложению для выполнения определенных операций. Обратите внимание, что в некоторых случаях способность приложения выполнять некоторые операции зависит от типа разрешения (только для приложений или делегированное), а в случае делегированных разрешений — от прав вошедшего пользователя. 

###<a name="access-scenarios-using-the-user-resource-and-the-required-scopes"></a>Сценарии доступа с использованием ресурса пользователя и необходимые разрешения

| **Задачи приложения, связанные с пользователем**   |  **Необходимые области** | **Разрешения** |
|:-------------------------------|:---------------------|:---------------|
| Приложение запрашивает разрешение просматривать основные сведения других пользователей (только отображаемое имя и изображение), например для отображения при выборе людей.   | _User.ReadBasic.All_  |  Чтение базовых профилей всех пользователей |
| Приложение запрашивает разрешение просматривать полный профиль вошедшего пользователя (подчиненные, руководитель и т. д).  | _User.Read_ | Вход в систему и чтение профиля пользователя|
| Приложение запрашивает разрешение просматривать полные профили всех пользователей.  | _User.Read.All_ |  Чтение полных профилей всех пользователей   |
| Приложение запрашивает разрешение просматривать файлы, почту и данные календаря вошедшего пользователя.  | _User.Read_, _Files.Read_, _Mail.Read_, _Calendar.Read_ | Вход в систему и чтение профиля пользователя, чтение файлов пользователей, чтение почты пользователей, чтение пользовательских календарей |
| Приложение запрашивает разрешение просматривать файлы вошедшего пользователя и файлы, доступ к которым ему предоставили другие пользователи. | _User.Read_, _Files.Read_, _Sites.Read.All_ | Вход в систему и чтение профиля пользователя, чтение файлов пользователей, чтение элементов во всех семействах веб-сайтов |
| Приложение запрашивает разрешение просматривать и записывать полный профиль вошедшего пользователя.   | _User.ReadWrite_ | Доступ для чтения и записи к профилю пользователя |
| Приложение запрашивает разрешение просматривать и записывать полные профили всех пользователей.    | _User.ReadWrite.All_ | Чтение и запись полных профилей всех пользователей |
| Приложение запрашивает разрешение просматривать и записывать файлы, почту и данные календаря вошедшего пользователя.    | _User.ReadWrite_, _Files.ReadWrite_, _Mail.ReadWrite_, _Calendar.ReadWrite_  |  Доступ для чтения и записи к профилю пользователя, доступ для чтения и записи к почте пользователя, полный доступ к пользовательским календарям |
   

###<a name="access-scenarios-using-the-group-resource-and-the-required-scopes"></a>Сценарии доступа с использованием ресурса группы и необходимые разрешения
    
| **Задачи приложения, связанные с группой**  |  **Необходимые области** |  **Разрешения** |
|:-------------------------------|:---------------------|:---------------|
| Приложение запрашивает разрешение просматривать основные сведения о группе (только отображаемое имя и изображение), например для отображения при выборе групп.  | _Group.Read.All_  | Чтение всех групп|
| Приложение запрашивает разрешение просматривать все содержимое во всех единых группах, в том числе файлы и беседы.  Ему также требуется показывать членов группы и обновлять эти данные (если пользователь — владелец).  |  _Group.Read.All_ | Чтение элементов во всех семействах веб-сайтов, чтение всех групп|
| Приложение запрашивает разрешение просматривать и записывать все содержимое во всех единых группах, в том числе файлы и беседы.  Ему также требуется показывать членов группы и обновлять эти данные (если пользователь — владелец).  |  _Group.ReadWrite.All_, _Sites.ReadWrite.All_ |  Чтение и запись всех групп, редактирование и удаление элементов во всех семействах веб-сайтов |
| Приложение запрашивает разрешение на поиск единой группы. Пользователь сможет найти определенную группу, выбрать ее из нумерованного списка, чтобы затем присоединиться к ней.     | _Group.ReadWrite.All_ | Чтение и запись всех групп|
| Приложение запрашивает разрешение на создание группы с помощью AAD Graph. |   _Group.ReadWrite.All_ | Чтение и запись всех групп|
 


