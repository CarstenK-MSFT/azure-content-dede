<properties 
	pageTitle="Planung und Ausführung mit Data Factory" 
	description="Informationen zu den Planungs- und Ausführungsaspekten des Azure Data Factory-Anwendungsmodells." 
	services="data-factory" 
	documentationCenter="" 
	authors="spelluru" 
	manager="jhubbard" 
	editor="monicar"/>

<tags 
	ms.service="data-factory" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="10/20/2015" 
	ms.author="spelluru"/>

# Planung und Ausführung mit Data Factory
  
In diesem Artikel werden die Planungs- und Ausführungsaspekte des Azure Data Factory-Anwendungsmodells erläutert. Dieser Artikel basiert auf den Artikeln [Erstellen von Pipelines](data-factory-create-pipelines.md) und [Erstellen von Datasets](data-factory-create-datasets.md) und setzt voraus, dass Sie mit den Konzepten des Data Factory- Anwendungsmodells (Aktivitäten, Pipelines, verknüpfte Dienste und Datasets) grundlegend vertraut sind.

## Planen von Aktivitäten

Im Abschnitt **scheduler** in der JSON der Aktivität können Sie einen sich wiederholenden Zeitplan für die Aktivität angeben. Beispielsweise können Sie folgendermaßen eine stündliche Ausführung der Aktivität planen:

	"scheduler": {
		"frequency": "Hour",
	    "interval": 1
	},  
    
![Beispiel für "scheduler"](./media/data-factory-scheduling-and-execution/scheduler-example.png)

Wie oben gezeigt, werden beim Angeben eines stündlichen Zeitplans Aktivitätsausführungen als eine Reihe rollierender Zeitfenster erzeugt. Rollierende Zeitfenster sind eine Reihe sich nicht überlappender zusammenhängender Zeitintervalle mit fester Größe.
 
Für die derzeit ausgeführte Aktivität kann in der JSON der Aktivität über die Systemvariablen **WindowStart** und **WindowEnd** auf das Zeitfenster zugegriffen werden. Sie können diese Variablen für verschiedene Zwecke in der JSON Ihrer Aktivität und Skripts verwenden, die der Aktivität zugeordnet sind, einschließlich Auswahl von Daten in den Ein- und Ausgabedatasets, die Zeitreihendaten darstellen.

Weitere Informationen zu den für "scheduler" verfügbaren Eigenschaften, einschließlich Planen eines bestimmten Zeitversatzes sowie Festlegen des Modus zum Abstimmen der Verarbeitung am Anfang oder Ende des Zeitfensters, finden Sie im Artikel [Erstellen von Pipelines](data-factory-create-pipelines.md).

## Datasets und Datenslices von Zeitreihen

Zeitreihendaten stellen eine fortlaufende Abfolge von Datenpunkten dar, die in der Regel aus aufeinander folgenden Messungen in einem Zeitintervall bestehen. Gängige Beispiele für Zeitreihendaten sind Sensordaten, Anwendungstelemetriedaten usw.

Mithilfe von Azure Data Factory können Sie Zeitreihendaten in Form von Aktivitätsausführungen im Batchmodus verarbeiten. Es gibt in der Regel einen regelmäßigen Rhythmus, in dem Eingabedaten erfasst und Ausgabedaten erstellt werden müssen. Dieser Rhythmus wird durch Festlegen des Abschnitts **availability** für das Dataset wie folgt bestimmt:

    "availability": {
      "frequency": "Hour",
      "interval": 1
    },

Jede Einheit von Daten, die durch eine Aktivitätsausführung genutzt und erstellt wird, heißt **Datenslice**. Das folgende Diagramm zeigt ein Beispiel einer Aktivität mit einem Dataset mit einer Eingabezeitreihe und einem Dataset mit einer Ausgabezeitreihe, wobei "frequency" für "availability" auf "hour" festgelegt ist.

![Scheduler für "availability"](./media/data-factory-scheduling-and-execution/availability-scheduler.png)

Die stündlichen Datenslices für das Ein- und Ausgabedataset werden im Diagramm gezeigt. Das Diagramm zeigt drei Eingabeslices, die für die Verarbeitung bereit sind, und die laufende Aktivitätsausführung "10-11AM", die den Ausgabeslice "10-11AM" erzeugt.

Auf das Zeitintervall des aktuell erzeugten Slices kann in der JSON des Datasets über die Variablen **SliceStart** und **SliceEnd** zugegriffen werden.

Weitere Informationen zu verschiedenen Eigenschaften, die im Abschnitt "availability" verfügbar sind, finden Sie im Artikel [Erstellen von Datasets](data-factory-create-datasets.md).

## Beispiel: Kopieraktivität zum Verschieben von Daten aus Azure SQL in ein Azure-Blob

Nun wollen wir uns nun diese Dinge in Aktion ansehen, indem wir das Beispiel der Kopieraktivität im Artikel [Erstellen von Pipelines](data-factory-create-pipelines.md) nehmen, bei den Daten stündlich aus einer Azure SQL-Tabelle in ein Azure-Blob kopiert werden.

**Eingabe: Azure SQL-Dataset**

	{
	    "name": "AzureSqlInput",
	    "properties": {
	        "published": false,
	        "type": "AzureSqlTable",
	        "linkedServiceName": "AzureSqlLinkedService",
	        "typeProperties": {
	            "tableName": "MyTable"
	        },
	        "availability": {
	            "frequency": "Hour",
	            "interval": 1
	        },
	        "external": true,
	        "policy": {}
	    }
	}


Beachten Sie, dass **frequency** auf **Hour** und **interval** auf **1** im Abschnitt **availability** festgelegt sind.

**Ausgabe: Azure-Blobdataset**
	
	{
	    "name": "AzureBlobOutput",
	    "properties": {
	        "published": false,
	        "type": "AzureBlob",
	        "linkedServiceName": "StorageLinkedService",
	        "typeProperties": {
	            "folderPath": "mypath/{Year}/{Month}/{Day}/{Hour}",
	            "format": {
	                "type": "TextFormat"
	            },
	            "partitionedBy": [
	                {
	                    "name": "Year",
	                    "value": {
	                        "type": "DateTime",
	                        "date": "SliceStart",
	                        "format": "yyyy"
	                    }
	                },
	                {
	                    "name": "Month",
	                    "value": {
	                        "type": "DateTime",
	                        "date": "SliceStart",
	                        "format": "%M"
	                    }
	                },
	                {
	                    "name": "Day",
	                    "value": {
	                        "type": "DateTime",
	                        "date": "SliceStart",
	                        "format": "%d"
	                    }
	                },
	                {
	                    "name": "Hour",
	                    "value": {
	                        "type": "DateTime",
	                        "date": "SliceStart",
	                        "format": "%H"
	                    }
	                }
	            ]
	        },
	        "availability": {
	            "frequency": "Hour",
	            "interval": 1
	        }
	    }
	}


Beachten Sie, dass **frequency** auf **Hour** und **interval** auf **1** im Abschnitt **availability** festgelegt sind.



**Aktivität: Kopieraktivität**

	{
	    "name": "SamplePipeline",
	    "properties": {
	        "description": "copy activity",
	        "activities": [
	            {
	                "type": "Copy",
	                "name": "AzureSQLtoBlob",
	                "description": "copy activity",	
	                "typeProperties": {
	                    "source": {
	                        "type": "SqlSource",
	                        "sqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
	                    },
	                    "sink": {
	                        "type": "BlobSink",
	                        "writeBatchSize": 100000,
	                        "writeBatchTimeout": "00:05:00"
	                    }
	                },
	                "inputs": [
	                    {
	                        "name": "AzureSQLInput"
	                    }
	                ],
	                "outputs": [
	                    {
	                        "name": "AzureBlobOutput"
	                    }
	                ],
	       			"scheduler": {
	          			"frequency": "Hour",
	          			"interval": 1
	        		}
	            }
	        ],
	        "start": "2015-01-01T08:00:00Z",
	        "end": "2015-01-01T11:00:00Z"
	    }
	}


Das Beispiel oben zeigt die Abschnitte mit dem Aktivitätszeitplan und der Verfügbarkeit von Datasets, die auf eine stündliche Frequenz festgelegt sind. Im Beispiel wird gezeigt, wie Sie die Variablen **WindowStart** und **WindowEnd** zur Auswahl der relevanten Daten für die angegebene Aktivitätsausführung nutzen und diese mit der entsprechenden dynamischen Einstellung für **folderPath** an ein Blob übermitteln können, sodass der Ordner für jede Stunde vorhanden ist.

In einer Azure-Beispieltabelle und einem Blob sehen wir uns nun das Ergebnis der Ausführung von drei der Slices zwischen 8:00 und 11:00 Uhr an.

Angenommen, Azure SQL enthält die folgenden Daten:

![Beispieleingabe](./media/data-factory-scheduling-and-execution/sample-input-data.png)

Bei Bereitstellen der obigen Pipeline wird das Azure-Blob wie folgt aufgefüllt:

1.	Datei "mypath/2015/1/1/8/Data.<Guid>.txt" mit Daten 

		10002345,334,2,2015-01-01 08:24:00.3130000
		10002345,347,15,2015-01-01 08:24:00.6570000
		10991568,2,7,2015-01-01 08:56:34.5300000

	**Hinweis:**<Guid> Wird durch eine tatsächliche GUID ersetzt. Beispieldateiname: Data.bcde1348-7620-4f93-bb89-0eed3455890b.txt
2.	Datei "mypath/2015/1/1/9/Data.<Guid>.txt" mit Daten:

		10002345,334,1,2015-01-01 09:13:00.3900000
		24379245,569,23,2015-01-01 09:25:00.3130000
		16777799,21,115,2015-01-01 09:47:34.3130000
3.	Datei "mypath/2015/1/1/10/Data.<Guid>.txt" ohne Daten.


## Datenslices, aktiver Zeitraum für die Pipeline und gleichzeitige Sliceausführung

Im Artikel [Erstellen von Pipelines](data-factory-create-pipelines.md) wurde das Konzept des aktiven Zeitraums für eine Pipeline eingeführt, der durch Festlegen der Eigenschaften **start** und **end** der Pipeline angegeben wird.
 
Sie können das Startdatum des aktiven Zeitraums der Pipeline in der Vergangenheit festlegen. Data Factory berechnet anschließend automatisch alle (nachträglich aufgefüllten Datenslices) in der Vergangenheit und beginnt mit ihrer Verarbeitung.

Nachträglich aufgefüllte Datenslices können für eine parallele Ausführung konfiguriert werden. Legen Sie hierzu die "concurrency"-Eigenschaft im Abschnitt **policy** der JSON der Aktivität fest, was im Artikel [Erstellen von Pipelines](data-factory-create-pipelines.md) erklärt wird.

## Wiederholen fehlerhafter Datenslices und automatische Nachverfolgung von Datenabhängigkeiten

Sie können die Ausführung von Slices umfassend visuell überwachen. Weitere Informationen finden Sie im Artikel [Überwachen und Verwalten von Pipelines](data-factory-monitor-manage-pipelines.md).

Betrachten Sie das folgende Beispiel mit zwei Aktivitäten. Aktivität1 erstellt ein Zeitreihen-Dataset mit Slices als Ausgabe, die als Eingabe von Aktivität2 verwendet wird, um die endgültige Ausgabe des Zeitreihen-Datasets zu erstellen.

![Fehlerhafter Slice](./media/data-factory-scheduling-and-execution/failed-slice.png)

<br/>

Das obige Diagramm zeigt, dass bei den letzten drei Slices ein Fehler beim Erstellen des Slices "9-10 AM" für **Dataset2** aufgetreten ist. Data Factory verfolgt Abhängigkeiten für das Zeitreihen-Dataset automatisch nach und unterlässt als Ergebnis das Auslösen der Aktivitätsausführung für den nachgelagerten Slice "9-10 AM".


Mit den Data Factory-Überwachungs- und Verwaltungstools können Sie die Diagnoseprotokolle detailliert nach dem fehlerhaften Slice durchsuchen, um die Ursache des Problems zu finden und zu beseitigen. Nachdem Sie das Problem behoben haben, können Sie auch ganz einfach die Aktivitätsausführung auslösen, um den fehlerhaften Slice zu erstellen. Weitere Informationen zum Auslösen von Wiederholungen und Grundlegendes zu Statusübergängen bei Datenslices finden Sie im Artikel [Überwachung und Verwaltung](data-factory-monitor-manage-pipelines.md).

Nachdem Sie die Wiederholung ausgelöst haben und der Slice "9-10 AM" für "Dataset2" bereit ist, löst Data Factory die Ausführung für den von "9-10 AM" abhängigen Slice für das endgültige Dataset aus (siehe die nachstehende Abbildung).

![Wiederholen eines fehlerhaften Slices](./media/data-factory-scheduling-and-execution/rerun-failed-slice.png)

Ausführlichere Informationen zum Abgeben von Abhängigkeiten und deren Nachverfolgung bei einer komplexe Kette von Aktivitäten und Datasets finden Sie in den folgenden Abschnitten.

## Modellieren von Datasets mit unterschiedlichen Frequenzen

In den obigen Beispielen waren die Frequenzen für Eingabe- und Ausgabedatasets und das Aktivitätszeitfenster identisch. Einige Szenarios erfordern die Fähigkeit, eine Ausgabe mit einer Frequenz zu erzeugen, die sich von den Frequenzen einer oder mehrerer Eingaben unterscheidet. Data Factory unterstützt diese Szenarios.

### Beispiel 1: Erzeugen eines täglichen Ausgabeberichts für Eingabedaten, die stündlich verfügbar sind

Nehmen wir ein Szenario mit Eingabemessdaten von Sensoren, die in einem Azure-Blob stündlich verfügbar sind. Mithilfe der [Hive-Aktivität](data-factory-hive-activity.md) von Data Factory wollen wir einen täglichen Zusammenfassungsbericht mit statistischen Angaben zu Mittel-, Höchst-, Mindestwert usw. generieren.

Sie können dies wie folgt mit Data Factory realisieren:

**Eingabe: Azure-Blobdataset:**

Die stündlichen Eingabedateien werden im Ordner für den jeweiligen Tag abgelegt. Für die Eingabe ist "availability" auf "Stündlich" festgelegt (frequency: Hour, interval: 1).

	{
	  "name": "AzureBlobInput",
	  "properties": {
	    "type": "AzureBlob",
	    "linkedServiceName": "StorageLinkedService",
	    "typeProperties": {
	      "folderPath": "mycontainer/myfolder/{Year}/{Month}/{Day}/",
	      "partitionedBy": [
	        { "name": "Year", "value": {"type": "DateTime","date": "SliceStart","format": "yyyy"}},
	        { "name": "Month","value": {"type": "DateTime","date": "SliceStart","format": "%M"}},
	        { "name": "Day","value": {"type": "DateTime","date": "SliceStart","format": "%d"}}
	      ],
	      "format": {
	        "type": "TextFormat"
	      }
	    },
		"external": true,
	    "availability": {
	      "frequency": "Hour",
	      "interval": 1
	    }
	  }
	}

**Ausgabe: Azure-Blobdataset**

Eine Ausgabedatei wird täglich im Ordner des jeweiligen Tags abgelegt. Für die Ausgabe ist "availability" auf "Täglich" festgelegt (frequency: Day, interval: 1).


	{
	  "name": "AzureBlobOutput",
	  "properties": {
	    "type": "AzureBlob",
	    "linkedServiceName": "StorageLinkedService",
	    "typeProperties": {
	      "folderPath": "mycontainer/myfolder/{Year}/{Month}/{Day}/",
	      "partitionedBy": [
	        { "name": "Year", "value": {"type": "DateTime","date": "SliceStart","format": "yyyy"}},
	        { "name": "Month","value": {"type": "DateTime","date": "SliceStart","format": "%M"}},
	        { "name": "Day","value": {"type": "DateTime","date": "SliceStart","format": "%d"}}
	      ],
	      "format": {
	        "type": "TextFormat"
	      }
	    },
	    "availability": {
	      "frequency": "Day",
	      "interval": 1
	    }
	  }
	}

**Aktivität: Hive-Aktivität in einer Pipeline**

Das Hive-Skript empfängt durch Verwenden der Variablen **WindowStart** die entsprechenden Datum/Uhrzeit-Informationen als Parameter (siehe unten). Das Hive-Skript verwendet diese Variable zum Laden der Daten aus dem richtigen Ordner für den jeweiligen Tag und zum Ausführen der Aggregation, um die Ausgabe zu generieren.

		{  
		    "name":"SamplePipeline",
		    "properties":{  
		    "start":"2015-01-01T08:00:00",
		    "end":"2015-01-01T11:00:00",
		    "description":"hive activity",
		    "activities": [
		        {
		            "name": "SampleHiveActivity",
		            "inputs": [
		                {
		                    "name": "AzureBlobInput"
		                }
		            ],
		            "outputs": [
		                {
		                    "name": "AzureBlobOutput"
		                }
		            ],
		            "linkedServiceName": "HDInsightLinkedService",
		            "type": "HDInsightHive",
		            "typeProperties": {
		                "scriptPath": "adftutorial\\hivequery.hql",
		                "scriptLinkedService": "StorageLinkedService",
		                "defines": {
		                    "Year": "$$Text.Format('{0:yyyy}',WindowsStart)",
		                    "Month": "$$Text.Format('{0:%M}',WindowStart)",
		                    "Day": "$$Text.Format('{0:%d}',WindowStart)"
		                }
		            },
		            "scheduler": {
		                "frequency": "Day",
		                "interval": 1
		            },			
		            "policy": {
		                "concurrency": 1,
		                "executionPriorityOrder": "OldestFirst",
		                "retry": 2,
		                "timeout": "01:00:00"
		            }
	             }
		     ]
		   }
		}

Hier sehen Sie die Datenabhängigkeit.

![Datenabhängigkeit](./media/data-factory-scheduling-and-execution/data-dependency.png)

Der Ausgabeslice für jeden Tag hängt von 24 stündlichen Slices aus dem Eingabedataset ab. Data Factory berechnet diese Abhängigkeiten automatisch, indem die Eingabedatenslices ermittelt werden, die im selben Zeitraum wie der zu erzeugende Ausgabeslice liegen. Wenn beliebige der 24 Eingabeslices nicht verfügbar sind (z. B. aufgrund einer Verarbeitung in einer vorgelagerten Aktivität, die diesen Slice erzeugt), wartet Data Factory ab, bis der Eingabeslice bereit ist, ehe die tägliche Aktivitätsausführung ausgelöst wird.


### Beispiel 2: Angeben von Abhängigkeiten mit Ausdrücken und Data Factory-Funktionen

Lassen Sie uns ein weiteres Szenario betrachten. Angenommen, Sie haben eine Hive-Aktivität, die zwei Eingabedatasets verarbeitet. Eines davon verfügt täglich über neue Daten, während das andere wöchentlich neue Daten empfängt. Angenommen, Sie möchten einen Join-Vorgang auf zwei Eingaben anwenden und eine tägliche Ausgabe erzeugen.
 
Der bislang vorgestellte einfache Ansatz, bei dem Data Factory die richtigen zu verarbeitenden Eingabeslices automatisch bestimmt, indem auf die Ausgabedatenslices abgestimmte Eingabedatenslices verwendet werden, funktioniert nicht mehr.

Sie benötigen eine Möglichkeit, um für jede Aktivitätsausführung anzugeben, dass die Data Factory den Datenslice der letzten Woche als wöchentlichen Eingabedataset verwenden soll. Dies können Sie, wie unten dargestellt, mithilfe von Azure Data Factory-Funktionen erreichen.

**Eingabe1: Azure-Blob**

Die erste Eingabe ist ein **täglich** aktualisierter Azure-Blob.
	
	{
	  "name": "AzureBlobInputDaily",
	  "properties": {
	    "type": "AzureBlob",
	    "linkedServiceName": "StorageLinkedService",
	    "typeProperties": {
	      "folderPath": "mycontainer/myfolder/{Year}/{Month}/{Day}/",
	      "partitionedBy": [
	        { "name": "Year", "value": {"type": "DateTime","date": "SliceStart","format": "yyyy"}},
	        { "name": "Month","value": {"type": "DateTime","date": "SliceStart","format": "%M"}},
	        { "name": "Day","value": {"type": "DateTime","date": "SliceStart","format": "%d"}}
	      ],
	      "format": {
	        "type": "TextFormat"
	      }
	    },
		"external": true,
	    "availability": {
	      "frequency": "Day",
	      "interval": 1
	    }
	  }
	}

**Eingabe2: Azure-Blob**

Eingabe2 ist ein **wöchentlich** aktualisierter Azure-Blob.

	{
	  "name": "AzureBlobInputWeekly",
	  "properties": {
	    "type": "AzureBlob",
	    "linkedServiceName": "StorageLinkedService",
	    "typeProperties": {
	      "folderPath": "mycontainer/myfolder/{Year}/{Month}/{Day}/",
	      "partitionedBy": [
	        { "name": "Year", "value": {"type": "DateTime","date": "SliceStart","format": "yyyy"}},
	        { "name": "Month","value": {"type": "DateTime","date": "SliceStart","format": "%M"}},
	        { "name": "Day","value": {"type": "DateTime","date": "SliceStart","format": "%d"}}
	      ],
	      "format": {
	        "type": "TextFormat"
	      }
	    },
		"external": true,
	    "availability": {
	      "frequency": "Day",
	      "interval": 7
	    }
	  }
	}

**Ausgabe: Azure-Blob**

Eine Ausgabedatei wird täglich im Ordner des jeweiligen Tags abgelegt. Für die Ausgabe ist "availability" auf "Täglich" festgelegt (frequency: Day, interval: 1).
	
	{
	  "name": "AzureBlobOutputDaily",
	  "properties": {
	    "type": "AzureBlob",
	    "linkedServiceName": "StorageLinkedService",
	    "typeProperties": {
	      "folderPath": "mycontainer/myfolder/{Year}/{Month}/{Day}/",
	      "partitionedBy": [
	        { "name": "Year", "value": {"type": "DateTime","date": "SliceStart","format": "yyyy"}},
	        { "name": "Month","value": {"type": "DateTime","date": "SliceStart","format": "%M"}},
	        { "name": "Day","value": {"type": "DateTime","date": "SliceStart","format": "%d"}}
	      ],
	      "format": {
	        "type": "TextFormat"
	      }
	    },
	    "availability": {
	      "frequency": "Day",
	      "interval": 1
	    }
	  }
	}

**Aktivität: Hive-Aktivität in einer Pipeline**

Die Hive-Aktivität verwendet zwei Eingaben und erzeugt täglich einen Ausgabeslice. Sie können den täglichen Ausgabeslice wie folgt so angeben, dass dieser für die wöchentliche Eingabe vom Eingabeslice der letzten Woche abhängt.
	
	{  
	    "name":"SamplePipeline",
	    "properties":{  
	    "start":"2015-01-01T08:00:00",
	    "end":"2015-01-01T11:00:00",
	    "description":"hive activity",
	    "activities": [
	      {
	        "name": "SampleHiveActivity",
	        "inputs": [
	          {
	            "name": "AzureBlobInputDaily"
	          },
	          {
	            "name": "AzureBlobInputWeekly",
	            "startTime": "Date.AddDays(SliceStart,  -7 - Date.DayOfWeek(SliceStart))",
	            "endTime": "Date.AddDays(SliceEnd,  -Date.DayOfWeek(SliceEnd))"  
	          }
	        ],
	        "outputs": [
	          {
	            "name": "AzureBlobOutputDaily"
	          }
	        ],
	        "linkedServiceName": "HDInsightLinkedService",
	        "type": "HDInsightHive",
	        "typeProperties": {
	          "scriptPath": "adftutorial\\hivequery.hql",
	          "scriptLinkedService": "StorageLinkedService",
	          "defines": {
	            "Year": "$$Text.Format('{0:yyyy}',WindowsStart)",
	            "Month": "$$Text.Format('{0:%M}',WindowStart)",
	            "Day": "$$Text.Format('{0:%d}',WindowStart)"
	          }
	        },
	        "scheduler": {
	          "frequency": "Day",
	          "interval": 1
	        },			
	        "policy": {
	          "concurrency": 1,
	          "executionPriorityOrder": "OldestFirst",
	          "retry": 2,  
	          "timeout": "01:00:00"
	        }
		   } 
	     ]
	   }
	}


## Verketten von Aktivitäten
Sie können zwei Aktivitäten verketten, indem Sie das Ausgabedataset einer Aktivität als Eingabedataset der anderen Aktivität verwenden. Die Aktivitäten können sich in derselben Pipeline oder in verschiedenen Pipelines befinden. Die zweite Aktivität wird nur ausgeführt, wenn die erste erfolgreich abgeschlossen wurde. Diese Verkettung erfolgt auf Ebenen von Zeitslices (eine einzelne Einheit innerhalb eines Datasets).

## Data Factory-Systemvariablen

Variablenname | Beschreibung | Objektbereich | JSON-Bereich und Anwendungsfälle
------------- | ----------- | ------------ | ------------------------
WindowStart | Anfang des Zeitfensters der aktuellen Aktivitätsausführung | Aktivität | <ol><li>Geben Sie Abfragen zur Datenauswahl an. Informationen finden Sie in den Artikeln zu Connectors, auf die im Artikel [Datenverschiebungsaktivitäten](data-factory-data-movement-activities.md) verwiesen wird.</li><li>Übergeben Sie Parameter an das Hive-Skript (siehe obiges Beispiel).</li>
WindowEnd | Ende des Zeitfensters der aktuellen Aktivitätsausführung | Aktivität | Wie oben
SliceStart | Anfang des Zeitfensters für den zu erstellenden Datenslice | Aktivität<br/>Dataset | <ol><li>Geben Sie bei der Arbeit mit [Azure Blob](data-factory-azure-blob-connector.md) und [Dateisystem-Datasets](data-factory-onprem-file-system-connector.md) dynamische Ordnerpfade und Dateinamen an.</li><li>Geben Sie Eingabeabhängigkeiten mit Data Factory-Funktionen in der Auflistung der Aktivitätseingaben an.</li></ol>
SliceEnd | Ende des Zeitfensters für den zu erstellenden Datenslice | Aktivität<br/>Dataset | Wie oben. 

> [AZURE.NOTE]Derzeit erfordert Data Factory, dass der in der Aktivität angegebene Zeitplan exakt mit dem Zeitplan übereinstimmt, der in "availability" für das Ausgabedataset angegeben ist. Das heißt, dass "WindowStart" und "WindowEnd" sowie "SliceStart" und "SliceEnd" immer dem gleichen Zeitraum und einem einzelnen Ausgabeslice zugeordnet sind.
 
## Referenz der Data Factory-Funktionen

Sie können Funktionen in Data Factory zusammen mit den zuvor genannten Systemvariablen für folgende Zwecke verwenden:

1.	Angeben von Abfragen zur Datenauswahl (siehe die Artikel zu Connectors, auf die im Artikel [Datenverschiebungsaktivitäten](data-factory-data-movement-activities.md) verwiesen wird).

	Die Syntax zum Aufrufen einer Data Factory-Funktion ist **$$<function>** für Abfragen zur Datenauswahl und andere Eigenschaften in der Aktivität und den Datasets.  
2. Angeben von Eingabeabhängigkeiten mit Data Factory-Funktionen in der Auflistung der Aktivitätseingaben an (siehe das Beispiel oben).

	$$ ist nicht erforderlich, um Eingabeabhängigkeitsausdrücke anzugeben.

Im folgenden Beispiel wird die **sqlReaderQuery**-Eigenschaft in einer JSON-Datei einem Wert zugewiesen, der von der **Text.Format**-Funktion zurückgegeben wird. Dieses Beispiel verwendet ebenfalls die Systemvariable **WindowStart**, die die Startzeit des Zeitfensters für die Aktivitätsausführung darstellt.
	
	{
	    "Type": "SqlSource",
	    "sqlReaderQuery": "$$Text.Format('SELECT * FROM MyTable WHERE StartTime = \\'{0:yyyyMMdd-HH}\\'', WindowStart)"
	}

### Funktionen

In den folgenden Tabellen werden alle Funktionen in Azure Data Factory aufgelistet:

Kategorie | Funktion | Parameter | Beschreibung
-------- | -------- | ---------- | ----------- 
Time | AddHours(X,Y) | X: DateTime <p>Y: int</p> | Fügt Y Stunden der angegebenen Uhrzeit X hinzu.<p>Beispiel: 05.09.2013 12:00:00 Uhr + 2 Stunden = 05.09.2013 14:00:00 Uhr</p>
Time | AddMinutes(X,Y) | X: DateTime <p>Y: int</p> | Fügt Y Minuten zu X hinzu.<p>Beispiel: 15.09.2013 12:00:00 Uhr + 15 Minuten = 15.09.2013 12:15:00 Uhr</p>
Time | StartOfHour(X) | X: Datetime | Ruft die Startzeit der Stunde ab, die von der Stundenkomponente von X dargestellt wird. <p>Beispiel: StartOfHour von 15.09.2013 05:10:23 Uhr ist 15.09.2013 05:00:00 Uhr</p>
Date | AddDays(X,Y) | X: DateTime<p>Y: int</p> | Fügt Y Tage zu X hinzu. <p>Beispiel: 15.09.2013 12:00:00 Uhr + 2 Tage = 17.09.2013 12:00:00 Uhr</p>
Date | AddMonths(X,Y) | X: DateTime<p>Y: int</p> | Fügt Y Monate zu X hinzu.<p>Beispiel: 15.09.2013 12:00:00 Uhr + 1 Monat = 15.10.2013 12:00:00 Uhr</p> 
Date | AddQuarters(X,Y) | X: DateTime <p>Y: int</p> | Fügt Y * 3 Monate zu X hinzu<p>Beispiel: 15.09.2013 12:00:00 Uhr + 1 Quartal = 15.12.2013 12:00:00 Uhr</p>
Date | AddWeeks(X,Y) | X: DateTime<p>Y: int</p> | Fügt Y * 7 Tage zu X hinzu. <p>Beispiel: 15.09.2013 12:00:00 Uhr + 1 Woche = 22.09.2013 12:00:00 Uhr</p>
Date | AddYears(X,Y) | X: DateTime<p>Y: int</p> | Fügt Y Jahre zu X hinzu.<p>Beispiel: 15.09.2013 12:00:00 Uhr + 1 Jahr = 15.09.2014 12:00:00 Uhr</p>
Date | Day(X) | X: DateTime | Ruft die Komponente „Tag“ von X ab.<p>Beispiel: Tag von 15.09.2013 12:00:00 Uhr ist der 15.</p>
Date | DayOfWeek(X) | X: DateTime | Ruft den Tag der Komponente „Wochentag“ von X ab.<p>Beispiel: DayOfWeek von 15.09.2013 12:00:00 Uhr ist Sonntag.</p>
Date | DayOfYear(X) | X: DateTime | Ruft den Tag des Jahres ab, der von der Komponente „Jahr“ von X dargestellt wird.<p>Beispiele:<br/>01.12.2015: Tag 335 von 2015<br/>31.12.2015: Tag 365 von 2015<br/>31.12.2016: Tag 366 von 2016 (Schaltjahr)</p>
Date | DaysInMonth(X) | X: DateTime | Ruft die Tage des Monats ab, die von der Komponente „Monat“ des Parameters X dargestellt werden.<p>Beispiel: DaysInMonth von 15.09.2013 sind 30, da der Monat September 30 Tage hat.</p>
Date | EndOfDay(X) | X: DateTime | Ruft die Datum/Uhrzeit-Angabe ab, die das Ende des Tages (Komponente „Tag“) von X darstellt.<p>Beispiel: EndOfDay 15.09.2013 17:10:23 Uhr ist 15.09.2013 23:59:59 Uhr.</p>
Date | EndOfMonth(X) | X: DateTime | Ruft das Ende des Monats ab, das von der Komponente „Monat“ des Parameters X dargestellt wird.<p>Beispiel: EndOfMonth 15.09.2013 17:10:23 Uhr ist 30.09.2013 23:59:59 Uhr (Datum/Uhrzeit-Angabe, die das Ende des Monats September darstellt)</p>
Date | StartOfDay(X) | X: DateTime | Ruft den Beginn des Tages ab, der von der Komponente „Tag“ des Parameters X dargestellt wird.<p>Beispiel: StartOfDay 15.09.2013 17:10:23 Uhr ist 15.09.2013 12:00:00 Uhr.</p>
DateTime | From(X) | X: String | Analysieren der Zeichenfolge X in einen Datum/Uhrzeit-Wert.
DateTime | Ticks(X) | X: DateTime | Ruft die Zeiteinheitseigenschaft des Parameters X ab. Eine Zeiteinheit entspricht 100 Nanosekunden. Der Wert dieser Eigenschaft stellt die Anzahl der Zeiteinheiten dar, die seit Mitternacht am 1. Januar 0001 verstrichen sind. 
Text | Format(X) | X: Stringvariable | Formatiert den Text.

#### Beispiel für "Text.Format"

	"defines": { 
	    "Year" : "$$Text.Format('{0:yyyy}',WindowStart)",
	    "Month" : "$$Text.Format('{0:MM}',WindowStart)",
	    "Day" : "$$Text.Format('{0:dd}',WindowStart)",
	    "Hour" : "$$Text.Format('{0:hh}',WindowStart)"
	}

> [AZURE.NOTE]Bei Verwenden einer Funktion in einer anderen Funktion müssen Sie für die innere Funktion nicht das Präfix **$$** verwenden. Beispiel: $$Text.Format('PartitionKey eq \\'my\_pkey\_filter\_value\\' und RowKey ge \\'{0:yyyy-MM-dd HH:mm:ss}\\'', Time.AddHours(SliceStart, -6)). Beachten Sie bei diesem Beispiel, dass das Präfix **$$** für die **Time.AddHours**-Funktion nicht verwendet wird.
  

## Datenabhängigkeiten – Detaillierte Informationen

Zum Generieren eines Datasetslices mittels einer Aktivitätsausführung verwendet Data Factory das folgende **Abhängigkeitsmodell**, um die Beziehungen zwischen den Datasets, die von einer Aktivität genutzt werden, und den von einer Aktivität erzeugten Datasets zu bestimmen.

Der Zeitraum der Eingabedatasets, der zum Generieren des Slices des Ausgabedatasets erforderlich ist, wird **Abhängigkeitszeitraum** genannt.

Bei einer Aktivitätsausführung wird ein Datasetslice erst erzeugt, sobald die Datenslices in Eingabedatasets innerhalb des Abhängigkeitszeitraums verfügbar sind. Dies bedeutet, dass alle Eingabeslices, die den Abhängigkeitszeitraum bilden, den Status **Bereit** haben müssen, damit der Slice des Ausgabedatasets durch eine Aktivitätsausführung erzeugt wird.

Zum Generieren des Datasetslices ["start", "end"] ist eine Funktion erforderlich, die den Datasetslice seinem Abhängigkeitszeitraum zuordnet. Diese Funktion ist im Wesentlichen eine Formel, die Anfang und Ende des Zeitraums des Datenslices in Anfang und Ende des Abhängigkeitszeitraums umwandelt. Formeller ausgedrückt:
	
	DatasetSlice = [start, end]
	DependecyPeriod = [f(start, end), g(start, end)]

"f" und "g" sind Zuordnungsfunktionen, die Anfang und Ende des Abhängigkeitszeitraums für jede Aktivitätseingabe berechnen.

Wie in den zuvor gezeigten Beispielen entspricht der Abhängigkeitszeitraum meist dem Zeitraum des zu erstellenden Datenslices. In diesen Fällen berechnet Daten Factory automatisch die Eingabeslices, die in den Abhängigkeitszeitraum fallen.

Beispiel: Beim obigen Aggregationsbeispiel, bei dem die Ausgabe täglich erzeugt wird und Eingabedaten stündlich verfügbar sind, ist der Zeitraum des Datenslices 24 Stunden. Data Factory sucht die relevanten stündlichen Eingabeslices für diesen Zeitraum und macht den Ausgabeslice vom Eingabeslice abhängig.

Sie können auch Ihre eigene Zuordnung für den Abhängigkeitszeitraum angeben (wie im obigen Beispiel gezeigt), bei dem eine der Eingabe wöchentlich erfolgte und der Ausgabeslice täglich erzeugt wurde.
   
## Datenabhängigkeit und -überprüfung

Für ein Dataset kann optional eine Überprüfungsrichtlinie definiert sein, die angibt, wie die von einer Sliceausführung generierten Daten überprüft werden können, ehe sie zur Nutzung bereit sind. Weitere Informationen finden Sie im Artikel [Erstellen von Datasets](data-factory-create-datasets.md).

In solchen Fällen wird nach Beendigung der Sliceausfürung der Status des Ausgabeslices in **Warten** mit dem Unterstatus **Überprüfung** geändert. Nach der Überprüfung der Slices ändert sich der Slicestatus in **Bereit**.
   
Wenn ein Datenslice erstellt wurde, jedoch die Überprüfung nicht bestanden hat, werden Aktivitätsausführungen für nachgelagerte Slices, die vom Slice abhängen, der die Überprüfung nicht bestanden hat, nicht verarbeitet.

Die verschiedenen Status von Datenslices in Data Factory werden im Artikel [Überwachen und Verwalten von Pipelines](data-factory-monitor-manage-pipelines.md) behandelt.

## Externe Daten

Ein Dataset kann als extern gekennzeichnet werden (siehe die nachstehende JSON), um anzugeben, dass es nicht mit Azure Data Factory erstellt wurde. In einem solchen Fall kann die Datasetrichtlinie eine Reihe zusätzliche Parameter zum Beschreiben der Überprüfung und eine Wiederholungsrichtlinie für das Dataset aufweisen. Unter [Erstellen von Pipelines](data-factory-create-pipelines.md) finden Sie eine Beschreibung aller Eigenschaften.

Ähnlich wie Datasets, die von Daten Factory erstellt werden, müssen die Datenslices für externe Daten bereit sein, ehe abhängige Slices verarbeitet werden können.

	{
		"name": "AzureSqlInput",
		"properties": 
		{
			"type": "AzureSqlTable",
			"linkedServiceName": "AzureSqlLinkedService",
			"typeProperties": 
			{
				"tableName": "MyTable"	
			},
			"availability": 
			{
				"frequency": "Hour",
				"interval": 1     
			},
			"external": true,
			"policy": 
			{
				"externalData": 
				{
					"retryInterval": "00:01:00",
					"retryTimeout": "00:10:00",
					"maximumRetry": 3
				}
			}  
		} 
	} 






  









 
 












      

  

<!---HONumber=Nov15_HO1-->