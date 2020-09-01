---
title: Weiterleiten von Datenverkehr mit einer geografisch verteilten App mithilfe von Azure und Azure Stack Hub
description: Es wird beschrieben, wie Sie mit einer geografisch verteilten App-Lösung Datenverkehr an bestimmte Endpunkte weiterleiten, indem Sie Azure und Azure Stack Hub verwenden.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 27d07070becfa902a715b451baae7c81c7e4b46f
ms.sourcegitcommit: 56980e3c118ca0a672974ee3835b18f6e81b6f43
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 08/26/2020
ms.locfileid: "88886831"
---
# <a name="direct-traffic-with-a-geo-distributed-app-using-azure-and-azure-stack-hub"></a><span data-ttu-id="f16cd-103">Weiterleiten von Datenverkehr mit einer geografisch verteilten App mithilfe von Azure und Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="f16cd-103">Direct traffic with a geo-distributed app using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="f16cd-104">Es wird beschrieben, wie Sie Datenverkehr basierend auf unterschiedlichen Metriken an bestimmte Endpunkte weiterleiten, indem Sie das Muster für geografisch verteilte Apps verwenden.</span><span class="sxs-lookup"><span data-stu-id="f16cd-104">Learn how to direct traffic to specific endpoints based on various metrics using the geo-distributed apps pattern.</span></span> <span data-ttu-id="f16cd-105">Durch die Erstellung eines Traffic Manager-Profils mit Weiterleitung und Endpunktkonfiguration anhand der Geografie wird sichergestellt, dass Informationen basierend auf regionalen Anforderungen, unternehmensinternen und internationalen Bestimmungen und Ihren Datenanforderungen an Endpunkte weitergeleitet werden.</span><span class="sxs-lookup"><span data-stu-id="f16cd-105">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>

<span data-ttu-id="f16cd-106">In dieser Lösung erstellen Sie eine Beispielumgebung, die Folgendes ermöglicht:</span><span class="sxs-lookup"><span data-stu-id="f16cd-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="f16cd-107">Erstellen einer geografisch verteilten App</span><span class="sxs-lookup"><span data-stu-id="f16cd-107">Create a geo-distributed app.</span></span>
> - <span data-ttu-id="f16cd-108">Verwenden von Traffic Manager für Ihre App</span><span class="sxs-lookup"><span data-stu-id="f16cd-108">Use Traffic Manager to target your app.</span></span>

## <a name="use-the-geo-distributed-apps-pattern"></a><span data-ttu-id="f16cd-109">Verwenden des Musters für geografisch verteilte Apps</span><span class="sxs-lookup"><span data-stu-id="f16cd-109">Use the geo-distributed apps pattern</span></span>

<span data-ttu-id="f16cd-110">Mit dem Muster für die geografische Verteilung erzielt Ihre App eine regionsübergreifende Abdeckung.</span><span class="sxs-lookup"><span data-stu-id="f16cd-110">With the geo-distributed pattern, your app spans regions.</span></span> <span data-ttu-id="f16cd-111">Sie können als Standard die öffentliche Cloud verwenden, aber für einige Ihrer Benutzer kann es erforderlich sein, dass die Daten in ihrer Region verbleiben.</span><span class="sxs-lookup"><span data-stu-id="f16cd-111">You can default to the public cloud, but some of your users may require that their data remain in their region.</span></span> <span data-ttu-id="f16cd-112">Je nach den Anforderungen können Sie Benutzer an die am besten geeignete Cloud weiterleiten.</span><span class="sxs-lookup"><span data-stu-id="f16cd-112">You can direct users to the most suitable cloud based on their requirements.</span></span>

### <a name="issues-and-considerations"></a><span data-ttu-id="f16cd-113">Probleme und Überlegungen</span><span class="sxs-lookup"><span data-stu-id="f16cd-113">Issues and considerations</span></span>

#### <a name="scalability-considerations"></a><span data-ttu-id="f16cd-114">Überlegungen zur Skalierbarkeit</span><span class="sxs-lookup"><span data-stu-id="f16cd-114">Scalability considerations</span></span>

<span data-ttu-id="f16cd-115">In die Lösung, die Sie im Zuge dieses Artikels erstellen, wird die Skalierbarkeit nicht einbezogen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-115">The solution you'll build with this article isn't to accommodate scalability.</span></span> <span data-ttu-id="f16cd-116">Wenn Sie sie in Kombination mit anderen Azure- und lokalen Lösungen verwenden, ist es aber möglich, dass Sie die Anforderungen an die Skalierbarkeit erfüllen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-116">However, if used in combination with other Azure and on-premises solutions, you can accommodate scalability requirements.</span></span> <span data-ttu-id="f16cd-117">Informationen zur Erstellung einer Hybridlösung mit automatischer Skalierung per Traffic Manager finden Sie unter [Erstellen von cloudübergreifenden Skalierungslösungen mit Azure](solution-deployment-guide-cross-cloud-scaling.md).</span><span class="sxs-lookup"><span data-stu-id="f16cd-117">For information on creating a hybrid solution with autoscaling via traffic manager, see [Create cross-cloud scaling solutions with Azure](solution-deployment-guide-cross-cloud-scaling.md).</span></span>

#### <a name="availability-considerations"></a><span data-ttu-id="f16cd-118">Überlegungen zur Verfügbarkeit</span><span class="sxs-lookup"><span data-stu-id="f16cd-118">Availability considerations</span></span>

<span data-ttu-id="f16cd-119">Wie bei den Skalierbarkeitsaspekten auch, geht es bei dieser Lösung nicht direkt um die Verfügbarkeit.</span><span class="sxs-lookup"><span data-stu-id="f16cd-119">As is the case with scalability considerations, this solution doesn't directly address availability.</span></span> <span data-ttu-id="f16cd-120">Azure- und lokale Lösungen können jedoch in diese Lösung implementiert werden, um für alle beteiligten Komponenten die Hochverfügbarkeit sicherzustellen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-120">However, Azure and on-premises solutions can be implemented within this solution to ensure high availability for all components involved.</span></span>

### <a name="when-to-use-this-pattern"></a><span data-ttu-id="f16cd-121">Verwendung dieses Musters</span><span class="sxs-lookup"><span data-stu-id="f16cd-121">When to use this pattern</span></span>

- <span data-ttu-id="f16cd-122">Ihre Organisation verfügt über internationale Zweigniederlassungen, für die benutzerdefinierte regionale Sicherheits- und Verteilungsrichtlinien erforderlich sind.</span><span class="sxs-lookup"><span data-stu-id="f16cd-122">Your organization has international branches requiring custom regional security and distribution policies.</span></span>

- <span data-ttu-id="f16cd-123">Alle Niederlassungen Ihrer Organisation rufen per Pullvorgang Daten zu Mitarbeitern, Geschäft und Einrichtungen ab, sodass Berichterstellungsaktivitäten gemäß den lokalen Bestimmungen und Zeitzonen benötigt werden.</span><span class="sxs-lookup"><span data-stu-id="f16cd-123">Each of your organization's offices pulls employee, business, and facility data, which requires reporting activity per local regulations and time zones.</span></span>

- <span data-ttu-id="f16cd-124">Anforderungen für eine hohe Skalierbarkeit werden durch das horizontale Hochskalieren von Apps erfüllt, wobei mehrere App-Bereitstellungen innerhalb einer Region sowie regionsübergreifend erfolgen, um extreme Lastanforderungen zu verarbeiten.</span><span class="sxs-lookup"><span data-stu-id="f16cd-124">High-scale requirements are met by horizontally scaling out apps with multiple app deployments within a single region and across regions to handle extreme load requirements.</span></span>

### <a name="planning-the-topology"></a><span data-ttu-id="f16cd-125">Planen der Topologie</span><span class="sxs-lookup"><span data-stu-id="f16cd-125">Planning the topology</span></span>

<span data-ttu-id="f16cd-126">Es ist hilfreich, wenn Sie Folgendes wissen, bevor Sie den Speicherbedarf für eine verteilte App erstellen:</span><span class="sxs-lookup"><span data-stu-id="f16cd-126">Before building out a distributed app footprint, it helps to know the following things:</span></span>

- <span data-ttu-id="f16cd-127">**Benutzerdefinierte Domäne für die App:** Wie lautet der Name der benutzerdefinierten Domäne, mit dem Kunden auf die App zugreifen?</span><span class="sxs-lookup"><span data-stu-id="f16cd-127">**Custom domain for the app:** What's the custom domain name that customers will use to access the app?</span></span> <span data-ttu-id="f16cd-128">Für die Beispiel-App lautet der Name der benutzerdefinierten Domäne *www\.scalableasedemo.com.*</span><span class="sxs-lookup"><span data-stu-id="f16cd-128">For the sample app, the custom domain name is *www\.scalableasedemo.com.*</span></span>

- <span data-ttu-id="f16cd-129">**Traffic Manager-Domäne:** Ein Domänenname wird ausgewählt, wenn Sie ein [Azure Traffic Manager-Profil](/azure/traffic-manager/traffic-manager-manage-profiles) erstellen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-129">**Traffic Manager domain:** A domain name is chosen when creating an [Azure Traffic Manager profile](/azure/traffic-manager/traffic-manager-manage-profiles).</span></span> <span data-ttu-id="f16cd-130">Dieser Name wird mit dem Suffix *trafficmanager.net* kombiniert, um einen Domäneneintrag zu registrieren, der von Traffic Manager verwaltet wird.</span><span class="sxs-lookup"><span data-stu-id="f16cd-130">This name is combined with the *trafficmanager.net* suffix to register a domain entry that's managed by Traffic Manager.</span></span> <span data-ttu-id="f16cd-131">Für die Beispiel-App ist der ausgewählte Name *scalable-ase-demo*.</span><span class="sxs-lookup"><span data-stu-id="f16cd-131">For the sample app, the name chosen is *scalable-ase-demo*.</span></span> <span data-ttu-id="f16cd-132">Der vollständige Domänenname, der von Traffic Manager verwaltet wird, lautet also *scalable-ase-demo.trafficmanager.net*.</span><span class="sxs-lookup"><span data-stu-id="f16cd-132">As a result, the full domain name that's managed by Traffic Manager is *scalable-ase-demo.trafficmanager.net*.</span></span>

- <span data-ttu-id="f16cd-133">**Strategie für die Skalierung der App:** Entscheiden Sie, ob die App über mehrere App Service-Umgebungen in einer Region, mehreren Regionen oder mit einer Mischung aus beiden Ansätzen verteilt werden soll.</span><span class="sxs-lookup"><span data-stu-id="f16cd-133">**Strategy for scaling the app footprint:** Decide whether the app footprint will be distributed across multiple App Service environments in a single region, multiple regions, or a mix of both approaches.</span></span> <span data-ttu-id="f16cd-134">Die Entscheidung sollte darauf basieren, woher der Kundendatenverkehr erwartet wird, aber auch darauf, wie gut der Rest der Back-End-Infrastruktur zur Unterstützung einer App skaliert werden kann.</span><span class="sxs-lookup"><span data-stu-id="f16cd-134">The decision should be based on expectations of where customer traffic will originate and how well the rest of an app's supporting back-end infrastructure can scale.</span></span> <span data-ttu-id="f16cd-135">Bei einer hundertprozentigen Zustandslosigkeit kann eine App beispielsweise hochgradig skaliert werden, indem eine Kombination von mehreren App Service-Umgebungen pro Azure-Region verwendet und dies mit App Service-Umgebungen, die in mehreren Azure-Regionen bereitgestellt sind, multipliziert wird.</span><span class="sxs-lookup"><span data-stu-id="f16cd-135">For example, with a 100% stateless app, an app can be massively scaled using a combination of multiple App Service environments per Azure region, multiplied by App Service environments deployed across multiple Azure regions.</span></span> <span data-ttu-id="f16cd-136">Da Kunden aus mehr als 15 globalen Azure-Regionen auswählen können, ist wirklich die Erstellung einer weltweiten, hoch skalierbaren App möglich.</span><span class="sxs-lookup"><span data-stu-id="f16cd-136">With 15+ global Azure regions available to choose from, customers can truly build a world-wide hyper-scale app footprint.</span></span> <span data-ttu-id="f16cd-137">Für die hier verwendete Beispiel-App wurden drei App Service-Umgebungen in einer einzelnen Azure-Region (USA (Mitte/Süden)) erstellt.</span><span class="sxs-lookup"><span data-stu-id="f16cd-137">For the sample app used here, three App Service environments were created in a single Azure region (South Central US).</span></span>

- <span data-ttu-id="f16cd-138">**Benennungskonvention für die App Service-Umgebungen:** Jede App Service-Umgebung muss über einen eindeutigen Namen verfügen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-138">**Naming convention for the App Service environments:** Each App Service environment requires a unique name.</span></span> <span data-ttu-id="f16cd-139">Bei mehr als ein oder zwei App Service-Umgebungen ist es hilfreich, über eine Benennungskonvention zu verfügen, um die Identifizierung der einzelnen App Service-Umgebungen zu vereinfachen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-139">Beyond one or two App Service environments, it's helpful to have a naming convention to help identify each App Service environment.</span></span> <span data-ttu-id="f16cd-140">Für die hier verwendete Beispiel-App wurde eine einfache Benennungskonvention verwendet.</span><span class="sxs-lookup"><span data-stu-id="f16cd-140">For the sample app used here, a simple naming convention was used.</span></span> <span data-ttu-id="f16cd-141">Die Namen der drei App Service-Umgebungen sind *fe1ase*, *fe2ase* und *fe3ase*.</span><span class="sxs-lookup"><span data-stu-id="f16cd-141">The names of the three App Service environments are *fe1ase*, *fe2ase*, and *fe3ase*.</span></span>

- <span data-ttu-id="f16cd-142">**Benennungskonvention für die Apps:** Da mehrere Instanzen der App bereitgestellt werden, ist ein Name für jede Instanz der bereitgestellten App erforderlich.</span><span class="sxs-lookup"><span data-stu-id="f16cd-142">**Naming convention for the apps:** Since multiple instances of the app will be deployed, a name is needed for each instance of the deployed app.</span></span> <span data-ttu-id="f16cd-143">Bei App Service-Umgebungen für Power Apps kann der gleiche App-Name über mehrere Umgebungen hinweg genutzt werden.</span><span class="sxs-lookup"><span data-stu-id="f16cd-143">With App Service Environment for Power Apps, the same app name can be used across multiple environments.</span></span> <span data-ttu-id="f16cd-144">Da jede App Service-Umgebung ein eindeutiges Domänensuffix aufweist, können Entwickler den gleichen App-Namen in jeder Umgebung wiederverwenden.</span><span class="sxs-lookup"><span data-stu-id="f16cd-144">Since each App Service environment has a unique domain suffix, developers can choose to reuse the exact same app name in each environment.</span></span> <span data-ttu-id="f16cd-145">Ein Entwickler kann Apps beispielsweise wie folgt benennen: *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net* usw.</span><span class="sxs-lookup"><span data-stu-id="f16cd-145">For example, a developer could have apps named as follows: *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net*, and so on.</span></span> <span data-ttu-id="f16cd-146">Für die hier verwendete App hat jede App-Instanz einen eindeutigen Namen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-146">For the app used here, each app instance has a unique name.</span></span> <span data-ttu-id="f16cd-147">Die verwendeten Namen für die App-Instanzen sind *webfrontend1*, *webfrontend2*, und *webfrontend3*.</span><span class="sxs-lookup"><span data-stu-id="f16cd-147">The app instance names used are *webfrontend1*, *webfrontend2*, and *webfrontend3*.</span></span>

> [!Tip]  
> <span data-ttu-id="f16cd-148">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="f16cd-148">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="f16cd-149">Microsoft Azure Stack Hub ist eine Erweiterung von Azure.</span><span class="sxs-lookup"><span data-stu-id="f16cd-149">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="f16cd-150">Mit Azure Stack Hub holen Sie sich die Agilität und Innovation von Cloud Computing in Ihre lokale Umgebung. Sie erhalten die einzige Hybrid Cloud, mit der Sie Hybrid-Apps überall entwickeln und bereitstellen können.</span><span class="sxs-lookup"><span data-stu-id="f16cd-150">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="f16cd-151">Im Artikel [Überlegungen zum Entwurf von Hybrid-Apps](overview-app-design-considerations.md) werden die wichtigen Aspekte in Bezug auf die Softwarequalität (Platzierung, Skalierbarkeit, Verfügbarkeit, Resilienz, Verwaltbarkeit und Sicherheit) beschrieben, die für das Entwerfen, Bereitstellen und Betreiben von Hybrid-Apps erforderlich sind.</span><span class="sxs-lookup"><span data-stu-id="f16cd-151">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="f16cd-152">Die Überlegungen zum Entwurf dienen als Hilfe beim Optimieren des Designs von Hybrid-Apps, um für Produktionsumgebungen das Auftreten von Problemen zu minimieren.</span><span class="sxs-lookup"><span data-stu-id="f16cd-152">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="part-1-create-a-geo-distributed-app"></a><span data-ttu-id="f16cd-153">Teil 1: Erstellen einer geografisch verteilten App</span><span class="sxs-lookup"><span data-stu-id="f16cd-153">Part 1: Create a geo-distributed app</span></span>

<span data-ttu-id="f16cd-154">In diesem Teil erstellen Sie eine Web-App.</span><span class="sxs-lookup"><span data-stu-id="f16cd-154">In this part, you'll create a web app.</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="f16cd-155">Erstellen von Web-Apps und Durchführen der Veröffentlichung</span><span class="sxs-lookup"><span data-stu-id="f16cd-155">Create web apps and publish.</span></span>
> - <span data-ttu-id="f16cd-156">Hinzufügen von Code zu Azure Repos</span><span class="sxs-lookup"><span data-stu-id="f16cd-156">Add code to Azure Repos.</span></span>
> - <span data-ttu-id="f16cd-157">Verweisen Sie für den App-Build auf mehrere Cloudziele.</span><span class="sxs-lookup"><span data-stu-id="f16cd-157">Point the app build to multiple cloud targets.</span></span>
> - <span data-ttu-id="f16cd-158">Verwalten und Konfigurieren des CD-Prozesses</span><span class="sxs-lookup"><span data-stu-id="f16cd-158">Manage and configure the CD process.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="f16cd-159">Voraussetzungen</span><span class="sxs-lookup"><span data-stu-id="f16cd-159">Prerequisites</span></span>

<span data-ttu-id="f16cd-160">Ein Azure-Abonnement und eine Azure Stack Hub-Installation sind erforderlich.</span><span class="sxs-lookup"><span data-stu-id="f16cd-160">An Azure subscription and Azure Stack Hub installation are required.</span></span>

### <a name="geo-distributed-app-steps"></a><span data-ttu-id="f16cd-161">Geografisch verteilte App – Schritte</span><span class="sxs-lookup"><span data-stu-id="f16cd-161">Geo-distributed app steps</span></span>

### <a name="obtain-a-custom-domain-and-configure-dns"></a><span data-ttu-id="f16cd-162">Abrufen einer benutzerdefinierten Domäne und Konfigurieren des DNS</span><span class="sxs-lookup"><span data-stu-id="f16cd-162">Obtain a custom domain and configure DNS</span></span>

<span data-ttu-id="f16cd-163">Aktualisieren Sie die DNS-Zonendatei für die Domäne.</span><span class="sxs-lookup"><span data-stu-id="f16cd-163">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="f16cd-164">Azure AD kann dann die Eigentümerschaft für den Namen der benutzerdefinierten Domäne überprüfen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-164">Azure AD can then verify ownership of the custom domain name.</span></span> <span data-ttu-id="f16cd-165">Verwenden Sie [Azure DNS](/azure/dns/dns-getstarted-portal) für Azure- oder Microsoft 365-Einträge bzw. externe DNS-Einträge in Azure, oder fügen Sie den DNS-Eintrag bei einer [anderen DNS-Registrierungsstelle](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider) hinzu.</span><span class="sxs-lookup"><span data-stu-id="f16cd-165">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

1. <span data-ttu-id="f16cd-166">Registrieren Sie eine benutzerdefinierte Domäne bei einer öffentlichen Registrierungsstelle.</span><span class="sxs-lookup"><span data-stu-id="f16cd-166">Register a custom domain with a public registrar.</span></span>

2. <span data-ttu-id="f16cd-167">Melden Sie sich an der Domänennamen-Registrierungsstelle für die Domäne an.</span><span class="sxs-lookup"><span data-stu-id="f16cd-167">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="f16cd-168">Unter Umständen ist ein genehmigter Administrator erforderlich, um die DNS-Updates durchzuführen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-168">An approved admin may be required to make the DNS updates.</span></span>

3. <span data-ttu-id="f16cd-169">Aktualisieren Sie die DNS-Zonendatei für die Domäne, indem Sie den DNS-Eintrag hinzufügen, der von Azure AD bereitgestellt wurde.</span><span class="sxs-lookup"><span data-stu-id="f16cd-169">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="f16cd-170">Der DNS-Eintrag nimmt keine Änderungen in Bezug auf das Verhalten vor, z.B. E-Mail-Routing oder Webhosting.</span><span class="sxs-lookup"><span data-stu-id="f16cd-170">The DNS entry doesn't change behaviors such as mail routing or web hosting.</span></span>

### <a name="create-web-apps-and-publish"></a><span data-ttu-id="f16cd-171">Erstellen von Web-Apps und Durchführen der Veröffentlichung</span><span class="sxs-lookup"><span data-stu-id="f16cd-171">Create web apps and publish</span></span>

<span data-ttu-id="f16cd-172">Richten Sie Hybrid-CI/CD (Continuous Integration/Continuous Delivery) ein, um die Web-App unter Azure und Azure Stack Hub bereitzustellen und Änderungen automatisch per Pushvorgang an beide Clouds zu übertragen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-172">Set up Hybrid Continuous Integration/Continuous Delivery (CI/CD) to deploy Web App to Azure and Azure Stack Hub, and auto push changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="f16cd-173">Azure Stack Hub mit den passenden syndizierten Images für die Ausführung (Windows Server und SQL) und eine App Service-Bereitstellung sind erforderlich.</span><span class="sxs-lookup"><span data-stu-id="f16cd-173">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="f16cd-174">Weitere Informationen finden Sie unter [Voraussetzungen für das Bereitstellen von App Service in Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span><span class="sxs-lookup"><span data-stu-id="f16cd-174">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

#### <a name="add-code-to-azure-repos"></a><span data-ttu-id="f16cd-175">Hinzufügen von Code zu Azure Repos</span><span class="sxs-lookup"><span data-stu-id="f16cd-175">Add Code to Azure Repos</span></span>

1. <span data-ttu-id="f16cd-176">Melden Sie sich bei Visual Studio mit einem **Konto an, das über Berechtigungen zum Erstellen von Projekten** in Azure Repos verfügt.</span><span class="sxs-lookup"><span data-stu-id="f16cd-176">Sign in to Visual Studio with an **account that has project creation rights** on Azure Repos.</span></span>

    <span data-ttu-id="f16cd-177">Der CI/CD-Ansatz kann sowohl für App-Code als auch für Infrastrukturcode verwendet werden.</span><span class="sxs-lookup"><span data-stu-id="f16cd-177">CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="f16cd-178">Verwenden Sie [Azure Resource Manager-Vorlagen](https://azure.microsoft.com/resources/templates/) für die Entwicklung von privaten und gehosteten Clouds.</span><span class="sxs-lookup"><span data-stu-id="f16cd-178">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Verbinden mit einem Projekt in Visual Studio](media/solution-deployment-guide-geo-distributed/image1.JPG)

2. <span data-ttu-id="f16cd-180">**Klonen Sie das Repository**, indem Sie die Standard-Web-App erstellen und öffnen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-180">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Klonen eines Repositorys in Visual Studio](media/solution-deployment-guide-geo-distributed/image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a><span data-ttu-id="f16cd-182">Erstellen der Web-App-Bereitstellung in beiden Clouds</span><span class="sxs-lookup"><span data-stu-id="f16cd-182">Create web app deployment in both clouds</span></span>

1. <span data-ttu-id="f16cd-183">Bearbeiten Sie die Datei **WebApplication.csproj**: Wählen Sie `Runtimeidentifier` aus, und fügen Sie `win10-x64` hinzu.</span><span class="sxs-lookup"><span data-stu-id="f16cd-183">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="f16cd-184">(Weitere Informationen finden Sie in der Dokumentation zur [eigenständigen Bereitstellung](/dotnet/core/deploying/deploy-with-vs#simpleSelf).)</span><span class="sxs-lookup"><span data-stu-id="f16cd-184">(See [Self-contained Deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Bearbeiten der Projektdatei der Web-App in Visual Studio](media/solution-deployment-guide-geo-distributed/image3.png)

2. <span data-ttu-id="f16cd-186">**Checken Sie den Code in Azure Repos ein**, indem Sie Team Explorer verwenden.</span><span class="sxs-lookup"><span data-stu-id="f16cd-186">**Check in the code to Azure Repos** using Team Explorer.</span></span>

3. <span data-ttu-id="f16cd-187">Vergewissern Sie sich, dass der **Anwendungscode** in Azure Repos eingecheckt wurde.</span><span class="sxs-lookup"><span data-stu-id="f16cd-187">Confirm that the **application code** has been checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="f16cd-188">Erstellen der Builddefinition</span><span class="sxs-lookup"><span data-stu-id="f16cd-188">Create the build definition</span></span>

1. <span data-ttu-id="f16cd-189">**Melden Sie sich bei Azure Pipelines an**, um sich zu vergewissern, dass Builddefinitionen erstellt werden können.</span><span class="sxs-lookup"><span data-stu-id="f16cd-189">**Sign in to Azure Pipelines** to confirm ability to create build definitions.</span></span>

2. <span data-ttu-id="f16cd-190">Fügen Sie den `-r win10-x64`-Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="f16cd-190">Add `-r win10-x64` code.</span></span> <span data-ttu-id="f16cd-191">Diese Hinzufügung ist erforderlich, um eine eigenständige Bereitstellung mit .NET Core auszulösen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-191">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Hinzufügen von Code zur Builddefinition in Azure Pipelines](media/solution-deployment-guide-geo-distributed/image4.png)

3. <span data-ttu-id="f16cd-193">**Führen Sie den Buildvorgang durch.**</span><span class="sxs-lookup"><span data-stu-id="f16cd-193">**Run the build**.</span></span> <span data-ttu-id="f16cd-194">Der Buildvorgang für die [eigenständige Bereitstellung](/dotnet/core/deploying/deploy-with-vs#simpleSelf) veröffentlicht Artefakte, die in Azure und Azure Stack Hub ausgeführt werden können.</span><span class="sxs-lookup"><span data-stu-id="f16cd-194">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="using-an-azure-hosted-agent"></a><span data-ttu-id="f16cd-195">Verwenden eines gehosteten Azure-Agents</span><span class="sxs-lookup"><span data-stu-id="f16cd-195">Using an Azure Hosted Agent</span></span>

<span data-ttu-id="f16cd-196">Mit einem gehosteten Agent lassen sich in Azure Pipelines komfortabel Web-Apps erstellen und bereitstellen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-196">Using a hosted agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="f16cd-197">Wartungsarbeiten und Upgrades werden automatisch von Microsoft Azure durchgeführt, was ununterbrochene Entwicklungs-, Test- und Bereitstellungsaktivitäten ermöglicht.</span><span class="sxs-lookup"><span data-stu-id="f16cd-197">Maintenance and upgrades are automatically performed by Microsoft Azure, which enables uninterrupted development, testing, and deployment.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="f16cd-198">Verwalten und Konfigurieren des CD-Prozesses</span><span class="sxs-lookup"><span data-stu-id="f16cd-198">Manage and configure the CD process</span></span>

<span data-ttu-id="f16cd-199">Azure DevOps Services bieten eine äußerst flexibel konfigurier- und verwaltbare Pipeline für Releases in mehreren Umgebungen (z.B. in Entwicklungs-, Staging-, Qualitätssicherungs- und Produktionsumgebungen) – einschließlich erforderlicher Genehmigungen in bestimmten Phasen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-199">Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments such as development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="f16cd-200">Erstellen einer Releasedefinition</span><span class="sxs-lookup"><span data-stu-id="f16cd-200">Create release definition</span></span>

1. <span data-ttu-id="f16cd-201">Klicken Sie in Azure DevOps Services im Abschnitt **Build und Release** auf der Registerkarte **Releases** auf die Schaltfläche mit dem **Pluszeichen**, um ein neues Release hinzuzufügen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-201">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![Erstellen einer Releasedefinition in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image5.png)

2. <span data-ttu-id="f16cd-203">Wenden Sie die Vorlage „Azure App Service-Bereitstellung“ an.</span><span class="sxs-lookup"><span data-stu-id="f16cd-203">Apply the Azure App Service Deployment template.</span></span>

   ![Anwenden einer Azure App Service-Bereitstellungsvorlage in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image6.png)

3. <span data-ttu-id="f16cd-205">Fügen Sie unter **Artefakt hinzufügen** das Artefakt für die Azure Cloud-Build-App hinzu.</span><span class="sxs-lookup"><span data-stu-id="f16cd-205">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Hinzufügen eines Artefakts zum Azure Cloud-Build in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image7.png)

4. <span data-ttu-id="f16cd-207">Klicken Sie auf der Registerkarte „Pipeline“ auf den Link **Phase, Aufgabe** der Umgebung, und legen Sie die Azure Cloud-Umgebungswerte fest.</span><span class="sxs-lookup"><span data-stu-id="f16cd-207">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Festlegen von Werten für die Azure Cloud-Umgebung in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image8.png)

5. <span data-ttu-id="f16cd-209">Legen Sie den **Umgebungsnamen** fest, und wählen Sie das **Azure-Abonnement** für den Azure Cloud-Endpunkt aus.</span><span class="sxs-lookup"><span data-stu-id="f16cd-209">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Auswählen des Azure-Abonnements für den Azure Cloud-Endpunkt in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image9.png)

6. <span data-ttu-id="f16cd-211">Legen Sie unter **App Service-Name** den erforderlichen Namen des Azure App Service fest.</span><span class="sxs-lookup"><span data-stu-id="f16cd-211">Under **App service name**, set the required Azure app service name.</span></span>

      ![Festlegen eines Azure App Service-Namens in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image10.png)

7. <span data-ttu-id="f16cd-213">Geben Sie unter **Agent-Warteschlange** für die gehostete Azure Cloud-Umgebung die Zeichenfolge „Hosted VS2017“ ein.</span><span class="sxs-lookup"><span data-stu-id="f16cd-213">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Festlegen der Agent-Warteschlange für eine in der Azure Cloud gehostete Umgebung in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image11.png)

8. <span data-ttu-id="f16cd-215">Wählen Sie im Menü „Deploy Azure App Service“ (Azure App Service bereitstellen) **das gültige Paket oder den gültigen Ordner** für die Umgebung aus.</span><span class="sxs-lookup"><span data-stu-id="f16cd-215">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="f16cd-216">Klicken Sie für den **Ordnerspeicherort** auf **OK**.</span><span class="sxs-lookup"><span data-stu-id="f16cd-216">Select **OK** to **folder location**.</span></span>
  
      ![Auswählen eines Pakets oder Ordners für eine Azure App Service-Umgebung in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image12.png)

      ![Auswählen eines Pakets oder Ordners für eine Azure App Service-Umgebung in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image13.png)

9. <span data-ttu-id="f16cd-219">Speichern Sie alle Änderungen, und kehren Sie zur **Releasepipeline** zurück.</span><span class="sxs-lookup"><span data-stu-id="f16cd-219">Save all changes and go back to **release pipeline**.</span></span>

    ![Speichern von Änderungen in der Releasepipeline in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image14.png)

10. <span data-ttu-id="f16cd-221">Fügen Sie ein neues Artefakt hinzu, und wählen Sie dabei den Build für die Azure Stack Hub-App aus.</span><span class="sxs-lookup"><span data-stu-id="f16cd-221">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Hinzufügen eines neuen Artefakts für eine Azure Stack Hub-App in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image15.png)


11. <span data-ttu-id="f16cd-223">Fügen Sie mindestens eine Umgebung hinzu, indem Sie die Azure App Service-Bereitstellung anwenden.</span><span class="sxs-lookup"><span data-stu-id="f16cd-223">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Hinzufügen einer Umgebung zur Azure App Service-Bereitstellung in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image16.png)

12. <span data-ttu-id="f16cd-225">Nennen Sie die neue Umgebung „Azure Stack Hub“.</span><span class="sxs-lookup"><span data-stu-id="f16cd-225">Name the new environment Azure Stack Hub.</span></span>

    ![Benennen einer Umgebung in einer Azure App Service-Bereitstellung in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image17.png)

13. <span data-ttu-id="f16cd-227">Suchen Sie auf der Registerkarte **Aufgabe** nach der Umgebung „Azure Stack Hub“.</span><span class="sxs-lookup"><span data-stu-id="f16cd-227">Find the Azure Stack Hub environment under **Task** tab.</span></span>

    ![Azure Stack Hub-Umgebung in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image18.png)

14. <span data-ttu-id="f16cd-229">Wählen Sie das Abonnement für den Azure Stack Hub-Endpunkt aus.</span><span class="sxs-lookup"><span data-stu-id="f16cd-229">Select the subscription for the Azure Stack Hub endpoint.</span></span>

    ![Auswählen des Abonnements für den Azure Stack Hub-Endpunkt in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image19.png)

15. <span data-ttu-id="f16cd-231">Legen Sie den App Service-Namen als den Namen der Azure Stack Hub-Web-App fest.</span><span class="sxs-lookup"><span data-stu-id="f16cd-231">Set the Azure Stack Hub web app name as the App service name.</span></span>

    ![Festlegen des Namens für die Azure Stack Hub-Web-App in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image20.png)

16. <span data-ttu-id="f16cd-233">Wählen Sie den Azure Stack Hub-Agent aus.</span><span class="sxs-lookup"><span data-stu-id="f16cd-233">Select the Azure Stack Hub agent.</span></span>

    ![Auswählen des Azure Stack Hub-Agents in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image21.png)

17. <span data-ttu-id="f16cd-235">Wählen Sie im Abschnitt „Deploy Azure App Service“ (Azure App Service bereitstellen) **das gültige Paket oder den gültigen Ordner** für die Umgebung aus.</span><span class="sxs-lookup"><span data-stu-id="f16cd-235">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="f16cd-236">Klicken Sie für den Ordnerspeicherort auf **OK**.</span><span class="sxs-lookup"><span data-stu-id="f16cd-236">Select **OK** to folder location.</span></span>

    ![Auswählen eines Ordners für die Azure App Service-Bereitstellung in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image22.png)

    ![Auswählen eines Ordners für die Azure App Service-Bereitstellung in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image23.png)

18. <span data-ttu-id="f16cd-239">Fügen Sie auf der Registerkarte „Variable“ eine Variable mit dem Namen `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` hinzu, und legen Sie ihren Wert auf **true** und den Bereich auf „Azure Stack Hub“ fest.</span><span class="sxs-lookup"><span data-stu-id="f16cd-239">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack Hub.</span></span>

    ![Hinzufügen einer Variablen zur Azure-App-Bereitstellung in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image24.png)

19. <span data-ttu-id="f16cd-241">Klicken Sie bei beiden Artefakten auf das Symbol **Continuous Deployment-Trigger**, und aktivieren Sie den Trigger für **Continuous Deployment**.</span><span class="sxs-lookup"><span data-stu-id="f16cd-241">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![Auswählen des Continuous Deployment-Triggers in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image25.png)

20. <span data-ttu-id="f16cd-243">Klicken Sie für die Azure Stack Hub-Umgebung auf das Symbol **Bedingungen vor der Bereitstellung**, und legen Sie den Trigger auf **Nach einem Release** fest.</span><span class="sxs-lookup"><span data-stu-id="f16cd-243">Select the **Pre-deployment** conditions icon in the Azure Stack Hub environment and set the trigger to **After release.**</span></span>

    ![Auswählen des Triggers für Bedingungen vor der Bereitstellung in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image26.png)

21. <span data-ttu-id="f16cd-245">Speichern Sie alle Änderungen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-245">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="f16cd-246">Bei der vorlagenbasierten Erstellung einer Releasedefinition wurden einige Einstellungen für die Aufgaben unter Umständen automatisch als [Umgebungsvariablen](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) definiert.</span><span class="sxs-lookup"><span data-stu-id="f16cd-246">Some settings for the tasks may have been automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="f16cd-247">Diese Einstellungen können in den Aufgabeneinstellungen nicht geändert werden. Zum Bearbeiten dieser Einstellungen müssen Sie das übergeordnete Umgebungselement auswählen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-247">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="part-2-update-web-app-options"></a><span data-ttu-id="f16cd-248">Teil 2: Aktualisieren der Web-App-Optionen</span><span class="sxs-lookup"><span data-stu-id="f16cd-248">Part 2: Update web app options</span></span>

<span data-ttu-id="f16cd-249">Von [Azure App Service](/azure/app-service/overview) wird ein hochgradig skalierbarer Webhostingdienst mit Self-Patching bereitgestellt.</span><span class="sxs-lookup"><span data-stu-id="f16cd-249">[Azure App Service](/azure/app-service/overview) provides a highly scalable, self-patching web hosting service.</span></span>

![Azure App Service](media/solution-deployment-guide-geo-distributed/image27.png)

> [!div class="checklist"]
> - <span data-ttu-id="f16cd-251">Ordnen Sie Azure-Web-Apps einen vorhandenen benutzerdefinierten DNS-Namen zu.</span><span class="sxs-lookup"><span data-stu-id="f16cd-251">Map an existing custom DNS name to Azure Web Apps.</span></span>
> - <span data-ttu-id="f16cd-252">Verwenden Sie einen **CNAME-Eintrag** und einen **A-Eintrag**, um App Service einen benutzerdefinierten DNS-Namen zuzuordnen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-252">Use a **CNAME record** and an **A record** to map a custom DNS name to App Service.</span></span>

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a><span data-ttu-id="f16cd-253">Zuordnen eines vorhandenen benutzerdefinierten DNS-Namens zu Azure-Web-Apps</span><span class="sxs-lookup"><span data-stu-id="f16cd-253">Map an existing custom DNS name to Azure Web Apps</span></span>

> [!Note]  
> <span data-ttu-id="f16cd-254">Verwenden Sie einen CNAME-Eintrag für alle benutzerdefinierten DNS-Namen – außer für Stammdomänen (z. B. „northwind.com“).</span><span class="sxs-lookup"><span data-stu-id="f16cd-254">Use a CNAME for all custom DNS names except a root domain (for example, northwind.com).</span></span>

<span data-ttu-id="f16cd-255">Informationen zum Migrieren einer Livewebsite und ihres DNS-Domänennamens zu App Service finden Sie unter [Migrieren einer aktiven benutzerdefinierten Domäne zu Azure App Service](/azure/app-service/manage-custom-dns-migrate-domain).</span><span class="sxs-lookup"><span data-stu-id="f16cd-255">To migrate a live site and its DNS domain name to App Service, see [Migrate an active DNS name to Azure App Service](/azure/app-service/manage-custom-dns-migrate-domain).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="f16cd-256">Voraussetzungen</span><span class="sxs-lookup"><span data-stu-id="f16cd-256">Prerequisites</span></span>

<span data-ttu-id="f16cd-257">Führen Sie für diese Lösung die folgenden Schritte aus:</span><span class="sxs-lookup"><span data-stu-id="f16cd-257">To complete this solution:</span></span>

- <span data-ttu-id="f16cd-258">[Erstellen Sie eine App Service-App](/azure/app-service/), oder verwenden Sie eine App, die für eine andere Lösung erstellt wurde.</span><span class="sxs-lookup"><span data-stu-id="f16cd-258">[Create an App Service app](/azure/app-service/), or use an app created for another  solution.</span></span>

- <span data-ttu-id="f16cd-259">Erwerben Sie einen Domänennamen, und stellen Sie sicher, dass der Zugriff auf die DNS-Registrierung für den Domänenanbieter möglich ist.</span><span class="sxs-lookup"><span data-stu-id="f16cd-259">Purchase a domain name and ensure access to the DNS registry for the domain provider.</span></span>

<span data-ttu-id="f16cd-260">Aktualisieren Sie die DNS-Zonendatei für die Domäne.</span><span class="sxs-lookup"><span data-stu-id="f16cd-260">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="f16cd-261">Azure AD überprüft die Eigentümerschaft für den Namen der benutzerdefinierten Domäne.</span><span class="sxs-lookup"><span data-stu-id="f16cd-261">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="f16cd-262">Verwenden Sie [Azure DNS](/azure/dns/dns-getstarted-portal) für Azure- oder Microsoft 365-Einträge bzw. externe DNS-Einträge in Azure, oder fügen Sie den DNS-Eintrag bei einer [anderen DNS-Registrierungsstelle](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider) hinzu.</span><span class="sxs-lookup"><span data-stu-id="f16cd-262">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

- <span data-ttu-id="f16cd-263">Registrieren Sie eine benutzerdefinierte Domäne bei einer öffentlichen Registrierungsstelle.</span><span class="sxs-lookup"><span data-stu-id="f16cd-263">Register a custom domain with a public registrar.</span></span>

- <span data-ttu-id="f16cd-264">Melden Sie sich an der Domänennamen-Registrierungsstelle für die Domäne an.</span><span class="sxs-lookup"><span data-stu-id="f16cd-264">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="f16cd-265">(Unter Umständen ist ein genehmigter Administrator erforderlich, um die DNS-Updates durchzuführen.)</span><span class="sxs-lookup"><span data-stu-id="f16cd-265">(An approved admin may be required to make DNS updates.)</span></span>

- <span data-ttu-id="f16cd-266">Aktualisieren Sie die DNS-Zonendatei für die Domäne, indem Sie den DNS-Eintrag hinzufügen, der von Azure AD bereitgestellt wurde.</span><span class="sxs-lookup"><span data-stu-id="f16cd-266">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span>

<span data-ttu-id="f16cd-267">Konfigurieren Sie beispielsweise die DNS-Einstellungen für die Stammdomäne „northwindcloud.com“, um DNS-Einträge für „northwindcloud.com“ und „www\.northwindcloud.com“ hinzuzufügen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-267">For example, to add DNS entries for northwindcloud.com and www\.northwindcloud.com, configure DNS settings for the northwindcloud.com root domain.</span></span>

> [!Note]  
> <span data-ttu-id="f16cd-268">Ein Domänenname kann über das [Azure-Portal](/azure/app-service/manage-custom-dns-buy-domain) erworben werden.</span><span class="sxs-lookup"><span data-stu-id="f16cd-268">A domain name may be purchased using the [Azure portal](/azure/app-service/manage-custom-dns-buy-domain).</span></span> <span data-ttu-id="f16cd-269">Um einer Web-App einen benutzerdefinierten DNS-Namen zuzuordnen, muss der [App Service-Plan](https://azure.microsoft.com/pricing/details/app-service/) der Web-App einen kostenpflichtigen Tarif (**Shared**, **Basic**, **Standard** oder **Premium**) aufweisen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-269">To map a custom DNS name to a web app, the web app's [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be a paid tier (**Shared**, **Basic**, **Standard**, or **Premium**).</span></span>

### <a name="create-and-map-cname-and-a-records"></a><span data-ttu-id="f16cd-270">Erstellen und Zuordnen von CNAME- und A-Einträgen</span><span class="sxs-lookup"><span data-stu-id="f16cd-270">Create and map CNAME and A records</span></span>

#### <a name="access-dns-records-with-domain-provider"></a><span data-ttu-id="f16cd-271">Zugreifen auf DNS-Einträge mit Domänenanbieter</span><span class="sxs-lookup"><span data-stu-id="f16cd-271">Access DNS records with domain provider</span></span>

> [!Note]  
>  <span data-ttu-id="f16cd-272">Verwenden Sie Azure DNS, um einen benutzerdefinierten DNS-Namen für Azure-Web-Apps zu konfigurieren.</span><span class="sxs-lookup"><span data-stu-id="f16cd-272">Use Azure DNS to configure a custom DNS name for Azure Web Apps.</span></span> <span data-ttu-id="f16cd-273">Weitere Informationen finden Sie unter [Bereitstellen von benutzerdefinierten Domäneneinstellungen für einen Azure-Dienst mit Azure DNS](/azure/dns/dns-custom-domain).</span><span class="sxs-lookup"><span data-stu-id="f16cd-273">For more information, see [Use Azure DNS to provide custom domain settings for an Azure service](/azure/dns/dns-custom-domain).</span></span>

1. <span data-ttu-id="f16cd-274">Melden Sie sich an der Website des Hauptanbieters an.</span><span class="sxs-lookup"><span data-stu-id="f16cd-274">Sign in to the website of the main provider.</span></span>

2. <span data-ttu-id="f16cd-275">Suchen Sie die Seite für die Verwaltung von DNS-Einträgen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-275">Find the page for managing DNS records.</span></span> <span data-ttu-id="f16cd-276">Jeder Domänenanbieter verfügt über eine eigene Oberfläche für DNS-Einträge.</span><span class="sxs-lookup"><span data-stu-id="f16cd-276">Every domain provider has its own DNS records interface.</span></span> <span data-ttu-id="f16cd-277">Suchen Sie nach Bereichen der Website, die mit **Domänenname**, **DNS** oder **Namenserververwaltung** gekennzeichnet sind.</span><span class="sxs-lookup"><span data-stu-id="f16cd-277">Look for areas of the site labeled **Domain Name**, **DNS**, or **Name Server Management**.</span></span>

<span data-ttu-id="f16cd-278">Die Seite mit den DNS-Einträgen kann unter **Eigene Domänen** angezeigt werden.</span><span class="sxs-lookup"><span data-stu-id="f16cd-278">DNS records page can be viewed in **My domains**.</span></span> <span data-ttu-id="f16cd-279">Suchen Sie nach dem Link mit dem Namen **Zone file** (Zonendatei), **DNS Records** (DNS-Einträge) oder **Advanced configuration** (Erweiterte Konfiguration).</span><span class="sxs-lookup"><span data-stu-id="f16cd-279">Find the link named **Zone file**, **DNS Records**, or **Advanced configuration**.</span></span>

<span data-ttu-id="f16cd-280">Der folgende Screenshot zeigt ein Beispiel für eine Seite mit DNS-Einträgen:</span><span class="sxs-lookup"><span data-stu-id="f16cd-280">The following screenshot is an example of a DNS records page:</span></span>

![Beispielseite mit DNS-Einträgen](media/solution-deployment-guide-geo-distributed/image28.png)

1. <span data-ttu-id="f16cd-282">Wählen Sie unter der Domänennamen-Registrierungsstelle die Option **Add or Create** (Hinzufügen oder erstellen), um einen Eintrag zu erstellen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-282">In Domain Name Registrar, select **Add or Create** to create a record.</span></span> <span data-ttu-id="f16cd-283">Einige Anbieter verfügen über unterschiedliche Links, um unterschiedliche Arten von Einträgen hinzuzufügen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-283">Some providers have different links to add different record types.</span></span> <span data-ttu-id="f16cd-284">Informationen hierzu finden Sie in der Dokumentation des Anbieters.</span><span class="sxs-lookup"><span data-stu-id="f16cd-284">Consult the provider's documentation.</span></span>

2. <span data-ttu-id="f16cd-285">Fügen Sie einen CNAME-Eintrag hinzu, um dem Standardhostnamen der App eine Unterdomäne zuzuordnen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-285">Add a CNAME record to map a subdomain to the app's default hostname.</span></span>

   <span data-ttu-id="f16cd-286">Fügen Sie für das Beispiel mit der Domäne „www\.northwindcloud.com“ einen CNAME-Eintrag hinzu, mit dem der Name der URL `<app_name>.azurewebsites.net` zugeordnet wird.</span><span class="sxs-lookup"><span data-stu-id="f16cd-286">For the www\.northwindcloud.com domain example, add a CNAME record that maps the name to `<app_name>.azurewebsites.net`.</span></span>

<span data-ttu-id="f16cd-287">Nach dem Hinzufügen des CNAME-Eintrags sieht die Seite mit den DNS-Einträgen wie im folgenden Beispiel aus:</span><span class="sxs-lookup"><span data-stu-id="f16cd-287">After adding the CNAME, the DNS records page looks like the following example:</span></span>

![Portalnavigation zur Azure-App](media/solution-deployment-guide-geo-distributed/image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a><span data-ttu-id="f16cd-289">Aktivieren der Zuordnung von CNAME-Einträgen in Azure</span><span class="sxs-lookup"><span data-stu-id="f16cd-289">Enable the CNAME record mapping in Azure</span></span>

1. <span data-ttu-id="f16cd-290">Melden Sie sich auf einer neuen Registerkarte beim Azure-Portal an.</span><span class="sxs-lookup"><span data-stu-id="f16cd-290">In a new tab, sign in to the Azure portal.</span></span>

2. <span data-ttu-id="f16cd-291">Navigieren Sie zu App Services.</span><span class="sxs-lookup"><span data-stu-id="f16cd-291">Go to App Services.</span></span>

3. <span data-ttu-id="f16cd-292">Wählen Sie die Web-App aus.</span><span class="sxs-lookup"><span data-stu-id="f16cd-292">Select web app.</span></span>

4. <span data-ttu-id="f16cd-293">Wählen Sie im linken Navigationsbereich der App-Seite im Azure-Portal die Option **Benutzerdefinierte Domänen**.</span><span class="sxs-lookup"><span data-stu-id="f16cd-293">In the left navigation of the app page in the Azure portal, select **Custom domains**.</span></span>

5. <span data-ttu-id="f16cd-294">Wählen Sie das **+** -Symbol neben der Option **Hostnamen hinzufügen**.</span><span class="sxs-lookup"><span data-stu-id="f16cd-294">Select the **+** icon next to **Add hostname**.</span></span>

6. <span data-ttu-id="f16cd-295">Geben Sie den vollqualifizierten Domänennamen ein, z. B. `www.northwindcloud.com`.</span><span class="sxs-lookup"><span data-stu-id="f16cd-295">Type the fully qualified domain name, like `www.northwindcloud.com`.</span></span>

7. <span data-ttu-id="f16cd-296">Wählen Sie **Überprüfen** aus.</span><span class="sxs-lookup"><span data-stu-id="f16cd-296">Select **Validate**.</span></span>

8. <span data-ttu-id="f16cd-297">Fügen Sie, falls angegeben, den DNS-Einträgen der Domänennamen-Registrierungsstelle weitere Einträge mit anderen Typen (`A` oder `TXT`) hinzu.</span><span class="sxs-lookup"><span data-stu-id="f16cd-297">If indicated, add additional records of other types (`A` or `TXT`) to the domain name registrars DNS records.</span></span> <span data-ttu-id="f16cd-298">Azure stellt die Werte und Typen dieser Einträge bereit:</span><span class="sxs-lookup"><span data-stu-id="f16cd-298">Azure will provide the values and types of these records:</span></span>

   <span data-ttu-id="f16cd-299">a.</span><span class="sxs-lookup"><span data-stu-id="f16cd-299">a.</span></span>  <span data-ttu-id="f16cd-300">Ein **A**-Eintrag, um die IP-Adresse der App zuzuordnen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-300">An **A** record to map to the app's IP address.</span></span>

   <span data-ttu-id="f16cd-301">b.</span><span class="sxs-lookup"><span data-stu-id="f16cd-301">b.</span></span>  <span data-ttu-id="f16cd-302">Ein **TXT**-Eintrag, um den Standardhostnamen der App `<app_name>.azurewebsites.net` zuzuordnen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-302">A **TXT** record to map to the app's default hostname `<app_name>.azurewebsites.net`.</span></span> <span data-ttu-id="f16cd-303">App Service nutzt diesen Eintrag nur während der Konfiguration, um die Eigentümerschaft der benutzerdefinierten Domäne zu überprüfen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-303">App Service uses this record only at configuration time to verify custom domain ownership.</span></span> <span data-ttu-id="f16cd-304">Löschen Sie den TXT-Eintrag, nachdem die Überprüfung abgeschlossen ist.</span><span class="sxs-lookup"><span data-stu-id="f16cd-304">After verification, delete the TXT record.</span></span>

9. <span data-ttu-id="f16cd-305">Schließen Sie diese Aufgabe auf der Registerkarte für die Domänenregistrierungsstelle ab, und führen Sie die Überprüfung erneut durch, bis die Schaltfläche **Hostnamen hinzufügen** aktiviert wird.</span><span class="sxs-lookup"><span data-stu-id="f16cd-305">Complete this task in the domain registrar tab and revalidate until the **Add hostname** button is activated.</span></span>

10. <span data-ttu-id="f16cd-306">Stellen Sie sicher, dass der **Typ des Hostnamenseintrags** auf **CNAME** (www.beispiel.com oder eine beliebige Unterdomäne) festgelegt ist.</span><span class="sxs-lookup"><span data-stu-id="f16cd-306">Make sure that **Hostname record type** is set to **CNAME** (www.example.com or any subdomain).</span></span>

11. <span data-ttu-id="f16cd-307">Wählen Sie **Hostnamen hinzufügen**.</span><span class="sxs-lookup"><span data-stu-id="f16cd-307">Select **Add hostname**.</span></span>

12. <span data-ttu-id="f16cd-308">Geben Sie den vollqualifizierten Domänennamen ein, z. B. `northwindcloud.com`.</span><span class="sxs-lookup"><span data-stu-id="f16cd-308">Type the fully qualified domain name, like `northwindcloud.com`.</span></span>

13. <span data-ttu-id="f16cd-309">Wählen Sie **Überprüfen** aus.</span><span class="sxs-lookup"><span data-stu-id="f16cd-309">Select **Validate**.</span></span> <span data-ttu-id="f16cd-310">Die Schaltfläche **Hinzufügen** wird aktiviert.</span><span class="sxs-lookup"><span data-stu-id="f16cd-310">The **Add** is activated.</span></span>

14. <span data-ttu-id="f16cd-311">Stellen Sie sicher, dass die Option für den **Typ des Hostnamenseintrags** auf **A-Eintrag** (beispiel.com) festgelegt ist.</span><span class="sxs-lookup"><span data-stu-id="f16cd-311">Make sure that **Hostname record type** is set to **A record** (example.com).</span></span>

15. <span data-ttu-id="f16cd-312">Wählen Sie **Hostnamen hinzufügen**.</span><span class="sxs-lookup"><span data-stu-id="f16cd-312">**Add hostname**.</span></span>

    <span data-ttu-id="f16cd-313">Unter Umständen dauert es eine Weile, bis die neuen Hostnamen auf der Seite **Benutzerdefinierte Domänen** der App angezeigt werden.</span><span class="sxs-lookup"><span data-stu-id="f16cd-313">It might take some time for the new hostnames to be reflected in the app's **Custom domains** page.</span></span> <span data-ttu-id="f16cd-314">Aktualisieren Sie den Browser, um die Daten zu aktualisieren.</span><span class="sxs-lookup"><span data-stu-id="f16cd-314">Try refreshing the browser to update the data.</span></span>
  
    ![Benutzerdefinierte Domänen](media/solution-deployment-guide-geo-distributed/image31.png) 
  
    <span data-ttu-id="f16cd-316">Bei einem Fehler wird unten auf der Seite eine Benachrichtigung mit einem Überprüfungsfehler angezeigt.</span><span class="sxs-lookup"><span data-stu-id="f16cd-316">If there's an error, a verification error notification will appear at the bottom of the page.</span></span> ![Fehler bei der Domänenüberprüfung](media/solution-deployment-guide-geo-distributed/image32.png)

> [!Note]  
>  <span data-ttu-id="f16cd-318">Sie können die obigen Schritte wiederholen, um eine Platzhalterdomäne zuzuordnen (\*.northwindcloud.com).</span><span class="sxs-lookup"><span data-stu-id="f16cd-318">The above steps may be repeated to map a wildcard domain (\*.northwindcloud.com).</span></span> <span data-ttu-id="f16cd-319">Dies ermöglicht das Hinzufügen von weiteren Unterdomänen zu diesem App Service, ohne dass jeweils ein separater CNAME-Eintrag erstellt werden muss.</span><span class="sxs-lookup"><span data-stu-id="f16cd-319">This allows the addition of any additional subdomains to this app service without having to create a separate CNAME record for each one.</span></span> <span data-ttu-id="f16cd-320">Befolgen Sie die Anleitung der Registrierungsstelle, um diese Einstellung zu konfigurieren.</span><span class="sxs-lookup"><span data-stu-id="f16cd-320">Follow the registrar instructions to configure this setting.</span></span>

#### <a name="test-in-a-browser"></a><span data-ttu-id="f16cd-321">Testen in einem Browser</span><span class="sxs-lookup"><span data-stu-id="f16cd-321">Test in a browser</span></span>

<span data-ttu-id="f16cd-322">Navigieren Sie zu den DNS-Namen, die Sie zuvor konfiguriert haben (z. B. `northwindcloud.com` oben `www.northwindcloud.com`).</span><span class="sxs-lookup"><span data-stu-id="f16cd-322">Browse to the DNS name(s) configured earlier (for example, `northwindcloud.com` or `www.northwindcloud.com`).</span></span>

## <a name="part-3-bind-a-custom-ssl-cert"></a><span data-ttu-id="f16cd-323">Teil 3: Binden eines benutzerdefinierten SSL-Zertifikats</span><span class="sxs-lookup"><span data-stu-id="f16cd-323">Part 3: Bind a custom SSL cert</span></span>

<span data-ttu-id="f16cd-324">In diesem Teil führen wir Folgendes durch:</span><span class="sxs-lookup"><span data-stu-id="f16cd-324">In this part, we will:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="f16cd-325">Binden des benutzerdefinierten SSL-Zertifikats an App Service.</span><span class="sxs-lookup"><span data-stu-id="f16cd-325">Bind the custom SSL certificate to App Service.</span></span>
> - <span data-ttu-id="f16cd-326">Erzwingen von HTTPS für die App.</span><span class="sxs-lookup"><span data-stu-id="f16cd-326">Enforce HTTPS for the app.</span></span>
> - <span data-ttu-id="f16cd-327">Automatisieren der SSL-Zertifikatbindung mit Skripts.</span><span class="sxs-lookup"><span data-stu-id="f16cd-327">Automate SSL certificate binding with scripts.</span></span>

> [!Note]  
> <span data-ttu-id="f16cd-328">Rufen Sie, falls erforderlich, im Azure-Portal ein SSL-Kundenzertifikat ab, und binden Sie es an die Web-App.</span><span class="sxs-lookup"><span data-stu-id="f16cd-328">If needed, obtain a customer SSL certificate in the Azure portal and bind it to the web app.</span></span> <span data-ttu-id="f16cd-329">Weitere Informationen finden Sie im [Tutorial „App Service-Zertifikate“](/azure/app-service/web-sites-purchase-ssl-web-site).</span><span class="sxs-lookup"><span data-stu-id="f16cd-329">For more information, see the [App Service Certificates tutorial](/azure/app-service/web-sites-purchase-ssl-web-site).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="f16cd-330">Voraussetzungen</span><span class="sxs-lookup"><span data-stu-id="f16cd-330">Prerequisites</span></span>

<span data-ttu-id="f16cd-331">Führen Sie im Rahmen dieser Lösung folgende Schritte aus:</span><span class="sxs-lookup"><span data-stu-id="f16cd-331">To complete this  solution:</span></span>

- <span data-ttu-id="f16cd-332">[Erstellen einer App Service-App](/azure/app-service/).</span><span class="sxs-lookup"><span data-stu-id="f16cd-332">[Create an App Service app.](/azure/app-service/)</span></span>
- <span data-ttu-id="f16cd-333">[Zuordnen eines benutzerdefinierten DNS-Namens zu Ihrer Web-App](/azure/app-service/app-service-web-tutorial-custom-domain).</span><span class="sxs-lookup"><span data-stu-id="f16cd-333">[Map a custom DNS name to your web app.](/azure/app-service/app-service-web-tutorial-custom-domain)</span></span>
- <span data-ttu-id="f16cd-334">Beschaffen eines SSL-Zertifikats von einer vertrauenswürdigen Zertifizierungsstelle und Verwenden des Schlüssels zum Signieren der Anforderung.</span><span class="sxs-lookup"><span data-stu-id="f16cd-334">Acquire an SSL certificate from a trusted certificate authority and use the key to sign the request.</span></span>

### <a name="requirements-for-your-ssl-certificate"></a><span data-ttu-id="f16cd-335">Anforderungen an Ihr SSL-Zertifikat</span><span class="sxs-lookup"><span data-stu-id="f16cd-335">Requirements for your SSL certificate</span></span>

<span data-ttu-id="f16cd-336">Das Zertifikat muss sämtliche der folgenden Anforderungen erfüllen, damit Ihr Zertifikat in App Service verwendet werden kann:</span><span class="sxs-lookup"><span data-stu-id="f16cd-336">To use a certificate in App Service, the certificate must meet all the following requirements:</span></span>

- <span data-ttu-id="f16cd-337">Von einer vertrauenswürdigen Zertifizierungsstelle signiert.</span><span class="sxs-lookup"><span data-stu-id="f16cd-337">Signed by a trusted certificate authority.</span></span>

- <span data-ttu-id="f16cd-338">Als kennwortgeschützte PFX-Datei exportiert.</span><span class="sxs-lookup"><span data-stu-id="f16cd-338">Exported as a password-protected PFX file.</span></span>

- <span data-ttu-id="f16cd-339">Enthält einen privaten Schlüssel mit mindestens 2048 Bit.</span><span class="sxs-lookup"><span data-stu-id="f16cd-339">Contains private key at least 2048 bits long.</span></span>

- <span data-ttu-id="f16cd-340">Enthält alle Zwischenzertifikate in der Zertifikatkette.</span><span class="sxs-lookup"><span data-stu-id="f16cd-340">Contains all intermediate certificates in the certificate chain.</span></span>

> [!Note]  
> <span data-ttu-id="f16cd-341">**ECC-Zertifikate (Elliptic Curve Cryptography, Kryptografie für elliptische Kurven)** funktionieren mit App Service, werden in diesem Leitfaden aber nicht behandelt.</span><span class="sxs-lookup"><span data-stu-id="f16cd-341">**Elliptic Curve Cryptography (ECC) certificates** work with App Service but aren't included in this guide.</span></span> <span data-ttu-id="f16cd-342">Wenden Sie sich an eine Zertifizierungsstelle, um Hilfe beim Erstellen von ECC-Zertifikaten zu erhalten.</span><span class="sxs-lookup"><span data-stu-id="f16cd-342">Consult a certificate authority for assistance in creating ECC certificates.</span></span>

#### <a name="prepare-the-web-app"></a><span data-ttu-id="f16cd-343">Vorbereiten der Web-App</span><span class="sxs-lookup"><span data-stu-id="f16cd-343">Prepare the web app</span></span>

<span data-ttu-id="f16cd-344">Um ein benutzerdefiniertes SSL-Zertifikat an die Web-App zu binden, muss der [App Service-Plan](https://azure.microsoft.com/pricing/details/app-service/) den Tarif **Basic**, **Standard** oder **Premium** aufweisen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-344">To bind a custom SSL certificate to the web app, the [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be in the **Basic**, **Standard**, or **Premium** tier.</span></span>

#### <a name="sign-in-to-azure"></a><span data-ttu-id="f16cd-345">Anmelden bei Azure</span><span class="sxs-lookup"><span data-stu-id="f16cd-345">Sign in to Azure</span></span>

1. <span data-ttu-id="f16cd-346">Öffnen Sie das [Azure-Portal](https://portal.azure.com/), und navigieren Sie zur Web-App.</span><span class="sxs-lookup"><span data-stu-id="f16cd-346">Open the [Azure portal](https://portal.azure.com/) and go to the web app.</span></span>

2. <span data-ttu-id="f16cd-347">Wählen Sie im linken Menü **App Services** und anschließend den Namen der Web-App aus.</span><span class="sxs-lookup"><span data-stu-id="f16cd-347">From the left menu, select **App Services**, and then select the web app name.</span></span>

![Auswählen der Web-App im Azure-Portal](media/solution-deployment-guide-geo-distributed/image33.png)

#### <a name="check-the-pricing-tier"></a><span data-ttu-id="f16cd-349">Überprüfen des Tarifs</span><span class="sxs-lookup"><span data-stu-id="f16cd-349">Check the pricing tier</span></span>

1. <span data-ttu-id="f16cd-350">Scrollen Sie im linken Navigationsbereich auf der Seite mit der Web-App zum Abschnitt **Einstellungen**, und wählen Sie **Hochskalieren (App Service-Plan)** .</span><span class="sxs-lookup"><span data-stu-id="f16cd-350">In the left-hand navigation of the web app page, scroll to the **Settings** section and select **Scale up (App Service plan)**.</span></span>

    ![Menü für das Hochskalieren in der Web-App](media/solution-deployment-guide-geo-distributed/image34.png)

1. <span data-ttu-id="f16cd-352">Stellen Sie sicher, dass für die Web-App nicht der Tarif **Free** oder **Shared** festgelegt ist.</span><span class="sxs-lookup"><span data-stu-id="f16cd-352">Ensure the web app isn't in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="f16cd-353">Der aktuelle Tarif der Web-App ist mit einem dunkelblauen Rahmen hervorgehoben.</span><span class="sxs-lookup"><span data-stu-id="f16cd-353">The web app's current tier is highlighted in a dark blue box.</span></span>

    ![Überprüfen des Tarifs in der Web-App](media/solution-deployment-guide-geo-distributed/image35.png)

<span data-ttu-id="f16cd-355">Benutzerdefiniertes SSL wird im Tarif **Free** oder **Shared** nicht unterstützt.</span><span class="sxs-lookup"><span data-stu-id="f16cd-355">Custom SSL isn't supported in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="f16cd-356">Führen Sie die Schritte im nächsten Abschnitt aus, um einen höheren Tarif auszuwählen, oder verwenden Sie die Seite **Tarif wählen**, und springen Sie zu [Hochladen und Binden Ihres SSL-Zertifikats](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="f16cd-356">To upscale, follow the steps in the next section or the **Choose your pricing tier** page and skip to [Upload and bind your SSL certificate](/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

#### <a name="scale-up-your-app-service-plan"></a><span data-ttu-id="f16cd-357">Hochskalieren Ihres App Service-Plans</span><span class="sxs-lookup"><span data-stu-id="f16cd-357">Scale up your App Service plan</span></span>

1. <span data-ttu-id="f16cd-358">Wählen Sie den Tarif **Basic**, **Standard** oder **Premium** aus.</span><span class="sxs-lookup"><span data-stu-id="f16cd-358">Select one of the **Basic**, **Standard**, or **Premium** tiers.</span></span>

2. <span data-ttu-id="f16cd-359">Wählen Sie **Auswählen**.</span><span class="sxs-lookup"><span data-stu-id="f16cd-359">Select **Select**.</span></span>

![Auswählen des Tarifs für Ihre Web-App](media/solution-deployment-guide-geo-distributed/image36.png)

<span data-ttu-id="f16cd-361">Der Skalierungsvorgang ist abgeschlossen, wenn die Benachrichtigung angezeigt wird.</span><span class="sxs-lookup"><span data-stu-id="f16cd-361">The scale operation is complete when notification is displayed.</span></span>

![Benachrichtigung zum Hochskalieren](media/solution-deployment-guide-geo-distributed/image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a><span data-ttu-id="f16cd-363">Binden Ihres SSL-Zertifikats und Zusammenführen von Zwischenzertifikaten</span><span class="sxs-lookup"><span data-stu-id="f16cd-363">Bind your SSL certificate and merge intermediate certificates</span></span>

<span data-ttu-id="f16cd-364">Führen Sie mehrere Zertifikate in einer Kette zusammen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-364">Merge multiple certificates in the chain.</span></span>

1. <span data-ttu-id="f16cd-365">**Öffnen Sie alle Zertifikate**, die Sie erhalten haben, in einem Text-Editor.</span><span class="sxs-lookup"><span data-stu-id="f16cd-365">**Open each certificate** you received in a text editor.</span></span>

2. <span data-ttu-id="f16cd-366">Erstellen Sie eine Datei für das zusammengeführte Zertifikat mit dem Namen *mergedcertificate.crt*.</span><span class="sxs-lookup"><span data-stu-id="f16cd-366">Create a file for the merged certificate called *mergedcertificate.crt*.</span></span> <span data-ttu-id="f16cd-367">Kopieren Sie den Inhalt der einzelnen Zertifikate in einem Text-Editor in diese Datei.</span><span class="sxs-lookup"><span data-stu-id="f16cd-367">In a text editor, copy the content of each certificate into this file.</span></span> <span data-ttu-id="f16cd-368">Die Reihenfolge Ihrer Zertifikate sollte der Reihenfolge in der Zertifikatkette folgen, beginnend mit Ihrem Zertifikat und endend mit dem Stammzertifikat.</span><span class="sxs-lookup"><span data-stu-id="f16cd-368">The order of your certificates should follow the order in the certificate chain, beginning with your certificate and ending with the root certificate.</span></span> <span data-ttu-id="f16cd-369">Dies sieht in etwa wie im folgenden Beispiel aus:</span><span class="sxs-lookup"><span data-stu-id="f16cd-369">It looks like the following example:</span></span>

    ```Text

    -----BEGIN CERTIFICATE-----

    <your entire Base64 encoded SSL certificate>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 1>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 2>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded root certificate>

    -----END CERTIFICATE-----
    ```

#### <a name="export-certificate-to-pfx"></a><span data-ttu-id="f16cd-370">Exportieren des Zertifikats als PFX-Datei</span><span class="sxs-lookup"><span data-stu-id="f16cd-370">Export certificate to PFX</span></span>

<span data-ttu-id="f16cd-371">Exportieren Sie das zusammengeführte SSL-Zertifikat mit dem privaten Schlüssel, der vom Zertifikat generiert wurde.</span><span class="sxs-lookup"><span data-stu-id="f16cd-371">Export the merged SSL certificate with the private key generated by the certificate.</span></span>

<span data-ttu-id="f16cd-372">Eine Datei für den privaten Schlüssel wird per OpenSSL erstellt.</span><span class="sxs-lookup"><span data-stu-id="f16cd-372">A private key file is created via OpenSSL.</span></span> <span data-ttu-id="f16cd-373">Führen Sie zum Exportieren des Zertifikats als PFX-Datei den folgenden Befehl aus, und ersetzen Sie die Platzhalter `<private-key-file>` und `<merged-certificate-file>` durch den Pfad zum privaten Schlüssel bzw. die zusammengeführte Zertifikatdatei:</span><span class="sxs-lookup"><span data-stu-id="f16cd-373">To export the certificate to PFX, run the following command and replace the placeholders `<private-key-file>` and `<merged-certificate-file>` with the private key path and the merged certificate file:</span></span>

```powershell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

<span data-ttu-id="f16cd-374">Definieren Sie bei entsprechender Aufforderung ein Exportkennwort zum späteren Hochladen Ihres SSL-Zertifikats in App Service.</span><span class="sxs-lookup"><span data-stu-id="f16cd-374">When prompted, define an export password for uploading your SSL certificate to App Service later.</span></span>

<span data-ttu-id="f16cd-375">Gehen Sie wie folgt vor, wenn IIS oder **Certreq.exe** zum Generieren der Zertifikatanforderung verwendet werden: Installieren Sie das Zertifikat auf einem lokalen Computer, und [exportieren Sie das Zertifikat dann nach PFX](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).</span><span class="sxs-lookup"><span data-stu-id="f16cd-375">When IIS or **Certreq.exe** are used to generate the certificate request, install the certificate to a local machine and then [export the certificate to PFX](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).</span></span>

#### <a name="upload-the-ssl-certificate"></a><span data-ttu-id="f16cd-376">Hochladen des SSL-Zertifikats</span><span class="sxs-lookup"><span data-stu-id="f16cd-376">Upload the SSL certificate</span></span>

1. <span data-ttu-id="f16cd-377">Wählen Sie im linken Navigationsbereich der Web-App die Option **SSL-Einstellungen**.</span><span class="sxs-lookup"><span data-stu-id="f16cd-377">Select **SSL settings** in the left navigation of the web app.</span></span>

2. <span data-ttu-id="f16cd-378">Wählen Sie **Zertifikat hochladen**.</span><span class="sxs-lookup"><span data-stu-id="f16cd-378">Select **Upload Certificate**.</span></span>

3. <span data-ttu-id="f16cd-379">Wählen Sie unter **PFX-Zertifikatdatei** die PFX-Datei aus.</span><span class="sxs-lookup"><span data-stu-id="f16cd-379">In **PFX Certificate File**, select PFX file.</span></span>

4. <span data-ttu-id="f16cd-380">Geben Sie unter **Zertifikatkennwort** das Kennwort ein, das beim Exportieren der PFX-Datei erstellt wurde.</span><span class="sxs-lookup"><span data-stu-id="f16cd-380">In **Certificate password**, type the password created when exporting the PFX file.</span></span>

5. <span data-ttu-id="f16cd-381">Wählen Sie die Option **Hochladen**.</span><span class="sxs-lookup"><span data-stu-id="f16cd-381">Select **Upload**.</span></span>

    ![Hochladen des SSL-Zertifikats](media/solution-deployment-guide-geo-distributed/image38.png)

<span data-ttu-id="f16cd-383">Wenn der Upload des Zertifikats in App Service abgeschlossen ist, wird es auf der Seite **SSL-Einstellungen** angezeigt.</span><span class="sxs-lookup"><span data-stu-id="f16cd-383">When App Service finishes uploading the certificate, it appears in the **SSL settings** page.</span></span>

![SSL-Einstellungen](media/solution-deployment-guide-geo-distributed/image39.png)

#### <a name="bind-your-ssl-certificate"></a><span data-ttu-id="f16cd-385">Binden Ihres SSL-Zertifikats</span><span class="sxs-lookup"><span data-stu-id="f16cd-385">Bind your SSL certificate</span></span>

1. <span data-ttu-id="f16cd-386">Wählen Sie im Abschnitt **SSL-Bindungen** die Option **Bindung hinzufügen**.</span><span class="sxs-lookup"><span data-stu-id="f16cd-386">In the **SSL bindings** section, select **Add binding**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="f16cd-387">Wenn das Zertifikat hochgeladen wurde, aber in der Dropdownliste **Hostname** nicht als Domänenname angezeigt wird, können Sie versuchen, die Browserseite zu aktualisieren.</span><span class="sxs-lookup"><span data-stu-id="f16cd-387">If the certificate has been uploaded, but doesn't appear in domain name(s) in the **Hostname** dropdown, try refreshing the browser page.</span></span>

2. <span data-ttu-id="f16cd-388">Wählen Sie auf der Seite **SSL-Bindung hinzufügen** aus den Dropdownlisten den zu schützenden Domänennamen und das zu verwendende Zertifikat aus.</span><span class="sxs-lookup"><span data-stu-id="f16cd-388">In the **Add SSL Binding** page, use the drop downs to select the domain name to secure and the certificate to use.</span></span>

3. <span data-ttu-id="f16cd-389">Wählen Sie unter **SSL-Typ** aus, ob SSL auf der [**Servernamensanzeige (Server Name Indication, SNI)** ](https://en.wikipedia.org/wiki/Server_Name_Indication) oder der IP basieren soll.</span><span class="sxs-lookup"><span data-stu-id="f16cd-389">In **SSL Type**, select whether to use [**Server Name Indication (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) or IP-based SSL.</span></span>

    - <span data-ttu-id="f16cd-390">**SNI-basiertes SSL**: Es können mehrere SNI-basierte SSL-Bindungen hinzugefügt werden.</span><span class="sxs-lookup"><span data-stu-id="f16cd-390">**SNI-based SSL**: Multiple SNI-based SSL bindings may be added.</span></span> <span data-ttu-id="f16cd-391">Bei dieser Option können mehrere zur selben IP-Adresse zugehörige Domänen durch mehrere SSL-Zertifikate geschützt werden.</span><span class="sxs-lookup"><span data-stu-id="f16cd-391">This option allows multiple SSL certificates to secure multiple domains on the same IP address.</span></span> <span data-ttu-id="f16cd-392">Die meisten modernen Browser (einschließlich Internet Explorer, Chrome, Firefox und Opera) unterstützen SNI (ausführlichere Informationen zur Browserunterstützung finden Sie unter [Servernamensanzeige](https://wikipedia.org/wiki/Server_Name_Indication)).</span><span class="sxs-lookup"><span data-stu-id="f16cd-392">Most modern browsers (including Internet Explorer, Chrome, Firefox, and Opera) support SNI (find more comprehensive browser support information at [Server Name Indication](https://wikipedia.org/wiki/Server_Name_Indication)).</span></span>

    - <span data-ttu-id="f16cd-393">**IP-basiertes SSL:** Ggf. kann nur eine IP-basierte SSL-Bindung hinzugefügt werden.</span><span class="sxs-lookup"><span data-stu-id="f16cd-393">**IP-based SSL**: Only one IP-based SSL binding may be added.</span></span> <span data-ttu-id="f16cd-394">Bei dieser Option kann eine dedizierte öffentliche IP-Adresse nur durch ein SSL-Zertifikat geschützt werden.</span><span class="sxs-lookup"><span data-stu-id="f16cd-394">This option allows only one SSL certificate to secure a dedicated public IP address.</span></span> <span data-ttu-id="f16cd-395">Schützen Sie alle Domänen mit demselben SSL-Zertifikat, wenn Sie den Schutz für mehrere Domänen einrichten möchten.</span><span class="sxs-lookup"><span data-stu-id="f16cd-395">To secure multiple domains, secure them all using the same SSL certificate.</span></span> <span data-ttu-id="f16cd-396">IP-basiertes SSL ist die herkömmliche Option für SSL-Bindungen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-396">IP-based SSL is the traditional option for SSL binding.</span></span>

4. <span data-ttu-id="f16cd-397">Wählen Sie **Bindung hinzufügen**.</span><span class="sxs-lookup"><span data-stu-id="f16cd-397">Select **Add Binding**.</span></span>

    ![Hinzufügen von SSL-Bindung](media/solution-deployment-guide-geo-distributed/image40.png)

<span data-ttu-id="f16cd-399">Wenn der Upload des Zertifikats in App Service abgeschlossen ist, wird es in den Abschnitten **SSL-Bindungen** angezeigt.</span><span class="sxs-lookup"><span data-stu-id="f16cd-399">When App Service finishes uploading the certificate, it appears in the **SSL bindings** sections.</span></span>

![Upload der SSL-Bindungen abgeschlossen](media/solution-deployment-guide-geo-distributed/image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a><span data-ttu-id="f16cd-401">Neuzuordnen des A-Eintrags für IP-SSL</span><span class="sxs-lookup"><span data-stu-id="f16cd-401">Remap the A record for IP SSL</span></span>

<span data-ttu-id="f16cd-402">Wenn IP-basiertes SSL in der Web-App nicht verwendet wird, können Sie mit [Testen von HTTPS für Ihre benutzerdefinierte Domäne](/azure/app-service/app-service-web-tutorial-custom-ssl) fortfahren.</span><span class="sxs-lookup"><span data-stu-id="f16cd-402">If IP-based SSL isn't used in the web app, skip to [Test HTTPS for your custom domain](/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

<span data-ttu-id="f16cd-403">Standardmäßig verwendet die Web-App eine freigegebene öffentliche IP-Adresse.</span><span class="sxs-lookup"><span data-stu-id="f16cd-403">By default, the web app uses a shared public IP address.</span></span> <span data-ttu-id="f16cd-404">Wenn das Zertifikat per IP-basiertem SSL gebunden ist, erstellt App Service eine neue und dedizierte IP-Adresse für die Web-App.</span><span class="sxs-lookup"><span data-stu-id="f16cd-404">When the certificate is bound with IP-based SSL, App Service creates a new and dedicated IP address for the web app.</span></span>

<span data-ttu-id="f16cd-405">Wenn ein A-Eintrag der Web-App zugeordnet wird, muss die Domänenregistrierung mit der dedizierten IP-Adresse aktualisiert werden.</span><span class="sxs-lookup"><span data-stu-id="f16cd-405">When an A record is mapped to the web app, the domain registry must be updated with the dedicated IP address.</span></span>

<span data-ttu-id="f16cd-406">Die Seite **Benutzerdefinierte Domäne** wird mit der neuen, dedizierten IP-Adresse aktualisiert.</span><span class="sxs-lookup"><span data-stu-id="f16cd-406">The **Custom domain** page is updated with the new, dedicated IP address.</span></span> <span data-ttu-id="f16cd-407">[Kopieren Sie diese IP-Adresse](/azure/app-service/app-service-web-tutorial-custom-domain), und [ordnen Sie dieser neuen IP-Adresse dann den A-Eintrag erneut zu](/azure/app-service/app-service-web-tutorial-custom-domain).</span><span class="sxs-lookup"><span data-stu-id="f16cd-407">Copy this [IP address](/azure/app-service/app-service-web-tutorial-custom-domain), then remap the [A record](/azure/app-service/app-service-web-tutorial-custom-domain) to this new IP address.</span></span>

#### <a name="test-https"></a><span data-ttu-id="f16cd-408">Testen von HTTPS</span><span class="sxs-lookup"><span data-stu-id="f16cd-408">Test HTTPS</span></span>

<span data-ttu-id="f16cd-409">Navigieren Sie in unterschiedlichen Browsern zu `https://<your.custom.domain>`, um sicherzustellen, dass die Web-App bereitgestellt wurde.</span><span class="sxs-lookup"><span data-stu-id="f16cd-409">In different browsers, go to `https://<your.custom.domain>` to ensure the web app is served.</span></span>

![Navigieren zur Web-App](media/solution-deployment-guide-geo-distributed/image42.png)

> [!Note]  
> <span data-ttu-id="f16cd-411">Wenn Zertifikatüberprüfungsfehler auftreten, kann ein selbstsigniertes Zertifikat der Grund sein, oder beim Exportieren in die PFX-Datei wurden Zwischenzertifikate nicht abgeschlossen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-411">If certificate validation errors occur, a self-signed certificate may be the cause, or intermediate certificates may have been left off when exporting to the PFX file.</span></span>

#### <a name="enforce-https"></a><span data-ttu-id="f16cd-412">Erzwingen von HTTPS</span><span class="sxs-lookup"><span data-stu-id="f16cd-412">Enforce HTTPS</span></span>

<span data-ttu-id="f16cd-413">Standardmäßig haben alle Benutzer per HTTP Zugriff auf die Web-App.</span><span class="sxs-lookup"><span data-stu-id="f16cd-413">By default, anyone can access the web app using HTTP.</span></span> <span data-ttu-id="f16cd-414">Alle HTTP-Anforderungen an den HTTPS-Port werden ggf. umgeleitet.</span><span class="sxs-lookup"><span data-stu-id="f16cd-414">All HTTP requests to the HTTPS port may be redirected.</span></span>

<span data-ttu-id="f16cd-415">Wählen Sie auf der Web-App-Seite die Option **SSL-Einstellungen**.</span><span class="sxs-lookup"><span data-stu-id="f16cd-415">In the web app page, select **SL settings**.</span></span> <span data-ttu-id="f16cd-416">Wählen Sie anschließend für **Nur HTTPS** die Option **Ein**.</span><span class="sxs-lookup"><span data-stu-id="f16cd-416">Then, in **HTTPS Only**, select **On**.</span></span>

![Erzwingen von HTTPS](media/solution-deployment-guide-geo-distributed/image43.png)

<span data-ttu-id="f16cd-418">Navigieren Sie nach Abschluss des Vorgangs zu einer beliebigen HTTP-URL, die auf die App verweist.</span><span class="sxs-lookup"><span data-stu-id="f16cd-418">When the operation is complete, go to any of the HTTP URLs that point to the app.</span></span> <span data-ttu-id="f16cd-419">Beispiel:</span><span class="sxs-lookup"><span data-stu-id="f16cd-419">For example:</span></span>

- <span data-ttu-id="f16cd-420">https://<Name_der_App>.azurewebsites.net</span><span class="sxs-lookup"><span data-stu-id="f16cd-420">https://<app_name>.azurewebsites.net</span></span>
- `https://northwindcloud.com`
- `https://www.northwindcloud.com`

#### <a name="enforce-tls-1112"></a><span data-ttu-id="f16cd-421">Erzwingen von TLS 1.1/1.2</span><span class="sxs-lookup"><span data-stu-id="f16cd-421">Enforce TLS 1.1/1.2</span></span>

<span data-ttu-id="f16cd-422">Die App lässt standardmäßig [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0 zu. Dies ist gemäß den Branchenstandards (z. B. [PCI-DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)) nicht mehr sicher.</span><span class="sxs-lookup"><span data-stu-id="f16cd-422">The app allows [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0 by default, which is no longer considered secure by industry standards (like [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)).</span></span> <span data-ttu-id="f16cd-423">Gehen Sie wie folgt vor, um höhere TLS-Versionen zu erzwingen:</span><span class="sxs-lookup"><span data-stu-id="f16cd-423">To enforce higher TLS versions, follow these steps:</span></span>

1. <span data-ttu-id="f16cd-424">Klicken Sie im linken Navigationsbereich der Web-App-Seite auf **SSL-Einstellungen**.</span><span class="sxs-lookup"><span data-stu-id="f16cd-424">In the web app page, in the left navigation, select **SSL settings**.</span></span>

2. <span data-ttu-id="f16cd-425">Wählen Sie unter **TLS-Version** die TLS-Mindestversion aus.</span><span class="sxs-lookup"><span data-stu-id="f16cd-425">In **TLS version**, select the minimum TLS version.</span></span>

    ![Erzwingen von TLS 1.1 oder 1.2](media/solution-deployment-guide-geo-distributed/image44.png)

### <a name="create-a-traffic-manager-profile"></a><span data-ttu-id="f16cd-427">Erstellen eines Traffic Manager-Profils</span><span class="sxs-lookup"><span data-stu-id="f16cd-427">Create a Traffic Manager profile</span></span>

1. <span data-ttu-id="f16cd-428">Wählen Sie **Ressource erstellen** > **Netzwerk** > **Traffic Manager-Profil** > **Erstellen**.</span><span class="sxs-lookup"><span data-stu-id="f16cd-428">Select **Create a resource** > **Networking** > **Traffic Manager profile** > **Create**.</span></span>

2. <span data-ttu-id="f16cd-429">Füllen Sie den Bereich **Traffic Manager-Profil erstellen** wie folgt aus:</span><span class="sxs-lookup"><span data-stu-id="f16cd-429">In the **Create Traffic Manager profile**, complete as follows:</span></span>

    1. <span data-ttu-id="f16cd-430">Geben Sie im Feld **Name** einen Namen für das Profil ein.</span><span class="sxs-lookup"><span data-stu-id="f16cd-430">In **Name**, provide a name for the profile.</span></span> <span data-ttu-id="f16cd-431">Dieser Name muss innerhalb der Zone „trafficmanager.net“ eindeutig sein und ergibt den DNS-Namen „trafficmanager.net“, der für den Zugriff auf das Traffic Manager-Profil verwendet wird.</span><span class="sxs-lookup"><span data-stu-id="f16cd-431">This name needs to be unique within the traffic manager.net zone and results in the DNS name, trafficmanager.net, which is used to access the Traffic Manager profile.</span></span>

    2. <span data-ttu-id="f16cd-432">Wählen Sie unter **Routingmethode** die **geografische Routingmethode** aus.</span><span class="sxs-lookup"><span data-stu-id="f16cd-432">In **Routing method**, select the **Geographic routing method**.</span></span>

    3. <span data-ttu-id="f16cd-433">Wählen Sie unter **Abonnement** das Abonnement aus, unter dem Sie dieses Profil erstellen möchten.</span><span class="sxs-lookup"><span data-stu-id="f16cd-433">In **Subscription**, select the subscription under which to create this profile.</span></span>

    4. <span data-ttu-id="f16cd-434">Erstellen Sie unter **Ressourcengruppe** eine neue Ressourcengruppe, unter der Sie das Profil platzieren möchten.</span><span class="sxs-lookup"><span data-stu-id="f16cd-434">In **Resource Group**, create a new resource group to place this profile under.</span></span>

    5. <span data-ttu-id="f16cd-435">Wählen Sie unter **Ressourcengruppenstandort** den Speicherort für die Ressourcengruppe aus.</span><span class="sxs-lookup"><span data-stu-id="f16cd-435">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="f16cd-436">Diese Einstellung bezieht sich auf den Speicherort der Ressourcengruppe und hat keine Auswirkungen auf das global bereitgestellte Traffic Manager-Profil.</span><span class="sxs-lookup"><span data-stu-id="f16cd-436">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile deployed globally.</span></span>

    6. <span data-ttu-id="f16cd-437">Klicken Sie auf **Erstellen**.</span><span class="sxs-lookup"><span data-stu-id="f16cd-437">Select **Create**.</span></span>

    7. <span data-ttu-id="f16cd-438">Wenn die globale Bereitstellung des Traffic Manager-Profils abgeschlossen ist, wird sie in der betreffenden Ressourcengruppe als eine der Ressourcen aufgelistet.</span><span class="sxs-lookup"><span data-stu-id="f16cd-438">When the global deployment of the Traffic Manager profile is complete, it's listed in the respective resource group as one of the resources.</span></span>

        ![Ressourcengruppe beim Erstellen im Traffic Manager-Profil](media/solution-deployment-guide-geo-distributed/image45.png)

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="f16cd-440">Hinzufügen von Traffic Manager-Endpunkten</span><span class="sxs-lookup"><span data-stu-id="f16cd-440">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="f16cd-441">Suchen Sie in der Suchleiste des Portals nach dem Namen für das **Traffic Manager-Profil**, das im vorherigen Abschnitt erstellt wurde, und wählen Sie das Traffic Manager-Profil in den angezeigten Ergebnissen aus.</span><span class="sxs-lookup"><span data-stu-id="f16cd-441">In the portal search bar, search for the **Traffic Manager profile** name created in the preceding section and select the traffic manager profile in the displayed results.</span></span>

2. <span data-ttu-id="f16cd-442">Wählen Sie im **Traffic Manager-Profil** im Abschnitt **Einstellungen** die Option **Endpunkte**.</span><span class="sxs-lookup"><span data-stu-id="f16cd-442">In **Traffic Manager profile**, in the **Settings** section, select **Endpoints**.</span></span>

3. <span data-ttu-id="f16cd-443">Wählen Sie **Hinzufügen**.</span><span class="sxs-lookup"><span data-stu-id="f16cd-443">Select **Add**.</span></span>

4. <span data-ttu-id="f16cd-444">Hinzufügen des Azure Stack Hub-Endpunkts.</span><span class="sxs-lookup"><span data-stu-id="f16cd-444">Adding the Azure Stack Hub Endpoint.</span></span>

5. <span data-ttu-id="f16cd-445">Wählen Sie unter **Typ** die Option **Externer Endpunkt**.</span><span class="sxs-lookup"><span data-stu-id="f16cd-445">For **Type**, select **External endpoint**.</span></span>

6. <span data-ttu-id="f16cd-446">Geben Sie unter **Name** einen Namen für diesen Endpunkt ein. Dies ist idealerweise der Name der Azure Stack Hub-Instanz.</span><span class="sxs-lookup"><span data-stu-id="f16cd-446">Provide a **Name** for this endpoint, ideally the name of the Azure Stack Hub.</span></span>

7. <span data-ttu-id="f16cd-447">Verwenden Sie für den vollqualifizierten Domänennamen (**FQDN**) die externe URL für die Azure Stack Hub-Web-App.</span><span class="sxs-lookup"><span data-stu-id="f16cd-447">For fully qualified domain name (**FQDN**), use the external URL for the Azure Stack Hub Web App.</span></span>

8. <span data-ttu-id="f16cd-448">Wählen Sie unter „Geografische Zuordnung“ die Region bzw. den Kontinent der Ressource aus.</span><span class="sxs-lookup"><span data-stu-id="f16cd-448">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="f16cd-449">Beispiel: **Europa**.</span><span class="sxs-lookup"><span data-stu-id="f16cd-449">For example, **Europe.**</span></span>

9. <span data-ttu-id="f16cd-450">Wählen Sie in der angezeigten Dropdownliste „Land/Region“ das Land aus, das für diesen Endpunkt gilt.</span><span class="sxs-lookup"><span data-stu-id="f16cd-450">Under the Country/Region drop-down that appears, select the country that applies to this endpoint.</span></span> <span data-ttu-id="f16cd-451">Beispiel: **Deutschland**.</span><span class="sxs-lookup"><span data-stu-id="f16cd-451">For example, **Germany**.</span></span>

10. <span data-ttu-id="f16cd-452">Lassen Sie **Als deaktiviert hinzufügen** deaktiviert.</span><span class="sxs-lookup"><span data-stu-id="f16cd-452">Keep **Add as disabled** unchecked.</span></span>

11. <span data-ttu-id="f16cd-453">Klicken Sie auf **OK**.</span><span class="sxs-lookup"><span data-stu-id="f16cd-453">Select **OK**.</span></span>

12. <span data-ttu-id="f16cd-454">Hinzufügen des Azure-Endpunkts:</span><span class="sxs-lookup"><span data-stu-id="f16cd-454">Adding the Azure Endpoint:</span></span>

    1. <span data-ttu-id="f16cd-455">Wählen Sie für **Typ** die Option **Azure-Endpunkt**.</span><span class="sxs-lookup"><span data-stu-id="f16cd-455">For **Type**, select **Azure endpoint**.</span></span>

    2. <span data-ttu-id="f16cd-456">Geben Sie einen **Namen** für den Endpunkt an.</span><span class="sxs-lookup"><span data-stu-id="f16cd-456">Provide a **Name** for the endpoint.</span></span>

    3. <span data-ttu-id="f16cd-457">Wählen Sie unter **Zielressourcentyp** die Option **App Service**.</span><span class="sxs-lookup"><span data-stu-id="f16cd-457">For **Target resource type**, select **App Service**.</span></span>

    4. <span data-ttu-id="f16cd-458">Wählen Sie unter **Zielressource** die Option **App Service auswählen**, um die Auflistung der Web-Apps desselben Abonnements anzuzeigen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-458">For **Target resource**, select **Choose an app service** to show the listing of the Web Apps under the same subscription.</span></span> <span data-ttu-id="f16cd-459">Wählen Sie unter **Ressource** die App Service-Instanz aus, die als erster Endpunkt verwendet werden soll.</span><span class="sxs-lookup"><span data-stu-id="f16cd-459">In **Resource**, pick the App service used as the first endpoint.</span></span>

13. <span data-ttu-id="f16cd-460">Wählen Sie unter „Geografische Zuordnung“ die Region bzw. den Kontinent der Ressource aus.</span><span class="sxs-lookup"><span data-stu-id="f16cd-460">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="f16cd-461">Beispiel: **Nordamerika/Zentralamerika/Karibik**.</span><span class="sxs-lookup"><span data-stu-id="f16cd-461">For example, **North America/Central America/Caribbean.**</span></span>

14. <span data-ttu-id="f16cd-462">Lassen Sie das Feld der Dropdownliste „Land/Region“ leer, um alle obigen regionalen Gruppierungen auszuwählen.</span><span class="sxs-lookup"><span data-stu-id="f16cd-462">Under the Country/Region drop-down that appears, leave this spot blank to select all of the above regional grouping.</span></span>

15. <span data-ttu-id="f16cd-463">Lassen Sie **Als deaktiviert hinzufügen** deaktiviert.</span><span class="sxs-lookup"><span data-stu-id="f16cd-463">Keep **Add as disabled** unchecked.</span></span>

16. <span data-ttu-id="f16cd-464">Klicken Sie auf **OK**.</span><span class="sxs-lookup"><span data-stu-id="f16cd-464">Select **OK**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="f16cd-465">Erstellen Sie mindestens einen Endpunkt mit dem geografischen Bereich „Alle (Welt)“, der als Standardendpunkt für die Ressource dient.</span><span class="sxs-lookup"><span data-stu-id="f16cd-465">Create at least one endpoint with a geographic scope of All (World) to serve as the default endpoint for the resource.</span></span>

17. <span data-ttu-id="f16cd-466">Wenn Sie das Hinzufügen beider Endpunkte abgeschlossen haben, werden diese unter **Traffic Manager-Profil** zusammen mit ihrem Überwachungsstatus als **Online** angezeigt.</span><span class="sxs-lookup"><span data-stu-id="f16cd-466">When the addition of both endpoints is complete, they're displayed in **Traffic Manager profile** along with their monitoring status as **Online**.</span></span>

    ![Endpunktstatus des Traffic Manager-Profils](media/solution-deployment-guide-geo-distributed/image46.png)

#### <a name="global-enterprise-relies-on-azure-geo-distribution-capabilities"></a><span data-ttu-id="f16cd-468">Globales Unternehmen nutzt Azure-Funktionen für die geografische Verteilung</span><span class="sxs-lookup"><span data-stu-id="f16cd-468">Global Enterprise relies on Azure geo-distribution capabilities</span></span>

<span data-ttu-id="f16cd-469">Die Weiterleitung des Datenverkehrs mit Azure Traffic Manager und geografieabhängigen Endpunkten ermöglicht es globalen Unternehmen, regionale Bestimmungen einzuhalten und für die Konformität und den Schutz der Daten zu sorgen, was für den Erfolg von lokalen und Remoteunternehmensstandorten von entscheidender Bedeutung ist.</span><span class="sxs-lookup"><span data-stu-id="f16cd-469">Directing data traffic via Azure Traffic Manager and geography-specific endpoints enables global enterprises to adhere to regional regulations and keep data compliant and secure, which is crucial to the success of local and remote business locations.</span></span>

## <a name="next-steps"></a><span data-ttu-id="f16cd-470">Nächste Schritte</span><span class="sxs-lookup"><span data-stu-id="f16cd-470">Next steps</span></span>

- <span data-ttu-id="f16cd-471">Weitere Informationen zu Azure-Cloudmustern finden Sie unter [Cloudentwurfsmuster](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="f16cd-471">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
