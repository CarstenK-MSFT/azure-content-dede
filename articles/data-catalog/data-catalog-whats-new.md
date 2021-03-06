<properties
   pageTitle="Neuigkeiten in Azure Data Catalog"
   description="Übersicht über neue Funktionen in der Azure Data Catalog-Vorschauversion."
   services="data-catalog"
   documentationCenter=""
   authors="steelanddata"
   manager="NA"
   editor=""
   tags=""/>
<tags
   ms.service="data-catalog"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-catalog"
   ms.date="11/20/2015"
   ms.author="maroche"/>

# Neuigkeiten in Azure Data Catalog

Updates für **Azure Data Catalog** werden in regelmäßigen Abständen veröffentlicht. Nicht jede Version umfasst neue Funktionen für Benutzer, da sich einige Versionen auf Back-End-Dienstfunktionen konzentrieren. Auf dieser Seite werden neue Funktionen für Benutzer aufgezeigt, die dem Dienst **Azure Data Catalog** hinzugefügt wurden.

## Neuigkeiten in der Version der Woche ab dem 20. November 2015

In der Woche ab dem 20. November 2015 wurden **Azure Data Catalog** die folgenden Funktionen hinzugefügt:

- Möglichkeit zum Anzeigen und Kopieren von Verbindungszeichenfolgen über das **Azure Data Catalog**-Portal für SQL Server- (einschließlich Azure SQL-Datenbank) und Oracle-Datenquellen. Benutzer können in den Verbindungsinformationen für eine Tabelle, Ansicht oder Datenbank von SQL Server oder Oracle auf den Link „Verbindungszeichenfolgen anzeigen“ klicken, um die Verbindungszeichenfolgen zum Herstellen der Verbindung mit der Datenquelle anzuzeigen. Für SQL Server-Datenquellen werden ADO.NET-, ODBC-, OLEDB- und JDBC-Verbindungszeichenfolgen bereitgestellt. Für Oracle-Datenquellen werden ODBC- und OLEDB-Verbindungszeichenfolgen bereitgestellt.
- Unterstützung für das Einbinden von Datenprofilen, wenn Teradata-Tabellen und -Ansichten registriert werden.
- Unterstützung für „Open in Power BI Desktop“ für SQL Server-Datenquellen (einschließlich Azure SQL-Datenbank und Azure SQL Data Warehouse), SQL Server Analysis Services-, Azure Storage- und HDFS-Quellen.  

> [AZURE.NOTE]Für „Open in Power BI Desktop“ muss eine aktuelle Version der Anwendung Power BI Desktop installiert sein. Wenn bei der Verwendung dieser Funktion Probleme oder Fehler auftreten, vergewissern Sie sich, dass Sie die neueste Version von Power BI Desktop von [PowerBI.com](https://powerbi.com) heruntergeladen haben.

## Neuigkeiten in der Version der Woche ab dem 13. November 2015

In der Woche ab dem 13. November 2015 wurden **Azure Data Catalog** die folgenden Funktionen hinzugefügt:

- Unterstützung für die LDAP-Authentifizierung für Teradata-Datenquellen. Bei der Registrierung von Teradata-Tabellen und -Ansichten können Benutzer festlegen, dass die Herstellung der Verbindung mit Teradata mithilfe der LDAP-Authentifizierung und der TD2-Authentifizierung erfolgt.
- Unterstützung für „In Excel öffnen“ für Teradata-Datenquellen.
- Unterstützung für zuletzt verwendete Suchbegriffe im **Azure Data Catalog**-Portal. Bei der Suche im Portal können Benutzer die zuletzt verwendeten Suchbegriffe auswählen, um den Ermittlungsvorgang zu beschleunigen.

## Neuigkeiten in der Version der Woche ab dem 6. November 2015

In der Woche ab dem 6. November 2015 wurden **Azure Data Catalog** die folgenden Funktionen hinzugefügt:

- Unterstützung für die Vorschau für Teradata-Datenquellen. Beim Registrieren von Teradata-Tabellen und -Sichten können Benutzer festlegen, dass Momentaufnahmedatensätze mit den extrahierten Metadaten aus der Datenquelle eingeschlossen werden sollen.
- Unterstützung für „In Excel öffnen“ für Azure SQL Data Warehouse-Datenquellen
- Unterstützung für die Definition und Bearbeitung von Schemas auf Spaltenebene für manuell registrierte Datenassets. Nach dem manuellen Erstellen eines Datenassets mit dem **Azure Data Catalog**-Portal können Benutzer in den Eigenschaften der Datenassets Spaltendefinitionen hinzufügen.
- Unterstützung für Abfragen vom Typ „has“ beim Durchsuchen von **Azure Data Catalog**, um die Ermittlung von registrierten Datenressourcen mit bestimmten Metadaten zu ermöglichen. Die **Azure Data Catalog**-Abfragesyntax enthält nun Folgendes:

| Abfragesyntax | Zweck |
|-------------------------|---------|
| has:previews | Sucht nach Datenassets, die eine Vorschau beinhalten. |
| has:documentation | Sucht nach Datenassets, für die eine Dokumentation bereitgestellt wurde. |
| has:tableDataProfiles | Sucht nach Datenassets mit Datenprofilinformationen auf Tabellenebene. |
| has:columnsDataProfiles | Sucht nach Datenressourcen mit Datenprofilinformationen auf Spaltenebene. |


## Neuigkeiten in der Version der Woche ab dem 30. Oktober 2015

In der Woche ab dem 30. Oktober 2015 wurden **Azure Data Catalog** die folgenden Funktionen hinzugefügt:

- Unterstützung für die Verschlüsselung ruhender Daten bei der Datenvorschau und in Datenprofilen für registrierte Datenquellen. **Azure Data Catalog** verschlüsselt alle Datenquellen von Vorschaudatensätzen und Datenprofilen, die im Dienst registriert sind, auf transparente Weise. Schlüsselmanagement durch Katalogadministratoren ist nicht erforderlich.

## Neuerungen in der Version der Woche ab dem 23. Oktober 2015

In der Woche ab dem 23. Oktober 2015 wurden **Azure Data Catalog** die folgenden Funktionen hinzugefügt:

- Unterstützung für Teradata-Datenquellen. Benutzer können nun Teradata-Tabellen und -Ansichten registrieren und ermitteln.

> [AZURE.NOTE]Bei der aktuellen Version wird nur die Teradata TD2-Authentifizierung unterstützt. Zusätzliche Authentifizierungsmethoden werden in zukünftigen Versionen hinzugefügt.

## Neuigkeiten in der Version der Woche ab dem 16. Oktober 2015

In der Woche ab dem 16. Oktober 2015 wurden **Azure Data Catalog** die folgenden Funktionen hinzugefügt:

- Unterstützung für lokale Hive-Datenquellen. Benutzer können jetzt Hive-Tabellen für Apache Hive in Hadoop in lokalen Datenquellen registrieren und ermitteln.
- Unterstützung für gespeicherte Suchen im **Azure Data Catalog**-Portal. Benutzer können Suchbegriffe speichern und die Auswahl filtern, um vorherige Suchen leicht wiederholen zu können und hilfreiche Ansichten des Kataloginhalts zu definieren. Außerdem können Benutzer eine gespeicherte Suche als Standardsuche markieren. Wenn ein Benutzer auf der Startseite des **Azure Data Catalog**-Portals oder auf der Seite mit den ersten Schritten auf das Suchsymbol mit der Lupe klickt, wird der Benutzer direkt zur gespeicherten Suche geleitet, die als Standard markiert ist.


## Neuerungen in der Version der Woche ab dem 9. Oktober 2015

In der Woche ab dem 9. Oktober 2015 wurden **Azure Data Catalog** die folgenden Funktionen hinzugefügt:

- Unterstützung für Rich Text-Dokumentation für registrierte Datenbestände und Container im Azure Data Catalog-Portal. Benutzer können jetzt Dokumentation für Datenbestände, z. B. Tabellen, Ansichten und Berichte, sowie für Container, z. B. Datenbanken und Modelle, für Szenarien bereitstellen, bei denen Tags und Beschreibungen nicht ausreichen.

## Neuigkeiten in der Version der Woche ab dem 2. Oktober 2015

In der Woche ab dem 2. Oktober 2015 wurden **Azure Data Catalog** die folgenden Funktionen hinzugefügt:

- Unterstützung für die manuelle Registrierung bekannter Datenquellentypen. Benutzer können Datenquelleninformationen manuell über das **Azure Data Catalog**-Portal für alle Datentypen eingeben, die von **Azure Data Catalog** unterstützt werden.
- Unterstützung für die Autorisierung von Azure Active Directory-Sicherheitsgruppen. Katalogadministratoren können den Katalogzugriff auf Sicherheitsgruppen und auf Benutzerkonten ermöglichen, um die Verwaltung des Zugriffs auf **Azure Data Catalog** zu vereinfachen.
- Unterstützung für das Öffnen von Hive-Datenquellen in Excel über das **Azure Data Catalog**-Portal.

> [AZURE.NOTE]Zum Verwenden des Features „In Excel öffnen“ für Hive-Datenquellen müssen Benutzer den ODBC-Treiber für Hive installieren.

## Neuigkeiten in der Version der Woche ab dem 25. September 2015

In der Woche ab dem 25. September 2015 wurden **Azure Data Catalog** die folgenden Funktionen hinzugefügt:

- Unterstützung für das Einbinden von Datenprofilen, wenn Hive-Datenquellen registriert werden.
- Unterstützung für programmgesteuertes Ermitteln der Catalog-API, wodurch es einfacher wird, Anwendungen in **Azure Data Catalog** zu integrieren.

## Neuigkeiten in der Version der Woche ab dem 18. September 2015

In der Woche ab dem 18. September 2015 wurden **Azure Data Catalog** die folgenden Funktionen hinzugefügt:

- Eine neue Übersicht über die Möglichkeiten zur Ermittlung von Datenquellen im **Azure Data Catalog**-Portal. Wenn Benutzer die Seite „Ermitteln“ des **Azure Data Catalog**-Portals aufrufen, ohne einen Suchbegriff einzugeben, wird eine Übersicht der Kataloginhalte angezeigt, einschließlich der am häufigsten verwendeten Tags sowie der Experten, Datenquellentypen und Objekttypen.
- Unterstützung für das Registrieren und Ermitteln von Azure SQL Data Warehouse-Objekten und -Datenbanken. Weitere Informationen zu Azure SQL Data Warehouse finden Sie unter [SQL Data Warehouse](http://azure.microsoft.com/services/sql-data-warehouse/).
- Unterstützung für das Registrieren und Ermitteln von SQL Server Analysis Services-Modellen und SQL Server Reporting Services-Servern als Container. **Azure Data Catalog** erstellt beim Registrieren von SSAS- und SSRS-Objekten einen Eintrag für das SSAS-Modell und den SSRS-Server sowie für die Berichte und andere Objekte. Die Container können über das **Azure Data Catalog**-Portal ermittelt und mit Anmerkungen versehen werden. Benutzer können neben den Inhalten des Katalogs auch die Inhalte eines Modells oder Servers durchsuchen und filtern.

> [AZURE.NOTE]SSAS- und SSRS-Objekte, die vor der Version vom 18. September registriert wurden, müssen mithilfe des Registrierungstools für Datenquellen erneut registriert werden, bevor der Modell- oder Servereintrag dem Katalog hinzugefügt wird. Das erneute Registrieren einer Datenquelle wirkt sich nicht auf Anmerkungen aus, die von Benutzern im **Azure Data Catalog**-Portal hinzugefügt wurden.

## Neuigkeiten in der Version der Woche ab dem 11. September 2015

In der Woche ab dem 11. September 2015 wurden **Azure Data Catalog** die folgenden Funktionen hinzugefügt:

- Unterstützung für das Registrieren und die Ermittlung von SQL Server Analysis Services-Objekten über HTTP/HTTPS. Benutzer können jetzt mithilfe einer URL (z. B. https://servername/olap/msmdpump.dll) anstelle eines Servernamens eine Verbindung mit SSAS-Servern herstellen und die Standardauthentifizierung und anonyme Verbindungen zusätzlich zur Windows-Authentifizierung verwenden. Weitere Informationen zu HTTP/HTTPS-Verbindungen mit SSAS finden Sie unter [Konfigurieren des HTTP-Zugriffs auf Analysis Services](https://msdn.microsoft.com/library/gg492140.aspx).
- Unterstützung für die Hive-Datenquellen auf HDInsight. Benutzer können jetzt Hive-Tabellen für Apache Hive in Hadoop in HDInsight-Datenquellen registrieren und ermitteln. Weitere Informationen zu Hive in HDInsight finden Sie im [HDInsight Documentation Center](../hdinsight-use-hive/).
- Unterstützung für das Registrieren und Ermitteln von Oracle-Datenbanken und HDFS-Clustern als Container Beim Registrieren von Oracle-Tabellen und -Sichten oder HDFS erstellt **Azure Data Catalog** einen Eintrag für die Datenbank sowie für die Tabellen und Sichten. Die Datenbank kann über das **Azure Data Catalog**-Portal ermittelt und mit Anmerkungen versehen werden. Benutzer können neben den Inhalten des Katalogs auch die Inhalte einer Datenbank oder eines Clusters durchsuchen und filtern.


> [AZURE.NOTE]Oracle-Tabellen und -Ansichten und HDFS-Dateien und -Verzeichnisse, die vor der Version vom 11. September registriert wurden, müssen mithilfe des Registrierungstools für Datenquellen erneut registriert werden, bevor der Datenbank- oder Clustereintrag dem Katalog hinzugefügt wird. Das erneute Registrieren einer Datenquelle wirkt sich nicht auf Anmerkungen aus, die von Benutzern im **Azure Data Catalog**-Portal hinzugefügt wurden.

## Neuigkeiten in der Version der Woche ab dem 4. September 2015

In der Woche ab dem 4. September 2015 wurden **Azure Data Catalog** die folgenden Funktionen hinzugefügt:

- Unterstützung für die manuelle Registrierung unbekannter Datenquellentypen Benutzer können über das **Azure Data Catalog**-Portal manuell Datenquelleninformationen eingeben, sodass Datenquellen, die nicht explizit vom Quellregistrierungstool unterstützt werden, mit Anmerkungen versehen und ermittelt werden können.
- Unterstützung für das Registrieren und Ermitteln von SQL Server-Datenbanken als Container Beim Registrieren von SQL Server-Tabellen und -Sichten erstellt **Azure Data Catalog** einen Eintrag für die Datenbank sowie für die Tabellen und Sichten. Die Datenbank kann über das **Azure Data Catalog**-Portal ermittelt und mit Anmerkungen versehen werden. Benutzer können neben den Inhalten des Katalogs auch die Inhalte einer Datenbank durchsuchen und filtern.

> [AZURE.NOTE]SQL Server-Tabellen und -Ansichten, die vor der Version vom 4. September registriert wurden, müssen mithilfe des Registrierungstools für Datenquellen erneut registriert werden, bevor der Datenbankeintrag dem Katalog hinzugefügt wird. Das erneute Registrieren einer Datenquelle wirkt sich nicht auf Anmerkungen aus, die von Benutzern im **Azure Data Catalog**-Portal hinzugefügt wurden.

## Neuigkeiten in der Version der Woche ab dem 28. August 2015

In der Woche ab dem 28. August 2015 wurden **Azure Data Catalog** die folgenden Funktionen hinzugefügt:

- Unterstützung für die Datenprofilerstellung von SQL Server- und Oracle-Datenquellen. Beim Registrieren von SQL Server- und Oracle-Tabellen und -Sichten können Benutzer optional Datenprofilinformationen für die registrierten Objekte einbeziehen. Das Datenprofil umfasst Statistiken auf Objektebene und auf Spaltenebene.
- Unterstützung für Hadoop HDFS-Datenquellen. Benutzer können nun HDFS-Dateien und -Verzeichnisse registrieren und ermitteln.

## Neuigkeiten in der Version der Woche ab dem 21. August 2015

Ab der Woche beginnend am 21. August 2015 wurden **Azure Data Catalog** die folgenden Funktionen hinzugefügt:

- Unterstützung zur Bereitstellung von Zugriffsanforderungsinformationen für registrierte Datenquellen. Benutzer können nun zur einfachen Integration mit vorhandenen Tools oder Prozessen für alle registrierten Datenassets Anweisungen für Zugriffsanforderungen angeben, z. B. E-Mail-Links oder URLs.
- QuickInfos für Tags und Experten, sodass leichter ermittelt werden kann, welche Benutzer welche Metadaten für registrierte Datenassets angegeben haben.
- In der oberen Navigationsleiste wurde die neue Schaltfläche "Benutzer" mit dem entsprechenden Menü hinzugefügt. In diesem Menü kann der Benutzer das für die Anmeldung bei **Azure Data Catalog** verwendete Konto anzeigen und sich bei Bedarf abmelden. In diesem Menü wird zudem der Katalogname angezeigt. Dies ist nützlich für Entwickler, die die **Azure Data Catalog**-REST-API verwenden.
- Nur Standard Edition: Beim Hinzufügen von Besitzern zu Datenbeständen unterstützt **Azure Data Catalog** jetzt sowohl Benutzerkonten als auch Sicherheitsgruppen als Besitzer. Um eine Sicherheitsgruppe als Besitzer für ausgewählte Datenasset hinzuzufügen, können Sie entweder den Anzeigenamen der Gruppe oder die UPN-E-Mail-Adresse der Gruppe (sofern vorhanden) eingeben.
- Unterstützung für Datenquellen in Azure-Blob-Speichern. Benutzer können nun Azure Storage-Blobs und -Verzeichnisse registrieren und ermitteln.

<!---HONumber=AcomDC_1125_2015-->