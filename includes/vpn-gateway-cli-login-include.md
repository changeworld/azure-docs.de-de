---
title: include file
description: include file
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/21/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 74d7bd087df4b00c0bafb5ec33fbbdfa5c57b379
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/29/2021
ms.locfileid: "92755677"
---
Melden Sie sich mit dem Befehl [az login](/cli/azure/) bei Ihrem Azure-Abonnement an, und befolgen Sie die Anweisungen auf dem Bildschirm. Weitere Informationen zum Anmelden finden Sie unter [Erste Schritte mit der Azure CLI](/cli/azure/get-started-with-azure-cli).

```azurecli
az login
```

Listen Sie die Abonnements für das Konto auf, wenn Sie über mehr als ein Azure-Abonnement verfügen.

```azurecli
az account list --all
```

Geben Sie das Abonnement an, das Sie verwenden möchten.

```azurecli
az account set --subscription <replace_with_your_subscription_id>
```
