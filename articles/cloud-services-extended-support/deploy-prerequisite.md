---
title: Voraussetzungen für die Bereitstellung von Azure Cloud Services (erweiterter Support)
description: Voraussetzungen für die Bereitstellung von Azure Cloud Services (erweiterter Support)
ms.topic: article
ms.service: cloud-services-extended-support
author: gachandw
ms.author: gachandw
ms.reviewer: mimckitt
ms.date: 10/13/2020
ms.custom: ''
ms.openlocfilehash: bce09fad6ffa169a019628498a686226eff266c7
ms.sourcegitcommit: 77d7639e83c6d8eb6c2ce805b6130ff9c73e5d29
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/05/2021
ms.locfileid: "106384975"
---
# <a name="prerequisites-for-deploying-azure-cloud-services-extended-support"></a>Voraussetzungen für die Bereitstellung von Azure Cloud Services (erweiterter Support)

Um eine erfolgreiche Bereitstellung von Cloud Services (erweiterter Support) zu gewährleisten, führen Sie die folgenden Schritte durch und schließen jeden Punkt ab, bevor Sie mit der Bereitstellung beginnen. 

## <a name="required-service-configuration-cscfg-file-updates"></a>Erforderliche Aktualisierungen der Dienstkonfigurationsdatei (CSCFG-Datei)

### <a name="1-virtual-network"></a>1) Virtuelles Netzwerk
Bereitstellungen von Cloud Services (erweiterter Support) müssen sich in einem virtuellen Netzwerk befinden. Das virtuelle Netzwerk kann über das [Azure-Portal](../virtual-network/quick-create-portal.md), mit [PowerShell](../virtual-network/quick-create-powershell.md), der [Azure CLI](../virtual-network/quick-create-cli.md) oder einer [ARM-Vorlage](../virtual-network/quick-create-template.md) erstellt werden. Auf das virtuelle Netzwerk und die Subnetze muss außerdem in der Dienstkonfiguration (CSCFG-Datei) im Abschnitt [NetworkConfiguration](schema-cscfg-networkconfiguration.md) verwiesen werden. 

Bei virtuellen Netzwerken, die derselben Ressourcengruppe wie der Clouddienst angehören, genügt das Verweisen auf den Namen des virtuellen Netzwerks in der Dienstkonfigurationsdatei (CSCFG-Datei). Wenn sich das virtuelle Netzwerk und der Clouddienst in zwei unterschiedlichen Ressourcengruppen befinden, muss die vollständige Azure Resource Manager-ID des virtuellen Netzwerks in der Dienstkonfigurationsdatei (CSCFG-Datei) angegeben werden.
 
#### <a name="virtual-network-located-in-same-resource-group"></a>Virtuelles Netzwerk in derselben Ressourcengruppe
```xml
<VirtualNetworkSite name="<vnet-name>"/> 
  <AddressAssignments> 
    <InstanceAddress roleName="<role-name>"> 
     <Subnets> 
       <Subnet name="<subnet-name>"/> 
     </Subnets> 
    </InstanceAddress> 
```

#### <a name="virtual-network-located-in-different-resource-group"></a>Virtuelles Netzwerk in einer anderen Ressourcengruppe
```xml
<VirtualNetworkSite name="/subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/Microsoft.Network/virtualNetworks/<vnet-name>"/> 
   <AddressAssignments> 
     <InstanceAddress roleName="<role-name>"> 
       <Subnets> 
        <Subnet name="<subnet-name>"/> 
       </Subnets> 
     </InstanceAddress> 
```
### <a name="2-remove-the-old-plugins"></a>2) Entfernen der alten Plug-Ins

Entfernen Sie alte Remotedesktopeinstellungen aus der Dienstkonfigurationsdatei (CSCFG-Datei).  

```xml
<Setting name="Microsoft.WindowsAzure.Plugins.RemoteAccess.Enabled" value="true" /> 
<Setting name="Microsoft.WindowsAzure.Plugins.RemoteAccess.AccountUsername" value="gachandw" /> 
<Setting name="Microsoft.WindowsAzure.Plugins.RemoteAccess.AccountEncryptedPassword" value="XXXX" /> 
<Setting name="Microsoft.WindowsAzure.Plugins.RemoteAccess.AccountExpiration" value="2021-12-17T23:59:59.0000000+05:30" /> 
<Setting name="Microsoft.WindowsAzure.Plugins.RemoteForwarder.Enabled" value="true" /> 
```
Entfernen Sie die alten Diagnoseeinstellungen für jede Rolle in der Dienstkonfigurationsdatei (.cscfg).

```xml
<Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" value="UseDevelopmentStorage=true" />
```

## <a name="required-service-definition-file-csdef-updates"></a>Erforderliche Aktualisierungen der Dienstdefinitionsdatei (CSDEF-Datei)

> [!NOTE]
> Für Änderungen in der Dienstdefinitionsdatei (CSDEF) muss die Paketdatei (CSPKG) erneut generiert werden. Um die neuesten Einstellungen für Ihren Clouddienst zu erhalten, erstellen Sie die CSPKG-Datei, und verpacken Sie sie neu, nachdem Sie die folgenden Änderungen in der CSDEF-Datei vorgenommen haben.

### <a name="1-virtual-machine-sizes"></a>1) VM-Größen
Die folgenden Größen sind in Azure Resource Manager veraltet. Wenn Sie diese jedoch weiterhin verwenden möchten, aktualisieren Sie den `vmsize`-Namen mit der zugeordneten Azure Resource Manager-Benennungskonvention.  

| Vorheriger Name der Größe | Aktualisierter Name der Größe | 
|---|---|
| Sehr klein | Standard_A0 | 
| Small | Standard_A1 |
| Medium | Standard_A2 | 
| Large | Standard_A3 | 
| Extragroß | Standard_A4 | 
| A5 | Standard_A5 | 
| A6 | Standard_A6 | 
| A7 | Standard_A7 |  
| A8 | Standard_A8 | 
| A9 | Standard_A9 |
| A10 | Standard_A10 | 
| A11 | Standard_A11 | 
| MSODSG5 | Standard_MSODSG5 | 

 Beispielsweise wird `<WorkerRole name="WorkerRole1" vmsize="Medium"` zu `<WorkerRole name="WorkerRole1" vmsize="Standard_A2"`.
 
> [!NOTE]
> Informationen zum Abrufen einer Liste der verfügbaren Größen finden Sie unter [Ressourcen-SKUs: Liste](/rest/api/compute/resourceskus/list). Wenden Sie die folgenden Filter an: <br>
`ResourceType = virtualMachines ` <br>
`VMDeploymentTypes = PaaS `


### <a name="2-remove-old-remote-desktop-plugins"></a>2) Entfernen alter Remotedesktop-Plug-Ins
In Bereitstellungen, die die alten Remotedesktop-Plug-Ins genutzt haben, müssen die Module aus der Dienstdefinitionsdatei (CSDEF-Datei) und allen zugehörigen Zertifikaten entfernt werden. 

```xml
<Imports> 
<Import moduleName="RemoteAccess" /> 
<Import moduleName="RemoteForwarder" /> 
</Imports> 
```
Für Bereitstellungen, die die alten Diagnose-Plug-Ins verwendet haben, müssen die Einstellungen für jede Rolle aus der Dienstdefinitionsdatei (.csdef) entfernt werden.

```xml
<Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" />
```

## <a name="key-vault-creation"></a>Erstellen einer Key Vault-Instanz 

Key Vault wird zum Speichern von Zertifikaten verwendet, die Cloud Services (erweiterter Support) zugeordnet sind. Fügen Sie Key Vault die Zertifikate hinzu, und verweisen Sie dann in der Dienstkonfigurationsdatei auf die Zertifikatfingerabdrücke. Außerdem müssen Sie (im Portal) in Key Vault „Zugriffsrichtlinien“ für „Azure Virtual Machines für Bereitstellung“ aktivieren, damit Cloud Services (erweiterter Support) als Geheimnisse gespeicherte Zertifikate aus Key Vault abrufen kann. Sie können einen Schlüsseltresor im [Azure-Portal](../key-vault/general/quick-create-portal.md) oder mithilfe von [PowerShell](../key-vault/general/quick-create-powershell.md) erstellen. Der Schlüsseltresor muss in derselben Region und demselben Abonnement wie der Clouddienst erstellt werden. Weitere Informationen finden Sie unter [Verwenden von Zertifikaten mit Azure Cloud Services (erweiterter Support)](certificates-and-key-vault.md).

## <a name="next-steps"></a>Nächste Schritte 
- Überprüfen Sie die [Bereitstellungsvoraussetzungen](deploy-prerequisite.md) für Cloud Services (erweiterter Support).
- Stellen Sie eine Cloud Service-Instanz (erweiterter Support) über das [Azure-Portal](deploy-portal.md), mit [PowerShell](deploy-powershell.md), einer [Vorlage](deploy-template.md) oder [Visual Studio](deploy-visual-studio.md) bereit.
- Sehen Sie sich die [häufig gestellten Fragen](faq.md) zu Cloud Services (erweiterter Support) an.
- Besuchen Sie das [Beispielrepository zu Cloud Services (erweiterter Support)](https://github.com/Azure-Samples/cloud-services-extended-support).
