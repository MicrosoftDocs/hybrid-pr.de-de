---
title: Muster für die cloudübergreifende Skalierung in Azure Stack Hub
description: Hier erfahren Sie, wie Sie eine skalierbare, cloudübergreifende App in Azure und Azure Stack Hub erstellen.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: a830f96e97c347cbbcc09a1b17f4836ecb6eb3e6
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910429"
---
# <a name="cross-cloud-scaling-pattern"></a><span data-ttu-id="b43bc-103">Cloudübergreifendes Skalierungsmuster</span><span class="sxs-lookup"><span data-stu-id="b43bc-103">Cross-cloud scaling pattern</span></span>

<span data-ttu-id="b43bc-104">Fügen Sie automatisch Ressourcen zu einer vorhandenen App hinzu, um eine höhere Auslastung zu ermöglichen.</span><span class="sxs-lookup"><span data-stu-id="b43bc-104">Automatically add resources to an existing app to accommodate an increase in load.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="b43bc-105">Kontext und Problem</span><span class="sxs-lookup"><span data-stu-id="b43bc-105">Context and problem</span></span>

<span data-ttu-id="b43bc-106">Ihre App kann die Kapazität zum Erfüllen einer unerwartet erhöhten Nachfrage nicht erhöhen.</span><span class="sxs-lookup"><span data-stu-id="b43bc-106">Your app can't increase capacity to meet unexpected increases in demand.</span></span> <span data-ttu-id="b43bc-107">Dieser Mangel an Skalierbarkeit führt dazu, dass Benutzer die App während Spitzennutzungszeiten nicht verwenden können.</span><span class="sxs-lookup"><span data-stu-id="b43bc-107">This lack of scalability results in users not reaching the app during peak usage times.</span></span> <span data-ttu-id="b43bc-108">Die App kann nur von einer festgelegten Anzahl von Benutzern genutzt werden.</span><span class="sxs-lookup"><span data-stu-id="b43bc-108">The app can service a fixed number of users.</span></span>

<span data-ttu-id="b43bc-109">Globale Unternehmen benötigen sichere, zuverlässige und verfügbare cloudbasierte Apps.</span><span class="sxs-lookup"><span data-stu-id="b43bc-109">Global enterprises require secure, reliable, and available cloud-based apps.</span></span> <span data-ttu-id="b43bc-110">Daher ist es wichtig, höhere Anforderungen zu erfüllen und die richtige Infrastruktur zur Unterstützung dieser Anforderungen zu verwenden.</span><span class="sxs-lookup"><span data-stu-id="b43bc-110">Meeting increases in demand and using the right infrastructure to support that demand is critical.</span></span> <span data-ttu-id="b43bc-111">Unternehmen haben Probleme beim Ausgleichen von Kosten und bei der Verwaltung der Sicherheit, des Speicherns und der Echtzeitverfügbarkeit von Geschäftsdaten.</span><span class="sxs-lookup"><span data-stu-id="b43bc-111">Businesses struggle to balance costs and maintenance with business data security, storage, and real-time availability.</span></span>

<span data-ttu-id="b43bc-112">Sie können Ihre App möglicherweise nicht in der öffentlichen Cloud ausführen.</span><span class="sxs-lookup"><span data-stu-id="b43bc-112">You may not be able to run your app in the public cloud.</span></span> <span data-ttu-id="b43bc-113">Es ist für das Unternehmen aber unter Umständen nicht wirtschaftlich, die für die lokale Umgebung erforderliche Kapazität beizubehalten, um für die App Auslastungsspitzen verarbeiten zu können.</span><span class="sxs-lookup"><span data-stu-id="b43bc-113">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="b43bc-114">Mit diesem Muster können Sie die Elastizität der öffentlichen Cloud für Ihre lokale Lösung verwenden.</span><span class="sxs-lookup"><span data-stu-id="b43bc-114">With this pattern, you can use the elasticity of the public cloud with your on-premises solution.</span></span>

## <a name="solution"></a><span data-ttu-id="b43bc-115">Lösung</span><span class="sxs-lookup"><span data-stu-id="b43bc-115">Solution</span></span>

<span data-ttu-id="b43bc-116">Das cloudübergreifende Skalierungsmuster erweitert eine App, die sich in einer lokalen Cloud befindet, um öffentliche Cloudressourcen.</span><span class="sxs-lookup"><span data-stu-id="b43bc-116">The cross-cloud scaling pattern extends an app located in a local cloud with public cloud resources.</span></span> <span data-ttu-id="b43bc-117">Das Muster wird durch eine Erhöhung oder Abnahme der Nachfrage ausgelöst. Dann werden dementsprechend Ressourcen in der Cloud hinzugefügt oder entfernt.</span><span class="sxs-lookup"><span data-stu-id="b43bc-117">The pattern is triggered by an increase or decrease in demand, and respectively adds or removes resources in the cloud.</span></span> <span data-ttu-id="b43bc-118">Diese Ressourcen bieten Redundanz, schnelle Verfügbarkeit und geokonformes Routing.</span><span class="sxs-lookup"><span data-stu-id="b43bc-118">These resources provide redundancy, rapid availability, and geo-compliant routing.</span></span>

![Cloudübergreifendes Skalierungsmuster](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> <span data-ttu-id="b43bc-120">Dieses Muster gilt nur für zustandslose Komponenten Ihrer App.</span><span class="sxs-lookup"><span data-stu-id="b43bc-120">This pattern applies only to stateless components of your app.</span></span>

## <a name="components"></a><span data-ttu-id="b43bc-121">Komponenten</span><span class="sxs-lookup"><span data-stu-id="b43bc-121">Components</span></span>

<span data-ttu-id="b43bc-122">Das Muster für die cloudübergreifende Skalierung besteht aus den folgenden Komponenten:</span><span class="sxs-lookup"><span data-stu-id="b43bc-122">The cross-cloud scaling pattern consists of the following components.</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="b43bc-123">Außerhalb der Cloud</span><span class="sxs-lookup"><span data-stu-id="b43bc-123">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="b43bc-124">Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="b43bc-124">Traffic Manager</span></span>

<span data-ttu-id="b43bc-125">Im Diagramm befindet sich Traffic Manager außerhalb der öffentlichen Cloudgruppe, muss allerdings sowohl den Datenverkehr im lokalen Rechenzentrum als auch in der öffentlichen Cloud koordinieren können.</span><span class="sxs-lookup"><span data-stu-id="b43bc-125">In the diagram, this is located outside of the public cloud group, but it would need to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="b43bc-126">Das Ausgleichsmodul bietet Hochverfügbarkeit für Apps, indem es Endpunkte überwacht und bei Bedarf die Failoverumverteilung ermöglicht.</span><span class="sxs-lookup"><span data-stu-id="b43bc-126">The balancer delivers high availability for app by monitoring endpoints and providing failover redistribution when required.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="b43bc-127">Domain Name System (DNS)</span><span class="sxs-lookup"><span data-stu-id="b43bc-127">Domain Name System (DNS)</span></span>

<span data-ttu-id="b43bc-128">Das Domain Name System (DNS) ist für die Übersetzung (oder Auflösung) eines Website- oder Dienstnamens in die IP-Adresse verantwortlich.</span><span class="sxs-lookup"><span data-stu-id="b43bc-128">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="cloud"></a><span data-ttu-id="b43bc-129">Cloud</span><span class="sxs-lookup"><span data-stu-id="b43bc-129">Cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="b43bc-130">Gehosteter Buildserver</span><span class="sxs-lookup"><span data-stu-id="b43bc-130">Hosted build server</span></span>

<span data-ttu-id="b43bc-131">Umgebung zum Hosten der Buildpipeline.</span><span class="sxs-lookup"><span data-stu-id="b43bc-131">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="b43bc-132">App-Ressourcen</span><span class="sxs-lookup"><span data-stu-id="b43bc-132">App resources</span></span>

<span data-ttu-id="b43bc-133">Die App-Ressourcen (wie VM-Skalierungsgruppen und Container) müssen ab- und aufskaliert werden können.</span><span class="sxs-lookup"><span data-stu-id="b43bc-133">The app resources need to be able to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="b43bc-134">Benutzerdefinierter Domänenname</span><span class="sxs-lookup"><span data-stu-id="b43bc-134">Custom domain name</span></span>

<span data-ttu-id="b43bc-135">Verwenden Sie einen benutzerdefinierten Domänennamen für globale Routinganforderungen.</span><span class="sxs-lookup"><span data-stu-id="b43bc-135">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="b43bc-136">Öffentliche IP-Adressen</span><span class="sxs-lookup"><span data-stu-id="b43bc-136">Public IP addresses</span></span>

<span data-ttu-id="b43bc-137">Öffentliche IP-Adressen werden zum Weiterleiten des eingehenden Datenverkehrs über Traffic Manager an den Endpunkt der App-Ressourcen in der öffentlichen Cloud verwendet.</span><span class="sxs-lookup"><span data-stu-id="b43bc-137">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-cloud"></a><span data-ttu-id="b43bc-138">Lokale Cloud</span><span class="sxs-lookup"><span data-stu-id="b43bc-138">Local cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="b43bc-139">Gehosteter Buildserver</span><span class="sxs-lookup"><span data-stu-id="b43bc-139">Hosted build server</span></span>

<span data-ttu-id="b43bc-140">Umgebung zum Hosten der Buildpipeline.</span><span class="sxs-lookup"><span data-stu-id="b43bc-140">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="b43bc-141">App-Ressourcen</span><span class="sxs-lookup"><span data-stu-id="b43bc-141">App resources</span></span>

<span data-ttu-id="b43bc-142">Die App-Ressourcen (wie VM-Skalierungsgruppen und Container) müssen ab- und aufskaliert werden können.</span><span class="sxs-lookup"><span data-stu-id="b43bc-142">The app resources need the ability to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="b43bc-143">Benutzerdefinierter Domänenname</span><span class="sxs-lookup"><span data-stu-id="b43bc-143">Custom domain name</span></span>

<span data-ttu-id="b43bc-144">Verwenden Sie einen benutzerdefinierten Domänennamen für globale Routinganforderungen.</span><span class="sxs-lookup"><span data-stu-id="b43bc-144">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="b43bc-145">Öffentliche IP-Adressen</span><span class="sxs-lookup"><span data-stu-id="b43bc-145">Public IP addresses</span></span>

<span data-ttu-id="b43bc-146">Öffentliche IP-Adressen werden zum Weiterleiten des eingehenden Datenverkehrs über Traffic Manager an den Endpunkt der App-Ressourcen in der öffentlichen Cloud verwendet.</span><span class="sxs-lookup"><span data-stu-id="b43bc-146">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="b43bc-147">Probleme und Überlegungen</span><span class="sxs-lookup"><span data-stu-id="b43bc-147">Issues and considerations</span></span>

<span data-ttu-id="b43bc-148">Beachten Sie die folgenden Punkte bei der Entscheidung, wie dieses Muster implementiert werden soll:</span><span class="sxs-lookup"><span data-stu-id="b43bc-148">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="b43bc-149">Skalierbarkeit</span><span class="sxs-lookup"><span data-stu-id="b43bc-149">Scalability</span></span>

<span data-ttu-id="b43bc-150">Das Hauptelement der cloudübergreifenden Skalierung ist die Fähigkeit, Skalierung bedarfsabhängig zu ermöglichen.</span><span class="sxs-lookup"><span data-stu-id="b43bc-150">The key component of cross-cloud scaling is the ability to deliver on-demand scaling.</span></span> <span data-ttu-id="b43bc-151">Die Skalierung muss zwischen öffentlicher und lokaler Cloudinfrastruktur erfolgen und einen einheitlichen, zuverlässigen Dienst abhängig vom Bedarf ermöglichen.</span><span class="sxs-lookup"><span data-stu-id="b43bc-151">Scaling must happen between public and local cloud infrastructure and provide a consistent, reliable service per the demand.</span></span>

### <a name="availability"></a><span data-ttu-id="b43bc-152">Verfügbarkeit</span><span class="sxs-lookup"><span data-stu-id="b43bc-152">Availability</span></span>

<span data-ttu-id="b43bc-153">Stellen Sie sicher, dass lokal bereitgestellte Apps in Bezug auf Hochverfügbarkeit konfiguriert sind, die auf der Konfiguration der lokalen Hardware und der Softwarebereitstellung basiert.</span><span class="sxs-lookup"><span data-stu-id="b43bc-153">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="b43bc-154">Verwaltbarkeit</span><span class="sxs-lookup"><span data-stu-id="b43bc-154">Manageability</span></span>

<span data-ttu-id="b43bc-155">Mit dem cloudübergreifenden Muster wird sichergestellt, dass zwischen Umgebungen eine nahtlose Verwaltung möglich und eine vertraute Benutzeroberfläche vorhanden ist.</span><span class="sxs-lookup"><span data-stu-id="b43bc-155">The cross-cloud pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="b43bc-156">Verwendung dieses Musters</span><span class="sxs-lookup"><span data-stu-id="b43bc-156">When to use this pattern</span></span>

<span data-ttu-id="b43bc-157">Verwenden Sie dieses Muster in folgenden Fällen:</span><span class="sxs-lookup"><span data-stu-id="b43bc-157">Use this pattern:</span></span>

- <span data-ttu-id="b43bc-158">Sie müssen die Kapazität Ihrer App aufgrund von unerwarteten Anforderungen oder regelmäßigen bedarfsgesteuerten Anforderungen erhöhen.</span><span class="sxs-lookup"><span data-stu-id="b43bc-158">When you need to increase your app capacity with unexpected demands or periodic demands in demand.</span></span>
- <span data-ttu-id="b43bc-159">Sie möchten nicht in Ressourcen investieren, die nur während Spitzenzeiten verwendet werden können.</span><span class="sxs-lookup"><span data-stu-id="b43bc-159">When you don't want to invest in resources that will only be used during peaks.</span></span> <span data-ttu-id="b43bc-160">Nutzungsabhängige Abrechnung.</span><span class="sxs-lookup"><span data-stu-id="b43bc-160">Pay for what you use.</span></span>

<span data-ttu-id="b43bc-161">Verwenden Sie dieses Muster in folgenden Fällen nicht:</span><span class="sxs-lookup"><span data-stu-id="b43bc-161">This pattern isn't recommended when:</span></span>

- <span data-ttu-id="b43bc-162">Ihre Lösung erfordert, dass Benutzer eine Verbindung über das Internet herstellen.</span><span class="sxs-lookup"><span data-stu-id="b43bc-162">Your solution requires users connecting over the internet.</span></span>
- <span data-ttu-id="b43bc-163">Ihr Unternehmen verfügt über lokale Bestimmungen, die erfordern, dass die Ursprungsverbindung von einem lokalen Aufruf erfolgt.</span><span class="sxs-lookup"><span data-stu-id="b43bc-163">Your business has local regulations that require that the originating connection to come from an onsite call.</span></span>
- <span data-ttu-id="b43bc-164">Ihr Netzwerk erfährt regelmäßig Engpässe, die die Leistung der Skalierung einschränken.</span><span class="sxs-lookup"><span data-stu-id="b43bc-164">Your network experiences regular bottlenecks that would restrict the performance of the scaling.</span></span>
- <span data-ttu-id="b43bc-165">Ihre Umgebung ist nicht mit dem Internet verbunden und kann nicht auf die öffentliche Cloud zugreifen.</span><span class="sxs-lookup"><span data-stu-id="b43bc-165">Your environment is disconnected from the internet and can't reach the public cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="b43bc-166">Nächste Schritte</span><span class="sxs-lookup"><span data-stu-id="b43bc-166">Next steps</span></span>

<span data-ttu-id="b43bc-167">Weitere Informationen zu den in diesem Artikel behandelten Themen:</span><span class="sxs-lookup"><span data-stu-id="b43bc-167">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="b43bc-168">In der [Übersicht über Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) erfahren Sie mehr dazu, wie dieser DNS-basierte Lastenausgleich für Datenverkehr funktioniert.</span><span class="sxs-lookup"><span data-stu-id="b43bc-168">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="b43bc-169">Unter [Überlegungen zum Entwurf von Hybrid-Apps](overview-app-design-considerations.md) erfahren Sie mehr zu bewährten Methoden und erhalten Antworten auf weitere Fragen.</span><span class="sxs-lookup"><span data-stu-id="b43bc-169">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="b43bc-170">Im Artikel zur [Azure Stack-Familie mit Produkten und Lösungen](/azure-stack) erfahren Sie mehr über das gesamte Portfolio von Produkten und Lösungen.</span><span class="sxs-lookup"><span data-stu-id="b43bc-170">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="b43bc-171">Wenn Sie bereit sind, das Lösungsbeispiel zu testen, fahren Sie mit dem [Bereitstellungsleitfaden für eine cloudübergreifende Skalierungslösung](solution-deployment-guide-cross-cloud-scaling.md) fort.</span><span class="sxs-lookup"><span data-stu-id="b43bc-171">When you're ready to test the solution example, continue with the [Cross-cloud scaling solution deployment guide](solution-deployment-guide-cross-cloud-scaling.md).</span></span> <span data-ttu-id="b43bc-172">In diesem Bereitstellungsleitfaden finden Sie detaillierte Anweisungen zum Bereitstellen und Testen der zugehörigen Komponenten.</span><span class="sxs-lookup"><span data-stu-id="b43bc-172">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="b43bc-173">Sie erfahren, wie Sie eine cloudübergreifende Lösung erstellen, um einen manuell ausgelösten Prozess zum Umschalten von einer in Azure Stack Hub gehosteten Web-App zu einer in Azure gehosteten Web-App bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="b43bc-173">You learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app.</span></span> <span data-ttu-id="b43bc-174">Sie erfahren auch, wie Sie die automatische Skalierung über Traffic Manager nutzen können, um ein flexibles und skalierbares Cloudhilfsprogramm unter Last zu gewährleisten.</span><span class="sxs-lookup"><span data-stu-id="b43bc-174">You also learn how to use autoscaling via traffic manager, ensuring flexible and scalable cloud utility when under load.</span></span>
