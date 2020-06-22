---
title: Fehlbestandserkennung mit Azure und Azure Stack Edge
description: Hier erfahren Sie, wie Sie Azure- und Azure Stack Edge-Dienste verwenden, um eine Fehlbestandserkennung zu implementieren.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 865f63bc4234e50ed169aa29cefdb1886750594c
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910453"
---
# <a name="out-of-stock-detection-at-the-edge-pattern"></a>Muster zur Fehlbestandserkennung im Edge-Bereich

Dieses Muster veranschaulicht, wie Sie mit einem Azure Stack Edge- oder Azure IoT Edge-Gerät und Netzwerkkameras feststellen können, ob Regale Fehlbestände aufweisen.

## <a name="context-and-problem"></a>Kontext und Problem

Stationäre Einzelhandelsgeschäfte verlieren Umsatz, sobald Kunden, die einen Artikel suchen, diesen nicht im Regal finden. Der Artikel kann sich jedoch im hinteren Teil des Geschäfts befinden und nicht wieder aufgefüllt worden sein. Geschäfte möchten ihr Personal effizienter einsetzen und automatisch benachrichtigt werden, wenn Artikel wieder aufgefüllt werden müssen.

## <a name="solution"></a>Lösung

Im Lösungsbeispiel wird ein Edgegerät, wie beispielsweise Azure Stack Edge, in jedem Geschäft verwendet, das Daten von Kameras im Geschäft effizient verarbeitet. Dieses optimierte Design ermöglicht es Händlern, nur relevante Ereignisse und Bilder in die Cloud zu senden. Das Design spart Bandbreite und Speicherplatz und stellt den Schutz von Kundendaten sicher. Während Frames von jeder Kamera gelesen werden, verarbeitet ein ML-Modell das Bild und gibt alle Bereiche mit Fehlbeständen zurück. Das Bild und die Bereiche mit Fehlbeständen werden in einer lokalen Web-App angezeigt. Diese Daten können an eine Time Series Insight-Umgebung gesendet werden, um in Power BI Erkenntnisse anzuzeigen.

![Architektur einer Edge-Lösung zum Erkennen von Fehlbeständen](media/pattern-out-of-stock-at-edge/solution-architecture.png)

Die Lösung funktioniert so:

1. Bilder werden von einer Netzwerkkamera über HTTP oder RTSP aufgenommen.
2. Die Größe des Bilds wird geändert und zum Rückschlusstreiber gesendet, der mit dem ML-Modell kommuniziert, um festzustellen, ob es Bilder zu einem Fehlbestand gibt.
3. Das ML-Modell gibt alle Bereiche mit Fehlbeständen zurück.
4. Der Rückschlusstreiber lädt das unbearbeitete Bild in ein Blob hoch (sofern angegeben) und sendet die Ergebnisse aus dem Modell an Azure IoT Hub und einen Begrenzungsrahmenprozessor auf dem Gerät.
5. Der Begrenzungsrahmenprozessor fügt dem Bild Begrenzungsrahmen hinzu und speichert den Bildpfad in einer In-Memory Database.
6. Die Web-App fragt Bildern ab und zeigt sie in der Reihenfolge des Empfangs.
7. Nachrichten von IoT Hub werden in Time Series Insights aggregiert.
8. Power BI zeigt einen interaktiven Bericht zu Artikeln mit Fehlbeständen im Zeitverlauf mit den Daten aus Time Series Insights an.


## <a name="components"></a>Komponenten

Diese Lösung verwendet die folgenden Komponenten:

| Ebene | Komponente | BESCHREIBUNG |
|----------|-----------|-------------|
| Lokale Hardware | Netzwerkkamera | Eine Netzwerkkamera mit HTTP- oder RTSP-Feed ist erforderlich, um die Bilder für Rückschlüsse zu liefern. |
| Azure | Azure IoT Hub | [Azure IoT Hub](/azure/iot-hub/) kümmert sich um die Gerätebereitstellung und das Messaging für die Edgegeräte. |
|  | Azure Time Series Insights | [Azure Time Series Insights](/azure/time-series-insights/) speichert die Nachrichten von IoT Hub für die Visualisierung. |
|  | Power BI | [Microsoft Power BI](https://powerbi.microsoft.com/) bietet geschäftsorientierte Berichte zu Fehlbestandsereignissen. Power BI bietet eine benutzerfreundliche Dashboardschnittstelle zur Anzeige der Ausgabe von Azure Stream Analytics. |
| Azure Stack Edge oder<br>Azure IoT Edge-Gerät | Azure IoT Edge | [Azure IoT Edge](/azure/iot-edge/) orchestriert die Laufzeit für die lokalen Container und übernimmt die Verwaltung und Aktualisierung von Geräten.|
| | Project Brainwave für Azure | Auf einem Azure Stack Edge-Gerät verwendet [Project Brainwave](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) FPGAs (Field-Programmable Gate Arrays) zum Beschleunigen von ML-Rückschlüssen.|

## <a name="issues-and-considerations"></a>Probleme und Überlegungen

Beachten Sie die folgenden Punkte bei der Entscheidung, wie diese Lösung implementiert werden soll:

### <a name="scalability"></a>Skalierbarkeit

Die meisten Machine Learning-Modelle können abhängig von der bereitgestellten Hardware nur mit einer bestimmten Anzahl von Frames pro Sekunde ausgeführt werden. Bestimmen Sie die optimale Abtastrate Ihrer Kamera(s), um sicherzustellen, dass es in der ML-Pipeline nicht zu Staus kommt. Verschiedene Arten von Hardware verarbeiten unterschiedliche Anzahlen von Kameras und Frameraten.

### <a name="availability"></a>Verfügbarkeit

Wichtig ist zu bedenken, was passieren kann, wenn die Verbindung mit dem Edgegerät unterbrochen wird. Berücksichtigen Sie, welche Daten im Time Series Insights- und Power BI-Dashboard verloren gehen könnten. Die vorliegende Beispiellösung ist nicht auf Hochverfügbarkeit ausgelegt.

### <a name="manageability"></a>Verwaltbarkeit

Diese Lösung kann eine Vielzahl von Geräten und Standorten umfassen und damit recht unhandlich werden. Mit den IoT-Diensten von Azure können neue Standorte und Geräte automatisch online geschaltet und auf dem neuesten Stand gehalten werden. Zudem sind die ordnungsgemäßen Verfahren für Datengovernance zu befolgen.

### <a name="security"></a>Sicherheit

Von diesem Muster werden potenziell vertrauliche Daten verarbeitet. Stellen Sie sicher, dass die Schlüssel regelmäßig rotiert werden und die Berechtigungen für das Azure Storage-Konto und die lokalen Freigaben ordnungsgemäß zugewiesen sind.

## <a name="next-steps"></a>Nächste Schritte

Weitere Informationen zu den in diesem Artikel behandelten Themen:
- In diesem Muster werden mehrere IoT-bezogene Dienste genutzt, darunter [Azure IoT Edge](/azure/iot-edge/), [Azure IoT Hub](/azure/iot-hub/) und [Azure Time Series Insights](/azure/time-series-insights/).
- Weitere Informationen zu Microsoft Project Brainwave finden Sie in [der Ankündigung im Blog](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) und im [Video zu Azure Accelerated Machine Learning mit Project Brainwave](https://www.youtube.com/watch?v=DJfMobMjCX0).
- Unter [Überlegungen zum Entwurf von Hybrid-Apps](overview-app-design-considerations.md) erfahren Sie mehr zu bewährten Methoden und erhalten Antworten auf weitere Fragen.
- Im Artikel zur [Azure Stack-Familie mit Produkten und Lösungen](/azure-stack) erfahren Sie mehr über das gesamte Portfolio von Produkten und Lösungen.

Wenn Sie bereit sind, das Lösungsbeispiel zu testen, fahren Sie mit dem [Bereitstellungsleitfaden für eine Analyselösung für mehrstufige Daten](https://aka.ms/edgeinferencingdeploy) fort. In diesem Bereitstellungsleitfaden finden Sie detaillierte Anweisungen zum Bereitstellen und Testen der zugehörigen Komponenten.
