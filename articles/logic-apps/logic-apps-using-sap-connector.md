---
title: Herstellen einer Verbindung mit SAP-Systemen
description: Zugreifen auf und Verwalten von SAP-Ressourcen durch das Automatisieren von Workflows mit Azure Logic Apps
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: estfan, daviburg, logicappspm
ms.topic: article
ms.date: 04/27/2021
tags: connectors
ms.openlocfilehash: 0bdb19104d7773d63f583b4013ac9d80893500ae
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/28/2021
ms.locfileid: "108147263"
---
# <a name="connect-to-sap-systems-from-azure-logic-apps"></a>Herstellen einer Verbindung zu SAP-Systemen: Azure Logic Apps

In diesem Artikel wird erläutert, wie Sie aus Logic Apps auf Ihre SAP-Ressourcen zugreifen können, indem Sie den [SAP-Connector](/connectors/sap/) verwenden.

## <a name="prerequisites"></a>Voraussetzungen

* Ein Azure-Abonnement. Wenn Sie noch kein Azure-Abonnement haben, [melden Sie sich für ein kostenloses Azure-Konto an](https://azure.microsoft.com/free/).

* Eine Logik-App, aus der Sie auf Ihre SMTP-Ressourcen zugreifen möchten. Wenn Sie mit Logic Apps noch nicht vertraut sind, helfen Ihnen die [Logic Apps-Dienstübersicht](../logic-apps/logic-apps-overview.md) und die [Schnellstartanleitung zur Erstellung Ihrer ersten Logik-App im Azure-Portal](../logic-apps/quickstart-create-first-logic-app-workflow.md) weiter.

    * Wenn Sie eine frühere Version des SAP-Connectors verwendet haben, die veraltet ist, müssen Sie [zum aktuellen Connector migrieren,](#migrate-to-current-connector), bevor Sie eine Verbindung mit Ihrem SAP-Server herstellen können.

    * Wenn Sie Ihre Logik-App in einer mehrinstanzenfähigen Azure-Umgebung ausführen, lesen Sie die [Voraussetzungen für Mehrinstanzenfähigkeit](#multi-tenant-azure-prerequisites).

    * Wenn Sie Ihre Logik-App in einer [Integrationsdienstumgebung (Integration Service Environment, ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md) der Premium-Ebene ausführen, lesen Sie die [ISE-Voraussetzungen](#ise-prerequisites).

* Ein [SAP-Anwendungsserver](https://wiki.scn.sap.com/wiki/display/ABAP/ABAP+Application+Server) oder ein [SAP-Nachrichtenserver](https://help.sap.com/saphelp_nw70/helpdata/en/40/c235c15ab7468bb31599cc759179ef/frameset.htm), auf den Sie aus Logic Apps zugreifen möchten. Informationen dazu, welche SAP-Server Sie mit dem Connector verwenden können, finden Sie unter [SAP-Kompatibilität](#sap-compatibility).

    * Sie müssen Ihren SAP-Server so konfigurieren, dass er die Verwendung von RFC zulässt. Weitere Informationen finden Sie im folgenden SAP-Hinweis [460089: Mindestautorisierungsprofile für externe RFC-Programme](https://launchpad.support.sap.com/#/notes/460089). 

* Nachrichteninhalte, die Sie an Ihren SAP-Server senden möchten, z.B. eine IDoc-Beispieldatei. Diese Inhalte müssen im XML-Format vorliegen und den Namespace der [SAP-Aktion](#actions) enthalten, die Sie verwenden möchten. Sie können [IDocs mit einem Flatfile-Schema senden, indem Sie sie mit einem XML-Umschlag umschließen](#send-flat-file-idocs).

* Wenn Sie den Trigger **Beim Empfang einer Nachricht von SAP** verwenden möchten, müssen Sie zudem Folgendes ausführen:

    * Richten Sie die Sicherheitsberechtigungen für Ihr SAP-Gateway mit der folgenden Einstellung ein: `"TP=Microsoft.PowerBI.EnterpriseGateway HOST=<gateway-server-IP-address> ACCESS=*"`

    * Richten Sie die Sicherheitsprotokollierung für Ihr SAP-Gateway ein, mit der Sie nach der Zugriffssteuerungsliste suchen können. Weitere Informationen finden Sie im [SAP-Hilfethema zum Einrichten der Gatewayprotokollierung](https://help.sap.com/erp_hcm_ias2_2015_02/helpdata/en/48/b2a710ca1c3079e10000000a42189b/frameset.htm). Andernfalls erhalten Sie diesen Fehler: `"Registration of tp Microsoft.PowerBI.EnterpriseGateway from host <host-name> not allowed"`

    > [!NOTE]
    > Dieser Trigger verwendet denselben URI-Speicherort sowohl zum Verlängern als auch zum Kündigen eines Webhook-Abonnements. Der Erneuerungsvorgang verwendet die HTTP-Methode `PATCH`, während der Abmeldevorgang die HTTP-Methode `DELETE` verwendet. Dieses Verhalten kann dazu führen, dass ein Verlängerungsvorgang im Verlauf Ihres Triggers als Abmeldevorgang angezeigt wird, doch der Vorgang ist immer noch eine Verlängerung, weil der Trigger `PATCH` als HTTP-Methode verwendet, nicht `DELETE`.

### <a name="sap-compatibility"></a>SAP-Kompatibilität

Der SAP-Connector ist mit den folgenden Typen von SAP-Systemen kompatibel:

* Lokale und cloudbasierte Hana-basierte SAP-Systeme wie S/4 HANA.

* Klassische lokale SAP-Systeme wie R/3 und ECC.

Der SAP-Connector unterstützt die folgenden Nachrichten- und Datenintegrationstypen aus SAP NetWeaver-basierten Systemen:

* IDoc (Intermediate Document)

* BAPI (Business Application Programming Interface)

* RFC (Remote Function Call) und tRFC (Transactional RFC)

Der SAP-Connector verwendet die [SAP-NCo-Bibliothek (.NET-Connector)](https://support.sap.com/en/product/connectors/msnet.html). 

Um die verfügbaren [SAP Trigger](#triggers) und [SAP-Aktionen](#actions) nutzen zu können, müssen Sie Ihre Verbindung zunächst mit einem Benutzernamen und einem Passwort authentifizieren. Der SAP-Connector unterstützt auch [Secure Network Communications (SNC)](https://help.sap.com/doc/saphelp_nw70/7.0.31/e6/56f466e99a11d1a5b00000e835363f/content.htm?no_cache=true). Sie können einmaliges Anmelden (SSO) mit SNC für SAP NetWeaver verwenden oder für zusätzliche Sicherheit Funktionen externer Produkte. Wenn Sie SNC verwenden, lesen Sie die [SNC-Voraussetzungen](#snc-prerequisites).

### <a name="migrate-to-current-connector"></a>Migrieren zum aktuellen Connector

Die vorherigen Connectors für SAP-Anwendungsserver und SAP-Nachrichtenserver wurden am 29. Februar 2020 eingestellt. Führen Sie die folgenden Schritte aus, um zum aktuellen SAP-Connector zu migrieren:

1. Aktualisieren Sie Ihr [lokales Datengateway](https://www.microsoft.com/download/details.aspx?id=53127) auf die aktuelle Version. Weitere Informationen finden Sie unter [Herstellen einer Verbindung mit einem lokalen SAP-System in Logik-Apps mit dem SAP-Connector](../logic-apps/logic-apps-gateway-install.md).

1. Löschen Sie in Ihrer Logik-App, die den veralteten SAP-Connector verwendet, die Aktion **An SAP senden**.

1. Fügen Sie die Aktion **Nachricht an SAP senden** aus dem aktuellen SAP-Connector hinzu.

1. Stellen Sie in der neuen Aktion erneut eine Verbindung mit Ihrem SAP-System her.

1. Speichern Sie Ihre Logik-App.

### <a name="multi-tenant-azure-prerequisites"></a>Voraussetzungen für eine mehrinstanzenfähige Azure-Umgebung

Diese Voraussetzungen gelten, wenn Ihre Logik-App in einer mehrinstanzenfähigen Azure-Umgebung ausgeführt wird. Der verwaltete SAP-Connector wird nicht nativ in einer [ISE](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md) ausgeführt.

> [!TIP]
> Wenn Sie eine ISE auf Premium-Ebene verwenden, können Sie anstelle des verwalteten SAP-Connectors den SAP-Connector für die ISE verwenden. Weitere Informationen finden Sie in den [ISE-Voraussetzungen](#ise-prerequisites).

Der verwaltete SAP-Connector kann über Ihr [lokales Datengateway](../logic-apps/logic-apps-gateway-connection.md) in SAP-Systeme integriert werden. In Szenarios, in denen beispielsweise eine Nachricht von einer Logik-App an ein SAP-System gesendet wird, agiert das Datengateway als RFC-Client und leitet die von der Logik-App erhaltenen Anforderungen an SAP weiter. Ebenso fungiert das Datengateway in Empfangsszenarios als RFC-Server, der Anforderungen von SAP empfängt und an die Logik-App weiterleitet.

* [Laden Sie das lokale Datengateway herunter, und installieren Sie es](../logic-apps/logic-apps-gateway-install.md) auf einem Hostcomputer oder einem virtuellen Computer, der sich im selben virtuellen Netzwerk wie das SAP-System befindet, mit dem Sie eine Verbindung herstellen.

* [Erstellen Sie eine Azure-Gatewayressource](../logic-apps/logic-apps-gateway-connection.md#create-azure-gateway-resource) für Ihr lokales Datengateway im Azure-Portal. Dieses Gateway hilft Ihnen, sicher auf lokale Daten und Ressourcen zuzugreifen. Stellen Sie sicher, dass Sie eine unterstützte Version des Gateways verwenden.

    * Führen Sie bei einem Problem mit dem Gateway ein [Upgrade auf die neueste Version](https://aka.ms/on-premises-data-gateway-installer) durch, die möglicherweise Updates zur Behebung des Problems enthält.

* [Laden Sie die neueste SAP-Clientbibliothek herunter](#sap-client-library-prerequisites), und installieren Sie sie auf demselben Computer, auf dem sich bereits Ihr lokales Datengateway befindet.

### <a name="ise-prerequisites"></a>ISE-Voraussetzungen

Diese Voraussetzungen gelten, wenn Sie Ihre Logik-App in einer ISE der Premium-Ebene ausführen. Sie gelten jedoch nicht für Logik-Apps, die in einer ISE auf Entwicklerebene ausgeführt werden. Eine ISE ermöglicht den Zugriff auf Ressourcen, die von einem virtuellen Azure-Netzwerk geschützt werden, und stellt weitere nativ in ISEs ausgeführte Connectors bereit, mit denen Logik-Apps direkt auf lokale Ressourcen zugreifen können, ohne das lokale Datengateway verwenden zu müssen.

1. Wenn Sie noch nicht über ein Azure Storage-Konto mit einem Blobcontainer verfügen, erstellen Sie einen Container im [Azure-Portal](../storage/blobs/storage-quickstart-blobs-portal.md) oder im [Azure Storage-Explorer](../storage/blobs/storage-quickstart-blobs-storage-explorer.md).

1. [Laden Sie die neueste SAP-Clientbibliothek](#sap-client-library-prerequisites) auf Ihren lokalen Computer herunter, und installieren Sie sie. Sie sollten über die folgenden Assemblydateien verfügen:

   * libicudecnumber.dll

   * rscp4n.dll

   * sapnco.dll

   * sapnco_utils.dll

1. Erstellen Sie eine ZIP-Datei, die diese Assemblydateien enthält. Laden Sie das Paket in Ihren Blobcontainer in Azure Storage hoch.

1. Navigieren Sie entweder im Azure-Portal oder im Azure Storage-Explorer zum Speicherort des Containers, in den Sie die ZIP-Datei hochgeladen haben.

1. Kopieren Sie die URL des Containerspeicherorts. Stellen Sie sicher, dass Sie das SAS-Token (Shared Access Signature) einschließen, damit das SAS-Token autorisiert wird. Andernfalls schlägt die Bereitstellung für den SAP-Connector für die ISE fehl.

1. Installieren Sie den SAP-Connector in Ihrer ISE, und stellen Sie ihn dort bereit. Weitere Informationen finden Sie unter [Hinzufügen von ISE-Connectors](../logic-apps/add-artifacts-integration-service-environment-ise.md#add-ise-connectors-environment).

   1. Suchen Sie im [Azure-Portal](https://portal.azure.com) nach Ihrer ISE, und öffnen Sie sie.

   1. Wählen Sie im ISE-Menü **Verwaltete Connectors**&gt;**Hinzufügen** aus. Suchen Sie in der Liste der Connectors nach **SAP**, und wählen Sie diesen Connector aus.

   1. Fügen Sie im Bereich **Neuen verwalteten Connector hinzufügen** im Feld **SAP-Paket** die URL für die ZIP-Datei mit den SAP-Assemblys ein. Vergewissern Sie sich erneut, dass das SAS-Token eingeschlossen ist.
 
  1. Wählen Sie **Erstellen** aus, um das Erstellen Ihres ISE-Connectors abzuschließen.

1. Wenn sich Ihre SAP-Instanz und die ISE in unterschiedlichen virtuellen Netzwerken befinden, müssen Sie außerdem ein [Peering zwischen diesen Netzwerken herstellen](../virtual-network/tutorial-connect-virtual-networks-portal.md), damit sie verbunden sind.

### <a name="sap-client-library-prerequisites"></a>Voraussetzungen für die SAP-Clientbibliothek

Dies sind die Voraussetzungen für die SAP-Clientbibliothek, die Sie mit dem Connector verwenden.

* Stellen Sie sicher, dass Sie über die neueste Version verfügen: [SAP-Connector (NCo 3.0) für Microsoft .NET 3.0.22.0 mit .NET Framework 4.0 kompiliert – Windows 64-Bit (x64)](https://support.sap.com/en/product/connectors/msnet.html). Bei früheren Versionen von SAP NCo könnte es zu Problemen kommen, wenn mehr als eine IDoc-Nachricht gleichzeitig gesendet wird. Durch diesen Zustand werden auch alle späteren Nachrichten blockiert, die an das SAP-Ziel gesendet werden, wodurch es zu einem Timeout der Nachricht kommt.

* Da das Datengateway nur auf 64-Bit-Systemen ausgeführt werden kann, müssen Sie die 64-Bit-Version der SAP-Clientbibliothek installiert haben. Die Installation der nicht unterstützten 32-Bit-Version führt zu einem Fehler „Ungültiges Image“.

* Kopieren Sie die Assemblydateien aus dem Standardinstallationsordner gemäß Ihrem Szenario wie folgt an einen anderen Speicherort.

    * Halten Sie für Logik-Apps, die in einer ISE ausgeführt werden, stattdessen die [ISE-Voraussetzungen](#ise-prerequisites) ein.

    * Kopieren Sie für Logik-Apps, die in einer mehrinstanzenfähigen Azure-Umgebung ausgeführt werden und Ihr lokales Datengateway verwenden, die Assemblydateien in den Installationsordner des Datengateways. 

        
        * Tritt für die SAP-Verbindung die Fehlermeldung **Bitte überprüfen Sie Ihre Kontoinformationen und/oder Berechtigungen, und wiederholen Sie den Vorgang** auf, vergewissern Sie sich, dass Sie die Assemblydateien in den Installationsordner des Datengateways kopiert haben.
        
        * Behandeln Sie weitere Probleme mithilfe der [Protokollanzeige für .NET-Assemblybindungen](/dotnet/framework/tools/fuslogvw-exe-assembly-binding-log-viewer). Mit diesem Tool können Sie überprüfen, ob sich Ihre Assemblydateien am richtigen Speicherort befinden. 
        
        * Wählen Sie optional bei der Installation der SAP-Clientbibliothek die Option **Globalen Assemblycache registrieren** aus.

Beachten Sie die folgenden Beziehungen zwischen der SAP-Clientbibliothek, .NET Framework, der .NET-Laufzeit und dem Gateway:

* Sowohl der SAP-Adapter von Microsoft als auch der Gatewayhostdienst verwenden .NET Framework 4.7.2.

* Der SAP .NET-Connector für .NET Framework 4.0 funktioniert mit Prozessen, die die .NET Runtime 4.0 bis 4.8 verwenden.

* Der SAP .NET-Connector für .NET Framework 2.0 funktioniert mit Prozessen, die die .NET-Laufzeit 2.0 bis 3.5 verwenden. Er kann jedoch nicht mehr mit dem neuesten Gateway verwendet werden.

### <a name="snc-prerequisites"></a>SNC-Voraussetzungen

Wenn Sie ein lokales Datengateway mit optionaler SNC (Secure Network Communications) verwenden, das nur in einer mehrinstanzenfähigen Azure-Umgebung unterstützt wird, müssen Sie auch die folgenden Einstellungen konfigurieren.

Stellen Sie bei Verwendung von SNC mit SSO (einmaliges Anmelden) sicher, dass der Datengatewaydienst als Benutzer ausgeführt wird, der dem SAP-Benutzer zugeordnet ist. Wählen Sie **Konto ändern** aus, und geben Sie die Anmeldeinformationen des Benutzers ein, um das Standardkonto zu ändern.

![Screenshot der Einstellungen des lokalen Datengateways im Azure-Portal mit der Seite „Diensteinstellungen“ mit der ausgewählten Schaltfläche zum Ändern des Gatewaydienstkontos.](./media/logic-apps-using-sap-connector/gateway-account.png)

Wenn Sie SNC über ein externes Sicherheitsprodukt aktivieren, kopieren Sie die SNC-Bibliothek oder -Dateien auf denselben Computer, auf dem Ihr Datengateway installiert ist. Einige Beispiele für SNC-Produkte sind [sapseculib](https://help.sap.com/saphelp_nw74/helpdata/en/7a/0755dc6ef84f76890a77ad6eb13b13/frameset.htm), Kerberos und NTLM. Weitere Informationen zum Aktivieren von SNC für das Datengateway finden Sie im Abschnitt[Aktivieren von Secure Network Communications (SNC)](#enable-secure-network-communications).

## <a name="send-idoc-messages-to-sap-server"></a>IDoc-Nachrichten an SAP-Server senden

Befolgen Sie diese Beispiele, um eine Logik-App zu erstellen, die eine IDoc-Nachricht an einen SAP-Server sendet und eine Antwort zurückgibt:

1. [Erstellen Sie eine Logik-App, die von einer HTTP-Anforderung ausgelöst wird.](#create-http-request-trigger)

1. [Erstellen Sie eine Aktion in Ihrem Workflow, um eine Nachricht an SAP zu senden.](#create-sap-action-to-send-message)

1. [Erstellen Sie eine HTTP-Antwortaktion in Ihrem Workflow.](#create-http-response-action)

1. [Erstellen Sie ein Anforderung/Antwort-Muster für einen Remotefunktionsaufruf (RFC), wenn Sie einen RFC verwenden, um Antworten von SAP ABAP zu empfangen.](#create-rfc-request-response)

1. [Testen Sie Ihre Logik-App.](#test-logic-app)

### <a name="create-http-request-trigger"></a>Erstellen eines HTTP-Anforderungstriggers

> [!NOTE]
> Wenn eine Logik-App IDocs von SAP empfängt, unterstützt der [Anforderungstrigger](../connectors/connectors-native-reqres.md) jetzt das einfache XML-Format von SAP. Um IDocs als einfaches XML zu empfangen, verwenden Sie den Trigger **Beim Empfang einer Nachricht von SAP**. Legen Sie den Parameter **IDOC-Format** auf **SapPlainXml** fest.

Erstellen Sie zunächst eine Logik-App mit einem Endpunkt in Azure, um *HTTP POST*-Anforderungen an Ihre Logik-App zu senden. Wenn Ihre Logik-App diese HTTP-Anforderungen empfängt, wird der [Trigger](../logic-apps/logic-apps-overview.md#logic-app-concepts) ausgelöst und führt den nächsten Schritt in Ihrem Workflow aus.

1. Erstellen Sie im [Azure-Portal](https://portal.azure.com) eine leere Logik-App, die den **Logik-App-Designer** öffnet.

1. Geben Sie im Suchfeld den Begriff `http request` als Filter ein. Wählen Sie in der Liste der **Trigger** die Option **Beim Empfang einer HTTP-Anforderung** aus.

   ![Screenshot des Logik-App-Designers, der einen neuen HTTP-Anforderungstrigger zeigt, der der Logik-App hinzugefügt wird.](./media/logic-apps-using-sap-connector/add-http-trigger-logic-app.png)

1. Speichern Sie Ihre Logik-App, damit Sie eine Endpunkt-URL dafür generieren können. Wählen Sie auf der Symbolleiste des Designers **Speichern** aus. Die Endpunkt-URL wird nun in Ihrem Trigger angezeigt. 

   ![Screenshot des Logik-App-Designers, der einen HTTP-Anforderungstrigger mit generierter POST-URL zeigt, die kopiert wird.](./media/logic-apps-using-sap-connector/generate-http-endpoint-url.png)

### <a name="create-sap-action-to-send-message"></a>Erstellen einer SAP-Aktion zum Senden von Nachrichten

Erstellen Sie als Nächstes eine Aktion zum Senden Ihrer IDoc-Nachricht an SAP, wenn Ihr [HTTP-Anforderungstrigger](#create-http-request-trigger) ausgelöst wird.

1. Wählen Sie im Logik-App-Designer unter dem Trigger die Option **Neuer Schritt** aus.

   ![Screenshot des Logik-App-Designers, der zeigt, wie die Logik-App bearbeitet wird, um einen neuen Schritt hinzuzufügen.](./media/logic-apps-using-sap-connector/add-sap-action-logic-app.png)

1. Geben Sie im Suchfeld den Begriff `sap` als Filter ein. Wählen Sie in der Liste **Aktionen** die Option **Nachricht an SAP senden** aus.
  
   ![Screenshot des Logik-App-Designers, der das Auswählen der Aktion „Nachricht an SAP senden“ zeigt.](media/logic-apps-using-sap-connector/select-sap-send-action.png)

   Alternativ können Sie die Registerkarte **Unternehmen** und dann die SAP-Aktion auswählen.

   ![Screenshot des Logik-App-Designers, der das Auswählen der Aktion „Nachricht an SAP senden“ auf der Registerkarte „Unternehmen“ zeigt.](media/logic-apps-using-sap-connector/select-sap-send-action-ent-tab.png)

1. Wenn Ihre Verbindung bereits besteht, fahren Sie mit dem nächsten Schritt fort. Wenn Sie aufgefordert werden, eine neue Verbindung herzustellen, geben Sie folgende Informationen an, um eine Verbindung mit Ihrem lokalen SAP-Server herzustellen.

   1. Geben Sie einen Namen für die Verbindung an.

   1. Wenn Sie das Datengateway verwenden, führen Sie die folgenden Schritte aus:
   
      1. Wählen Sie im Abschnitt **Datengateway** unter **Abonnement** das Azure-Abonnement für die Datengatewayressource aus, die Sie im Azure-Portal für die Installation Ihres Datengateways erstellt haben.
   
      1. Wählen Sie unter **Verbindungsgateway** Ihre Datengatewayressource in Azure aus.

   1. Fahren Sie mit dem Angeben von Informationen fort. Führen Sie die Schritte für die Eigenschaft **Anmeldetyp** abhängig davon aus, ob die Eigenschaft auf **Anwendungsserver** oder **Gruppe** festgelegt ist:
   
      * Bei **Anwendungsserver** sind die folgenden Eigenschaften erforderlich, die sonst optional sind:

        ![Verbindung zum SAP-Anwendungsserver herstellen](media/logic-apps-using-sap-connector/create-SAP-application-server-connection.png)

      * Bei **Gruppe** sind die folgenden Eigenschaften erforderlich, die sonst optional sind:

        ![Verbindung zum SAP-Nachrichtenserver herstellen](media/logic-apps-using-sap-connector/create-SAP-message-server-connection.png)  

      Standardmäßig wird starke Typisierung verwendet, um nach ungültigen Werten zu suchen, indem eine XML-Validierung anhand des Schemas durchgeführt wird. Durch dieses Verhalten können Sie Probleme früher feststellen. Die Option **Sichere Typisierung** ist aus Gründen der Abwärtskompatibilität verfügbar und überprüft nur die Länge der Zeichenfolge. Erfahren Sie mehr über die Option [Sichere Typisierung](#safe-typing).

   1. Wählen Sie **Erstellen** aus, wenn Sie fertig sind.

      Die Verbindung wird in Logic Apps eingerichtet und getestet, sodass sichergestellt ist, dass sie ordnungsgemäß funktioniert.

    > [!NOTE]

    > Wenn der folgende Fehler auftritt, liegt ein Problem mit Ihrer Installation der SAP NCo-Clientbibliothek vor: 
    >
    > **Fehler beim Testen der Verbindung. Fehler „Fehler beim Verarbeiten der Anforderung. Fehlerdetails: „Die Datei oder Assembly ‚sapnco, Version=3.0.0.42, Culture=neutral, PublicKeyToken 50436dca5c7f7d23‘ oder eine Abhängigkeit davon konnte nicht geladen werden. Das System kann die angegebene Datei nicht finden.“.“**
    >
    > Stellen Sie sicher, dass Sie [die erforderliche Version der SAP NCo-Clientbibliothek installieren und alle anderen Voraussetzungen erfüllen](#sap-client-library-prerequisites).

1. Suchen Sie nun eine Aktion für Ihren SAP-Server, und wählen Sie sie aus.

   1. Wählen Sie im Feld **SAP-Aktion** das Ordnersymbol aus. Suchen Sie in der Dateiliste nach der gewünschten SAP-Nachricht, und wählen Sie sie aus. Verwenden Sie die Pfeile zum Durchlaufen der Liste.

      In diesem Beispiel wird ein IDoc mit dem Typ **Aufträge** ausgewählt.

      ![Aktion „IDoc“ suchen und auswählen](./media/logic-apps-using-sap-connector/SAP-app-server-find-action.png)

      Wenn Sie die Aktion, die Sie verwenden möchten, nicht finden können, können Sie einen Pfad auch manuell eingeben:

      ![Pfad zur Aktion „IDoc“ manuell bereitstellen](./media/logic-apps-using-sap-connector/SAP-app-server-manually-enter-action.png)

      > [!TIP]
      > Geben Sie den Wert für die **SAP-Aktion** über den Ausdrucks-Editor ein. Auf diese Weise können Sie dieselbe Aktion für verschiedene Nachrichtentypen verwenden.

      Weitere Informationen zu IDoc-Vorgängen finden Sie unter [Nachrichtenschemas für IDoc-Vorgänge](/biztalk/adapters-and-accelerators/adapter-sap/message-schemas-for-idoc-operations).

   1. Klicken Sie in das Feld **Input Message** (Eingabenachricht), damit die dynamische Inhaltsliste angezeigt wird. Klicken Sie in dieser Liste unter **Beim Empfang einer HTTP-Anforderung** auf das Feld **Hauptteil**.

      Dadurch wird der Hauptteil des HTTP-Anforderungstriggers eingefügt und diese Ausgabe an Ihren SAP-Server gesendet.

      ![Auswählen der Eigenschaft „Text“ über den Auslöser](./media/logic-apps-using-sap-connector/SAP-app-server-action-select-body.png)

      Ihre SAP-Aktion sieht dann wie im folgenden Beispiel aus:

      ![Vervollständigen der SAP-Aktion](./media/logic-apps-using-sap-connector/SAP-app-server-complete-action.png)

1. Speichern Sie Ihre Logik-App. Wählen Sie auf der Symbolleiste des Designers **Speichern** aus.

#### <a name="send-flat-file-idocs"></a>Senden von Flatfile-IDocs

Sie können IDocs mit einem Flatfile-Schema verwenden, wenn Sie sie mit einem XML-Umschlag umschließen. Um ein Flatfile-IDoc zu senden, verwenden Sie die generischen Anweisungen zum [Erstellen einer SAP-Aktion zum Senden Ihrer IDoc-Nachricht](#create-sap-action-to-send-message) mit den folgenden Änderungen.

1. Verwenden Sie für die Aktion **Nachricht an SAP senden** den SAP-Aktions-URI `http://microsoft.lobservices.sap/2007/03/Idoc/SendIdoc`.

1. Formatieren Sie Ihre Eingabenachricht mit einem XML-Umschlag. Ein Beispiel finden Sie in der folgenden XML-Beispielnutzlast:

```xml
<ReceiveIdoc xmlns="http://Microsoft.LobServices.Sap/2007/03/Idoc/"><idocData>EDI_DC 3000000001017945375750 30INVOIC011BTSVLINV30KUABCABCFPPC LDCA X004010810 4 SAPMSX LSEDI ABCABCFPPC 000d3ae4-723e-1edb-9ca4-cc017365c9fd 20210217054521INVOICINVOIC01ZINVOIC2RE 20210217054520
E2EDK010013000000001017945375000001E2EDK01001000000010 ABCABC1.00000 0060 INVO9988298128 298.000 298.000 LB Z4LR EN 0005065828 L
E2EDKA1 3000000001017945375000002E2EDKA1 000000020 RS ABCABCFPPC 0005065828 ABCABCABC ABCABC Inc. Limited Risk Distributor ABCABC 1950 ABCABCABCA Blvd ABCABAABCAB L5N8L9 CA ABCABC E ON V-ABCABC LDCA
E2EDKA1 3000000001017945375000003E2EDKA1 000000020 AG 0005065828 ABCABCFPPC ABCABC ABCABC ABCABC - FPP ONLY 88 ABCABC Crescent ABCABAABCAB L5R 4A2 CA ABCABC 111 111 1111 E ON ABCABCFPPC EN
E2EDKA1 3000000001017945375000004E2EDKA1 000000020 RE 0005065828 ABCABCFPPC ABCABC ABCABC ABCABC - FPP ONLY 88 ABCABC Crescent ABCABAABCAB L5R 4A2 CA ABCABC 111 111 1111 E ON ABCABCFPPC EN
E2EDKA1 3000000001017945375000005E2EDKA1 000000020 RG 0005065828 ABCABCFPPC ABCABC ABCABC ABCABC - FPP ONLY 88 ABCABC Crescent ABCABAABCAB L5R 4A2 CA ABCABC 111 111 1111 E ON ABCABCFPPC EN
E2EDKA1 3000000001017945375000006E2EDKA1 000000020 WE 0005001847 41 ABCABC ABCABC INC (ABCABC) DC A. ABCABCAB 88 ABCABC CRESCENT ABCABAABCAB L5R 4A2 CA ABCABC 111-111-1111 E ON ABCABCFPPC EN
E2EDKA1 3000000001017945375000007E2EDKA1 000000020 Z3 0005533050 ABCABCABC ABCABC Inc. ABCA Bank Swift Code -ABCABCABCAB Sort Code - 1950 ABCABCABCA Blvd. Acc No -1111111111 ABCABAABCAB L5N8L9 CA ABCABC E ON ABCABCFPPC EN
E2EDKA1 3000000001017945375000008E2EDKA1 000000020 BK 1075 ABCABCABC ABCABC Inc 1950 ABCABCABCA Blvd ABCABAABCAB ON L5N 8L9 CA ABCABC (111) 111-1111 (111) 111-1111 ON
E2EDKA1 3000000001017945375000009E2EDKA1 000000020 CR 1075 CONTACT ABCABCABC 1950 ABCABCABCA Blvd ABCABAABCAB ON L5N 8L9 CA ABCABC (111) 111-1111 (111) 111-1111 ON
E2EDK02 3000000001017945375000010E2EDK02 000000020 0099988298128 20210217
E2EDK02 3000000001017945375000011E2EDK02 000000020 00140-N6260-S 20210205
E2EDK02 3000000001017945375000012E2EDK02 000000020 0026336270425 20210217
E2EDK02 3000000001017945375000013E2EDK02 000000020 0128026580537 20210224
E2EDK02 3000000001017945375000014E2EDK02 000000020 01740-N6260-S
E2EDK02 3000000001017945375000015E2EDK02 000000020 900IAC
E2EDK02 3000000001017945375000016E2EDK02 000000020 901ZSH
E2EDK02 3000000001017945375000017E2EDK02 000000020 9078026580537 20210217
E2EDK03 3000000001017945375000018E2EDK03 000000020 02620210217
E2EDK03 3000000001017945375000019E2EDK03 000000020 00120210224
E2EDK03 3000000001017945375000020E2EDK03 000000020 02220210205
E2EDK03 3000000001017945375000021E2EDK03 000000020 01220210217
E2EDK03 3000000001017945375000022E2EDK03 000000020 01120210217
E2EDK03 3000000001017945375000023E2EDK03 000000020 02420210217
E2EDK03 3000000001017945375000024E2EDK03 000000020 02820210418
E2EDK03 3000000001017945375000025E2EDK03 000000020 04820210217
E2EDK17 3000000001017945375000026E2EDK17 000000020 001DDPDelivered Duty Paid
E2EDK17 3000000001017945375000027E2EDK17 000000020 002DDPdestination
E2EDK18 3000000001017945375000028E2EDK18 000000020 00160 0 Up to 04/18/2021 without deduction
E2EDK28 3000000001017945375000029E2EDK28 000000020 CA BOFACATT Bank of ABCABAB ABCABC ABCABAB 50127217 ABCABCABC ABCABC Inc.
E2EDK28 3000000001017945375000030E2EDK28 000000020 CA 026000082 ABCAbank ABCABC ABCABAB 201456700OLD ABCABCABC ABCABC Inc.
E2EDK28 3000000001017945375000031E2EDK28 000000020 GB ABCAGB2L ABCAbank N.A ABCABA E14, 5LB GB63ABCA18500803115593 ABCABCABC ABCABC Inc. GB63ABCA18500803115593
E2EDK28 3000000001017945375000032E2EDK28 000000020 CA 020012328 ABCABANK ABCABC ABCABAB ON M5J 2M3 2014567007 ABCABCABC ABCABC Inc.
E2EDK28 3000000001017945375000033E2EDK28 000000020 CA 03722010 ABCABABC ABCABABC Bank of Commerce ABCABAABCAB 64-04812 ABCABCABC ABCABC Inc.
E2EDK28 3000000001017945375000034E2EDK28 000000020 IE IHCC In-House Cash Center IHCC1075 ABCABCABC ABCABC Inc.
E2EDK28 3000000001017945375000035E2EDK28 000000020 CA 000300002 ABCAB Bank of ABCABC ABCABAB 0021520584OLD ABCABCABC ABCABC Inc.
E2EDK28 3000000001017945375000036E2EDK28 000000020 US USCC US Cash Center (IHC) city USCC1075 ABCABCABC ABCABC Inc.
E2EDK29 3000000001017945375000037E2EDK29 000000020 0064848944US A CAD CA ABCABC CA United States US CA A Air Air
E2EDKT1 3000000001017945375000038E2EDKT1 000000020 ZJ32E EN
E2EDKT2 3000000001017945375000039E2EDKT2 000038030 GST/HST877845941RT0001 *
E2EDKT2 3000000001017945375000040E2EDKT2 000038030 QST1021036966TQ0001 *
E2EDKT1 3000000001017945375000041E2EDKT1 000000020 Z4VL
E2EDKT2 3000000001017945375000042E2EDKT2 000041030 0.000 *
E2EDKT1 3000000001017945375000043E2EDKT1 000000020 Z4VH
E2EDKT2 3000000001017945375000044E2EDKT2 000043030 *
E2EDK14 3000000001017945375000045E2EDK14 000000020 008LDCA
E2EDK14 3000000001017945375000046E2EDK14 000000020 00710
E2EDK14 3000000001017945375000047E2EDK14 000000020 00610
E2EDK14 3000000001017945375000048E2EDK14 000000020 015Z4F2
E2EDK14 3000000001017945375000049E2EDK14 000000020 0031075
E2EDK14 3000000001017945375000050E2EDK14 000000020 021M
E2EDK14 3000000001017945375000051E2EDK14 000000020 0161075
E2EDK14 3000000001017945375000052E2EDK14 000000020 962M
E2EDP010013000000001017945375000053E2EDP01001000000020 000011 2980.000 EA 298.000 LB MOUSE 298.000 Z4TN 4260
E2EDP02 3000000001017945375000054E2EDP02 000053030 00140-N6260-S 00000120210205 DFUE
E2EDP02 3000000001017945375000055E2EDP02 000053030 0026336270425 00001120210217
E2EDP02 3000000001017945375000056E2EDP02 000053030 0168026580537 00001020210224
E2EDP02 3000000001017945375000057E2EDP02 000053030 9100000 00000120210205 DFUE
E2EDP02 3000000001017945375000058E2EDP02 000053030 911A 00000120210205 DFUE
E2EDP02 3000000001017945375000059E2EDP02 000053030 912PP 00000120210205 DFUE
E2EDP02 3000000001017945375000060E2EDP02 000053030 91300 00000120210205 DFUE
E2EDP02 3000000001017945375000061E2EDP02 000053030 914CONTACT ABCABCABC 00000120210205 DFUE
E2EDP02 3000000001017945375000062E2EDP02 000053030 963 00000120210205 DFUE
E2EDP02 3000000001017945375000063E2EDP02 000053030 965 00000120210205 DFUE
E2EDP02 3000000001017945375000064E2EDP02 000053030 9666336270425 00000120210205 DFUE
E2EDP02 3000000001017945375000065E2EDP02 000053030 9078026580537 00001020210205 DFUE
E2EDP03 3000000001017945375000066E2EDP03 000053030 02920210217
E2EDP03 3000000001017945375000067E2EDP03 000053030 00120210224
E2EDP03 3000000001017945375000068E2EDP03 000053030 01120210217
E2EDP03 3000000001017945375000069E2EDP03 000053030 02520210217
E2EDP03 3000000001017945375000070E2EDP03 000053030 02720210217
E2EDP03 3000000001017945375000071E2EDP03 000053030 02320210217
E2EDP03 3000000001017945375000072E2EDP03 000053030 02220210205
E2EDP19 3000000001017945375000073E2EDP19 000053030 001418VVZ
E2EDP19 3000000001017945375000074E2EDP19 000053030 002RJR-00001 AB ABCABCABC Mouse FORBUS BLUETOOTH
E2EDP19 3000000001017945375000075E2EDP19 000053030 0078471609000
E2EDP19 3000000001017945375000076E2EDP19 000053030 003889842532685
E2EDP19 3000000001017945375000077E2EDP19 000053030 011CN
E2EDP26 3000000001017945375000078E2EDP26 000053030 00459064.20
E2EDP26 3000000001017945375000079E2EDP26 000053030 00352269.20
E2EDP26 3000000001017945375000080E2EDP26 000053030 01052269.20
E2EDP26 3000000001017945375000081E2EDP26 000053030 01152269.20
E2EDP26 3000000001017945375000082E2EDP26 000053030 0126795.00
E2EDP26 3000000001017945375000083E2EDP26 000053030 01552269.20
E2EDP26 3000000001017945375000084E2EDP26 000053030 00117.54
E2EDP26 3000000001017945375000085E2EDP26 000053030 00252269.20
E2EDP26 3000000001017945375000086E2EDP26 000053030 940 2980.000
E2EDP26 3000000001017945375000087E2EDP26 000053030 939 2980.000
E2EDP05 3000000001017945375000088E2EDP05 000053030 + Z400MS List Price 52269.20 17.54 1 EA CAD 2980
E2EDP05 3000000001017945375000089E2EDP05 000053030 + XR1 Tax Jur Code Level 6795.00 13.000 52269.20
E2EDP05 3000000001017945375000090E2EDP05 000053030 + Tax Subtotal1 6795.00 2.28 1 EA CAD 2980
E2EDP05 3000000001017945375000091E2EDP05 000053030 + Taxable Amount + TaxSubtotal1 59064.20 19.82 1 EA CAD 2980
E2EDP04 3000000001017945375000092E2EDP04 000053030 CX 13.000 6795.00 7000000000
E2EDP04 3000000001017945375000093E2EDP04 000053030 CX 0 0 7001500000
E2EDP04 3000000001017945375000094E2EDP04 000053030 CX 0 0 7001505690
E2EDP28 3000000001017945375000095E2EDP28 000053030 00648489440000108471609000 CN CN ABCAB ZZ 298.000 298.000 LB US 400 United Stat KY
E2EDPT1 3000000001017945375000096E2EDPT1 000053030 0001E EN
E2EDPT2 3000000001017945375000097E2EDPT2 000096040 AB ABCABCABC Mouse forBus Bluetooth EN/XC/XD/XX Hdwr Black For Bsnss *
E2EDS01 3000000001017945375000098E2EDS01 000000020 0011
E2EDS01 3000000001017945375000099E2EDS01 000000020 01259064.20 CAD
E2EDS01 3000000001017945375000100E2EDS01 000000020 0056795.00 CAD
E2EDS01 3000000001017945375000101E2EDS01 000000020 01159064.20 CAD
E2EDS01 3000000001017945375000102E2EDS01 000000020 01052269.20 CAD
E2EDS01 3000000001017945375000103E2EDS01 000000020 94200000 CAD
E2EDS01 3000000001017945375000104E2EDS01 000000020 9440.00 CAD
E2EDS01 3000000001017945375000105E2EDS01 000000020 9450.00 CAD
E2EDS01 3000000001017945375000106E2EDS01 000000020 94659064.20 CAD
E2EDS01 3000000001017945375000107E2EDS01 000000020 94752269.20 CAD
E2EDS01 3000000001017945375000108E2EDS01 000000020 EXT
Z2XSK010003000000001017945375000109Z2XSK01000000108030 Z400 52269.20
Z2XSK010003000000001017945375000110Z2XSK01000000108030 XR1 13.000 6795.00 CX
</idocData></ReceiveIdoc>
```

### <a name="create-http-response-action"></a>Erstellen einer HTTP-Antwortaktion

Fügen Sie nun dem Workflow Ihrer Logik-App eine Antwortaktion hinzu, und fügen Sie die Ausgabe der SAP-Aktion ein. So gibt die Logik-App die Ergebnisse Ihres SAP-Servers an den ursprünglichen Anforderer zurück.

1. Wählen Sie im Logik-App-Designer unter der SAP-Aktion die Option **Neuer Schritt** aus.

1. Geben Sie im Suchfeld den Begriff `response` als Filter ein. Wählen Sie in der Liste **Aktionen** die Option **Antwort** aus.

1. Klicken Sie in das Feld **Hauptteil**, damit die dynamische Inhaltsliste angezeigt wird. Wählen Sie aus dieser Liste unter **Send message to SAP** (Nachricht an SAP senden) das Feld **Hauptteil** aus.

   ![Vollständige SAP-Aktion](./media/logic-apps-using-sap-connector/select-sap-body-for-response-action.png)

1. Speichern Sie Ihre Logik-App.

#### <a name="create-rfc-request-response"></a>Erstellen einer RFC-Anforderung/Antwort

> [!NOTE]
> Der SAP-Trigger empfängt IDocs per tRFC, und hierfür ist standardmäßig kein Antwortparameter vorhanden. 

Sie müssen ein Anforderung/Antwort-Muster erstellen, falls Sie Antworten empfangen müssen, indem Sie einen Remotefunktionsaufruf (Remote Function Call, RFC) an Logic Apps über SAP ABAP verwenden. Zum Empfangen von IDocs in Ihrer Logik-App sollten Sie als erste Aktion dafür eine [HTTP-Anforderung](../connectors/connectors-native-reqres.md#add-a-response-action) mit dem Statuscode `200 OK` und ohne Inhalt einrichten. Mit diesem empfohlenen Schritt wird die asynchrone SAP LUW-Übertragung per tRFC sofort abgeschlossen, und die SAP CPIC-Konversation ist wieder verfügbar. Sie können in Ihrer Logik-App dann weitere Aktionen hinzufügen, um das empfangene IDoc zu verarbeiten, ohne weitere Übertragungen zu blockieren.

Zum Implementieren eines Anforderung/Antwort-Musters müssen Sie zuerst mit dem [Befehl `generate schema`](#generate-schemas-for-artifacts-in-sap) das RFC-Schema ermitteln. Das generierte Schema verfügt über zwei mögliche Stammknoten: 

1. Den Anforderungsknoten, bei dem es um den Aufruf geht, den Sie von SAP erhalten.

1. Den Antwortknoten, bei dem es um Ihre Antwort an SAP geht.

Im folgenden Beispiel wird ein Anforderung/Antwort-Muster über das RFC-Modul `STFC_CONNECTION` generiert. Der XML-Code der Anforderung wird analysiert, um einen Knotenwert zu extrahieren, mit dem `<ECHOTEXT>` von SAP angefordert wird. In der Antwort wird der aktuelle Zeitstempel als dynamischer Wert eingefügt. Sie erhalten eine ähnliche Antwort, wenn Sie einen Remotefunktionsaufruf vom Typ `STFC_CONNECTION` aus einer Logik-App an SAP senden.

```http

<STFC_CONNECTIONResponse xmlns="http://Microsoft.LobServices.Sap/2007/03/Rfc/">
  <ECHOTEXT>@{first(xpath(xml(triggerBody()?['Content']), '/*[local-name()="STFC_CONNECTION"]/*[local-name()="REQUTEXT"]/text()'))}</ECHOTEXT>
  <RESPTEXT>Azure Logic Apps @{utcNow()}</RESPTEXT>


```

### <a name="test-logic-app"></a>Testen der Logik-App

1. Wenn Ihre Logik-App nicht bereits aktiviert ist, wählen Sie im Menü der Logik-App die Option **Überblick** aus. Wählen Sie auf der Symbolleiste **Aktivieren** aus.

1. Wählen Sie auf der Symbolleiste des Designers **Ausführen** aus. Dadurch wird die Logik-App manuell gestartet.

1. Lösen Sie Ihre Logik-App aus, indem Sie eine HTTP POST-Anforderung an die URL im HTTP-Anforderungstrigger senden.
Fügen Sie den Nachrichteninhalt in Ihre Anforderung ein. Um die Anforderung zu senden, können Sie ein Tool wie [Postman](https://www.getpostman.com/apps) nutzen.

   In diesem Artikel sendet die Anforderung eine IDoc-Datei, die im XML-Format vorliegen muss, und enthält den Namespace der SAP-Aktion, die Sie verwenden möchten, z.B.:

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <Send xmlns="http://Microsoft.LobServices.Sap/2007/03/Idoc/2/ORDERS05//720/Send">
      <idocData>
         <...>
      </idocData>
   </Send>
   ```

1. Nachdem Sie die HTTP-Anforderung gesendet haben, warten Sie auf die Antwort der Logik-App.

   > [!NOTE]
   > Es kann ein Timeout auftreten, wenn nicht alle für die Antwort erforderlichen Schritte innerhalb des [Timeoutlimits der Anforderung](./logic-apps-limits-and-config.md) abgeschlossen wurden. Wenn dies passiert, können Anforderungen blockiert werden. Um die Problemdiagnose zu vereinfachen, sollten Sie sich mit dem [Überprüfen und Überwachen Ihrer Logik-Apps](../logic-apps/monitor-logic-apps.md) vertraut machen.

Sie haben eine Logik-App erstellt, die mit dem SAP-Server kommunizieren kann. Da Sie eine SAP-Verbindung für die Logik-App eingerichtet haben, können Sie nun andere SAP Aktionen kennenlernen, z.B. BAPI und RFC.

## <a name="receive-message-from-sap"></a>Empfangen von Nachrichten von SAP

Dieses Beispiel verwendet eine Logik-App, die bei Empfang einer Nachricht von einem SAP-System ausgelöst wird.

### <a name="add-an-sap-trigger"></a>Hinzufügen eines SAP-Triggers

1. Erstellen Sie im Azure-Portal eine leere Logik-App, wodurch der Logik-App-Designer geöffnet wird.

1. Geben Sie im Suchfeld den Begriff `sap` als Filter ein. Wählen Sie in der Liste der **Trigger** die Option **Beim Empfang einer Nachricht von SAP** aus.

   ![Hinzufügen eines SAP-Triggers](./media/logic-apps-using-sap-connector/add-sap-trigger-logic-app.png)

   Alternativ können Sie die Registerkarte **Unternehmen** und dann den Auslöser auswählen:

   ![Hinzufügen eines SAP-Triggers auf der Registerkarte „Unternehmen“](./media/logic-apps-using-sap-connector/add-sap-trigger-ent-tab.png)

1. Sollte die Verbindung bereits bestehen, fahren Sie mit dem nächsten Schritt fort, damit Sie Ihre SAP-Aktion einrichten können. Wenn Sie jedoch zur Eingabe von Verbindungsdetails aufgefordert werden, sollten Sie die Informationen angeben, damit Sie eine Verbindung mit Ihrem lokalen SAP-Server herstellen können.

   1. Geben Sie einen Namen für die Verbindung an.

   1. Wenn Sie das Datengateway verwenden, führen Sie die folgenden Schritte aus:

      1. Wählen Sie im Abschnitt **Datengateway** unter **Abonnement** das Azure-Abonnement für die Datengatewayressource aus, die Sie im Azure-Portal für die Installation Ihres Datengateways erstellt haben.

      1. Wählen Sie unter **Verbindungsgateway** Ihre Datengatewayressource in Azure aus.

   1. Geben Sie weitere Informationen zu Verbindung an. Führen Sie die Schritte für die Eigenschaft **Anmeldetyp** abhängig davon aus, ob die Eigenschaft auf **Anwendungsserver** oder **Gruppe** festgelegt ist:

      * Bei **Anwendungsserver** sind die folgenden Eigenschaften erforderlich, die sonst optional sind:

        ![Verbindung zum SAP-Anwendungsserver herstellen](media/logic-apps-using-sap-connector/create-SAP-application-server-connection.png)

      * Bei **Gruppe** sind die folgenden Eigenschaften erforderlich, die sonst optional sind:

        ![Verbindung zum SAP-Nachrichtenserver herstellen](media/logic-apps-using-sap-connector/create-SAP-message-server-connection.png)

      Standardmäßig wird starke Typisierung verwendet, um nach ungültigen Werten zu suchen, indem eine XML-Validierung anhand des Schemas durchgeführt wird. Durch dieses Verhalten können Sie Probleme früher feststellen. Die Option **Sichere Typisierung** ist aus Gründen der Abwärtskompatibilität verfügbar und überprüft nur die Länge der Zeichenfolge. Erfahren Sie mehr über die Option [Sichere Typisierung](#safe-typing).

   1. Wählen Sie **Erstellen** aus, wenn Sie fertig sind.

      Die Verbindung wird in Logic Apps eingerichtet und getestet, sodass sichergestellt ist, dass sie ordnungsgemäß funktioniert.

1. Geben Sie die [erforderlichen Parameter](#parameters) basierend auf der Konfiguration Ihres SAP-Systems an. 

   Sie können [die Nachrichten, die Sie von Ihrem SAP-Server empfangen, filtern, indem Sie eine Liste der SAP-Aktionen angeben](#filter-with-sap-actions).

   Sie können eine SAP-Aktion aus der Dateiauswahl auswählen:

   ![Hinzufügen der SAP-Aktion zur Logik-App](media/logic-apps-using-sap-connector/select-SAP-action-trigger.png)  

   Sie können eine Aktion auch manuell angeben:

   ![Manuelles Eingeben der gewünschten SAP-Aktion](media/logic-apps-using-sap-connector/manual-enter-SAP-action-trigger.png)

   Dieses Beispiel stellt dar, wie die Aktion angezeigt wird, wenn Sie den Trigger für den Empfang mehrerer Nachrichten einrichten.

   ![Beispiel für einen Auslöser, der mehrere Nachrichten empfängt](media/logic-apps-using-sap-connector/example-trigger.png)

   Weitere Informationen zur SAP-Aktion finden Sie unter [Nachrichtenschemas für IDoc-Vorgänge](/biztalk/adapters-and-accelerators/adapter-sap/message-schemas-for-idoc-operations).

1. Speichern Sie jetzt Ihre Logik-App, um den Empfang von Nachrichten von Ihrem SAP-System zu starten. Wählen Sie auf der Symbolleiste des Designers **Speichern** aus.

Ihre Logik-App kann jetzt Nachrichten von Ihrem SAP-System empfangen.

> [!NOTE]
> Der SAP-Trigger ist kein Abfragetrigger, sondern er basiert auf Webhooks. Bei Verwendung des Datengateways wird der Trigger nur dann vom Datengateway aufgerufen, wenn eine Nachricht vorhanden ist. Abrufe sind daher nicht erforderlich.

Wenn Sie einen Fehler **Ungültiges Gateway (500)** mit einer ähnlichen Meldung wie **Dienst ‚sapgw00‘ unbekannt** erhalten, ersetzen Sie den Namen Ihres Gatewaydiensts in Ihrer API-Verbindung, und lösen Sie die Konfiguration mit seiner Portnummer aus. Im folgenden Beispielfehler muss `sapgw00` durch eine echte Portnummer ersetzt werden, z. B. `3300`. 

```json
{
    "body": {
        "error": {
            "code": 500,
            "source": "EXAMPLE-FLOW-NAME.eastus.environments.microsoftazurelogicapps.net",
            "clientRequestId": "00000000-0000-0000-0000-000000000000",
            "message": "BadGateway",
            "innerError": {
                "error": {
                    "code": "UnhandledException",
                    "message": "\nERROR service 'sapgw00' unknown\nTIME Wed Nov 11 19:37:50 2020\nRELEASE 721\nCOMPONENT NI (network interface)\nVERSION 40\nRC -3\nMODULE ninti.c\nLINE 933\nDETAIL NiPGetServByName: 'sapgw00' not found\nSYSTEM CALL getaddrinfo\nCOUNTER 1\n\nRETURN CODE: 20"
                }
            }
        }
    }
}
```

#### <a name="parameters"></a>Parameter

Neben einfachen Zeichenfolgen- und Zahleneingaben werden vom SAP-Connector auch die folgenden Tabellenparameter (`Type=ITAB`-Eingaben) akzeptiert:

* Direktionale Tabellenparameter (Ein- und Ausgabe) für ältere SAP-Releases

* Änderungsparameter, die die direktionalen Tabellenparameter bei neueren SAP-Releases ersetzen

* Hierarchische Tabellenparameter

#### <a name="filter-with-sap-actions"></a>Filtern mit SAP-Aktionen

Sie können die Nachrichten, die ihre Logik-App von Ihrem SAP-Server empfängt, filtern, indem Sie eine Liste oder ein Array mit einer einzelnen oder mehreren SAP-Aktionen bereitstellen. Standardmäßig ist dieses Array leer, was bedeutet, dass Ihre Logik-App alle Nachrichten von Ihrem SAP-Server ohne Filterung empfängt. 

Wenn Sie den Arrayfilter einrichten, empfängt der Trigger nur Nachrichten der angegebenen SAP-Aktionstypen und lehnt alle anderen Nachrichten von Ihrem SAP-Server ab. Dieser Filter beeinflusst jedoch nicht, ob die Typisierung der empfangenen Nutzlast schwach oder stark ist.

Jegliche SAP-Aktionsfilterung erfolgt auf Ebene des SAP-Adapters für Ihr lokales Datengateway. Weitere Informationen finden Sie unter [Senden von Test-IDocs an Logic-Apps von SAP](#test-sending-idocs-from-sap).

Wenn Sie keine IDoc-Pakete von SAP an den Trigger Ihrer Logik-App senden können, finden Sie weitere Informationen in der tRFC-Aufrufablehnungsmeldung (Transactional RFC) im tRFC-Dialogfeld von SAP (T-Code SM58). Auf der SAP-Oberfläche erhalten Sie unter Umständen die folgenden Fehlermeldungen, die aufgrund der Limits für Teilzeichenfolgen im Feld **Statustext** abgeschnitten sind.

##### <a name="the-requestcontext-on-the-ireplychannel-was-closed-without-a-reply-being-sent"></a>Der RequestContext für den IReplyChannel wurde geschlossen, ohne dass eine Antwort gesendet wurde.

Diese Fehlermeldung bedeutet, dass unerwartete Fehler auftreten, wenn der „Catch-All“-Handler für den Kanal den Kanal aufgrund eines Fehlers beendet und den Kanal neu erstellt, um weitere Meldungen zu verarbeiten.

Um zu bestätigen, dass Ihre Logik-App das IDoc empfangen hat, [fügen Sie eine Antwortaktion hinzu](../connectors/connectors-native-reqres.md#add-a-response-action), die den Statuscode `200 OK` zurückgibt. Lassen Sie den Text leer, ändern Sie die Header nicht, und fügen Sie ihnen auch nichts hinzu. Das IDoc wird über tRFC transportiert, was keine Antwortnutzlast zulässt.

Um stattdessen das IDoc abzulehnen, antworten Sie mit einem HTTP-Statuscode, der nicht `200 OK` ist. Der SAP-Adapter gibt dann in Ihrem Namen eine Ausnahme an SAP zurück. Sie sollten das IDoc nur ablehnen, um Transportfehler an SAP zurückzumelden, z. B. ein falsch geroutetes IDoc, das Ihre Anwendung nicht verarbeiten kann. Sie sollten ein IDoc nicht wegen Fehlern auf Anwendungsebene ablehnen, wie z. B. bei Problemen mit den im IDoc enthaltenen Daten. Wenn Sie die Annahme des Transports für die Validierung auf Anwendungsebene verzögern, kann es zu einer negativen Leistung kommen, weil Ihre Verbindung am Transport anderer IDocs gehindert wird.

Wenn Sie diese Fehlermeldung erhalten und systematische Fehler beim Aufrufen von Logic Apps auftreten, überprüfen Sie, ob Sie die Netzwerkeinstellungen für Ihren lokalen Datengatewaydienst für Ihre jeweilige Umgebung konfiguriert haben. Wenn Ihre Netzwerkumgebung beispielsweise die Verwendung eines Proxys zum Aufrufen von Azure-Endpunkten erfordert, müssen Sie Ihren lokalen Datengatewaydienst für die Verwendung Ihres Proxys konfigurieren. Weitere Informationen finden Sie unter [Proxykonfiguration](/dotnet/framework/network-programming/proxy-configuration).

Wenn Sie diese Fehlermeldung erhalten und zeitweilige Fehler beim Aufrufen von Logic Apps auftreten, müssen Sie möglicherweise die Wiederholungsanzahl und/oder das Wiederholungsintervall erhöhen. 

1. Überprüfen Sie die SAP-Einstellungen in der Konfigurationsdatei `Microsoft.PowerBI.EnterpriseGateway.exe.config` Ihres lokalen Datengateways. Die Einstellung für die Wiederholungsanzahl sieht wie folgt aus: `WebhookRetryMaximumCount="2"`. Die Einstellung für das Wiederholungsintervall sieht wie `WebhookRetryDefaultDelay="00:00:00.10"` aus und das Zeitraumformat ist `HH:mm:ss.ff`. 
    
1. Speichern Sie Ihre Änderungen, und starten Sie Ihr lokales Datengateway neu.

##### <a name="the-segment-or-group-definition-e2edk36001-was-not-found-in-the-idoc-meta"></a>Die Segment- oder Gruppendefinition E2EDK36001 wurde in den IDoc-Metadaten nicht gefunden.

Diese Fehlermeldung bedeutet, dass erwartete Ausfälle mit anderen Fehlern auftreten. Beispielsweise die Unfähigkeit, eine IDoc XML-Nutzlast zu generieren, weil ihre Segmente nicht von SAP freigegeben werden. Folglich fehlen die für die Konvertierung erforderlichen Segmenttypmetadaten.

Damit diese Segmente von SAP freigegeben werden, wenden Sie sich an den ABAP-Techniker Ihres SAP-Systems.
### <a name="asynchronous-request-reply-for-triggers"></a>Asynchrone Anforderung/Antwort für Trigger

Der SAP-Connector unterstützt das [asynchrone Anforderung/Antwort-Muster von Azure](/azure/architecture/patterns/async-request-reply) für Logic Apps-Trigger. Sie können dieses Muster verwenden, um erfolgreiche Anforderungen zu erstellen, die andernfalls mit dem standardmäßigen synchronen Anforderung/Antwort-Muster fehlschlagen würden. 

> [!TIP]
> In Logik-Apps mit mehreren Antwortaktionen müssen alle Antwortaktionen dasselbe Anforderung/Antwort-Muster verwenden. Wenn Ihre Logik-App z. B. ein Switch-Steuerelement mit mehreren möglichen Antwortaktionen verwendet, müssen Sie alle Antwortaktionen so konfigurieren, dass sie dasselbe Anforderung/Antwort-Muster verwenden, entweder synchron oder asynchron. 

Wenn Sie asynchrone Antworten für Ihre Antwortaktion aktivieren, kann Ihre Logik-App mit einer `202 Accepted`-Antwort antworten, wenn eine Anforderung für die Verarbeitung akzeptiert wurde. Die Antwort enthält einen location-Header, den Sie verwenden können, um den endgültigen Status Ihrer Anforderung abzurufen.

So konfigurieren Sie ein asynchrones Anforderung/Antwort-Muster für Ihre Logik-App mithilfe des SAP-Connectors

1. Öffnen Sie Ihre Logik-App im **Logik-App-Designer**.

1. Vergewissern Sie sich, dass der SAP-Connector der Trigger für Ihre Logik-App ist.

1. Öffnen Sie die **Antwort**-Aktion Ihrer Logik-App. Wählen Sie in der Titelleiste der Aktion das Menü ( **...** ) &gt; **Einstellungen** aus.

1. Aktivieren Sie in den **Einstellungen** für Ihre Antwortaktion die Umschaltfläche unter **Asynchrone Antwort**. Wählen Sie „Fertig“ aus.

1. Speichern Sie die Änderungen in Ihrer Logik-App.

## <a name="find-extended-error-logs"></a>Suchen erweiterter Fehlerprotokolle

Vollständige Fehlermeldungen finden Sie in den erweiterten Protokolle Ihres SAP-Adapters. Sie können auch [eine erweiterte Protokolldatei für den SAP-Connector aktivieren](#extended-sap-logging-in-on-premises-data-gateway).

Für lokale Datengatewayreleases ab Juni 2020 und später können Sie [Gatewayprotokolle in den App-Einstellungen aktivieren](/data-integration/gateway/service-gateway-tshoot#collect-logs-from-the-on-premises-data-gateway-app). 

* Der Standardprotokolliergrad ist **Warnung**.

* Wenn Sie **Zusätzliche Protokollierung** in den Einstellungen für die **Diagnose** der lokalen Datengateway-App aktivieren, wird der Protokolliergrad auf **Information** erhöht.

* Um den Protokolliergrad auf **Ausführlich** zu erhöhen, aktualisieren Sie die folgende Einstellung in Ihrer Konfigurationsdatei. Normalerweise befindet sich die Konfigurationsdatei unter `C:\Program Files\On-premises data gateway\Microsoft.PowerBI.DataMovement.Pipeline.GatewayCore.dll.config`.

```json
<setting name="SapTraceLevel" serializeAs="String">
   <value>Verbose</value>
</setting>
```

Für lokale Datengatewayreleases ab April 2020 und früher sind Protokolle standardmäßig deaktiviert.

### <a name="extended-sap-logging-in-on-premises-data-gateway"></a>Erweiterte SAP-Protokollierung im lokalen Datengateway

Wenn Sie ein [lokales Datengateway für Logic Apps](../logic-apps/logic-apps-gateway-install.md) verwenden, können Sie eine erweiterte Protokolldatei für den SAP-Connector konfigurieren. Sie können Ihr lokales Datengateway verwenden, um Ereignisse der Ereignisablaufverfolgung für Windows (Event Tracing for Windows, ETW) in rotierende Protokolldateien umzuleiten, die in die ZIP-Protokollierungsdateien Ihres Gateways eingeschlossen werden. 

Sie können [alle Konfigurations- und Dienstprotokolle Ihres Gateways](/data-integration/gateway/service-gateway-tshoot#collect-logs-from-the-on-premises-data-gateway-app) aus den Einstellungen der Gateway-App in eine ZIP-Datei exportieren.

> [!NOTE]
> Die erweiterte Protokollierung kann die Leistung Ihrer Logik-Apps beeinträchtigen, wenn sie immer aktiviert ist. Es ist eine bewährte Methode, erweiterte Protokolldateien zu deaktivieren, nachdem Sie mit der Analyse und Behebung eines Problems fertig sind.

#### <a name="capture-etw-events"></a>Erfassen von ETW-Ereignissen

Optional können fortgeschrittene Benutzer ETW-Ereignisse direkt erfassen. Sie können dann [anschließend Ihre Daten in Azure-Diagnose in Event Hubs verwenden](../azure-monitor/agents/diagnostics-extension-stream-event-hubs.md) oder [Ihre Daten in Azure Monitor-Protokollen sammeln](../azure-monitor/agents/diagnostics-extension-logs.md). Weitere Informationen finden Sie unter [Bewährte Methoden für das Sammeln und Speichern von Daten](/azure/architecture/best-practices/monitoring#collecting-and-storing-data). Sie können [PerfView](https://github.com/Microsoft/perfview/blob/master/README.md) verwenden, um mit den sich ergebenden ETL-Dateien zu arbeiten, oder Sie können ein eigenes Programm schreiben. In dieser exemplarischen Vorgehensweise wird PerfView verwendet:

1. Wählen Sie im PerfView-Menü **Erfassen** &gt; **Sammeln** aus, um die Ereignisse zu erfassen.

1. Geben Sie im Feld **Zusätzlicher Anbieter** `*Microsoft-LobAdapter` ein, um den SAP-Anbieter für die Erfassung von SAP-Adapterereignisse anzugeben. Wenn Sie diese Informationen nicht angeben, enthält Ihre Ablaufverfolgung nur allgemeine ETW-Ereignisse.

1. Übernehmen Sie alle anderen Standardeinstellungen. Wenn Sie möchten, können Sie den Dateinamen oder den Speicherort im Feld **Datendatei** ändern.

1. Wählen Sie **Sammlung starten** aus, um Ihre Ablaufverfolgung zu beginnen.

1. Nachdem Sie Ihr Problem reproduziert oder genügend Analysedaten gesammelt haben, wählen Sie **Sammlung beenden** aus.

Um Ihre Daten mit einer anderen Partei zu teilen, z. B. den Azure-Supporttechnikern, komprimieren Sie die ETL-Datei.

So zeigen Sie den Inhalt Ihrer Ablaufverfolgung an

1. Wählen Sie in PerfView **Datei** &gt; **Öffnen** aus, und wählen Sie die soeben generierte ETL-Datei aus.

1. In der PerfView-Randleiste wird der Abschnitt **Ereignisse** unter Ihrer ETL-Datei angezeigt.

1. Filtern Sie unter **Filter** nach `Microsoft-LobAdapter`, um nur die relevanten Ereignisse und Gatewayprozesse anzuzeigen.

### <a name="test-your-logic-app"></a>Testen Ihrer Logik-App

1. Um Ihre Logik-App auszulösen, senden Sie eine Nachricht von Ihrem SAP-System.

1. Wählen Sie in der Logik-App-Menü **Übersicht** aus. Überprüfen Sie den **Ausführungsverlauf** für alle neuen Ausführungen für Ihre Logik-App.

1. Öffnen Sie die letzte Ausführung, in der die von Ihrem SAP-System gesendete Nachricht im Abschnitt mit den Triggerausgaben angezeigt wird.

### <a name="test-sending-idocs-from-sap"></a>Testen des Sendens von IDocs von SAP

Um IDocs von SAP an Ihre Logik-App zu senden, benötigen Sie die folgende Mindestkonfiguration:

> [!IMPORTANT]
> Verwenden Sie diese Schritte nur, wenn Sie Ihre SAP-Konfiguration mit ihrer Logik-App testen. Produktionsumgebungen erfordern eine zusätzliche Konfiguration.

1. [Konfigurieren eines RFC-Ziels in SAP](#create-rfc-destination)

1. [Erstellen einer ABAP-Verbindung mit Ihrem RFC-Ziel](#create-abap-connection)

1. [Erstellen eines Empfangsports](#create-receiver-port)

1. [Erstellen eines Sendeports](#create-sender-port)

1. [Erstellen eines logischen Systempartners](#create-logical-system-partner)

1. [Erstellen eines Partnerprofils](#create-partner-profiles)

1. [Testen des Sendens von Nachrichten](#test-sending-messages)

#### <a name="create-rfc-destination"></a>Erstellen eines RFC-Ziels

1. Um die Einstellungen unter **Konfiguration von RFC-Verbindungen** zu öffnen, verwenden Sie auf Ihrer SAP-Oberfläche den Transaktionscode (T-Code) **sm59** mit dem Präfix **/n**.

1. Wählen Sie **TCP/IP-Verbindungen**  > **Erstellen** aus.

1. Erstellen Sie ein neues RFC-Ziel mit den folgenden Einstellungen:
    
    * Geben Sie für Ihr **RFC-Ziel** einen Namen ein.
    
    * Wählen Sie auf der Registerkarte **Technische Einstellungen** für **Aktivierungstyp** die Option **Registriertes Serverprogramm** aus. Geben Sie für Ihre **Programm-ID** einen Wert ein. Der Triggerwert Ihrer Logik-App wird in SAP mithilfe dieses Bezeichners registriert.

    > [!IMPORTANT]
    > Die SAP-**Programm-ID** berücksichtigt die Groß-/Kleinschreibung. Stellen Sie sicher, dass Sie beim Konfigurieren ihrer Logik-APP und des SAP-Servers für Ihre **Programm-ID** stets dieselbe Groß-/Kleinschreibung verwenden. Andernfalls erhalten Sie möglicherweise die folgenden Fehler im tRFC-Monitor (T-Code SM58), wenn Sie versuchen, ein IDoc an SAP zu senden:
    >
    > * **Funktion IDOC_INBOUND_ASYNCHRONOUS nicht gefunden**
    > * **Nicht-ABAP-RFC-Client (Partnertyp) nicht unterstützt**
    >
    > Weitere Informationen von SAP finden Sie in den folgenden Hinweisen (Anmeldung erforderlich): <https://launchpad.support.sap.com/#/notes/2399329> und <https://launchpad.support.sap.com/#/notes/353597>.
    
    * Wählen Sie auf der Registerkarte **Unicode** für **Kommunikationstyp mit Zielsystem** die Option **Unicode** aus.

1. Speichern Sie die Änderungen.

1. Registrieren Sie Ihre neue **Programm-ID** bei Azure Logic Apps.

1. Um die Verbindung zu testen, wählen Sie in der SAP-Oberfläche unter Ihrem neuen **RFC-Ziel** die Option **Verbindungstest** aus.

#### <a name="create-abap-connection"></a>Erstellen einer ABAP-Verbindung

1. Um die Einstellungen unter **Konfiguration von RFC-Verbindungen** zu öffnen, verwenden Sie auf Ihrer SAP-Oberfläche den Transaktionscode (T-Code) **sm59** _ mit dem Präfix _ */n**.

1. Wählen Sie **ABAP-Verbindungen**  > **Erstellen** aus.

1. Geben Sie für **RFC-Ziel** den Bezeichner für [Ihr SAP-Testsystem](#create-rfc-destination) ein.

1. Speichern Sie die Änderungen.

1. Um Ihre Verbindung zu testen, wählen Sie **Verbindungstest** aus.

#### <a name="create-receiver-port"></a>Erstellen eines Empfangsports

1. Um die Einstellungen unter **Ports in IDOC-Verarbeitung** zu öffnen, verwenden Sie auf Ihrer SAP-Oberfläche den Transaktionscode (T-Code) **we21** mit dem Präfix **/n**.

1. Wählen Sie **Ports** > **Transaktions-RFC** > **Erstellen** aus.

1. Wählen Sie im daraufhin geöffneten Feld den **eigenen Portnamen** aus. Geben Sie für Ihren Testport einen **Namen** ein. Speichern Sie die Änderungen.

1. Geben Sie in den Einstellungen für Ihren neuen Empfangsport für **RFC-Ziel** den Bezeichner für [Ihr RFC-Testziel](#create-rfc-destination) ein.

1. Speichern Sie die Änderungen.

#### <a name="create-sender-port"></a>Erstellen eines Sendeports

1.  Um die Einstellungen unter **Ports in IDOC-Verarbeitung** zu öffnen, verwenden Sie auf Ihrer SAP-Oberfläche den Transaktionscode (T-Code) **we21** mit dem Präfix **/n**.

1. Wählen Sie **Ports** > **Transaktions-RFC** > **Erstellen** aus.

1. Wählen Sie im daraufhin geöffneten Feld den **eigenen Portnamen** aus. Geben Sie für Ihren Testport einen **Namen** ein, der mit **SAP** beginnt. Alle Namen von Sendeports müssen mit den Buchstaben **SAP** beginnen, z. B. **SAPTEST**. Speichern Sie die Änderungen.

1. Geben Sie in den Einstellungen für Ihren neuen Sendeport für **RFC-Ziel** den Bezeichner für [Ihre ABAP-Verbindung](#create-abap-connection) ein.

1. Speichern Sie die Änderungen.

#### <a name="create-logical-system-partner"></a>Erstellen eines logischen Systempartners

1. Um die Einstellungen **Ansicht „Logische Systeme“ ändern: Übersicht** zu öffnen, verwenden Sie auf Ihrer SAP-Oberfläche den Transaktionscode (T-Code) **bd54**.

1. Akzeptieren Sie die angezeigte Warnmeldung: **Vorsicht: Die Tabelle ist clientübergreifend**.

1. Wählen Sie oberhalb der Liste, in der Ihre vorhandenen logischen Systeme angezeigt werden, **Neue Einträge** aus.

1. Geben Sie für Ihr neues logisches System einen **Log.System**-Bezeichner und eine kurze **Namen**-Beschreibung ein. Speichern Sie die Änderungen.

1. Wenn die **Aufforderung für Workbench** angezeigt wird, erstellen Sie eine neue Anforderung, indem Sie eine Beschreibung angeben, oder wenn Sie bereits eine Anforderung erstellt haben, überspringen Sie diesen Schritt.

1. Nachdem Sie die Workbench-Anforderung erstellt haben, verknüpfen Sie diese Anforderung mit der Tabellenaktualisierungsanforderung. Um zu bestätigen, dass Ihre Tabelle aktualisiert wurde, speichern Sie Ihre Änderungen.

#### <a name="create-partner-profiles"></a>Erstellen von Partnerprofilen

Für Produktionsumgebungen müssen Sie zwei Partnerprofile erstellen. Das erste Profil ist für den Absender, bei dem es sich um Ihre Organisation und Ihr SAP-System handelt. Das zweite Profil ist für den Empfänger, bei dem es sich um Ihre Logik-App handelt.

1. Um die Einstellungen unter **Partnerprofile** zu öffnen, verwenden Sie auf Ihrer SAP-Oberfläche den Transaktionscode (T-Code) **we20** mit dem Präfix **/n**.

1. Wählen Sie unter **Partnerprofile** die Option **Partnertyp-LS** > **Erstellen** aus.

1. Erstellen Sie ein neues Partnerprofil mit den folgenden Einstellungen:

    * Geben Sie für **Partner-Nr.** den [Bezeichner Ihres logischen Systempartners](#create-logical-system-partner) ein.

    * Geben Sie für **Partn. -Typ** den Wert **LS** ein.

    * Geben Sie für **Agent** den Bezeichner für das SAP-Benutzerkonto ein, das verwendet werden soll, wenn Sie Programmbezeichner für Azure Logic Apps oder andere Nicht-SAP-Systeme registrieren.

1. Speichern Sie die Änderungen. Wenn Sie [den logischen Systempartner nicht erstellt haben](#create-logical-system-partner), erhalten Sie die Fehlermeldung **Geben Sie eine gültige Partnernummer ein**.

1. Wählen Sie in den Einstellungen Ihres Partnerprofils unter **Ausgehende Parameter** die Option **Ausgehenden Parameter erstellen** aus.

1. Erstellen Sie einen neuen ausgehenden Parameter mit den folgenden Einstellungen:

    * Geben Sie Ihren **Nachrichtentyp** ein, z. B. **CREMAS**.

    * Geben Sie den [Bezeichner Ihre Empfangsports](#create-receiver-port) ein.

    * Geben Sie eine IDoc-Größe für **Paket größe** ein. Oder, um [IDocs einzeln von SAP zu senden](#receive-idoc-packets-from-sap), wählen Sie **IDoc sofort übergeben** aus.

1. Speichern Sie die Änderungen.

#### <a name="test-sending-messages"></a>Testen des Sendens von Nachrichten

1. Um die Einstellungen unter **Testtool für IDoc-Verarbeitung** zu öffnen, verwenden Sie auf Ihrer SAP-Oberfläche den Transaktionscode (T-Code) **we19** mit dem Präfix **/n**.

1. Wählen Sie unter **Vorlage für Test** die Option **Über Nachrichtentyp** aus, und geben Sie Ihren Nachrichtentyp ein, z. B. **CREMAS**. Klicken Sie auf **Erstellen**.

1. Bestätigen Sie die Meldung **Welcher IDoc-Typ?** , indem Sie **Weiter** auswählen.

1. Wählen Sie den Konten **EDIDC** aus. Geben Sie die geeigneten Werte für Ihren Empfangs- und Sendeport ein. Wählen Sie **Weiter**.

1. Wählen Sie **Standardmäßige ausgehende Verarbeitung** aus.

1. Um die ausgehende IDoc-Verarbeitung zu starten, wählen Sie **Weiter** aus. Bei Abschluss der Verarbeitung wird die Meldung **IDoc an SAP-System oder externes Programm gesendet** angezeigt.

1.  Um eine Überprüfung auf Verarbeitungsfehler durchzuführen, verwenden Sie den Transaktionscode (T-Code) **sm58** mit dem Präfix **/n**.

## <a name="receive-idoc-packets-from-sap"></a>Empfangen von IDoc-Paketen von SAP

Sie können SAP für das [Senden von IDoc-Paketen](https://help.sap.com/viewer/8f3819b0c24149b5959ab31070b64058/7.4.16/4ab38886549a6d8ce10000000a42189c.html) einrichten. Dabei handelt es sich um IDoc-Batches oder -Gruppen. Zum Empfangen von IDoc-Paketen ist für den SAP-Connector und insbesondere für den Trigger keine zusätzliche Konfiguration erforderlich. Damit Sie jedoch alle Elemente in einem IDoc-Paket verarbeiten können, das vom Trigger empfangen wurde, sind einige zusätzliche Schritte erforderlich, um das Paket in einzelne IDoc-Elemente aufzuteilen.

Im folgenden Beispiel wird veranschaulicht, wie die einzelnen IDoc-Elemente eines Pakets mithilfe der [`xpath()`-Funktion](./workflow-definition-language-functions-reference.md#xpath) extrahiert werden:

1. Zunächst benötigen Sie eine Logik-App mit einem SAP-Trigger. Wenn Sie dies in Ihrer Logik-App noch nicht haben, führen Sie die vorherigen Schritte in diesem Artikel aus, um [eine Logik-App mit einem SAP-Trigger](#receive-message-from-sap) einzurichten.

    > [!IMPORTANT]
    > Die SAP-**Programm-ID** berücksichtigt die Groß-/Kleinschreibung. Stellen Sie sicher, dass Sie beim Konfigurieren ihrer Logik-APP und des SAP-Servers für Ihre **Programm-ID** stets dieselbe Groß-/Kleinschreibung verwenden. Andernfalls erhalten Sie möglicherweise die folgenden Fehler im tRFC-Monitor (T-Code SM58), wenn Sie versuchen, ein IDoc an SAP zu senden:
    >
    > * **Funktion IDOC_INBOUND_ASYNCHRONOUS nicht gefunden**
    > * **Nicht-ABAP-RFC-Client (Partnertyp) nicht unterstützt**
    >
    > Weitere Informationen von SAP finden Sie in den folgenden Hinweisen (Anmeldung erforderlich): <https://launchpad.support.sap.com/#/notes/2399329> und <https://launchpad.support.sap.com/#/notes/353597>.

   Beispiel:

   ![Hinzufügen eines SAP-Auslösers zur Logik-App](./media/logic-apps-using-sap-connector/first-step-trigger.png)

1. [Fügen Sie Ihrer Logik-App eine Antwortaktion hinzu](../connectors/connectors-native-reqres.md#add-a-response-action), um sofort mit dem Status Ihrer SAP-Anforderung zu antworten. Es ist eine bewährte Methode, diese Aktion unmittelbar hinter Ihrem Trigger hinzuzufügen, um den Kommunikationskanal mit Ihrem SAP-Server freizugeben. Wählen Sie einen der folgenden Statuscodes (`statusCode`) aus, um ihn in Ihrer Antwortaktion zu verwenden:

    * **202 Akzeptiert**. Dies bedeutet, dass die Anforderung zur Verarbeitung angenommen wurde, die Verarbeitung aber noch nicht abgeschlossen ist.

    * **204 Kein Inhalt**. Dies bedeutet, dass der Server die Anforderung erfolgreich ausgeführt hat und es keinen zusätzlichen Inhalt gibt, der im Text der Antwortnutzlast gesendet werden kann. 

    * **200 OK**. Dieser Statuscode enthält immer eine Nutzlast, auch wenn der Server einen Nutzlasttext der Länge 0 (Null) generiert. 

1. Rufen Sie den Stammnamespace aus dem XML-IDoc-Element ab, das Ihre Logik-App von SAP empfängt. Fügen Sie zum Extrahieren dieses Namespace aus dem XML-Dokument einen Schritt hinzu, der eine lokale Zeichenfolgenvariable erstellt und diesen Namespace mithilfe eines `xpath()`-Ausdrucks speichert:

   `xpath(xml(triggerBody()?['Content']), 'namespace-uri(/*)')`

   ![Abrufen des Stammnamespace aus dem IDoc-Element](./media/logic-apps-using-sap-connector/get-namespace.png)

1. Fügen Sie zum Extrahieren eines einzelnen IDoc-Elements einen Schritt hinzu, in dem eine Arrayvariable erstellt und die IDoc-Sammlung mithilfe eines anderen `xpath()`-Ausdrucks gespeichert wird:

   `xpath(xml(triggerBody()?['Content']), '/*[local-name()="Receive"]/*[local-name()="idocData"]')`

   ![Abrufen eines Arrays von Elementen](./media/logic-apps-using-sap-connector/get-array.png)

   Die Arrayvariable stellt alle IDoc-Elemente durch Auflisten der Sammlung bereit, damit sie einzeln von Ihrer Logik-App verarbeitet werden können. In diesem Beispiel wird jedes IDoc-Element von der Logik-App mithilfe einer Schleife an einen SFTP-Server übertragen:

   ![Senden von IDoc-Elementen an einen SFTP-Server](./media/logic-apps-using-sap-connector/loop-batch.png)

   Der Stammnamespace muss in jedem IDoc-Element enthalten sein. Daher wird der Inhalt der Datei zusammen mit dem Stammnamespace von einem `<Receive></Receive`-Element umschlossen, bevor das IDoc-Element an die Downstream-App oder in diesem Fall an den SFTP-Server gesendet wird.

Sie können die Schnellstartvorlage für dieses Muster verwenden, indem Sie diese Vorlage im Logik-App-Designer auswählen, wenn Sie eine neue Logik-App erstellen.

![Auswählen der Vorlage Batch-Logik-App](./media/logic-apps-using-sap-connector/select-batch-logic-app-template.png)

## <a name="generate-schemas-for-artifacts-in-sap"></a>Generieren von Schemas für Artefakte in SAP

In diesem Beispiel wird eine Logik-App verwendet, die Sie mit einer HTTP-Anforderung auslösen können. Um die Schemas für das angegebene IDoc und BAPI zu generieren, sendet die SAP-Aktion **Schema generieren** eine Anforderung an ein SAP-System.

Diese SAP-Aktion gibt ein [XML-Schema](#sample-xml-schemas) und nicht den Inhalt oder die Daten des XML-Dokuments selbst zurück. In der Antwort zurückgegebene Schemas werden über den Azure Resource Manager-Connector in ein Integrationskonto hochgeladen. Schemas enthalten die folgenden Teile:

* Die Struktur der Anforderungsnachricht. Verwenden Sie diese Informationen, um Ihre `get`-BAPI-Liste zu erstellen.

* Die Struktur der Antwortnachricht. Verwenden Sie diese Informationen, um die Antwort zu analysieren. 

Um die Anforderungsnachricht zu senden, verwenden Sie die generische **SAP-Aktion Nachricht an SAP** senden oder die gezielte **\[BAPI]-Aufrufmethode in SAP**-Aktionen.

### <a name="sample-xml-schemas"></a>Beispiele für XML-Schemas

Falls Sie lernen möchten, wie Sie ein XML-Schema zur Verwendung beim Erstellen eines Beispieldokuments generieren, helfen Ihnen die folgenden Beispiele weiter. In diesen Beispielen wird veranschaulicht, wie Sie viele verschiedene Arten von Nutzlasten verwenden können, z. B.:

* [RFC-Anforderungen](#xml-samples-for-rfc-requests)

* [BAPI-Anforderungen](#xml-samples-for-bapi-requests)

* [IDoc-Anforderungen](#xml-samples-for-idoc-requests)

* Einfache oder komplexe XML-Schemadatentypen

* Tabellenparameter

* Optionales XML-Verhalten

Sie können Ihr XML-Schema mit einem optionalen XML-Prolog beginnen. Der SAP-Connector funktioniert mit oder ohne XML-Prolog.

```xml

<?xml version="1.0" encoding="utf-8">

```

#### <a name="xml-samples-for-rfc-requests"></a>XML-Beispiele für RFC-Anforderungen

Das folgende Beispiel ist ein einfacher Remotefunktionsaufruf (RFC). Der RFC-Name lautet `STFC_CONNECTION`. Für diese Anforderung wird der Standardnamespace `xmlns=` verwendet, aber Sie können auch Namespacealiase zuweisen und verwenden, z. B. `xmmlns:exampleAlias=`. Der Namespacewert ist der Namespace für alle Remotefunktionsaufrufe für Dienste vom Typ „SAP für Microsoft“. Die Anforderung `<REQUTEXT>` enthält einen einfachen Eingabeparameter.

```xml

<STFC_CONNECTION xmlns="http://Microsoft.LobServices.Sap/2007/03/Rfc/">
  <REQUTEXT>exampleInput</REQUTEXT>
</STFC_CONNECTION>

```

Das folgende Beispiel enthält einen Remotefunktionsaufruf mit einem Tabellenparameter. Dieser Beispielaufruf und die zugehörige Gruppe mit Test-RFCs sind im Rahmen aller SAP-Systeme verfügbar. Der Name des Tabellenparameters lautet `TCPICDAT`. Der Tabellenzeilentyp lautet `ABAPTEXT`, und dieses Element wird für jede Zeile der Tabelle wiederholt. Dieses Beispiel enthält nur eine Zeile, die den Namen `LINE` hat. Anforderungen mit einem Tabellenparameter können beliebig viele Felder enthalten, sofern die Anzahl eine positive ganze Zahl (*n*) ist. 

```xml

<STFC_WRITE_TO_TCPIC xmlns="http://Microsoft.LobServices.Sap/2007/03/Rfc/">
  <RESTART_QNAME>exampleQName</RESTART_QNAME>
    <TCPICDAT>
      <ABAPTEXT xmlns="http://Microsoft.LobServices.Sap/2007/03/Rfc/">
        <LINE>exampleFieldInput1</LINE>
      <ABAPTEXT xmlns="http://Microsoft.LobServices.Sap/2007/03/Rfc/">
        <LINE>exampleFieldInput2</LINE>
      <ABAPTEXT xmlns="http://Microsoft.LobServices.Sap/2007/03/Rfc/">
        <LINE>exampleFieldInput3</LINE>
      </ABAPTEXT>
    </TCPICDAT>
</STFC_WRITE_TO_TCPIC>

```

Das folgende Beispiel enthält einen Remotefunktionsaufruf mit einem Tabellenparameter, der über ein anonymes Feld verfügt. Ein Feld ist anonym, wenn dem Feld kein Name zugewiesen ist. Komplexe Typen werden unter einem separaten Namespace deklariert. Bei der Deklaration wird ein neuer Standardwert für den aktuellen Knoten und alle untergeordneten Elemente festgelegt. Im Beispiel wird der Hexadezimalcode `x002F` als Escapezeichen für das Symbol */* verwendet, weil dieses Symbol im SAP-Feldnamen reserviert ist.

```xml

<RFC_XML_TEST_1 xmlns="http://Microsoft.LobServices.Sap/2007/03/Rfc/">
  <IM_XML_TABLE>
    <RFC_XMLCNT xmlns="http://Microsoft.LobServices.Sap/2007/03/Rfc/">
      <_x002F_AnonymousField>exampleFieldInput</_x002F_AnonymousField>
    </RFC_XMLCNT>
  </IM_XML_TABLE>
</RFC_XML_TEST_1>

```

Das folgende Beispiel enthält Präfixe für die Namespaces. Sie können alle Präfixe gleichzeitig oder eine beliebige Anzahl von Präfixen als Attribute eines Knotens deklarieren. Der RFC-Namespacealias `ns0` wird als Stamm- und Parameterwert für den einfachen Typ genutzt.

> [!NOTE]
> Komplexe Typen werden unter einem anderen Namespace für RFC-Typen mit dem Alias `ns3` deklariert, anstatt unter dem regulären RFC-Namespace mit dem Alias `ns0`.

```xml

<ns0:BBP_RFC_READ_TABLE xmlns:ns0="http://Microsoft.LobServices.Sap/2007/03/Rfc/" xmlns:ns3="http://Microsoft.LobServices.Sap/2007/03/Types/Rfc/">
  <ns0:DELIMITER>0</ns0:DELIMITER>
  <ns0:QUERY_TABLE>KNA1</ns0:QUERY_TABLE>
  <ns0:ROWCOUNT>250</ns0:ROWCOUNT>
  <ns0:ROWSKIPS>0</ns0:ROWSKIPS>
  <ns0:FIELDS>
    <ns3:RFC_DB_FLD>
      <ns3:FIELDNAME>KUNNR</ns3:FIELDNAME>
    </ns3:RFC_DB_FLD>
  </ns0:FIELDS>
</ns0:BBP_RFC_READ_TABLE>

```

#### <a name="xml-samples-for-bapi-requests"></a>XML-Beispiele für BAPI-Anforderungen

Die folgenden XML-Beispiele sind Beispielanforderungen zum [Aufrufen der BAPI-Methode](#actions).

> [!NOTE]
> SAP macht Geschäftsobjekte für externe Systeme verfügbar, indem diese als Antwort auf den Remotefunktionsaufruf `RPY_BOR_TREE_INIT` beschrieben werden. Dieser Aufruf wird von Logic Apps ohne Eingabefilter ausgegeben. Logic Apps untersucht die Ausgabetabelle `BOR_TREE`. Das Feld `SHORT_TEXT` wird für Namen von Geschäftsobjekten verwendet. Geschäftsobjekte, die von SAP nicht in der Ausgabetabelle zurückgegeben werden, sind für Logic Apps nicht zugänglich.
> 
> Bei Verwendung von benutzerdefinierten Geschäftsobjekten müssen Sie sicherstellen, dass Sie diese in SAP veröffentlichen und als Release bereitstellen. Andernfalls listet SAP Ihre benutzerdefinierten Geschäftsobjekte in der Ausgabetabelle `BOR_TREE` nicht auf. Sie können in Logic Apps erst auf Ihre benutzerdefinierten Geschäftsobjekte zugreifen, nachdem Sie sie über SAP verfügbar gemacht haben. 

Im folgenden Beispiel wird eine Liste mit Banken abgerufen, indem die BAPI-Methode `GETLIST` verwendet wird. Dieses Beispiel enthält das Geschäftsobjekt für eine Bank: `BUS1011`. 

```xml

<GETLIST xmlns="http://Microsoft.LobServices.Sap/2007/03/Bapi/BUS1011">
  <BANK_CTRY>US</BANK_CTRY>
  <MAX_ROWS>10</MAX_ROWS>
</GETLIST>

```

Im folgenden Beispiel wird ein Bankobjekt mit der Methode `CREATE` erstellt. In diesem Beispiel wird das gleiche Geschäftsobjekt wie im vorherigen Beispiel verwendet: `BUS1011`. Wenn Sie die Methode `CREATE` zum Erstellen einer Bank verwenden, sollten Sie Ihre Änderungen unbedingt committen, weil dies bei dieser Methode nicht standardmäßig durchgeführt wird.

> [!TIP]
> Vergewissern Sie sich, dass für Ihr XML-Dokument alle Überprüfungsregeln eingehalten werden, die in Ihrem SAP-System konfiguriert sind. In diesem Beispieldokument muss der Bankschlüssel (`<BANK_KEY>`) beispielsweise eine Bankleitzahl sein (in den USA auch als „ABA Number“ bezeichnet).

```xml

<CREATE xmlns="http://Microsoft.LobServices.Sap/2007/03/Bapi/BUS1011">
  <BANK_ADDRESS>
    <BANK_NAME xmlns="http://Microsoft.LobServices.Sap/2007/03/Types/Rfc">ExampleBankName</BANK_NAME>
    <REGION xmlns="http://Microsoft.LobServices.Sap/2007/03/Types/Rfc">ExampleRegionName</REGION>
    <STREET xmlns="http://Microsoft.LobServices.Sap/2007/03/Types/Rfc">ExampleStreetAddress</STREET>
    <CITY xmlns="http://Microsoft.LobServices.Sap/2007/03/Types/Rfc">Redmond</CITY>
  </BANK_ADDRESS>
  <BANK_COUNTRY>US</BANK_COUNTRY>
  <BANK_KEY>123456789</BANK_KEY>
</CREATE>

```

Im folgenden Beispiel werden die Details für eine Bank über die Bankleitzahl (Wert für `<BANK_KEY>`) abgerufen. 

```xml

<GETDETAIL xmlns="http://Microsoft.LobServices.Sap/2007/03/Bapi/BUS1011">
  <BANK_COUNTRY>US</BANK_COUNTRY>
  <BANK_KEY>123456789</BANK_KEY>
</GETDETAIL>

```

#### <a name="xml-samples-for-idoc-requests"></a>XML-Beispiele für IDoc-Anforderungen

Verwenden Sie zum Generieren eines grundlegenden SAP IDoc-XML-Schemas die Anwendung für die **SAP-Anmeldung** und den T-Code `WE-60`. Greifen Sie auf die SAP-Dokumentation über die GUI zu, und generieren Sie XML-Schemas im XSD-Format für Ihre IDoc-Typen und -Erweiterungen. Eine Beschreibung der generischen SAP-Formate und -Nutzlasten sowie der integrierten Dialogfelder finden Sie in der [SAP-Dokumentation](https://help.sap.com/viewer/index).

In diesem Beispiel werden der Stammknoten und die Namespaces deklariert. Mit dem URI `http://Microsoft.LobServices.Sap/2007/03/Idoc/3/ORDERS05//700/Send` im Beispielcode wird die folgende Konfiguration deklariert:

* `/IDoc` ist der Stammknoten für alle IDocs.
* `/3` ist die Datensatztypversion für gängige Segmentdefinitionen.
* `/ORDERS05` ist der IDoc-Typ.
* `//` ist ein leeres Segment, weil keine IDoc-Erweiterung vorhanden ist.
* `/700` ist die SAP-Version.
* `/Send` ist die Aktion zum Senden der Informationen an SAP.

```xml

<ns0:Send xmlns:ns0="http://Microsoft.LobServices.Sap/2007/03/Idoc/3/ORDERS05//700/Send" xmlns:ns3="http://schemas.microsoft.com/2003/10/Serialization" xmlns:ns1="http://Microsoft.LobServices.Sap/2007/03/Types/Idoc/Common/" xmlns:ns2="http://Microsoft.LobServices.Sap/2007/03/Idoc/3/ORDERS05//700">
  <ns0:idocData>

```

Sie können den Knoten `idocData` wiederholen, um einen Batch mit IDocs in nur einem Aufruf zu senden. Das Beispiel unten enthält den Steuerungsdatensatz `EDI_DC40` und mehrere Datensätze mit Daten.

```xml

<...>
  <ns0:idocData>
    <ns2:EDI_DC40>
      <ns1:TABNAM>EDI_DC40</ns1:TABNAM>
<...>
      <ns1:ARCKEY>Cor1908207-5</ns1:ARCKEY>
    </ns2:EDI_DC40>
    <ns2:E2EDK01005>
      <ns2:DATAHEADERCOLUMN_SEGNAM>E23DK01005</ns2:DATAHEADERCOLUMN_SEGNAM>
      <ns2:CURCY>USD</ns2:CURCY>
    </ns2:E2EDK01005>
    <ns2:E2EDK03>
<...>
  </ns0:idocData>

```

Das folgende Beispiel enthält einen IDoc-Steuerungsdatensatz, für den das Präfix `EDI_DC` verwendet wird. Sie müssen die Werte so aktualisieren, dass sie zu Ihrer SAP-Installation und zum IDoc-Typ passen. Ihr IDoc-Clientcode darf beispielsweise nicht `800` lauten. Wenden Sie sich an Ihr SAP-Team, um sicherzustellen, dass Sie die richtigen Werte für Ihre SAP-Installation nutzen.

```xml

<ns2:EDI_DC40>
  <ns:TABNAM>EDI_DC40</ns1:TABNAM>
  <ns:MANDT>800</ns1:MANDT>
  <ns:DIRECT>2</ns1:DIRECT>
  <ns:IDOCTYP>ORDERS05</ns1:IDOCTYP>
  <ns:CIMTYP></ns1:CIMTYP>
  <ns:MESTYP>ORDERS</ns1:MESTYP>
  <ns:STD>X</ns1:STD>
  <ns:STDVRS>004010</ns1:STDVRS>
  <ns:STDMES></ns1:STDMES>
  <ns:SNDPOR>SAPENI</ns1:SNDPOR>
  <ns:SNDPRT>LS</ns1:SNDPRT>
  <ns:SNDPFC>AG</ns1:SNDPFC>
  <ns:SNDPRN>ABAP1PXP1</ns1:SNDPRN>
  <ns:SNDLAD></ns1:SNDLAD>
  <ns:RCVPOR>BTSFILE</ns1:RCVPOR>
  <ns:RCVPRT>LI</ns1:RCVPRT>

```

Das folgende Beispiel enthält einen Beispieldatensatz mit einfachen Segmenten. Hierbei wird das SAP-Datumsformat verwendet. Für stark typisierte Dokumente können native XML-Datumsformate verwendet werden, z. B. `2020-12-31 23:59:59`.

```xml

<ns2:E2EDK01005>
  <ns2:DATAHEADERCOLUMN_SEGNAM>E2EDK01005</ns2:DATAHEADERCOLUMN_SEGNAM>
    <ns2:CURCY>USD</ns2:CURCY>
    <ns2:BSART>OR</ns2:BSART>
    <ns2:BELNR>1908207-5</ns2:BELNR>
    <ns2:ABLAD>CC</ns2:ABLAD>
  </ns2>
  <ns2:E2EDK03>
    <ns2:DATAHEADERCOLUMN_SEGNAM>E2EDK03</ns2:DATAHEADERCOLUMN_SEGNAM>
      <ns2:IDDAT>002</ns2:IDDAT>
      <ns2:DATUM>20160611</ns2:DATUM>
  </ns2:E2EDK03>

```

Das folgende Beispiel enthält einen Datensatz mit gruppierten Segmenten. Der Datensatz enthält den übergeordneten Gruppenknoten `E2EDKT1002GRP` und mehrere untergeordnete Knoten, z. B. `E2EDKT1002` und `E2EDKT2001`. 

```xml

<ns2:E2EDKT1002GRP>
  <ns2:E2EDKT1002>
    <ns2:DATAHEADERCOLUMN_SEGNAM>E2EDKT1002</ns2:DATAHEADERCOLUMN_SEGNAM>
      <ns2:TDID>ZONE</ns2:TDID>
  </ns2:E2EDKT1002>
  <ns2:E2EDKT2001>
    <ns2:DATAHEADERCOLUMN_SEGNAM>E2EDKT2001</ns2:DATAHEADERCOLUMN_SEGNAM>
      <ns2:TDLINE>CRSD</ns2:TDLINE>
  </ns2:E2EDKT2001>
</ns2:E2EDKT1002GRP>

```


Die empfohlene Methode ist die Erstellung eines IDoc-Bezeichners für die Verwendung mit tRFC. Sie können diesen Transaktionsbezeichner `tid` festlegen, indem Sie in der SAP-Connector-API den [Vorgang zum Senden von IDocs](/connectors/sap/#send-idoc) nutzen.

Das folgende Beispiel enthält ein alternatives Verfahren zum Festlegen des Transaktionsbezeichners (`tid`). In diesem Beispiel werden der letzte Datensatz-Segmentknoten und der IDoc-Datenknoten geschlossen. Anschließend wird die GUID `guid` als tRFC-Bezeichner zum Erkennen von Duplikaten verwendet. 

```xml

    </E2STZUM002GRP>
  </idocData>
  <guid>8820ea40-5825-4b2f-ac3c-b83adc34321c</guid>
</Send>

```

### <a name="add-an-http-request-trigger"></a>Hinzufügen eines HTTP-Anforderungstriggers

1. Erstellen Sie im Azure-Portal eine leere Logik-App, wodurch der Logik-App-Designer geöffnet wird.

1. Geben Sie im Suchfeld den Begriff `http request` als Filter ein. Wählen Sie in der Liste der **Trigger** die Option **Beim Empfang einer HTTP-Anforderung** aus.

   ![Hinzufügen eines HTTP-Anforderungstriggers](./media/logic-apps-using-sap-connector/add-http-trigger-logic-app.png)

1. Speichern Sie jetzt Ihre Logik-App, damit Sie eine Endpunkt-URL dafür erstellen können.
Wählen Sie auf der Symbolleiste des Designers **Speichern** aus.

   Die Endpunkt-URL wird nun im Trigger angezeigt, z.B.:

   ![URL für den Endpunkt generieren](./media/logic-apps-using-sap-connector/generate-http-endpoint-url.png)

### <a name="add-an-sap-action-to-generate-schemas"></a>Hinzufügen einer SAP-Aktion zum Generieren von Schemas

1. Wählen Sie im Logik-App-Designer unter dem Trigger die Option **Neuer Schritt** aus.

   ![Hinzufügen eines neuen Schritts zum Logik-App](./media/logic-apps-using-sap-connector/add-sap-action-logic-app.png)

1. Geben Sie im Suchfeld den Begriff `sap` als Filter ein. Wählen Sie in der Liste mit den **Aktionen** die Aktion **Schemas generieren** aus.
  
   ![Hinzufügen der Aktion „Schemas generieren“ zur Logik-App](media/logic-apps-using-sap-connector/select-sap-schema-generator-action.png)

   Alternativ können Sie die Registerkarte **Unternehmen** und dann die SAP-Aktion auswählen.

   ![SAP-Sendeaktion auf der Registerkarte „Unternehmen“ auswählen](media/logic-apps-using-sap-connector/select-sap-schema-generator-ent-tab.png)

1. Sollte die Verbindung bereits bestehen, fahren Sie mit dem nächsten Schritt fort, damit Sie Ihre SAP-Aktion einrichten können. Wenn Sie jedoch zur Eingabe von Verbindungsdetails aufgefordert werden, sollten Sie die Informationen angeben, damit Sie eine Verbindung mit Ihrem lokalen SAP-Server herstellen können.

   1. Geben Sie einen Namen für die Verbindung an.

   1. Wählen Sie im Abschnitt **Datengateway** unter **Abonnement** das Azure-Abonnement für die Datengatewayressource aus, die Sie im Azure-Portal für die Installation Ihres Datengateways erstellt haben. 
   
   1. Wählen Sie unter **Verbindungsgateway** Ihre Datengatewayressource in Azure aus.

   1. Geben Sie weitere Informationen zu Verbindung an. Führen Sie die Schritte für die Eigenschaft **Anmeldetyp** abhängig davon aus, ob die Eigenschaft auf **Anwendungsserver** oder **Gruppe** festgelegt ist:
   
      * Bei **Anwendungsserver** sind die folgenden Eigenschaften erforderlich, die sonst optional sind:

        ![Verbindung zum SAP-Anwendungsserver herstellen](media/logic-apps-using-sap-connector/create-SAP-application-server-connection.png)

      * Bei **Gruppe** sind die folgenden Eigenschaften erforderlich, die sonst optional sind:

        ![Verbindung zum SAP-Nachrichtenserver herstellen](media/logic-apps-using-sap-connector/create-SAP-message-server-connection.png)  

      Standardmäßig wird starke Typisierung verwendet, um nach ungültigen Werten zu suchen, indem eine XML-Validierung anhand des Schemas durchgeführt wird. Durch dieses Verhalten können Sie Probleme früher feststellen. Die Option **Sichere Typisierung** ist aus Gründen der Abwärtskompatibilität verfügbar und überprüft nur die Länge der Zeichenfolge. Erfahren Sie mehr über die Option [Sichere Typisierung](#safe-typing).

   1. Wählen Sie **Erstellen** aus, wenn Sie fertig sind.

      Die Verbindung wird in Logic Apps eingerichtet und getestet, sodass sichergestellt ist, dass sie ordnungsgemäß funktioniert.

1. Geben Sie den Pfad zu dem Artefakt an, für das Sie das Schema generieren möchten.

   Sie können die SAP-Aktion über die Dateiauswahl auswählen:

   ![SAP-Aktion auswählen](media/logic-apps-using-sap-connector/select-SAP-action-schema-generator.png)  

   Alternativ dazu können Sie die Aktion manuell eingeben:

   ![SAP-Aktion manuell eingeben](media/logic-apps-using-sap-connector/manual-enter-SAP-action-schema-generator.png)

   Um Schemas für mehrere Artefakte zu generieren, geben Sie die Details der SAP-Aktion für jedes Artefakt an. Beispiel:

   ![„Neues Element hinzufügen“ auswählen](media/logic-apps-using-sap-connector/schema-generator-array-pick.png)

   ![Zwei Elemente anzeigen](media/logic-apps-using-sap-connector/schema-generator-example.png)

   Weitere Informationen zur SAP-Aktion finden Sie unter [Nachrichtenschemas für IDoc-Vorgänge](/biztalk/adapters-and-accelerators/adapter-sap/message-schemas-for-idoc-operations).

1. Speichern Sie Ihre Logik-App. Wählen Sie auf der Symbolleiste des Designers **Speichern** aus.

### <a name="test-your-logic-app"></a>Testen Ihrer Logik-App

1. Wählen Sie in der Designer-Symbolleiste die Option **Ausführen**, um eine Ausführung für Ihre Logik-App auszulösen.

1. Öffnen Sie die Ausführung, und überprüfen Sie die Ausgaben für die Aktion **Schemas generieren**.

   Die Ausgaben zeigen die generierten Schemas für die angegebene Liste von Nachrichten.

### <a name="upload-schemas-to-an-integration-account"></a>Hochladen von Schemas in ein Integrationskonto

Optional können Sie die generierten Schemas in Repositorys wie ein Blob-, Speicher- oder Integrationskonto herunterladen oder speichern. Integrationskonten eignen sich hervorragend für die Verwendung mit anderen XML-Aktionen. Daher zeigt dieses Beispiel, wie Sie Schemas mithilfe des Azure Resource Manager-Connectors in ein Integrationskonto für die gleiche Logik-App hochladen.

1. Wählen Sie im Logik-App-Designer unter dem Trigger die Option **Neuer Schritt** aus.

1. Geben Sie im Suchfeld den Begriff `Resource Manager` als Filter ein. Wählen Sie **Ressource erstellen oder aktualisieren** aus.

   ![Azure Resource Manager-Aktion auswählen](media/logic-apps-using-sap-connector/select-azure-resource-manager-action.png)

1. Geben Sie die Details für die Aktion einschließlich des Azure-Abonnements, der Azure-Ressourcengruppe und des Integrationskontos ein. Klicken Sie in die Felder für diese Felder, und wählen Sie aus der angezeigten dynamischen Inhaltsliste die SAP-Token aus, die den Feldern hinzugefügt werden sollen.

   1. Öffnen Sie die Liste **Neuen Parameter hinzufügen**, und wählen Sie die Felder **Speicherort** und **Eigenschaften** aus.

   1. Geben Sie wie in diesem Beispiel veranschaulicht Details für diese neuen Felder an.

      ![Details für Azure Resource Manager-Aktion eingeben](media/logic-apps-using-sap-connector/azure-resource-manager-action.png)

   Die SAP-Aktion **Schemas generieren** generiert Schemas als Sammlung, daher fügt der Designer der Aktion automatisch eine **For each**-Schleife hinzu. In diesem Beispiel wird die Darstellung dieser Aktion gezeigt:

   ![Azure Resource Manager-Aktion mit „for each“-Schleife](media/logic-apps-using-sap-connector/azure-resource-manager-action-foreach.png)

   > [!NOTE]
   > Die Schemas verwenden das Base64-codierte Format. Damit die Schemas in ein Integrationskonto hochgeladen werden können, müssen sie mithilfe der Funktion `base64ToString()` decodiert werden. Das folgende Beispiel zeigt den Code für das Element `"properties"`:
   >
   > ```json
   > "properties": {
   >    "Content": "@base64ToString(items('For_each')?['Content'])",
   >    "ContentType": "application/xml",
   >    "SchemaType": "Xml"
   > }
   > ```

1. Speichern Sie Ihre Logik-App. Wählen Sie auf der Symbolleiste des Designers **Speichern** aus.

### <a name="test-your-logic-app"></a>Testen Ihrer Logik-App

1. Wählen Sie in der Designer-Symbolleiste die Option **Ausführen**, um Ihre Logik-App manuell auszulösen.

1. Wechseln Sie nach einer erfolgreichen Ausführung zum Integrationskonto, und überprüfen Sie, ob die generierten Schemas vorhanden sind.

## <a name="enable-secure-network-communications"></a>Aktivieren von Secure Network Communications (SNC)

Vergewissern Sie sich zunächst, dass Sie die zuvor aufgeführten [Voraussetzungen](#prerequisites) erfüllen. Diese gelten jedoch nur, wenn Sie das Datengateway und Ihre Logik-Apps in einer mehrinstanzenfähigen Azure-Umgebung verwenden:

* Das lokale Datengateway muss auf einem Computer installiert sein, der sich im selben Netzwerk wie Ihr SAP-System befindet.

* Für SSO wird das Datengateway als Benutzer ausgeführt, der dem SAP-Benutzer zugeordnet ist.

* Die SNC-Bibliothek, die die zusätzlichen Sicherheitsfunktionen bereitstellt, wurde auf demselben Computer wie das Datengateway installiert. Einige Beispiele sind [sapseculib](https://help.sap.com/saphelp_nw74/helpdata/en/7a/0755dc6ef84f76890a77ad6eb13b13/frameset.htm), Kerberos und NTLM.

   Um SNC für Ihre Anforderungen an oder aus dem SAP-System zu aktivieren, aktivieren Sie das Kontrollkästchen **SNC verwenden** in der SAP-Verbindung, und geben Sie diese Eigenschaften an:

   ![Konfigurieren von SAP-SNC in der Verbindung](media/logic-apps-using-sap-connector/configure-sapsnc.png)

   | Eigenschaft | BESCHREIBUNG |
   |----------| ------------|
   | **SNC-Bibliothekspfad** | Der Name oder Pfad der SNC-Bibliothek, der relativ zum NCo-Installationspfad oder absoluten Pfad ist. Bespiele sind `sapsnc.dll`, `.\security\sapsnc.dll` oder `c:\security\sapsnc.dll`. |
   | **SNC SSO** | Bei einer Verbindung über SNC wird die SNC-Identität typischerweise zur Authentifizierung des Aufrufers verwendet. Eine weitere Möglichkeit besteht darin, die Benutzer- und Kennwortinformationen so zu überschreiben, dass sie für die Authentifizierung des Aufrufers verwendet werden können, die Verbindung aber dennoch verschlüsselt ist. |
   | **SNC Mein Name** | In den meisten Fällen kann diese Eigenschaft ausgelassen werden. Die installierte SNC-Lösung kennt in der Regel den eigenen SNC-Namen. Nur bei Lösungen, die mehrere Identitäten unterstützen, müssen Sie möglicherweise die Identität angeben, die für dieses spezielle Ziel oder diesen Server verwendet werden soll. |
   | **Name des SNC-Partners** | Name für das Back-End-SNC |
   | **SNC-Qualität des Schutzes** | Die Servicequalität, die für die SNC-Kommunikation dieses bestimmten Ziels oder Servers verwendet werden soll. Der Standardwert wird durch das Back-End-System definiert. Der Maximalwert wird durch das für SNC verwendete Sicherheitsprodukt definiert. |
   |||

   > [!NOTE]
   > Legen Sie die Umgebungsvariablen SNC_LIB und SNC_LIB_64 nicht auf dem Computer fest, auf dem Sie das Datengateway und die SNC-Bibliothek betreiben. Wenn sie festgelegt werden, haben sie Vorrang vor dem Wert der SNC-Bibliothek, der über den Connector übergeben wird.

## <a name="safe-typing"></a>Sichere Typisierung

Standardmäßig wird beim Erstellen der SAP-Verbindung starke Typisierung verwendet, um nach ungültigen Werten zu suchen, indem eine XML-Validierung anhand des Schemas durchgeführt wird. Durch dieses Verhalten können Sie Probleme früher feststellen. Die Option **Sichere Typisierung** ist aus Gründen der Abwärtskompatibilität verfügbar und überprüft nur die Länge der Zeichenfolge. Wenn Sie **Sichere Typisierung** auswählen, werden die DATS- und TIMS-Typen in SAP als Zeichenfolgen und nicht als ihre XML-Entsprechungen (`xs:date` und `xs:time`, `xmlns:xs="http://www.w3.org/2001/XMLSchema"`) behandelt. Die sichere Typisierung beeinflusst das Verhalten für die gesamte Schemagenerierung, die Sendenachricht für die Nutzlast „wurde gesendet“ und die Antwort „wurde empfangen“ sowie den Trigger. 

Wenn starke Typisierung verwendet wird (**Sichere Typisierung** ist nicht aktiviert) ordnet das Schema die DATS-und TIMS-Typen einfacheren XML-Typen zu:

```xml
<xs:element minOccurs="0" maxOccurs="1" name="UPDDAT" nillable="true" type="xs:date"/>
<xs:element minOccurs="0" maxOccurs="1" name="UPDTIM" nillable="true" type="xs:time"/>
```

Beim Senden von Nachrichten mit starker Typisierung entspricht die DATS- und TIMS-Antwort dem entsprechenden XML-Typformat:

```xml
<DATE>9999-12-31</DATE>
<TIME>23:59:59</TIME>
```

Wenn **Sichere Typisierung** aktiviert ist, ordnet das Schema die DATS- und TIMS-Typen nur XML-Zeichenfolgenfeldern mit Längeneinschränkungen zu, z.B.:

```xml
<xs:element minOccurs="0" maxOccurs="1" name="UPDDAT" nillable="true">
  <xs:simpleType>
    <xs:restriction base="xs:string">
      <xs:maxLength value="8" />
    </xs:restriction>
  </xs:simpleType>
</xs:element>
<xs:element minOccurs="0" maxOccurs="1" name="UPDTIM" nillable="true">
  <xs:simpleType>
    <xs:restriction base="xs:string">
      <xs:maxLength value="6" />
    </xs:restriction>
  </xs:simpleType>
</xs:element>
```

Beim Senden von Nachrichten mit aktivierter **Sicherer Typisierung** sieht die DATS- und TIMS-Antwort wie im folgenden Beispiel aus:

```xml
<DATE>99991231</DATE>
<TIME>235959</TIME>
```

## <a name="advanced-scenarios"></a>Erweiterte Szenarien

### <a name="change-language-headers"></a>Ändern von Sprachheadern

Wenn Sie von Logic Apps aus eine Verbindung mit SAP herstellen, ist die Standardsprache für die Verbindung Englisch. Sie können die Sprache für Ihre Verbindung festlegen, indem Sie [den HTTP-Standardheader `Accept-Language`](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.4) mit Ihren eingehenden Anforderungen verwenden.

> [!TIP]
> Die meisten Webbrowser fügen auf Grundlage der Einstellungen des Benutzers einen `Accept-Language`-Header hinzu. Der Webbrowser wendet diesen Header an, wenn Sie eine neue SAP-Verbindung im Logic Apps-Designer erstellen. Wenn Sie keine SAP-Verbindungen in der bevorzugten Sprache Ihres Webbrowsers erstellen möchten, aktualisieren Sie entweder die Einstellungen Ihres Webbrowsers so, dass Ihre bevorzugte Sprache verwendet wird, oder Sie erstellen Ihre SAP-Verbindung mithilfe von Azure Resource Manager anstelle des Logic Apps-Designers. 

Beispielsweise können Sie eine Anforderung mit dem `Accept-Language`-Header an Ihre Logik-App senden, indem Sie den **HTTP-Anforderung**-Trigger verwenden. Alle Aktionen in ihrer Logik-App empfangen den Header. SAP verwendet dann die angegebenen Sprachen in seinen Systemmeldungen, z. B. in BAPI-Fehlermeldungen.

Die SAP-Verbindungsparameter für eine Logik-App verfügen über keine Spracheigenschaft. Wenn Sie also den `Accept-Language`-Header verwenden, erhalten Sie unter Umständen die folgende Fehlermeldung: **Überprüfen Sie Ihre Kontoinformationen und/oder Berechtigungen, und versuchen Sie es noch mal.** Überprüfen Sie in diesem Fall stattdessen die Fehlerprotokolle der SAP-Komponente. Der Fehler tritt tatsächlich in der SAP-Komponente auf, die den Header verwendet, sodass Sie möglicherweise eine der folgenden Fehlermeldungen erhalten:

* `"SAP.Middleware.Connector.RfcLogonException: Select one of the installed languages"`
* `"SAP.Middleware.Connector.RfcAbapMessageException: Select one of the installed languages"`

### <a name="confirm-transaction-explicitly"></a>Explizites Bestätigen der Transaktion

Wenn Sie Transaktionen von Logic Apps an SAP senden, erfolgt dieser Austausch in zwei Schritten, wie im SAP-Dokument [Transactional RFC Server Programs](https://help.sap.com/doc/saphelp_nwpi71/7.1/22/042ad7488911d189490000e829fbbd/content.htm?no_cache=true) (Transaktionsbezogene RFC-Serverprogramme) beschrieben. Von der Aktion **Send to SAP** (An SAP senden) werden standardmäßig sowohl die Schritte für die Funktionsübertragung als auch die Schritte für die Transaktionsbestätigung in einem einzelnen Aufruf abgewickelt. Der SAP-Connector ermöglicht die Entkoppelung dieser Schritte. Sie können ein IDoc-Element senden und anstatt der automatischen Transaktionsbestätigung die explizite Aktion **\[Transaktions-ID bestätigen** verwenden.

Diese Entkoppelung der Transaktions-ID-Bestätigung ist hilfreich, wenn Sie in SAP keine Transaktionen duplizieren möchten (etwa in Szenarien, in denen beispielsweise Fehler aufgrund von Netzwerkproblemen auftreten können). Durch die separate Bestätigung der Transaktions-ID wird die Transaktion nur einmal in Ihrem SAP-System durchgeführt.

Hier sehen Sie ein Beispiel für dieses Muster:

1. Erstellen Sie eine leere Logik-App, und fügen Sie einen HTTP-Trigger hinzu.

1. Fügen Sie im SAP-Konnektor die Aktion **\[IDOC] Dokument an SAP** senden hinzu. Geben Sie die Details für das IDoc-Element an, das Sie an Ihr SAP-System senden.

1. Wählen Sie im Feld **TID bestätigen** die Option **Nein** aus, um die Transaktions-ID explizit in einem separaten Schritt zu bestätigen. Für das optionale Feld **Transaktions-ID-GUID** können Sie den Wert entweder manuell angeben oder diese GUID automatisch durch den Connector generieren und in der Antwort auf die Aktion „IDoc senden“ zurückgeben lassen.

   ![Eigenschaften der Aktion „Send IDOC“ (IDOC senden)](./media/logic-apps-using-sap-connector/send-idoc-action-details.png)

1. Fügen Sie zum expliziten Bestätigen der Transaktions-ID die Aktion **\[Confirm transaction ID** (Transaktions-ID bestätigen) hinzu, und achten Sie darauf, dass [Sie das Senden von doppelten IDocs an SAP](#avoid-sending-duplicate-idocs) vermeiden. Klicken Sie auf das Feld **Transaction ID** (Transaktions-ID), um die dynamische Inhaltsliste anzuzeigen. Wählen Sie in dieser Liste den Wert der **Transaktions-ID** aus, der von der Aktion **\[IDOC] Dokument an SAP** senden zurückgegeben wird.

   ![Aktion zum Bestätigen der Transaktions-ID](./media/logic-apps-using-sap-connector/explicit-transaction-id.png)

   Im Anschluss an diesen Schritt wird die aktuelle Transaktion auf beiden Seiten (also aufseiten des SAP-Connectors und aufseiten des SAP-Systems) als abgeschlossen markiert.

#### <a name="avoid-sending-duplicate-idocs"></a>Vermeiden des Sendens von doppelten IDocs

Falls bei Ihnen ein Problem aufgrund des Sendens von doppelten IDocs an SAP über Ihre Logik-App besteht, sollten Sie diese Schritte zum Erstellen einer Zeichenfolgenvariablen befolgen, die als Ihr IDoc-Transaktionsbezeichner dienen kann. Die Erstellung dieses Transaktionsbezeichners verhindert doppelte Netzwerkübertragungen, wenn es zu Problemen wie vorübergehenden Ausfällen, Netzwerkproblemen oder verlorenen gegangenen Bestätigungen kommt.

> [!NOTE]
> In SAP-Systemen wird ein Transaktionsbezeichner nach einer bestimmten Zeit vergessen (standardmäßig nach 24 Stunden). Aus diesem Grund wird bei SAP niemals die Bestätigung eines Transaktionsbezeichners übersehen, falls die ID oder GUID unbekannt ist.
> Wenn die Bestätigung für einen Transaktionsbezeichner nicht erfolgt, ist dies ein Hinweis darauf, dass bei der Kommunikation mit dem SAP-System ein Fehler aufgetreten ist, bevor die Bestätigung von SAP verifiziert werden konnte.

1. Fügen Sie im Logik-Apps-Designer Ihrer Logik-App die Aktion **Variable initialisieren** hinzu. 

1. Konfigurieren Sie im Editor für die Aktion **Variable initialisieren** die folgenden Einstellungen. Speichern Sie anschließend die Änderungen.

    1. Geben Sie unter **Name** einen Namen für Ihre Variable ein. Beispiel: `IDOCtransferID`.

    1. Wählen Sie unter **Typ** die Option **Zeichenfolge** als Variablentyp aus.

    1. Wählen Sie unter **Wert** das Textfeld **Anfangswert eingeben** aus, um das Menü für dynamischen Inhalt zu öffnen. Wählen Sie die Registerkarte **Ausdrücke** aus. Geben Sie in der Liste mit den Funktionen die Funktion `guid()` ein. Wählen Sie anschließend **OK** aus, um Ihre Änderungen zu speichern. Das Feld **Wert** ist jetzt auf die Funktion `guid()` festgelegt, mit der eine GUID generiert wird.

1. Fügen Sie nach der Aktion **Variable initialisieren** die Aktion  **\[ IDOC] Dokument an SAP senden hinzu**.

1. Nehmen Sie im Editor für die Aktion **\[IDOC] Dokument an SAP**, senden die folgenden Einstellungen vor. Speichern Sie anschließend die Änderungen.

    1. Wählen Sie unter **IDOC type** (IDoc-Typ) Ihren Nachrichtentyp aus, und geben Sie unter **Input IDOC message** (IDoc-Nachricht eingeben) Ihre Nachricht an.

    1. Wählen Sie unter **SAP release version** (Version des SAP-Release) die Werte Ihrer SAP-Konfiguration aus.

    1. Wählen Sie unter **Record types version** (Datensatztypversion) die Werte Ihrer SAP-Konfiguration aus.

    1. Wählen Sie unter **Confirm TID** (Transaktions-ID bestätigen) die Option **Nein** aus.

    1. Wählen Sie **Liste „Neuen Parameter hinzufügen“**  > **Transaction ID GUID** (GUID der Transaktions-ID) aus. Wählen Sie das Textfeld aus, um das Menü für dynamischen Inhalt zu öffnen. Wählen Sie auf der Registerkarte **Variablen** den Namen der Variablen aus, die Sie erstellt haben. Beispiel: `IDOCtransferID`.

1. Wählen Sie in der Titelleiste der Aktion **\[IDOC] Dokument an SAP** senden, wählen Sie **...**  > **Einstellungen**. Es wird empfohlen, für **Wiederholungsrichtlinie** den Wert **Standard** &gt; **Fertig** auszuwählen. Sie können jedoch stattdessen eine benutzerdefinierte Richtlinie für Ihre spezifischen Anforderungen konfigurieren. Bei benutzerdefinierten Richtlinien wird empfohlen, mindestens einen Wiederholungsversuch zu konfigurieren, um vorübergehende Netzwerkausfälle zu überwinden.

1. Fügen Sie nach der Aktion **\[ IDOC] Dokument an SAP senden** die Aktion **\[ IDOC] Transaktions-ID bestätigen hinzu**.

1. Konfigurieren Sie im Editor für die Aktion **\[IDOC] Confirm transaction**, ID (Transaktions-ID bestätigen) die folgenden Einstellungen. Speichern Sie anschließend die Änderungen.

    1. Geben Sie für **Transaktions-ID** den Namen Ihrer Variablen erneut ein. Beispiel: `IDOCtransferID`.

1. Optional können Sie die Deduplizierung in Ihrer Testumgebung überprüfen. Wiederholen Sie die **Aktion \[IDOC] Dokument an SAP** senden mit der gleichen **Transaktions-ID-** GUID, die Sie im vorherigen Schritt verwendet haben. Wenn Sie dasselbe IDoc zweimal senden, können Sie überprüfen, ob SAP die Duplizierung des tRFC-Aufrufs identifizieren und die beiden Aufrufe einer einzelnen eingehenden IDoc-Nachricht auflösen kann.

## <a name="known-issues-and-limitations"></a>Einschränkungen und bekannte Probleme

Zurzeit sind für den verwalteten SAP-Connector (Nicht-ISE) folgende Probleme und Einschränkungen bekannt: 

* Im allgemeinen unterstützt der SAP-Trigger keine Datengatewaycluster. In einigen Failoverfällen unterscheidet sich der Datengatewayknoten, der mit dem SAP-System kommuniziert, von dem aktiven Knoten. Dies führt zu unerwartetem Verhalten.

  * In Sendeszenarien werden Datengatewaycluster im Failovermodus unterstützt. 

  * Datengatewaycluster im Lastenausgleichsmodus werden von zustandsbehafteten[SAP-Aktionen](#actions) nicht unterstützt. Zu diesen Aktionen gehören **\[BAPI - RFC] Anlegen einer zustandsbehafteten Sitzung,** , **\[BAPI] Commit-Transaktion,** , **\[BAPI] Rollback-Transaktion,** , **\[ BAPI - RFC] Schließen einer zustandsbehafteten Sitzung und alle Aktionen**, die einen **Sitzungs-ID**-Wert angeben. Zustandsbehaftete Kommunikationen müssen auf demselben Datengatewayclusterknoten verbleiben. 

  * Verwenden Sie für zustandsbehaftete SAP-Aktionen das Datengateway entweder im Nicht-Clustermodus oder in einem Cluster, der nur für Failover eingerichtet ist.

* Der SAP-Connector unterstützt derzeit keine SAP-Router-Zeichenfolgen. Das lokale Datengateway muss in demselben LAN wie das SAP-System vorhanden sein, mit dem eine Verbindung hergestellt werden soll.

* Für [Logik-Apps in einer ISE](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md) verwendet die mit ISE bezeichnete Version dieses Connectors stattdessen die [ISE-Nachrichtengrenzwerte](../logic-apps/logic-apps-limits-and-config.md#message-size-limits).

## <a name="connector-reference"></a>Connector-Referenz 

Weitere Informationen zum SAP-Konnektor finden Sie in der [Konnektorreferenz.](/connectors/sap/) Hier finden Sie Details zu Limits, Parametern und Rückgaben für den SAP-Konnektor, Trigger und Aktionen.

### <a name="triggers"></a>Trigger

:::row:::
    :::column span="1":::
        [**Wenn eine Nachricht von SAP empfangen wird**](/connectors/sap/#when-a-message-is-received)
    :::column-end:::
    :::column span="3":::
        Wenn eine Nachricht von SAP empfangen wird, tun Sie etwas. 
    :::column-end:::
:::row-end:::

### <a name="actions"></a>Aktionen

:::row:::
    :::column span="1":::
        [ **[BAPI - RFC] Stateful-Sitzung schließen**](/connectors/sap/#[bapi---rfc]-close-stateful-session-(preview))
    :::column-end:::
    :::column span="3":::
        Schließen Sie eine bestehende Zustandsbehaftete-Verbindungssitzung zu Ihrem SAP-System.
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="1":::
        [ **[BAPI - RFC] Zustandsbehaftete Sitzung erstellen**](/connectors/sap/#[bapi---rfc]-create-stateful-session-(preview))
    :::column-end:::
    :::column span="3":::
        Erstellt eine zustandsbehaftete Verbindungssitzung mit Ihrem SAP-System.
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="1":::
        [ **[BAPI] Aufrufen der Methode in SAP**](/connectors/sap/#[bapi]-call-method-in-sap-(preview))
    :::column-end:::
    :::column span="3":::
        Rufen Sie die BAPI-Methode in Ihrem SAP-System auf.
        \
        \
        Sie müssen die folgenden Parameter mit Ihrem Aufruf verwenden:     \
        \
        **Geschäftsobjekt** (`businessObject`): Hierbei handelt es sich um ein durchsuchbares Dropdownmenü.
        \
        \
        **Methode** (`method`): Hiermit werden die verfügbaren Methoden aufgefüllt, nachdem Sie ein **Geschäftsobjekt** ausgewählt haben. Die verfügbaren Methoden sind vom ausgewählten **Geschäftsobjekt** abhängig.
        \
        \
        **BAPI-Eingabeparameter** (`body`): Hierin rufen Sie das XML-Dokument auf, das die Eingabeparameterwerte der BAPI-Methode für den Befehl enthält, oder den URI des Speicherblobs, das Ihre BAPI-Parameter enthält.
        \
        \
        Detaillierte Beispiele für die Verwendung der Methode **[BAPI] Call in SAP** Action finden Sie in den [XML-Beispielen für BAPI-Anforderungen](#xml-samples-for-bapi-requests). 
        \
        Bei Verwendung des Logik-Apps-Designers zum Bearbeiten Ihrer BAPI-Anforderung können Sie die folgenden Suchfunktionen nutzen:     \
        \
        Wählen Sie im Designer ein Objekt aus, um ein Dropdownmenü mit den verfügbaren Methoden anzuzeigen.
        \
        \
        Filtern Sie Geschäftsobjekttypen nach Schlüsselwort, indem Sie die durchsuchbare Liste verwenden, die über den BAPI-API-Aufruf bereitgestellt wird.
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="1":::
        [ **[BAPI] Vorgang committen**](/connectors/sap/#[bapi]-commit-transaction-(preview))
    :::column-end:::
    :::column span="3":::
        BAPI-Transaktion für die Sitzung committen.
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="1":::
        [ **[BAPI] Rollback der Transaktion**](/connectors/sap/#[bapi]-roll-back-transaction-(preview))
    :::column-end:::
    :::column span="3":::
        Rollback für die BAPI-Transaktion für die Sitzung ausführen.
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="1":::
        [ **[IDOC - RFC] Transaktions-ID bestätigen**](/connectors/sap/#[idoc---rfc]-confirm-transaction-id-(preview))
    :::column-end:::
    :::column span="3":::
        Senden Sie die Bestätigung der Transaktionskennung an SAP.
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="1":::
        [ **[IDOC] IDOC-Liste für Transaktion erhalten**](/connectors/sap/#[idoc]-get-idoc-list-for-transaction-(preview))
    :::column-end:::
    :::column span="3":::
        Eine Liste der IDocs für die Transaktion nach Sitzungskennzeichen oder Transaktionskennzeichen abrufen.
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="1":::
        [ **[IDOC] IDOC-Status erhalten**](/connectors/sap/#[idoc]-get-idoc-status-(preview))
    :::column-end:::
    :::column span="3":::
        Holt den Status eines IDocs.
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="1":::
        [ **[IDOC] Dokument an SAP senden**](/connectors/sap/#[idoc]-send-document-to-sap-(preview))
    :::column-end:::
    :::column span="3":::
        Sendet die IDoc-Nachricht an Ihren SAP-Server.
        \
        \
        Sie müssen die folgenden Parameter mit Ihrem Aufruf verwenden:     \
        \
        **IDoc-Typ mit optionaler Erweiterung** (`idocType`): Hierbei handelt es sich um ein durchsuchbares Dropdownmenü.
        \
        \
        **IDOC-Eingabenachricht** (`body`): Hierin rufen Sie das XML-Dokument auf, das die IDoc-Nutzlast enthält, oder den URI des Speicherblobs, das Ihr IDoc-XML-Dokument enthält. Dieses Dokument muss entweder dem SAP IDOC-XML-Schema gemäß der WE60 IDoc-Dokumentation oder dem generierten Schema für den entsprechenden SAP IDoc-Aktions-URI entsprechen.
        \
        \
        Der optionale Parameter **SAP-Releaseversion** (`releaseVersion`) füllt Werte auf, nachdem Sie den IDoc-Typ ausgewählt haben, und ist vom ausgewählten IDoc-Typ abhängig.
        \
        \
        Ausführliche Beispiele für die Verwendung der Aktion „IDoc senden“ finden Sie in der [exemplarischen Vorgehensweise zum Senden von IDoc-Nachrichten an Ihren SAP-Server](#send-idoc-messages-to-sap-server).
        \
        \
        Informationen zum Verwenden des optionalen Parameters **TID bestätigen** (`confirmTid`) finden Sie in der [exemplarischen Vorgehensweise zur expliziten Bestätigung der Transaktion](#confirm-transaction-explicitly).
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="1":::
        [ **[RFC] RFC zur Transaktion hinzufügen**](/connectors/sap/#[rfc]-add-rfc-to-transaction-(preview))
    :::column-end:::
    :::column span="3":::
        Fügen Sie Ihrer Transaktion einen RFC-Aufruf hinzu. 
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="1":::
        [ **[RFC] Aufrufen der Funktion in SAP**](/connectors/sap/#[rfc]-call-function-in-sap-(preview))
    :::column-end:::
    :::column span="3":::
        Aufrufen einer RFC-Operation (sRFC, tRFC oder qRFC) auf Ihrem SAP-System.
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="1":::
        [**TRANSAKTION Committen**](/connectors/sap/#[rfc]-commit-transaction-(preview))
    :::column-end:::
    :::column span="3":::
        De RFC-Transaktion für die Sitzung und/oder die Warteschlange committen.
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="1":::
        [ **[RFC]-Transaktion erstellen**](/connectors/sap/#[rfc]-create-transaction-(preview))
    :::column-end:::
    :::column span="3":::
        Erstellen Sie eine neue Transaktion nach Bezeichner und/oder Warteschlangenname. Wenn die Transaktion existiert, erhalten Sie die Details. 
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="1":::
        [ **[RFC]Transaktion erhalten**](/connectors/sap/#[rfc]-get-transaction-(preview))
    :::column-end:::
    :::column span="3":::
        Ruft die Details einer Transaktion nach Bezeichner und/oder Warteschlangenname ab. Legt eine neue Transaktion an, wenn keine vorhanden ist.
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="1":::
        [**Schemas generieren**](/connectors/sap/#generate-schemas)
    :::column-end:::
    :::column span="3":::
        Generieren von Schemas für die SAP-Artefakte für IDoc, BAPI oder RFC.
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="1":::
        [**Lesen der SAP-Tabelle**](/connectors/sap/#read-sap-table-(preview))
    :::column-end:::
    :::column span="3":::
        Lesen einer SAP-Tabelle.
    :::column-end:::
:::row-end:::
:::row:::
    :::column span="1":::
        [**Senden von Nachrichten an SAP**](/connectors/sap/#send-message-to-sap)
    :::column-end:::
    :::column span="3":::
        Senden Sie einen beliebigen Nachrichtentyp (RFC, BAPI, IDoc) an SAP.
    :::column-end:::
:::row-end:::

## <a name="next-steps"></a>Nächste Schritte

* [Herstellen einer Verbindung von Azure Logic Apps zu lokalen Systemen](../logic-apps/logic-apps-gateway-connection.md)

* In diesem Artikel lernen Sie die Überprüfung, die Transformation und die Verwendung anderer Nachrichtenvorgänge im [Enterprise Integration Pack](../logic-apps/logic-apps-enterprise-integration-overview.md) kennen.

* Informationen zu anderen [Logic Apps-Connectors](../connectors/apis-list.md)