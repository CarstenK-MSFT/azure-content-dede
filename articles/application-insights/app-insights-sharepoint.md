<properties 
	pageTitle="Überwachen einer SharePoint-Website mit Application Insights" 
	description="Beginnen Sie mit der Überwachung einer neuen Anwendung mit einem neuen Instrumentationsschlüssel." 
	services="application-insights" 
    documentationCenter=""
	authors="alancameronwills" 
	manager="douge"/>

<tags 
	ms.service="application-insights" 
	ms.workload="tbd" 
	ms.tgt_pltfrm="ibiza" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="11/06/2015" 
	ms.author="awills"/>

# Überwachen einer SharePoint-Website mit Application Insights


[AZURE.INCLUDE [app-insights-selector-get-started](../../includes/app-insights-selector-get-started.md)]

Visual Studio Application Insights überwacht die Verfügbarkeit, Leistung und Nutzung Ihrer Apps. Hier erfahren Sie, wie Sie dieses Tool für eine SharePoint-Website einrichten.


## Erstellen einer Application Insights-Ressource


Erstellen Sie im [Azure-Portal](http://portal.azure.com) eine neue Application Insights-Ressource. Wählen Sie als Anwendungstyp "ASP.NET" aus.

![Klicken Sie auf "Eigenschaften", wählen Sie den Schlüssel aus, und drücken Sie STRG+C](./media/app-insights-sharepoint/01-new.png)


Auf dem nun geöffneten Blatt werden die Leistungs- und Nutzungsdaten über Ihre App angezeigt. Um bei der nächsten Anmeldung bei Azure dorthin zu gelangen, sollten Sie eine Kachel auf dem Startbildschirm anlegen. Klicken Sie alternativ auf "Durchsuchen", um das Blatt zu finden.
    


## Hinzufügen unseres Skripts zu Ihren Webseiten

Rufen Sie im Schnellstart das Skript für Webseiten ab:

![](./media/app-insights-sharepoint/02-monitor-web-page.png)

Fügen Sie das Skript direkt vor dem &lt;/head&gt;-Tag jeder Seite ein, die Sie nachverfolgen möchten. Wenn Ihre Website über eine Masterseite verfügt, können Sie das Skript dort ablegen. Beispielsweise würden Sie es in einem ASP.NET MVC-Projekt unter "View\\Shared\_Layout.cshtml" ablegen.

Das Skript enthält den Instrumentationsschlüssel, der die Telemetriedaten an Ihre Application Insights-Ressource leitet.

### Hinzufügen des Codes zu den Seiten Ihrer Website

#### Auf der Masterseite

Wenn Sie die Masterseite der Website bearbeiten können, kann dadurch die Überwachung jeder Seite der Website bereitgestellt werden.

Checken Sie die Masterseite aus, und bearbeiten Sie sie mit SharePoint Designer oder einem anderen Editor.

![](./media/app-insights-sharepoint/03-master.png)


Fügen Sie den Code direkt vor dem Tag </head> ein.


![](./media/app-insights-sharepoint/04-code.png)

#### Auf einzelnen Seiten

Um eine begrenzte Anzahl von Seiten zu überwachen, fügen Sie das Skript separat auf jeder Seite hinzu.

Fügen Sie ein Webpart ein, und betten Sie den Codeausschnitt darin ein.


![](./media/app-insights-sharepoint/05-page.png)


## Anzeigen von Daten über Ihre App

Stellen Sie Ihre App erneut bereit.

Kehren Sie zum Blatt Ihrer Anwendung im [Azure-Portal](http://portal.azure.com) zurück.

Die ersten Ereignisse werden in der Suche angezeigt.

![](./media/app-insights-sharepoint/09-search.png)

Klicken Sie nach einigen Sekunden auf "Aktualisieren", wenn Sie mehr Daten erwarten.

Klicken Sie in der Übersicht auf **Nutzungsanalyse**, um Diagramme, Sitzungen und Seitenansichten anzuzeigen:

![](./media/app-insights-sharepoint/06-usage.png)

Klicken Sie auf ein beliebiges Diagramm, um weitere Details anzuzeigen, z. B. Seitenansichten:

![](./media/app-insights-sharepoint/07-pages.png)

Oder Benutzer:


![](./media/app-insights-sharepoint/08-users.png)



## Nächste Schritte

* [Webtests](app-insights-monitor-web-app-availability.md) zur Überwachung der Verfügbarkeit Ihrer Website.

* [Application Insights](app-insights-overview.md) für andere App-Typen.



<!--Link references-->

<!---HONumber=Nov15_HO3-->