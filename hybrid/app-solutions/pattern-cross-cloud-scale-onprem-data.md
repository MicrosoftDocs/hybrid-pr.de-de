---
title: Muster für die cloudübergreifende Skalierung (lokale Daten) in Azure Stack Hub
description: Hier erfahren Sie, wie Sie in Azure und Azure Stack Hub eine skalierbare, cloudübergreifende App erstellen, die lokale Daten verwendet.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: edbb608fbf8e5288f29572bfe4cca98ffb3cb8fc
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910424"
---
# <a name="cross-cloud-scaling-on-premises-data-pattern"></a>Muster für die cloudübergreifende Skalierung (lokale Daten)

Erfahren Sie, wie Sie eine Hybrid-App entwickeln, die sich über Azure und Azure Stack Hub erstreckt. Dieses Muster zeigt auch, wie Sie eine einzelne lokale Datenquelle für Compliancezwecke verwenden.

## <a name="context-and-problem"></a>Kontext und Problem

Viele Organisationen erfassen und speichern riesige Mengen vertraulicher Kundendaten. Häufig werden sie aufgrund unternehmensinterner oder gesetzlicher Vorschriften daran gehindert, vertrauliche Daten in der öffentlichen Cloud zu speichern. Diese Organisationen möchten auch in den Genuss der Skalierbarkeit der öffentlichen Cloud kommen. Die öffentliche Cloud kann saisonale Spitzen beim Datenverkehr abfedern, sodass Kunden nur entsprechend ihren Anforderungen für Hardwarenutzung zahlen müssen.

## <a name="solution"></a>Lösung

Die Lösung nutzt die Vorzüge bei der Einhaltung von Vorschriften der privaten Cloud und kombiniert sie mit der Skalierbarkeit der öffentlichen Cloud. Die Hybrid Cloud auf Grundlage von Azure und Azure Stack Hub bietet Entwicklern eine einheitliche Erfahrung. Diese Einheitlichkeit ermöglicht Ihnen, dass ihre Fähigkeiten sowohl hinsichtlich öffentlicher Cloud als auch lokaler Umgebungen zum Tragen kommen.

Der Leitfaden für die Bereitstellung von Lösungen ermöglicht Ihnen das Bereitstellen einer identischen Web-App in einer öffentlichen und privaten Cloud. Sie können auch auf ein Netzwerk zugreifen, das nicht über das Internet geroutet werden kann und in der privaten Cloud gehostet wird. Die Web-Apps werden auf Last überwacht. Bei einem signifikanten Anstieg des Datenverkehrs werden DNS-Einträge von einem Programm so modifiziert, dass der Datenverkehr zur öffentlichen Cloud umgeleitet wird. Wenn der Datenverkehr nicht mehr so stark ist, werden die DNS-Einträge so aktualisiert, dass der Datenverkehr wieder zurück zur privaten Cloud geleitet wird.

[![Muster für die cloudübergreifende Skalierung mit lokalen Daten](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)

## <a name="components"></a>Komponenten

Diese Lösung verwendet die folgenden Komponenten:

| Ebene | Komponente | BESCHREIBUNG |
|----------|-----------|-------------|
| Azure | Azure App Service | Mit [Azure App Service](/azure/app-service/) können Sie Web-Apps, Rest-API-Apps und Azure Functions erstellen und hosten. Und dies in der Programmiersprache Ihrer Wahl, ohne Infrastruktur verwalten zu müssen. |
| | Virtuelles Azure-Netzwerk| [Azure Virtual Network (VNET)](/azure/virtual-network/virtual-networks-overview) ist der grundlegende Baustein für private Netzwerke in Azure. Mit einem VNET können zahlreiche Arten von Azure-Ressourcen, beispielsweise virtuelle Computer (VMs) sicher untereinander sowie mit dem Internet und lokalen Netzwerken kommunizieren. Die Lösung veranschaulicht außerdem die Nutzung weiterer Netzwerkkomponenten:<br>- App- und Gatewaysubnetze<br>- lokales Netzwerkgateway<br>- Gateway für virtuelle Netzwerke, das als Site-to-Site-VPN-Gatewayverbindung fungiert<br>- öffentliche IP-Adresse<br>- Point-to-Site-VPN-Verbindung<br>- Azure DNS zum Hosten von DNS-Domänen und Bereitstellen der Namensauflösung |
| | Azure Traffic Manager | [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) ist ein auf DNS basierender Lastenausgleichsdienst. Der Dienst ermöglicht die Verteilung von Benutzerdatenverkehr für Dienstendpunkte in unterschiedlichen Rechenzentren. |
| | Azure Application Insights | [Application Insights](/azure/azure-monitor/app/app-insights-overview) ist ein erweiterbarer Dienst zur Verwaltung der Anwendungsleistung für Webentwickler, die Apps auf mehreren Plattformen erstellen und verwalten.|
| | Azure-Funktionen | Mit [Azure Functions](/azure/azure-functions/) können Sie Code in einer serverlosen Umgebung ausführen, ohne vorher eine VM erstellen oder eine Web-App veröffentlichen zu müssen. |
| | Automatische Skalierung in Azure | [Automatische Skalierung](/azure/azure-monitor/platform/autoscale-overview) ist ein integriertes Feature von Cloud Services, VMs und Web-Apps. Es ermöglicht Apps, bei sich ändernden Anforderungen die bestmögliche Leistung zu erbringen. Apps passen sich an Datenverkehrsspitzen an, benachrichtigen Sie, wenn sich Metriken ändern, und werden nach Bedarf skaliert. |
| Azure Stack Hub | IaaS/Compute | Azure Stack Hub ermöglicht Ihnen, dasselbe App-Modell, Self-Service-Portal und die APIs zu verwenden, die von Azure aktiviert wurden. Azure Stack Hub IaaS bietet eine breite Palette von Open-Source-Technologien für einheitliche Hybrid Cloud-Bereitstellungen. Das Lösungsbeispiel verwendet beispielsweise eine Windows Server-VM für SQL Server.|
| | Azure App Service | Genau wie die Azure-Web-App verwendet die Lösung [Azure App Service in Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) zum Hosten der Web-App. |
| | Netzwerk | Das virtuelle Azure Stack Hub-Netzwerk funktioniert genauso wie das virtuelle Azure-Netzwerk. Es verwendet viele identische Netzwerkkomponenten, einschließlich benutzerdefinierter Hostnamen.
| Azure DevOps Services | Registrieren | Nutzen Sie das schnelle Einrichten von Continuous Integration für Build, Test und Bereitstellung. Weitere Informationen finden Sie unter [Registrieren und Anmelden bei Azure DevOps](/azure/devops/user-guide/sign-up-invite-teammates?view=azure-devops). |
| | Azure Pipelines | Verwenden Sie [Azure Pipelines](/azure/devops/pipelines/agents/agents?view=azure-devops) für Continuous Integration/Continuous Delivery. Azure Pipelines ermöglicht Ihnen die Verwaltung gehosteter Build- und Release-Agents und ihrer Definitionen. |
| | Coderepository | Nutzen Sie zum Optimieren Ihrer Entwicklungspipeline mehrere Coderepositorys. Verwenden Sie vorhandene Coderepositorys in GitHub, Bitbucket, Dropbox, OneDrive und Azure Repos. |

## <a name="issues-and-considerations"></a>Probleme und Überlegungen

Beachten Sie die folgenden Punkte bei der Entscheidung, wie diese Lösung implementiert werden soll:

### <a name="scalability"></a>Skalierbarkeit

Azure und Azure Stack Hub sind besonders geeignet, die Anforderungen des heutigen global verteilten Geschäftsbetriebs zu erfüllen.

#### <a name="hybrid-cloud-without-the-hassle"></a>Hybrid Cloud ohne Aufwand

Microsoft bietet in einer zentralen Lösung eine konkurrenzlose Integration lokaler Ressourcen mit Azure Stack Hub und Azure. Durch diese Integration entfällt der Aufwand für die Verwaltung mehrerer Punktlösungen und verschiedener Cloudanbieter. Dank cloudübergreifender Skalierung ist die Leistungsfähigkeit von Azure nur ein paar Mausklicks entfernt. Verbinden Sie Ihre Azure Stack Hub-Instanz einfach mit Azure und Cloudbursting, sodass Ihre Daten und Apps bei Bedarf in Azure verfügbar sind.

- Vermeiden Sie die Notwendigkeit der Errichtung und Wartung eines sekundären Standorts für die Notfallwiederherstellung.
- Sparen Sie Zeit und Geld, indem Sie auf Bandsicherungen verzichten und Sicherungsdaten bis zu 99 Jahre in Azure aufbewahren.
- Migrieren Sie problemlos laufende Hyper-V-, physische (in der Vorschauphase) und VMware-Workloads (in der Vorschauphase) nach Azure, um die Wirtschaftlichkeit und Flexibilität der Cloud zu nutzen.
- Führen Sie rechenintensive Berichte oder Analysen in einer replizierten Kopie Ihrer lokalen Ressourcen in Azure aus, ohne Produktionsworkloads zu beeinträchtigen.
- Nutzen Sie Cloudbursting, und führen Sie lokale Workloads in Azure aus, und zwar bei Bedarf mit größeren Computevorlagen. Die Hybridlösung bietet Ihnen die benötigte Leistung, wenn Sie sie brauchen.
- Erstellen Sie mit wenigen Klicks mehrschichtige Entwicklungsumgebungen in Azure, und replizieren Sie sogar aktuelle Produktionsdaten in Ihre Entwicklungs-/Testumgebung, um sie in Quasi-Echtzeit synchron zu halten.

#### <a name="economy-of-cross-cloud-scaling-with-azure-stack-hub"></a>Wirtschaftlichkeit der cloudübergreifenden Skalierung mit Azure Stack Hub

Der Hauptvorteil des Cloudburstings ist die Wirtschaftlichkeit. Sie zahlen für die zusätzlichen Ressourcen nur dann, wenn eine Nachfrage danach besteht. Dadurch entfallen unnötige Ausgaben für zusätzliche Kapazitäten oder der Versuch, Bedarfsspitzen und -schwankungen vorherzusagen.

#### <a name="reduce-high-demand-loads-into-the-cloud"></a>Verringern anspruchsvoller Verarbeitungslasten durch Verlagerung in die Cloud

Die cloudübergreifende Skalierung kann zum Bewältigen von Verarbeitungslasten genutzt werden. Die Lastverteilung erfolgt durch die Verlagerung von Basis-Apps in die öffentliche Cloud, wodurch lokale Ressourcen für unternehmenskritische Apps frei werden. Eine App kann zunächst der privaten Cloud zugewiesen werden und dann in die öffentliche Cloud übergehen – aber nur, wenn es nötig ist, um Anforderungen zu erfüllen.

### <a name="availability"></a>Verfügbarkeit

Die globale Bereitstellung birgt ihre eigenen Herausforderungen, wie z. B. variable Konnektivität und unterschiedliche staatliche Vorschriften je nach Region. Entwickler müssen bloß eine App entwickeln und können diese dann aus verschiedenen Gründen bei unterschiedlichen Anforderungen einsetzen. Stellen Sie Ihre App in der öffentlichen Azure-Cloud und anschließend weitere Instanzen oder Komponenten lokal bereit. Sie können Datenverkehr zwischen allen Instanzen mithilfe von Azure verwalten.

### <a name="manageability"></a>Verwaltbarkeit

#### <a name="a-single-consistent-development-approach"></a>Ein einzelner, einheitlicher Entwicklungsansatz

Azure und Azure Stack Hub ermöglichen Ihnen, einen einheitlichen Satz von Entwicklungstools in der gesamten Organisation einzusetzen. Diese Einheitlichkeit erleichtert die Implementierung einer Praxis für Continuous Integration und Continuous Development (CI/CD). Zahlreiche Apps und Dienste, die in Azure und Azure Stack Hub bereitgestellt werden, sind austauschbar und können in beiden Umgebungen reibungslos ausgeführt werden.

Eine hybride CI/CD-Pipeline kann Ihnen bei folgenden Aufgaben helfen:

- Auslösen eines neuen und auf Codecommits basierenden Builds für Ihr Coderepository.
- Stellen Sie Ihren neu erstellten Code für Benutzerakzeptanztests automatisch in Azure bereit.
- Sobald Ihr Code den Test bestanden hat, stellen Sie diesen automatisch in Azure Stack Hub bereit.

### <a name="a-single-consistent-identity-management-solution"></a>Ein einzelne, einheitliche Identitätsverwaltungslösung

Azure Stack Hub funktioniert sowohl mit Azure Active Directory (Azure AD) als auch mit Active Directory-Verbunddienste (AD FS). Azure Stack Hub kann mit Azure AD in vernetzten Szenarien verwendet werden. In nicht vernetzten Umgebungen können Sie AD FS als Lösung ohne Netzwerkverbindung nutzen. Dienstprinzipale werden verwendet, um Zugriff auf Apps zu gewähren, sodass sie Ressourcen über Azure Resource Manager bereitstellen oder konfigurieren können.

### <a name="security"></a>Sicherheit

#### <a name="ensure-compliance-and-data-sovereignty"></a>Sicherstellen von Compliance und Datenhoheit

Azure Stack Hub ermöglicht Ihnen die Ausführung des gleichen Diensts in mehreren Ländern, und zwar so wie bei Verwendung einer öffentlichen Cloud. Durch die Bereitstellung derselben App in Rechenzentren in den einzelnen Ländern können Anforderungen an Datenhoheit erfüllt werden. Diese Fähigkeit stellt sicher, dass personenbezogene Daten innerhalb der Grenzen der einzelnen Länder gespeichert bleiben.

#### <a name="azure-stack-hub---security-posture"></a>Azure Stack Hub: Sicherheitsstatus

Ohne einen soliden kontinuierlichen Wartungsprozess kann von Sicherheitsstatus keine Rede sein. Aus diesem Grund hat Microsoft in ein Orchestrierungsmodul investiert, das Patches und Updates nahtlos in der gesamten Infrastruktur anwendet.

Dank Partnerschaften mit Azure Stack Hub-OEM-Partnern dehnt Microsoft denselben Sicherheitsstatus auf OEM-spezifische Komponenten aus. Dies gilt beispielsweise für den Hardwarelebenszyklus-Host (Hardware Lifecycle Host, HLH) und die darauf ausgeführte Software. Diese Partnerschaft stellt sicher, dass Azure Stack Hub in der gesamten Infrastruktur einen einheitlichen, zuverlässigen Sicherheitsstatus aufweist. Kunden können wiederum ihre App-Workloads erstellen und absichern.

#### <a name="use-of-service-principals-via-powershell-cli-and-azure-portal"></a>Verwendung von Dienstprinzipalen über PowerShell, CLI und Azure-Portal

Um Ressourcenzugriff auf ein Skript oder eine Anwendung zu gewähren, richten Sie eine Identität für Ihre App ein, und authentifizieren Sie die App mit ihren eigenen Anmeldeinformationen. Diese Identität wird als Dienstprinzipal bezeichnet und ermöglicht Folgendes:

- Sie können der App-Identität Berechtigungen zuweisen, die sich von Ihren eigenen Berechtigungen unterscheiden und genau auf die Anforderungen der App beschränkt sind.
- Sie können ein Zertifikat für die Authentifizierung beim Ausführen eines unbeaufsichtigten Skripts verwenden.

Informationen zum Erstellen eines Dienstprinzipals und Verwenden eines Zertifikats für Anmeldeinformationen finden Sie unter [Verwenden einer App-Identität für den Ressourcenzugriff](/azure-stack/operator/azure-stack-create-service-principals).

## <a name="when-to-use-this-pattern"></a>Verwendung dieses Musters

- Ihre Organisation nutzt einen DevOps-Ansatz oder hat dies für die nahe Zukunft geplant.
- Sie möchten CI/CD-Methoden für Ihre gesamte Azure Stack Hub-Implementierung und die öffentliche Cloud implementieren.
- Sie möchten die CI/CD-Pipeline für Cloud- und lokale Umgebungen konsolidieren.
- Sie möchten die Möglichkeit haben, Apps mit cloudbasierten oder lokalen Diensten nahtlos zu entwickeln.
- Sie möchten einheitliche Entwicklerfähigkeiten für cloudbasierte und lokale Anwendungen nutzen.
- Sie verwenden Azure, haben aber Entwickler, die in einer lokalen Azure Stack Hub-Cloud arbeiten.
- Ihre lokalen Apps verzeichnen Nachfragespitzen aufgrund saisonaler, zyklischer oder unvorhersehbarer Schwankungen.
- Sie verfügen über lokale Komponenten und möchten die Cloud nutzen, um Sie nahtlos zu skalieren.
- Sie wünschen sich Skalierbarkeit mithilfe der Cloud, möchten aber Ihre App so weit wie möglich lokal ausführen.

## <a name="next-steps"></a>Nächste Schritte

Weitere Informationen zu den in diesem Artikel behandelten Themen:

- Sehen Sie sich das Video [Dynamically scale apps between data centers and public cloud](https://www.youtube.com/watch?v=2lw8zOpJTn0) (Dynamisches Skalieren zwischen Rechenzentren und öffentlicher Cloud) mit einer Übersicht der Nutzung dieses Musters an.
- Unter [Überlegungen zum Entwurf von Hybrid-Apps](overview-app-design-considerations.md) erfahren Sie mehr zu bewährten Methoden und erhalten Antworten auf mögliche weitere Fragen.
- Für dieses Muster wird die Azure Stack Hub-Produktfamilie, einschließlich Azure Stack Hub, verwendet. Im Artikel zur [Azure Stack-Familie mit Produkten und Lösungen](/azure-stack) erfahren Sie mehr über das gesamte Portfolio von Produkten und Lösungen.

Wenn Sie bereit sind, das Lösungsbeispiel zu testen, fahren Sie mit dem [Bereitstellungsleitfaden für eine cloudübergreifende Skalierungslösung (lokale Daten)](solution-deployment-guide-cross-cloud-scaling-onprem-data.md) fort. In diesem Bereitstellungsleitfaden finden Sie detaillierte Anweisungen zum Bereitstellen und Testen der zugehörigen Komponenten.
