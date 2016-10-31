# <a name="use-rest-apis-to-access-mailboxes-in-exchange-hybrid-deployments-(preview)"></a>Verwenden von REST-APIs für den Zugriff auf Postfächer in Exchange-Hybridbereitstellungen (Vorschau)

Microsoft Graph unterstützte stets den Zugriff auf Postfächer von Kunden in der Cloud in Exchange Online als Teil von Office 365. Das kumulative Update 3 für Exchange 2016 (CU3) wurde im September 2016 für lokale Exchange-Server veröffentlicht und umfasst Unterstützung für die REST-API-Integration in Office 365. Wenn Ihre App v1.0 der [Mail](http://graph.microsoft.io/en-us/docs/api-reference/v1.0/resources/message)-, [Kalender](http://graph.microsoft.io/en-us/docs/api-reference/v1.0/resources/calendar)- oder [Kontakte](http://graph.microsoft.io/en-us/docs/api-reference/v1.0/resources/contact)-API verwendet, finden Sie nun eine nahtlose Authentifizierungs- und Anwendungserfahrung in _Hybrid_bereitstellungen, unabhängig davon, ob die Postfächer lokal oder in der Cloud gespeichert sind, vorausgesetzt, die Bereitstellung erfüllt bestimmte [Anforderungen](#requirements-for-rest-api-to-work-in-hybrid-deployments). 


Wenn Microsoft Graph erkennt, dass ein REST-API-Aufruf versucht, auf ein lokales Postfach in einer Hybridbereitstellung zuzugreifen, wird die REST-Anfrage an einen lokalen REST-Endpunkt übergeben, der die Anfrage verarbeitet. Durch diese Erkennung ist ein Zugriff auf die REST-API möglich.

>**Hinweis:** Die Funktion zur Verwendung dieseser REST-APIs in Hybridbereitstellungen befindet sich aktuell in der Vorschauphase.

>Für Postfächer in Hybridbereitstellungen ist nur v1.0 der Mail-, Kalender- und Kontakte-API verfügbar. Andere v1.0-API-Sätze, wie beispielsweise die [Gruppen](http://graph.microsoft.io/en-us/docs/api-reference/v1.0/resources/group)-API oder APIs in anderen Versionen sind nicht verfügbar. Wenn Sie versuchen, eine API zu verwenden, die nicht Teil in eines unterstützten Satzes in einer Hybridumgebung sind, erhalten Sie folgende Fehlermeldung:

>„REST-APIs für das Postfach sind derzeit in der Vorschau. Weitere Informationen zu den REST-APIs in der Vorschau finden Sie unter https://dev.outlook.com.“

## <a name="requirements-for-the-rest-api-to-work-in-hybrid-deployments"></a>Anforderungen für die REST-API für Hybridbereitstellungen

Microsoft Graph bietet Offenheit (Unterstützung offener Standards wie JSON, OAUTH und ODATA für die Herstellung von Verbindungen von den meisten gängigen Plattformen) und Flexibilität (detaillierte, eng gefasste Zugriffsberechtigungen für Benutzerdaten). Wenn Ihre Organisation Interesse an der Entwicklung von Microsoft Graph-Apps hat und derzeit eine Hybridbereitstellung durchführt oder in Betracht zieht, sind folgende Anforderungen zu beachten:

- Postfachanforderungen

  - Alle lokal gehosteten Postfächer, die die REST-APIs benutzen werden, müssen sich in Datenbanken auf Exchange 2016 CU3-Servern befinden. 

- Infrastrukturanforderungen

  - Alle Exchange 2016-Server müssen auf CU3 oder später aktualisiert werden.  
  - Das lokale Active Directory muss mit dem Azure Active Directory synchronisiert werden.
  - Exchange 2013-Server, die gemeinsam mit Exchange 2016-Servern in demselben Lastenausgleich-Array vorhanden sind, müssen aus dem Array entfernt werden.

- Netzwerkanforderungen

  - Aus DNS-Perspektive benötigen der AutoErmittlung-Namespace und der lokale Clientnamespace Internet-DNS-Datensätze. 
  - Wenn Sie eine Firewall oder ein Anwendungsgateway haben, die/das den Zugriff prüft und einschränkt, aktualisieren Sie die entsprechenden Einstellungen, um Erkennung und Zugriff zu erlauben.


IT-Administratoren finden weitere Informationen in den folgenden Ressourcen:

- 
  [Hybridbereitstellungen in Exchange Server](https://technet.microsoft.com/en-us/library/jj200581(v=exchg.150).aspx)
- [Kumulatives Update-Release September 2016](https://blogs.technet.microsoft.com/exchange/2016/09/20/released-september-2016-quarterly-exchange-updates/) 
- [Anforderungen an die lokale Architektur für die REST-API](https://blogs.technet.microsoft.com/exchange/2016/09/26/on-premises-architectural-requirements-for-the-rest-api/)
