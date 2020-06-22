---
title: Das DevOps-Muster in Azure Stack Hub
description: Hier erfahren Sie mehr über das DevOps-Muster, mit dem Sie die Konsistenz zwischen Bereitstellungen in Azure und Azure Stack Hub sicherstellen können.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 306cc9604a8e919724f9f76b7e5122d534d2d1ae
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910423"
---
# <a name="devops-pattern"></a>DevOps-Muster

Code von einem einzelnen Speicherort und Bereitstellen für mehrere Ziele in Entwicklungs-, Test- und Produktionsumgebungen, die sich möglicherweise in Ihrem Datacenter, privaten Clouds oder öffentlichen Clouds befinden.

## <a name="context-and-problem"></a>Kontext und Problem

Bei der Anwendungsbereitstellung sind Kontinuität, Sicherheit und Zuverlässigkeit für Organisationen und Entwicklungsteams unverzichtbar.

Für die Ausführung von Apps in jeder Zielumgebung ist häufig umgestalteter Code erforderlich. Dies bedeutet, dass eine App nicht vollständig portierbar ist. Sie muss für jede Umgebung aktualisiert, getestet und überprüft werden. Beispielsweise muss in einer Entwicklungsumgebung geschriebener Code umgeschrieben werden, damit er in einer Testumgebung funktioniert, und danach für die Produktionsumgebung erneut umgeschrieben werden. Außerdem ist dieser Code speziell an den Host gebunden. Dies erhöht die Kosten und Komplexität der Wartung der App. Jede Version der App ist an jede Umgebung gebunden. Die erhöhte Komplexität und Duplizierung erhöhen das Risiko von Beeinträchtigungen bei der Sicherheits- und Codequalität. Außerdem kann der Code nicht sofort erneut bereitgestellt werden, wenn Sie Hosts entfernen, bei deren Wiederherstellung ein Fehler aufgetreten ist, oder Sie weitere Hosts zum Bewältigen höherer Anforderungen bereitstellen.

## <a name="solution"></a>Lösung

Das DevOps-Muster ermöglicht das Erstellen, Testen und Bereitstellen einer App, die auf mehreren Clouds ausgeführt wird. Dieses Muster vereinigt die Praxis von Continuous Integration (CI) und Continuous Delivery. Mit Continuous Integration wird Code jedes Mal erstellt und getestet, wenn ein Teammitglied eine Änderung an der Versionskontrolle committet. Continuous Delivery automatisiert jeden einzelnen Schritt von einem Build bis zu einer Produktionsumgebung. Mit diesen beiden Prozessen wird ein Releaseprozess erstellt, der die Bereitstellung über mehrere Umgebungen unterstützt. Mit diesem Muster können Sie Ihren Code entwerfen und diesen anschließend in einer lokalen Umgebung, verschiedenen privaten Clouds und öffentlichen Clouds bereitstellen. Unterschiede bei der Umgebung erfordern eine Änderung der Konfigurationsdatei anstelle von Änderungen am Code.

![DevOps-Muster](media/pattern-cicd-pipeline/hybrid-ci-cd.png)

Mit konsistenten Entwicklungstools für lokale, private und öffentliche Cloudumgebungen können Sie die Ansätze mit Continuous Integration und Continuous Delivery implementieren. Apps und Dienste, die mithilfe des DevOps-Musters bereitgestellt werden, sind austauschbar und können an beiden dieser Orte ausgeführt werden. Dabei können Features und Funktionen der lokalen und öffentlichen Cloud genutzt werden.

Die Verwendung einer DevOps-Releasepipeline unterstützt Sie bei Folgendem:

- Initiieren eines neuen und auf Codecommits basierenden Builds für ein einzelnes Repository.
- Automatisches Bereitstellen Ihres neu erstellten Codes in der öffentlichen Cloud für Benutzerakzeptanztests.
- Automatisches Bereitstellen in eine private Cloud, nachdem Ihr Code den Test bestanden hat.

## <a name="issues-and-considerations"></a>Probleme und Überlegungen

Das DevOps-Muster soll unabhängig von der Zielumgebung die Konsistenz zwischen Bereitstellungen gewährleisten. Die Funktionen in Cloudumgebungen und lokalen Umgebungen unterscheiden sich jedoch. Beachten Sie die folgenden Punkte:

- Sind die Funktionen, Endpunkte, Dienste und andere Ressourcen in Ihrer Bereitstellung an den Zielstandorten der Bereitstellung verfügbar?
- Werden Konfigurationsartefakte an Standorten gespeichert, auf die über Clouds zugegriffen werden kann?
- Funktionieren Bereitstellungsparameter in allen Zielumgebungen?
- Sind in allen Zielclouds ressourcenspezifische Eigenschaften verfügbar?

Weitere Informationen finden Sie unter [Develop Azure Resource Manager templates for cloud consistency (Entwickeln von Azure Resource Manager-Vorlagen für die Cloudkonsistenz)](https://docs.microsoft.com/azure/azure-resource-manager/templates-cloud-consistency).

Bei der Entscheidung, wie dieses Muster implementiert werden soll, sind zusätzlich die folgenden Punkte zu beachten:

### <a name="scalability"></a>Skalierbarkeit

Automatisierungssysteme für die Bereitstellung machen einen der wichtigsten Steuerungspunkte in DevOps-Mustern aus. Implementierungen können variieren. Die Auswahl der richtigen Servergröße hängt von der Größe der erwarteten Workload ab. Das Skalieren von virtuellen Computern kostet mehr als das Skalieren von Containern. Damit Sie Container für die Skalierung verwenden können, muss Ihr Buildprozess jedoch mit Containern ausgeführt werden.

### <a name="availability"></a>Verfügbarkeit

Verfügbarkeit im Kontext von DevPattern bedeutet Folgendes: Sie können jegliche Zustandsinformationen im Zusammenhang mit Ihrem Workflow (etwa Testergebnisse, Codeabhängigkeiten oder andere Artefakte) wiederherstellen. Berücksichtigen Sie beim Bewerten der Verfügbarkeitsanforderungen zwei allgemeine Metriken:

- Recovery Time Objective (RTO) gibt an, wie lange Sie Vorgänge ohne ein System ausführen können.

- Recovery Point Objective (RPO) gibt an, wie viele Daten verloren gehen dürfen, wenn sich eine Dienstunterbrechung auf das System auswirkt.

In der Praxis geben RTO und RPO Redundanz und Sicherung an. In der globalen Azure-Cloud ist Verfügbarkeit keine Frage der Hardwarewiederherstellung (diese ist Teil von Azure), sondern eine Frage der Aufrechterhaltung des Zustands Ihres DevOps-Systems. In Azure Stack Hub kann die Hardwarewiederherstellung ein Aspekt sein.

Ein weiterer wichtiger Aspekt beim Entwerfen des Systems für die Bereitstellungsautomatisierung ist die Zugriffssteuerung und die ordnungsgemäße Verwaltung der Rechte, die zum Bereitstellen von Diensten in Cloudumgebungen benötigt werden. Welche Rechte sind zum Erstellen, Löschen oder Ändern von Bereitstellungen erforderlich? In der Regel sind zum Erstellen einer Ressourcengruppe in Azure beispielsweise mehrere Rechte erforderlich. Für das Bereitstellen von Diensten in der Ressourcengruppe werden allerdings noch weitere Rechte benötigt.

### <a name="manageability"></a>Verwaltbarkeit

Beim Entwerfen eines Systems, das auf einem DevOps-Muster basiert, muss die Automatisierung, Protokollierung und Benachrichtigung für jeden Dienst im Portfolio berücksichtigt werden. Verwenden Sie gemeinsame Dienste und/oder ein Anwendungsteam. Außerdem sollten Sie Sicherheitsrichtlinien und Governance nachverfolgen.

Stellen Sie Produktionsumgebungen und Entwicklungs-/Testumgebungen in separaten Ressourcengruppen in Azure oder Azure Stack Hub bereit. Anschließend können Sie die Ressourcen jeder Umgebung überwachen und die Abrechnungskosten nach Ressourcengruppe zusammenfassen. Sie können auch Ressourcen als Gruppe löschen. Dies ist für Testbereitstellungen nützlich.

## <a name="when-to-use-this-pattern"></a>Verwendung dieses Musters

Verwendung Sie dieses Muster für folgende Zwecke:

- Sie können Code in einer den Anforderungen Ihrer Entwickler entsprechenden Umgebung entwickeln und dann in einer lösungsspezifischen Umgebung bereitstellen, in der das Entwickeln von neuem Code schwierig ist.
- Sie können den Code und die Tools verwenden, die Ihre Entwickler verwenden möchten, sofern diese den Continuous Integration- und Continuous Delivery-Prozess im DevOps-Muster befolgen können.

Verwenden Sie dieses Muster in folgenden Fällen nicht:

- Sie können Infrastruktur, Bereitstellungsressourcen, Konfiguration, Identitäten und Sicherheitsaufgaben nicht automatisieren.
- Teams haben keinen Zugriff auf Hybrid Cloud-Ressourcen zum Implementieren eines Continuous Integration- oder Continuous Development-Ansatzes (CI/CD).

## <a name="next-steps"></a>Nächste Schritte

Weitere Informationen zu den in diesem Artikel behandelten Themen:

- Siehe die [Azure DevOps-Dokumentation](/azure/devops), um mehr über Azure DevOps und zugehörige Tools, einschließlich Azure Repos und Azure Pipelines, zu erfahren.
- Im Artikel zur [Azure Stack-Familie mit Produkten und Lösungen](/azure-stack) erfahren Sie mehr über das gesamte Portfolio von Produkten und Lösungen.

Wenn Sie bereit sind, das Lösungsbeispiel zu testen, fahren Sie mit dem [Bereitstellungsleitfaden für eine hybride CI/CD-Lösung für DevOps](https://aka.ms/hybriddevopsdeploy) fort. In diesem Bereitstellungsleitfaden finden Sie detaillierte Anweisungen zum Bereitstellen und Testen der zugehörigen Komponenten. Sie erfahren, wie Sie mithilfe einer hybriden Pipeline für Continuous Integration/Continuous Delivery (CI/CD) eine App in Azure und Azure Stack Hub bereitstellen.
