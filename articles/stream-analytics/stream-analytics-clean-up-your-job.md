---
title: Bereinigen Ihres Azure Stream Analytics-Auftrags
description: Dieser Artikel macht Sie mit verschiedenen Methoden zum Löschen Ihrer Azure Stream Analytics-Aufträge bekannt.
author: enkrumah
ms.author: ebnkruma
ms.service: stream-analytics
ms.topic: how-to
ms.date: 06/21/2019
ms.custom: seodec18
ms.openlocfilehash: 31812ac805bd3465b1b735842b45adb372286d66
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/29/2021
ms.locfileid: "98016064"
---
# <a name="stop-or-delete-your-azure-stream-analytics-job"></a>Beenden oder Löschen Ihres Azure Stream Analytics-Auftrags

Azure Stream Analytics-Aufträge können auf einfache Weise über das Azure-Portal, Azure PowerShell, das Azure SDK für .NET oder die REST-API beendet oder gelöscht werden. Ein Stream Analytics-Auftrag kann nicht wiederhergestellt werden, nachdem er gelöscht wurde.

>[!NOTE] 
>Wenn Sie Ihren Stream Analytics-Auftrag beenden, werden die Daten nur im Eingabe- und Ausgabespeicher beibehalten, z.B. in Event Hubs oder Azure SQL-Datenbank. Wenn Sie die Daten aus Azure entfernen müssen, führen Sie das folgende Verfahren zum Entfernen der Eingabe- und Ausgaberessourcen Ihres Stream Analytics-Auftrags aus.

## <a name="stop-a-job-in-azure-portal"></a>Beenden eines Auftrags im Azure-Portal

Wenn Sie einen Auftrag beenden, wird die Bereitstellung von Ressourcen aufgehoben, und die Verarbeitung von Ereignissen wird beendet. Außerdem fallen keine weiteren Gebühren mehr für diesen Auftrag an. Trotzdem bleiben all Ihre Konfiguration erhalten, und Sie können den Auftrag später neu starten. 

1. Melden Sie sich beim [Azure-Portal](https://portal.azure.com) an. 

2. Suchen Sie nach Ihrem ausgeführten Stream Analytics-Auftrag, und wählen Sie ihn aus.

3. Klicken Sie auf der Seite für Stream Analytics-Aufträge auf **Beenden**, um den Auftrag zu beenden. 

   ![Beenden eines Azure Stream Analytics-Auftrags](./media/stream-analytics-clean-up-your-job/stop-stream-analytics-job.png)


## <a name="delete-a-job-in-azure-portal"></a>Löschen eines Auftrags im Azure-Portal

>[!WARNING] 
>Ein Stream Analytics-Auftrag kann nicht wiederhergestellt werden, nachdem er gelöscht wurde.

1. Melden Sie sich beim Azure-Portal an. 

2. Suchen Sie Ihren vorhandenen Stream Analytics-Auftrag, und wählen Sie ihn aus.

3. Klicken Sie auf der Seite für Stream Analytics-Aufträge auf **Löschen**, um den Auftrag zu löschen. 

   ![Löschen eines Azure Stream Analytics-Auftrags](./media/stream-analytics-clean-up-your-job/delete-stream-analytics-job.png)


## <a name="stop-or-delete-a-job-using-powershell"></a>Beenden oder Löschen eines Auftrags mit PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Um einen Auftrag mithilfe von PowerShell zu beenden, verwenden Sie das Cmdlet [Stop-AzStreamAnalyticsJob](/powershell/module/az.streamanalytics/stop-azstreamanalyticsjob). Um einen Auftrag mithilfe von PowerShell zu löschen, verwenden Sie das Cmdlet [Remove-AzStreamAnalyticsJob](/powershell/module/az.streamanalytics/Remove-azStreamAnalyticsJob).

## <a name="stop-or-delete-a-job-using-azure-sdk-for-net"></a>Beenden oder Löschen eines Auftrags mit dem Azure SDK für .NET

Um einen Auftrag mit dem Azure SDK für .NET zu beenden, verwenden Sie die Methode [StreamingJobsOperationsExtensions.BeginStop](/dotnet/api/microsoft.azure.management.streamanalytics.streamingjobsoperationsextensions.beginstop). Um einen Auftrag mit dem Azure SDK für .NET zu löschen, verwenden Sie die Methode [StreamingJobsOperationsExtensions.BeginDelete](/dotnet/api/microsoft.azure.management.streamanalytics.streamingjobsoperationsextensions.begindelete).

## <a name="stop-or-delete-a-job-using-rest-api"></a>Beenden oder Löschen eines Auftrags mit der REST-API

Um einen Auftrag mithilfe der REST-API zu beenden, verwenden Sie die Methode [Stop](/rest/api/streamanalytics/2016-03-01/streamingjobs/stop). Um einen Auftrag mithilfe der REST-API zu löschen, verwenden Sie die Methode [Delete](/rest/api/streamanalytics/2016-03-01/streamingjobs/delete).