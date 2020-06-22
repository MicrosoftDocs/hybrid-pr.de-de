---
title: Muster für geografisch verteilte Apps in Azure Stack Hub
description: Erfahren Sie mehr über das Muster einer geografisch verteilten App für Intelligent Edge unter Verwendung von Azure und Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod2019
ms.openlocfilehash: 1f6243927390c7a520c2607c722664b2d31fc07f
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910321"
---
# <a name="geo-distributed-app-pattern"></a><span data-ttu-id="b04d7-103">Muster für geografisch verteilte Apps</span><span class="sxs-lookup"><span data-stu-id="b04d7-103">Geo-distributed app pattern</span></span>

<span data-ttu-id="b04d7-104">Hier erfahren Sie, wie Sie App-Endpunkte in mehreren Regionen bereitstellen und Benutzerdatenverkehr je nach Standort- und Complianceanforderungen weiterleiten.</span><span class="sxs-lookup"><span data-stu-id="b04d7-104">Learn how to provide app endpoints across multiple regions and route user traffic based on location and compliance needs.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="b04d7-105">Kontext und Problem</span><span class="sxs-lookup"><span data-stu-id="b04d7-105">Context and problem</span></span>

<span data-ttu-id="b04d7-106">Organisationen mit umfassenden geografischen Ausmaßen bemühen sich um eine sichere und fehlerfreie Verteilung von und den Zugriff auf diese Daten, während das erforderliche Maß an Sicherheit, Compliance und Leistung pro Benutzer, Standort und Gerät über Grenzen hinweg auch weiterhin sichergestellt werden soll.</span><span class="sxs-lookup"><span data-stu-id="b04d7-106">Organizations with wide-reaching geographies strive to securely and accurately distribute and enable access to data while ensuring required levels of security, compliance and performance per user, location, and device across borders.</span></span>

## <a name="solution"></a><span data-ttu-id="b04d7-107">Lösung</span><span class="sxs-lookup"><span data-stu-id="b04d7-107">Solution</span></span>

<span data-ttu-id="b04d7-108">Durch das Muster für das geografische Datenverkehrsrouting von Azure Stack Hub oder die Verwendung geografisch verteilter Apps kann Datenverkehr basierend auf verschiedenen Metriken an bestimmte Endpunkte weitergeleitet werden.</span><span class="sxs-lookup"><span data-stu-id="b04d7-108">The Azure Stack Hub geographic traffic routing pattern, or geo-distributed apps, lets traffic be directed to specific endpoints based on various metrics.</span></span> <span data-ttu-id="b04d7-109">Durch das Erstellen von Traffic Manager mit geografiebasiertem Routing und der Endpunktkonfiguration wird Datenverkehr basierend auf regionalen Anforderungen bzw. unternehmensinternen und internationalen Bestimmungen an Endpunkte weitergeleitet.</span><span class="sxs-lookup"><span data-stu-id="b04d7-109">Creating a Traffic Manager with geographic-based routing and endpoint configuration routes traffic to endpoints based on regional requirements, corporate and international regulation, and data needs.</span></span>

![Geografisch verteiltes Muster](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a><span data-ttu-id="b04d7-111">Komponenten</span><span class="sxs-lookup"><span data-stu-id="b04d7-111">Components</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="b04d7-112">Außerhalb der Cloud</span><span class="sxs-lookup"><span data-stu-id="b04d7-112">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="b04d7-113">Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="b04d7-113">Traffic Manager</span></span>

<span data-ttu-id="b04d7-114">In der Abbildung befindet sich Traffic Manager außerhalb der öffentlichen Cloud, muss allerdings sowohl den Datenverkehr im lokalen Rechenzentrum als auch in der öffentlichen Cloud koordinieren können.</span><span class="sxs-lookup"><span data-stu-id="b04d7-114">In the diagram, Traffic Manager is located outside of the public cloud, but it needs to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="b04d7-115">Der Balancer leitet Datenverkehr an geografische Standorte weiter.</span><span class="sxs-lookup"><span data-stu-id="b04d7-115">The balancer routes traffic to geographical locations.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="b04d7-116">Domain Name System (DNS)</span><span class="sxs-lookup"><span data-stu-id="b04d7-116">Domain Name System (DNS)</span></span>

<span data-ttu-id="b04d7-117">Das Domain Name System (DNS) ist für die Übersetzung (oder Auflösung) eines Website- oder Dienstnamens in die IP-Adresse verantwortlich.</span><span class="sxs-lookup"><span data-stu-id="b04d7-117">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="public-cloud"></a><span data-ttu-id="b04d7-118">Öffentliche Cloud</span><span class="sxs-lookup"><span data-stu-id="b04d7-118">Public cloud</span></span>

#### <a name="cloud-endpoint"></a><span data-ttu-id="b04d7-119">Cloudendpunkt</span><span class="sxs-lookup"><span data-stu-id="b04d7-119">Cloud Endpoint</span></span>

<span data-ttu-id="b04d7-120">Öffentliche IP-Adressen werden zum Weiterleiten des eingehenden Datenverkehrs über Traffic Manager an den Endpunkt der App-Ressourcen in der öffentlichen Cloud verwendet.</span><span class="sxs-lookup"><span data-stu-id="b04d7-120">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-clouds"></a><span data-ttu-id="b04d7-121">Lokale Clouds</span><span class="sxs-lookup"><span data-stu-id="b04d7-121">Local clouds</span></span>

#### <a name="local-endpoint"></a><span data-ttu-id="b04d7-122">Lokaler Endpunkt</span><span class="sxs-lookup"><span data-stu-id="b04d7-122">Local endpoint</span></span>

<span data-ttu-id="b04d7-123">Öffentliche IP-Adressen werden zum Weiterleiten des eingehenden Datenverkehrs über Traffic Manager an den Endpunkt der App-Ressourcen in der öffentlichen Cloud verwendet.</span><span class="sxs-lookup"><span data-stu-id="b04d7-123">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="b04d7-124">Probleme und Überlegungen</span><span class="sxs-lookup"><span data-stu-id="b04d7-124">Issues and considerations</span></span>

<span data-ttu-id="b04d7-125">Beachten Sie die folgenden Punkte bei der Entscheidung, wie dieses Muster implementiert werden soll:</span><span class="sxs-lookup"><span data-stu-id="b04d7-125">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="b04d7-126">Skalierbarkeit</span><span class="sxs-lookup"><span data-stu-id="b04d7-126">Scalability</span></span>

<span data-ttu-id="b04d7-127">Das Muster verarbeitet bei erhöhtem Datenverkehr eher das Routing von geografischem Datenverkehr anstelle der Skalierung.</span><span class="sxs-lookup"><span data-stu-id="b04d7-127">The pattern handles geographical traffic routing rather than scaling to meet increases in traffic.</span></span> <span data-ttu-id="b04d7-128">Sie können dieses Muster mit anderen Azure-Lösungen und lokalen Lösungen kombinieren.</span><span class="sxs-lookup"><span data-stu-id="b04d7-128">However, you can combine this pattern with other Azure and on-premises solutions.</span></span> <span data-ttu-id="b04d7-129">Dieses Muster kann beispielsweise mit dem cloudübergreifenden Skalierungsmuster verwendet werden.</span><span class="sxs-lookup"><span data-stu-id="b04d7-129">For example, this pattern can be used with the cross-cloud scaling Pattern.</span></span>

### <a name="availability"></a><span data-ttu-id="b04d7-130">Verfügbarkeit</span><span class="sxs-lookup"><span data-stu-id="b04d7-130">Availability</span></span>

<span data-ttu-id="b04d7-131">Stellen Sie sicher, dass lokal bereitgestellte Apps in Bezug auf Hochverfügbarkeit konfiguriert sind, die auf der Konfiguration der lokalen Hardware und der Softwarebereitstellung basiert.</span><span class="sxs-lookup"><span data-stu-id="b04d7-131">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="b04d7-132">Verwaltbarkeit</span><span class="sxs-lookup"><span data-stu-id="b04d7-132">Manageability</span></span>

<span data-ttu-id="b04d7-133">Mit dem Muster wird sichergestellt, dass zwischen Umgebungen eine nahtlose Verwaltung möglich und eine vertraute Benutzeroberfläche vorhanden ist.</span><span class="sxs-lookup"><span data-stu-id="b04d7-133">The pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="b04d7-134">Verwendung dieses Musters</span><span class="sxs-lookup"><span data-stu-id="b04d7-134">When to use this pattern</span></span>

- <span data-ttu-id="b04d7-135">Meine Organisation verfügt über internationale Zweigniederlassungen, für die benutzerdefinierte regionale Sicherheits- und Verteilungsrichtlinien erforderlich sind.</span><span class="sxs-lookup"><span data-stu-id="b04d7-135">My organization has international branches requiring custom regional security and distribution policies.</span></span>
- <span data-ttu-id="b04d7-136">Alle Niederlassungen meiner Organisation rufen per Pullvorgang Daten zu Mitarbeitern, Geschäft und Einrichtungen ab, sodass Berichterstellungsaktivitäten gemäß den lokalen Bestimmungen und Zeitzonen benötigt werden.</span><span class="sxs-lookup"><span data-stu-id="b04d7-136">Each of my organization's offices pulls employee, business, and facility data, requiring reporting activity per local regulations and time zone.</span></span>
- <span data-ttu-id="b04d7-137">Anforderungen für eine hohe Skalierbarkeit können durch das horizontale Hochskalieren von Apps erfüllt werden, wobei mehrere App-Bereitstellungen innerhalb einer Region sowie regionsübergreifend erfolgen, um extreme Lastanforderungen zu verarbeiten.</span><span class="sxs-lookup"><span data-stu-id="b04d7-137">High-scale requirements can be met by horizontally scaling out apps, with multiple app deployments being made within a single region and across regions to handle extreme load requirements.</span></span>
- <span data-ttu-id="b04d7-138">Die Apps müssen selbst bei einem Ausfall einer einzelnen Region hochverfügbar sein und auf Clientanforderungen reagieren.</span><span class="sxs-lookup"><span data-stu-id="b04d7-138">The apps must be highly available and responsive to client requests even in single-region outages.</span></span>

## <a name="next-steps"></a><span data-ttu-id="b04d7-139">Nächste Schritte</span><span class="sxs-lookup"><span data-stu-id="b04d7-139">Next steps</span></span>

<span data-ttu-id="b04d7-140">Weitere Informationen zu den in diesem Artikel behandelten Themen:</span><span class="sxs-lookup"><span data-stu-id="b04d7-140">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="b04d7-141">In der [Übersicht über Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) erfahren Sie mehr dazu, wie dieser DNS-basierte Lastenausgleich für Datenverkehr funktioniert.</span><span class="sxs-lookup"><span data-stu-id="b04d7-141">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="b04d7-142">Unter [Überlegungen zum Entwurf von Hybrid-Apps](overview-app-design-considerations.md) erfahren Sie mehr zu bewährten Methoden und erhalten Antworten auf weitere Fragen.</span><span class="sxs-lookup"><span data-stu-id="b04d7-142">See [Hybrid app design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="b04d7-143">Im Artikel zur [Azure Stack-Familie mit Produkten und Lösungen](/azure-stack) erfahren Sie mehr über das gesamte Portfolio von Produkten und Lösungen.</span><span class="sxs-lookup"><span data-stu-id="b04d7-143">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="b04d7-144">Wenn Sie bereit sind, das Lösungsbeispiel zu testen, fahren Sie mit dem [Bereitstellungsleitfaden für eine Lösung für geografisch verteilte Apps](solution-deployment-guide-geo-distributed.md) fort.</span><span class="sxs-lookup"><span data-stu-id="b04d7-144">When you're ready to test the solution example, continue with the [Geo-distributed app solution deployment guide](solution-deployment-guide-geo-distributed.md).</span></span> <span data-ttu-id="b04d7-145">In diesem Bereitstellungsleitfaden finden Sie detaillierte Anweisungen zum Bereitstellen und Testen der zugehörigen Komponenten.</span><span class="sxs-lookup"><span data-stu-id="b04d7-145">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="b04d7-146">Sie erfahren, wie Sie Datenverkehr basierend auf unterschiedlichen Metriken an bestimmte Endpunkte weiterleiten, indem Sie das Muster für geografisch verteilte Apps verwenden.</span><span class="sxs-lookup"><span data-stu-id="b04d7-146">You learn how to direct traffic to specific endpoints, based on various metrics using the geo-distributed app pattern.</span></span> <span data-ttu-id="b04d7-147">Durch die Erstellung eines Traffic Manager-Profils mit Weiterleitung und Endpunktkonfiguration anhand der Geografie wird sichergestellt, dass Informationen basierend auf regionalen Anforderungen, unternehmensinternen und internationalen Bestimmungen und Ihren Datenanforderungen an Endpunkte weitergeleitet werden.</span><span class="sxs-lookup"><span data-stu-id="b04d7-147">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>
