---
title: Vertrauenswürdiger Speicher für Media Services
description: Dank der Authentifizierung verwalteter Identitäten kann von Media Services per vertrauenswürdigem Speicherzugriff auf das Speicherkonto zugegriffen werden, das mit einer Firewall oder mit einer VNet-Einschränkung konfiguriert wurde.
keywords: vertrauenswürdiger Speicher, verwaltete Identitäten
services: media-services
author: IngridAtMicrosoft
manager: femila
ms.service: media-services
ms.topic: conceptual
ms.date: 1/29/2020
ms.author: inhenkel
ms.openlocfilehash: 87f52bef07644370c268168c984a452d5ae3f9bd
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/03/2021
ms.locfileid: "106279458"
---
# <a name="trusted-storage-for-media-services"></a>Vertrauenswürdiger Speicher für Media Services

Wenn Sie ein Media Services-Konto erstellen, müssen Sie es einem Speicherkonto zuordnen. Von Media Services kann per Systemauthentifizierung oder Authentifizierung verwalteter Identitäten auf dieses Speicherkonto zugegriffen werden. Media Services überprüft, ob sich das Media Services-Konto und das Speicherkonto im gleichen Abonnement befinden und ob der Benutzer, der die Zuordnung hinzufügt, Zugriff auf das Speicherkonto hat (mittels rollenbasierter Zugriffssteuerung für Azure Resource Manager).

>[!NOTE]
>Der vertrauenswürdige Speicher ist nur in der API verfügbar und derzeit nicht im Azure-Portal aktiviert.

## <a name="trusted-storage-with-a-firewall"></a>Vertrauenswürdiger Speicher mit einer Firewall

Wenn Sie jedoch eine Firewall einsetzen möchten, um Ihr Speicherkonto zu schützen und vertrauenswürdigen Speicher zu ermöglichen, ist die Authentifizierung mit [verwalteten Identitäten](concept-managed-identities.md) die bevorzugte Option. Dadurch kann von Media Services per vertrauenswürdigem Speicherzugriff auf das Speicherkonto zugegriffen werden, das mit einer Firewall oder mit einer VNET-Einschränkung konfiguriert wurde.

## <a name="tutorial"></a>Lernprogramm

Weitere Informationen zum Aktivieren von vertrauenswürdigem Speicher finden Sie im Tutorial [Media Services: Vertrauenswürdiger Speicher](security-trusted-storage-rest-tutorial.md).

> [!NOTE]
> Sie müssen den Zugriff „Mitwirkender an Storage-Blobdaten“ der Funktion „Verwaltete Identität“ von Azure Media Services gewähren, damit Media Services Speicherkontodaten lesen und schreiben kann.  Das Gewähren der allgemeinen Rolle „Mitwirkender“ funktioniert nicht, da dadurch nicht die ordnungsgemäßen Berechtigungen auf Datenebene aktiviert werden.

## <a name="further-reading"></a>Weiterführende Themen

Um die Methoden zum Erstellen von vertrauenswürdigem Speicher mit verwalteten Identitäten zu verstehen, lesen Sie [Verwaltete Identitäten und Media Services](concept-managed-identities.md).

Weitere Informationen zu vertrauenswürdigen Microsoft-Diensten finden Sie unter [Konfigurieren von Azure Storage-Firewalls und virtuellen Netzwerken](../../storage/common/storage-network-security.md#trusted-microsoft-services).

## <a name="next-steps"></a>Nächste Schritte

Weitere Informationen zu den Funktionen verwalteter Identitäten für Sie und Ihre Azure-Anwendungen finden Sie unter [Was sind verwaltete Identitäten für Azure-Ressourcen?](../../active-directory/managed-identities-azure-resources/overview.md).
