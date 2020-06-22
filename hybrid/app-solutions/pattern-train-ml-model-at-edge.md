---
title: Muster zum Trainieren eines Machine Learning-Modells im Edge-Bereich
description: Hier erfahren Sie, wie Sie mit Azure und Azure Stack Hub ein Machine Learning-Modell (ML) im Edge-Bereich trainieren.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: da1012cc8847f221de6bb540ba4e191e43920f41
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910412"
---
# <a name="train-machine-learning-model-at-the-edge-pattern"></a>Muster zum Trainieren eines Machine Learning-Modells im Edge-Bereich

Generieren Sie portierbare Machine Learning-Modelle (ML) anhand von Daten, die nur lokal vorhanden sind.

## <a name="context-and-problem"></a>Kontext und Problem

Viele Organisationen möchten mithilfe von Tools, die ihre Datenanalysten verstehen, Erkenntnisse aus ihren lokalen oder älteren Datenbeständen gewinnen. [Azure Machine Learning](/azure/machine-learning/) bietet cloudnative Tools zum Trainieren, Optimieren und Bereitstellen von ML- und Deep Learning-Modellen.  

Einige Datenmengen sind jedoch zu groß, um sie in die Cloud zu senden, oder können aus rechtlichen Gründen nicht in die Cloud übertragen werden. Anhand dieses Musters können Datenanalysten mit Azure Machine Learning Modelle trainieren, die lokale Daten und Computeressourcen nutzen.

## <a name="solution"></a>Lösung

Für das Muster zum Training im Edge-Bereich wird ein in Azure Stack Hub ausgeführter virtueller Computer (VM) verwendet. Die VM ist als Computeziel in Azure Machine Learning registriert, sodass sie auf Daten zugreifen kann, die nur lokal verfügbar sind. In diesem Fall werden die Daten im Blobspeicher von Azure Stack Hub gespeichert.

Nach dem Training des Modells wird es bei Azure ML registriert, in Containern organisiert und zur Bereitstellung einer Azure Container Registry-Instanz hinzugefügt. Für diese Iteration des Musters muss die Azure Stack Hub-Trainings-VM über das öffentliche Internet erreichbar sein.

[![Architektur für das Trainieren von ML-Modellen im Edge-Bereich](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)

Das Muster funktioniert wie folgt:

1. Die Azure Stack Hub-VM wird als Computeziel bei Azure ML bereitgestellt und registriert.
2. In Azure ML wird ein Experiment erstellt, das die Azure Stack Hub-VM als Computeziel verwendet.
3. Sobald das Modell trainiert ist, wird es registriert und in Containern organisiert.
4. Das Modell kann nun an Speicherorten bereitgestellt werden, die sich entweder lokal oder in der Cloud befinden.

## <a name="components"></a>Komponenten

Diese Lösung verwendet die folgenden Komponenten:

| Ebene | Komponente | BESCHREIBUNG |
|----------|-----------|-------------|
| Azure | Azure Machine Learning | [Azure Machine Learning](/azure/machine-learning/) orchestriert das Training des ML-Modells. |
| | Azure Container Registry | Azure ML packt das Modell in einen Container und speichert es für die Bereitstellung in einer [Azure Container Registry](/azure/container-registry/)-Instanz.|
| Azure Stack Hub | App Service | [Azure Stack Hub mit App Service](/azure-stack/operator/azure-stack-app-service-overview) stellt die Basis für die Komponenten im Edge-Bereich bereit. |
| | Compute | Eine Azure Stack Hub-VM, auf der Ubuntu mit Docker ausgeführt wird, wird zum Trainieren des ML-Modells eingesetzt. |
| | Storage | Private Daten können im Blob Storage von Azure Stack Hub gehostet werden. |

## <a name="issues-and-considerations"></a>Probleme und Überlegungen

Beachten Sie die folgenden Punkte bei der Entscheidung, wie diese Lösung implementiert werden soll:

### <a name="scalability"></a>Skalierbarkeit

Damit diese Lösung skalierbar ist, müssen Sie in Azure Stack Hub eine entsprechend große VM für das Training erstellen.

### <a name="availability"></a>Verfügbarkeit

Stellen Sie sicher, dass die Trainingsskripts und die Azure Stack Hub-VM Zugriff auf die für das Training verwendeten lokalen Daten haben.

### <a name="manageability"></a>Verwaltbarkeit

Sorgen Sie dafür, dass Modelle und Experimente ordnungsgemäß registriert sowie mit Versionsangaben und mit Tags versehen sind, um Verwechslungen bei der Modellbereitstellung zu vermeiden.

### <a name="security"></a>Sicherheit

Dieses Muster ermöglicht Azure Machine Learning den Zugriff auf lokale Daten, die möglicherweise vertraulich sind. Vergewissern Sie sich, dass das Konto, das für die SSH-Verbindung mit der Azure Stack Hub-VM verwendet wird, über ein sicheres Kennwort verfügt und dass Trainingsskripts keine Daten speichern oder in die Cloud hochladen.

## <a name="next-steps"></a>Nächste Schritte

Weitere Informationen zu den in diesem Artikel behandelten Themen:

- Eine Übersicht über ML und verwandte Themen finden Sie in der [Azure Machine Learning-Dokumentation](/azure/machine-learning).
- Unter [Azure Container Registry](/azure/container-registry/) erfahren Sie, wie Sie Images für Containerbereitstellungen erstellen, speichern und verwalten.
- Weitere Informationen zum Ressourcenanbieter und zur Bereitstellung finden Sie unter [App Service in Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview).
- Unter [Überlegungen zum Entwurf von Hybrid-Apps](overview-app-design-considerations.md) finden Sie weitere Informationen zu den bewährten Methoden und Antworten auf Ihre Fragen.
- Im Artikel zur [Azure Stack-Familie mit Produkten und Lösungen](/azure-stack) erfahren Sie mehr über das gesamte Portfolio von Produkten und Lösungen.

Wenn Sie bereit sind, das Lösungsbeispiel zu testen, fahren Sie mit dem [Bereitstellungsleitfaden zum Trainieren eines ML-Modells im Edge-Bereich](https://aka.ms/edgetrainingdeploy) fort. In diesem Bereitstellungsleitfaden finden Sie detaillierte Anweisungen zum Bereitstellen und Testen der zugehörigen Komponenten.
