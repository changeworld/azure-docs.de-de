---
title: Entfernen von VM-Erweiterungen über das Azure-Portal
description: In diesem Artikel wird beschrieben, wie Sie über das Azure-Portal VM-Erweiterungen auf Azure Arc-fähigen Servern bereitstellen, die in Hybrid Cloud-Umgebungen ausgeführt werden.
ms.date: 04/13/2021
ms.topic: conceptual
ms.openlocfilehash: b5b4ff79d68ec9ff0cc61b9dbb7d3c5d7fe93598
ms.sourcegitcommit: aa00fecfa3ad1c26ab6f5502163a3246cfb99ec3
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/14/2021
ms.locfileid: "107388277"
---
# <a name="enable-azure-vm-extensions-from-the-azure-portal"></a>Aktivieren von Azure-VM-Erweiterungen über das Azure-Portal

In diesem Artikel erfahren Sie, wie Sie Azure-VM-Erweiterungen, die von Azure Arc-fähigen Servern unterstützt werden, auf einem Linux- oder Windows-Hybridcomputer unter Verwendung des Azure-Portals bereitstellen oder deinstallieren.

> [!NOTE]
> Die Key Vault-VM-Erweiterung (Vorschau) unterstützt keine Bereitstellung über das Azure-Portal, sondern lediglich über die Azure-Befehlszeilenschnittstelle, Azure PowerShell oder eine Azure Resource Manager-Vorlage.

> [!NOTE]
> Das Bereitstellen und Verwalten von VM-Erweiterungen auf Azure-VMs wird auf Servern mit Azure Arc-Unterstützung nicht unterstützt. Informationen zu Azure-VMs finden Sie im Artikel [Erweiterungen und Features für virtuelle Azure-Computer](../../virtual-machines/extensions/overview.md).

## <a name="enable-extensions-from-the-portal"></a>Aktivieren von Erweiterungen über das Portal

VM-Erweiterungen können über das Azure-Portal auf Ihren mit Arc für Server verwalteten Computer angewendet werden.

1. Navigieren Sie in Ihrem Browser zum [Azure-Portal](https://portal.azure.com).

2. Navigieren Sie im Portal zu **Server - Azure Arc**, und wählen Sie in der Liste Ihren Hybridcomputer aus.

3. Wählen Sie **Erweiterungen** und dann **Hinzufügen** aus. Wählen Sie die Erweiterung aus, die Sie aus der Liste verfügbarer Erweiterungen erhalten, und befolgen Sie die Anweisungen im Assistenten. In diesem Beispiel stellen wir die Log Analytics-VM-Erweiterung bereit.

    ![Auswählen der VM-Erweiterung für den ausgewählten Computer](./media/manage-vm-extensions/add-vm-extensions.png)

    Im folgenden Beispiel wird die Installation der Log Analytics-VM-Erweiterung über das Azure-Portal gezeigt:

    ![Installieren der Log Analytics VM-Erweiterung](./media/manage-vm-extensions/mma-extension-config.png)

    Um die Installation abzuschließen, müssen Sie die Arbeitsbereichs-ID und den Primärschlüssel angeben. Wenn Sie mit dem Ermitteln dieser Informationen nicht vertraut sind, lesen Sie [Abrufen von Arbeitsbereichs-ID und Schlüssel](../../azure-monitor/agents/log-analytics-agent.md#workspace-id-and-key).

4. Nach dem Bestätigen der bereitgestellten erforderlichen Informationen wählen Sie **Erstellen** aus. Es wird eine Zusammenfassung der Bereitstellung angezeigt, und Sie können den Status der Bereitstellung überprüfen.

>[!NOTE]
>Zwar können mehrere Erweiterungen in einem Batch zusammengefasst und verarbeitet werden, die Installation erfolgt jedoch nacheinander. Sobald die Installation der ersten Erweiterung abgeschlossen ist, wird die Installation der nächsten Erweiterung versucht.

## <a name="list-extensions-installed"></a>Auflisten der Installierten Erweiterungen

Sie können eine Liste der VM-Erweiterungen auf dem Arc-fähigen Server im Azure-Portal abrufen. Führen Sie zu Ihrer Anzeige die folgenden Schritte aus.

1. Navigieren Sie in Ihrem Browser zum [Azure-Portal](https://portal.azure.com).

2. Navigieren Sie im Portal zu **Server - Azure Arc**, und wählen Sie in der Liste Ihren Hybridcomputer aus.

3. Wählen Sie **Erweiterungen** aus, und die Liste der installierten Erweiterungen wird zurückgegeben.

    ![Auflisten der auf dem ausgewählten Computer bereitgestellten VM-Erweiterungen](./media/manage-vm-extensions/list-vm-extensions.png)

## <a name="uninstall-extension"></a>Deinstallieren der Erweiterung

Sie können über das Azure-Portal Erweiterungen von einem Arc-fähigen Server entfernen. Führen Sie dann die folgenden Schritte aus, um eine Erweiterung zu entfernen.

1. Navigieren Sie in Ihrem Browser zum [Azure-Portal](https://portal.azure.com).

2. Navigieren Sie im Portal zu **Server - Azure Arc**, und wählen Sie in der Liste Ihren Hybridcomputer aus.

3. Wählen Sie **Erweiterungen** aus, und wählen Sie dann eine Erweiterung aus der Liste der installierten Erweiterungen aus.

4. Wählen Sie **Deinstallieren** aus. Wenn Sie zur Bestätigung aufgefordert werden, wählen Sie **Ja** aus, um fortzufahren.

## <a name="next-steps"></a>Nächste Schritte

- Sie können VM-Erweiterungen über die [Azure-Befehlszeilenschnittstelle](manage-vm-extensions-cli.md), [PowerShell](manage-vm-extensions-powershell.md) oder [Azure Resource Manager-Vorlagen](manage-vm-extensions-template.md) bereitstellen, verwalten und entfernen.

- Informationen zur Problembehandlung finden Sie im [Problembehandlungs-Handbuch für VM-Erweiterungen](troubleshoot-vm-extensions.md).