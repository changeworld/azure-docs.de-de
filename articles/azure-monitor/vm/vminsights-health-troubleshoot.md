---
title: Problembehandlung für das Feature „Gastintegrität“ von VM Insights (Vorschau)
description: Enthält eine Beschreibung der Problembehandlungsschritte, die Sie ausführen können, wenn Probleme mit der Integrität in VM Insights auftreten.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 02/25/2021
ms.openlocfilehash: 834c70e02ab25fa6dcadb5f6c997be09aaf5e353
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/30/2021
ms.locfileid: "105932776"
---
# <a name="troubleshoot-vm-insights-guest-health-preview"></a>Problembehandlung für das Feature „Gastintegrität“ von VM Insights (Vorschau)
Dieser Artikel enthält eine Beschreibung der Problembehandlungsschritte, die Sie ausführen können, wenn Probleme mit der Integrität in VM Insights auftreten.


## <a name="upgrade-available-message-is-still-displayed-after-upgrading-guest-health"></a>Meldung „Upgrade verfügbar“ wird nach dem Upgrade des Features „Gastintegrität“ weiterhin angezeigt 

- Vergewissern Sie sich, dass die VM in Azure weltweit ausgeführt wird. Server mit Arc-Unterstützung werden noch nicht unterstützt.
- Überprüfen Sie, ob die Region und die Betriebssystemversion des virtuellen Computers unterstützt werden, wie es unter [Aktivieren des Features „Gastintegrität“ von Azure Monitor für VMs (Vorschau)](vminsights-health-enable.md) beschrieben ist.
- Vergewissern Sie sich, dass die Erweiterung für die Gastintegrität erfolgreich mit Exitcode 0 installiert wurde.
- Stellen Sie sicher, dass die Azure Monitor-Agent-Erweiterung erfolgreich installiert wurde.
- Vergewissern Sie sich, dass die systemseitig zugewiesene verwaltete Identität für den virtuellen Computer aktiviert ist.
- Stellen Sie sicher, dass für den virtuellen Computer keine benutzerseitig zugewiesenen verwalteten Identitäten angegeben sind.
- Überprüfen Sie, ob das Gebietsschema für virtuelle Windows-Computer auf *Englisch (USA)* festgelegt ist. Die Lokalisierung wird von Azure Monitor-Agent derzeit nicht unterstützt.
- Vergewissern Sie sich, dass der virtuelle Computer nicht den Netzwerkproxy verwendet. Der Azure Monitor-Agent unterstützt derzeit keine Proxys.
- Überprüfen Sie, ob der Integritätserweiterungs-Agent fehlerfrei gestartet wurde. Wenn der Agent nicht gestartet werden kann, ist der Status des Agents möglicherweise beschädigt. Löschen Sie den Inhalt des Ordners mit dem Agentstatus, und starten Sie den Agent neu.
  - Für Linux: Der Daemon ist *vmGuestHealthAgent*. Der Statusordner ist */var/opt/vmGuestHealthAgent/* *.
  - Für Windows: Der Dienst ist *VM-Gastintegritäts-Agent*. Der Statusordner ist _%ProgramData%\Microsoft\VMGuestHealthAgent\\*_ .
- Überprüfen Sie, ob der Azure Monitor-Agent über Netzwerkkonnektivität verfügt. 
  - Versuchen Sie, vom virtuellen Computer einen Pingbefehl an _<region>.handler.control.monitor.azure.com_ zu senden. Bei einem virtuellen Computer in Westeuropa versuchen Sie beispielsweise, den Pingbefehl _westeurope.handler.control.monitor.azure.com:443_ zu senden.
- Vergewissern Sie sich, dass der virtuelle Computer eine Zuordnung zu einer Datensammlungsregel in derselben Region wie der Log Analytics-Arbeitsbereich aufweist.
  -  Informationen zur Überprüfung, ob die Struktur der Datensammlungsregel korrekt ist, finden Sie im Abschnitt **Erstellen einer Datensammlungsregel** unter [Aktivieren des Features „Gastintegrität“ von Azure Monitor für VMs (Vorschau)](vminsights-health-enable.md). Achten Sie insbesondere auf das Vorhandensein des Abschnitts *performanceCounters* für die Datenquellen, die zum Abrufen von drei Leistungsindikatoren eingerichtet wurden, und auf das Vorhandensein des Abschnitts *inputDataSources* in der Konfiguration der Integritätserweiterung, um Leistungsindikatoren an die Erweiterung zu senden.
-  Überprüfen Sie den virtuellen Computer auf Fehler bei der Erweiterung für die Gastintegrität.
   -  Für Linux: Überprüfen Sie die Protokolle unter _/var/log/Azure/Microsoft.Azure.Monitor.VirtualMachines.GuestHealthLinuxAgent/*.log_.
   -  Für Windows: Überprüfen Sie die Protokolle unter _C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.Monitor.VirtualMachines.GuestHealthWindowsAgent\{Erweiterungsversion}\*.log_.
-  Überprüfen Sie den virtuellen Computer auf Fehler des Azure Monitor-Agents.
   -  Für Linux: Überprüfen Sie die Protokolle unter _/var/log/mdsd.*_ .
   -  Für Windows: Überprüfen Sie die Protokolle unter _C:\WindowsAzure\Resources\*{vmName}.AMADataStore_.
 



## <a name="error-message-that-no-data-is-available"></a>Fehlermeldung: Keine Daten verfügbar 

![Keine Daten](media/vminsights-health-troubleshoot/no-data.png)


### <a name="verify-that-the-virtual-machine-meets-configuration-requirements"></a>Überprüfen, ob der virtuelle Computer die Konfigurationsanforderungen erfüllt

1. Vergewissern Sie sich, dass die VM ein virtueller Azure-Computer ist. Azure Arc für Server wird derzeit nicht unterstützt.
2. Vergewissern Sie sich, dass auf dem virtuellen Computer ein [unterstütztes Betriebssystem](vminsights-health-enable.md?current-limitations.md) ausgeführt wird.
3. Stellen Sie sicher, dass der virtuelle Computer in einer [unterstützten Region](vminsights-health-enable.md?current-limitations.md) installiert ist.
4. Stellen Sie sicher, dass der Log Analytics-Arbeitsbereich in einer [unterstützten Region](vminsights-health-enable.md?current-limitations.md) installiert ist.

### <a name="verify-that-the-vm-is-properly-onboarded"></a>Überprüfen, ob das Onboarding für die VM korrekt durchgeführt wurde
Vergewissern Sie sich, dass die Bereitstellung der Azure Monitor-Agent-Erweiterung und des Agents für die Integrität der Gast-VM auf dem virtuellen Computer erfolgreich war. Wählen Sie im Azure-Portal im Menü des virtuellen Computers die Option **Erweiterungen** aus, und vergewissern Sie sich, dass die beiden Agents aufgelistet sind.

![VM-Erweiterungen](media/vminsights-health-troubleshoot/extensions.png)

### <a name="verify-the-system-assigned-identity-is-enabled-on-the-virtual-machine"></a>Überprüfen, ob auf dem virtuellen Computer die systemseitig zugewiesene Identität aktiviert ist
Überprüfen Sie, ob auf dem virtuellen Computer die systemseitig zugewiesene Identität aktiviert ist. Wählen Sie im Azure-Portal im Menü des virtuellen Computers die Option **Identität** aus. Wenn die vom Benutzer verwaltete Identität aktiviert ist, kann der Azure Monitor-Agent, unabhängig vom Status der vom System verwalteten Identität, nicht mit dem Konfigurationsdienst kommunizieren, und die Erweiterung für die Gastintegrität funktioniert nicht.

![Systemzugewiesene Identität](media/vminsights-health-troubleshoot/system-identity.png)

### <a name="verify-data-collection-rule"></a>Überprüfen der Datensammlungsregel
Vergewissern Sie sich, dass die Datensammlungsregel, mit der die Integritätserweiterung als Datenquelle angegeben wird, dem virtuellen Computer zugeordnet ist.

## <a name="error-message-for-bad-request-due-to-insufficient-permissions"></a>Fehlermeldung: Ungültige Anforderung aufgrund von unzureichenden Berechtigungen
Dieser Fehler ist ein Hinweis darauf, dass der Ressourcenanbieter **Microsoft.WorkloadMonitor** unter dem Abonnement nicht registriert wurde. Ausführliche Informationen zum Registrieren dieses Ressourcenanbieters finden Sie unter [Azure-Ressourcenanbieter und -typen](../../azure-resource-manager/management/resource-providers-and-types.md#register-resource-provider). 

![Ungültige Anforderung](media/vminsights-health-troubleshoot/bad-request.png)

## <a name="health-shows-as-unknown-after-guest-health-is-enabled"></a>Integrität wird nach Aktivieren der Gastintegrität als „unbekannt“ angezeigt

### <a name="verify-that-performance-counters-on-windows-nodes-are-working-correctly"></a>Überprüfen, ob die Leistungsindikatoren auf Windows-Knoten ordnungsgemäß funktionieren 
Die Gastintegrität basiert darauf, dass der Agent Leistungsindikatoren vom Knoten sammeln kann. Der Basissatz der Leistungsindikatorbibliotheken ist möglicherweise beschädigt und muss neu erstellt werden. Folgen Sie den Anweisungen unter [Manuelles Neuerstellen von Leistungsindikatorbibliothekswerten](/troubleshoot/windows-server/performance/rebuild-performance-counter-library-values), um die Leistungsindikatoren neu zu erstellen.





## <a name="next-steps"></a>Nächste Schritte

- [Übersicht über das Feature „Gastintegrität“ von VM Insights](vminsights-health-overview.md)