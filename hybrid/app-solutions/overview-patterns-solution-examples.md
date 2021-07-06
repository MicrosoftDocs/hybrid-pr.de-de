---
title: Beispiele für Hybridmuster und -lösungen für Azure und Azure Stack Hub
description: Hier finden Sie eine Übersicht über Beispiele für Hybridmuster und -lösungen, mit denen Sie sich mit Hybridlösungen in Azure und Azure Stack Hub vertraut machen und solche Art von Lösungen erstellen können.
author: BryanLa
ms.topic: overview
ms.date: 05/24/2021
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 05/24/2021
ms.openlocfilehash: 9f3f13c23bec31c5132c7e90294356b9463fd72b
ms.sourcegitcommit: cf2c4033d1b169f5b63980ce1865281366905e2e
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 05/25/2021
ms.locfileid: "110343857"
---
# <a name="hybrid-solution-patterns-and-examples-for-azure-and-azure-stack"></a>Hybridlösungsmuster und -beispiele für Azure und Azure Stack

Microsoft stellt Azure- und Azure Stack-Produkte und -Lösungen als konsistentes Azure-Ökosystem bereit. Die Microsoft Azure Stack-Familie ist eine Erweiterung von Azure.

## <a name="the-hybrid-cloud-and-hybrid-apps"></a>Hybrid Cloud und Hybrid-Apps

Mit Azure Stack können Sie die Agilität des Cloud Computing für Ihre lokale Umgebung und den Edge über eine *Hybrid Cloud*-Infrastruktur nutzen. Mit Azure Stack Hub, Azure Stack HCI und Azure Stack Edge ist Azure nicht nur in der Cloud, sondern etwa auch in Ihren unabhängigen Rechenzentren, in Ihren Zweigstellen und im Außendienst genutzt werden. Diese vielfältigen Funktionen ermöglichen Folgendes:

- Wiederverwenden von Code und konsistentes Ausführen von cloudnativen Apps in Azure und Ihren lokalen Umgebungen.
- Ausführen herkömmlicher virtualisierter Workloads mit optionalen Verbindungen mit Azure-Diensten
- Übertragen von Daten in die Cloud oder Speichern der Daten in Ihrem unabhängigen Rechenzentrum zur Einhaltung von Richtlinien
- Ausführen von hardwarebeschleunigten Workloads für maschinelles Lernen bzw. containerbasierten oder virtualisierten Workloads mit Intelligent Edge

Apps, die mehrere Clouds nutzen, werden auch als *Hybrid-Apps* bezeichnet. Sie können Hybrid Cloud-Apps in Azure erstellen und sie anschließend in Ihrem verbundenen oder nicht verbundenen Rechenzentrum bereitstellen, egal wo sich dieses befindet.

Szenarien mit Hybrid-Apps unterscheiden sich stark je nach den für die Bereitstellung verfügbaren Ressourcen. Darüber hinaus bringen sie Überlegungen zu Geografie, Sicherheit, Internetzugriff usw. mit sich. Obwohl die hier beschriebenen Lösungsmuster und -beispiele möglicherweise nicht alle Anforderungen abdecken, bieten sie Richtlinien und Beispiele, die für die Implementierung von Hybridlösungen näher untersucht und wiederverwendet werden können.

## <a name="solution-patterns"></a>Lösungsmuster

Lösungsmuster filtern generalisierte wiederholbare Entwurfsrichtlinien aus realen Kundenszenarien und -umgebungen heraus. Ein Muster ist abstrakt, sodass es auf verschiedene Arten von Szenarien oder vertikalen Branchen angewendet werden kann. Jedes Muster dokumentiert den Kontext und das Problem und bietet einen Überblick über ein Lösungsbeispiel. Das Lösungsbeispiel ist als mögliche Implementierung des Musters gedacht.

Es gibt zwei Arten von Musterartikeln:

- Einzelnes Muster: Enthält Entwurfsrichtlinien für ein einzelnes allgemeines Szenario.
- Mehrere Muster: Enthält Entwurfsrichtlinien, bei denen mehrere Muster angewendet werden. Dieses Muster ist häufig zur Lösung komplexerer Szenarien oder branchenspezifischer Probleme erforderlich.

## <a name="solution-deployment-guides"></a>Leitfäden zur Lösungsbereitstellung

Schritt-für-Schritt-Anleitungen unterstützen Sie bei der Bereitstellung eines Lösungsbeispiels. Der Leitfaden kann auch auf ein begleitendes Codebeispiel im [Repository mit Lösungsbeispielen](https://github.com/Azure-Samples/azure-intelligent-edge-patterns) auf GitHub verweisen.

## <a name="next-steps"></a>Nächste Schritte

- Im Artikel zur [Azure Stack-Familie mit Produkten und Lösungen](/azure-stack) erfahren Sie mehr über das gesamte Portfolio von Produkten und Lösungen.
- In den Abschnitten „Muster“ und „Leitfäden zur Lösungsbereitstellung“ des Inhaltsverzeichnisses erfahren Sie mehr über Muster und Lösungen.
- Im Artikel mit [Entwurfsüberlegungen für Hybrid-Apps](overview-app-design-considerations.md) werden die wichtigen Aspekte in Bezug auf die Softwarequalität beschrieben, die für das Entwerfen, Bereitstellen und Betreiben von Hybrid-Apps erforderlich sind.
- [Richten Sie eine Entwicklungsumgebung in Azure Stack ein](/azure-stack/user/azure-stack-dev-start), und [stellen Sie Ihre erste App in Azure Stack bereit](/azure-stack/user/azure-stack-dev-start-deploy-app).
