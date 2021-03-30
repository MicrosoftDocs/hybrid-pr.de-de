---
title: Bereitstellen einer App mit cloudübergreifender Skalierung in Azure und Azure Stack Hub
description: Es wird beschrieben, wie Sie eine App mit cloudübergreifender Skalierung in Azure und Azure Stack Hub bereitstellen.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: ed2ad5bed8f4bd80d4a40ab7600842d5544ff97d
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895413"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a><span data-ttu-id="52048-103">Bereitstellen einer App, die mithilfe von Azure und Azure Stack Hub cloudübergreifend skaliert wird</span><span class="sxs-lookup"><span data-stu-id="52048-103">Deploy an app that scales cross-cloud using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="52048-104">Es wird beschrieben, wie Sie eine cloudübergreifende Lösung erstellen, um einen manuell ausgelösten Prozess zum Umschalten von einer unter Azure Stack Hub gehosteten Web-App zu einer unter Azure gehosteten Web-App mit automatischer Skalierung per Traffic Manager bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="52048-104">Learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app with autoscaling via traffic manager.</span></span> <span data-ttu-id="52048-105">So wird sichergestellt, dass das Cloudhilfsprogramm auch bei hoher Last flexibel und skalierbar bleibt.</span><span class="sxs-lookup"><span data-stu-id="52048-105">This process ensures flexible and scalable cloud utility when under load.</span></span>

<span data-ttu-id="52048-106">Bei diesem Muster ist Ihr Mandant ggf. noch nicht bereit für die Ausführung Ihrer App in der öffentlichen Cloud.</span><span class="sxs-lookup"><span data-stu-id="52048-106">With this pattern, your tenant may not be ready to run your app in the public cloud.</span></span> <span data-ttu-id="52048-107">Es ist für das Unternehmen aber unter Umständen nicht wirtschaftlich, die für die lokale Umgebung erforderliche Kapazität beizubehalten, um für die App Auslastungsspitzen verarbeiten zu können.</span><span class="sxs-lookup"><span data-stu-id="52048-107">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="52048-108">Ihr Mandant kann die Elastizität der öffentlichen Cloud für die lokale Lösung nutzen.</span><span class="sxs-lookup"><span data-stu-id="52048-108">Your tenant can make use of the elasticity of the public cloud with their on-premises solution.</span></span>

<span data-ttu-id="52048-109">In dieser Lösung erstellen Sie eine Beispielumgebung, die Folgendes ermöglicht:</span><span class="sxs-lookup"><span data-stu-id="52048-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="52048-110">Erstellen Sie eine Web-App mit mehreren Knoten.</span><span class="sxs-lookup"><span data-stu-id="52048-110">Create a multi-node web app.</span></span>
> - <span data-ttu-id="52048-111">Konfigurieren und verwalten Sie den CD-Prozess (Continuous Deployment).</span><span class="sxs-lookup"><span data-stu-id="52048-111">Configure and manage the Continuous Deployment (CD) process.</span></span>
> - <span data-ttu-id="52048-112">Veröffentlichen Sie die Web-App in Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="52048-112">Publish the web app to Azure Stack Hub.</span></span>
> - <span data-ttu-id="52048-113">Erstellen Sie ein Release.</span><span class="sxs-lookup"><span data-stu-id="52048-113">Create a release.</span></span>
> - <span data-ttu-id="52048-114">Es wird beschrieben, wie Sie Ihre Bereitstellungen überwachen und nachverfolgen.</span><span class="sxs-lookup"><span data-stu-id="52048-114">Learn to monitor and track your deployments.</span></span>

> [!Tip]  
> <span data-ttu-id="52048-115">![Diagramm der Hybridsäulen](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="52048-115">![hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="52048-116">Microsoft Azure Stack Hub ist eine Erweiterung von Azure.</span><span class="sxs-lookup"><span data-stu-id="52048-116">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="52048-117">Mit Azure Stack Hub holen Sie sich die Agilität und Innovation von Cloud Computing in Ihre lokale Umgebung, indem Sie die einzige Hybrid Cloud aktivieren, mit der Sie Hybrid-Apps überall entwickeln und bereitstellen können.</span><span class="sxs-lookup"><span data-stu-id="52048-117">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="52048-118">Im Artikel [Überlegungen zum Entwurf von Hybrid-Apps](overview-app-design-considerations.md) werden die wichtigen Aspekte in Bezug auf die Softwarequalität (Platzierung, Skalierbarkeit, Verfügbarkeit, Resilienz, Verwaltbarkeit und Sicherheit) beschrieben, die für das Entwerfen, Bereitstellen und Betreiben von Hybrid-Apps erforderlich sind.</span><span class="sxs-lookup"><span data-stu-id="52048-118">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="52048-119">Die Überlegungen zum Entwurf dienen als Hilfe beim Optimieren des Designs von Hybrid-Apps, um für Produktionsumgebungen das Auftreten von Problemen zu minimieren.</span><span class="sxs-lookup"><span data-stu-id="52048-119">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="52048-120">Voraussetzungen</span><span class="sxs-lookup"><span data-stu-id="52048-120">Prerequisites</span></span>

- <span data-ttu-id="52048-121">Azure-Abonnement.</span><span class="sxs-lookup"><span data-stu-id="52048-121">Azure subscription.</span></span> <span data-ttu-id="52048-122">Erstellen Sie bei Bedarf ein [kostenloses Konto](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), bevor Sie beginnen.</span><span class="sxs-lookup"><span data-stu-id="52048-122">If needed, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before beginning.</span></span>
- <span data-ttu-id="52048-123">Ein integriertes Azure Stack Hub-System oder eine Bereitstellung des Azure Stack Development Kit (ASDK).</span><span class="sxs-lookup"><span data-stu-id="52048-123">An Azure Stack Hub integrated system or deployment of Azure Stack Development Kit (ASDK).</span></span>
  - <span data-ttu-id="52048-124">Eine Anleitung zur Installation von Azure Stack Hub finden Sie unter [Installieren des ASDK](/azure-stack/asdk/asdk-install).</span><span class="sxs-lookup"><span data-stu-id="52048-124">For instructions on installing Azure Stack Hub, see [Install the ASDK](/azure-stack/asdk/asdk-install).</span></span>
  - <span data-ttu-id="52048-125">Ein Skript zur Automatisierung der Vorgänge nach der Bereitstellung des ASDK finden Sie hier: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span><span class="sxs-lookup"><span data-stu-id="52048-125">For an ASDK post-deployment automation script, go to: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span></span>
  - <span data-ttu-id="52048-126">Es kann einige Stunden dauern, bis diese Installation abgeschlossen ist.</span><span class="sxs-lookup"><span data-stu-id="52048-126">This installation may require a few hours to complete.</span></span>
- <span data-ttu-id="52048-127">Stellen Sie PaaS-Dienste als [App Service](/azure-stack/operator/azure-stack-app-service-deploy) für Azure Stack Hub bereit.</span><span class="sxs-lookup"><span data-stu-id="52048-127">Deploy [App Service](/azure-stack/operator/azure-stack-app-service-deploy) PaaS services to Azure Stack Hub.</span></span>
- <span data-ttu-id="52048-128">[Erstellen Sie Pläne/Angebote](/azure-stack/operator/service-plan-offer-subscription-overview) in Ihrer Azure Stack Hub-Umgebung.</span><span class="sxs-lookup"><span data-stu-id="52048-128">[Create plans/offers](/azure-stack/operator/service-plan-offer-subscription-overview) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="52048-129">[Erstellen Sie ein Mandantenabonnement](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm) in Ihrer Azure Stack Hub-Umgebung.</span><span class="sxs-lookup"><span data-stu-id="52048-129">[Create tenant subscription](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="52048-130">Erstellen Sie eine Web-App in Ihrem Mandantenabonnement.</span><span class="sxs-lookup"><span data-stu-id="52048-130">Create a web app within the tenant subscription.</span></span> <span data-ttu-id="52048-131">Notieren Sie sich die URL der neuen Web-App zur späteren Verwendung.</span><span class="sxs-lookup"><span data-stu-id="52048-131">Make note of the new web app URL for later use.</span></span>
- <span data-ttu-id="52048-132">Stellen Sie einen virtuellen Azure Pipelines-Computer in Ihrem Mandantenabonnement bereit.</span><span class="sxs-lookup"><span data-stu-id="52048-132">Deploy Azure Pipelines virtual machine (VM) within the tenant subscription.</span></span>
- <span data-ttu-id="52048-133">Es wird ein virtueller Windows Server 2016-Computer mit .NET 3.5 benötigt.</span><span class="sxs-lookup"><span data-stu-id="52048-133">Windows Server 2016 VM with .NET 3.5 is required.</span></span> <span data-ttu-id="52048-134">Diese VM wird im Mandantenabonnement unter Azure Stack Hub als privater Build-Agent erstellt.</span><span class="sxs-lookup"><span data-stu-id="52048-134">This VM will be built in the tenant subscription on Azure Stack Hub as the private build agent.</span></span>
- <span data-ttu-id="52048-135">Das [Image „Windows Server 2016 mit SQL 2017-VM“](/azure-stack/operator/azure-stack-add-vm-image) ist auf dem Azure Stack Hub Marketplace verfügbar.</span><span class="sxs-lookup"><span data-stu-id="52048-135">[Windows Server 2016 with SQL 2017 VM image](/azure-stack/operator/azure-stack-add-vm-image) is available in the Azure Stack Hub Marketplace.</span></span> <span data-ttu-id="52048-136">Falls dieses Image nicht verfügbar sein sollte, können Sie einen Azure Stack Hub-Bediener bitten, es der Umgebung hinzuzufügen.</span><span class="sxs-lookup"><span data-stu-id="52048-136">If this image isn't available, work with an Azure Stack Hub Operator to ensure it's added to the environment.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="52048-137">Probleme und Überlegungen</span><span class="sxs-lookup"><span data-stu-id="52048-137">Issues and considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="52048-138">Skalierbarkeit</span><span class="sxs-lookup"><span data-stu-id="52048-138">Scalability</span></span>

<span data-ttu-id="52048-139">Die wichtigste Komponente der cloudübergreifenden Skalierung ist die Möglichkeit, sofortige Skalierung bei Bedarf zwischen der öffentlichen und lokalen Cloudinfrastruktur bereitzustellen und so einen konsistenten und zuverlässigen Dienst bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="52048-139">The key component of cross-cloud scaling is the ability to deliver immediate and on-demand scaling between public and on-premises cloud infrastructure, providing consistent and reliable service.</span></span>

### <a name="availability"></a><span data-ttu-id="52048-140">Verfügbarkeit</span><span class="sxs-lookup"><span data-stu-id="52048-140">Availability</span></span>

<span data-ttu-id="52048-141">Stellen Sie sicher, dass lokal bereitgestellte Apps in Bezug auf Hochverfügbarkeit konfiguriert sind, die auf der Konfiguration der lokalen Hardware und der Softwarebereitstellung basiert.</span><span class="sxs-lookup"><span data-stu-id="52048-141">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="52048-142">Verwaltbarkeit</span><span class="sxs-lookup"><span data-stu-id="52048-142">Manageability</span></span>

<span data-ttu-id="52048-143">Mit der cloudübergreifenden Lösung wird sichergestellt, dass zwischen Umgebungen eine nahtlose Verwaltung möglich ist und eine vertraute Benutzeroberfläche vorhanden ist.</span><span class="sxs-lookup"><span data-stu-id="52048-143">The cross-cloud solution ensures seamless management and familiar interface between environments.</span></span> <span data-ttu-id="52048-144">PowerShell wird für die plattformübergreifende Verwaltung empfohlen.</span><span class="sxs-lookup"><span data-stu-id="52048-144">PowerShell is recommended for cross-platform management.</span></span>

## <a name="cross-cloud-scaling"></a><span data-ttu-id="52048-145">Cloudübergreifende Skalierung</span><span class="sxs-lookup"><span data-stu-id="52048-145">Cross-cloud scaling</span></span>

### <a name="get-a-custom-domain-and-configure-dns"></a><span data-ttu-id="52048-146">Abrufen einer benutzerdefinierten Domäne und Konfigurieren des DNS</span><span class="sxs-lookup"><span data-stu-id="52048-146">Get a custom domain and configure DNS</span></span>

<span data-ttu-id="52048-147">Aktualisieren Sie die DNS-Zonendatei für die Domäne.</span><span class="sxs-lookup"><span data-stu-id="52048-147">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="52048-148">Azure AD überprüft die Eigentümerschaft für den Namen der benutzerdefinierten Domäne.</span><span class="sxs-lookup"><span data-stu-id="52048-148">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="52048-149">Verwenden Sie [Azure DNS](/azure/dns/dns-getstarted-portal) für Azure- oder Microsoft 365-Einträge bzw. externe DNS-Einträge in Azure, oder fügen Sie den DNS-Eintrag bei einer [anderen DNS-Registrierungsstelle](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider) hinzu.</span><span class="sxs-lookup"><span data-stu-id="52048-149">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

1. <span data-ttu-id="52048-150">Registrieren Sie eine benutzerdefinierte Domäne bei einer öffentlichen Registrierungsstelle.</span><span class="sxs-lookup"><span data-stu-id="52048-150">Register a custom domain with a public registrar.</span></span>
2. <span data-ttu-id="52048-151">Melden Sie sich an der Domänennamen-Registrierungsstelle für die Domäne an.</span><span class="sxs-lookup"><span data-stu-id="52048-151">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="52048-152">Unter Umständen ist ein genehmigter Administrator erforderlich, um die DNS-Updates durchzuführen.</span><span class="sxs-lookup"><span data-stu-id="52048-152">An approved admin may be required to make DNS updates.</span></span>
3. <span data-ttu-id="52048-153">Aktualisieren Sie die DNS-Zonendatei für die Domäne, indem Sie den DNS-Eintrag hinzufügen, der von Azure AD bereitgestellt wurde.</span><span class="sxs-lookup"><span data-stu-id="52048-153">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="52048-154">(Der DNS-Eintrag hat keinerlei Auswirkung auf das E-Mail-Routing oder das Webhosting-Verhalten.)</span><span class="sxs-lookup"><span data-stu-id="52048-154">(The DNS entry won't affect email routing or web hosting behaviors.)</span></span>

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a><span data-ttu-id="52048-155">Erstellen einer Standard-Web-App mit mehreren Knoten in Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="52048-155">Create a default multi-node web app in Azure Stack Hub</span></span>

<span data-ttu-id="52048-156">Richten Sie Hybrid-CI/CD (Continuous Integration/Continuous Deployment) ein, um Web-Apps unter Azure und Azure Stack Hub bereitzustellen und Änderungen automatisch per Pushvorgang an beide Clouds zu übertragen.</span><span class="sxs-lookup"><span data-stu-id="52048-156">Set up hybrid continuous integration and continuous deployment (CI/CD) to deploy web apps to Azure and Azure Stack Hub and to autopush changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="52048-157">Azure Stack Hub mit den passenden syndizierten Images für die Ausführung (Windows Server und SQL) und eine App Service-Bereitstellung sind erforderlich.</span><span class="sxs-lookup"><span data-stu-id="52048-157">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="52048-158">Weitere Informationen finden Sie in der App Service-Dokumentation unter [Voraussetzungen für das Bereitstellen von App Service unter Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started).</span><span class="sxs-lookup"><span data-stu-id="52048-158">For more information, review the App Service documentation [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started).</span></span>

### <a name="add-code-to-azure-repos"></a><span data-ttu-id="52048-159">Hinzufügen von Code zu Azure Repos</span><span class="sxs-lookup"><span data-stu-id="52048-159">Add Code to Azure Repos</span></span>

<span data-ttu-id="52048-160">Azure Repos</span><span class="sxs-lookup"><span data-stu-id="52048-160">Azure Repos</span></span>

1. <span data-ttu-id="52048-161">Melden Sie sich bei Azure Repos mit einem Konto an, das über Berechtigungen zum Erstellen von Projekten in Azure Repos verfügt.</span><span class="sxs-lookup"><span data-stu-id="52048-161">Sign in to Azure Repos with an account that has project creation rights on Azure Repos.</span></span>

    <span data-ttu-id="52048-162">Der hybride CI/CD-Ansatz kann sowohl für App-Code als auch für Infrastrukturcode verwendet werden.</span><span class="sxs-lookup"><span data-stu-id="52048-162">Hybrid CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="52048-163">Verwenden Sie [Azure Resource Manager-Vorlagen](https://azure.microsoft.com/resources/templates/) für die Entwicklung von privaten und gehosteten Clouds.</span><span class="sxs-lookup"><span data-stu-id="52048-163">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Herstellen einer Verbindung mit einem Projekt in Azure Repos](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. <span data-ttu-id="52048-165">**Klonen Sie das Repository**, indem Sie die Standard-Web-App erstellen und öffnen.</span><span class="sxs-lookup"><span data-stu-id="52048-165">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Klonen des Repos in der Azure-Web-App](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="52048-167">Erstellen einer eigenständigen Web-App-Bereitstellung für App Services in beiden Clouds</span><span class="sxs-lookup"><span data-stu-id="52048-167">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="52048-168">Bearbeiten Sie die Datei **WebApplication.csproj**.</span><span class="sxs-lookup"><span data-stu-id="52048-168">Edit the **WebApplication.csproj** file.</span></span> <span data-ttu-id="52048-169">Wählen Sie `Runtimeidentifier` aus, und fügen Sie `win10-x64` hinzu.</span><span class="sxs-lookup"><span data-stu-id="52048-169">Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="52048-170">(Weitere Informationen finden Sie in der Dokumentation zur [eigenständigen Bereitstellung](/dotnet/core/deploying/deploy-with-vs#simpleSelf).)</span><span class="sxs-lookup"><span data-stu-id="52048-170">(See [Self-contained deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Bearbeiten Projektdatei einer Web-App](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. <span data-ttu-id="52048-172">Checken Sie den Code in Azure Repos ein, indem Sie Team Explorer verwenden.</span><span class="sxs-lookup"><span data-stu-id="52048-172">Check in the code to Azure Repos using Team Explorer.</span></span>

3. <span data-ttu-id="52048-173">Vergewissern Sie sich, dass der App-Code in Azure Repos eingecheckt wurde.</span><span class="sxs-lookup"><span data-stu-id="52048-173">Confirm that the app code has been checked into Azure Repos.</span></span>

## <a name="create-the-build-definition"></a><span data-ttu-id="52048-174">Erstellen der Builddefinition</span><span class="sxs-lookup"><span data-stu-id="52048-174">Create the build definition</span></span>

1. <span data-ttu-id="52048-175">Melden Sie sich bei Azure Pipelines an, um sich zu vergewissern, dass Builddefinitionen erstellt werden können.</span><span class="sxs-lookup"><span data-stu-id="52048-175">Sign in to Azure Pipelines to confirm the ability to create build definitions.</span></span>

2. <span data-ttu-id="52048-176">Fügen Sie den Code **-r win10-x64** hinzu.</span><span class="sxs-lookup"><span data-stu-id="52048-176">Add **-r win10-x64** code.</span></span> <span data-ttu-id="52048-177">Diese Hinzufügung ist erforderlich, um eine eigenständige Bereitstellung mit .NET Core auszulösen.</span><span class="sxs-lookup"><span data-stu-id="52048-177">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Hinzufügen von Code zur Web-App](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. <span data-ttu-id="52048-179">Führen Sie den Buildvorgang aus.</span><span class="sxs-lookup"><span data-stu-id="52048-179">Run the build.</span></span> <span data-ttu-id="52048-180">Der Buildvorgang für die [eigenständige Bereitstellung](/dotnet/core/deploying/deploy-with-vs#simpleSelf) veröffentlicht Artefakte, die in Azure und Azure Stack Hub ausgeführt werden.</span><span class="sxs-lookup"><span data-stu-id="52048-180">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that run on Azure and Azure Stack Hub.</span></span>

## <a name="use-an-azure-hosted-agent"></a><span data-ttu-id="52048-181">Verwenden eines gehosteten Azure-Agents</span><span class="sxs-lookup"><span data-stu-id="52048-181">Use an Azure hosted agent</span></span>

<span data-ttu-id="52048-182">Mit einem gehosteten Build-Agent lassen sich in Azure Pipelines komfortabel Web-Apps erstellen und bereitstellen.</span><span class="sxs-lookup"><span data-stu-id="52048-182">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="52048-183">Wartungsarbeiten und Upgrades werden automatisch von Microsoft Azure durchgeführt, was einen kontinuierlichen und ununterbrochenen Entwicklungszyklus ermöglicht.</span><span class="sxs-lookup"><span data-stu-id="52048-183">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="52048-184">Verwalten und Konfigurieren des CD-Prozesses</span><span class="sxs-lookup"><span data-stu-id="52048-184">Manage and configure the CD process</span></span>

<span data-ttu-id="52048-185">Azure Pipelines und Azure DevOps Services bieten eine äußerst flexibel konfigurier- und verwaltbare Pipeline für Releases in mehreren Umgebungen (z. B. in Entwicklungs-, Staging-, Qualitätssicherungs- und Produktionsumgebungen) – einschließlich erforderlicher Genehmigungen in bestimmten Phasen.</span><span class="sxs-lookup"><span data-stu-id="52048-185">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="52048-186">Erstellen einer Releasedefinition</span><span class="sxs-lookup"><span data-stu-id="52048-186">Create release definition</span></span>

1. <span data-ttu-id="52048-187">Klicken Sie in Azure DevOps Services im Abschnitt **Build und Release** auf der Registerkarte **Releases** auf die Schaltfläche mit dem **Pluszeichen**, um ein neues Release hinzuzufügen.</span><span class="sxs-lookup"><span data-stu-id="52048-187">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![Erstellen einer Releasedefinition](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. <span data-ttu-id="52048-189">Wenden Sie die Vorlage „Azure App Service-Bereitstellung“ an.</span><span class="sxs-lookup"><span data-stu-id="52048-189">Apply the Azure App Service Deployment template.</span></span>

   ![Anwenden der Bereitstellungsvorlage für Azure App Service](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. <span data-ttu-id="52048-191">Fügen Sie unter **Artefakt hinzufügen** das Artefakt für die Azure Cloud-Build-App hinzu.</span><span class="sxs-lookup"><span data-stu-id="52048-191">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Hinzufügen des Artefakts zum Azure Cloud-Build](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. <span data-ttu-id="52048-193">Klicken Sie auf der Registerkarte „Pipeline“ auf den Link **Phase, Aufgabe** der Umgebung, und legen Sie die Azure Cloud-Umgebungswerte fest.</span><span class="sxs-lookup"><span data-stu-id="52048-193">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Festlegen der Werte für die Azure Cloud-Umgebung](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. <span data-ttu-id="52048-195">Legen Sie den **Umgebungsnamen** fest, und wählen Sie das **Azure-Abonnement** für den Azure Cloud-Endpunkt aus.</span><span class="sxs-lookup"><span data-stu-id="52048-195">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Auswählen des Azure-Abonnements für den Azure Cloud-Endpunkt](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. <span data-ttu-id="52048-197">Legen Sie unter **App Service-Name** den erforderlichen Namen des Azure App Service fest.</span><span class="sxs-lookup"><span data-stu-id="52048-197">Under **App service name**, set the required Azure app service name.</span></span>

      ![Festlegen des Azure App Service-Namens](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. <span data-ttu-id="52048-199">Geben Sie unter **Agent-Warteschlange** für die gehostete Azure Cloud-Umgebung die Zeichenfolge „Hosted VS2017“ ein.</span><span class="sxs-lookup"><span data-stu-id="52048-199">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Festlegen der Agent-Warteschlange für die gehostete Azure Cloud-Umgebung](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. <span data-ttu-id="52048-201">Wählen Sie im Menü „Deploy Azure App Service“ (Azure App Service bereitstellen) **das gültige Paket oder den gültigen Ordner** für die Umgebung aus.</span><span class="sxs-lookup"><span data-stu-id="52048-201">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="52048-202">Klicken Sie für den **Ordnerspeicherort** auf **OK**.</span><span class="sxs-lookup"><span data-stu-id="52048-202">Select **OK** to **folder location**.</span></span>
  
      ![Auswählen des Pakets oder Ordners für die Azure App Service-Umgebung](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![Dialogfeld 1: Ordnerauswahl](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. <span data-ttu-id="52048-205">Speichern Sie alle Änderungen, und kehren Sie zur **Releasepipeline** zurück.</span><span class="sxs-lookup"><span data-stu-id="52048-205">Save all changes and go back to **release pipeline**.</span></span>

    ![Speichern der Änderungen in der Releasepipeline](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. <span data-ttu-id="52048-207">Fügen Sie ein neues Artefakt hinzu, und wählen Sie dabei den Build für die Azure Stack Hub-App aus.</span><span class="sxs-lookup"><span data-stu-id="52048-207">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Hinzufügen eines neuen Artefakts für die Azure Stack Hub-App](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. <span data-ttu-id="52048-209">Fügen Sie mindestens eine Umgebung hinzu, indem Sie die Azure App Service-Bereitstellung anwenden.</span><span class="sxs-lookup"><span data-stu-id="52048-209">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Hinzufügen einer Umgebung zur Azure App Service-Bereitstellung](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. <span data-ttu-id="52048-211">Nennen Sie die neue Umgebung „Azure Stack“.</span><span class="sxs-lookup"><span data-stu-id="52048-211">Name the new environment "Azure Stack".</span></span>

    ![Benennen der Umgebung in der Azure App Service-Bereitstellung](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. <span data-ttu-id="52048-213">Suchen Sie auf der Registerkarte **Aufgabe** nach der Umgebung „Azure Stack“.</span><span class="sxs-lookup"><span data-stu-id="52048-213">Find the Azure Stack environment under **Task** tab.</span></span>

    ![Azure Stack-Umgebung](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. <span data-ttu-id="52048-215">Wählen Sie das Abonnement für den Azure Stack-Endpunkt aus.</span><span class="sxs-lookup"><span data-stu-id="52048-215">Select the subscription for the Azure Stack endpoint.</span></span>

    ![Auswählen des Abonnements für den Azure Stack-Endpunkt](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. <span data-ttu-id="52048-217">Legen Sie den Namen der Azure Stack-Web-App als App Service-Name fest.</span><span class="sxs-lookup"><span data-stu-id="52048-217">Set the Azure Stack web app name as the App service name.</span></span>
    <span data-ttu-id="52048-218">![Festlegen des Namens der Azure Stack-Web-App](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span><span class="sxs-lookup"><span data-stu-id="52048-218">![Set Azure Stack web app name](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span></span>

16. <span data-ttu-id="52048-219">Wählen Sie den Azure Stack-Agent aus.</span><span class="sxs-lookup"><span data-stu-id="52048-219">Select the Azure Stack agent.</span></span>

    ![Auswählen des Azure Stack-Agents](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. <span data-ttu-id="52048-221">Wählen Sie im Abschnitt „Deploy Azure App Service“ (Azure App Service bereitstellen) **das gültige Paket oder den gültigen Ordner** für die Umgebung aus.</span><span class="sxs-lookup"><span data-stu-id="52048-221">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="52048-222">Klicken Sie für den Ordnerspeicherort auf **OK**.</span><span class="sxs-lookup"><span data-stu-id="52048-222">Select **OK** to folder location.</span></span>

    ![Wählen Sie den Ordner für die Azure App Service-Bereitstellung aus.](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![Dialogfeld 2: Ordnerauswahl](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. <span data-ttu-id="52048-225">Fügen Sie auf der Registerkarte „Variable“ eine Variable mit dem Namen `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` hinzu, und legen Sie ihren Wert auf **true** und den Bereich auf „Azure Stack“ fest.</span><span class="sxs-lookup"><span data-stu-id="52048-225">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack.</span></span>

    ![Hinzufügen einer Variablen zur Azure App-Bereitstellung](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. <span data-ttu-id="52048-227">Klicken Sie bei beiden Artefakten auf das Symbol **Continuous Deployment-Trigger**, und aktivieren Sie den Trigger für **Continuous Deployment**.</span><span class="sxs-lookup"><span data-stu-id="52048-227">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![Festlegen des Continuous Deployment-Triggers](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. <span data-ttu-id="52048-229">Klicken Sie für die Azure Stack-Umgebung auf das Symbol **Bedingungen vor der Bereitstellung**, und legen Sie den Trigger auf **Nach einem Release** fest.</span><span class="sxs-lookup"><span data-stu-id="52048-229">Select the **Pre-deployment** conditions icon in the Azure Stack environment and set the trigger to **After release.**</span></span>

    ![Festlegen der Bedingungen vor der Bereitstellung](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. <span data-ttu-id="52048-231">Speichern Sie alle Änderungen.</span><span class="sxs-lookup"><span data-stu-id="52048-231">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="52048-232">Bei der vorlagenbasierten Erstellung einer Releasedefinition wurden einige Einstellungen für die Aufgaben unter Umständen automatisch als [Umgebungsvariablen](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) definiert.</span><span class="sxs-lookup"><span data-stu-id="52048-232">Some settings for the tasks may have been automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="52048-233">Diese Einstellungen können in den Aufgabeneinstellungen nicht geändert werden. Zum Bearbeiten dieser Einstellungen müssen Sie das übergeordnete Umgebungselement auswählen.</span><span class="sxs-lookup"><span data-stu-id="52048-233">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a><span data-ttu-id="52048-234">Veröffentlichen in Azure Stack Hub mit Visual Studio</span><span class="sxs-lookup"><span data-stu-id="52048-234">Publish to Azure Stack Hub via Visual Studio</span></span>

<span data-ttu-id="52048-235">Durch die Erstellung von Endpunkten kann ein Azure DevOps Services-Build Azure Service-Apps in Azure Stack Hub bereitstellen.</span><span class="sxs-lookup"><span data-stu-id="52048-235">By creating endpoints, an Azure DevOps Services build can deploy Azure Service apps to Azure Stack Hub.</span></span> <span data-ttu-id="52048-236">Azure Pipelines stellt eine Verbindung mit dem Build-Agent her, der wiederum eine Verbindung mit Azure Stack Hub herstellt.</span><span class="sxs-lookup"><span data-stu-id="52048-236">Azure Pipelines connects to the build agent, which connects to Azure Stack Hub.</span></span>

1. <span data-ttu-id="52048-237">Melden Sie sich bei Azure DevOps Services an, und navigieren Sie zur Seite mit den App-Einstellungen.</span><span class="sxs-lookup"><span data-stu-id="52048-237">Sign in to Azure DevOps Services and go to the app settings page.</span></span>

2. <span data-ttu-id="52048-238">Wählen Sie unter **Einstellungen** die Option **Sicherheit**.</span><span class="sxs-lookup"><span data-stu-id="52048-238">On **Settings**, select **Security**.</span></span>

3. <span data-ttu-id="52048-239">Klicken Sie unter **VSTS-Gruppen** auf **Endpunktersteller**.</span><span class="sxs-lookup"><span data-stu-id="52048-239">In **VSTS Groups**, select **Endpoint Creators**.</span></span>

4. <span data-ttu-id="52048-240">Klicken Sie auf der Registerkarte **Mitglieder** auf **Hinzufügen**.</span><span class="sxs-lookup"><span data-stu-id="52048-240">On the **Members** tab, select **Add**.</span></span>

5. <span data-ttu-id="52048-241">Geben Sie unter **Benutzer und Gruppen hinzufügen** einen Benutzernamen ein, und wählen Sie den Benutzer aus der Liste der Benutzer aus.</span><span class="sxs-lookup"><span data-stu-id="52048-241">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

6. <span data-ttu-id="52048-242">Klicken Sie auf **Save changes** (Änderungen speichern).</span><span class="sxs-lookup"><span data-stu-id="52048-242">Select **Save changes**.</span></span>

7. <span data-ttu-id="52048-243">Wählen Sie in der Liste **VSTS-Gruppen** die Option **Endpunktadministratoren** aus.</span><span class="sxs-lookup"><span data-stu-id="52048-243">In the **VSTS Groups** list, select **Endpoint Administrators**.</span></span>

8. <span data-ttu-id="52048-244">Klicken Sie auf der Registerkarte **Mitglieder** auf **Hinzufügen**.</span><span class="sxs-lookup"><span data-stu-id="52048-244">On the **Members** tab, select **Add**.</span></span>

9. <span data-ttu-id="52048-245">Geben Sie unter **Benutzer und Gruppen hinzufügen** einen Benutzernamen ein, und wählen Sie den Benutzer aus der Liste der Benutzer aus.</span><span class="sxs-lookup"><span data-stu-id="52048-245">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

10. <span data-ttu-id="52048-246">Klicken Sie auf **Save changes** (Änderungen speichern).</span><span class="sxs-lookup"><span data-stu-id="52048-246">Select **Save changes**.</span></span>

<span data-ttu-id="52048-247">Die Endpunktinformationen sind vorhanden, und die Verbindung zwischen Azure Pipelines und Azure Stack Hub kann nun verwendet werden.</span><span class="sxs-lookup"><span data-stu-id="52048-247">Now that the endpoint information exists, the Azure Pipelines to Azure Stack Hub connection is ready to use.</span></span> <span data-ttu-id="52048-248">Der Build-Agent in Azure Stack Hub erhält Anweisungen von Azure Pipelines. Daraufhin übermittelt der Agent Endpunktinformationen für die Kommunikation mit Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="52048-248">The build agent in Azure Stack Hub gets instructions from Azure Pipelines and then the agent conveys endpoint information for communication with Azure Stack Hub.</span></span>

## <a name="develop-the-app-build"></a><span data-ttu-id="52048-249">Entwickeln des App-Builds</span><span class="sxs-lookup"><span data-stu-id="52048-249">Develop the app build</span></span>

> [!Note]  
> <span data-ttu-id="52048-250">Azure Stack Hub mit den passenden syndizierten Images für die Ausführung (Windows Server und SQL) und eine App Service-Bereitstellung sind erforderlich.</span><span class="sxs-lookup"><span data-stu-id="52048-250">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="52048-251">Weitere Informationen finden Sie unter [Voraussetzungen für das Bereitstellen von App Service in Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started).</span><span class="sxs-lookup"><span data-stu-id="52048-251">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started).</span></span>

<span data-ttu-id="52048-252">Verwenden Sie [Azure Resource Manager-Vorlagen](https://azure.microsoft.com/resources/templates/), z. B. Web-App-Code aus Azure Repos, für die Bereitstellung in beiden Clouds.</span><span class="sxs-lookup"><span data-stu-id="52048-252">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) like web app code from Azure Repos to deploy to both clouds.</span></span>

### <a name="add-code-to-an-azure-repos-project"></a><span data-ttu-id="52048-253">Hinzufügen von Code zu einem Azure Repos-Projekt</span><span class="sxs-lookup"><span data-stu-id="52048-253">Add code to an Azure Repos project</span></span>

1. <span data-ttu-id="52048-254">Melden Sie sich bei Azure Repos mit einem Konto an, das über Berechtigungen zum Erstellen von Projekten in Azure Stack Hub verfügt.</span><span class="sxs-lookup"><span data-stu-id="52048-254">Sign in to Azure Repos with an account that has project creation rights on Azure Stack Hub.</span></span>

2. <span data-ttu-id="52048-255">**Klonen Sie das Repository**, indem Sie die Standard-Web-App erstellen und öffnen.</span><span class="sxs-lookup"><span data-stu-id="52048-255">**Clone the repository** by creating and opening the default web app.</span></span>

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="52048-256">Erstellen einer eigenständigen Web-App-Bereitstellung für App Services in beiden Clouds</span><span class="sxs-lookup"><span data-stu-id="52048-256">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="52048-257">Bearbeiten Sie die Datei **WebApplication.csproj**: Wählen Sie `Runtimeidentifier` aus, und fügen Sie dann `win10-x64` hinzu.</span><span class="sxs-lookup"><span data-stu-id="52048-257">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and then add `win10-x64`.</span></span> <span data-ttu-id="52048-258">Weitere Informationen finden Sie in der Dokumentation zur [eigenständigen Bereitstellung](/dotnet/core/deploying/deploy-with-vs#simpleSelf).</span><span class="sxs-lookup"><span data-stu-id="52048-258">For more information, see [Self-contained deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.</span></span>

2. <span data-ttu-id="52048-259">Checken Sie den Code über den Team Explorer in Azure Repos ein.</span><span class="sxs-lookup"><span data-stu-id="52048-259">Use Team Explorer to check the code into Azure Repos.</span></span>

3. <span data-ttu-id="52048-260">Vergewissern Sie sich, dass der App-Code in Azure Repos eingecheckt wurde.</span><span class="sxs-lookup"><span data-stu-id="52048-260">Confirm that the app code was checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="52048-261">Erstellen der Builddefinition</span><span class="sxs-lookup"><span data-stu-id="52048-261">Create the build definition</span></span>

1. <span data-ttu-id="52048-262">Melden Sie sich bei Azure Pipelines mit einem Konto an, mit dem eine Builddefinition erstellt werden kann.</span><span class="sxs-lookup"><span data-stu-id="52048-262">Sign in to Azure Pipelines with an account that can create a build definition.</span></span>

2. <span data-ttu-id="52048-263">Navigieren Sie zur Seite **Build Web Application** (Webanwendung erstellen) für das Projekt.</span><span class="sxs-lookup"><span data-stu-id="52048-263">Go to the **Build Web Application** page for the project.</span></span>

3. <span data-ttu-id="52048-264">Fügen Sie unter **Argumente** den Code **-r win10-x64** hinzu.</span><span class="sxs-lookup"><span data-stu-id="52048-264">In **Arguments**, add **-r win10-x64** code.</span></span> <span data-ttu-id="52048-265">Diese Hinzufügung ist erforderlich, um eine eigenständige Bereitstellung mit .NET Core auszulösen.</span><span class="sxs-lookup"><span data-stu-id="52048-265">This addition is required to trigger a self-contained deployment with .NET Core.</span></span>

4. <span data-ttu-id="52048-266">Führen Sie den Buildvorgang aus.</span><span class="sxs-lookup"><span data-stu-id="52048-266">Run the build.</span></span> <span data-ttu-id="52048-267">Der Buildvorgang für die [eigenständige Bereitstellung](/dotnet/core/deploying/deploy-with-vs#simpleSelf) veröffentlicht Artefakte, die in Azure und Azure Stack Hub ausgeführt werden können.</span><span class="sxs-lookup"><span data-stu-id="52048-267">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="use-an-azure-hosted-build-agent"></a><span data-ttu-id="52048-268">Verwenden eines in Azure gehosteten Build-Agents</span><span class="sxs-lookup"><span data-stu-id="52048-268">Use an Azure hosted build agent</span></span>

<span data-ttu-id="52048-269">Mit einem gehosteten Build-Agent lassen sich in Azure Pipelines komfortabel Web-Apps erstellen und bereitstellen.</span><span class="sxs-lookup"><span data-stu-id="52048-269">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="52048-270">Wartungsarbeiten und Upgrades werden automatisch von Microsoft Azure durchgeführt, was einen kontinuierlichen und ununterbrochenen Entwicklungszyklus ermöglicht.</span><span class="sxs-lookup"><span data-stu-id="52048-270">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="configure-the-continuous-deployment-cd-process"></a><span data-ttu-id="52048-271">Konfigurieren des CD-Prozesses (Continuous Deployment)</span><span class="sxs-lookup"><span data-stu-id="52048-271">Configure the continuous deployment (CD) process</span></span>

<span data-ttu-id="52048-272">Azure Pipelines und Azure DevOps Services verfügen über eine äußerst flexibel konfigurier- und verwaltbare Pipeline für Releases in mehreren Umgebungen (z. B. in Entwicklungs-, Staging-, Qualitätssicherungs- und Produktionsumgebungen).</span><span class="sxs-lookup"><span data-stu-id="52048-272">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, quality assurance (QA), and production.</span></span> <span data-ttu-id="52048-273">Dabei sind ggf. Genehmigungen in bestimmten Phasen des App-Lebenszyklus erforderlich.</span><span class="sxs-lookup"><span data-stu-id="52048-273">This process can include requiring approvals at specific stages of the app life cycle.</span></span>

#### <a name="create-release-definition"></a><span data-ttu-id="52048-274">Erstellen einer Releasedefinition</span><span class="sxs-lookup"><span data-stu-id="52048-274">Create release definition</span></span>

<span data-ttu-id="52048-275">Die Erstellung einer Releasedefinition ist der letzte Schritt im App-Buildprozess.</span><span class="sxs-lookup"><span data-stu-id="52048-275">Creating a release definition is the final step in the app build process.</span></span> <span data-ttu-id="52048-276">Diese Releasedefinition wird zum Erstellen eines Release und zum Bereitstellen eines Builds verwendet.</span><span class="sxs-lookup"><span data-stu-id="52048-276">This release definition is used to create a release and deploy a build.</span></span>

1. <span data-ttu-id="52048-277">Melden Sie sich bei Azure Pipelines an, und navigieren Sie für das Projekt zu **Build und Release**.</span><span class="sxs-lookup"><span data-stu-id="52048-277">Sign in to Azure Pipelines and go to **Build and Release** for the project.</span></span>

2. <span data-ttu-id="52048-278">Klicken Sie auf der Registerkarte **Releases** auf **[ + ]** , und wählen Sie dann **Releasedefinition erstellen**.</span><span class="sxs-lookup"><span data-stu-id="52048-278">On the **Releases** tab, select **[ + ]** and then pick **Create release definition**.</span></span>

3. <span data-ttu-id="52048-279">Klicken Sie unter **Vorlage auswählen** auf **Azure App Service-Bereitstellung** und dann auf **Anwenden**.</span><span class="sxs-lookup"><span data-stu-id="52048-279">On **Select a Template**, choose **Azure App Service Deployment**, and then select **Apply**.</span></span>

4. <span data-ttu-id="52048-280">Wählen Sie unter **Artefakt hinzufügen** und **Quelle (Builddefinition)** die Azure Cloud-Build-App aus.</span><span class="sxs-lookup"><span data-stu-id="52048-280">On **Add artifact**, from the **Source (Build definition)**, select the Azure Cloud build app.</span></span>

5. <span data-ttu-id="52048-281">Klicken Sie auf der Registerkarte **Pipeline** auf den Link **1 Phase**, **1 Task** (1 Phase, 1 Aufgabe), um **Umgebungsaufgaben anzuzeigen**.</span><span class="sxs-lookup"><span data-stu-id="52048-281">On the **Pipeline** tab, select the **1 Phase**, **1 Task** link to **View environment tasks**.</span></span>

6. <span data-ttu-id="52048-282">Geben Sie auf der Registerkarte **Aufgaben** unter **Umgebungsname** den Namen „Azure“ ein, und wählen Sie in der Liste **Azure-Abonnement** den Eintrag „AzureCloud Traders-Web EP“ aus.</span><span class="sxs-lookup"><span data-stu-id="52048-282">On the **Tasks** tab, enter Azure as the **Environment name** and select the AzureCloud Traders-Web EP from the **Azure subscription** list.</span></span>

7. <span data-ttu-id="52048-283">Geben Sie den **Azure App Service-Namen** ein. Im nächsten Screenshot ist dies `northwindtraders`.</span><span class="sxs-lookup"><span data-stu-id="52048-283">Enter the **Azure app service name**, which is `northwindtraders` in the next screen capture.</span></span>

8. <span data-ttu-id="52048-284">Wählen Sie unter „Agent-Phase“ in der Liste **Agent-Warteschlange** den Eintrag **Hosted VS2017** aus.</span><span class="sxs-lookup"><span data-stu-id="52048-284">For the Agent phase, select **Hosted VS2017** from the **Agent queue** list.</span></span>

9. <span data-ttu-id="52048-285">Wählen Sie im Menü **Deploy Azure App Service** (Azure App Service bereitstellen) unter **Paket oder Ordner** den gültigen Eintrag für die Umgebung aus.</span><span class="sxs-lookup"><span data-stu-id="52048-285">In **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span>

10. <span data-ttu-id="52048-286">Klicken Sie unter **Datei oder Ordner auswählen** auf **OK**, um den **Speicherort** anzugeben.</span><span class="sxs-lookup"><span data-stu-id="52048-286">In **Select File or Folder**, select **OK** to **Location**.</span></span>

11. <span data-ttu-id="52048-287">Speichern Sie alle Änderungen, und kehren Sie zur **Pipeline** zurück.</span><span class="sxs-lookup"><span data-stu-id="52048-287">Save all changes and go back to **Pipeline**.</span></span>

12. <span data-ttu-id="52048-288">Klicken Sie auf der Registerkarte **Pipeline** auf **Artefakt hinzufügen**, und wählen Sie in der Liste **Quelle (Builddefinition)** den Eintrag **NorthwindCloud Traders-Vessel** aus.</span><span class="sxs-lookup"><span data-stu-id="52048-288">On the **Pipeline** tab, select **Add artifact**, and choose the **NorthwindCloud Traders-Vessel** from the **Source (Build Definition)** list.</span></span>

13. <span data-ttu-id="52048-289">Fügen Sie unter **Vorlage auswählen** eine andere Umgebung aus.</span><span class="sxs-lookup"><span data-stu-id="52048-289">On **Select a Template**, add another environment.</span></span> <span data-ttu-id="52048-290">Wählen Sie **Azure App Service-Bereitstellung** und dann **Anwenden** aus.</span><span class="sxs-lookup"><span data-stu-id="52048-290">Pick **Azure App Service Deployment** and then select **Apply**.</span></span>

14. <span data-ttu-id="52048-291">Geben Sie `Azure Stack Hub` als **Umgebungsnamen** ein.</span><span class="sxs-lookup"><span data-stu-id="52048-291">Enter `Azure Stack Hub` as the **Environment name**.</span></span>

15. <span data-ttu-id="52048-292">Suchen Sie auf der Registerkarte **Aufgaben** nach „Azure Stack Hub“.</span><span class="sxs-lookup"><span data-stu-id="52048-292">On the **Tasks** tab, find and select Azure Stack Hub.</span></span>

16. <span data-ttu-id="52048-293">Wählen Sie in der Liste **Azure-Abonnement** als Azure Stack Hub-Endpunkt **AzureStack Traders-Vessel EP** aus.</span><span class="sxs-lookup"><span data-stu-id="52048-293">From the **Azure subscription** list, select **AzureStack Traders-Vessel EP** for the Azure Stack Hub endpoint.</span></span>

17. <span data-ttu-id="52048-294">Geben Sie unter **App Service-Name** den Namen der Azure Stack Hub-Web-App ein.</span><span class="sxs-lookup"><span data-stu-id="52048-294">Enter the Azure Stack Hub web app name as the **App service name**.</span></span>

18. <span data-ttu-id="52048-295">Wählen Sie unter **Agent-Auswahl** in der Liste **Agent-Warteschlange** den Eintrag **AzureStack -b Douglas Fir** aus.</span><span class="sxs-lookup"><span data-stu-id="52048-295">Under **Agent selection**, pick **AzureStack -b Douglas Fir** from the **Agent queue** list.</span></span>

19. <span data-ttu-id="52048-296">Wählen Sie für **Deploy Azure App Service** (Azure App Service bereitstellen) unter **Paket oder Ordner** den gültigen Eintrag für die Umgebung aus.</span><span class="sxs-lookup"><span data-stu-id="52048-296">For **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span> <span data-ttu-id="52048-297">Klicken Sie unter **Datei oder Ordner auswählen** für den **Speicherort** des Ordners auf **OK**.</span><span class="sxs-lookup"><span data-stu-id="52048-297">On **Select File Or Folder**, select **OK** for the folder **Location**.</span></span>

20. <span data-ttu-id="52048-298">Suchen Sie auf der Registerkarte **Variable** nach der Variablen mit dem Namen `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`.</span><span class="sxs-lookup"><span data-stu-id="52048-298">On the **Variable** tab, find the variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`.</span></span> <span data-ttu-id="52048-299">Legen Sie den Wert der Variablen auf **true** und ihren Bereich auf **Azure Stack Hub** fest.</span><span class="sxs-lookup"><span data-stu-id="52048-299">Set the variable value to **true**, and set its scope to **Azure Stack Hub**.</span></span>

21. <span data-ttu-id="52048-300">Klicken Sie auf der Registerkarte **Pipeline** für das Artefakt „NorthwindCloud Traders-Web“ auf **Continuous Deployment-Trigger**, und legen Sie **Continuous Deployment-Trigger** auf **Aktiviert** fest.</span><span class="sxs-lookup"><span data-stu-id="52048-300">On the **Pipeline** tab, select the **Continuous deployment trigger** icon for the NorthwindCloud Traders-Web artifact and set the **Continuous deployment trigger** to **Enabled**.</span></span> <span data-ttu-id="52048-301">Gehen Sie für das Artefakt **NorthwindCloud Traders-Vessel** genauso vor.</span><span class="sxs-lookup"><span data-stu-id="52048-301">Do the same thing for the **NorthwindCloud Traders-Vessel** artifact.</span></span>

22. <span data-ttu-id="52048-302">Klicken Sie für die Azure Stack Hub-Umgebung auf das Symbol **Bedingungen vor der Bereitstellung**, und legen Sie den Trigger auf **Nach einem Release** fest.</span><span class="sxs-lookup"><span data-stu-id="52048-302">For the Azure Stack Hub environment, select the **Pre-deployment conditions** icon set the trigger to **After release**.</span></span>

23. <span data-ttu-id="52048-303">Speichern Sie alle Änderungen.</span><span class="sxs-lookup"><span data-stu-id="52048-303">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="52048-304">Bei der vorlagenbasierten Erstellung einer Releasedefinition werden einige Einstellungen für die Releaseaufgaben unter Umständen automatisch als [Umgebungsvariablen](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) definiert.</span><span class="sxs-lookup"><span data-stu-id="52048-304">Some settings for release tasks are automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="52048-305">Diese Einstellungen können nicht in den Aufgabeneinstellungen geändert werden, sondern nur in den übergeordneten Umgebungselementen.</span><span class="sxs-lookup"><span data-stu-id="52048-305">These settings can't be modified in the task settings but can be modified in the parent environment items.</span></span>

## <a name="create-a-release"></a><span data-ttu-id="52048-306">Erstellen eines Release</span><span class="sxs-lookup"><span data-stu-id="52048-306">Create a release</span></span>

1. <span data-ttu-id="52048-307">Öffnen Sie auf der Registerkarte **Pipeline** die Liste **Release**, und wählen Sie **Release erstellen**.</span><span class="sxs-lookup"><span data-stu-id="52048-307">On the **Pipeline** tab, open the **Release** list and select **Create release**.</span></span>

2. <span data-ttu-id="52048-308">Geben Sie eine Beschreibung für das Release ein, vergewissern Sie sich, dass die korrekten Artefakte ausgewählt sind, und wählen Sie anschließend **Erstellen**.</span><span class="sxs-lookup"><span data-stu-id="52048-308">Enter a description for the release, check to see that the correct artifacts are selected, and then select **Create**.</span></span> <span data-ttu-id="52048-309">Kurz darauf erscheint ein Banner mit dem Hinweis, dass das neue Release erstellt wurde, und der Releasename wird als Link angezeigt.</span><span class="sxs-lookup"><span data-stu-id="52048-309">After a few moments, a banner appears indicating that the new release was created and the release name is displayed as a link.</span></span> <span data-ttu-id="52048-310">Wählen Sie den Link aus, um die Zusammenfassungsseite des Release anzuzeigen.</span><span class="sxs-lookup"><span data-stu-id="52048-310">Select the link to see the release summary page.</span></span>

3. <span data-ttu-id="52048-311">Die Zusammenfassungsseite enthält Details zum Release.</span><span class="sxs-lookup"><span data-stu-id="52048-311">The release summary page shows details about the release.</span></span> <span data-ttu-id="52048-312">Im folgenden Screenshot für „Release-2“ wird im Abschnitt **Umgebungen** als **Bereitstellungsstatus** für Azure „IN BEARBEITUNG“ und als Status für Azure Stack Hub „SUCCEEDED“ (ERFOLGREICH) angezeigt.</span><span class="sxs-lookup"><span data-stu-id="52048-312">In the following screen capture for "Release-2", the **Environments** section shows the **Deployment status** for Azure as "IN PROGRESS", and the status for Azure Stack Hub is "SUCCEEDED".</span></span> <span data-ttu-id="52048-313">Wenn sich der Bereitstellungsstatus für die Azure-Umgebung in „SUCCEEDED“ (ERFOLGREICH) ändert, wird ein Banner mit dem Hinweis angezeigt, dass das Release nun genehmigt werden kann.</span><span class="sxs-lookup"><span data-stu-id="52048-313">When the deployment status for the Azure environment changes to "SUCCEEDED", a banner appears indicating that the release is ready for approval.</span></span> <span data-ttu-id="52048-314">Wenn eine Bereitstellung noch aussteht oder nicht erfolgreich war, wird ein blaues Informationssymbol **(i)** angezeigt.</span><span class="sxs-lookup"><span data-stu-id="52048-314">When a deployment is pending or has failed, a blue **(i)** information icon is shown.</span></span> <span data-ttu-id="52048-315">Zeigen Sie mit der Maus auf das Symbol, um ein Popupfenster mit dem Grund für die Verzögerung oder den Fehler anzuzeigen.</span><span class="sxs-lookup"><span data-stu-id="52048-315">Hover over the icon to see a pop-up that contains the reason for delay or failure.</span></span>

4. <span data-ttu-id="52048-316">In anderen Ansichten (z. B. in der Liste mit den Releases) wird auch ein Symbol als Hinweis darauf angezeigt, dass die Genehmigung aussteht.</span><span class="sxs-lookup"><span data-stu-id="52048-316">Other views, like the list of releases, also display an icon that indicates approval is pending.</span></span> <span data-ttu-id="52048-317">Das Popupfenster für dieses Symbol enthält den Umgebungsnamen und weitere Details zur Bereitstellung.</span><span class="sxs-lookup"><span data-stu-id="52048-317">The pop-up for this icon shows the environment name and more details related to the deployment.</span></span> <span data-ttu-id="52048-318">So kann sich ein Administrator ganz einfach über den Gesamtstatus aller Releases informieren und erkennen, bei welchen Releases eine Genehmigung aussteht.</span><span class="sxs-lookup"><span data-stu-id="52048-318">It's easy for an admin see the overall progress of releases and see which releases are waiting for approval.</span></span>

## <a name="monitor-and-track-deployments"></a><span data-ttu-id="52048-319">Überwachen und Nachverfolgen von Bereitstellungen</span><span class="sxs-lookup"><span data-stu-id="52048-319">Monitor and track deployments</span></span>

1. <span data-ttu-id="52048-320">Klicken Sie auf der Zusammenfassungsseite **Release-2** auf **Protokolle**.</span><span class="sxs-lookup"><span data-stu-id="52048-320">On the **Release-2** summary page, select **Logs**.</span></span> <span data-ttu-id="52048-321">Während einer Bereitstellung wird auf dieser Seite das Liveprotokoll des Agents angezeigt.</span><span class="sxs-lookup"><span data-stu-id="52048-321">During a deployment, this page shows the live log from the agent.</span></span> <span data-ttu-id="52048-322">Im linken Bereich wird der Status der einzelnen Vorgänge in der Bereitstellung für die jeweilige Umgebung angezeigt.</span><span class="sxs-lookup"><span data-stu-id="52048-322">The left pane shows the status of each operation in the deployment for each environment.</span></span>

2. <span data-ttu-id="52048-323">Wählen Sie zum Anzeigen von Informationen zur Genehmigung vor oder nach der Bereitstellung das Personensymbol in der Spalte **Aktion** aus. So können Sie ermitteln, wer die Bereitstellung genehmigt (oder abgelehnt) hat und welche Nachricht vorhanden ist.</span><span class="sxs-lookup"><span data-stu-id="52048-323">Select the person icon in the **Action** column for a pre-deployment or post-deployment approval to see who approved (or rejected) the deployment and the message they provided.</span></span>

3. <span data-ttu-id="52048-324">Nach Abschluss der Bereitstellung wird im rechten Bereich die gesamte Protokolldatei angezeigt.</span><span class="sxs-lookup"><span data-stu-id="52048-324">After the deployment finishes, the entire log file is displayed in the right pane.</span></span> <span data-ttu-id="52048-325">Wählen Sie einen beliebigen **Schritt** im linken Bereich aus, um die Protokolldatei für einen einzelnen Schritt (z. B. **Auftrag initialisieren**) anzuzeigen.</span><span class="sxs-lookup"><span data-stu-id="52048-325">Select any **Step** in the left pane to see the log file for a single step, like **Initialize Job**.</span></span> <span data-ttu-id="52048-326">Dank der Möglichkeit zum Anzeigen einzelner Protokolle lassen sich Teile der Gesamtbereitstellung leichter nachverfolgen und debuggen.</span><span class="sxs-lookup"><span data-stu-id="52048-326">The ability to see individual logs makes it easier to trace and debug parts of the overall deployment.</span></span> <span data-ttu-id="52048-327">**Speichern** Sie die Protokolldatei für einen Schritt, oder verwenden Sie die Option **Alle Protokolle als ZIP-Datei herunterladen**.</span><span class="sxs-lookup"><span data-stu-id="52048-327">**Save** the log file for a step or **Download all logs as zip**.</span></span>

4. <span data-ttu-id="52048-328">Öffnen Sie die Registerkarte **Zusammenfassung**, um allgemeine Informationen zum Release anzuzeigen.</span><span class="sxs-lookup"><span data-stu-id="52048-328">Open the **Summary** tab to see general information about the release.</span></span> <span data-ttu-id="52048-329">In dieser Ansicht werden die Details zum Build, die Umgebungen, in denen er bereitgestellt wurde, Bereitstellungsstatus und andere Informationen zum Release angezeigt.</span><span class="sxs-lookup"><span data-stu-id="52048-329">This view shows details about the build, the environments it was deployed to, deployment status, and other information about the release.</span></span>

5. <span data-ttu-id="52048-330">Klicken Sie auf den Link für eine Umgebung (**Azure** oder **Azure Stack Hub**), um Informationen zu vorhandenen und ausstehenden Bereitstellungen für eine bestimmte Umgebung anzuzeigen.</span><span class="sxs-lookup"><span data-stu-id="52048-330">Select an environment link (**Azure** or **Azure Stack Hub**) to see information about existing and pending deployments to a specific environment.</span></span> <span data-ttu-id="52048-331">Mithilfe dieser Ansichten können Sie schnell überprüfen, ob der gleiche Build in beiden Umgebungen bereitgestellt wurde.</span><span class="sxs-lookup"><span data-stu-id="52048-331">Use these views as a quick way to check that the same build was deployed to both environments.</span></span>

6. <span data-ttu-id="52048-332">Öffnen Sie die **bereitgestellte Produktions-App** in einem Browser.</span><span class="sxs-lookup"><span data-stu-id="52048-332">Open the **deployed production app** in a browser.</span></span> <span data-ttu-id="52048-333">Öffnen Sie für die Azure App Service-Website beispielsweise die URL `https://[your-app-name\].azurewebsites.net`.</span><span class="sxs-lookup"><span data-stu-id="52048-333">For example, for the Azure App Services website, open the URL `https://[your-app-name\].azurewebsites.net`.</span></span>

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a><span data-ttu-id="52048-334">Integration von Azure und Azure Stack Hub ergibt eine skalierbare cloudübergreifende Lösung</span><span class="sxs-lookup"><span data-stu-id="52048-334">Integration of Azure and Azure Stack Hub provides a scalable cross-cloud solution</span></span>

<span data-ttu-id="52048-335">Ein flexibler und stabiler Dienst für mehrere Clouds ermöglicht Datensicherheit, Sicherung und Redundanz, konsistente und schnelle Verfügbarkeit, skalierbare Speicherung und Verteilung sowie geokonformes Routing.</span><span class="sxs-lookup"><span data-stu-id="52048-335">A flexible and robust multi-cloud service provides data security, back up and redundancy, consistent and rapid availability, scalable storage and distribution, and geo-compliant routing.</span></span> <span data-ttu-id="52048-336">Mit diesem manuell ausgelösten Prozess wird das zuverlässige und effiziente Umschalten der Last zwischen gehosteten Web-Apps und die sofortige Verfügbarkeit wichtiger Daten sichergestellt.</span><span class="sxs-lookup"><span data-stu-id="52048-336">This manually triggered process ensures reliable and efficient load switching between hosted web apps and immediate availability of crucial data.</span></span>

## <a name="next-steps"></a><span data-ttu-id="52048-337">Nächste Schritte</span><span class="sxs-lookup"><span data-stu-id="52048-337">Next steps</span></span>

- <span data-ttu-id="52048-338">Weitere Informationen zu Azure-Cloudmustern finden Sie unter [Cloudentwurfsmuster](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="52048-338">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
