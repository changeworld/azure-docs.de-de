---
title: 'Azure Percept: Bekannte Probleme'
description: Enthält Informationen zu den bekannten Problemen für Azure Percept und den Problemumgehungen.
author: mimcco
ms.author: mimcco
ms.service: azure-percept
ms.topic: reference
ms.date: 03/25/2021
ms.openlocfilehash: 49ac497505930d82a3ab8e90fb3f386cc1741dc7
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/30/2021
ms.locfileid: "105605058"
---
# <a name="known-issues"></a>Bekannte Probleme

Wenn eines dieser Probleme auftritt, ist es nicht erforderlich, ein Ticket für einen Fehler zu erstellen. Erstellen Sie ein Ticket, falls bei einer Problemumgehung ein Problem auftritt.

|Bereich|Beschreibung des Problems|Problemumgehung|
|-------|---------|---------|
| Onboardingumgebung | Das Onboarding kann nur durchgeführt werden, wenn WLAN für das Gerät konfiguriert ist (Fehler bei Azure-Anmeldung). | 1. [Stellen Sie eine SSH-Verbindung mit Ihrem Azure Percept DK-Gerät her](./how-to-ssh-into-percept-dk.md). <br> 2. Ermitteln und kopieren Sie die Ethernet-IP-Adresse des Geräts. <br> 3. Greifen Sie auf die Onboardingumgebung zu, indem Sie die auf der Ethernet-IP-Adresse basierende URL verwenden. |
| Onboardingumgebung | Wenn Sie auf Links in der EULA (Lizenzvertrag) klicken, wird manchmal keine neue Webseite geöffnet. | Kopieren Sie den Link, und öffnen Sie ihn in einem separaten Browserfenster. |
| Onboardingumgebung | Bei einer Verbindung mit einem WLAN-Hotspot für mobile Geräte kann der Onboardingvorgang nicht durchgeführt werden. | Verbinden Sie Ihr Gerät direkt mit SoftAP, einem WLAN oder per Ethernetverbindung mit einem Netzwerk. |
| WLAN | Es kann vorkommen, dass die SoftAP-Verbindung getrennt oder nicht mehr angezeigt wird. | Wir untersuchen dieses Problem.  Wenn Sie das Gerät neu starten, ist die Verbindung normalerweise wieder vorhanden. |
| WLAN | Die Taste, mit der WLAN/SoftAP ein- und ausgeschaltet wird, funktioniert manchmal nicht. | Drücken Sie weiterhin auf die Taste, oder starten Sie das Gerät neu. |
| WLAN | Benutzer sehen möglicherweise eine Meldung, nachdem sie sich mit dem WLAN verbunden haben: <br> „Dieses WLAN-Netzwerk verwendet einen älteren Sicherheitsstandard.“ | Für die SoftAP-Verbindung des DevKit wird der Verschlüsselungsalgorithmus WEP verwendet. |
| WLAN | Die Verbindung mit der SoftAP kann für einen PC mit Windows 10 nicht hergestellt werden, und es wird eine Fehlermeldung der folgenden Art angezeigt: <br> „Keine Verbindung mit dem Netzwerk möglich.“ | Starten Sie das DevKit und den Computer neu. |
| Geräteupdate | Container werden nach einem drahtlosen Update (Over-The-Air, OTA) nicht mehr ausgeführt. | Stellen Sie eine SSH-Verbindung mit dem Gerät her, und starten Sie den IoT Edge-Container mit dem folgenden Befehl neu: `systemctl restart iotedge`. Hierdurch werden alle Container neu gestartet. |
| Geräteupdate | Benutzer erhalten ggf. auch dann eine Meldung, dass beim Update ein Fehler aufgetreten ist, wenn der Vorgang erfolgreich war. | Bestätigen Sie die Aktualisierung, indem Sie in IoT Hub zum Gerätezwilling des DevKits navigieren und den Wert von `swVersion` überprüfen. Dieser Fehler tritt nach dem ersten Update nicht mehr auf. |
| Geräteupdate | Für Benutzer gehen nach dem ersten Update ggf. die Einstellungen für die WLAN-Verbindung verloren. | Führen Sie nach dem Update den Onboardingvorgang durch, um die WLAN-Verbindung einzurichten. Dieser Fehler tritt nach dem ersten Update nicht mehr auf. |
| Geräteupdate | Nach einem OTA-Update können Benutzer sich nicht mehr per SSH anmelden, indem sie die zuvor erstellten Benutzerkonten verwenden, und beim Onboardingvorgang können keine neuen SSH-Benutzer erstellt werden. Dieses Problem tritt für Systeme auf, für die OTA-Updates mit den folgenden vorinstallierten Imageversionen durchgeführt werden: 2020.110.114.105 und 2020.109.101.105. | Führen Sie nach dem OTA-Update die folgenden Schritte aus, um Ihre Benutzerprofile wiederherzustellen: <br> [Stellen Sie eine SSH-Verbindung mit Ihrem DevKit](./how-to-ssh-into-percept-dk.md) her, indem Sie „root“ als Benutzernamen verwenden. Falls Sie die SSH-Benutzeranmeldung mit „root“ für den Onboardingvorgang deaktiviert haben, müssen Sie dies wieder aktivieren. Führen Sie diesen Befehl aus, nachdem die Verbindungsherstellung erfolgreich war: <br> ```mkdir -p /var/custom-configs/home; chmod 755 /var/custom-configs/home``` <br> Führen Sie den folgenden Befehl aus, um die vorherigen Benutzerdaten (home) wiederherzustellen: <br> ```mkdir -p /tmp/prev-rootfs && mount /dev/mmcblk0p3 /tmp/prev-rootfs && [ ! -L /tmp/prev-rootfs/home ] && cp -a /tmp/prev-rootfs/home/* /var/custom-configs/home/. && echo "User home migrated!"; umount /tmp/prev-rootfs``` |
| Geräteupdate | Nach einem OTA-Update gehen Updategruppen verloren. | Aktualisieren Sie das Tag des Geräts gemäß [dieser Anleitung](./how-to-update-over-the-air.md#create-a-device-update-group). |
| Dev Tools-Paketinstallationsprogramm | Die optionale Caffe-Installation kann fehlschlagen, wenn Docker nicht korrekt auf dem System ausgeführt wird. | Stellen Sie sicher, dass Docker installiert ist und ausgeführt wird, und wiederholen Sie anschließend die Installation von Caffe. |
| Dev Tools-Paketinstallationsprogramm | Auf inkompatiblen Systemen tritt bei der optionalen CUDA-Installation ein Fehler auf. | Überprüfen Sie vor dem Ausführen des Installationsprogramms die Kompatibilität des Systems mit CUDA. |
| Docker, Netzwerk, IoT Edge | Wenn für Ihr internes Netzwerk „172.x.x.x“ genutzt wird, können Docker-Container keine Verbindung mit IoT Edge herstellen. | Fügen Sie der Datei „/etc/docker/daemon.json“ wie folgt den Abschnitt „bip“ hinzu: `{    "bip": "192.168.168.1/24"}` |
|Azure Percept Studio | Bei Links vom Typ „Stream anzeigen“ in Azure Percept Studio wird kein neues Fenster mit dem Webstream des Geräts geöffnet. | 1. Öffnen Sie das [Azure-Portal](https://portal.azure.com), und wählen Sie **IoT Hub** aus. <br> 2. Klicken Sie auf die IoT Hub-Instanz, mit der Ihr Gerät verbunden ist. <br> 3. Wählen Sie auf Ihrer IoT Hub-Seite unter **Automatische Geräteverwaltung** die Option **IoT Edge** aus. <br> 4. Wählen Sie in der Liste Ihr Gerät aus. <br> 5. Wählen Sie oben auf Ihrer Geräteseite die Option **Module festlegen** aus. <br> 6. Klicken Sie auf das Papierkorbsymbol neben **HostIpModule**, um das Modul zu löschen. <br> 7. Klicken Sie zum Bestätigen der Aktion auf **Überprüfen und erstellen** und dann auf **Erstellen**. <br> 8. Öffnen Sie [Azure Percept Studio](https://go.microsoft.com/fwlink/?linkid=2135819), und klicken Sie im linken Menübereich auf **Geräte**. <br> 9. Wählen Sie in der Liste Ihr Gerät aus. <br> 10. Klicken Sie auf der Registerkarte **Vision** auf **Gerätestream anzeigen**. Ihr Gerät lädt eine neue Version von „HostIpModule“ herunter und öffnet eine Browserregisterkarte mit dem Webstream Ihres Geräts. |