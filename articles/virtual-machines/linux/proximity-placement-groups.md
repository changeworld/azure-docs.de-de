---
title: Erstellen einer Näherungsplatzierungsgruppe mithilfe der Azure-Befehlszeilenschnittstelle
description: Erfahren Sie mehr über das Erstellen und Verwenden von Näherungsplatzierungsgruppen für VMs in Azure.
author: cynthn
ms.service: virtual-machines
ms.subservice: proximity-placement-groups
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 3/8/2021
ms.author: cynthn
ms.openlocfilehash: e4f91afa86a0d99b4ce42e96295bf2ae1f9fcd9f
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/20/2021
ms.locfileid: "107771441"
---
# <a name="deploy-vms-to-proximity-placement-groups-using-azure-cli"></a>Bereitstellen von VMs für Näherungsplatzierungsgruppen mit Azure CLI

Um den Abstand zwischen den VMs so stark wie möglich zu verringern und somit die geringstmögliche Latenz zu erzielen, sollten Sie sie in einer [Näherungsplatzierungsgruppe](../co-location.md#proximity-placement-groups) bereitstellen.

Eine Näherungsplatzierungsgruppe ist eine logische Gruppierung, mit der ein möglichst geringer Abstand zwischen Azure-Compute-Ressourcen sichergestellt wird. Näherungsplatzierungsgruppen sind für Workloads hilfreich, die eine geringe Latenz erfordern.


## <a name="create-the-proximity-placement-group"></a>Erstellen der Näherungsplatzierungsgruppe
Erstellen Sie eine Näherungsplatzierungsgruppe mit [`az ppg create`](/cli/azure/ppg#az_ppg_create). 

```azurecli-interactive
az group create --name myPPGGroup --location westus
az ppg create \
   -n myPPG \
   -g myPPGGroup \
   -l westus \
   -t standard 
```

## <a name="list-proximity-placement-groups"></a>Auflisten von Näherungsplatzierungsgruppen

Mit [az ppg list](/cli/azure/ppg#az_ppg_list) können Sie alle Ihre Näherungsplatzierungsgruppen auflisten.

```azurecli-interactive
az ppg list -o table
```

## <a name="create-a-vm"></a>Erstellen einer VM

Erstellen Sie einen virtuellen Computer innerhalb der Näherungsplatzierungsgruppe mit [new az vm](/cli/azure/vm#az_vm_create).

```azurecli-interactive
az vm create \
   -n myVM \
   -g myPPGGroup \
   --image UbuntuLTS \
   --ppg myPPG  \
   --generate-ssh-keys \
   --size Standard_D1_v2  \
   -l westus
```

Sie können die VM in der Näherungsplatzierungsgruppe mit [az ppg show](/cli/azure/ppg#az_ppg_show) anzeigen.

```azurecli-interactive
az ppg show --name myppg --resource-group myppggroup --query "virtualMachines"
```

## <a name="availability-sets"></a>Verfügbarkeitsgruppen
Sie können auch eine Verfügbarkeitsgruppe in ihrer Näherungsplatzierungsgruppe erstellen. Verwenden Sie den gleichen `--ppg`-Parameter mit [az vm availability-set create](/cli/azure/vm/availability-set#az_vm_availability_set_create), um eine Verfügbarkeitsgruppe zu erstellen, und alle in der Verfügbarkeitsgruppe erstellten virtuellen Computer werden ebenfalls in derselben Näherungsplatzierungsgruppe erstellt.

## <a name="scale-sets"></a>Skalierungsgruppen

Sie können auch eine Skalierungsgruppe in ihrer Näherungsplatzierungsgruppe erstellen. Verwenden Sie den gleichen `--ppg`-Parameter mit [az vmss create](/cli/azure/vmss#az_vmss_create), um eine Skalierungsgruppe zu erstellen, und alle Instanzen werden in derselben Näherungsplatzierungsgruppe erstellt.

## <a name="next-steps"></a>Nächste Schritte

Erfahren Sie mehr über die [Azure CLI](/cli/azure/ppg)-Befehle für Näherungsplatzierungsgruppen.