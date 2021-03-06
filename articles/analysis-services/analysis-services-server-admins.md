---
title: Verwalten von Serveradministratoren in Azure Analysis Services | Microsoft-Dokumentation
description: In diesem Artikel wird beschrieben, wie Sie Serveradministratoren für einen Azure Analysis Services-Server mithilfe des Azure-Portals bzw. mit PowerShell- oder REST-APIs verwalten können.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 2/4/2021
ms.author: owend
ms.reviewer: minewiskan
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: c0986b8508a7e21dee7c7c87e535eb2726944de8
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/20/2021
ms.locfileid: "107769191"
---
# <a name="manage-server-administrators"></a>Verwalten von Serveradministratoren

Serveradministratoren müssen ein gültiger Benutzer, ein Dienstprinzipal oder eine Sicherheitsgruppe in Azure Active Directory (Azure AD) für den Mandanten sein, in dem sich der Server befindet. **Analysis Services-Administratoren** für Ihren Server können im Azure-Portal, in Servereigenschaften in SSMS, in PowerShell oder in der REST-API verwendet werden, um Serveradministratoren zu verwalten. 

Verwenden Sie beim Hinzufügen einer **Sicherheitsgruppe** `obj:groupid@tenantid`. Dienstprinzipale werden nicht in Sicherheitsgruppen unterstützt, die der Serveradministratorrolle hinzugefügt sind.

Weitere Informationen zum Hinzufügen eines Dienstprinzipals zur Serveradministratorrolle finden Sie unter [Hinzufügen eines Dienstprinzipals zur Serveradministratorrolle](analysis-services-addservprinc-admins.md).

Wenn die Serverfirewall aktiviert ist, müssen die IP-Adressen des Serveradministrator-Clientcomputers in eine Firewallregel eingeschlossen werden. Weitere Informationen finden Sie unter [Konfigurieren einer Serverfirewall](analysis-services-qs-firewall.md).

## <a name="to-add-server-administrators-by-using-azure-portal"></a>So fügen Sie Serveradministratoren über das Azure-Portal hinzu

1. Klicken Sie im Portal im Bereich für Ihren Server auf **Analysis Services-Administratoren**.
2. Klicken Sie unter **\<servername> – Analysis Services-Administratoren** auf **Hinzufügen**.
3. Wählen Sie unter **Serveradministratoren hinzufügen** Benutzerkonten aus Ihrem Azure AD-Verzeichnis aus, oder laden Sie externe Benutzer per E-Mail ein.

    ![Serveradministratoren im Azure-Portal](./media/analysis-services-server-admins/aas-manage-users-admins.png)

## <a name="to-add-server-administrators-by-using-ssms"></a>So fügen Sie Serveradministratoren über SSMS hinzu

1. Klicken Sie mit der rechten Maustaste auf den Server, und wählen Sie dann **Eigenschaften** aus.
2. Klicken Sie in **Eigenschaften für Analysis-Server** auf **Sicherheit**.
3. Klicken Sie auf **Hinzufügen**, und geben Sie dann die E-Mail-Adresse eines Benutzers oder einer Gruppe in Ihrem Azure AD ein.
   
    ![Hinzufügen von Serveradministratoren in SSMS](./media/analysis-services-server-admins/aas-manage-users-ssms.png)

## <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Verwenden Sie das Cmdlet [New-AzAnalysisServicesServer](/powershell/module/az.analysisservices/new-azanalysisservicesserver), um den Administrator-Parameter beim Erstellen eines neuen Servers anzugeben. <br>
Verwenden Sie das Cmdlet [Set-AzAnalysisServicesServer](/powershell/module/az.analysisservices/set-azanalysisservicesserver), um den Administrator-Parameter für einen bereits vorhandenen Server zu ändern.

## <a name="rest-api"></a>REST-API

Verwenden Sie [Create](/rest/api/analysisservices/servers/create), um die Eigenschaft „asAdministrator“ beim Erstellen eines neuen Servers anzugeben. <br>
Verwenden Sie [Update](/rest/api/analysisservices/servers/update), um die Eigenschaft „asAdministrator“ beim Ändern eines bereits vorhandenen Servers anzugeben. <br>



## <a name="next-steps"></a>Nächste Schritte 

[Authentifizierung und Benutzerberechtigungen](analysis-services-manage-users.md)  
[Verwalten von Datenbankrollen und Benutzern](analysis-services-database-users.md)  
[Rollenbasierte Zugriffssteuerung von Azure (Azure-RBAC)](../role-based-access-control/overview.md)
