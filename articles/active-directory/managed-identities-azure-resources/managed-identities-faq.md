---
title: Häufig gestellte Fragen zu verwalteten Identitäten für Azure-Ressourcen – Azure AD
description: Häufig gestellte Fragen zu verwalteten Identitäten
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: ''
ms.topic: conceptual
ms.tgt_pltfrm: ''
ms.workload: identity
ms.date: 04/08/2021
ms.author: barclayn
ms.openlocfilehash: 07b106630cffae75c5e4588d14de7ae938945614
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/16/2021
ms.locfileid: "107534123"
---
# <a name="managed-identities-for-azure-resources-frequently-asked-questions---azure-ad"></a>Häufig gestellte Fragen zu verwalteten Identitäten für Azure-Ressourcen – Azure AD

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

> [!NOTE]
> „Verwaltete Identitäten für Azure-Ressourcen“ ist der neue Name für den Dienst, der früher als „Verwaltete Dienstidentität“ (Managed Service Identity, MSI) bezeichnet wurde.

## <a name="administration"></a>Verwaltung

### <a name="how-can-you-find-resources-that-have-a-managed-identity"></a>Wie können Sie nach Ressourcen mit einer verwalteten Identität suchen?

Die Liste der Ressourcen, die über eine vom System zugewiesene verwaltete Identität verfügen, können Sie mit dem folgenden Azure CLI-Befehl abrufen: 

```azurecli-interactive
az resource list --query "[?identity.type=='SystemAssigned'].{Name:name,  principalId:identity.principalId}" --output table
```


### <a name="what-azure-rbac-permissions-are-required-to-managed-identity-on-a-resource"></a>Welche Azure RBAC-Berechtigungen sind für eine verwaltete Identität auf einer Ressource erforderlich? 

- Systemseitig zugewiesene verwaltete Identität: Sie benötigen Schreibberechtigungen für die Ressource. Für virtuelle Computer benötigen Sie z.B. „Microsoft.Compute/virtualMachines/write“. Diese Aktion ist in ressourcenspezifischen integrierten Rollen wie [Mitwirkender von virtuellen Computern](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) enthalten.
- Benutzerseitig zugewiesene verwaltete Identität: Sie benötigen Schreibberechtigungen für die Ressource. Für virtuelle Computer benötigen Sie z.B. „Microsoft.Compute/virtualMachines/write“. Zusätzlich zu der Rollenzuweisung [Operator für verwaltete Identität](../../role-based-access-control/built-in-roles.md#managed-identity-operator) für die verwaltete Identität.

### <a name="how-do-i-prevent-the-creation-of-user-assigned-managed-identities"></a>Wie verhindere ich die Erstellung von verwalteten Identitäten, die vom Benutzer zugewiesen werden?

Mit [Azure Policy](../../governance/policy/overview.md) können Sie Ihre Benutzer am Erstellen von vom Benutzer zugewiesenen verwalteten Identitäten hindern.

1. Navigieren Sie zum [Azure-Portal](https://portal.azure.com) und dann zu **Policy**.
2. Wählen Sie **Definitionen** aus.
3. Wählen Sie **+ Richtliniendefinition** aus, und geben Sie die erforderlichen Informationen ein.
4. Einfügen im Abschnitt „Richtlinienregel“
    
    ```json
    {
      "mode": "All",
      "policyRule": {
        "if": {
          "field": "type",
          "equals": "Microsoft.ManagedIdentity/userAssignedIdentities"
        },
        "then": {
          "effect": "deny"
        }
      },
      "parameters": {}
    }
    
    ```

Nachdem Sie die Richtlinie erstellt haben, weisen Sie sie der gewünschten Ressourcengruppe zu.

1. Navigieren Sie zu „Ressourcengruppen“.
2. Suchen Sie die Ressourcengruppe, die Sie zu Testzwecken verwenden möchten.
3. Wählen Sie im Menü auf der linken Seite **Richtlinien** aus.
4. Wählen Sie **Richtlinie zuweisen** aus.
5. Geben Sie im Abschnitt **Grundeinstellungen** Folgendes an:
    1. **Bereich**: die Ressourcengruppe, die Sie zu Testzwecken verwenden.
    1. **Richtliniendefinition**: die zuvor erstellte Richtlinie.
6. Übernehmen Sie für alle anderen Einstellungen die Standardwerte, und wählen Sie **Überprüfen + Erstellen** aus.

Ab diesem Zeitpunkt schlägt jeder Versuch, eine vom Benutzer zugewiesene verwaltete Identität in der Ressourcengruppe zu erstellen, fehl.

  ![Richtlinienverletzung](./media/known-issues/policy-violation.png)

## <a name="concepts"></a>Konzepte

### <a name="do-managed-identities-have-a-backing-app-object"></a>Verfügen verwaltete Identitäten über ein unterstützendes App-Objekt?

Nein. Verwaltete Identitäten und Azure AD-App-Registrierungen sind im Verzeichnis nicht identisch.

App-Registrierungen setzen sich aus zwei Komponenten zusammen: Ein Anwendungsobjekt und ein Dienstprinzipalobjekt.
Verwaltete Identitäten für Azure-Ressourcen verfügen nur über eine dieser Komponenten: Ein Dienstprinzipalobjekt.

Verwaltete Identitäten haben kein Anwendungsobjekt im Verzeichnis, womit im Allgemeinen App-Berechtigungen für MS Graph erteilt werden. Stattdessen müssen dem Dienstprinzipal MS Graph-Berechtigungen für verwaltete Identitäten direkt erteilt werden.

### <a name="what-is-the-credential-associated-with-a-managed-identity-how-long-is-it-valid-and-how-often-is-it-rotated"></a>Welche Anmeldeinformationen sind mit einer verwalteten Identität verknüpft? Wie lange sind sie gültig, und wie oft werden sie rotiert?

> [!NOTE]
> Die Authentifizierung von verwalteten Identitäten ist ein internes Implementierungsdetail, das ohne vorherige Ankündigung geändert werden kann.

Verwaltete Identitäten verwenden die zertifikatbasierte Authentifizierung. Die Anmeldeinformationen für jede verwaltete Identität haben eine Gültigkeitsdauer von 90 Tagen und werden nach 45 Tagen rotiert.

### <a name="what-identity-will-imds-default-to-if-dont-specify-the-identity-in-the-request"></a>Wie lautet die Identität, die IMDS standardmäßig verwendet, wenn die Identität nicht in der Anforderung angegeben ist?

- Wenn die vom System zugewiesene verwaltete Identität aktiviert und in der Anforderung keine Identität angegeben ist, verwendet IMDS standardmäßig die vom System zugewiesene verwaltete Identität.
- Wenn die vom System zugewiesene verwaltete Identität nicht aktiviert und nur eine vom Benutzer zugewiesene verwaltete Identität vorhanden ist, verwendet IMDS standardmäßig diese einzelne, vom Benutzer zugewiesene verwaltete Identität.
- Wenn die vom System zugewiesene verwaltete Identität nicht aktiviert ist und mehrere vom Benutzer zugewiesene verwaltete Identitäten vorhanden sind, müssen Sie in der Anforderung eine verwaltete Identität angeben.

## <a name="limitations"></a>Einschränkungen

### <a name="can-the-same-managed-identity-be-used-across-multiple-regions"></a>Kann dieselbe verwaltete Identität in mehreren Regionen verwendet werden?

Kurz gesagt, ja. Sie können vom Benutzer zugewiesene verwaltete Identitäten in mehr als einer Azure-Region verwenden. Etwas ausführlicher: Während vom Benutzer zugewiesene verwaltete Identitäten als regionale Ressourcen erstellt werden, ist der zugehörige [Dienstprinzipal](../develop/app-objects-and-service-principals.md#service-principal-object), der in Azure AD erstellt wird, global verfügbar. Das Dienstprinzipal kann in jeder Azure-Region verwendet werden. Seine Verfügbarkeit ist von der von Azure AD abhängig. Wenn Sie z. B. eine vom Benutzer zugewiesene verwaltete Identität in der Region „USA, Süden-Mitte“ erstellt haben und diese Region nicht mehr verfügbar ist, wirkt sich dieses Problem nur auf Aktivitäten auf [Steuerungsebene](../../azure-resource-manager/management/control-plane-and-data-plane.md) für die verwaltete Identität selbst aus.  Die Aktivitäten von Ressourcen, die bereits für die Nutzung der verwalteten Identitäten konfiguriert sind, werden dadurch nicht beeinträchtigt.

### <a name="does-managed-identities-for-azure-resources-work-with-azure-cloud-services"></a>Können verwaltete Identitäten für Azure-Ressourcen mit Azure Cloud Services verwendet werden?

Nein. Es gibt bisher keine Pläne zur Unterstützung von verwalteten Identitäten für Azure-Ressourcen in Azure Cloud Services.


### <a name="what-is-the-security-boundary-of-managed-identities-for-azure-resources"></a>Welche Sicherheitsgrenze gilt für verwaltete Identitäten für Azure-Ressourcen?

Bei der Sicherheitsgrenze der Identität handelt es sich um die Ressource, an die sie angefügt ist. Beispielsweise ist die Sicherheitsgrenze für einen virtuellen Computer, für den verwaltete Identitäten für Azure-Ressourcen aktiviert sind, dieser virtuelle Computer. Jeglicher Code, der auf diesem virtuellen Computer ausgeführt wird, kann den Endpunkt für die verwalteten Identitäten für Azure-Ressourcen aufrufen und Token anfordern. Ähnlich verhält es sich mit anderen Ressourcen, die verwaltete Identitäten für Azure-Ressourcen unterstützen.

### <a name="will-managed-identities-be-recreated-automatically-if-i-move-a-subscription-to-another-directory"></a>Werden verwaltete Identitäten automatisch neu erstellt, wenn ich ein Abonnement in ein anderes Verzeichnis verschiebe?

Nein. Wenn Sie ein Abonnement in ein anderes Verzeichnis verschieben, müssen Sie die Identitäten manuell neu erstellen und erneut Rollenzuweisungen in Azure erteilen.
- Für vom System zugewiesene verwaltete Identitäten: Deaktivieren Sie die Identitäten, und aktivieren Sie sie erneut. 
- Für vom Benutzer zugewiesene verwaltete Identitäten: Löschen Sie die Identitäten, erstellen Sie sie neu, und fügen Sie sie erneut an die erforderlichen Ressourcen an (z. B. virtuelle Computer).

### <a name="can-i-use-a-managed-identity-to-access-a-resource-in-a-different-directorytenant"></a>Kann ich eine verwaltete Identität verwenden, um auf eine Ressource in einem anderen Verzeichnis/Mandanten zuzugreifen?

Nein. Verwaltete Identitäten unterstützen derzeit keine verzeichnisübergreifenden Szenarien. 

### <a name="are-there-any-rate-limits-that-apply-to-managed-identities"></a>Gibt es Ratenbegrenzungen, die für verwaltete Identitäten gelten?

Begrenzungen für verwaltete Identitäten hängen von Azure-Diensteinschränkungen, von IMDS-Grenzwerten (Azure Instance Metadata Service) und von Azure Active Directory-Diensteinschränkungen ab.

- **Azure-Diensteinschränkungen**: Sie definieren die Anzahl der Erstellungsvorgänge, die auf Mandanten- und Abonnementebene ausgeführt werden können. Für vom Benutzer zugewiesene verwaltete Identitäten gibt es außerdem [Einschränkungen](../../azure-resource-manager/management/azure-subscription-service-limits.md#managed-identity-limits) hinsichtlich der Namensgebung.
- **IMDS**: Im Allgemeinen sind Anforderungen an IMDS auf fünf (5) Anforderungen pro Sekunde beschränkt. Anforderungen, die diesen Schwellenwert überschreiten, werden mit der Antwort 429 abgelehnt. Anforderungen an die Kategorie Verwaltete Identität sind auf 20 Anforderungen pro Sekunde und 5 gleichzeitige Anforderungen beschränkt. Weitere Informationen finden Sie im Artikel [Verwaltete Identität](../../virtual-machines/windows/instance-metadata-service.md?tabs=windows#managed-identity).
- **Azure Active Directory-Dienst**: Jede verwaltete Identität wird auf den Objektkontingentgrenzwert in einem Azure AD-Mandanten angerechnet, wie unter [Dienst- und andere Einschränkungen für Azure AD](../enterprise-users/directory-service-limits-restrictions.md) beschrieben.


### <a name="is-it-possible-to-move-a-user-assigned-managed-identity-to-a-different-resource-groupsubscription"></a>Ist es möglich, eine benutzerseitig zugewiesene verwaltete Identität in eine andere Ressourcengruppe/ein anderes Abonnement zu verschieben?

Das Verschieben einer benutzerseitig zugewiesenen verwalteten Identität in eine andere Ressourcengruppe wird nicht unterstützt.

## <a name="next-steps"></a>Nächste Schritte

- Erfahren Sie mehr über die [Funktionsweise von verwalteten Identitäten mit virtuellen Computern](how-managed-identities-work-vm.md)
