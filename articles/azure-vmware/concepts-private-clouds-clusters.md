---
title: Konzepte – Private Clouds und Cluster
description: Hier erfahren Sie mehr über die wichtigsten Funktionen von softwaredefinierten Azure VMware Solution-Rechenzentren und vSphere-Clustern.
ms.topic: conceptual
ms.date: 05/13/2021
ms.openlocfilehash: 7b32752e368f9aac003a45f96f7c0477e9221481
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 05/19/2021
ms.locfileid: "110089612"
---
#  <a name="azure-vmware-solution-private-cloud-and-cluster-concepts"></a>Azure VMware Solution: Konzepte – Private Clouds und Cluster

Azure VMware Solution bietet VMware-basierte private Clouds in Azure. Hardware- und Softwarebereitstellungen für private Clouds sind vollständig in Azure integriert und automatisiert. Die Bereitstellung und Verwaltung der privaten Cloud erfolgt über das Azure-Portal, die Befehlszeilenschnittstelle oder PowerShell.  

Eine private Cloud umfasst Cluster mit Folgendem:

- Dedizierte Bare-Metal-Serverknoten, die mit VMware ESXi-Hypervisor bereitgestellt werden 
- vCenter Server für die Verwaltung von ESXi und vSAN 
- Software-Defined Networking in VMware NSX-T für VMs für vSphere-Workloads  
- VMware vSAN-Datenspeicher für virtuelle Computer für vSphere-Workloads  
- VMware HCX für Workloadmobilität  
- Die in Azure zugrunde liegenden Ressourcen (für die Konnektivität und den Betrieb der privaten Cloud erforderlich)

Private Clouds werden wie andere Ressourcen innerhalb eines Azure-Abonnements installiert und verwaltet. Die Anzahl der privaten Clouds innerhalb eines Abonnements ist skalierbar. Anfänglich besteht eine Begrenzung von einer privaten Cloud pro Abonnement.  Es besteht eine logische Beziehung zwischen Azure-Abonnements, privaten Azure VMware Solution-Clouds, vSAN-Clustern und Hosts. 

Die Abbildung zeigt ein einzelnes Azure-Abonnement mit zwei privaten Clouds, die für eine Entwicklungs- und eine Produktionsumgebung stehen. In jeder dieser privaten Clouds gibt es zwei Cluster. 

:::image type="content" source="media/hosts-clusters-private-clouds-final.png" alt-text="Abbildung von zwei privaten Clouds in einem Kundenabonnement":::

## <a name="hosts"></a>Hosts

[!INCLUDE [disk-capabilities-of-the-host](includes/disk-capabilities-of-the-host.md)]

## <a name="clusters"></a>Cluster

[!INCLUDE [hosts-minimum-initial-deployment-statement](includes/hosts-minimum-initial-deployment-statement.md)]

[!INCLUDE [azure-vmware-solutions-limits](includes/azure-vmware-solutions-limits.md)]

## <a name="vmware-software-versions"></a>Versionen von VMware-Software

[!INCLUDE [vmware-software-versions](includes/vmware-software-versions.md)]

## <a name="host-maintenance-and-lifecycle-management"></a>Hostwartung und Lebenszyklusverwaltung

Einer der Vorteile von privaten Azure VMware Solution-Clouds besteht darin, dass die Plattform für Sie verwaltet wird.  Microsoft ist für die Lebenszyklusverwaltung der VMware-Software (ESXi, vCenter und vSAN) zuständig. Außerdem ist Microsoft für die Lebenszyklusverwaltung von NSX-T-Appliances und das Bootstrapping der Netzwerkkonfiguration zuständig, z. B. die Erstellung des Gateways für Schicht 0 und die Aktivierung des Nord-Süd-Routings. Sie sind für die NSX-T-SDN-Konfiguration verantwortlich, also Netzwerksegmente, Regeln für verteilte Firewalls, Gateways für Schicht 1 und Lastenausgleich. 

[!INCLUDE [vmware-software-update-frequency](includes/vmware-software-update-frequency.md)]

## <a name="host-monitoring-and-remediation"></a>Hostüberwachung und -wartung

Azure VMware Solution überwacht kontinuierlich sowohl die Integrität der zugrunde liegenden Ressourcen als auch der Integrität der VMware-Komponenten. Wenn Azure VMware Solution einen Fehler erkennt, ergreift es Maßnahmen, um die fehlerhaften Komponenten zu reparieren. Wenn von Azure VMware Solution ein Leistungsabfall oder ein Fehler auf dem Azure VMware Solution-Knoten erkannt wird, wird der Fehlerbehebungsprozess für den Host ausgelöst. 

Die Wartung für den Host umfasst das Ersetzen des fehlerhaften Knotens durch einen neuen fehlerfreien Knoten im Cluster. Anschließend wird der fehlerhafte Host nach Möglichkeit in den VMware vSphere-Wartungsmodus versetzt. VMware vMotion verschiebt die virtuellen Computer vom fehlerhaften Host auf andere verfügbare Server im Cluster. Hierbei ist es unter Umständen möglich, eine Livemigration von Workloads ohne Ausfallzeiten durchzuführen. Wenn der fehlerhafte Host nicht in den Wartungsmodus versetzt werden kann, wird er aus dem Cluster entfernt.

In Azure VMware Solution werden die folgenden Bedingungen auf dem Host überwacht:  

- Prozessorstatus 
- Speicherstatus 
- Verbindung und Betriebszustand 
- Zustand des Hardwarelüfters 
- Verlust der Netzwerkkonnektivität 
- Zustand der Hardware-Hauptplatine 
- Aufgetretene Fehler auf den Datenträgern eines vSAN-Hosts 
- Hardwarespannung 
- Hardware-Temperaturstatus 
- Hardware-Betriebszustand 
- Speicherstatus 
- Verbindungsfehler 

> [!NOTE]
> Administratoren von Azure VMware Solution-Mandanten dürfen die obigen definierten VMware vCenter-Alarme nicht bearbeiten oder löschen, da diese über die Azure VMware Solution-Steuerungsebene in vCenter verwaltet werden. Diese Alarme werden von der Azure VMware Solution-Überwachung verwendet, um den Fehlerbehebungsprozess für den Azure VMware Solution-Host auszulösen.

## <a name="backup-and-restoration"></a>Sichern und Wiederherstellen

Die vCenter- und NSX-T-Konfigurationen der privaten Cloud werden stündlich gesichert.  Sicherungen werden drei Tage lang aufbewahrt. Wenn Sie eine Wiederherstellung auf Grundlage einer Sicherung durchführen möchten, öffnen Sie eine [Supportanfrage](https://rc.portal.azure.com/#create/Microsoft.Support) im Azure-Portal, um die Wiederherstellung anzufordern.

Azure VMware Solution überwacht kontinuierlich sowohl die Integrität der zugrunde liegenden Ressourcen als auch der Integrität der VMware-Komponenten. Wenn Azure VMware Solution einen Fehler erkennt, ergreift es Maßnahmen, um die fehlerhaften Komponenten zu reparieren.

## <a name="next-steps"></a>Nächste Schritte

Nachdem Sie jetzt die Konzepte der privaten Azure VMware Solution-Cloud kennengelernt haben, möchten Sie vielleicht mehr über Folgendes erfahren: 

- [Azure VMware Solution – Netzwerk- und Interkonnektivitätskonzepte](concepts-networking.md)
- [Speicherkonzepte von Azure VMware Solution](concepts-storage.md)
- [So aktivieren Sie die Azure VMware Solution-Ressource](deploy-azure-vmware-solution.md#step-1-register-the-microsoftavs-resource-provider)

<!-- LINKS - internal -->
[concepts-networking]: ./concepts-networking.md

<!-- LINKS - external-->
[VCSA versions]: https://kb.vmware.com/s/article/2143838
[ESXi versions]: https://kb.vmware.com/s/article/2143832
[vSAN versions]: https://kb.vmware.com/s/article/2150753

