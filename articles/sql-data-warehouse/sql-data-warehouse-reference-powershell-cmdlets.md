<properties
   pageTitle="Erste Schritte mit SQL Data Warehouse-Cmdlets"
   description="Anhalten und Fortsetzen von SQL Data Warehouse mithilfe von PowerShell-Cmdlets"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="sidneyh"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="12/15/2015"
   ms.author="twounder;sidneyh;barbkess"/>

# Erste Schritte mit Azure Data Warehouse-Cmdlets und REST-APIs

Für die Verwaltung von SQL Data Warehouse können entweder Azure PowerShell-Cmdlets oder REST-APIs verwendet werden.

Die Befehle für **Azure SQL-Datenbank** werden auch für **SQL Data Warehouse** verwendet. Eine aktuelle Liste finden Sie unter [Azure SQL-Cmdlets](https://msdn.microsoft.com/library/mt574084.aspx). Die Cmdlets **Suspend-AzureSqlDatabase** und **Resume-AzureSqlDatabase** (siehe weiter unten) wurden speziell für SQL Data Warehouse hinzugefügt.

Analog dazu können die REST-APIs für **SQL Azure-Datenbank** auch für Instanzen von **SQL Data Warehouse** verwendet werden. Eine aktuelle Liste finden Sie unter [Vorgänge für Azure SQL-Datenbanken](https://msdn.microsoft.com/library/azure/dn505719.aspx).

## Abrufen und Ausführen der Azure PowerShell-Cmdlets

1. Führen Sie zum Herunterladen des Azure PowerShell-Moduls den [Microsoft-Webplattform-Installer](http://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409) aus. 
2. Geben Sie zum Ausführen des Moduls auf der Startseite **Microsoft Azure PowerShell** ein.
3. Falls Sie Ihr Konto noch nicht dem Computer hinzugefügt haben, führen Sie das folgende Cmdlet aus. (Weitere Informationen finden Sie unter [Installieren und Konfigurieren von Azure PowerShell]():

		Add-AzureAccount
3. Verwenden Sie zum Wechseln des Modus das folgende Cmdlet:

		Switch-AzureMode AzureResourceManager

## Suspend-AzureSqlDatabase
### Beispiel 1: Anhalten einer Datenbank auf einem Server anhand des Namens

In diesem Beispiel wird die auf dem Server „Server01“ gehostete Datenbank mit der Bezeichnung „Database02“ angehalten. Der Server befindet sich in einer Azure-Ressourcengruppe namens „ResourceGroup1“.

    Suspend-AzureSqlDatabase –ResourceGroupName "ResourceGroup11" –ServerName
    "Server01" –DatabaseName "Database02"

### Beispiel 2: Anhalten eines Datenbankobjekts

In diesem Beispiel wird eine Datenbank namens „Database02“ von einem Server mit der Bezeichnung „Server01“ abgerufen, der sich in einer Ressourcengruppe namens „ResourceGroup1“ befindet. Das abgerufene Objekt wird an **Suspend-AzureSqlDatabase** weitergeleitet. Dadurch wird die Datenbank angehalten.

	$database = Get-AzureSqlDatabase –ResourceGroupName "ResourceGroup11" –ServerName "Server01" –DatabaseName "Database02"
	$resultDatabase = $database | Suspend-AzureSqlDatabase

## Resume-AzureSqlDatabase

### Beispiel 1: Fortsetzen einer Datenbank auf einem Server anhand des Namens

In diesem Beispiel wird der Betrieb der auf dem Server „Server01“ gehosteten Datenbank mit der Bezeichnung „Database02“ fortgesetzt. Der Server befindet sich in einer Ressourcengruppe namens „ResourceGroup1“.

	Resume-AzureSqlDatabase –ResourceGroupName "ResourceGroup11" –ServerName "Server01" –DatabaseName "Database02"

### Beispiel 2: Fortsetzen eines Datenbankobjekts

In diesem Beispiel wird eine Datenbank namens „Database02“ von einem Server mit der Bezeichnung „Server01“ abgerufen, der sich in einer Ressourcengruppe namens „ResourceGroup1“ befindet. Das Objekt wird an **Resume-AzureSqlDatabase** übergeben.

	$database = Get-AzureSqlDatabase –ResourceGroupName "ResourceGroup11" –ServerName "Server01" –DatabaseName "Database02"
	$resultDatabase = $database | Resume-AzureSqlDatabase

## Get-AzureSqlDatabaseRestorePoints

Dieses Cmdlet führt die Sicherungswiederherstellungspunkte für eine Azure SQL-Datenbank auf. Die Wiederherstellungspunkte dienen zum Wiederherstellen der Datenbank. Das zurückgegebene Objekt besitzt folgende Eigenschaften:

Eigenschaft|Beschreibung
---|---
RestorePointType|DISCRETE/CONTINUOUS. Diskrete Wiederherstellungspunkte beschreiben die möglichen Zeitpunkte, zu denen eine Azure SQL-Datenbank wiederhergestellt werden kann. Kontinuierliche Wiederherstellungspunkte beschreiben die frühestmöglichen Zeitpunkte, zu denen eine Azure SQL-Datenbank wiederhergestellt werden kann. Die Datenbank kann zu jedem Zeitpunkt wiederhergestellt werden, der hinter dem frühestmöglichen Zeitpunkt liegt.
EarliestRestoreDate|Früheste Wiederherstellungszeit (wird aufgefüllt, wenn "restorePointType" = CONTINUOUS ist)
RestorePointCreationDate |Uhrzeit der Sicherungsmomentaufnahme (aufgefüllt, wenn "restorePointType = DISCRETE")

### Beispiel 1: Abrufen der Wiederherstellungspunkte einer Datenbank auf einem Server anhand des Namens
In diesem Beispiel werden die Wiederherstellungspunkte für eine Datenbank namens „Database02“ von einem Server mit der Bezeichnung „Server01“ abgerufen, der sich in einer Ressourcengruppe namens „ResourceGroup1“ befindet.

	$restorePoints = Get-AzureSqlDatabaseRestorePoints –ResourceGroupName "ResourceGroup11" –ServerName "Server01" –DatabaseName "Database02"



### Beispiel 2: Fortsetzen eines Datenbankobjekts

In diesem Beispiel wird eine Datenbank namens „Database02“ von einem Server mit der Bezeichnung „Server01“ abgerufen, der sich in einer Ressourcengruppe namens „ResourceGroup1“ befindet. Das Datenbankobjekt wird an **Get-AzureSqlDatabase** übergeben, was in den Wiederherstellungspunkten der Datenbank resultiert.

	$database = Get-AzureSqlDatabase –ResourceGroupName "ResourceGroup11" –ServerName "Server01" –DatabaseName "Database02"
	$restorePoints = $database | Get-AzureSqlDatabaseRestorePoints



> [AZURE.NOTE]Hinweis: Wenn Ihr Server „foo.database.windows.net“ heißt, verwenden Sie in den PowerShell-Cmdlets für „-ServerName“ den Namen „foo“.


## Nächste Schritte
Weitere Referenzinformationen finden Sie unter [SQL Data Warehouse-Referenz – Übersicht][].

<!--Image references-->

<!--Article references-->
[SQL Data Warehouse-Referenz – Übersicht]: sql-data-warehouse-overview-reference.md
[How to install and configure Azure PowerShell]: ../articles/powershell-install-configure.md

<!--MSDN references-->


<!--Other Web references-->
[gog]: http://google.com/
[yah]: http://search.yahoo.com/
[msn]: http://search.msn.com/

<!---HONumber=AcomDC_1217_2015-->