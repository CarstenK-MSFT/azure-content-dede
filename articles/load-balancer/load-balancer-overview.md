<properties 
   pageTitle="Übersicht über Azure-Lastenausgleich | Microsoft Azure"
   description="Übersicht über Funktionen, Architektur und Implementierung des Azure-Lastenausgleichsmoduls. Dieser Artikel dient dem Verständnis der Arbeitsweise und der Nutzung des Lastenausgleichs in der Cloud."
   services="load-balancer"
   documentationCenter="na"
   authors="joaoma"
   manager="adinah"
   editor="tysonn" />
<tags 
   ms.service="load-balancer"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="12/09/2015"
   ms.author="joaoma" />


# Was ist der Azure Load Balancer?
 
Der Azure-Lastenausgleich bietet hohe Verfügbarkeit und Netzwerkleistung für Ihre Anwendungen. Es handelt sich um einen Lastenausgleich des Layer-4-Typs (TCP, UDP), der eingehenden Datenverkehr zwischen funktionierenden Dienstinstanzen in Clouddiensten oder auf virtuelle Computer verteilt, die in einem Lastenausgleich definiert wurden.
 
Es gibt folgende Konfigurationsmöglichkeiten:

- Lastenausgleich des eingehenden Internetdatenverkehrs für virtuelle Computer. Wir bezeichnen dies als [Lastenausgleich für Internetzugriff](load-balancer-internet-overview.md).
- Lastenausgleich für Datenverkehr zwischen virtuellen Computern in einem virtuellen Netzwerk, zwischen virtuellen Computern in Clouddiensten oder zwischen lokalen und virtuellen Computern in einem standortübergreifenden virtuellen Netzwerk. Wir bezeichnen dies als [Internen Lastenausgleich (ILB)](load-balancer-internal-overview.md).
- 	Weiterleiten von externem Datenverkehr an eine bestimmte Instanz eines virtuellen Computers

## Grundlegendes zum Azure Load Balancer im klassischen Azure und Azure-Ressourcen-Manager (ARM)

Alle Ressourcen in der Cloud benötigen eine öffentliche IP-Adresse, damit sie im Internet erreichbar sind. Die Cloudinfrastruktur in Microsoft Azure verwendet nicht routbare IP-Adressen in ihren Ressourcen und Netzwerkadressübersetzung (Network Address Translation, NAT) mit öffentlichen IP-Adressen zur Kommunikation mit dem Internet.

Es gibt 2 Bereitstellungsmodelle in Microsoft Azure und ihre Lastenausgleichsimplementierungen:

 
### Klassisches Azure

Klassisches Azure ist das erste in Microsoft Azure implementierte Bereitstellungsmodell. In diesem Modell sind eine öffentliche IP-Adresse und ein FQDN einem Clouddienst zugewiesen, und in einem Clouddienst bereitgestellte virtuelle Computer können gruppiert werden, um einen Lastenausgleich zu verwenden. Der Lastenausgleich übernimmt Portübersetzung und Lastenausgleich im Netzwerkdatenverkehr, wobei die öffentliche IP-Adresse für den Clouddienst genutzt wird.

In einem klassischen Bereitstellungsmodell erfolgt die Portübersetzung mit Endpunkten, die eine 1:1-Beziehung zwischen dem öffentlichen zugewiesenen Port der öffentlichen IP-Adresse und dem lokalen Port bilden, der zum Senden von Datenverkehr an einen bestimmten virtuellen Computer zugewiesen wurde.

Lastenausgleich erfolgt über vom Load Balancer festgelegte Endpunkte. Diese Endpunkte bilden eine 1:n-Beziehung zwischen der öffentlichen IP-Adresse und lokalen Ports, die allen virtuellen Computern in der Gruppe zugewiesen wurden, die auf den ausgeglichenen Netzwerkverkehr reagieren.

Die Domänenbezeichnung für die öffentliche IP-Adresse, die ein Lastenausgleich in diesem Bereitstellungsmodell verwenden würde, wäre `<cloud service name>.cloudapp.net`.

Dies ist eine grafische Darstellung eines Lastenausgleichs in einem klassischen Bereitstellungsmodell: ![Lastenausgleich auf Hashbasis](./media/load-balancer-overview/asm-lb.png)

### Azure-Ressourcen-Manager
 
Das Konzept des Lastenausgleichs ändert sich beim Azure-Ressourcen-Manager (ARM), da dort keine Notwendigkeit besteht, dass ein Clouddienst einen Lastenausgleich erstellt.

In ARM ist eine öffentliche IP-Adresse ihre eigene Ressource und kann einer Domänenbezeichnung oder einem DNS-Namen zugeordnet werden. Die öffentliche IP-Adresse ist in diesem Fall der Lastenausgleichsressource zugeordnet, sodass Lastenausgleichsregeln, eingehende NAT-Regeln die öffentliche IP-Adresse als Internetendpunkt für die Ressourcen verwenden, die ausgeglichenen Netzwerkverkehr empfangen.

Eine Netzwerkschnittstellenressource (NIC) enthält die IP-Adresskonfiguration (private oder öffentliche IP) für einen virtuellen Computer. Sobald eine NIC einem Lastenausgleichs-Back-End-IP-Adresspool hinzugefügt wurde, beginnt der Lastenausgleich, ausgeglichenen Netzwerkverkehr basierend auf den Lastenausgleichsregeln zu senden.

Eine Verfügbarkeitsgruppe ist die Gruppierungsmethode, die verwendet wird, um virtuelle Computer dem Lastenausgleich hinzuzufügen. Die Verfügbarkeitsgruppe garantiert, dass die virtuellen Computer sich nicht auf der gleichen physischen Hardware befinden, sodass bei einem Fehler im Zusammenhang mit der physischen Cloudinfrastruktur sichergestellt ist, dass der Lastenausgleich immer über einen virtuellen Computer verfügt, der ausgeglichenen Netzwerkverkehr empfängt.

Dies ist eine grafische Darstellung eines Lastenausgleichs im Azure-Ressourcen-Manager (ARM):

![Lastenausgleich auf Hashbasis](./media/load-balancer-overview/arm-lb.png)

## Lastenausgleichsfunktionen

### Layer-4-Lastenausgleich, Verteilung auf Hashbasis

Der Azure-Lastenausgleich verwendet einen Verteilungsalgorithmus auf Hashbasis. Standardmäßig wird ein 5-Tupelhash (Quell-IP, Quellport, IP-Zieladresse, Zielport Protokolltyp) zum Zuordnen des Datenverkehrs an verfügbare Server verwendet. Dabei wird Bindung nur in einer Transportsitzung angeboten. Pakete in derselben TCP- oder UDP-Sitzung werden an die gleiche Instanz der Rechenzentrums-IP (DIP) hinter dem Lastenausgleichs-Endpunkt geleitet. Wenn der Client die Verbindung schließt und erneut öffnet oder eine neue Sitzung über die gleiche Quell-IP startet, wird der Quellport geändert. Dadurch kann der Datenverkehr an einen anderen DIP-Endpunkt geleitet werden.


Weitere Informationen finden Sie unter [Verteilungsmodus des Lastenausgleichs](load-balancer-distribution-mode.md)

![Lastenausgleich auf Hashbasis](./media/load-balancer-overview/load-balancer-distribution.png)

### Portweiterleitung

Mit dem Azure-Lastenausgleich steuern Sie die Verwaltung der eingehenden Kommunikation wie Datenverkehr von Internethosts oder virtuellen Computern in anderen Clouddiensten bzw. virtuellen Netzwerken. Dieses Steuerelement wird von einem Endpunkt (auch als Eingabeendpunkt bezeichnet) dargestellt.

Ein Endpunkt mit einem öffentlichen Port überwacht Datenverkehr und leitet diesen an einen internen Port. Sie können dieselben Ports einem internen oder externen Endpunkt zuordnen oder einen anderen Port verwenden. Beispiel: Sie können einen Webserver konfigurieren, der Port 81 überwacht, während Port 80 dem öffentlichen Endpunkt zugeordnet ist. Die Erstellung eines öffentlichen Endpunkts löst die Erstellung eines Azure-Lastenausgleichs aus.

Die Endpunkte auf virtuellen Computern, die Sie mit dem Azure-Verwaltungsportal erstellen, werden hauptsächlich für das Remotedesktopprotokoll (RDP) und Windows PowerShell-Sitzungsverkehr verwendet. Über diese Endpunkte können Sie die virtuellen Computer über das Internet verwalten.


### Automatische Neukonfiguration bei vertikaler/horizontaler Skalierung

Der Azure-Lastenausgleich konfiguriert sich unmittelbar neu, wenn Sie Instanzen vertikal oder horizontal skalieren (aufgrund der Zunahme der Anzahl an Instanzen für die Web-/Workerrolle oder durch Einfügen zusätzlicher virtueller Computer in der gleichen Lastenausgleichsgruppe).


### Dienstüberwachung
Der Azure-Lastenausgleich bietet die Möglichkeit, die Integrität der verschiedenen Serverinstanzen zu testen. Wenn ein Test nicht reagiert, beendet der Azure-Lastenausgleich das Senden neuer Verbindungen an die fehlerhaften Instanzen. Vorhandene Verbindungen sind nicht betroffen.

Es gibt drei unterstützte Tests:
 
- Gast-Agent-Test (nur PaaS-VMs). Der Azure-Lastenausgleich nutzt den Gast-Agent auf dem virtuellen Computer und überwacht und antwortet nur mit einer HTTP-OK-Antwort 200, wenn die Instanz bereit ist (d. h. die Instanz nicht ausgelastet ist bzw. recycelt oder angehalten wird usw.). Wenn der Gast-Agent nicht mit dem HTTP-OK-Code 200 antwortet, kennzeichnet der Azure-Lastenausgleich die Instanz als nicht reagierend und sendet keinen Datenverkehr mehr an diese Instanz. Der Azure-Lastenausgleich sendet weiterhin ein Ping an die Instanz, und falls der Gast-Agent mit HTTP-Code 200 antwortet, sendet der Azure-Lastenausgleich wieder Datenverkehr an diese Instanz. Bei Verwendung einer Webrolle wird Ihr Websitecode in der Regel in der ausführbaren Datei "w3wp.exe" ausgeführt, die von der Azure-Struktur oder dem Gast-Agent nicht überwacht wird. Fehler in "w3wp.exe" (z. B. HTTP 500-Antworten) werden nicht an den Gast-Agent gemeldet, und das Lastenausgleichsmodul weiß nicht, dass es diese Instanz aus der Rotation entfernen sollte.

- Benutzerdefinierte HTTP-Tests. Der benutzerdefinierte Lastenausgleichstest überschreibt den Standard-Gast-Agent-Test und ermöglicht es Ihnen, Ihre eigene Logik zum Bestimmen der Integrität der Rolleninstanz zu erstellen. Der Lastenausgleich testet regelmäßig den Endpunkt (standardmäßig alle 15 Sekunden), und die Instanz wird als rotierend berücksichtigt, wenn sie innerhalb des Zeitlimits (Standardwert 31 Sekunden) mit einem TCP-ACK oder HTTP-Code 200 reagiert. Dies kann hilfreich sein, um eine eigene Logik zum Entfernen von Instanzen aus der Lastenausgleichsrotation zu implementieren, wie z. B. durch Rückgabe eines Nicht-200-Status, wenn die Instanz über 90 % CPU-Auslastung aufweist. Für Webrollen mit "w3wp.exe" bedeutet dies auch automatische Überwachung der Website, da Fehler im Websitecode einen Nicht-200-Status an den Lastenausgleichstest zurückgeben.

- Benutzerdefinierte TCP-Tests. TCP-Tests basieren auf der erfolgreichen Herstellung von TCP-Sitzungen an einem definierten Testport.

Weitere Informationen finden Sie unter [LoadBalancerProbe-Schema](https://msdn.microsoft.com/library/azure/jj151530.aspx).

### Quell-NAT (SNAT)


Auf den gesamten ausgehenden Internetdatenverkehr, der vom Dienst stammt, wird Quell-NAT (SNAT, Source Network Address Translation) mit der gleichen VIP-Adresse wie für eingehenden Datenverkehr angewendet. SNAT bietet wichtige Vorteile:

- Sie ermöglicht einfache Upgrades und Notfallwiederherstellung von Diensten, da die VIP-Adresse dynamisch einer anderen Instanz des Diensts zugeordnet werden kann.

- Sie vereinfacht die ACL-Verwaltung, da die ACL im Hinblick auf VIPs ausgedrückt werden kann. Daher müssen Sie keine Änderungen vornehmen, wenn Sie Dienste horizontal oder vertikal skalieren oder erneut bereitstellen.

Die Azure-Lastenausgleichskonfiguration unterstützt vollständige Cone-NAT für UDP. Vollständige Cone-NAT ist ein NAT-Typ, bei dem der Port eingehende Verbindungen von jedem externen Host (als Reaktion auf eine externe Anforderung) erlaubt.


>[AZURE.NOTE]Beachten Sie, dass für jede neue ausgehende Verbindung, die von einem virtuellen Computer initiiert wurde, auch ein ausgehender Port vom Azure-Lastenausgleich zugeordnet wird. Dem externen Host wird eingehender Datenverkehr über eine VIP mit zugewiesenem Port angezeigt. Wenn Ihre Szenarios eine große Anzahl ausgehender Verbindungen benötigen, empfiehlt es sich, dass die virtuellen Computer öffentliche IP-Adressen auf Instanzebene verwenden, damit diese über eine festgelegte ausgehende IP für die Quell-NAT (SNAT, Source Network Address Translation) verfügen. Dies reduziert das Risiko von Portauslastung.
>
>Die maximale Anzahl von Ports, die für VIP oder ILPIP verwendet werden können, beträgt 64k. Dies ist eine standardmäßige Einschränkung für TCP.


**Unterstützung für mehrere IPs mit Lastenausgleich für virtuelle Computer**

Sie können einer Gruppe von virtuellen Computern mehr als eine öffentliche IP-Adresse mit Lastenausgleich zuweisen. Mit dieser Funktion können Sie mehrere SSL-Websites und/oder mehrere Listener für SQL-AlwaysOn-Verfügbarkeitsgruppen in der gleichen Gruppe von virtuellen Computern hosten. Weitere Informationen finden Sie unter [Mehrere VIPs pro Clouddienst](load-balancer-multivip.md)

****Vorlagenbasierte Bereitstellungen mithilfe des Azure-Ressourcen-Managers** Azure-Ressourcen-Manager (ARM) ist das neue Verwaltungsframework für Dienste in Azure. Der Azure-Lastenausgleich kann jetzt mithilfe von APIs und Tools auf Basis von Azure-Ressourcen-Manager verwaltet werden. Weitere Informationen zum Azure-Ressourcen-Manager finden Sie unter [Iaas just got easier with Azure Resource Manager](http://azure.microsoft.com/blog/2015/04/29/iaas-just-got-easier-again/) (in englischer Sprache).


## Nächste Schritte

[Lastenausgleich für Internetzugriff (Übersicht)](load-balancer-internet-overview.md)

[Interner Lastenausgleich (Übersicht)](load-balancer-internal-overview.md)

[Erste Schritte – Lastenausgleich für Internetzugriff](load-balancer-internet-getstarted.md)
 

<!---HONumber=AcomDC_1217_2015-->