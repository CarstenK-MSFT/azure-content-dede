<properties
	pageTitle="Planen der Sicherungsinfrastruktur für virtuelle Computer in Azure | Microsoft Azure"
	description="Wichtige Hinweise für die Planung der Sicherungsinfrastruktur für virtuelle Computer in Azure"
	services="backup"
	documentationCenter=""
	authors="Jim-Parker"
	manager="jwhit"
	editor=""/>

<tags
	ms.service="backup"
	ms.workload="storage-backup-recovery"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="10/29/2015"
	ms.author="trinadhk; aashishr; jimpark; markgal"/>

# Planen der Sicherungsinfrastruktur für virtuelle Computer in Azure
Dieser Artikel behandelt die wichtigsten Faktoren, die Sie beim Planen Ihrer Sicherungsinfrastruktur für virtuelle Computer im Auge behalten sollten. Wenn Sie [Ihre Umgebung vorbereitet](backup-azure-vms-prepare.md) haben, ist dies der nächste Schritt, bevor Sie mit dem [Sichern Ihrer virtuellen Computer](backup-azure-vms.md) beginnen. Weitere Informationen zu virtuellen Azure-Computern finden Sie in der [Dokumentation zu Virtual Machines](https://azure.microsoft.com/documentation/services/virtual-machines/).

## Wie werden virtuelle Computer in Azure gesichert?
Wenn der Azure Backup-Dienst einen Sicherungsauftrag zur geplanten Zeit initiiert, löst er die Sicherungserweiterung zum Erstellen einer Momentaufnahme aus. Diese Momentaufnahme wird in Koordination mit dem Volumeschattenkopie-Dienst (VSS) erstellt, um eine konsistente Momentaufnahme der Datenträger auf dem virtuellen Computer zu erhalten, ohne dass dieser heruntergefahren werden muss.

Nachdem die Momentaufnahme erstellt wurde, werden die Daten vom Azure Backup-Dienst an den Sicherungstresor übertragen. Um den Sicherungsvorgang effizienter zu gestalten, werden vom Dienst nur diejenigen Datenblöcke identifiziert und übertragen, die seit der letzten Sicherung geändert wurden.

![Architektur der Sicherung von virtuellen Azure-Computern](./media/backup-azure-vms-introduction/vmbackup-architecture.png)

Wenn die Datenübertragung abgeschlossen ist, wird die Momentaufnahme entfernt und ein Wiederherstellungspunkt erstellt.

### Datenkonsistenz
Das Sichern und Wiederherstellen unternehmenskritischer Daten ist dadurch kompliziert, dass die Daten während der Ausführung der Anwendungen gesichert werden, welche die Daten generieren. Hierfür bietet Azure Backup anwendungskonsistente Sicherungen für Microsoft-Workloads mithilfe des Volumeschattenkopie-Diensts, um sicherzustellen, dass die Daten ordnungsgemäß in den Speicher geschrieben werden.

>[AZURE.NOTE]Für virtuelle Linux-Computer sind nur dateikonsistente Sicherungen möglich, da Linux keine dem VSS entsprechende Plattform umfasst.

Azure Backup erstellt vollständige VSS-Sicherungen auf virtuellen Windows-Computern (weitere Informationen hierzu finden Sie unter [VSS Full Backup](http://blogs.technet.com/b/filecab/archive/2008/05/21/what-is-the-difference-between-vss-full-backup-and-vss-copy-backup-in-windows-server-2008.aspx)). Um VSS-Kopiesicherungen zu aktivieren, müssen die folgenden Registrierungsschlüssel auf dem virtuellen Computer festgelegt werden.

```
[HKEY_LOCAL_MACHINE\SOFTWARE\MICROSOFT\BCDRAGENT]
"USEVSSCOPYBACKUP"="TRUE"
```


In dieser Tabelle werden die Konsistenztypen aufgeführt und die Umstände erläutert, unter denen sie während der Sicherungs- und Wiederherstellungsvorgänge für virtuelle Azure-Computer auftreten.

| Konsistenz | VSS-basiert | Erklärung und Details |
|-------------|-----------|---------|
| Anwendungskonsistenz | Ja | Dies ist der ideale Konsistenztyp für Microsoft-Workloads, da Folgendes gewährleistet wird:<ol><li> Der virtuelle Computer wird *hochgefahren*. <li>Es gibt *keine Datenbeschädigung*. <li>Es gibt *keinen Datenverlust*.<li> Die Daten sind mit der Anwendung konsistent, in der die Daten verwendet werden, da die Anwendung zum Zeitpunkt der Sicherung über den VSS einbezogen wird.</ol> Die meisten Microsoft-Workloads verfügen über VSS Writer, die workloadspezifische Aktionen im Zusammenhang mit der Datenkonsistenz ausführen. Beispielsweise umfasst Microsoft SQL Server einen VSS Writer, der die ordnungsgemäße Durchführung der Schreibvorgänge in die Transaktionsprotokolldatei und die Datenbank gewährleistet.<br><br> Bei Azure-VM-Sicherungen bedeutet das Erhalten eines anwendungskonsistenten Wiederherstellungspunkts, dass die Sicherungserweiterung den VSS-Workflow aufrufen und *ordnungsgemäß* abschließen konnte, bevor die Momentaufnahme des virtuellen Computers erstellt wurde. Das bedeutet natürlich, dass die VSS Writer aller Anwendungen auf dem virtuellen Azure-Computer ebenfalls aufgerufen wurden.<br><br>(Unter [The Basics of the Volume Shadow Copy Service (VSS)](http://blogs.technet.com/b/josebda/archive/2007/10/10/the-basics-of-the-volume-shadow-copy-service-vss.aspx) erhalten Sie grundlegende Informationen hierzu. [How Volume Shadow Copy Service Works](https://technet.microsoft.com/library/cc785914%28v=ws.10%29.aspx) bietet weiterführende Details.) |
| Dateisystemkonsistenz | Ja, bei Windows-basierten Computern | Es gibt zwei Szenarien, in denen der Wiederherstellungspunkt *dateisystemkonsistent* sein kann: <ul><li>Sicherungen virtueller Linux-Computer in Azure, da Linux über keine entsprechende Plattform wie VSS verfügt. <li>VSS-Fehler bei der Sicherung virtueller Windows-Computer in Azure.</li></ul> In beiden Fällen ist es das beste, Folgendes sicherzustellen: <ol><li> Der virtuelle Computer wird *hochgefahren*. <li>Es gibt *keine Datenbeschädigung*.<li>Es gibt *keinen Datenverlust*.</ol> Anwendungen müssen einen eigenen Reparaturmechanismus für die wiederhergestellten Daten implementieren.|
| Absturzkonsistenz | Nein | Diese Situation entspricht dem Absturz eines virtuellen Computers (durch eine Teil- oder Vollrückstellung). Dies geschieht normalerweise, wenn zum Zeitpunkt der Sicherung der virtuelle Azure-Computer heruntergefahren wird. Bei Azure-VM-Sicherungen bedeutet das Erhalten eines absturzkonsistenten Wiederherstellungspunkts, dass Azure Backup keine Garantie für die Konsistenz der Daten auf dem Speichermedium gewährt – weder im Hinblick auf das Betriebssystem noch im Hinblick auf die Anwendung. Nur Daten, die zum Zeitpunkt der Sicherung bereits auf dem Datenträger vorhanden sind, werden erfasst und gesichert. <br/> <br/> Auch wenn es keine Garantie gibt, wird das Betriebssystem in den meisten Fällen gestartet. In der Regel folgt eine Prozedur zur Datenträgerüberprüfung wie „chkdsk“, um eine mögliche Datenbeschädigung zu reparieren. Alle Daten im Arbeitsspeicher oder Schreibvorgänge, die nicht vollständig auf den Datenträger übertragen wurden, gehen verloren. Die Anwendung folgt normalerweise mit einem eigenen Überprüfungsmechanismus, falls ein Datenrollback durchgeführt werden muss. <br><br>Wenn das Transaktionsprotokoll beispielsweise Einträge enthält, die nicht in der Datenbank vorhanden sind, führt die Datenbanksoftware eine Zurücksetzung durch, bis die Daten konsistent sind. Wenn Daten über mehrere virtuelle Laufwerke verteilt sind (z. B. übergreifende Volumes), bietet ein absturzkonsistenter Wiederherstellungspunkt keine Garantie für die Richtigkeit der Daten.|


## Leistung und Ressourcenverwendung
Wie bei lokal bereitgestellter Sicherungssoftware muss die Sicherung von virtuellen Computern in Azure auch im Hinblick auf Kapazität und Ressourcenverwendung geplant werden. Die [Azure Storage-Begrenzungen](azure-subscription-service-limits.md#storage-limits) definieren, wie die VM-Bereitstellungen strukturiert werden, um maximale Leistung bei minimaler Beeinträchtigung der ausgeführten Workloads zu erzielen.

Die beiden wichtigsten Azure-Speicherbegrenzungen, die sich auf die Sicherungsleistung auswirken, sind:

- Max. Ausgang pro Speicherkonto
- Gesamtanforderungsrate pro Speicherkonto

### Speicherkontobegrenzungen
Wenn Sicherungsdaten aus dem Kundenspeicherkonto kopiert werden, werden diese auf die Metriken des Speicherkontos für IOPS (Input/Output Operations Per Second, Eingabe-/Ausgabevorgänge pro Sekunde) und Ausgang (Speicherdurchsatz) angerechnet. Zur gleichen Zeit werden die virtuellen Computer ausgeführt und verbrauchen IOPS und Durchsatz. Das Ziel ist, dafür zu sorgen, dass der Datenverkehr insgesamt – d. h. Sicherung und virtuelle Computer – die Speicherkontogrenzwerte nicht überschreitet.

### Anzahl der Datenträger
Der Sicherungsvorgang ist anspruchsvoll und versucht stets, alle verfügbaren Ressourcen zu nutzen, da die Sicherung möglichst schnell zum Abschluss gebracht werden soll. Sämtliche E/A-Vorgänge sind jedoch begrenzt durch den *Zieldurchsatz pro Einzelblob*, der auf 60 MB pro Sekunde beschränkt ist. Zur Beschleunigung der Sicherung wird versucht, die einzelnen Datenträger des virtuellen Computers *parallel* zu sichern. Bei einem virtuellen Computer mit vier Datenträgern versucht Azure Backup also, alle vier Datenträger parallel zu sichern. Der wichtigste Faktor, der den ausgehenden Sicherungsdatenverkehr für ein Kundenspeicherkonto bestimmt, ist daher die **Anzahl der Datenträger**, die über das Speicherkonto gesichert werden.

### Sicherungszeitplan
Ein weiterer Faktor, der sich auf die Leistung auswirkt, ist der **Sicherungszeitplan**. Wenn Sie alle virtuellen Computer für eine gleichzeitige Sicherung konfigurieren, steigt die Anzahl von *parallel* gesicherten Datenträgern, da Azure Backup versucht, so viele Datenträger wie möglich zu sichern. Eine Möglichkeit zur Reduzierung des Sicherungsdatenverkehrs für ein Speicherkonto ist daher, die einzelnen virtuellen Computer zu unterschiedlichen Tageszeiten zu sichern und Überlappungen zu vermeiden.

## Kapazitätsplanung
Zusammenfassend folgt aus diesen Faktoren, dass die Speicherkontoverwendung einer gründlichen Planung bedarf. Laden Sie das [Excel-Arbeitsblatt zur Kapazitätsplanung für VM-Sicherungen](https://gallery.technet.microsoft.com/Azure-Backup-Storage-a46d7e33) herunter, um die Auswirkungen Ihrer Datenträger- und Sicherungszeitplanauswahl zu überprüfen.

### Sicherungsdurchsatz
Für jeden zu sichernden Datenträger liest Azure Backup die Blöcke auf dem Datenträger und speichert nur die geänderten Daten (inkrementelle Sicherung). Diese Tabelle zeigt die durchschnittlichen Durchsatzwerte, die Sie von Azure Backup erwarten können. Anhand dieser Informationen können Sie die Zeitdauer schätzen, die zur Sicherung eines Datenträgers mit einer bestimmten Größe benötigt wird.

| Sicherungsvorgang | Optimaler Durchsatz |
| ---------------- | ---------- |
| Erste Sicherung | 160 MBit/s |
| Inkrementelle Sicherung (DR) | 640 MBit/s <br><br> Dieser Durchsatz kann erheblich sinken, wenn auf dem Datenträger viele verstreute Daten gesichert werden müssen. |

### Gesamtdauer der VM-Sicherung
Obgleich ein Großteil der Sicherungszeit für das Lesen und Kopieren von Daten aufgewendet wird, gibt es noch andere Vorgänge, die sich auf die Gesamtdauer einer VM-Sicherung auswirken:

- Die Zeit, die zum [Installieren oder Aktualisieren der Sicherungserweiterung](backup-azure-vms.md#offline-vms) benötigt wird.
- Zeit für Momentaufnahme: Die zum Auslösen einer Momentaufnahme erforderliche Zeit. Momentaufnahmen werden kurz vor der geplanten Sicherungszeit ausgelöst.
- Wartezeit in der Warteschlange. Da der Sicherungsdienst Sicherungen von mehreren Kunden verarbeitet, wird das Kopieren der Sicherungsdaten von der Momentaufnahme zum Azure-Sicherungstresor möglicherweise nicht sofort gestartet. Während Spitzenauslastungszeiten können die Wartezeiten aufgrund der Anzahl der verarbeiteten Sicherungen bis zu 8 Stunden betragen. Die Gesamtdauer der VM-Sicherung wird jedoch bei täglichen Sicherungsrichtlinien weniger als 24 Stunden betragen.

## Datenverschlüsselung

Azure Backup verschlüsselt die Daten während des Sicherungsvorgangs nicht. Sie können jedoch die Daten auf dem virtuellen Computer verschlüsseln und dann die geschützten Daten nahtlos sichern (erfahren Sie mehr über das [Sichern verschlüsselter Daten](backup-azure-vms-encryption.md)).


## Wie werden geschützte Instanzen berechnet?
Für virtuelle Azure-Computer, die über Azure Backup gesichert werden, gelten die [Azure Backup-Preise](http://azure.microsoft.com/pricing/details/backup/). Die Berechnung der geschützten Instanzen basiert auf der *tatsächlichen* Größe des virtuellen Computers, d. h. der Summe aller Daten auf dem virtuellen Computer, mit Ausnahme des Ressourcendatenträgers.

Ihnen wird *nicht* die maximal unterstützte Größe für jeden mit dem virtuellen Computer verknüpften Datenträger, sondern die Größe der tatsächlich auf dem jeweiligen Datenträger gespeicherten Daten berechnet. Gleichermaßen basiert die Berechnung der Sicherungsspeicherung auf der Menge der mit Azure Backup gespeicherten Daten, d. h. auf der Summe der tatsächlichen Daten in jedem Wiederherstellungspunkt.

Beispiel: virtueller Computer mit A2-Standardgröße und zwei zusätzlichen Datenträgern mit einer maximalen Größe von jeweils 1 TB. In der folgenden Tabelle sind die auf jedem Datenträger tatsächlich gespeicherten Daten aufgeführt:

|Datenträgertyp|Max. Größe|Tatsächliche Daten|
|---------|--------|------|
| Betriebssystem-Datenträger | 1023 GB | 17 GB |
| Lokaler Datenträger/Ressourcendatenträger | 135 GB | 5 GB (bei der Sicherung nicht enthalten) |
| Datenträger 1 |	1023 GB | 30 GB |
| Datenträger 2 | 1023 GB | 0 GB |

Die *tatsächliche* Größe des virtuellen Computers in diesem Fall ist 17 GB + 30 GB + 0 GB = 47 GB. Dies wird in der monatlichen Rechnung als Größe der geschützten Instanz abgerechnet. Mit zunehmender Datenmenge auf dem virtuellen Computer ändert sich entsprechend auch die Größe der geschützten Instanz, die für die Abrechnung verwendet wird.

Die Abrechnung erfolgt erst nach dem erfolgreichen Abschluss der ersten Sicherung. Zu diesem Zeitpunkt beginnt die Abrechnung für die Speicherung und die geschützten Instanzen. Die Abrechnung erfolgt so lange, wie *Sicherungsdaten mit Azure Backup für den virtuellen Computer gespeichert* werden. Durch Durchführen des Vorgangs "Schutz beenden" wird die Abrechnung nicht beendet, wenn Sicherungsdaten weiterhin beibehalten werden.

Die Abrechnung für einen bestimmten virtuellen Computer endet nur, wenn der Schutz beendet wird *und* alle Sicherungsdaten gelöscht werden. Wenn keine aktiven Sicherungsaufträge vorhanden sind (wenn der Schutz beendet wurde), wird die Größe des virtuellen Computers zum Zeitpunkt der letzten erfolgreichen Sicherung in der monatlichen Rechnung als Größe der geschützten Instanz angesetzt.

## Fragen?
Wenn Sie Fragen haben oder Anregungen zu gewünschten Funktionen mitteilen möchten, [senden Sie uns Ihr Feedback](http://aka.ms/azurebackup_feedback).

## Nächste Schritte

- [Sichern virtueller Computer](backup-azure-vms.md)
- [Verwalten der Sicherung virtueller Computer](backup-azure-manage-vms.md)
- [Wiederherstellen virtueller Computer](backup-azure-restore-vms.md)
- [Problembehandlung bei der Sicherung virtueller Computer](backup-azure-vms-troubleshoot.md)

<!---HONumber=AcomDC_1217_2015-->