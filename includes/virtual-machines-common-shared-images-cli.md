---
title: include file
description: include file
services: virtual-machines
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 03/24/2020
ms.author: cynthn
ms.custom: include file, devx-track-azurecli
ms.openlocfilehash: fc35f277bfafa80f6239ef807f1a83591646a503
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/20/2021
ms.locfileid: "107799866"
---
## <a name="create-an-image-gallery"></a>Erstellen eines Imagekatalogs 

Ein Imagekatalog ist die primäre Ressource, die zur Ermöglichung des Teilens von Images verwendet wird. 

Zulässige Zeichen für Katalognamen sind Groß- und Kleinbuchstaben, Zahlen und Punkte. Der Katalogname darf keine Bindestriche enthalten.   Katalognamen müssen innerhalb Ihres Abonnements eindeutig sein. 

Erstellen Sie einen Imagekatalog mit [az sig create](/cli/azure/sig#az_sig_create). Im folgenden Beispiel werden eine Ressourcengruppe namens *myGalleryRG* in der Region *USA, Osten* und ein Katalog namens *myGallery* erstellt.

```azurecli-interactive
az group create --name myGalleryRG --location eastus
az sig create --resource-group myGalleryRG --gallery-name myGallery
```

## <a name="share-the-gallery"></a>Teilen des Katalogs

Sie können Images zwischen Abonnements mithilfe der rollenbasierten Zugriffssteuerung (Role-Based Access Control, RBAC) teilen. Sie können Images auf der Ebene des Katalogs, der Imagedefinition oder der Imageversion freigeben. Jeder Benutzer, der Leseberechtigungen für eine Imageversion besitzt (auch über Abonnements hinweg), kann mithilfe der Imageversion einen virtuellen Computer bereitstellen.

Wir empfehlen, dass Sie mit anderen Benutzern auf Ebene des Katalogs teilen. Um die Objekt-ID Ihres Katalogs abzurufen, verwenden Sie [az sig show](/cli/azure/sig#az_sig_show).

```azurecli-interactive
az sig show \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   --query id
```

Verwenden Sie die Objekt-ID als Bereich, zusammen mit einer E-Mail-Adresse und [az role assignment create](/cli/azure/role/assignment#az_role_assignment_create), um einem Benutzer Zugriff auf den geteilten Imagekatalog zu gewähren. Ersetzen Sie `<email-address>` und `<gallery iD>` durch Ihre eigenen Angaben.

```azurecli-interactive
az role assignment create \
   --role "Reader" \
   --assignee <email address> \
   --scope <gallery ID>
```

Weitere Informationen zum Teilen von Ressourcen mittels RBAC finden Sie unter [Verwalten des Zugriffs mithilfe von RBAC und der Azure CLI](../articles/role-based-access-control/role-assignments-cli.md).