<properties 
	pageTitle="Code für XEvent-Ringpuffer für SQL-Datenbank | Microsoft Azure" 
	description="Zeigt ein Transact-SQL-Codebeispiel (für Azure SQL-Datenbank), das durch die Verwendung des Ringpufferziels eine einfache und schnelle Methode zum Erfassen und Ausgeben von Daten bietet." 
	services="sql-database" 
	documentationCenter="" 
	authors="MightyPen" 
	manager="jeffreyg" 
	editor="" 
	tags=""/>


<tags 
	ms.service="sql-database" 
	ms.workload="data-management" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="10/22/2015" 
	ms.author="genemi"/>


# Code des Ringpufferziels für erweiterte Ereignisse in SQL-Datenbank


Sie suchen ein vollständiges Codebeispiel für die einfachste schnelle Möglichkeit zum Erfassen und Melden von Informationen für erweiterte Ereignisse während eines Tests. Das einfachste Ziel für erweiterte Ereignisdaten ist das [Ringpufferziel](http://msdn.microsoft.com/library/ff878182.aspx).


In diesem Thema finden Sie ein Transact-SQL-Codebeispiel, mit dem folgende Aufgaben ausgeführt werden:


1. Erstellen einer Tabelle mit Daten zur Veranschaulichung.

2. Erstellen einer Sitzung für ein vorhandenes erweitertes Ereignis (**sqlserver.sql\_statement\_starting**).
 - Das Ereignis ist auf SQL-Anweisungen beschränkt, die eine bestimmte Update-Zeichenfolge enthalten: **statement LIKE '%UPDATE tabEmployee%'**.
 - Senden der Ausgabe des Ereignisses an ein Ziel vom Typ Ringpuffer (**package0.ring\_buffer**).

3. Starten der Ereignissitzung.

4. Ausgeben einer Reihe einfacher SQL UPDATE-Anweisungen.

5. Ausgeben einer SQL SELECT-Anweisung, um die Ereignisausgabe vom Ringpuffer abzurufen.
 - **sys.dm\_xe\_database\_session\_targets** und andere dynamische Verwaltungssichten (DMVs) werden verknüpft.

6. Anhalten der Ereignissitzung.

7. Verwerfen des Ringpufferziels, um die zugehörigen Ressourcen freizugeben.

8. Verwerfen der Ereignissitzung und der Demotabelle.


## Voraussetzungen


- Ein Azure-Konto und ein Azure-Abonnement. Sie können sich für eine [kostenlose Testversion](http://azure.microsoft.com/pricing/free-trial/) registrieren.


- Jede Datenbank, in der eine Tabelle erstellt werden kann.
 - Optional können Sie in wenigen Minuten [eine **AdventureWorksLT**-Demodatenbank erstellen](sql-database-get-started.md).


- SQL Server Management Studio ("ssms.exe"), Vorschauversion August 2015 oder eine spätere Version. Sie können "ssms.exe" in der neuesten Version über eine der folgenden Methoden herunterladen:
 - [Über einen Link im Thema.](http://msdn.microsoft.com/library/mt238290.aspx)
 - [Über einen direkten Link zum Herunterladen.](http://go.microsoft.com/fwlink/?linkid=616025)
 - Microsoft empfiehlt, dass Sie die Datei "ssms.exe" in regelmäßigen Abständen aktualisieren.


## Codebeispiel


Das folgende Codebeispiel für einen Ringpuffer kann mit kleineren Änderungen sowohl für Azure SQL-Datenbank als auch für Microsoft SQL Server ausgeführt werden. Der Unterschied besteht im Vorhandensein des Knotens "\_database" im Namen einiger dynamischer Verwaltungssichten (DMVs) in der FROM-Klausel in Schritt 5. Beispiel:

- sys.dm\_xe**\_database**\_session\_targets
- sys.dm\_xe\_session\_targets


&nbsp;


```
GO
----  Transact-SQL.
---- Step set 1.

SET NOCOUNT ON;
GO


IF EXISTS
	(SELECT * FROM sys.objects
		WHERE type = 'U' and name = 'tabEmployee')
BEGIN
	DROP TABLE tabEmployee;
END
GO


CREATE TABLE tabEmployee
(
	EmployeeGuid         uniqueIdentifier   not null  default newid()  primary key,
	EmployeeId           int                not null  identity(1,1),
	EmployeeKudosCount   int                not null  default 0,
	EmployeeDescr        nvarchar(256)          null
);
GO


INSERT INTO tabEmployee ( EmployeeDescr )
	VALUES ( 'Jane Doe' );
GO

---- Step set 2.


IF EXISTS
	(SELECT * from sys.database_event_sessions
		WHERE name = 'eventsession_gm_azuresqldb51')
BEGIN
	DROP EVENT SESSION eventsession_gm_azuresqldb51
		ON DATABASE;
END
GO


CREATE
	EVENT SESSION eventsession_gm_azuresqldb51
	ON DATABASE
	ADD EVENT
		sqlserver.sql_statement_starting
			(
			ACTION (sqlserver.sql_text)
			WHERE statement LIKE '%UPDATE tabEmployee%'
			)
	ADD TARGET
		package0.ring_buffer
			(SET
				max_memory = 500   -- Units of KB.
			);
GO

---- Step set 3.


ALTER EVENT SESSION eventsession_gm_azuresqldb51
	ON DATABASE
	STATE = START;
GO

---- Step set 4.


SELECT 'BEFORE_Updates', EmployeeKudosCount, * FROM tabEmployee;

UPDATE tabEmployee
	SET EmployeeKudosCount = EmployeeKudosCount + 102;

UPDATE tabEmployee
	SET EmployeeKudosCount = EmployeeKudosCount + 1015;

SELECT 'AFTER__Updates', EmployeeKudosCount, * FROM tabEmployee;
GO

---- Step set 5.


SELECT
		se.name  AS [session-name],
		ev.event_name,
		ac.action_name,
		st.target_name,
		se.session_source,
		st.target_data,
		CAST(st.target_data AS XML)  AS [target_data_XML]
	FROM
				   sys.dm_xe_database_session_event_actions  AS ac
		INNER JOIN sys.dm_xe_database_session_events         AS ev  ON ev.event_name            = ac.event_name
		                                                           AND ev.event_session_address = ac.event_session_address
		INNER JOIN sys.dm_xe_database_session_object_columns AS oc  ON oc.event_session_address = ac.event_session_address
		INNER JOIN sys.dm_xe_database_session_targets        AS st  ON st.event_session_address = ac.event_session_address
		INNER JOIN sys.dm_xe_database_sessions               AS se  ON ac.event_session_address = se.address
	WHERE
		oc.column_name = 'occurrence_number'
		AND
		se.name        = 'eventsession_gm_azuresqldb51'
		AND
		ac.action_name = 'sql_text'
	ORDER BY
		se.name,
		ev.event_name,
		ac.action_name,
		st.target_name,
		se.session_source;
GO

---- Step set 6.


ALTER EVENT SESSION eventsession_gm_azuresqldb51
	ON DATABASE
	STATE = STOP;
GO

---- Step set 7.


ALTER EVENT SESSION eventsession_gm_azuresqldb51
	ON DATABASE
	DROP TARGET package0.ring_buffer;
GO

---- Step set 8.


DROP EVENT SESSION eventsession_gm_azuresqldb51
	ON DATABASE;
GO

DROP TABLE tabEmployee;
GO
```


&nbsp;


## Ringpufferinhalte


Zum Ausführen des Codebeispiels wurde "ssms.exe" verwendet.


Um die Ergebnisse anzuzeigen, haben wir auf die Zelle unterhalb der Spaltenüberschrift **target\_data\_XML** geklickt.

Anschließend haben wir im Ergebnisbereich auf die Zelle unterhalb der Spaltenüberschrift **target\_data\_XML** geklickt. Dadurch wurde in "ssms.exe" eine weitere Dateiregisterkarte erstellt, auf welcher der Inhalt der Ergebniszelle angezeigt wurde (als XML).


Der folgende Block zeigt die Ausgabe. Wenngleich die Ausgabe lang erscheint, umfasst sie lediglich zwei **<event>**-Elemente.


&nbsp;


```
<RingBufferTarget truncated="0" processingTime="0" totalEventsProcessed="2" eventCount="2" droppedCount="0" memoryUsed="1728">
  <event name="sql_statement_starting" package="sqlserver" timestamp="2015-09-22T15:29:31.317Z">
    <data name="state">
      <type name="statement_starting_state" package="sqlserver" />
      <value>0</value>
      <text>Normal</text>
    </data>
    <data name="line_number">
      <type name="int32" package="package0" />
      <value>7</value>
    </data>
    <data name="offset">
      <type name="int32" package="package0" />
      <value>184</value>
    </data>
    <data name="offset_end">
      <type name="int32" package="package0" />
      <value>328</value>
    </data>
    <data name="statement">
      <type name="unicode_string" package="package0" />
      <value>UPDATE tabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 102</value>
    </data>
    <action name="sql_text" package="sqlserver">
      <type name="unicode_string" package="package0" />
      <value>
---- Step set 4.


SELECT 'BEFORE_Updates', EmployeeKudosCount, * FROM tabEmployee;

UPDATE tabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 102;

UPDATE tabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 1015;

SELECT 'AFTER__Updates', EmployeeKudosCount, * FROM tabEmployee;
</value>
    </action>
  </event>
  <event name="sql_statement_starting" package="sqlserver" timestamp="2015-09-22T15:29:31.327Z">
    <data name="state">
      <type name="statement_starting_state" package="sqlserver" />
      <value>0</value>
      <text>Normal</text>
    </data>
    <data name="line_number">
      <type name="int32" package="package0" />
      <value>10</value>
    </data>
    <data name="offset">
      <type name="int32" package="package0" />
      <value>340</value>
    </data>
    <data name="offset_end">
      <type name="int32" package="package0" />
      <value>486</value>
    </data>
    <data name="statement">
      <type name="unicode_string" package="package0" />
      <value>UPDATE tabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 1015</value>
    </data>
    <action name="sql_text" package="sqlserver">
      <type name="unicode_string" package="package0" />
      <value>
---- Step set 4.


SELECT 'BEFORE_Updates', EmployeeKudosCount, * FROM tabEmployee;

UPDATE tabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 102;

UPDATE tabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 1015;

SELECT 'AFTER__Updates', EmployeeKudosCount, * FROM tabEmployee;
</value>
    </action>
  </event>
</RingBufferTarget>
```


#### Freigeben der von Ihrem Ringpuffer verwendeten Ressourcen


Wenn Sie mit der Verwendung Ihres Ringpuffers fertig sind, können Sie ihn entfernen und seine Ressourcen freigeben. Verwenden Sie dazu wie nachfolgend gezeigt einen **ALTER**-Befehl:


```
ALTER EVENT SESSION eventsession_gm_azuresqldb51
	ON DATABASE
	DROP TARGET package0.ring_buffer;
GO
```


Die Definition Ihrer Ereignissitzung wird aktualisiert, jedoch nicht verworfen. Sie können zu einem späteren Zeitpunkt eine weitere Instanz des Ringpuffers zu Ihrer Ereignissitzung hinzufügen.


```
ALTER EVENT SESSION eventsession_gm_azuresqldb51
	ON DATABASE
	ADD TARGET
		package0.ring_buffer
			(SET
				max_memory = 500   -- Units of KB.
			);
```


## Weitere Informationen


Hauptthemen für erweiterte Ereignisse in Azure SQL-Datenbank:


- [Überlegungen zu erweiterten Ereignissen in SQL-Datenbank](sql-database-xevent-db-diff-from-svr.md): In diesem Thema werden verschiedene Aspekte im Zusammenhang mit erweiterten Ereignissen gegenübergestellt, die sich bei Azure SQL-Datenbank und Microsoft SQL Server unterscheiden.


Weitere Themen mit Codebeispielen für erweiterte Ereignisse finden Sie unter den folgenden Links. Allerdings müssen Sie routinemäßig alle Beispiele dahingehend prüfen, ob das Beispiel für Microsoft SQL Server oder Azure SQL-Datenbank gilt. Dann können Sie entscheiden, ob kleinere Änderungen erforderlich sind, um das Beispiel auszuführen.


- Codebeispiel für Azure SQL-Datenbank: [Code des Ereignisdateiziels für erweiterte Ereignisse in SQL-Datenbank](sql-database-xevent-code-event-file.md)


<!--
('lock_acquired' event.)

- Code sample for SQL Server: [Determine Which Queries Are Holding Locks](http://msdn.microsoft.com/library/bb677357.aspx)
- Code sample for SQL Server: [Find the Objects That Have the Most Locks Taken on Them](http://msdn.microsoft.com/library/bb630355.aspx)
-->

<!---HONumber=Nov15_HO2-->