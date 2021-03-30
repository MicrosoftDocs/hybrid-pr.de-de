---
title: Muster zur Ermittlung der Kundenfrequenz unter Verwendung von Azure und Azure Stack Hub
description: Hier erfahren Sie, wie Sie mithilfe von Azure und Azure Stack Hub eine KI-basierte Lösung zur Ermittlung der Kundenfrequenz implementieren, um den Kundenverkehr in einem Einzelhandelsgeschäft zu analysieren.
author: BryanLa
ms.topic: article
ms.date: 10/31/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 10/31/2019
ms.openlocfilehash: 866557ec3af2337e9f034da84cf417675508563b
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895328"
---
# <a name="footfall-detection-pattern"></a>Muster zur Ermittlung der Kundenfrequenz

Dieses Muster bietet eine Übersicht über eine Lösung zur Implementierung eines KI-basierten Musters zur Ermittlung der Kundenfrequenz, um den Kundenverkehr in Einzelhandelsgeschäften zu analysieren. Die Lösung generiert mithilfe von Azure, Azure Stack Hub und dem Custom Vision AI Dev Kit Erkenntnisse anhand von Aktionen aus der Praxis.

## <a name="context-and-problem"></a>Kontext und Problem

Die Contoso Stores möchten gerne Erkenntnisse darüber gewinnen, wie Kunden ihre aktuellen Produkte in der derzeitigen Aufteilung und Gestaltung der Ladengeschäfte wahrnehmen. Das Unternehmen kann nicht in jedem Bereich Mitarbeiter platzieren, und es ist nicht effizient, sämtliche Kameraaufnahmen eines gesamten Geschäfts durch ein Analystenteam überprüfen zu lassen. Darüber hinaus verfügt keines der Ladengeschäfte über genügend Bandbreite, um die Videodaten aller Kameras zu Analysezwecken in die Cloud zu streamen.

Contoso möchte eine unaufdringliche Möglichkeit finden, demografische Daten, Informationen zur Kundentreue sowie die Reaktionen der Kunden auf Werbeflächen und Produkte im Geschäft zu ermitteln, ohne den Datenschutz für die Kunden zu gefährden.

## <a name="solution"></a>Lösung

Das Analysemuster für Einzelhandelsgeschäfte nutzt einen gestaffelten Ansatz im Edge-Bereich zum Ziehen von Rückschlüssen. Mit dem Custom Vision AI Dev Kit werden nur Bilder mit Gesichtern von Menschen zur Analyse an eine private Azure Stack Hub-Instanz gesendet, die Azure Cognitive Services ausführt. Anonymisierte Daten werden aggregiert und an Azure gesendet. Dort werden alle Daten aus allen Geschäften zusammengefasst und in Power BI visualisiert. Durch eine Kombination aus Edge und öffentlicher Cloud kann Contoso von moderner KI-Technologie profitieren, und gleichzeitig sind die Einhaltung der Unternehmensrichtlinien und der Datenschutz für die Kunden gewährleistet.

[![Lösung mit Muster zur Ermittlung der Kundenfrequenz](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)

So funktioniert die Lösung:

1. Das Custom Vision AI Dev Kit ruft eine Konfiguration aus IoT Hub an, die die IoT Edge-Runtime und ein ML-Modell installiert.
2. Wenn das Modell eine Person sieht, nimmt es ein Bild auf und lädt es in Blob Storage für Azure Stack Hub hoch.
3. Der Blobdienst löst eine Azure-Funktion in Azure Stack Hub aus.
4. Die Azure-Funktion ruft einen Container mit der Gesichtserkennungs-API auf, um demografische und emotionsbezogene Daten aus dem Bild abzurufen.
5. Die Daten werden anonymisiert und an einen Azure Event Hubs-Cluster gesendet.
6. Der Event Hubs-Cluster pusht die Daten in Stream Analytics.
7. Stream Analytics aggregiert die Daten und pusht sie in Power BI.

## <a name="components"></a>Komponenten

Diese Lösung verwendet die folgenden Komponenten:

| Ebene | Komponente | BESCHREIBUNG |
|----------|-----------|-------------|
| Hardware im Ladengeschäft | [Custom Vision AI Dev Kit](https://azure.github.io/Vision-AI-DevKit-Pages/) | Diese Komponente führt im Geschäft eine Filterung anhand eines lokalen ML-Modells aus, das nur Bilder von Personen zu Analysezwecken erfasst. Die Daten werden über IoT Hub sicher bereitgestellt und aktualisiert.<br><br>|
| Azure | [Azure Event Hubs](/azure/event-hubs/) | Azure Event Hubs bietet eine skalierbare Plattform für die Erfassung anonymisierter Daten, die sich nahtlos in Azure Stream Analytics integrieren lassen. |
|  | [Azure Stream Analytics](/azure/stream-analytics/) | Ein Azure Stream Analytics-Auftrag aggregiert die anonymisierten Daten und gruppiert sie zur Visualisierung in 15 Sekunden große Zeitfenster. |
|  | [Microsoft Power BI](https://powerbi.microsoft.com/) | Power BI bietet eine benutzerfreundliche Dashboardschnittstelle zur Anzeige der Ausgabe von Azure Stream Analytics. |
| Azure Stack Hub | [App Service](/azure-stack/operator/azure-stack-app-service-overview) | Der App Service-Ressourcenanbieter stellt eine Basis für Edgekomponenten bereit, u. a. Hosting- und Verwaltungsfeatures für Web-Apps/-APIs und Azure Functions. |
| | Cluster für die [AKS-Engine](https://github.com/Azure/aks-engine) (Azure Kubernetes Service) | Der AKS-Ressourcenanbieter mit dem in Azure Stack Hub bereitgestellten AKS-Engine-Cluster bietet eine skalierbare, resiliente Engine für die Ausführung des Containers der Gesichtserkennungs-API. |
| | [Container der Gesichtserkennungs-API](/azure/cognitive-services/face/face-how-to-install-containers) in Azure Cognitive Services| Der Azure Cognitive Services-Ressourcenanbieter mit Containern der Gesichtserkennungs-API liefert demografische und emotionsbezogene Daten sowie Daten zur Erkennung einmaliger Besucher in das private Netzwerk von Contoso. |
| | Blob Storage | Die vom AI Dev Kit erfassten Bilder werden in Blob Storage von Azure Stack Hub hochgeladen. |
| | Azure-Funktionen | Eine in Azure Stack Hub ausgeführte Azure-Funktion empfängt Eingangsdaten aus dem Blobspeicher und verarbeitet die Interaktionen mit der Gesichtserkennungs-API. Die Funktion gibt anonymisierte Daten an einen Event Hubs-Cluster in Azure aus.<br><br>|

## <a name="issues-and-considerations"></a>Probleme und Überlegungen

Beachten Sie die folgenden Punkte bei der Entscheidung, wie diese Lösung implementiert werden soll:

### <a name="scalability"></a>Skalierbarkeit

Um diese Lösung auf mehrere Kameras und Standorte zu skalieren, müssen Sie sicherstellen, dass alle Komponenten die entsprechende höhere Last verarbeiten können. Möglicherweise müssen Sie Aktionen wie die folgenden ausführen:

- Erhöhen der Anzahl von Stream Analytics-Streamingeinheiten
- Aufskalieren der Bereitstellung der Gesichtserkennungs-API
- Erhöhen des Durchsatzes des Event Hubs-Clusters
- In Extremfällen ist möglicherweise eine Migration von Azure Functions zu einem virtuellen Computer erforderlich.

### <a name="availability"></a>Verfügbarkeit

Da in dieser Lösung verschiedene Ebenen involviert sind, müssen Sie darüber nachdenken, was bei Netzwerk- oder Stromausfällen passieren soll. Je nach Geschäftsanforderungen empfiehlt es sich unter Umständen, einen Mechanismus zu implementieren, der Bilder lokal zwischenspeichert und sie dann an Azure Stack Hub weiterleitet, wenn wieder eine Verbindung besteht. Wenn der Standort groß genug ist, ist die Bereitstellung einer Data Box Edge-Instanz mit einem auf diesen Standort ausgerichteten Gesichtserkennungs-API-Container möglicherweise eine bessere Option.

### <a name="manageability"></a>Verwaltbarkeit

Diese Lösung kann eine Vielzahl von Geräten und Standorten umfassen und damit recht unhandlich werden. Um neue Standorte und Geräte automatisch online zu schalten und auf dem neuesten Stand zu halten, können die [IoT-Dienste von Azure](/azure/iot-fundamentals/) verwendet werden.

### <a name="security"></a>Sicherheit

Da diese Lösung Bilder von Kunden erfasst, ist das Thema Sicherheit von größter Bedeutung. Stellen Sie sicher, dass alle Speicherkonten mit geeigneten Zugriffsrichtlinien geschützt sind, und rotieren Sie Schlüssel regelmäßig. Sorgen Sie dafür, dass Speicherkonten und Event Hubs über Beibehaltungsrichtlinien verfügen, die die Datenschutzbestimmungen des Unternehmens und des zuständigen Gesetzgebers erfüllen. Vergewissern Sie sich, dass Sie den Benutzerzugriff abstufen. Das Abstufen stellt sicher, dass Benutzer nur auf die Daten zugreifen können, die sie für ihre Rolle benötigen.

## <a name="next-steps"></a>Nächste Schritte

Weitere Informationen zu den in diesem Artikel behandelten Themen:

- Sehen Sie sich das [Muster für mehrstufige Daten](https://aka.ms/tiereddatadeploy) an, das vom Muster zur Ermittlung der Kundenfrequenz genutzt wird.
- Weitere Informationen zum benutzerdefinierten maschinellen Sehen finden Sie unter [Custom Vision AI Dev Kit](https://azure.github.io/Vision-AI-DevKit-Pages/). 

Wenn Sie bereit sind, das Lösungsbeispiel zu testen, fahren Sie mit dem [Bereitstellungsleitfaden für das Muster zur Ermittlung der Kundenfrequenz ](solution-deployment-guide-retail-footfall-detection.md) fort. In diesem Bereitstellungsleitfaden finden Sie detaillierte Anweisungen zum Bereitstellen und Testen der zugehörigen Komponenten.
