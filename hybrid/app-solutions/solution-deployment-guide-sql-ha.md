---
title: Bereitstellen einer SQL Server 2016-Verfügbarkeitsgruppe in Azure und Azure Stack Hub
description: Es wird beschrieben, wie Sie eine SQL Server 2016-Verfügbarkeitsgruppe in Azure und Azure Stack Hub bereitstellen.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 2c20d621247ec8e1278feb092586232cc08d5480
ms.sourcegitcommit: 485a1f97fa1579364e2be1755cadfc5ea89db50e
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 10/08/2020
ms.locfileid: "91852472"
---
# <a name="deploy-a-sql-server-2016-availability-group-to-azure-and-azure-stack-hub"></a><span data-ttu-id="78572-103">Bereitstellen einer SQL Server 2016-Verfügbarkeitsgruppe in Azure und Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="78572-103">Deploy a SQL Server 2016 availability group to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="78572-104">In diesem Artikel werden Sie schrittweise durch die automatisierte Bereitstellung eines einfachen SQL Server 2016 Enterprise-Hochverfügbarkeitsclusters (HA) mit einem asynchronen Standort für die Notfallwiederherstellung (DR) in zwei Azure Stack Hub-Umgebungen geführt.</span><span class="sxs-lookup"><span data-stu-id="78572-104">This article will step you through an automated deployment of a basic highly available (HA) SQL Server 2016 Enterprise cluster with an asynchronous disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="78572-105">Weitere Informationen zu SQL Server 2016 und zur Hochverfügbarkeit finden Sie unter [Always On-Verfügbarkeitsgruppen: eine Lösung mit Hochverfügbarkeit und Notfallwiederherstellung](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).</span><span class="sxs-lookup"><span data-stu-id="78572-105">To learn more about SQL Server 2016 and high availability, see [Always On availability groups: a high-availability and disaster-recovery solution](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).</span></span>

<span data-ttu-id="78572-106">In dieser Lösung erstellen Sie eine Beispielumgebung, die Folgendes ermöglicht:</span><span class="sxs-lookup"><span data-stu-id="78572-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="78572-107">Orchestrieren einer Bereitstellung in zwei Azure Stack Hub-Instanzen</span><span class="sxs-lookup"><span data-stu-id="78572-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="78572-108">Verwenden von Docker zur Minimierung von Abhängigkeitsproblemen mit Azure-API-Profilen</span><span class="sxs-lookup"><span data-stu-id="78572-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="78572-109">Bereitstellen eines einfachen SQL Server 2016 Enterprise-Hochverfügbarkeitsclusters mit einem Standort für die Notfallwiederherstellung</span><span class="sxs-lookup"><span data-stu-id="78572-109">Deploy a basic highly available SQL Server 2016 Enterprise cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="78572-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="78572-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="78572-111">Microsoft Azure Stack Hub ist eine Erweiterung von Azure.</span><span class="sxs-lookup"><span data-stu-id="78572-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="78572-112">Mit Azure Stack Hub holen Sie sich die Agilität und Innovation von Cloud Computing in Ihre lokale Umgebung, indem Sie die einzige Hybrid Cloud aktivieren, mit der Sie Hybrid-Apps überall entwickeln und bereitstellen können.</span><span class="sxs-lookup"><span data-stu-id="78572-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="78572-113">Im Artikel [Überlegungen zum Entwurf von Hybrid-Apps](overview-app-design-considerations.md) werden die wichtigen Aspekte in Bezug auf die Softwarequalität (Platzierung, Skalierbarkeit, Verfügbarkeit, Resilienz, Verwaltbarkeit und Sicherheit) beschrieben, die für das Entwerfen, Bereitstellen und Betreiben von Hybrid-Apps erforderlich sind.</span><span class="sxs-lookup"><span data-stu-id="78572-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="78572-114">Die Überlegungen zum Entwurf dienen als Hilfe beim Optimieren des Designs von Hybrid-Apps, um für Produktionsumgebungen das Auftreten von Problemen zu minimieren.</span><span class="sxs-lookup"><span data-stu-id="78572-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-sql-server-2016"></a><span data-ttu-id="78572-115">Architektur für SQLServer 2016</span><span class="sxs-lookup"><span data-stu-id="78572-115">Architecture for SQL Server 2016</span></span>

![SQL Server 2016 SQL HA Azure Stack Hub](media/solution-deployment-guide-sql-ha/image1.png)

## <a name="prerequisites-for-sql-server-2016"></a><span data-ttu-id="78572-117">Voraussetzungen für SQL Server 2016</span><span class="sxs-lookup"><span data-stu-id="78572-117">Prerequisites for SQL Server 2016</span></span>

- <span data-ttu-id="78572-118">Zwei verbundene integrierte Azure Stack Hub-Systeme (Azure Stack Hub).</span><span class="sxs-lookup"><span data-stu-id="78572-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="78572-119">Diese Bereitstellung funktioniert nicht für das Azure Stack Development Kit (ASDK).</span><span class="sxs-lookup"><span data-stu-id="78572-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="78572-120">Weitere Informationen zu Azure Stack Hub finden Sie in der [Übersicht zu Azure Stack](https://azure.microsoft.com/overview/azure-stack/).</span><span class="sxs-lookup"><span data-stu-id="78572-120">To learn more about Azure Stack Hub, see the [Azure Stack overview](https://azure.microsoft.com/overview/azure-stack/).</span></span>
- <span data-ttu-id="78572-121">Ein Mandantenabonnement für jede Azure Stack-Instanz.</span><span class="sxs-lookup"><span data-stu-id="78572-121">A tenant subscription on each Azure Stack Hub.</span></span>
  - <span data-ttu-id="78572-122">**Notieren Sie sich jede Abonnement-ID und den Azure Resource Manager-Endpunkt für jede Azure Stack Hub-Instanz.**</span><span class="sxs-lookup"><span data-stu-id="78572-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="78572-123">Ein Dienstprinzipal für Azure Active Directory (Azure AD), der über Berechtigungen für das Mandantenabonnement für jede Azure Stack Hub-Instanz verfügt.</span><span class="sxs-lookup"><span data-stu-id="78572-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="78572-124">Möglicherweise müssen Sie zwei Dienstprinzipale erstellen, wenn die Azure Stack Hub-Instanzen in unterschiedlichen Azure AD-Mandanten bereitgestellt sind.</span><span class="sxs-lookup"><span data-stu-id="78572-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="78572-125">Informationen zum Erstellen eines Dienstprinzipals für Azure Stack Hub finden Sie unter [Verwenden einer App-Identität für den Zugriff auf Azure Stack Hub-Ressourcen](/azure-stack/user/azure-stack-create-service-principals).</span><span class="sxs-lookup"><span data-stu-id="78572-125">To learn how to create a service principal for Azure Stack Hub, see [Create service principals to give apps access to Azure Stack Hub resources](/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="78572-126">**Notieren Sie sich die Anwendungs-ID, den geheimen Clientschlüssel und den Mandantennamen (xxxxx.onmicrosoft.com) jedes Dienstprinzipals.**</span><span class="sxs-lookup"><span data-stu-id="78572-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="78572-127">SQL Server 2016 Enterprise syndiziert in jeden Marketplace von Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="78572-127">SQL Server 2016 Enterprise syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="78572-128">Weitere Informationen zur Marketplace-Syndikation finden Sie unter [Herunterladen von Marketplace-Elementen in Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span><span class="sxs-lookup"><span data-stu-id="78572-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
    <span data-ttu-id="78572-129">**Stellen Sie sicher, dass Ihre Organisation über die geeigneten SQL-Lizenzen verfügt.**</span><span class="sxs-lookup"><span data-stu-id="78572-129">**Make sure that your organization has the appropriate SQL licenses.**</span></span>
- <span data-ttu-id="78572-130">[Docker für Windows](https://docs.docker.com/docker-for-windows/), auf Ihrem lokalen Computer installiert.</span><span class="sxs-lookup"><span data-stu-id="78572-130">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="78572-131">Abrufen des Docker-Images</span><span class="sxs-lookup"><span data-stu-id="78572-131">Get the Docker image</span></span>

<span data-ttu-id="78572-132">Docker-Images für jede Bereitstellung beseitigen Abhängigkeitsprobleme zwischen verschiedenen Versionen von Azure PowerShell.</span><span class="sxs-lookup"><span data-stu-id="78572-132">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="78572-133">Stellen Sie sicher, dass Docker für Windows Windows-Container verwendet.</span><span class="sxs-lookup"><span data-stu-id="78572-133">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="78572-134">Führen Sie das folgende Skript an einer Eingabeaufforderung mit erhöhten Rechten aus, um den Docker-Container mit den Bereitstellungsskripts abzurufen.</span><span class="sxs-lookup"><span data-stu-id="78572-134">Run the following script in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/sqlserver2016-hadr:1.0.0
    ```

## <a name="deploy-the-availability-group"></a><span data-ttu-id="78572-135">Bereitstellen der Verfügbarkeitsgruppe</span><span class="sxs-lookup"><span data-stu-id="78572-135">Deploy the availability group</span></span>

1. <span data-ttu-id="78572-136">Sobald das Containerimage erfolgreich abgerufen wurde, starten Sie das Image.</span><span class="sxs-lookup"><span data-stu-id="78572-136">Once the container image has been successfully pulled, start the image.</span></span>

      ```powershell  
      docker run -it intelligentedge/sqlserver2016-hadr:1.0.0 powershell
      ```

2. <span data-ttu-id="78572-137">Nachdem der Container gestartet wurde, erhalten Sie im Container ein PowerShell-Terminal mit erhöhten Rechten.</span><span class="sxs-lookup"><span data-stu-id="78572-137">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="78572-138">Wechseln Sie die Verzeichnisse, um zu dem Bereitstellungsskript zu gelangen.</span><span class="sxs-lookup"><span data-stu-id="78572-138">Change directories to get to the deployment script.</span></span>

      ```powershell  
      cd .\SQLHADRDemo\
      ```

3. <span data-ttu-id="78572-139">Führen Sie die Bereitstellung aus.</span><span class="sxs-lookup"><span data-stu-id="78572-139">Run the deployment.</span></span> <span data-ttu-id="78572-140">Geben Sie die Anmeldeinformationen und Ressourcennamen an, wenn erforderlich.</span><span class="sxs-lookup"><span data-stu-id="78572-140">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="78572-141">Hochverfügbarkeit bezieht sich auf die Azure Stack Hub-Instanz, in der der hoch verfügbare Cluster bereitgestellt wird.</span><span class="sxs-lookup"><span data-stu-id="78572-141">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="78572-142">Notfallwiederherstellung bezieht sich auf die Azure Stack Hub-Instanz, in der der Notfallwiederherstellungscluster bereitgestellt wird.</span><span class="sxs-lookup"><span data-stu-id="78572-142">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

      ```powershell
      > .\Deploy-AzureResourceGroup.ps1 `
      -AzureStackApplicationId_HA "applicationIDforHAServicePrincipal" `
      -AzureStackApplicationSercet_HA "clientSecretforHAServicePrincipal" `
      -AADTenantName_HA "hatenantname.onmicrosoft.com" `
      -AzureStackResourceGroup_HA "haresourcegroupname" `
      -AzureStackArmEndpoint_HA "https://management.haazurestack.com" `
      -AzureStackSubscriptionId_HA "haSubscriptionId" `
      -AzureStackApplicationId_DR "applicationIDforDRServicePrincipal" `
      -AzureStackApplicationSercet_DR "ClientSecretforDRServicePrincipal" `
      -AADTenantName_DR "drtenantname.onmicrosoft.com" `
      -AzureStackResourceGroup_DR "drresourcegroupname" `
      -AzureStackArmEndpoint_DR "https://management.drazurestack.com" `
      -AzureStackSubscriptionId_DR "drSubscriptionId"
      ```

4. <span data-ttu-id="78572-143">Geben Sie `Y` ein, um die Installation des NuGet-Anbieters zuzulassen, wodurch die Installation der „2018-03-01-Hybrid“-Module des API-Profils ausgelöst wird.</span><span class="sxs-lookup"><span data-stu-id="78572-143">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="78572-144">Warten Sie, bis die Ressourcenbereitstellung abgeschlossen ist.</span><span class="sxs-lookup"><span data-stu-id="78572-144">Wait for resource deployment to complete.</span></span>

6. <span data-ttu-id="78572-145">Nach Abschluss der Bereitstellung der DR-Ressource beenden Sie den Container.</span><span class="sxs-lookup"><span data-stu-id="78572-145">Once DR resource deployment has completed, exit the container.</span></span>

      ```powershell
      exit
      ```

7. <span data-ttu-id="78572-146">Überprüfen Sie die Bereitstellung, indem Sie sich die Ressourcen im Portal jeder Azure Stack Hub-Instanz ansehen.</span><span class="sxs-lookup"><span data-stu-id="78572-146">Inspect the deployment by viewing the resources in each Azure Stack Hub's portal.</span></span> <span data-ttu-id="78572-147">Stellen Sie eine Verbindung mit einer SQL-Instanz in der Hochverfügbarkeitsumgebung her, und untersuchen Sie die Verfügbarkeitsgruppe über SQL Server Management Studio (SSMS).</span><span class="sxs-lookup"><span data-stu-id="78572-147">Connect to one of the SQL instances on the HA environment and inspect the Availability Group through SQL Server Management Studio (SSMS).</span></span>

    ![SQL Server 2016 SQL HA](media/solution-deployment-guide-sql-ha/image2.png)

## <a name="next-steps"></a><span data-ttu-id="78572-149">Nächste Schritte</span><span class="sxs-lookup"><span data-stu-id="78572-149">Next steps</span></span>

- <span data-ttu-id="78572-150">Verwenden Sie SQL Server Management Studio, um das Failover für den Cluster manuell auszuführen.</span><span class="sxs-lookup"><span data-stu-id="78572-150">Use SQL Server Management Studio to manually fail over the cluster.</span></span> <span data-ttu-id="78572-151">Weitere Informationen finden Sie unter [Ausführen eines erzwungenen manuellen Failovers einer Always On-Verfügbarkeitsgruppe (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017).</span><span class="sxs-lookup"><span data-stu-id="78572-151">See [Perform a Forced Manual Failover of an Always On Availability Group (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)</span></span>
- <span data-ttu-id="78572-152">Informieren Sie sich über Hybrid Cloud-Apps.</span><span class="sxs-lookup"><span data-stu-id="78572-152">Learn more about hybrid cloud apps.</span></span> <span data-ttu-id="78572-153">Sehen Sie sich die Informationen zu [Hybrid Cloud-Lösungen](/azure-stack/user/) an.</span><span class="sxs-lookup"><span data-stu-id="78572-153">See [Hybrid Cloud Solutions.](/azure-stack/user/)</span></span>
- <span data-ttu-id="78572-154">Ändern Sie den Code dieses Beispiels auf [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span><span class="sxs-lookup"><span data-stu-id="78572-154">Use your own data or modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>