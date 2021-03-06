---
title: 'Azure Event Grid: Diagnoseprotokolle für Themen oder Domänen'
description: In diesem Artikel erhalten Sie konzeptionelle Informationen über Diagnoseprotokolle für ein Azure Event Grid-Thema oder eine -Domäne.
ms.topic: conceptual
ms.date: 12/03/2020
ms.openlocfilehash: 13577e02cba07fc788bb4846cddd3c0ed894f2b1
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/30/2021
ms.locfileid: "108314925"
---
#  <a name="diagnostic-logs-for-azure-event-grid-topicsdomains"></a>Diagnoseprotokolle für Azure Event Grid-Themen und -Domänen
Diagnoseeinstellungen ermöglichen es Event Grid-Benutzern, Protokolle zu **Veröffentlichungs- und Zustellungsfehlern** entweder in einem Speicherkonto, einer Event Hub-Instanz oder einem Log Analytics-Arbeitsbereich zu erfassen und anzuzeigen. In diesem Artikel werden das Schema für die Protokolle und ein Beispielprotokolleintrag bereitgestellt.


## <a name="schema-for-publishdelivery-failure-logs"></a>Schema für Veröffentlichungs-/Zustellungsfehlerprotokolle

| Eigenschaftenname | Datentyp | BESCHREIBUNG |
| ------------- | --------- | ----------- | 
| Time | Datetime | Der Zeitpunkt, zu dem der Protokolleintrag generiert wurde. <p>**Beispielwert:**  01-29-2020 09:52:02.700</p> |
| EventSubscriptionName | String | Der Name des Ereignisabonnements. <p>**Beispielwert:** „EVENTSUB1“</p> <p>Diese Eigenschaft ist nur für Zustellungsfehlerprotokolle vorhanden.</p>  |
| Category | String | Der Name der Protokollkategorie. <p>**Beispielwerte:** „DeliveryFailures“ oder „PublishFailures“ | 
| Vorgangsname | String | Der Name des Vorgangs hat den Fehler verursacht.<p>**Beispielwerte:** „Deliver“ für Zustellungsfehler. |
| `Message` | String | Die Protokollnachricht für den Benutzer, in der der Grund für den Fehler und andere zusätzliche Details erläutert werden. |
| resourceId | String | Die Ressourcen-ID für die Themen-/Domänenressource.<p>**Beispielwerte:** `/SUBSCRIPTIONS/SAMPLE-SUBSCRIPTION-ID/RESOURCEGROUPS/SAMPLE-RESOURCEGROUP/PROVIDERS/MICROSOFT.EVENTGRID/TOPICS/TOPIC1` |

## <a name="example"></a>Beispiel

```json
{
    "time": "2019-11-01T00:17:13.4389048Z",
    "resourceId": "/SUBSCRIPTIONS/SAMPLE-SUBSCTIPTION-ID /RESOURCEGROUPS/SAMPLE-RESOURCEGROUP-NAME/PROVIDERS/MICROSOFT.EVENTGRID/TOPICS/SAMPLE-TOPIC-NAME ",
    "eventSubscriptionName": "SAMPLEDESTINATION",
    "category": "DeliveryFailures",
    "operationName": "Deliver",
    "message": "Message:outcome=NotFound, latencyInMs=2635, id=xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxxx, systemId=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx, state=FilteredFailingDelivery, deliveryTime=11/1/2019 12:17:10 AM, deliveryCount=0, probationCount=0, deliverySchema=EventGridEvent, eventSubscriptionDeliverySchema=EventGridEvent, fields=InputEvent, EventSubscriptionId, DeliveryTime, State, Id, DeliverySchema, LastDeliveryAttemptTime, SystemId, fieldCount=, requestExpiration=1/1/0001 12:00:00 AM, delivered=False publishTime=11/1/2019 12:17:10 AM, eventTime=11/1/2019 12:17:09 AM, eventType=Type, deliveryTime=11/1/2019 12:17:10 AM, filteringState=FilteredWithRpc, inputSchema=EventGridEvent, publisher=DIAGNOSTICLOGSTEST-EASTUS.EASTUS-1.EVENTGRID.AZURE.NET, size=363, fields=Id, PublishTime, SerializedBody, EventType, Topic, Subject, FilteringHashCode, SystemId, Publisher, FilteringTopic, TopicCategory, DataVersion, MetadataVersion, InputSchema, EventTime, fieldCount=15, url=sb://diagnosticlogstesting-eastus.servicebus.windows.net/, deliveryResponse=NotFound: The messaging entity 'sb://diagnosticlogstesting-eastus.servicebus.windows.net/eh-diagnosticlogstest' could not be found. TrackingId:c98c5af6-11f0-400b-8f56-c605662fb849_G14, SystemTracker:diagnosticlogstesting-eastus.servicebus.windows.net:eh-diagnosticlogstest, Timestamp:2019-11-01T00:17:13, referenceId: ac141738a9a54451b12b4cc31a10dedc_G14:"
}
```

Die möglichen Werte von `Outcome` sind `Aborted`, `TimedOut`, `GenericError` und `Busy`. Event Grid protokolliert alle Informationen, die der Dienst vom Ereignishandler in `message` empfängt. Für `GenericError` werden beispielsweise der HTTP-Statuscode, der Fehlercode und die Fehlermeldung protokolliert. 

## <a name="next-steps"></a>Nächste Schritte
Weitere Informationen zum Aktivieren von Diagnoseprotokollen für Themen oder Domänen finden Sie unter [Aktivieren von Diagnoseprotokollen](enable-diagnostic-logs-topic.md).
