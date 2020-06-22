---
title: Muster für mehrstufige Daten für die Analyse unter Verwendung von Azure und Azure Stack Hub
description: Hier erfahren Sie, wie Sie mithilfe von Azure und Azure Stack Hub eine Lösung für mehrstufige Daten in der Hybrid Cloud implementieren.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: b671fa9f47fa51ab6e40633c04964957d613fec2
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910479"
---
# <a name="tiered-data-for-analytics-pattern"></a>Mehrstufige Daten für Analysemuster

Dieses Muster veranschaulicht, wie Sie mit Azure Stack Hub und Azure Daten an mehreren lokalen und Cloudspeicherorten stagen, analysieren, verarbeiten, bereinigen und speichern können.

## <a name="context-and-problem"></a>Kontext und Problem

Eines der Probleme, mit denen Organisationen in der modernen IT-Landschaft konfrontiert sind, ist die sichere Speicherung, Verarbeitung und Analyse von Daten. Diese Aspekte spielen eine Rolle:

- Dateninhalt
- location
- Sicherheits- und Datenschutzanforderungen
- Zugriffsberechtigungen
- Wartung
- (Langfristige) Speicherung

Azure bietet in Kombination mit Azure Stack Hub kostengünstige Lösungen für auf Daten bezogene Problemstellungen. Diese Lösung lässt sich am besten durch ein dezentrales Fertigungs- oder Logistikunternehmen ausdrücken.

Die Lösung basiert auf dem folgenden Szenario:

- Ein großes Fertigungsunternehmen mit mehreren Niederlassungen.
- Schnelle und sichere Speicherung, Verarbeitung und Verteilung von Daten zwischen globalen Remotestandorten und der Unternehmenszentrale sind erforderlich.
- Aktivitäten von Mitarbeitern und Maschinen, Informationen zu Anlagen und Geschäftsberichtsdaten, die geschützt bleiben müssen. Die Daten müssen in geeigneter Form verteilt werden und regionalen Compliance- und Branchenvorschriften entsprechen.

## <a name="solution"></a>Lösung

Die Nutzung von lokalen und öffentlichen Cloudumgebungen erfüllt die Anforderungen von Unternehmen mit mehreren Standorten. Mit Azure Stack Hub wird eine schnelle, sichere und flexible Lösung zum Erfassen, Verarbeiten, Speichern und Verteilen von lokalen und Remotedaten bereitgestellt. Dieses Muster ist besonders nützlich, wenn sich die Anforderungen in den Bereichen Sicherheit, Vertraulichkeit, Unternehmensrichtlinie und gesetzliche Bestimmungen für Standorte und Benutzer ggf. unterscheiden.

![Muster für mehrstufige Daten für die Analyse: Lösungsarchitektur](media/pattern-tiered-data-analytics/solution-architecture.png)

## <a name="components"></a>Komponenten

Dieses Muster nutzt die folgenden Komponenten:

| Ebene | Komponente | BESCHREIBUNG |
|----------|-----------|-------------|
| Azure | Storage | Ein [Azure Storage](/azure/storage/)-Konto bietet einen neutralen Endpunkt für die Datennutzung. Azure Storage ist die Cloudspeicherlösung von Microsoft für moderne Datenspeicherszenarien. Azure Storage bietet einen massiv skalierbaren Objektspeicher für Datenobjekte und einen Dateisystemdienst für die Cloud. Der Dienst bietet außerdem einen Messagingspeicher für zuverlässiges Messaging und einen NoSQL-Speicher. |
| Azure Stack Hub | Storage | Ein [Azure Stack Hub](/azure-stack/user/azure-stack-storage-overview)-Speicherkonto wird für mehrere Dienste verwendet:<br><br>- **Blob Storage** für die Speicherung von Rohdaten. In Blob Storage können alle Arten von Text- oder Binärdaten enthalten sein, z. B. ein Dokument, eine Mediendatei oder ein Installationsprogramm einer App. Jedes Blob ist unter einem Container organisiert. Container bieten eine praktische Möglichkeit, Gruppen von Objekten Sicherheitsrichtlinien zuzuweisen. Ein Speicherkonto kann eine beliebige Anzahl von Containern enthalten, und ein Container kann eine beliebige Anzahl von Blobs enthalten, und zwar bis zum maximalen Kapazitätsgrenzwert des Speicherkontos (500 TB).<br>- **Blob Storage** für die Datenarchivierung. Kostengünstiger Speicher für die Archivierung kalter Daten bringt Vorteile. Beispiele kalter Daten sind Sicherungen, Medieninhalte, wissenschaftliche, Compliance- und Archivdaten. Im Allgemeinen kommen alle Daten, auf die selten zugegriffen wird, für kalten Speicher in Frage. Daten werden auf Grundlage von Attributen wie Zugriffshäufigkeit und Beibehaltungsdauer unterteilt. Auf Kundendaten wird zwar nur selten zugegriffen, sie erfordern jedoch eine ähnliche Latenz und Leistung wie heiße Daten.<br>- **Azure Queue Storage** als Speicher für verarbeitete Daten. Azure Queue Storage ermöglicht Cloudmessaging zwischen App-Komponenten. Bei der Entwicklung skalierbarer Apps werden App-Komponenten häufig entkoppelt, um eine unabhängige Skalierung zu ermöglichen. Queue Storage bietet asynchrones Messaging für die Kommunikation zwischen App-Komponenten, unabhängig davon, ob diese in der Cloud, auf dem Desktop, auf einem lokalen Server oder einem mobilen Gerät ausgeführt werden. Queue Storage unterstützt auch die Verwaltung asynchroner Aufgaben und den Aufbau von Prozessworkflows. |
| | Azure-Funktionen | Der Dienst [Azure Functions](/azure/azure-functions/) wird vom Ressourcenanbieter [Azure App Service in Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) bereitgestellt. Azure Functions ermöglicht es Ihnen, Ihren Code in einer einfachen, serverlosen Umgebung als Reaktion auf eine Vielzahl von Ereignissen auszuführen. Azure Functions kann nach Bedarf skaliert werden, ohne eine VM erstellen oder eine Web-App veröffentlichen zu müssen, und zwar mit der Programmiersprache Ihrer Wahl. Funktionen werden von der Lösung für Folgendes verwendet:<br><br>- **Dateneingang**<br>- **Datenbereinigung**. Mithilfe manuell ausgelöster Funktionen können Sie die geplante Verarbeitung, Bereinigung und Archivierung von Daten durchführen. Beispiele sind u. a. nächtliche Kundenlistenbereinigungen und die monatliche Berichtsverarbeitung.|

## <a name="issues-and-considerations"></a>Probleme und Überlegungen

Beachten Sie die folgenden Punkte bei der Entscheidung, wie diese Lösung implementiert werden soll:

### <a name="scalability"></a>Skalierbarkeit

Azure Functions und Speicherlösungen können skaliert werden, um Anforderungen in Bezug auf die Datenmenge und -verarbeitung zu erfüllen. Informationen und Ziele für die Azure-Skalierbarkeit finden Sie in der [Dokumentation zur Skalierbarkeit von Azure Storage](/azure/storage/common/storage-scalability-targets).

### <a name="availability"></a>Verfügbarkeit

Der Speicher ist für dieses Muster der wichtigste Verfügbarkeitsaspekt. Eine Verbindung über schnelle Links ist für die Verarbeitung und Verteilung von großen Datenmengen erforderlich.

### <a name="manageability"></a>Verwaltbarkeit

Die Verwaltbarkeit dieser Lösung hängt von den verwendeten Erstellungstools und der Einbindung der Quellcodeverwaltung ab.

## <a name="next-steps"></a>Nächste Schritte

Weitere Informationen zu den in diesem Artikel behandelten Themen:

- Siehe die Dokumentation zu [Azure Storage](/azure/storage/) und [Azure Functions](/azure/azure-functions/). Dieses Muster nutzt intensiv Azure Storage-Konten und Azure Functions – sowohl in Azure als auch in Azure Stack Hub.
- Unter [Überlegungen zum Entwurf von Hybrid-Apps](overview-app-design-considerations.md) erfahren Sie mehr zu bewährten Methoden und erhalten Antworten auf weitere Fragen.
- Im Artikel zur [Azure Stack-Familie mit Produkten und Lösungen](/azure-stack) erfahren Sie mehr über das gesamte Portfolio von Produkten und Lösungen.

Wenn Sie bereit sind, das Lösungsbeispiel zu testen, fahren Sie mit dem [Bereitstellungsleitfaden für eine Analyselösung für mehrstufige Daten](https://aka.ms/tiereddatadeploy) fort. In diesem Bereitstellungsleitfaden finden Sie detaillierte Anweisungen zum Bereitstellen und Testen der zugehörigen Komponenten.
