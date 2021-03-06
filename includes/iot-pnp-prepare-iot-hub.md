---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 03/17/2020
ms.custom: references_regions, devx-track-azurecli
ms.openlocfilehash: ea5d4ef26fb14e22b871bb4bfa1054cb749d38e8
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/29/2021
ms.locfileid: "97673023"
---
## <a name="prepare-an-iot-hub"></a>Vorbereiten eines IoT-Hubs

Für die Schritte in diesem Artikel benötigen Sie eine Azure IoT Hub-Instanz in Ihrem Azure-Abonnement. Wenn Sie kein Azure-Abonnement besitzen, erstellen Sie ein [kostenloses Konto](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), bevor Sie beginnen.

Wenn Sie die Azure CLI lokal verwenden, melden Sie sich zunächst mithilfe des Befehls `az login` bei Ihrem Azure-Abonnement an. Wenn Sie diese Befehle in der Azure Cloud Shell ausführen, werden Sie automatisch angemeldet.

Wenn Sie die Azure-Befehlszeilenschnittstelle lokal verwenden, sollte mindestens `az`-Version **2.8.0** verwendet werden. Für Azure Cloud Shell wird die neueste Version verwendet. Überprüfen Sie die auf dem Computer installierte Version mit dem Befehl `az --version`.

Führen Sie den folgenden Befehl aus, um Ihrer Instanz die Microsoft Azure IoT-Erweiterung für die Azure CLI hinzuzufügen:

```azurecli-interactive
az extension add --name azure-iot
```

Wenn Sie noch keinen IoT-Hub besitzen, führen Sie die folgenden Befehle aus, um eine Ressourcengruppe und einen IoT-Hub im Free-Tarif in Ihrem Abonnement zu erstellen. Ersetzen Sie `<YourIoTHubName>` durch einen Hubnamen Ihrer Wahl.

```azurecli-interactive
az group create --name my-pnp-resourcegroup \
    --location centralus
az iot hub create --name <YourIoTHubName> \
    --resource-group my-pnp-resourcegroup --sku F1
```

Führen Sie den folgenden Befehl aus, um die Geräteidentität in Ihrem IoT-Hub zu erstellen. Ersetzen Sie die Platzhalter `<YourIoTHubName>` und `<YourDeviceID>` durch Ihren eigenen _IoT Hub-Namen_ und eine _Geräte-ID_ Ihrer Wahl.

```azurecli-interactive
az iot hub device-identity create --hub-name <YourIoTHubName> --device-id <YourDeviceID>
```
