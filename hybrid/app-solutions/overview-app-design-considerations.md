---
title: Überlegungen zum Entwurf von Hybrid-Apps in Azure und Azure Stack Hub
description: Hier erfahren Sie mehr zu Entwurfsüberlegungen beim Erstellen einer Hybrid-App für die intelligente Cloud und Intelligent Edge, u. a. zu Platzierung, Skalierbarkeit, Verfügbarkeit und Resilienz.
author: BryanLa
ms.topic: article
ms.date: 06/07/2020
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: c56575ac8ea6cb35d60bb9419269db89b0295721
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477217"
---
# <a name="hybrid-app-design-considerations"></a>Überlegungen zum Entwurf von Hybrid-Apps

Microsoft Azure ist die einzige konsistente Hybrid Cloud-Anwendung. Mit Azure können Sie einen Nutzen aus Ihren Investitionen in die Entwicklung ziehen und Apps verwenden, die sich über globale Azure-Instanzen, Sovereign Azure-Clouds sowie Azure Stack (eine Erweiterung von Azure in Ihrem Rechenzentrum) erstrecken können. Apps, die mehrere Clouds nutzen, werden auch als *Hybrid-Apps* bezeichnet.

Im [*Azure-Anwendungsarchitekturleitfaden*](/azure/architecture/guide) wird eine strukturierte Vorgehensweise zum Entwerfen von Apps beschrieben, die skalierbar, resilient und hochverfügbar sind. Die im [*Azure-Anwendungsarchitekturleitfaden*](/azure/architecture/guide) beschriebenen Überlegungen gelten auch für Apps, die für eine einzelne Cloud entworfen werden, sowie Apps, die sich über mehrere Clouds erstrecken.

In diesem Artikel werden die [*Säulen der Softwarequalität*](/azure/architecture/guide/pillars) erläutert, die im [*Azure-* ](/azure/architecture/guide/)[*Anwendungsarchitekturleitfaden*](/azure/architecture/guide/) angesprochen werden. Der Fokus liegt dabei insbesondere auf dem Entwerfen von Hybrid-Apps. Außerdem wird hier die Säule *Platzierung* erläutert, da Hybrid-Apps nicht explizit einer einzelnen Cloud oder einem lokalen Rechenzentrum zuzuordnen sind.

Hybridszenarios unterscheiden sich stark je nach den für die Bereitstellung verfügbaren Ressourcen und bringen Überlegungen zu Geografie, Sicherheit, Internetzugriff usw. mit sich. In diesem Leitfaden können zwar nicht die speziell für Sie geltenden Umstände besprochen werden, aber Sie erhalten Informationen zu einigen wichtigen Richtlinien und bewährten Methoden, auf denen Sie aufbauen können. Das erfolgreiche Entwerfen, Konfigurieren, Bereitstellen und Verwalten einer hybriden Anwendungsarchitektur umfasst viele Entwurfsfaktoren, die Ihnen möglicherweise nicht bekannt sind.

In diesem Dokument sollen alle Fragen zusammengefasst werden, die bei der Implementierung von Hybrid-Apps aufkommen können. Außerdem werden bestimmt Faktoren (die Säulen) und bewährte Methoden für die Arbeit mit diesen erläutert. Wenn Sie diese Fragen bereits in der Entwurfsphase angehen, vermeiden Sie Probleme, die ansonsten in der Produktion auftreten könnten.

Im Wesentlichen handelt es sich hierbei um Fragen, die Sie vor dem Erstellen einer Hybrid-App beantworten sollten. Sie müssen Folgendes tun, bevor Sie beginnen:

- Identifizieren Sie die App-Komponenten, und werten Sie sie aus.
- Werten Sie die App-Komponenten anhand der Säulen aus.

## <a name="evaluate-the-app-components"></a>Auswerten der App-Komponenten

Jede einzelne App-Komponente hat ihre eine eigene spezifische Rolle innerhalb der größeren App und sollte auf alle Entwurfsfaktoren geprüft werden. Die Anforderungen und Features der einzelnen Komponenten sollten diesen Faktoren zugeordnet werden, um die App-Architektur leichter bestimmen zu können.

Zerlegen Sie Ihre App in ihre Komponenten, indem Sie ihre Architektur untersuchen und ermitteln, wie diese aufgebaut ist. Auch andere Apps, mit denen die App interagiert, können Komponenten sein. Bewerten Sie bei der Identifizierung der Komponenten die vorgesehenen Hybridvorgänge entsprechend ihren Merkmalen, indem Sie die folgenden Fragen stellen:

- Welchem Zweck erfüllt diese Komponente?
- Welche Abhängigkeiten bestehen zwischen den Komponenten?

Beispielsweise können Front-End und Back-End einer App als zwei Komponenten definiert sein. In einem Hybridszenario befindet sich das Front-End in einer Cloud und das Back-End in einer anderen. Die App bietet Kommunikationskanäle zwischen dem Front-End und dem Benutzer sowie zwischen dem Front-End und dem Back-End.

Eine App-Komponente kann in verschiedenen Formen und Szenarien auftreten. Die wichtigste Aufgabe besteht darin, die Komponenten mitsamt ihrem Speicherort (in der Cloud oder lokal) zu identifizieren.

Allgemeine App-Komponenten, die in Ihren Bestand aufgenommen werden sollen, sind in Tabelle 1 aufgeführt.

### <a name="table-1-common-app-components"></a>Tabelle 1. Allgemeine App-Komponenten

| **Komponente** | **Leitfaden für Hybrid-Apps** |
| ---- | ---- |
| Clientverbindungen | Ihre App kann (auf jedem Gerät) auf verschiedene Weise über einen einzelnen Einstiegspunkt auf Benutzer zugreifen, z. B. über Folgendes:<br>– Ein Client-Server-Modell, das erfordert, dass der Benutzer einen Client installiert hat, damit die App funktioniert. Eine serverbasierte App, auf die über einen Browser zugegriffen wird<br>– Clientverbindungen können Benachrichtigungen, wenn die Verbindung getrennt wird, oder Warnungen umfassen, wenn Roaminggebühren anfallen könnten. |
| Authentifizierung  | Die Authentifizierung kann erforderlich sein, damit ein Benutzer eine Verbindung mit der App herstellen kann oder sich zwei Komponenten miteinander verbinden können. |
| APIs  | Sie können Entwicklern über API-Sätze und Klassenbibliotheken programmgesteuerten Zugriff auf Ihre App erteilen und eine Verbindungsschnittstelle basierend auf Internetstandards bereitstellen. Außerdem können Sie auch APIs verwenden, um eine App in unabhängige logische Einheiten zu zerlegen. |
| Dienste  | Sie können prägnante Dienste verwenden, um die Features für eine App bereitzustellen. Mit „Dienst“ kann die Engine gemeint sein, auf der die App ausgeführt wird. |
| Warteschlangen | Sie können Warteschlangen verwenden, um den Status der Lebenszyklen und die Zustände der Komponenten Ihrer App zu sortieren. Diese Warteschlangen können für abonnierende Parteien Messaging-, Benachrichtigungs- und Pufferfunktionen bereitstellen. |
| Datenspeicher | Eine App kann entweder zustandslos oder zustandsbehaftet sein. Zustandsbehaftete Apps benötigen Datenspeicher, die mit zahlreichen Formaten und Volumes kompatibel sein müssen. |
| Datenzwischenspeicherung  | Eine Komponente zum Zwischenspeichern von Daten in Ihrem Entwurf kann Probleme mit der Latenz auf strategische Weise angehen und Cloudbursting unterstützen. |
| Datenerfassung | Daten können auf die unterschiedlichsten Weisen an eine App übermittelt werden, angefangen bei vom Benutzer in einem Webformular übermittelten Werten bis hin zu Datenflüssen mit gleichbleibend hohem Volumen. |
| Datenverarbeitung | Ihre Datenverarbeitungstasks (z. B. Berichte, Analysen, Batchexporte und Datentransformationen) können entweder an der Quelle verarbeitet oder auf einer separaten Komponente durch eine Kopie der Daten ausgelagert werden. |

## <a name="assess-app-components-for-pillars"></a>Bewerten von App-Komponenten für Säulen

Überprüfen Sie jede Komponente auf die Merkmale der einzelnen Säulen. Wenn Sie jede Komponente mit allen Säulen abgleichen, werden möglicherweise Fragen beantwortet, die Sie sich vorher zwar nicht gestellt haben, die aber den Entwurf der Hybrid-Apps beeinflussen. Wenn Sie diese Faktoren berücksichtigen, können Sie Ihre App entsprechend optimieren. Tabelle 2 enthält eine Beschreibung der einzelnen Säulen in Bezug auf Hybrid-Apps.

### <a name="table-2-pillars"></a>Tabelle 2: Säulen

| **Säule** | **Beschreibung** |
| ----------- | --------------------------------------------------------- |
| Platzierung  | Die strategische Positionierung von Komponenten in Hybrid-Apps |
| Skalierbarkeit  | Die Fähigkeit eines Systems, eine höhere Last zu verarbeiten. |
| Verfügbarkeit  | Die Zeit, in der eine Hybrid-App funktioniert und erreichbar ist |
| Resilienz | Die Fähigkeit einer Hybrid-App, sich wiederherzustellen |
| Verwaltbarkeit | Operative Prozesse, die für die Ausführung eines Systems in der Produktion sorgen. |
| Sicherheit | Schützen von Hybrid-Apps und Daten vor Bedrohungen |

## <a name="placement"></a>Platzierung

Es muss grundsätzlich die Platzierung der Hybrid-App berücksichtigt werden, z. B. für das Rechenzentrum.

Die Platzierung ist eine wichtige Aufgabe der Positionierungskomponenten für die optimale Bereitstellung einer Hybrid-App. Der Definition nach erstrecken sich Hybrid-Apps über mehrere Speicherorte, z. B. einen lokalen Speicher und eine Cloud oder mehrere Clouds. Es gibt zwei Möglichkeiten, Komponenten der App in Clouds zu platzieren:

- **Vertikale Hybrid-Apps**  
    App-Komponenten werden auf mehrere Speicherorte verteilt. Jede einzelne Komponente kann über mehrere Instanzen verfügen, die sich nur an einem einzigen Speicherort befinden.

- **Horizontale Hybrid-Apps**  
    App-Komponenten werden auf mehrere Speicherorte verteilt. Jede einzelne Komponente kann über mehrere Instanzen verfügen, die sich auf mehrere Speicherorte erstrecken.

    Einige Komponenten kennen ihren Speicherort, während andere keine Informationen über ihren Speicherort und ihre Platzierung haben. Dieses Verhalten kann über eine Abstraktionsebene erreicht werden. Diese Ebene umfasst ein modernes App-Framework wie Microservices und kann definieren, wie die App durch die Platzierung von App-Komponenten bedient wird, die auf Knoten in den verschiedenen Clouds ausgeführt werden.

### <a name="placement-checklist"></a>Checkliste für die Platzierung

**Überprüfen Sie die erforderlichen Speicherorte.** Stellen Sie sicher, dass die App oder eine der zugehörigen Komponenten in einer bestimmten Cloud ausgeführt werden muss oder eine Zertifizierung erforderlich ist. Dies kann die von Ihrem Unternehmen oder vom Gesetzgeber vorgeschriebenen Anforderungen an die Datenhoheit umfassen. Überprüfen Sie außerdem, ob für einen bestimmten Standort oder ein bestimmtes Gebietsschema lokale Vorgänge erforderlich sind.

**Bestimmen Sie die Konnektivitätsabhängigkeiten.** Die erforderlichen Speicherorte sowie andere Faktoren können die Konnektivitätsabhängigkeiten zwischen den einzelnen Komponenten vorgeben. Bestimmen Sie bei der Platzierung der Komponenten die optimale Konnektivität und Sicherheit für die Kommunikation zwischen diesen. Es gibt die folgenden Optionen: [*VPN*,](/azure/vpn-gateway/) [*ExpressRoute*](/azure/expressroute/) und [*Hybrid Connections*](/azure/app-service/app-service-hybrid-connections).

**Bewerten Sie die Plattformfunktionen.** Überprüfen Sie für jede App-Komponente, ob der jeweils erforderliche Ressourcenanbieter in der Cloud verfügbar ist und ob die Bandbreite den erwarteten Durchsatz- und Latenzanforderungen gerecht werden kann.

**Berücksichtigen Sie die Portabilität.** Verwenden Sie moderne App-Frameworks wie Container oder Microservices, um Verschiebungsvorgänge zu planen und Dienstabhängigkeiten zu vermeiden.

**Bestimmen Sie Anforderungen an die Datenhoheit.** Hybrid-Apps sind darauf ausgerichtet, Datenisolation zu berücksichtigen, z. B. in einem lokalen Rechenzentrum. Überprüfen Sie die Platzierung Ihrer Ressourcen, um den Erfolg bei der Erfüllung dieser Anforderung zu optimieren.

**Berücksichtigen Sie die Latenz.** Cloudübergreifende Vorgänge können zur Folge haben, dass App-Komponenten physisch voneinander getrennt werden. Bestimmen Sie die Anforderungen, die für Latenz gelten.

**Steuern Sie den Datenverkehrsfluss.** Bewältigen Sie Zeiten mit Spitzenauslastungen und der entsprechenden gesicherten Kommunikation für personenbezogene Daten, die über das Front-End in der Public Cloud aufgerufen werden.

## <a name="scalability"></a>Skalierbarkeit

Skalierbarkeit ist die Fähigkeit eines Systems, die höhere Auslastung einer App zu verarbeiten, die im Laufe der Zeit variieren kann, wenn neben der Größe und dem Umfang der App andere Faktoren die Zielgruppengröße zusätzlich beeinflussen.

Diese Säule wird ausführlich im Artikel „Azure Architecture Framework“ unter [*Skalierbarkeit*](/azure/architecture/guide/pillars#scalability) erörtert.

Der horizontale Skalierungsansatz für Hybrid-Apps ermöglicht es Ihnen, weitere Instanzen hinzufügen, um den Bedarf zu decken, und sie dann in ruhigeren Zeiträumen zu deaktivieren.

In Hybridszenarios müssen bei der horizontalen Skalierung einzelner Komponenten zusätzliche Aspekte berücksichtigt werden, wenn diese auf mehrere Clouds verteilt sind. Wenn ein Teil der App skaliert wird, muss ggf. auch ein weiterer Teil skaliert werden. Wenn beispielsweise die Anzahl von Clientverbindungen steigt, aber die Webdienste der App nicht entsprechend aufskaliert werden, kann die App durch die Auslastung der Datenbank vollständig ausgelastet werden.

Einige App-Komponenten können linear aufskaliert werden, während bei anderen Skalierungsabhängigkeiten bestehen und die Skalierfähigkeit möglicherweise eingeschränkt ist. Beispielsweise gelten für einen VPN-Tunnel, der Hybridkonnektivität für die Speicherorte der App-Komponenten bereitstellt, Einschränkungen für die Bandbreite und Latenz der Skalierung. Wie werden die Komponenten der App skaliert, um sicherzustellen, dass diese Anforderungen erfüllt werden?

### <a name="scalability-checklist"></a>Checkliste für die Skalierbarkeit

**Bestimmen Sie Schwellenwerte für die Skalierung.** Zur Verarbeitung der verschiedenen Abhängigkeiten in Ihrer App können Sie festlegen, in welchem Ausmaß App-Komponenten in verschiedenen Clouds unabhängig voneinander skaliert werden können, während die Anforderungen zum Ausführen der App weiterhin erfüllt werden. Hybrid-Apps müssen häufig bestimmte Bereiche in der App skalieren, um ein Feature zu verarbeiten, während es interagiert und den Rest der App beeinflusst. Wenn beispielsweise eine bestimmte Anzahl von Front-End-Instanzen überschritten wird, muss das Back-End möglicherweise skaliert werden.

**Erstellen Sie einen Zeitplan für die Skalierung.** Bei den meisten Apps kann es zu Zeiten mit hoher Auslastung kommen. Daher sollten Sie die Spitzenzeiten in Zeitplänen erfassen, um die Skalierung bestmöglich zu koordinieren.

**Verwenden Sie ein zentralisiertes Überwachungssystem.** Funktionen zur Plattformüberwachung können die automatische Skalierung umfassen. Hybrid-Apps benötigen jedoch ein zentralisiertes Überwachungssystem, das die Systemintegrität und -auslastung aggregiert. Ein zentralisiertes Überwachungssystem sorgt dafür, dass an einem Speicherort mit der Skalierung einer Ressource begonnen wird, während eine abhängige Ressource an einem anderen Speicherort ebenfalls skaliert wird. Außerdem kann bei einem solchen System nachverfolgt werden, welche Clouds Ressourcen automatisch skalieren und welche nicht.

**Nutzen Sie die Funktionen für die automatische Skalierung (sofern verfügbar).** Wenn Ihre Architektur Funktionen zur automatischen Skalierung umfasst, können Sie diese implementieren, indem Sie Schwellenwerte festlegen, die definieren, wann eine App-Komponente hoch-, herunter- oder abskaliert werden muss. Es kommt z. B. bei einer Clientverbindung zur automatischen Skalierung in einer Cloud, um Kapazitätsanstiege zu verarbeiten. Dies hat aber zur Folge, dass andere Abhängigkeiten der App, die auf verschiedene Clouds verteilt sind, ebenfalls skaliert werden. Diese abhängigen Komponenten müssen auf die Funktionen zur automatischen Skalierung geprüft werden.

Wenn die automatische Skalierung nicht verfügbar ist, sollten Sie Skripts und andere Ressourcen implementieren, um die manuelle Skalierung zu ermöglichen, die durch Schwellenwerte im zentralisierten Überwachungssystem ausgelöst wird.

**Bestimmen Sie die erwartete Auslastung je Speicherort.** Hybrid-Apps, die Clientanforderungen verarbeiten, sind möglicherweise von einem einzigen Speicherort abhängig. Wenn die Auslastung durch Clientanforderungen einen bestimmten Schwellenwert überschreitet, können zusätzliche Ressourcen an einem anderen Speicherort hinzugefügt werden, um die Auslastung durch eingehende Anforderungen besser zu verteilen. Vergewissern Sie sich, dass die Clientverbindungen die erhöhte Auslastung verarbeiten können, und bestimmen Sie dafür außerdem alle automatisierten Prozeduren.

## <a name="availability"></a>Verfügbarkeit

Die Verfügbarkeit ist die Zeit, in der ein System funktioniert und erreichbar ist. Sie wird üblicherweise gemessen, indem der prozentuale Anteil der Betriebszeit bestimmt wird. App-Fehler, Infrastrukturprobleme und Systemauslastung können die Verfügbarkeit reduzieren.

Diese Säule wird ausführlich im Artikel „Azure Architecture Framework“ unter [*Verfügbarkeit*](/azure/architecture/framework/) erörtert.

### <a name="availability-checklist"></a>Checkliste für die Verfügbarkeit

**Sorgen Sie für Konnektivitätsredundanz.** Hybrid-Apps erfordern Konnektivität zwischen den verschiedenen Clouds, auf die die App verteilt ist. Es stehen Ihnen verschiedene Technologien für die Bereitstellung von Hybridkonnektivität zur Verfügung, sodass Sie zusätzlich zu Ihrer primären Technologie auch eine andere Technologie verwenden können, um Redundanz mithilfe von automatisierten Failoverfunktionen zu gewährleisten, falls die primäre Technologie ausfallen sollte.

**Klassifizieren Sie Fehlerdomänen.** Fehlertolerante Apps erfordern mehrere Fehlerdomänen. Mithilfe von Fehlerdomänen kann die Fehlerquelle isoliert werden, z. B. bei einem lokalen Ausfall einer einzelnen Festplatte, eines Top-of-Rack-Switches oder eines ganzen Rechenzentrums. In einer Hybrid-App können Speicherorte als Fehlerdomänen klassifiziert werden. Je höher die Verfügbarkeitsanforderungen sind, desto eindeutiger müssen Sie bewerten, wie eine einzelne Fehlerdomäne klassifiziert werden soll.

**Klassifizieren Sie Upgradedomänen.** Upgradedomänen werden verwendet, um sicherzustellen, dass bestimmte Instanzen von App-Komponenten verfügbar sind, während Updates oder Featureupgrades für andere Instanzen derselben Komponente ausgeführt werden. Genauso wie Fehlerdomänen können Upgradedomänen anhand ihrer Platzierung an den verschiedenen Speicherorten klassifiziert werden. Sie müssen bestimmen, ob das Upgrade für eine App-Komponente an einem Speicherort vor dem Upgrade an einem anderen Speicherort ausgeführt werden kann oder ob andere Domänenkonfigurationen erforderlich sind. Ein einzelner Speicherort kann über mehrere Upgradedomänen verfügen.

**Überwachen Sie die Instanzen und die Verfügbarkeit.** App-Komponenten mit Hochverfügbarkeit können über einen Lastenausgleich und synchrone Datenreplikation zur Verfügung gestellt werden. Sie müssen bestimmen, wie viele Instanzen offline sein können, bevor der Dienst unterbrochen wird.

**Implementieren Sie Funktionen zur Selbstreparatur.** Wenn ein Problem zu Einschränkungen der App-Verfügbarkeit führt, könnte eine in das Überwachungssystem integrierte Erkennungsfunktion Aktivitäten zur Selbstreparatur einleiten. Beispielsweise könnte die fehlerhafte Instanz entfernt und noch mal bereitgestellt werden. Höchstwahrscheinlich ist hierfür eine zentrale Überwachungslösung erforderlich, die in eine hybride Pipeline für Continuous Integration und Continuous Delivery (CI/CD) integriert ist. Die App ist in ein Überwachungssystem integriert, damit Probleme identifiziert werden können, die eine nochmalige Bereitstellung einer App-Komponente erfordern könnten. Das Überwachungssystem kann auch eine hybride CI/CD-Lösung zum nochmaligen Bereitstellen der App-Komponente und potenziell anderer abhängiger Komponenten am selben Speicherort oder an anderen Speicherorten auslösen.

**Verwalten Sie Vereinbarungen zum Servicelevel (Service-level agreement).** Die Verfügbarkeit ist für alle Vereinbarungen wichtig, um die Konnektivität mit den Diensten und Apps zu gewährleisten, die Sie mit Ihren Kunden vereinbart haben. Alle Speicherorte, von denen Ihre Hybrid-App abhängig ist, verfügen möglicherweise über eine eigene SLA. Diese unterschiedlichen SLAs können sich auf die allgemeine SLA Ihrer Hybrid-App auswirken.

## <a name="resiliency"></a>Resilienz

Resilienz ist die Fähigkeit einer Hybrid-App und eines Systems, nach Ausfällen eine Wiederherstellung durchzuführen und die Betriebsbereitschaft sicherzustellen. Das Ziel der Resilienz besteht darin, nach einem Ausfall wieder die volle Funktionsfähigkeit einer App herzustellen. Resilienzstrategien beinhalten Lösungen wie Sicherungen, Replikationen und Notfallwiederherstellung.

Diese Säule wird ausführlich im Artikel „Azure Architecture Framework“ unter [*Resilienz*](/azure/architecture/guide/pillars#resiliency) erörtert.

### <a name="resiliency-checklist"></a>Checkliste für Resilienz

**Legen Sie die Abhängigkeiten für die Notfallwiederherstellung offen.** Die Notfallwiederherstellung in einer Cloud erfordert möglicherweise Änderungen von App-Komponenten in einer anderen Cloud. Wenn für mindestens eine Komponente aus einer Cloud ein Failover auf einen anderen Speicherort (entweder innerhalb derselben Cloud oder in einer anderen Cloud) durchgeführt wird, müssen die abhängigen Komponenten diese Änderungen berücksichtigen. Dies umfasst auch die Konnektivitätsabhängigkeiten. Damit Resilienz erreicht wird, ist für jede Cloud ein vollständig getesteter Wiederherstellungsplan für Apps erforderlich.

**Erarbeiten Sie einen Wiederherstellungsworkflow.** Ein Entwurf für eine effektive Wiederherstellung umfasst App-Komponenten, die im Hinblick auf ihre Fähigkeit ausgewertet werden, Puffer, Wiederholungsversuche, Wiederholungsversuche bei Fehlern bei der Datenübertragung zu integrieren und ggf. auf einen anderen Dienst oder Workflow zurückzugreifen. Sie müssen festlegen, welche Sicherungsmechanismen verwendet werden, wie die Wiederherstellungsprozedur aufgebaut ist und wie oft diese getestet wird. Sie sollten auch die Häufigkeit inkrementeller und vollständiger Sicherungen bestimmen.

**Testen Sie Teilwiederherstellungen.** Durch eine Teilwiederherstellung bestimmter App-Komponenten wird den Benutzern garantiert, dass Teile der Anwendung verfügbar bleiben. Dieser Teil des Plans sollte sicherstellen, dass eine Teilwiederherstellung keine Nebenwirkungen hat, z. B. kann ein Sicherungs- und Wiederherstellungsdienst integriert werden, der mit der App interagiert, um diese ordnungsgemäß herunterzufahren, bevor die Sicherung durchgeführt wird.

**Bestimmen Sie Initiatoren für die Notfallwiederherstellung, und weisen Sie Verantwortungsbereiche zu.** In einem Wiederherstellungsplan sollte neben den Elementen, die gesichert und wieder hergestellt werden können, beschrieben sein, welche Personen und Rollen Sicherungs- und Wiederherstellungsaktionen initiieren können.

**Vergleichen Sie Schwellenwerte für die Selbstreparatur mit der Notfallwiederherstellung.** Bestimmen Sie die Funktionen für die Selbstreparatur einer App zur Initiierung der automatischen Wiederherstellung sowie die Zeit, die für die Selbstreparatur einer App erforderlich ist, damit sie entweder als fehlerhaft oder als erfolgreich gilt. Bestimmen Sie die Schwellenwerte für die einzelnen Clouds.

**Überprüfen Sie die Verfügbarkeit von Resilienzfeatures.** Bestimmen Sie die Verfügbarkeit von Resilienzfeatures und -funktionen für die einzelnen Speicherorte. Wenn ein Speicherort nicht die erforderlichen Funktionen bereitstellt, sollten Sie diesen ggf. in einen zentralisierten Dienst integrieren, der diese Funktionen umfasst.

**Legen Sie Downtimes fest.** Bestimmen Sie die erwartete Downtime aufgrund von Wartungen der gesamten App und von App-Komponenten.

**Dokumentieren Sie Verfahren zur Problembehandlung.** Definieren Sie Problembehandlungsverfahren für die nochmalige Bereitstellung von Ressourcen und App-Komponenten.

## <a name="manageability"></a>Verwaltbarkeit

Die hier beschriebenen Faktoren zur Verwaltung Ihrer Hybrid-Apps sind wichtig beim Entwerfen Ihrer Architektur. Eine gut verwaltete Hybrid-App bietet eine Infrastruktur in Form von Code, der die Integration von konsistentem App-Code in eine allgemeine Entwicklungspipeline ermöglicht. Durch die Implementierung von konsistenten systemweiten und individuellen Tests von Änderungen an der Infrastruktur können Sie eine integrierte Bereitstellung gewährleisten, wenn die Änderungen die Tests bestehen, sodass sie mit dem Quellcode zusammengeführt werden können.

Diese Säule wird ausführlich im Artikel „Azure Architecture Framework“ unter [*DevOps*](/azure/architecture/framework/#devops) erörtert.

### <a name="manageability-checklist"></a>Checkliste zur Verwaltbarkeit

**Implementieren Sie Überwachungsfunktionen.** Verwenden Sie ein zentralisiertes Überwachungssystem für App-Komponenten, die auf mehrere Clouds verteilt sind, um eine aggregierte Ansicht zur Integrität und Leistung bereitzustellen. Dieses System umfasst die Überwachung der App-Komponenten einerseits und der zugehörigen Plattformfunktionen andererseits.

Bestimmen Sie, welche App-Komponenten überwacht werden müssen.

**Koordinieren Sie Richtlinien.** Sämtliche Speicherorte einer Hybrid-App können über eine eigene Richtlinie verfügen, die zulässige Ressourcentypen, Benennungskonventionen, Tags und andere Kriterien betreffen.

**Definieren Sie Rollen, und verwenden Sie sie.** Als Datenbankadministrator müssen Sie die erforderlichen Berechtigungen für verschiedene Personas (z. B. einen App-Besitzer, einen Datenbankadministrator und einen Endbenutzer) festlegen, die Zugriff auf App-Ressourcen benötigen. Diese Berechtigungen müssen für die Ressourcen und innerhalb der App konfiguriert werden. Über ein RBAC-System (Role-Based Access Control, rollenbasierte Zugriffssteuerung) können Sie diese Berechtigungen für die App-Ressourcen festlegen. Diese Zugriffsrechte stellen schon eine besondere Herausforderung dar, wenn alle Ressourcen in einer einzigen Cloud bereitgestellt werden, erfordern aber noch mehr Aufmerksamkeit, wenn die Ressourcen auf verschiedene Clouds verteilt sind. Berechtigungen für Ressourcen in einer Cloud gelten nicht für Ressourcen in einer anderen Cloud.

**Verwenden Sie CI/CD-Pipelines.** Eine CI/CD-Pipeline (Continuous Integration and Continuous Development) kann einen konsistenten Prozess zum Erstellen und Bereitstellen von Apps, die sich über mehrere Clouds erstrecken, sowie die Qualitätssicherung für deren Infrastruktur und App gewährleisten. Mit dieser Pipeline können die Infrastruktur und die App in einer Cloud getestet und in einer anderen bereitgestellt werden. Sie macht es Ihnen sogar möglich, bestimmte Komponenten Ihrer Hybrid-App in einer Cloud und andere Komponenten in einer anderen Cloud bereitzustellen, und stellt somit im Wesentlichen die Grundlage für die Bereitstellung von Hybrid-Apps dar. Ein CI/CD-System ist wichtig für die Verarbeitung der Abhängigkeiten von App-Komponenten während der Installation, z. B. bei einer Web-App, die eine Verbindungszeichenfolge für die Datenbank benötigt.

**Verwalten Sie den Lebenszyklus.** Da sich die Ressourcen einer Hybrid-App über mehrere Speicherorte erstrecken können, muss die Fähigkeit zum Verwalten des Lebenszyklus jedes einzelnen Speicherorts in eine einzige Einheit zum Verwalten des Lebenszyklus aggregiert werden. Berücksichtigen Sie dabei, wie diese erstellt, aktualisiert und gelöscht werden.

**Überprüfen Sie Strategien für die Problembehandlung.** Die Problembehandlung bei einer Hybrid-App umfasst mehr App-Komponenten als bei Apps, die in einer einzigen Cloud ausgeführt werden. Neben der Konnektivität zwischen den Clouds wird die App auf zwei Plattformen anstatt auf einer ausgeführt. Eine wichtige Aufgabe bei der Behandlung von Problemen von Hybrid-Apps besteht darin, die aggregierte Integritäts- und Leistungsüberwachung der App-Komponenten zu untersuchen.

## <a name="security"></a>Sicherheit

Die Sicherheit ist einer der wichtigsten Faktoren aller Cloud-Apps und gewinnt im Zusammenhang mit Hybrid Cloud-Apps noch mehr an Bedeutung.

Diese Säule wird ausführlich im Artikel „Azure Architecture Framework“ unter [*Sicherheit*](/azure/architecture/guide/pillars#security) erörtert.

### <a name="security-checklist"></a>Checkliste für die Sicherheit

**Gehen Sie von einer Sicherheitsverletzung aus.** Wenn eine Komponente der App kompromittiert ist, sollte es nicht nur für den betreffenden Speicherort, sondern für alle Speicherorte Lösungen geben, um den Schaden zu begrenzen.

**Überwachen Sie zulässigen Netzwerkzugriff.** Bestimmen Sie die Netzwerkzugriffsrichtlinien für die App. Sie können beispielsweise festlegen, dass nur über ein bestimmtes Subnetz auf die App zugegriffen werden kann, und nur die Mindestanzahl an Ports und Protokollen zwischen den Komponenten zulassen, die benötigt werden, damit die App richtig funktionieren kann.

**Stellen Sie eine stabile Option zur Authentifizierung bereit.** Ein robustes Authentifizierungsschema ist wichtig für die Sicherheit Ihrer App. Sie sollten einen Verbundidentitätsanbieter verwenden, der Funktionen für einmaliges Anmelden anbietet und mindestens eins der folgenden Schemas anwendet: Anmeldung über Benutzernamen und Kennwort, öffentliche und private Schlüssel, zweistufige oder mehrstufige Authentifizierung und vertrauenswürdige Sicherheitsgruppen. Bestimmen Sie neben den Zertifikattypen und deren Anforderungen auch geeignete Ressourcen zum Speichern sensibler Daten und anderer Geheimnisse für die App-Authentifizierung.

**Verwenden Sie Verschlüsselungsfunktionen.** Ermitteln Sie, welche Bereiche der App verschlüsselt werden, z. B. für die Datenspeicherung oder Clientkommunikation und den Zugriff.

**Verwenden Sie sichere Kanäle.** Ein sicherer Kanal für alle Clouds ist wichtig für die Bereitstellung von Sicherheits- und Authentifizierungsüberprüfungen, Echtzeitschutz, Quarantäne und anderen Diensten der Clouds.

**Definieren Sie Rollen, und verwenden Sie sie.** Implementieren Sie Rollen für Ressourcenkonfigurationen und den Zugriff auf einzelne Identitäten über mehrere Clouds hinweg. Legen Sie die Anforderungen an die rollenbasierten Zugriffssteuerung für die App und ihre Plattformressourcen fest.

**Überprüfen Sie Ihr System.** Über die Systemüberwachung können sowohl Daten der App-Komponenten als auch Daten der zugehörigen Cloudplattformvorgänge protokolliert und aggregiert werden.

## <a name="summary"></a>Zusammenfassung

Dieser Artikel enthält eine Checkliste mit den Punkten, die beim Erstellen und Entwerfen von Hybrid-Apps zu beachten sind. Wenn Sie diese Säulen überprüfen, bevor Sie Ihre App bereitstellen, vermeiden Sie, dass diese Faktoren zu Produktionsausfällen führen und Sie Ihren Entwurf möglicherweise überarbeiten müssen.

Das ist zwar zunächst sehr aufwendig, aber Sie können davon schnell profitieren, wenn Sie Ihre App basierend auf diesen Säulen entwerfen.

## <a name="next-steps"></a>Nächste Schritte

Weitere Informationen finden Sie in den folgenden Ressourcen:

- [Hybrid Cloud](https://azure.microsoft.com/overview/hybrid-cloud/)
- [Hybrid Cloud-Apps](https://azure.microsoft.com/solutions/hybrid-cloud-app/)
- [Informationen zum Entwickeln von Azure Resource Manager-Vorlagen für cloudübergreifende Konsistenz](https://aka.ms/consistency)
