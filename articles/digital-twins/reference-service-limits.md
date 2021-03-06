---
title: Diensteinschränkungen
titleSuffix: Azure Digital Twins
description: Diagramm mit den Diensteinschränkungen von Azure Digital Twins.
author: baanders
ms.author: baanders
ms.date: 04/08/2021
ms.topic: article
ms.service: digital-twins
ms.openlocfilehash: bc2dee9f1aa24708197b7edc53bb30e6d851666d
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 05/06/2021
ms.locfileid: "108763348"
---
# <a name="azure-digital-twins-service-limits"></a>Einschränkungen des Azure Digital Twins-Diensts

Dies sind die Diensteinschänkungen von Azure Digital Twins.

> [!NOTE]
> In einigen Bereichen dieses Diensts lassen sich Grenzwerte anpassen. Diese sind in der nachfolgenden Tabelle in der Spalte *Anpassbar?* entsprechend gekennzeichnet. Wenn der Grenzwert angepasst werden kann, enthält die Spalte *Anpassbar?* den Wert *Ja*.
>
> Falls Ihr Unternehmen die Anhebung eines anpassbaren Grenzwertes oder Kontingents über den Standardgrenzwert hinaus benötigt, können Sie zusätzliche Ressourcen anfordern, indem Sie [ein Supportticket eröffnen](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="limits-by-type"></a>Grenzwerte nach Typ

[!INCLUDE [Azure Digital Twins limits](../../includes/digital-twins-limits.md)]

## <a name="working-with-limits"></a>Arbeiten mit Grenzwerten

Wenn ein Grenzwert erreicht wird, drosselt der Dienst zusätzliche Anforderungen. Dies führt zu einer 429-Fehlerantwort von diesen Anforderungen.

Um dies zu verwalten, finden Sie hier einige Empfehlungen für die Arbeit mit Grenzwerten.
* **Verwenden Sie Wiederholungslogik.** Die [Azure Digital Twins SDKs](concepts-apis-sdks.md) implementieren Wiederholungslogik für fehlgeschlagene Anforderungen. Wenn Sie also mit einem bereitgestellten SDK arbeiten, ist dies bereits integriert. Andernfalls sollten Sie die Wiederholungslogik in Ihrer eigenen Anwendung implementieren. Der Dienst sendet einen `Retry-After`-Header in der Fehlerantwort zurück, den Sie verwenden können, um zu bestimmen, wie lange vor der Wiederholung gewartet werden soll.
* **Verwenden Sie Schwellenwerte und Benachrichtigungen, um vor dem Erreichen von Grenzwerten zu warnen.** Einige der Dienstgrenzwerte für Azure Digital Twins verfügen über entsprechende [Metriken](troubleshoot-metrics.md), die zum Nachverfolgen der Nutzung in diesen Bereichen verwendet werden können. Informationen zum Konfigurieren von Schwellenwerten und Einrichten einer Warnung für eine Metrik, wenn ein Schwellenwert erreicht wird, finden Sie in den Anweisungen unter [Problembehandlung: Einrichten von Warnungen](troubleshoot-alerts.md). Um Benachrichtigungen für andere Grenzwerte einzurichten, bei denen keine Metriken bereitgestellt werden, sollten Sie diese Logik in Ihren eigenen Anwendungscode implementieren.
* **Bereitstellung in großem Umfang über mehrere Instanzen hinweg.** Vermeiden Sie einen Single Point of Failure. Statt eines großen Diagramms für Ihre gesamte Bereitstellung sollten Sie erwägen, Teilmengen von Zwillingen logisch (z. B. nach Region oder Mandant) über mehrere Instanzen hinweg zu bereitzustellen. 

## <a name="next-steps"></a>Nächste Schritte

In der Übersicht über den Dienst erfahren Sie mehr über die aktuelle Version von Azure Digital Twins:
* [Übersicht: Was ist Azure Digital Twins?](overview.md)
