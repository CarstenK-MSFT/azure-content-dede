<properties 
	pageTitle="Shard-Zuordnungsverwaltung | Microsoft Azure" 
	description="Erfahren Sie, wie Sie ";ShardMapManager"; und die Clienbtbibliothek für elastische Datenbanken verwenden." 
	services="sql-database" 
	documentationCenter="" 
	manager="jeffreyg" 
	authors="ddove" 
	editor=""/>

<tags 
	ms.service="sql-database" 
	ms.workload="sql-database" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="11/04/2015" 
	ms.author="ddove;sidneyh"/>

# Shard-Zuordnungsverwaltung

Verwenden Sie die [Clientbibliothek für elastische Datenbanken](sql-database-elastic-database-client-library.md) zum Verwalten von horizontal partitionierten Anwendungen. In einer Umgebung mit einer horizontal partitionierten Datenbank verwaltet eine [**Shardzuordnung**](sql-database-elastic-scale-glossary.md) Informationen, die es einer Anwendung ermöglichen, eine Verbindung mit der richtigen Datenbank basierend auf dem Wert des **Shardingschlüssels** herzustellen. Für die Shard-Zuordnungsverwaltung ist es unabdingbar, dass Sie die Grundlagen des Aufbaus dieser Zuordnungen verstehen.

## Shard-Karten und Shard-Zuordnungen
 
### Unterstützte .NET-Typen für Sharding-Schlüssel

Elastic Scale unterstützt die folgenden .net Framework-Typen als Sharding-Schlüssel:

* ganze Zahl
* lang
* GUID
* Byte  
* datetime
* timespan
* datetimeoffset

### Listen- und Bereichs-Shard-Zuordnungen
Shardzuordnungen können mit **Listen von einzelnen Shardingschlüsselwerten** oder mit **Bereichen von Shardingschlüsselwerten** erstellt werden.

###Liste der Shard-Zuordnungen
**Shards** enthalten **Shardlets**, und die Zuordnung von Shardlets zu Shards wird von einer „Shardzuordnung“ verwaltet. Eine **Listen-Shardzuordnung** ist eine Zuordnung zwischen den einzelnen Schlüsselwerten, mit denen die Shardlets und die Datenbanken identifiziert werden, die als Shards dienen. **Listenzuordnungen** sind explizit (Beispiel: Schlüssel 1 ist Datenbank A zugeordnet), und verschiedene Schlüsselwerte können derselben Datenbank zugeordnet werden (Schlüsselwerte 3 und 6 verweisen beide auf Datenbank B).

| Schlüssel | Shard-Speicherort |
|-----|----------------|
| 1 | Datenbank\_A |
| 3 | Datenbank\_B |
| 4 | Datenbank\_C |
| 6 | Datenbank\_B |
| ... | ... |
 

### Bereichs-Shard-Zuordnungen 
In einer **Bereichs-Shardzuordnung** wird der Schlüsselbereich durch ein Paar **[niedriger Wert, hoher Wert)** beschrieben, wobei der *niedrige Wert* den minimalen Schlüssel im Bereich und der *hohe Wert* den ersten Wert darstellt, der höher ist als die Werte in jenem Bereich.

**[0, 100)** enthält z. B. alle ganzen Zahlen größer oder gleich 0 und kleiner 100. Mehrere Bereiche können auf dieselbe Datenbank verweisen, und unzusammenhängende Bereiche werden unterstützt (z. B. [100,200) und [400,600), die im folgenden Beispiel beide auf Database\_C verweisen).

| Schlüssel | Shard-Speicherort |
|-----------|----------------|
| [1,50) | Datenbank\_A |
| [50,100) | Datenbank\_B |
| [100,200) | Datenbank\_C |
| [400,600) | Datenbank\_C |
| ... | ...            

Jede der obigen Tabellen ist ein grundlegendes Beispiel für ein **ShardMap**-Objekt. Jede Zeile ist ein vereinfachtes Beispiel für ein einzelnes **PointMapping**-Objekt (für die Listen-Shardzuordnung) oder für das **RangeMapping**-Objekt (für die Bereichs-Shardzuordnung).

## Shard-Zuordnungs-Manager 

In der Clientbibliothek stellt der Shard-Zuordnungs-Manager eine Auflistung von Shard-Zuordnungen dar. Die von einem **ShardMapManager**-.NET-Objekt verwalteten Daten werden an drei Orten gespeichert:

1. **Globale Shardzuordnung (GSM):** Wenn Sie ein **ShardMapManager**-Element erstellen, geben Sie eine Datenbank als Repository für alle Shard-Karten und -Zuordnungen an. Spezielle Tabellen und gespeicherte Prozeduren werden automatisch zur Verwaltung der Informationen erstellt. Hierfür wird üblicherweise eine kleine Datenbank verwendet, auf die leicht zugegriffen werden kann. Jedoch sollte diese nicht für andere Anforderungen der Anwendung verwendet werden. Die Tabellen befinden sich in einem speziellen Schema namens **\_\_ShardManagement**.

2. **Lokale Shardzuordnung (LSM):** Jede Datenbank, die Sie als Shard innerhalb einer Shardzuordnung angeben, wird dahingehend geändert, dass sie mehrere kleine Tabellen und spezielle gespeicherte Prozeduren enthält, die Shardzuordnungsinformationen speziell für diesen Shard enthalten und verwalten. Diese Informationen sind für die Informationen in der GSM redundant, jedoch kann die Anwendung die zwischengespeicherten Shard Map-Informationen überprüfen, ohne dass Last auf der GSM anfällt. Die Anwendung verwendet die LSM, um festzustellen, ob eine zwischengespeicherte Zuordnung (Mapping) noch gültig ist. Die Tabellen für die LSM zu jedem Shard sind im Schema **\_\_ShardManagement** enthalten.

3. **Anwendungscache:** Jede Anwendungsinstanz, die Zugriff auf ein **ShardMapManager**-Objekt hat, verwaltet lokal einen speicherinternen Cache ihrer Zuordnungen. Sie speichert die Routinginformationen, die zuletzt abgerufen wurden.

## Erstellen eines ShardMapManager
Ein **ShardMapManager**-Objekt wird in der Anwendung mithilfe eines Factory-Musters instanziiert. Die **ShardMapManagerFactory.GetSqlShardMapManager**-Methode übernimmt die Anmeldeinformationen (einschließlich Server- und Datenbankname mit der GSM) in Form eines **ConnectionString**-Elements und gibt die Instanz eines **ShardMapManager**-Elements zurück.

Das **ShardMapManager**-Element sollte nur einmal pro App-Domäne innerhalb des Initialisierungscodes für eine Anwendung instanziiert werden. Ein **ShardMapManager**-Element kann eine beliebige Anzahl von Shard-Zuordnungen enthalten. Während eine einzelne Shard Map für viele Anwendungen ausreichend sein mag, werden in einigen Fällen unterschiedliche Sätze von Datenbanken für ein anderes Schema oder einen eindeutigen Zweck verwendet. In diesen Fällen sind möglicherweise mehrfache Shard Maps vorzuziehen.

In diesem Code versucht eine Anwendung, ein bestehendes **ShardMapManager**-Element zu öffnen. Wenn Objekte, die ein globales **ShardMapManager**-Element darstellen, in der Datenbank noch nicht vorhanden sind, werden sie von der Clientbibliothek dort erstellt.

    // Try to get a reference to the Shard Map Manager via the Shard Map Manager database.  
    // If it doesn't already exist, then create it. 
    ShardMapManager shardMapManager; 
    bool shardMapManagerExists = ShardMapManagerFactory.TryGetSqlShardMapManager(
                                        connectionString, 
                                        ShardMapManagerLoadPolicy.Lazy, 
                                        out shardMapManager); 

    if (shardMapManagerExists) 
     { 
        Console.WriteLine("Shard Map Manager already exists");
    } 
    else
    {
        // Create the Shard Map Manager. 
        ShardMapManagerFactory.CreateSqlShardMapManager(connectionString);
        Console.WriteLine("Created SqlShardMapManager"); 

        shardMapManager = ShardMapManagerFactory.GetSqlShardMapManager(
            connectionString, 
            ShardMapManagerLoadPolicy.Lazy);

        // The connectionString contains server name, database name, and admin credentials 
        // for privileges on both the GSM and the shards themselves.
    } 
 
Alternativ können Sie PowerShell zur Erstellung eines neuen Shard-Zuordnungs-Managers nutzen. Ein Beispiel ist [hier](https://gallery.technet.microsoft.com/scriptcenter/Azure-SQL-DB-Elastic-731883db) verfügbar.

### Administratoranmeldeinformationen für Shard-Zuordnungen

In der Regel unterscheiden sich Anwendungen, die Shard Maps verwalten und bearbeiten, von jenen Anwendungen, die Shard Maps zu Route-Verbindungen verwenden.

Bei Anwendungen, die Shardzuordnungen verwalten (Hinzufügen oder Ändern von Shards, Shard-Karten, Shard-Zuordnungen usw.) , müssen Sie das **ShardMapManager**-Element mithilfe von **Anmeldeinformationen instanziieren, die über Lese-/Schreibzugriff sowohl auf die GSM-Datenbank als auch auf die Datenbank verfügen, die als Shard fungiert**. Die Anmeldeinformationen müssen Schreibvorgänge in Bezug auf die Tabellen sowohl in der GSM als auch in der LSM ermöglichen, wenn Shard Map-Informationen eingegeben oder geändert werden, und sie müssen außerdem das Erstellen von LSM-Tabellen auf neuen Shards erlauben.

### Nur Metadaten betroffen 

Methoden zum Auffüllen oder Ändern der **ShardMapManager**-Daten ändern nicht die Benutzerdaten, die in den Shards selbst gespeichert sind. Beispielsweise wirken sich Methoden wie etwa **CreateShard**, **DeleteShard**, **UpdateMapping** usw. nur auf die Shardzuordnungsmetadaten aus. Sie entfernen keine Benutzerdaten in den Shards, fügen diese hinzu oder ändern sie. Stattdessen sollten diese Methoden in Verbindung mit separaten Vorgängen verwendet werden, die Sie zum Erstellen oder Entfernen der eigentlichen Datenbanken durchführen oder mit denen Sie Zeilen von einem Shard zum nächsten verschieben, um eine Sharding-Umgebung neu auszurichten. (Das **Split-Merge-Tool**, das Teil der Tools für elastische Datenbanken ist, nutzt diese APIs zusammen mit der Durchführung einer tatsächlichen Datenverschiebung zwischen den Shards.)

## Auffüllen einer Shard-Zuordnung: Beispiel
 
Eine Beispielsequenz von Vorgängen zum Auffüllen einer bestimmten Shard Map ist unten dargestellt. Mit dem Code werden folgende Schritte ausgeführt:

1. Eine neue Shard Map wird in einem Shard Map-Manager erstellt. 
2. Die Metadaten für zwei verschiedene Shards werden zur Shard Map hinzugefügt. 
3. Eine Vielzahl von Schlüsselbereichszuordnungen wird hinzugefügt, und der gesamte Inhalt der Shard Map wird angezeigt. 

Der Code ist so geschrieben, dass die gesamte Methode problemlos erneut ausgeführt werden kann, falls ein unerwarteter Fehler auftritt. Jede Anforderung überprüft dann, ob bereits ein Shard oder eine Zuordnung (Mapping) vorhanden ist, bevor ein Versuch unternommen wird, diese zu erstellen. Beim unten angegebenen Code wird davon ausgegangen, dass die Datenbanken namens **sample\_shard\_0**, **sample\_shard\_1** und **sample\_shard\_2** bereits auf dem Server erstellt wurden, auf den über die Zeichenfolge **shardServer** verwiesen wird.

    public void CreatePopulatedRangeMap(ShardMapManager smm, string mapName) 
        {            
            RangeShardMap<long> sm = null; 

            // check if shardmap exists and if not, create it 
            if (!smm.TryGetRangeShardMap(mapName, out sm)) 
            { 
                sm = smm.CreateRangeShardMap<long>(mapName); 
            } 

            Shard shard0 = null, shard1=null; 
            // check if shard exists and if not, 
            // create it (Idempotent / tolerant of re-execute) 
            if (!sm.TryGetShard(new ShardLocation(shardServer, "sample_shard_0"), out shard0)) 
            { 
                Shard0 = sm.CreateShard(new ShardLocation(shardServer, "sample_shard_0")); 
            } 

            if (!sm.TryGetShard(new ShardLocation(shardServer, "sample_shard_1"), out shard1)) 
            { 
                Shard1 = sm.CreateShard(new ShardLocation(shardServer, "sample_shard_1"));  
            } 

            RangeMapping<long> rmpg=null; 

            // Check if mapping exists and if not,
            // create it (Idempotent / tolerant of re-execute) 
            if (!sm.TryGetMappingForKey(0, out rmpg)) 
            { 
                sm.CreateRangeMapping(new RangeMappingCreationInfo<long> 
                    (new Range<long>(0, 50), shard0, MappingStatus.Online)); 
            } 

            if (!sm.TryGetMappingForKey(50, out rmpg)) 
            { 
                sm.CreateRangeMapping(new RangeMappingCreationInfo<long> 
                    (new Range<long>(50, 100), shard1, MappingStatus.Online)); 
            } 

            if (!sm.TryGetMappingForKey(100, out rmpg)) 
            { 
                sm.CreateRangeMapping(new RangeMappingCreationInfo<long> 
                    (new Range<long>(100, 150), shard0, MappingStatus.Online)); 
            } 

            if (!sm.TryGetMappingForKey(150, out rmpg)) 
            { 
                sm.CreateRangeMapping(new RangeMappingCreationInfo<long> 
                    (new Range<long>(150, 200), shard1, MappingStatus.Online)); 
            } 

            if (!sm.TryGetMappingForKey(200, out rmpg)) 
            { 
               sm.CreateRangeMapping(new RangeMappingCreationInfo<long> 
                   (new Range<long>(200, 300), shard0, MappingStatus.Online)); 
            } 

            // List the shards and mappings 
            foreach (Shard s in sm.GetShards()
                                    .OrderBy(s => s.Location.DataSource)
                                    .ThenBy(s => s.Location.Database))
            { 
               Console.WriteLine("shard: "+ s.Location); 
            } 

            foreach (RangeMapping<long> rm in sm.GetMappings()) 
            { 
                Console.WriteLine("range: [" + rm.Value.Low.ToString() + ":" 
                        + rm.Value.High.ToString()+ ")  ==>" +rm.Shard.Location); 
            } 
        } 
 
Als Alternative können Sie PowerShell-Skripts verwenden, um das gleiche Ergebnis zu erzielen. Einige der PowerShell-Beispiele finden Sie [hier](https://gallery.technet.microsoft.com/scriptcenter/Azure-SQL-DB-Elastic-731883db).

Sobald Shard Maps aufgefüllt wurden, können Anwendungen für den Datenzugriff erstellt oder angepasst werden, um mit den Shard Maps arbeiten zu können. Ein Auffüllen oder Bearbeiten der Shardzuordnungen muss so lange nicht erneut erfolgen, bis das **Zuordnungslayout** geändert werden muss.

## Datenabhängiges Routing 

In den meisten Fällen ist die Nutzung des Shard Map-Managers auf jene Anwendungen zurückzuführen, die Verbindungen mit der Datenbank für App-spezifische Datenvorgänge erforderlich machen. In einer Sharding-Anwendung müssen diese Verbindungen mit der richtigen Datenbank verknüpft sein. Dies wird als **datenabhängiges Routing** bezeichnet. Instanziieren Sie für diese Anwendungen ein Shard Map-Managerobjekt ab Werk mit den Anmeldeinformationen, die über schreibgeschützten Zugriff auf die GSM-Datenbank verfügen. Bei einzelnen Anforderungen für Verbindungen werden später die für die Verbindung mit der entsprechenden Shard-Datenbank erforderlichen Anmeldeinformationen zur Verfügung gestellt.

Beachten Sie, dass bei diesen Anwendungen (unter Verwendung eines **ShardMapManager**-Elements, das mit schreibgeschützten Anmeldeinformationen geöffnet wird) keine Änderungen an den Karten oder Zuordnungen vorgenommen werden können. Erstellen Sie für diese Anforderungen administrativ-spezifische Anwendungen oder PowerShell-Skripts, die Anmeldeinformationen mit höheren Berechtigungen, wie bereits erwähnt, zur Verfügung stellen.

Weitere Einzelheiten finden Sie unter [Datenabhängiges Routing](sql-database-elastic-scale-data-dependent-routing.md).

## Ändern von Shard-Zuordnungen 

Eine Shard Map kann auf unterschiedliche Weise geändert werden. Sämtliche der folgenden Methoden ändern die Metadaten, die die Shards und ihre Mappings beschreiben. Sie ändern aber die Daten innerhalb der Shards weder physisch noch erstellen oder löschen sie die Datenbanken. Einige der Vorgänge zur unten beschriebenen Shard Map müssen möglicherweise mit den administrativen Aktionen abgestimmt werden, über welche die Daten physisch verschoben oder über welche die als Shards fungierenden Datenbanken hinzugefügt oder entfernt werden.

Diese Methoden arbeiten zusammen als Bausteine für die Änderung der Gesamtverteilung von Daten in der Sharded-Datenbankumgebung.

* Hinzufügen oder Entfernen von Shards: Verwenden Sie **CreateShard** und **DeleteShard**. 
    
    Der Server und die Datenbank, die das Ziel-Shard repräsentieren, müssen bereits für diese auszuführenden Vorgänge vorhanden sein. Diese Methoden haben keine Auswirkungen auf die Datenbanken selbst, lediglich auf die Metadaten in der Shard Map.

* So erstellen oder entfernen Sie Punkte oder Bereiche, die den Shards zugeordnet sind: Verwenden Sie **CreateRangeMapping**, **DeleteMapping**, **CreatePointMapping**.
    
    Demselben Shard können viele verschiedene Punkte oder Bereiche zugeordnet werden. Diese Methoden wirken sich nur auf Metadaten aus – sie haben keine Auswirkungen auf Daten, die bereits in Shards vorhanden sein können. Wenn Daten aus der Datenbank entfernt werden müssen, damit diese mit **DeleteMapping**-Vorgängen konsistent ist, müssen Sie diese Vorgänge separat (aber im Kontext dieser Methoden) ausführen.

* So unterteilen Sie vorhandene Bereiche in zwei Hälften oder führen benachbarte Bereiche zusammen: Verwenden Sie **SplitMapping** und **MergeMappings**.

    Beachten Sie, dass Aufteilungs- und Zusammenführungsvorgänge **nicht den Shard ändern, dem Schlüsselwerte zugeordnet sind**. Eine Aufteilung teilt einen vorhandenen Bereich in zwei Teile, wobei beide jedoch demselben Shard zugeordnet bleiben. Bei einer Zusammenführung werden zwei benachbarte Bereiche, die bereits demselben Shard zugeordnet sind, zu einem einzigen Bereich zusammengefügt. Das Verschieben von Punkten oder Bereichen zwischen den Shards selbst muss mit **UpdateMapping** in Verbindung mit der eigentlichen Datenverschiebung koordiniert werden. Sie können den **Split-Merge**-Dienst verwenden, der Teil der Tools für elastische Datenbanken ist, um Änderungen an Shardzuordnungen bei Datenverschiebungen zu koordinieren, sofern eine Verschiebung erforderlich ist.

* So ordnen Sie einzelne Punkte oder Bereiche unterschiedlichen Shards neu zu (oder verschieben diese): Verwenden Sie **UpdateMapping**.

    Da Daten unter Umständen aus einem Shard in einen anderen verschoben werden, damit sie mit **UpdateMapping**-Vorgängen konsistent sind, müssen Sie diese Verschiebung separat (aber im Kontext dieser Methoden) ausführen.

* So schalten Sie Zuordnungen online und offline: Verwenden Sie **MarkMappingOffline** und **MarkMappingOnline**, um den Onlinestatus einer Zuordnung zu steuern.

    Bestimmte Vorgänge sind bei Shardzuordnungen nur dann zulässig, wenn für eine Zuordnung der Zustand „offline“ gilt, einschließlich **UpdateMapping** und **DeleteMapping**. Wenn eine Zuordnung offline ist, gibt eine datenabhängige Anforderung auf Grundlage eines Schlüssels, der in dieser Zuordnung enthalten ist, einen Fehler zurück. Wenn darüber hinaus ein Bereich zuerst offline geschaltet wird, werden alle Verbindungen mit dem betroffenen Shard automatisch abgebrochen. So soll verhindert werden, dass inkonsistente oder unvollständige Ergebnisse bei Abfragen gegen jene Bereiche auftreten, die geändert werden.

Zuordnungen sind in .NET unveränderliche Objekte. Alle oben genannten Methoden zum Ändern von Zuordnungen machen auch alle Verweise auf diese in Ihrem Code ungültig. Zur einfacheren Durchführung von Vorgangsabfolgen, die den Zustand einer Zuordnung ändern, geben alle Methoden, die eine Zuordnung ändern, einen neuen Zuordnungsverweis zurück, sodass die Vorgänge verkettet werden können. Um z. B. eine vorhandene Zuordnung in "shardmap sm" mit dem Schlüssel 25 zu löschen, können Sie Folgendes ausführen:

        sm.DeleteMapping(sm.MarkMappingOffline(sm.GetMappingForKey(25)));

## Hinzufügen eines Shards 

Anwendungen müssen häufig einfach neue Shards hinzufügen, um Daten zu verwalten, die von neuen Schlüsseln oder Schlüsselbereichen für eine Shard Map erwartet werden, welche bereits vorhanden ist. Eine Anwendung beispielsweise, bei der Sharding über die Mandanten-ID durchgeführt wird, muss unter Umständen einen neuen Shard für einen neuen Mandanten bereitstellen, oder Daten, bei denen das Sharding monatlich durchgeführt wird, benötigen möglicherweise einen neuen Shard, der vor dem Start eines jeweils neuen Monats bereitgestellt wird.

Wenn der neue Bereich von Schlüsselwerten nicht bereits Teil einer vorhandenen Zuordnung (Mapping) ist und keine Datenmigration erforderlich ist, ist es sehr einfach, den neuen Shard hinzuzufügen und den neuen Schlüssel oder Bereich zu dem Shard zuzuordnen. Weitere Informationen zum Hinzufügen neuer Shards finden Sie unter [Hinzufügen eines neuen Shards](sql-database-elastic-scale-add-a-shard.md).

Bei Szenarios, die eine Datenverschiebung erforderlich machen, wird jedoch das Split-Merge-Tool benötigt, um die Datenverschiebung zwischen Shards in Kombination mit den erforderlichen Shard-Zuordnungsaktualisierungen zu koordinieren. Weitere Informationen zur Verwendung des Split-Merge-Tools finden Sie unter [Übersicht über Split-Merge](sql-database-elastic-scale-overview-split-and-merge.md).

[AZURE.INCLUDE [elastic-scale-include](../../includes/elastic-scale-include.md)]
 

<!---HONumber=Nov15_HO2-->