---
title: Azure Event Grid – Handbuch zur Problembehandlung
description: Dieser Artikel bietet eine Liste mit Fehlercodes, Fehlermeldungen, Beschreibungen und empfohlenen Aktionen.
ms.topic: conceptual
ms.date: 04/09/2021
ms.openlocfilehash: 4647039f67cf84a5471dc7b3f38ca754ca9eecb0
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/13/2021
ms.locfileid: "107310868"
---
# <a name="troubleshoot-azure-event-grid-errors"></a>Problembehandlung von Azure Event Grid-Fehlern
In diesem Handbuch zur Problembehandlung finden Sie die folgenden Informationen: 

- Azure Event Grid-Fehlercodes
- Fehlermeldungen
- Beschreibungen der Fehler
- Empfohlene Maßnahmen, die Sie ausführen sollten, wenn Sie diese Fehlermeldungen erhalten. 

## <a name="error-code-400"></a>Fehlercode: 400
| Fehlercode | Fehlermeldung | BESCHREIBUNG | Empfehlung |
| ---------- | ------------- | ----------- | -------------- | 
| HttpStatusCode.BadRequest<br/>400 | Der Themenname muss zwischen 3 und 50 Zeichen lang sein. | Der benutzerdefinierte Themenname sollte zwischen 3 und 50 Zeichen lang sein. Themennamen dürfen nur alphanumerische Buchstaben, Ziffern und das Zeichen „-“ (Bindestrich) enthalten. Außerdem sollte der Name nicht mit den folgenden reservierten Wörtern beginnen: <ul><li>Microsoft-</li><li>EventGrid-</li><li>System-</li></ul> | Wählen Sie einen anderen Themennamen aus, der die Anforderungen an Themennamen erfüllt. |
| HttpStatusCode.BadRequest<br/>400 | Der Domänenname muss zwischen 3 und 50 Zeichen lang sein. | Die Länge des Domänennamens sollte zwischen 3 und 50 Zeichen betragen. Domänennamen dürfen nur alphanumerische Buchstaben, Ziffern und das Zeichen „-“ (Bindestrich) enthalten. Außerdem sollte der Name nicht mit den folgenden reservierten Wörtern beginnen:<ul><li>Microsoft-</li><li>EventGrid-</li><li>System-</li> | Wählen Sie einen anderen Domänennamen aus, der die Anforderungen an Domänennamen erfüllt. |
| HttpStatusCode.BadRequest<br/>400 | Ungültige Ablaufzeit. | Die Ablaufzeit für das Ereignisabonnement bestimmt, wann das Ereignisabonnement abläuft. Dieser Wert sollte in der Zukunft liegender, gültiger DateTime-Wert sein.| Stellen Sie sicher, dass die Ablaufzeit des Ereignisabonnements ein gültiges DateTime-Format hat und in der Zukunft liegt. |

## <a name="error-code-409"></a>Fehlercode: 409
| Fehlercode | Fehlermeldung | BESCHREIBUNG | Empfohlene Maßnahme |
| ---------- | ------------- | ----------- | -------------- | 
| HttpStatusCode.Conflict <br/>409 | Ein Thema mit dem angegebenen Namen ist bereits vorhanden. Wählen Sie einen anderen Themennamen aus.   | Der benutzerdefinierte Themenname sollte in einer einzelnen Azure-Region eindeutig sein, um einen korrekten Veröffentlichungsvorgang sicherzustellen. Derselbe Name kann in verschiedenen Azure-Regionen verwendet werden. | Wählen Sie einen anderen Namen für das Thema aus. |
| HttpStatusCode.Conflict <br/> 409 | Eine Domäne mit dem angegebenen Namen ist bereits vorhanden. Wählen Sie einen anderen Domänennamen aus. | Der Domänenname sollte in einer einzelnen Azure-Region eindeutig sein, um einen korrekten Veröffentlichungsvorgang sicherzustellen. Derselbe Name kann in verschiedenen Azure-Regionen verwendet werden. | Wählen Sie einen anderen Namen für die Domäne aus. |
| HttpStatusCode.Conflict<br/>409 | Kontingentlimit erreicht. Weitere Informationen zu diesen Limits finden Sie unter [Azure Event Grid-Grenzwerte](../azure-resource-manager/management/azure-subscription-service-limits.md#event-grid-limits).  | Für jedes Azure-Abonnement gilt ein Limit hinsichtlich der Anzahl von Azure Event Grid-Ressourcen, die es verwenden kann. Einige oder alle dieses Kontingente wurden überschritten, und es konnten keine Ressourcen mehr erstellt werden. | Überprüfen Sie Ihre aktuelle Ressourcennutzung, und löschen Sie alle nicht benötigten Ressourcen. Wenn Sie keine Ressourcen löschen können, erstellen Sie ein weiteres Azure-Abonnement, und erstellen Sie Event Grid-Ressourcen in diesem Abonnement. |

## <a name="error-code-403"></a>Fehlercode: 403

| Fehlercode | Fehlermeldung | BESCHREIBUNG | Empfohlene Maßnahme |
| ---------- | ------------- | ----------- | ------------------ |
| HttpStatusCode.Forbidden <br/>403 | Die Veröffentlichung in {Topic/Domain} durch Client-{IpAddress} wird aufgrund von IpAddress-Filterregeln abgelehnt. | Für das Thema oder die Domäne sind IP-Firewallregeln konfiguriert, und der Zugriff ist nur auf konfigurierte IP-Adressen beschränkt. | Fügen Sie die IP-Adresse den IP-Firewallregeln hinzu, siehe [Konfigurieren der IP-Firewall.](configure-firewall.md) |
| HttpStatusCode.Forbidden <br/> 403 | Die Veröffentlichung in {Topic/Domain} durch den Client wird abgelehnt, weil die Anforderung vom privaten Endpunkt stammt und keine passende private Endpunktverbindung für die Ressource gefunden wurde. | Das Thema oder die Domäne hat private Endpunkte, und die Veröffentlichungsanforderung stammte von einem privaten Endpunkt, der nicht konfiguriert oder genehmigt ist. | Konfigurieren Sie einen privaten Endpunkt für das Thema bzw. die Domäne. [Konfigurieren privater Endpunkte](configure-private-endpoints.md) |

Überprüfen Sie auch, ob sich Ihr Webhook hinter einem Azure Application Gateway oder einer Web Application Firewall befindet. Wenn dies der Fall ist, deaktivieren Sie die folgenden Firewallregeln, und führen Sie erneut einen HTTP POST-Aufruf aus:

- 920300 (Fehlender Accept-Header für Anforderung)
- 942430 (Eingeschränkte Anomalieerkennung für SQL-Zeichen (Argumente): Anzahl von Sonderzeichen überschritten (12))
- 920230 (Mehrere URL-Codierungen erkannt)
- 942130 (Angriff mit Einschleusung von SQL-Befehlen: SQL-Tautologie erkannt.)
- 931130 (Möglicher RFI-Angriff (Remote File Inclusion) = Domänenexterner Verweis/Link)



## <a name="next-steps"></a>Nächste Schritte
Wenn Sie weitere Hilfe benötigen, veröffentlichen Sie Ihr Problem im [Stack Overflow-Forum](https://stackoverflow.com/questions/tagged/azure-eventgrid), oder öffnen Sie ein [Supportticket](https://azure.microsoft.com/support/options/). 
