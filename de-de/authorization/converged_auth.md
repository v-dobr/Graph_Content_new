# <a name="authenticate-microsoft-graph-apps-with-the-azure-ad-v2.0-endpoint"></a>Authentifizieren von Microsoft Graph-Apps mit dem Azure AD v2.0-Endpunkt

> **Sie erstellen Apps für Unternehmenskunden?** Ihre App funktioniert möglicherweise nicht, wenn Ihr Unternehmenskunde Enterprise Mobility-Sicherheitsfunktionen wie <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-conditional-access-device-policies/" target="_newtab">bedingten Gerätezugriff</a> aktiviert.  

> Zur Unterstützung **aller Unternehmenskunden** über **alle Unternehmensszenarien** hinweg müssen Sie den Azure AD-Endpunkt verwenden und Ihr Apps mithilfe des [Azure-Verwaltungsportals](https://aka.ms/aadapplist) verwalten. Weitere Informationen finden Sie unter [Entscheiden zwischen dem Azure AD- und dem Azure AD v2.0-Endpunkt](auth_overview.md#deciding-between-azure-ad-and-the-v2-authentication-endpoint).


Mit dem Azure AD v2.0-Endpunkt können Sie Apps erstellen, für die sowohl Geschäfts- und Schulkonten- (Azure AD) als auch persönliche Identitäten (Microsoft-Konto) verwendet werden können.

In der Vergangenheit mussten App-Entwickler, wenn sie Unterstützung für Microsoft-Konten und Azure Active Directory integrieren wollten, zwei separate Systeme integrieren. Wenn Sie den Azure AD v2.0-Endpunkt verwenden, können Sie nun beide Kontotypen mit einer einzigen Integration unterstützen – ein einfacher Prozess, um ein Publikum zu erreichen, das Millionen von Benutzern mit privaten und Geschäfts-/Schulkonten umfasst.  

Nachdem Sie Ihre Apps mit dem Azure AD v2.0-Endpunkt integriert haben, können sie sofort auf die Microsoft Graph-Endpunkte für private und Geschäfts-/Schulkonten zugreifen, zum Beispiel: 

| Daten              | Endpunkt                                       |
|:------------------|:-----------------------------------------------|
| Benutzerprofil      | `https://graph.microsoft.com/v1.0/me`          |
| Outlook-Mail      | `https://graph.microsoft.com/v1.0/me/messages` |
| Outlook-Kontakte  | `https://graph.microsoft.com/v1.0/me/contacts` |
| Outlook-Kalender | `https://graph.microsoft.com/v1.0/me/events`   |
| OneDrive          | `https://graph.microsoft.com/v1.0/me/drive`    |

 >**Hinweis:** Einige Microsoft Graph-Endpunkte, wie z. B. Gruppen und Aufgaben, gelten nicht für private Konten.  

## <a name="microsoft-graph-authentication-scopes"></a>Microsoft Graph-Authentifizierungsbereiche

Der Azure AD v2.0-Enpunkt unterstützt alle Berechtigungsbereiche, die unter [Microsoft Graph-Berechtigungsbereiche](permission_scopes.md) aufgeführt sind. 

Weitere Informationen zur Verwendung von Bereichen mit dem Azure AD v2.0-Authentifizierungsendpunkt und zu Unterschieden zur Verwendung von Ressourcen in Azure AD finden Sie unter <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/#scopes-not-resources" target="_newtab">Bereiche, keine Ressourcen</a>.

## <a name="see-it-in-action"></a>In Aktion erleben

Der Artikel [Verbinden von Beispielen im Microsoft Graph-Repository](https://github.com/microsoftgraph?utf8=%E2%9C%93&query=connect) enthält einfache Beispiele zur Authentifizierung von Benutzern und zur Verbindung mit Microsoft Graph über eine breite Palette an Plattformen hinweg.

Darüber hinaus enthält der Abschnitt [Erste Schritte](http://graph.microsoft.io/en-us/docs/platform/get-started) Artikel, in denen die Erstellung dieser Beispie-Apps beschrieben wird, darunter auch die Authentifizierungsbibliotheken, die auf jeder Plattform verwendet werden.

## <a name="see-also"></a>Siehe auch

- [Registrieren einer App mit dem Azure AD v2.0-Endpunkt](auth_register_app_v2.md)
- [App-Authentifizierung mit Microsoft Graph](auth_overview.md)
- <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare" target="_newtab">Neuerungen des Azure AD v2.0-Modells</a>
- <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/" target="_newtab">Sollte ich einen Azure AD v2.0-Endpunkt verwenden?</a>
- <a href="https://azure.microsoft.com/en-us/documentation/articles/?product=active-directory&term=azure+ad+v2.0" target="_newtab">Azure AD v2.0-Endpunktdokumentation auf Azure.com</a>
- <a href="https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-app-registration/#build-a-quick-start-app" target="_newtab">Azure AD v2.0-Code-Schnellstarts auf Azure.com</a>

