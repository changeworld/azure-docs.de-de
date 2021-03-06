---
title: Bereitstellen von Azure Monitor für SAP-Lösungen mit dem Azure-Portal
description: Bereitstellen von Azure Monitor für SAP-Lösungen mit dem Azure-Portal
author: sameeksha91
ms.author: sakhare
ms.topic: how-to
ms.service: virtual-machines-sap
ms.date: 08/17/2020
ms.openlocfilehash: a9208101777cd88f0237e661a414550759a069b0
ms.sourcegitcommit: aaba99b8b1c545ad5d19f400bcc2d30d59c63f39
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/26/2021
ms.locfileid: "108007379"
---
# <a name="deploy-azure-monitor-for-sap-solutions-with-azure-portal"></a>Bereitstellen von Azure Monitor für SAP-Lösungen mit dem Azure-Portal

Ressourcen für Azure Monitor für SAP-Lösungen können über das [Azure-Portal](https://azure.microsoft.com/features/azure-portal) erstellt werden. Diese Methode bietet eine browserbasierte Benutzeroberfläche zum Bereitstellen von Azure Monitor für SAP-Lösungen und zum Konfigurieren von Anbietern.

## <a name="sign-in-to-azure-portal"></a>Anmelden beim Azure-Portal

Melden Sie sich unter https://portal.azure.com beim Azure-Portal an.

## <a name="create-monitoring-resource"></a>Erstellen einer Überwachungsressource

1. Wählen Sie im **Azure Marketplace** das Angebot **Azure Monitor für SAP-Lösungen** aus.

   :::image type="content" source="./media/azure-monitor-sap/azure-monitor-quickstart-1.png" alt-text="Abbildung zum Auswählen des Angebots „Azure Monitor für SAP-Lösungen“ im Azure Marketplace" lightbox="./media/azure-monitor-sap/azure-monitor-quickstart-1.png":::

2. Geben Sie auf der Registerkarte **Grundlagen** die erforderlichen Werte ein. Gegebenenfalls können Sie einen vorhandenen Log Analytics-Arbeitsbereich verwenden.

   :::image type="content" source="./media/azure-monitor-sap/azure-monitor-quickstart-2.png" alt-text="Anzeige der Konfigurationsoptionen im Azure-Portal" lightbox="./media/azure-monitor-sap/azure-monitor-quickstart-2.png":::

3. Stellen Sie bei Auswahl eines virtuellen Netzwerks sicher, dass die Systeme, die Sie überwachen möchten, innerhalb des VNET erreichbar sind. 

   > [!IMPORTANT]
   > Wenn Sie **Freigeben** zur Datenfreigabe für Microsoft auswählen, können unsere Supportteams zusätzlichen Support bereitstellen.

## <a name="configure-providers"></a>Konfigurieren von Anbietern

### <a name="sap-netweaver-provider"></a>Hinzufügen von SAP NetWeaver als Anbieter

#### <a name="prerequisites-for-adding-netweaver-provider"></a>Voraussetzungen für das Hinzufügen von NetWeaver als Anbieter

Der SAP-Startdienst („sapstartsrv“) stellt eine Vielzahl von Diensten einschließlich der Überwachung des SAP-Systems bereit. Wir nutzen „SAPControl“, eine SOAP-Webdienstschnittstelle, die diese Funktionen verfügbar macht. Diese SAPControl-Webdienstschnittstelle unterscheidet zwischen [geschützten und ungeschützten](https://wiki.scn.sap.com/wiki/display/SI/Protected+web+methods+of+sapstartsrv) Webdienstmethoden. Um bestimmte Metriken abrufen zu können, müssen Sie den Schutz einiger Methoden aufheben. Führen Sie die folgenden Schritte für jedes SAP-System aus, um den Schutz der erforderlichen Methoden für das aktuelle Release aufzuheben:

1. Öffnen Sie eine SAP GUI-Verbindung mit dem SAP-Server.
2. Melden Sie sich mit einem Administratorkonto an.
3. Führen Sie die Transaktion RZ10 aus.
4. Wählen Sie das entsprechende Profil (DEFAULT.PFL) aus.
5. Wählen Sie „Erweiterte Wartung“ aus, und klicken Sie auf „Ändern“. 
6. Ändern Sie den Wert des betroffenen Parameters „service/protectedwebmethods“ für „SDEFAULT -GetQueueStatistic –ABAPGetWPTable –EnqGetStatistic –GetProcessList“ in die empfohlene Einstellung, und klicken Sie dann auf „Kopieren“.
7. Navigieren Sie zurück, und wählen Sie „Profil > Speichern“ aus.
8. Starten Sie das System neu, damit die Parametereinstellung wirksam wird.

>[!Tip]
> Verwenden Sie eine Access Control List (ACL), um den Zugriff auf einen Serverport zu filtern. Siehe den [SAP-Hinweis](https://launchpad.support.sap.com/#/notes/1495075)

#### <a name="installing-netweaver-provider-on-the-azure-portal"></a>Installieren des NetWeaver-Anbieters über das Azure-Portal
1.  Stellen Sie sicher, dass Sie alle erforderlichen Schritte ausgeführt haben und der Server neu gestartet wurde.
2.  Wählen Sie im Azure-Portal unter „AMS“ die Option „Anbieter hinzufügen“ aus, und wählen Sie dann im Dropdownfeld den Eintrag „SAP NetWeaver“ aus.
3.  Geben Sie den Hostnamen des SAP-Systems und die Unterdomäne (falls zutreffend) ein.
4.  Geben Sie die Instanznummer ein, die dem eingegebenen Hostnamen entspricht. 
5.  Geben Sie die System-ID (SID) ein.
6.  Wählen Sie abschließend „Anbieter hinzufügen“ aus.
7.  Fügen Sie nach Bedarf weitere Anbieter hinzu, oder wählen Sie „Überprüfen und erstellen“ aus, um die Bereitstellung abzuschließen.

![image](https://user-images.githubusercontent.com/75772258/114583569-5c777d80-9c9f-11eb-99a2-8c60987700c2.png)

### <a name="sap-hana-provider"></a>SAP HANA-Anbieter 

1. Wählen Sie die Registerkarte **Anbieter** aus, um die Anbieter hinzuzufügen, die Sie konfigurieren möchten. Sie können nacheinander mehrere Anbieter hinzufügen oder Anbieter nach der Bereitstellung der Überwachungsressource hinzufügen. 

   :::image type="content" source="./media/azure-monitor-sap/azure-monitor-quickstart-3.png" alt-text="Anzeige der Registerkarte „Anbieter“ zum Hinzufügen weiterer Anbieter zu Azure Monitor für SAP-Lösungen" lightbox="./media/azure-monitor-sap/azure-monitor-quickstart-3.png":::

2. Wählen Sie **Anbieter hinzufügen** und dann in der Dropdownliste den Eintrag **SAP HANA** aus. 

   > [!IMPORTANT]
   > Stellen Sie sicher, dass der SAP HANA-Anbieter für den SAP HANA-Knoten „Master“ konfiguriert ist.

3. Geben Sie die private IP-Adresse für den HANA-Server ein.

4. Geben Sie den Namen des Datenbankmandanten ein, den Sie verwenden möchten. Sie können einen beliebigen Mandanten auswählen, doch wird die Verwendung von **SYSTEMDB** empfohlen, da dieser umfassendere Überwachungsbereiche ermöglicht. 

5. Geben Sie die mit Ihrer HANA-Datenbank verknüpfte SQL-Portnummer ein. Die Portnummer sollte das Format **[3]**  +  **[Instanznr.]**  +  **[13]** aufweisen. Beispiel: 30013. 

6. Geben Sie den Datenbankbenutzernamen ein, den Sie verwenden möchten. Stellen Sie sicher, dass dem Datenbankbenutzer die Rollen für **Überwachung** und **Katalog lesen** zugewiesen sind. 

7. Wählen Sie abschließend die Option **Anbieter hinzufügen** aus. Fügen Sie nach Bedarf weitere Anbieter hinzu, oder wählen Sie **Überprüfen und erstellen** aus, um die Bereitstellung abzuschließen.

   :::image type="content" source="./media/azure-monitor-sap/azure-monitor-quickstart-4.png" alt-text="Abbildung der Konfigurationsoptionen beim Hinzufügen von Anbieterinformationen" lightbox="./media/azure-monitor-sap/azure-monitor-quickstart-4.png":::
   
### <a name="microsoft-sql-server-provider"></a>Microsoft SQL Server-Anbieter

1. Vor dem Hinzufügen des Microsoft SQL Server-Anbieters sollten Sie das folgende Skript in SQL Server Management Studio ausführen, um einen Benutzer mit den erforderlichen Berechtigungen zum Konfigurieren des Anbieters zu erstellen.

   ```sql
   USE [<Database to monitor>]
   DROP USER [AMS]
   GO
   USE [master]
   DROP USER [AMS]
   DROP LOGIN [AMS]
   GO
   CREATE LOGIN [AMS] WITH PASSWORD=N'<password>', DEFAULT_DATABASE=[<Database to monitor>], DEFAULT_LANGUAGE=[us_english], CHECK_EXPIRATION=OFF, CHECK_POLICY=OFF
   CREATE USER AMS FOR LOGIN AMS
   ALTER ROLE [db_datareader] ADD MEMBER [AMS]
   ALTER ROLE [db_denydatawriter] ADD MEMBER [AMS]
   GRANT CONNECT TO AMS
   GRANT VIEW SERVER STATE TO AMS
   GRANT VIEW SERVER STATE TO AMS
   GRANT VIEW ANY DEFINITION TO AMS
   GRANT EXEC ON xp_readerrorlog TO AMS
   GO
   USE [<Database to monitor>]
   CREATE USER [AMS] FOR LOGIN [AMS]
   ALTER ROLE [db_datareader] ADD MEMBER [AMS]
   ALTER ROLE [db_denydatawriter] ADD MEMBER [AMS]
   GO
   ``` 

2. Wählen Sie **Anbieter hinzufügen** und dann in der Dropdownliste den Eintrag **Microsoft SQL Server** aus. 

3. Füllen Sie die Felder mit Informationen entsprechend Ihrer Microsoft SQL Server-Instanz aus. 

4. Wählen Sie abschließend die Option **Anbieter hinzufügen** aus. Fügen Sie nach Bedarf weitere Anbieter hinzu, oder wählen Sie **Überprüfen und erstellen** aus, um die Bereitstellung abzuschließen.

     :::image type="content" source="./media/azure-monitor-sap/azure-monitor-quickstart-6.png" alt-text="Abbildung der Informationen im Zusammenhang mit dem Hinzufügen des Microsoft SQL Server-Anbieters" lightbox="./media/azure-monitor-sap/azure-monitor-quickstart-6.png":::

### <a name="high-availability-cluster-pacemaker-provider"></a>Anbieter für Hochverfügbarkeitscluster (Pacemaker)

1. Wählen Sie in der Dropdownliste den Eintrag **Hochverfügbarkeitscluster (Pacemaker)** aus. 

   > [!IMPORTANT]
   > Stellen Sie zum Konfigurieren des Anbieters für Hochverfügbarkeitscluster (Pacemaker) sicher, dass ha_cluster_provider in jedem Knoten installiert ist. Weitere Informationen finden Sie unter [ha_cluster_exporter](https://github.com/ClusterLabs/ha_cluster_exporter#installation).

2. Geben Sie den Prometheus-Endpunkt in der Form http://IP:9664/metrics ein. 
 
3. Geben Sie die System-ID (SID), den Hostnamen und den Clusternamen ein.

4. Wählen Sie abschließend die Option **Anbieter hinzufügen** aus. Fügen Sie nach Bedarf weitere Anbieter hinzu, oder wählen Sie **Überprüfen und erstellen** aus, um die Bereitstellung abzuschließen.

   :::image type="content" source="./media/azure-monitor-sap/azure-monitor-quickstart-5.png" alt-text="Abbildung der Optionen im Zusammenhang mit dem Anbieter für Hochverfügbarkeitscluster (Pacemaker)" lightbox="./media/azure-monitor-sap/azure-monitor-quickstart-5.png":::

### <a name="os-linux-provider"></a>Betriebssystemanbieter (Linux) 

1. Wählen Sie in der Dropdownliste die Option für das Betriebssystem (Linux) aus. 

   >[!IMPORTANT]
   > Stellen Sie zum Konfigurieren des Betriebssystemanbieters (Linux) sicher, dass die aktuelle Version von „Node_Exporter“ auf jedem Host (BareMetal-Instanz oder VM) installiert ist, den Sie überwachen möchten. Installieren Sie die [neueste Version von Node Exporter](https://prometheus.io/download/#node_exporter). Klicken Sie [hier](https://github.com/prometheus/node_exporter), um weitere Informationen zu erhalten.

2. Geben Sie einen Namen ein, der der Bezeichner für die BareMetal-Instanz sein wird.
3. Geben Sie den Endpunkt für Knotenexport in Form von http://IP:9100/metrics ein.

   >[!IMPORTANT]
   >Verwenden Sie die private IP-Adresse des Linux-Hosts. Stellen Sie sicher, dass sich der Host und die AMS-Ressource im gleichen VNet befinden. 

   >[!Note]
   > Auf dem Linux-Host muss der Firewallport 9100 geöffnet sein.
   >Bei Verwendung von „firewall-cmd“: firewall-cmd --permanent --add-port=9100/tcp firewall-cmd --reload. Bei Verwendung von „ufw“: ufw allow 9100/tcp ufw reload

    >[!Tip]
    > Ist der Linux-Host ein virtueller Azure-Computer, stellen Sie sicher, dass alle anwendbaren NSGs eingehenden Datenverkehr an Port 9100 von der Quelle „VirtualNetwork“ zulassen.
 
5. Wählen Sie abschließend die Option  **Anbieter hinzufügen** aus. Fügen Sie nach Bedarf weitere Anbieter hinzu, oder wählen Sie  **Überprüfen und erstellen** aus, um die Bereitstellung abzuschließen. 


## <a name="next-steps"></a>Nächste Schritte

Weitere Informationen zu [Azure Monitor für SAP-Lösungen](azure-monitor-overview.md)
