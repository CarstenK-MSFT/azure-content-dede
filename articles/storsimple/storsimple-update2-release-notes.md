<properties 
   pageTitle="Versionsanmerkungen zu Update 2 der StorSimple 8000-Serie | Microsoft Azure"
   description="Beschreibt die neuen Features sowie Probleme und Problemumgehungen für Update 2 der StorSimple 8000-Serie."
   services="storsimple"
   documentationCenter="NA"
   authors="SharS"
   manager="carolz"
   editor="" />
 <tags 
   ms.service="storsimple"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="TBD"
   ms.date="12/14/2015"
   ms.author="v-sharos" />

# Versionsanmerkungen zu Update 2 der StorSimple 8000-Serie  

## Übersicht

Die folgenden Versionsanmerkungen beschreiben die neuen Features und weisen auf wichtige offene Punkte bei Update 2 der StorSimple 8000-Serie hin. Sie enthalten außerdem eine Liste der Updates für StorSimple-Software, -Treiber und -Datenträgerfirmware, die in dieser Version enthalten sind.

Update 2 kann auf alle StorSimple-Geräte angewendet werden, auf denen Release (GA) oder Update 0.1 bis Update 1.2 ausgeführt wird.

Lesen Sie vor der Bereitstellung des Updates in Ihrer StorSimple-Lösung die Informationen in den Versionsanmerkungen sorgfältig durch.

>[AZURE.IMPORTANT]
> 
- Die Installation dieses Updates (einschließlich der Windows-Updates) dauert ungefähr 4 bis 7 Stunden. 
- Update 2 verfügt über Updates für Software, USM, LSI-Treiber und SSD-Firmware.
- Bei neuen Versionen werden Updates möglicherweise nicht sofort angezeigt, da diese in mehreren Phasen bereitgestellt werden. Warten Sie einige Tage, und suchen Sie dann erneut nach Updates, da diese bald verfügbar werden.


## Neuerungen in Update 2

In Update 2 werden die folgenden neuen Features eingeführt:

- **Lokale Volumes**: In früheren Versionen der StorSimple 8000-Serie wurden Datenblöcke je nach Auslastung in die Cloud ausgelagert. Es gab keine Möglichkeit sicherzustellen, dass Blöcke auf dem lokalen Volume verbleiben. Wenn Sie in Update 2 ein Volume erstellen, können Sie ein Volume als lokal kennzeichnen. Primäre Daten von diesem Volume werden dann nicht in die Cloud ausgelagert. Momentaufnahmen lokaler Volumes werden weiterhin zur Sicherung in die Cloud kopiert, damit die Cloud für Datenmobilität und Notfallwiederherstellung verwendet werden kann. Darüber hinaus können Sie den Volumetyp ändern (also mehrstufige Volumes in lokale Volumes konvertieren und umgekehrt). 

- **Verbesserungen an virtuellen StorSimple-Geräten**: Zuvor wurde das virtuelle Gerät in der StorSimple 8000-Serie als Lösung für die Notfallwiederherstellung oder Entwicklung/Testing eingesetzt. Es gab nur ein virtuelles Gerätemodell (Modell 1100). In Update 2 werden zwei virtuelle Gerätemodelle eingeführt:

     - 8010 (früher: 1100): Keine Änderung; besitzt eine Kapazität von 30 TB und verwendet den Azure-Standardspeicher.
     - 8020: Besitzt eine Kapazität von 64 TB und verwendet den Azure Premium-Speicher für eine verbesserte Leistung.

    Es gibt eine einzige VHD für beide virtuellen Gerätemodelle (8010/8020). Wenn Sie das virtuelle Gerät zum ersten Mal starten, erkennt es die Plattformparameter und wendet die richtige Modellversion an.

- **Netzwerkverbesserungen**: Update 2 umfasst die folgenden Netzwerkverbesserungen:

     - Mehrere NICs können für die Cloud aktiviert werden, sodass ein Failover durchgeführt werden kann, wenn eine NIC fehlschlägt.
     - Routingverbesserungen mit festen Metriken für cloudfähige Blöcke.
     - Onlinewiederholung bei Ressourcenfehlern vor einem Failover.
     - Neue Warnungen für Dienstfehler.

- **Updateverbesserungen**: In Update 1.2 und früheren Versionen wurde die StorSimple 8000-Serie über zwei Kanäle aktualisiert: Windows Update für Clustering, iSCSI und so weiter und Microsoft Update für Binärdateien und Firmware. Update 2 verwendet Microsoft Update für alle Updatepakete. Auf diese Weise verringert sich der Zeitaufwand für Patches oder Failover.

- **Firmwareupdates**: Die folgenden Firmwareupdates sind enthalten:
    - USM 3.33 RC5
    - LSI: lsi\_sas2.sys, Produktversion 2.00.72.10
    - Nur SSD (keine HDD-Updates): XMGG, XGEG, KZ50, F6C2 und VR08

- **Proaktiver Support**: Mit Update 2 kann Microsoft zusätzliche Diagnoseinformationen vom Gerät abrufen. Wenn unser Betriebsteam Geräte erkennt, auf denen Probleme auftreten, besitzen wir bessere Möglichkeiten zum Sammeln von Informationen vom Gerät und zum Diagnostizieren von Problemen. **Durch das Akzeptieren von Update 2 stimmen Sie dieser Bereitstellung von proaktivem Support zu**.
 

## In Update 2 behobene Probleme

Die folgende Tabelle enthält eine Zusammenfassung der Probleme, die in Update 2 behoben wurden.

| Nr. | Funktion | Problem | Gilt für das physische Gerät | Gilt für das virtuelle Gerät |
|-----|---------|-------|--------------------------------|--------------------------------|
| 1 | Netzwerkschnittstellen | Nach dem Upgrade auf Update 1 hat der StorSimple Manager-Dienst gemeldet, dass die Ports „Data2“ und „Data3“ auf einem Controller Fehler verursachen. Dieses Problem wurde behoben. | Ja | Nein |
| 2 | Aktualisierungen | Nach dem Upgrade auf Update 1 traten im Azure-Portal auf mehreren Geräten akustische Alarme auf. Dieses Problem wurde behoben. | Ja | Nein |
| 3 | Openstack-Authentifizierung | Wenn Sie Openstack als Clouddienstanbieter verwenden, haben Sie möglicherweise eine Fehlermeldung erhalten, dass die Zeichenfolge zur Cloudauthentifizierung zu lang war. Dies wurde korrigiert. | Ja | Nein |


## Bekannte Probleme in Update 2

Die folgende Tabelle enthält eine Zusammenfassung der bekannten Probleme in dieser Version.

| Nr. | Funktion | Problem | Kommentare/Problemumgehung | Gilt für das physische Gerät | Gilt für das virtuelle Gerät |
|-----|---------|-------|----------------------------|----------------------------|---------------------------|
| 1 | Datenträgerquorum | In seltenen Fällen kann der Speicherpool offline geschaltet werden, wenn der Großteil der Datenträger im EBOD-Gehäuse eines 8600-Geräts getrennt wird, sodass kein Datenträgerquorum verfügbar ist. Der Speicherpool bleibt offline, auch wenn die Verbindung zu den Datenträgern wiederhergestellt wird. | Sie müssen das Gerät neu starten. Wenn das Problem weiterhin auftritt, wenden Sie sich an den Microsoft-Support, um Informationen zu den nächsten Schritten zu erhalten. | Ja | Nein |
| 2 | Falsche Controller-ID | Beim Austausch eines Controllers kann es vorkommen, dass Controller 0 als Controller 1 angezeigt wird. Während des Controlleraustauschs kann die Controller-ID anfänglich als ID des Peercontrollers angezeigt werden, wenn das Image vom Peerknoten geladen wurde. In seltenen Fällen kann dieses Verhalten auch nach einem Neustart des Systems auftreten. | Es ist keine Benutzeraktion erforderlich. Dieses Problem löst sich von selbst, nachdem der Controlleraustausch abgeschlossen ist. | Ja | Nein |
| 3 | Speicherkonten | Das Verwenden des Speicherdiensts zum Löschen des Speicherkontos wird nicht unterstützt. Dies führt dazu, dass keine Benutzerdaten abgerufen werden können. | Ja | Ja |
| 4 | Gerätefailover | Mehrere Failover eines Volumecontainers von demselben Quellgerät auf verschiedene Zielgeräte werden nicht unterstützt. Das Failover von einem einzelnen nicht reagierenden Gerät auf mehrere Geräte führt dazu, dass die Volumecontainer auf dem ersten Gerät mit erfolgtem Failover die Dateneigentümerschaft verlieren. Wenn Sie diese Volumecontainer nach einem solchen Failover im klassischen Azure-Portal betrachten, werden sie anders angezeigt oder verhalten sie sich anders. | | Ja | Nein |
| 5 | Installation | Während der Installation von StorSimple-Adapter für SharePoint müssen Sie die IP-Adresse eines Geräts angeben, damit die Installation erfolgreich abgeschlossen wird. | | Ja | Nein |
| 6 | Webproxy | Wenn Ihre Webproxykonfiguration das Protokoll "HTTPS" verwendet, ist die Kommunikation zwischen dem Gerät und dem Dienst beeinträchtigt, und das Gerät wird offline geschaltet. Supportpakete werden bei diesem Vorgang ebenfalls generiert. Sie beanspruchen auf Ihrem Gerät erhebliche Ressourcen. | Stellen Sie sicher, dass "HTTP" als Protokoll für die Webproxy-URL angegeben ist. Weitere Informationen finden Sie unter [Konfigurieren des Webproxys für Ihr Gerät](storsimple-configure-web-proxy.md). | Ja | Nein |
| 7 | Webproxy | Wenn Sie den Webproxy für ein registriertes Gerät konfigurieren und aktivieren, müssen Sie den aktiven Controller auf Ihrem Gerät neu starten. | | Ja | Nein |
| 8 | Hohe Cloud-Latenzen und hohe E/A-Arbeitsauslastung | Wenn Ihr StorSimple-Gerät mit einer Kombination aus sehr hohen Cloud-Latenzen (mehrere Sekunden) und hoher E/A-Arbeitsauslastung konfrontiert wird, verschlechtert sich die Leistung der Gerätevolumes, und es tritt ggf. der E/A-Fehler "Gerät nicht bereit" auf. | Sie müssen die Gerätecontroller manuell neu starten oder ein Gerätefailover ausführen, um dieses Problem zu beheben. | Ja | Nein |
| 9 | Azure PowerShell | Bei Verwendung des StorSimple-Cmdlets **Get-AzureStorSimpleStorageAccountCredential | Select-Object -First 1 -Wait**, um das erste Objekt zu markieren, damit ein neues **Volumecontainer** -Objekt erstellt werden kann, gibt das Cmdlet alle Objekte zurück. | Schließen Sie das Cmdlet wie folgt in Klammern ein:**(Get-Azure-StorSimpleStorageAccountCredential) | Select-Object -First 1 -Wait** | Ja | Ja |
| 10| Migration | Wenn mehrere Volumecontainer für die Migration übergeben werden, ist der ETA für die neueste Sicherung nur für den ersten Volumecontainer korrekt. Außerdem tritt nach erfolgter Migration der ersten 4 Sicherungen im ersten Volumecontainer Parallelmigration auf. | Es wird empfohlen, immer nur einen Volumecontainer gleichzeitig zu migrieren. | Ja | Nein |
| 11| Migration | Nach der Wiederherstellung werden Volumes nicht der Sicherungsrichtlinie oder dem virtuellen Datenträger hinzugefügt. | Zum Erstellen von Sicherungen müssen diese Volumes einer Sicherungsrichtlinie hinzugefügt werden. | Ja | Ja |
| 12| Migration | Nach Abschluss die Migration darf das Gerät der 5000/7000 Serie nicht auf die migrierten Datencontainer zugreifen. | Es wird empfohlen, die migrierten Datencontainer zu löschen, nachdem die Migration vollständig abgeschlossen ist. | Ja | Nein |
| 13| Klonen und Notfallwiederherstellung | Ein StorSimple-Gerät mit Update 1 kann für ein Gerät, auf dem ältere Software als Update 1 ausgeführt wird, keinen Klonvorgang bzw. keine Notfallwiederherstellung durchführen. | Das Zielgerät muss auf Update 1 aktualisiert werden, damit diese Operationen ausgeführt werden können. | Ja | Ja |
| 14 | Migration | Die Konfigurationssicherung für die Migration kann auf einem Gerät der Serie 5000-7000 fehlschlagen, wenn Volumegruppen ohne zugehörige Volumes vorhanden sind. | Löschen Sie die leeren Volumegruppen ohne zugehörige Volumes, und wiederholen Sie dann die Konfigurationssicherung.| Ja | Nein |
| 15 | Azure PowerShell-Cmdlets und lokale Volumes | Sie können über Azure PowerShell-Cmdlets kein lokales Volume erstellen. (Alle Volumes, die Sie mithilfe von Azure PowerShell erstellen, sind mehrstufig.) |Verwenden Sie immer den StorSimple Manager-Dienst, um lokale Volumes zu konfigurieren.|
| 16 |Für lokale Volumes verfügbarer Speicherplatz | Wenn Sie ein lokales Volume löschen, wird der für neue Volumes verfügbare Speicherplatz möglicherweise nicht sofort aktualisiert. Der StorSimple Manager-Dienst aktualisiert den verfügbaren lokalen Speicherplatz ungefähr stündlich.| Warten Sie eine Stunde, bevor Sie versuchen, das neue Volume zu erstellen. |

## Controller- und Firmwareupdates in Update 2

Mit dieser Version werden der Treiber und die Datenträgerfirmware auf Ihrem Gerät aktualisiert.
 
- Weitere Informationen zum USM- und LSI-Firmwareupdate finden Sie im Microsoft Knowledge Base-Artikel 3121900. 
- Weitere Informationen zum Datenträgerfirmware-Update finden Sie im Microsoft Knowledge Base-Artikel 3121899.
 
## Updates von virtuellen Geräten in Update 2

Dieses Update kann nicht auf das virtuelle Gerät angewendet werden. Es müssen neue virtuelle Geräte erstellt werden.

<!---HONumber=AcomDC_1217_2015-->