---
title: Hybridrelaymuster in Azure und Azure Stack Hub
description: Verwenden Sie das Hybridrelaymuster in Azure und Azure Stack Hub, um eine Verbindung mit von Firewalls geschützten Edgeressourcen herzustellen.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 03b20a20a04f620c977fb20e1ea26f5982e42721
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910435"
---
# <a name="hybrid-relay-pattern"></a>Hybridrelaymuster

Erfahren Sie, wie Sie mit dem Hybrid Relay-Muster und Azure Relay eine Verbindung mit Edge-Ressourcen oder Geräten herstellen, die von Firewalls geschützt werden

## <a name="context-and-problem"></a>Kontext und Problem

Edge-Geräte befinden sich oft hinter einer Unternehmensfirewall oder einem NAT-Gerät. Obwohl sie sicher sind, können sie möglicherweise nicht mit der öffentlichen Cloud oder Edgegeräten in anderen Unternehmensnetzwerken kommunizieren. Es kann notwendig sein, bestimmte Ports und Funktionen Benutzern in der öffentlichen Cloud auf sichere Weise verfügbar zu machen.

## <a name="solution"></a>Lösung

Das Hybrid Relay-Muster verwendet Azure Relay, um einen websockets-Tunnel zwischen zwei Endpunkten einzurichten, die nicht direkt miteinander kommunizieren können. Geräte, die nicht lokal sind, aber eine Verbindung mit einem lokalen Endpunkt herstellen müssen, werden mit einem Endpunkt in der öffentlichen Cloud verbunden. Dieser Endpunkt leitet den Datenverkehr auf vordefinierten Routen über einen sicheren Kanal um. Ein Endpunkt innerhalb der lokalen Umgebung empfängt den Datenverkehr und leitet ihn an das gewünschte Ziel weiter.

![Hybridrelaymuster: Lösungsarchitektur](media/pattern-hybrid-relay/solution-architecture.png)

So funktioniert das Hybridrelaymuster:

1. Ein Gerät stellt an einem vordefinierten Port eine Verbindung mit einem virtuellen Computer (VM) in Azure her.
2. Der Datenverkehr wird an die Azure Relay in Azure weitergeleitet.
3. Der virtuelle Computer auf Azure Stack Hub, der bereits eine langlebige Verbindung mit dem Azure Relay hergestellt hat, empfängt den Datenverkehr und leitet ihn an das Ziel weiter.
4. Der lokale Dienst oder Endpunkt verarbeitet die Anforderung.

## <a name="components"></a>Komponenten

Diese Lösung verwendet die folgenden Komponenten:

| Ebene | Komponente | BESCHREIBUNG |
|----------|-----------|-------------|
| Azure | Azure VM | Eine Azure-VM bietet einen öffentlich zugänglichen Endpunkt für die lokale Ressource. |
| | Azure Relay | Eine [Azure Relay](/azure/azure-relay/) stellt die Infrastruktur zum Verwalten des Tunnels und der Verbindung zwischen dem virtuellen Azure-Computer und Azure Stack Hub-VM bereit.|
| Azure Stack Hub | Compute | Eine Azure Stack Hub-VM stellt die Serverseite des Hybridrelaytunnels bereit. |
| | Storage | Der in Azure Stack Hub bereitgestellte AKS-Engine-Cluster bietet eine skalierbare, resiliente Engine für die Ausführung des Gesichtserkennungs-API-Containers.|

## <a name="issues-and-considerations"></a>Probleme und Überlegungen

Beachten Sie die folgenden Punkte bei der Entscheidung, wie diese Lösung implementiert werden soll:

### <a name="scalability"></a>Skalierbarkeit

Dieses Muster erlaubt auf dem Client und Server nur 1:1-Portzuordnungen. Wenn beispielsweise Port 80 für einen Dienst auf dem Azure-Endpunkt getunnelt ist, kann er nicht für einen anderen Dienst verwendet werden. Portzuordnungen müssen entsprechend geplant werden. Die Azure Relay und VMS sollten entsprechend skaliert werden, um den Datenverkehr zu verarbeiten.

### <a name="availability"></a>Verfügbarkeit

Diese Tunnel und Verbindungen sind nicht redundant. Um Hochverfügbarkeit sicherzustellen, sollten Sie Code zur Prüfung auf Fehler implementieren. Eine andere Möglichkeit besteht darin, einen Pool mit Azure Relay verbundenen VMS hinter einem Load Balancer zu haben.

### <a name="manageability"></a>Verwaltbarkeit

Diese Lösung kann eine Vielzahl von Geräten und Standorten umfassen und damit recht unhandlich werden. Mit den IoT-Diensten von Azure können neue Standorte und Geräte automatisch online geschaltet und auf dem neuesten Stand gehalten werden.

### <a name="security"></a>Sicherheit

Dieses hier gezeigte Muster ermöglicht im Edge-Bereich den ungehinderten Zugriff auf einen Port auf einem internen Gerät. Erwägen Sie, dem Dienst auf dem internen Gerät oder vor dem Hybridrelayendpunkt einen Authentifizierungsmechanismus hinzuzufügen.

## <a name="next-steps"></a>Nächste Schritte

Weitere Informationen zu den in diesem Artikel behandelten Themen:

- Dieses Muster verwendet Azure Relay. Weitere Informationen finden Sie in der [Azure Relay-Dokumentation](/azure/azure-relay/).
- Unter [Überlegungen zum Entwurf von Hybrid-Apps](overview-app-design-considerations.md) erfahren Sie mehr zu bewährten Methoden und erhalten Antworten auf weitere Fragen.
- Im Artikel zur [Azure Stack-Familie mit Produkten und Lösungen](/azure-stack) erfahren Sie mehr über das gesamte Portfolio von Produkten und Lösungen.

Wenn Sie bereit sind, das Lösungsbeispiel zu testen, fahren Sie mit dem [Bereitstellungsleitfaden für eine Hybridrelaylösung](https://aka.ms/hybridrelaydeployment) fort. In diesem Bereitstellungsleitfaden finden Sie detaillierte Anweisungen zum Bereitstellen und Testen der zugehörigen Komponenten.