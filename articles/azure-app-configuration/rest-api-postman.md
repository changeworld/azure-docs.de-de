---
title: Azure Active Directory-REST-API – Testen über Postman
description: Verwenden von Postman zum Testen der Azure App Configuration-REST-API
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.topic: reference
ms.date: 08/17/2020
ms.openlocfilehash: 0b4444b049d181c2f66f8b02a5202dd17a3f4b6b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/29/2021
ms.locfileid: "96932488"
---
# <a name="test-the-azure-app-configuration-rest-api-using-postman"></a>Testen der Azure App Configuration-REST-API über Postman

Zum Testen der REST-API über [Postman](https://www.getpostman.com/) müssen Sie die zur [Authentifizierung](./rest-api-authentication-hmac.md) erforderlichen HTTP-Header in Ihre Anforderungen einfügen. So konfigurieren Sie Postman zum Testen der REST-API und automatischen Generieren der Authentifizierungsheader

1. Erstellen Sie eine neue [Anforderung](https://learning.getpostman.com/docs/postman/sending_api_requests/requests/).
1. Fügen Sie die Funktion `signRequest` aus dem [JavaScript-Authentifizierungsbeispiel](./rest-api-authentication-hmac.md#javascript) im [Voranforderungsskript](https://learning.getpostman.com/docs/postman/scripts/pre_request_scripts/) für die Anforderung hinzu.
1. Fügen Sie am Ende des Voranforderungsskripts den folgenden Code ein. Aktualisieren Sie den Zugriffsschlüssel, wie im TODO-Kommentar angegeben.

    ```js
    // TODO: Replace the following placeholders with your access key
    var credential = "<Credential>"; // Id
    var secret = "<Secret>"; // Value

    var isBodyEmpty = pm.request.body === null || pm.request.body === undefined || pm.request.body.isEmpty();

    var headers = signRequest(
        pm.request.url.getHost(),
        pm.request.method,
        pm.request.url.getPathWithQuery(),
        isBodyEmpty ? undefined : pm.request.body.toString(),
        credential,
        secret);

    // Add headers to the request
    headers.forEach(header => {
        pm.request.headers.upsert({key: header.name, value: header.value});
    })
    ```

1. Senden der Anforderung
