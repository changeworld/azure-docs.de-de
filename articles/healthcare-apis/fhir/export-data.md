---
title: Ausführen des Exports durch Aufrufen des Befehls „$export“ in Azure API for FHIR
description: In diesem Artikel wird beschrieben, wie FHIR-Daten mithilfe von $export exportiert werden.
author: caitlinv39
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 5/25/2021
ms.author: cavoeg
ms.openlocfilehash: 30e9b5cf5f296ac161301f27c82ac6a2f98f4611
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 06/10/2021
ms.locfileid: "111970161"
---
# <a name="how-to-export-fhir-data"></a>Exportieren von FHIR-Daten


Mit der Funktion für Massenexport können Daten vom FHIR-Server gemäß der [FHIR-Spezifikation](https://hl7.org/fhir/uv/bulkdata/export/index.html) exportiert werden. 

Bevor Sie $export verwenden, sollten Sie sicherstellen, dass die Azure API for FHIR für die Verwendung konfiguriert ist. Informationen zum Konfigurieren von Exporteinstellungen und zum Erstellen eines Azure Storage-Kontos finden Sie auf der [Seite zum Konfigurieren von Exportdaten](configure-export-data.md).

## <a name="using-export-command"></a>Verwenden $export-Befehls

Nachdem Sie Azure API for FHIR für den Export konfiguriert haben, können Sie den $export-Befehl verwenden, um die Daten aus dem Dienst zu exportieren. Die Daten werden in dem Speicherkonto gespeichert, das Sie beim Konfigurieren des Exports angegeben haben. Informationen zum Aufrufen des Befehls $export auf dem FHIR-Server finden Sie in der Dokumentation zur [$export-Spezifikation von HL7 FHIR](https://hl7.org/Fhir/uv/bulkdata/export/index.html).


**Aufträge hängen in einem fehlerhaften Zustand**

In einigen Situationen besteht die Möglichkeit, dass ein Auftrag in einem fehlerhaften Zustand hängen bleibt. Dies kann insbesondere auftreten, wenn die Speicherkontoberechtigungen nicht ordnungsgemäß eingerichtet wurden. Eine Möglichkeit, zu überprüfen, ob der Export erfolgreich ist, besteht in der Überprüfung Ihres Speicherkontos, um zu überprüfen, ob die entsprechenden Containerdateien (d. h. ndjson) vorhanden sind. Wenn sie nicht vorhanden sind und keine anderen Exportaufträge ausgeführt werden, besteht die Möglichkeit, dass der aktuelle Auftrag in einem fehlerhaften Zustand hängen bleibt. Sie sollten den Exportauftrag abbrechen, indem Sie eine Abbruchanforderung senden und versuchen, den Auftrag erneut in die Warteschlange zu stellen. Die Standard-Laufzeit für einen Export in einem fehlerhaften Zustand beträgt 10 Minuten, bevor er anzuhalten und zu einem neuen Auftrag wechseln oder den Export wiederholen wird. 

Die Azure API for FHIR unterstützt $export auf folgenden Ebenen:
* [System:](https://hl7.org/Fhir/uv/bulkdata/export/index.html#endpoint---system-level-export) `GET https://<<FHIR service base URL>>/$export>>`
* [Patient:](https://hl7.org/Fhir/uv/bulkdata/export/index.html#endpoint---all-patients) `GET https://<<FHIR service base URL>>/Patient/$export>>`
* [Gruppe von Patienten*](https://hl7.org/Fhir/uv/bulkdata/export/index.html#endpoint---group-of-patients) – Azure API for FHIR alle zugehörigen Ressourcen exportiert, aber nicht die Merkmale der Gruppe: `GET https://<<FHIR service base URL>>/Group/[ID]/$export>>`

Beim Exportieren von Daten wird eine separate Datei für jeden Ressourcentyp erstellt. Um sicherzustellen, dass die exportierten Dateien nicht zu groß werden. Wir erstellen eine neue Datei, nachdem die Größe einer einzelnen exportierten Datei größer als 64 MB ist. Das Ergebnis ist, dass Sie möglicherweise mehrere Dateien für jeden Ressourcentyp erhalten, der aufzählt wird (d.h. Patient-1.ndjson, Patient-2.ndjson). 


> [!Note] 
> `Patient/$export` und `Group/[ID]/$export` exportieren möglicherweise doppelt vorhandene Ressourcen, wenn sich die Ressource in einem Depot mehrerer Ressourcen befindet oder in mehreren Gruppen vorhanden ist.

Darüber hinaus wird die Überprüfung des Exportstatus über die URL, die von der Standortkopfzeile während der Warteschlangendauer zurückgegeben wird, sowie das Abbrechen des eigentlichen Exportauftrags unterstützt.

### <a name="exporting-fhir-data-to-adls-gen2"></a>Exportieren von FHIR-Daten nach ADLS Gen2

Derzeit wird „$export“ für ADLS Gen2-fähige Speicherkonten unterstützt. Hierbei gilt aber die folgende Einschränkung:

- Der Benutzer kann keine hierarchischen Namespaces nutzen, aber es gibt keine Möglichkeit, den Export in ein [bestimmtes](../../storage/blobs/data-lake-storage-namespace.md)Unterverzeichnis innerhalb des Containers als Ziel zu verwenden. Es kann nur ein bestimmter Container angegeben werden (wobei für jeden Export ein neuer Ordner erstellt wird).
- Nach Abschluss eines Exportvorgangs werden nicht noch einmal Daten in diesen Ordner exportiert, da für die nachfolgenden Exporte in denselben Container dann ein neu erstellter Ordner genutzt wird.


## <a name="settings-and-parameters"></a>Einstellungen und Parameter

### <a name="headers"></a>Header
Es gibt zwei erforderliche Headerparameter, die für $export-Aufträge festgelegt werden müssen. Die Werte werden durch die aktuelle [$export-Spezifikation](https://hl7.org/Fhir/uv/bulkdata/export/index.html#headers) definiert.
* **Accept:** application/fhir+json
* **Prefer:** respond-async

### <a name="query-parameters"></a>Abfrageparameter
Die Azure API for FHIR unterstützt die folgenden Abfrageparameter, die alle optional sind:

|Query parameter (Abfrageparameter)        | Definiert durch die FHIR-Spezifikation?    |  Beschreibung|
|------------------------|---|------------|
| \_outputFormat | Ja | Unterstützt derzeit drei Werte gemäß FHIR-Spezifikation: „application/fhir+ndjson“, „application/ndjson“ oder nur „ndjson“. Alle Exportaufträge geben `ndjson` zurück, und der übergebene Wert hat keine Auswirkung auf das Codeverhalten. |
| \_since | Ja | Ermöglicht nur das Exportieren von Ressourcen, die seit dem angegebenen Zeitpunkt geändert wurden |
| \_type | Ja | Ermöglicht die Angabe, welche Ressourcentypen eingeschlossen werden sollen. Beispielsweise gibt „\_type=Patient“ nur Patientenressourcen zurück.|
| \_typeFilter | Ja | Um eine differenziertere Filterung anzufordern, können Sie „\_typeFilter“ zusammen mit dem Parameter „\_type“ verwenden. Der Wert des Parameters „_typeFilter“ ist eine durch Trennzeichen getrennte Liste von FHIR-Abfragen, mit denen die Ergebnisse weiter eingegrenzt werden. |
| \_container | Nein |  Gibt den Container innerhalb des konfigurierten Speicherkontos an, in den die Daten exportiert werden sollen. Wenn ein Container angegeben wird, werden die Daten in einen Ordner in diesen Container exportiert. Wenn der Container nicht angegeben wird, werden die Daten in einen neuen Container exportiert. |

> [!Note]
> Nur Speicherkonten im gleichen Abonnement wie für Azure API for FHIR können als Ziel für $export registriert werden.

## <a name="secure-export-to-azure-storage"></a>Sicherer Export in Azure Storage

Für Azure API for FHIR wird ein sicherer Exportvorgang unterstützt. Wählen Sie eine der beiden folgenden Optionen aus:

* Zulassen Azure API for FHIR als vertrauenswürdiger Microsoft-Dienst für den Zugriff auf das Azure-Speicherkonto.
 
* Zulassen, dass bestimmte IP-Adressen, die Azure API for FHIR, auf das Azure-Speicherkonto zugreifen können. Diese Option bietet zwei unterschiedliche Konfigurationen, je nachdem, ob sich das Speicherkonto am gleichen Standort wie oder an einem anderen Speicherort als das der Azure API for FHIR.

### <a name="allowing-azure-api-for-fhir-as-a-microsoft-trusted-service"></a>Zulassen Azure API for FHIR als vertrauenswürdiger Microsoft-Dienst

Wählen Sie in der Azure-Portal ein Speicherkonto aus, und wählen Sie dann das **Blatt Netzwerk** aus. Wählen **Sie auf der Registerkarte** **Firewalls und virtuelle Netzwerke** die Option Ausgewählte Netzwerke aus.

> [!IMPORTANT]
> Stellen Sie sicher, dass Sie dem Speicherkonto mithilfe der verwalteten Identität Azure API for FHIR Zugriffsberechtigung erteilt haben. Weitere Informationen finden Sie unter [Konfigurieren der Exporteinstellung und Einrichten des Speicherkontos.](./configure-export-data.md)

  :::image type="content" source="media/export-data/storage-networking.png" alt-text="Netzwerkeinstellungen für Azure-Speicher" lightbox="media/export-data/storage-networking.png":::

Aktivieren Sie **im Abschnitt** Ausnahmen das Kontrollkästchen Vertrauenswürdigen Benutzern Microsoft-Dienste Zugriff **auf** dieses Speicherkonto erlauben, und speichern Sie die Einstellung. 

:::image type="content" source="media/export-data/exceptions.png" alt-text="Vertrauenswürdigen Microsoft-Dienste Zugriff auf dieses Speicherkonto erlauben.":::

Sie können jetzt FHIR-Daten sicher in das Speicherkonto exportieren. Beachten Sie, dass sich das Speicherkonto in ausgewählten Netzwerken befindet und nicht öffentlich zugänglich ist. Für den Zugriff auf die Dateien können Sie entweder private Endpunkte für das Speicherkonto aktivieren und verwenden oder für kurze Zeit alle Netzwerke für das Speicherkonto aktivieren.

> [!IMPORTANT]
> Die Benutzeroberfläche wird später aktualisiert, damit Sie den Ressourcentyp für Azure API for FHIR eine bestimmte Dienstinstanz auswählen können.

### <a name="allowing-specific-ip-addresses-for-the-azure-storage-account-in-a-different-region"></a>Zulassen bestimmter IP-Adressen für das Azure-Speicherkonto in einer anderen Region

Wählen **Sie im** Portal Netzwerk des Azure-Speicherkontos aus. 
   
Klicken Sie auf **Ausgewählte Netzwerke**. Geben Sie im Abschnitt Firewall die IP-Adresse im Feld **Adressbereich** an. Fügen Sie IP-Adressbereiche hinzu, um den Zugriff über das Internet oder Ihre lokalen Netzwerke zu ermöglichen. Sie finden die IP-Adresse in der folgenden Tabelle für die Azure-Region, in der der Azure API for FHIR bereitgestellt wird.

|**Azure-Region**         |**Öffentliche IP-Adresse** |
|:----------------------|:-------------------|
| Australien (Osten)       | 20.53.44.80       |
| Kanada, Mitte       | 20.48.192.84      |
| USA (Mitte)           | 52.182.208.31     |
| East US              | 20.62.128.148     |
| USA (Ost) 2            | 20.49.102.228     |
| USA, Osten 2 (EUAP)       | 20.39.26.254      |
| Deutschland, Norden        | 51.116.51.33      |
| Deutschland, Westen-Mitte | 51.116.146.216    |
| Japan, Osten           | 20.191.160.26     |
| Korea, Mitte        | 20.41.69.51       |
| USA Nord Mitte     | 20.49.114.188     |
| Nordeuropa         | 52.146.131.52     |
| Südafrika, Norden   | 102.133.220.197   |
| USA Süd Mitte     | 13.73.254.220     |
| Asien, Südosten       | 23.98.108.42      |
| Schweiz, Norden    | 51.107.60.95      |
| UK, Süden             | 51.104.30.170     |
| UK, Westen              | 51.137.164.94     |
| USA, Westen-Mitte      | 52.150.156.44     |
| Europa, Westen          | 20.61.98.66       |
| USA, Westen 2            | 40.64.135.77      |

> [!NOTE]
> Die obigen Schritte ähneln den Konfigurationsschritten, die im Dokument Konvertieren von Daten in FHIR (Vorschauversion) beschrieben sind. Weitere Informationen finden Sie unter [Hosten und Verwenden von Vorlagen.](./convert-data.md#host-and-use-templates)

### <a name="allowing-specific-ip-addresses-for-the-azure-storage-account-in-the-same-region"></a>Zulassen bestimmter IP-Adressen für das Azure-Speicherkonto in derselben Region

Der Konfigurationsprozess entspricht dem obigen Prozess, aber mit der Ausnahme, dass ein bestimmter IP-Adressbereich im CIDR-Format verwendet wird: 100.64.0.0/10. Der IP-Adressbereich (100.64.0.0 bis 100.127.255.255) muss angegeben werden, da die tatsächlich vom Dienst genutzte IP-Adresse variiert. Sie liegt für jede $export-Anforderung aber immer in diesem Bereich.

> [!Note] 
> Es ist möglich, dass stattdessen eine private IP-Adresse im Bereich 10.0.2.0/24 genutzt wird. In diesem Fall ist der $export Vorgang nicht erfolgreich. Sie können die $export Anforderung wiederholen, aber es gibt keine Garantie, dass eine IP-Adresse im Bereich von 100.64.0.0/10 beim nächsten Mal verwendet wird. Dies ist das bekannte standardmäßige Netzwerkverhalten. Die Alternative besteht darin, das Speicherkonto in einer anderen Region zu konfigurieren.
    
## <a name="next-steps"></a>Nächste Schritte

In diesem Artikel haben Sie erfahren, wie Sie FHIR-Ressourcen mit $export Befehl exportieren. Weitere Informationen zum Exportieren von nicht identifizierten Daten finden Sie unter:
 
>[!div class="nextstepaction"]
>[Exportieren anonymisierter Daten](de-identified-export.md)