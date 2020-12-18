---
title: Bereitstellen einer MongoDB-Hochverfügbarkeitslösung in Azure und Azure Stack Hub
description: Erfahren Sie, wie Sie eine MongoDB-Hochverfügbarkeitslösung in Azure und Azure Stack Hub bereitstellen.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 624f032def509d8e42d55807d72176e5fce85910
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 12/09/2020
ms.locfileid: "96901506"
---
# <a name="deploy-a-highly-available-mongodb-solution-across-two-azure-stack-hub-environments"></a><span data-ttu-id="79b06-103">Bereitstellen einer MongoDB-Hochverfügbarkeitslösung in zwei Azure Stack Hub-Umgebungen</span><span class="sxs-lookup"><span data-stu-id="79b06-103">Deploy a highly available MongoDB solution across two Azure Stack Hub environments</span></span>

<span data-ttu-id="79b06-104">In diesem Artikel werden Sie schrittweise durch die automatisierte Bereitstellung eines einfachen MongoDB-Hochverfügbarkeitsclusters (HA) mit einem Standort für die Notfallwiederherstellung (DR) in zwei Azure Stack Hub-Umgebungen geführt.</span><span class="sxs-lookup"><span data-stu-id="79b06-104">This article will step you through an automated deployment of a basic highly available (HA) MongoDB cluster with a disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="79b06-105">Weitere Informationen zu MongoDB und zur Hochverfügbarkeit finden Sie unter [Replikatgruppenmitglieder](https://docs.mongodb.com/manual/core/replica-set-members/).</span><span class="sxs-lookup"><span data-stu-id="79b06-105">To learn more about MongoDB and high availability, see [Replica Set Members](https://docs.mongodb.com/manual/core/replica-set-members/).</span></span>

<span data-ttu-id="79b06-106">In dieser Lösung erstellen Sie eine Beispielumgebung, die Folgendes ermöglicht:</span><span class="sxs-lookup"><span data-stu-id="79b06-106">In this solution, you'll create a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="79b06-107">Orchestrieren einer Bereitstellung in zwei Azure Stack Hub-Instanzen</span><span class="sxs-lookup"><span data-stu-id="79b06-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="79b06-108">Verwenden von Docker zur Minimierung von Abhängigkeitsproblemen mit Azure-API-Profilen</span><span class="sxs-lookup"><span data-stu-id="79b06-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="79b06-109">Bereitstellen eines einfachen MongoDB-Hochverfügbarkeitsclusters mit einem Standort für die Notfallwiederherstellung</span><span class="sxs-lookup"><span data-stu-id="79b06-109">Deploy a basic highly available MongoDB cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="79b06-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="79b06-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="79b06-111">Microsoft Azure Stack Hub ist eine Erweiterung von Azure.</span><span class="sxs-lookup"><span data-stu-id="79b06-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="79b06-112">Mit Azure Stack Hub holen Sie sich die Agilität und Innovation von Cloud Computing in Ihre lokale Umgebung, indem Sie die einzige Hybrid Cloud aktivieren, mit der Sie Hybrid-Apps überall entwickeln und bereitstellen können.</span><span class="sxs-lookup"><span data-stu-id="79b06-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="79b06-113">Im Artikel [Überlegungen zum Entwurf von Hybrid-Apps](overview-app-design-considerations.md) werden die wichtigen Aspekte in Bezug auf die Softwarequalität (Platzierung, Skalierbarkeit, Verfügbarkeit, Resilienz, Verwaltbarkeit und Sicherheit) beschrieben, die für das Entwerfen, Bereitstellen und Betreiben von Hybrid-Apps erforderlich sind.</span><span class="sxs-lookup"><span data-stu-id="79b06-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="79b06-114">Die Überlegungen zum Entwurf dienen als Hilfe beim Optimieren des Designs von Hybrid-Apps, um für Produktionsumgebungen das Auftreten von Problemen zu minimieren.</span><span class="sxs-lookup"><span data-stu-id="79b06-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="79b06-115">Architektur für MongoDB mit Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="79b06-115">Architecture for MongoDB with Azure Stack Hub</span></span>

![Hoch verfügbare MongoDB-Architektur in Azure Stack Hub](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="79b06-117">Voraussetzungen für MongoDB mit Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="79b06-117">Prerequisites for MongoDB with Azure Stack Hub</span></span>

- <span data-ttu-id="79b06-118">Zwei verbundene integrierte Azure Stack Hub-Systeme (Azure Stack Hub).</span><span class="sxs-lookup"><span data-stu-id="79b06-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="79b06-119">Diese Bereitstellung funktioniert nicht für das Azure Stack Development Kit (ASDK).</span><span class="sxs-lookup"><span data-stu-id="79b06-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="79b06-120">Weitere Informationen zu Azure Stack finden Sie unter [Was ist Azure Stack Hub?](https://azure.microsoft.com/products/azure-stack/hub/).</span><span class="sxs-lookup"><span data-stu-id="79b06-120">To learn more about Azure Stack Hub, see [What is Azure Stack Hub?](https://azure.microsoft.com/products/azure-stack/hub/)</span></span>
  - <span data-ttu-id="79b06-121">Ein Mandantenabonnement für jede Azure Stack-Instanz.</span><span class="sxs-lookup"><span data-stu-id="79b06-121">A tenant subscription on each Azure Stack Hub.</span></span> 
  - <span data-ttu-id="79b06-122">**Notieren Sie sich jede Abonnement-ID und den Azure Resource Manager-Endpunkt für jede Azure Stack Hub-Instanz.**</span><span class="sxs-lookup"><span data-stu-id="79b06-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="79b06-123">Ein Dienstprinzipal für Azure Active Directory (Azure AD), der über Berechtigungen für das Mandantenabonnement für jede Azure Stack Hub-Instanz verfügt.</span><span class="sxs-lookup"><span data-stu-id="79b06-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="79b06-124">Möglicherweise müssen Sie zwei Dienstprinzipale erstellen, wenn die Azure Stack Hub-Instanzen in unterschiedlichen Azure AD-Mandanten bereitgestellt sind.</span><span class="sxs-lookup"><span data-stu-id="79b06-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="79b06-125">Informationen zur Erstellung eines Dienstprinzipals für Azure Stack Hub finden Sie unter [Verwenden einer App-Identität für den Zugriff auf Azure Stack Hub-Ressourcen](/azure-stack/user/azure-stack-create-service-principals).</span><span class="sxs-lookup"><span data-stu-id="79b06-125">To learn how to create a service principal for Azure Stack Hub, see [Use an app identity to access Azure Stack Hub resources](/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="79b06-126">**Notieren Sie sich die Anwendungs-ID, den geheimen Clientschlüssel und den Mandantennamen (xxxxx.onmicrosoft.com) jedes Dienstprinzipals.**</span><span class="sxs-lookup"><span data-stu-id="79b06-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="79b06-127">Ubuntu 16.04 syndiziert in jeden Marketplace von Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="79b06-127">Ubuntu 16.04 syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="79b06-128">Weitere Informationen zur Marketplace-Syndikation finden Sie unter [Herunterladen von Marketplace-Elementen in Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span><span class="sxs-lookup"><span data-stu-id="79b06-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
- <span data-ttu-id="79b06-129">[Docker für Windows](https://docs.docker.com/docker-for-windows/), auf Ihrem lokalen Computer installiert.</span><span class="sxs-lookup"><span data-stu-id="79b06-129">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="79b06-130">Abrufen des Docker-Images</span><span class="sxs-lookup"><span data-stu-id="79b06-130">Get the Docker image</span></span>

<span data-ttu-id="79b06-131">Docker-Images für jede Bereitstellung beseitigen Abhängigkeitsprobleme zwischen verschiedenen Versionen von Azure PowerShell.</span><span class="sxs-lookup"><span data-stu-id="79b06-131">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="79b06-132">Stellen Sie sicher, dass Docker für Windows Windows-Container verwendet.</span><span class="sxs-lookup"><span data-stu-id="79b06-132">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="79b06-133">Führen Sie den folgenden Befehl an einer Eingabeaufforderung mit erhöhten Rechten aus, um den Docker-Container mit den Bereitstellungsskripts abzurufen.</span><span class="sxs-lookup"><span data-stu-id="79b06-133">Run the following command in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a><span data-ttu-id="79b06-134">Bereitstellen der Cluster</span><span class="sxs-lookup"><span data-stu-id="79b06-134">Deploy the clusters</span></span>

1. <span data-ttu-id="79b06-135">Sobald das Containerimage erfolgreich abgerufen wurde, starten Sie das Image.</span><span class="sxs-lookup"><span data-stu-id="79b06-135">Once the container image has been successfully pulled, start the image.</span></span>

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. <span data-ttu-id="79b06-136">Nachdem der Container gestartet wurde, erhalten Sie im Container ein PowerShell-Terminal mit erhöhten Rechten.</span><span class="sxs-lookup"><span data-stu-id="79b06-136">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="79b06-137">Wechseln Sie die Verzeichnisse, um zu dem Bereitstellungsskript zu gelangen.</span><span class="sxs-lookup"><span data-stu-id="79b06-137">Change directories to get to the deployment script.</span></span>

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. <span data-ttu-id="79b06-138">Führen Sie die Bereitstellung aus.</span><span class="sxs-lookup"><span data-stu-id="79b06-138">Run the deployment.</span></span> <span data-ttu-id="79b06-139">Geben Sie die Anmeldeinformationen und Ressourcennamen an, wenn erforderlich.</span><span class="sxs-lookup"><span data-stu-id="79b06-139">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="79b06-140">Hochverfügbarkeit bezieht sich auf die Azure Stack Hub-Instanz, in der der hoch verfügbare Cluster bereitgestellt wird.</span><span class="sxs-lookup"><span data-stu-id="79b06-140">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="79b06-141">Notfallwiederherstellung bezieht sich auf die Azure Stack Hub-Instanz, in der der Notfallwiederherstellungscluster bereitgestellt wird.</span><span class="sxs-lookup"><span data-stu-id="79b06-141">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

    ```powershell
    .\Deploy-AzureResourceGroup.ps1 `
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

4. <span data-ttu-id="79b06-142">Geben Sie `Y` ein, um die Installation des NuGet-Anbieters zuzulassen, wodurch die Installation der „2018-03-01-Hybrid“-Module des API-Profils ausgelöst wird.</span><span class="sxs-lookup"><span data-stu-id="79b06-142">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="79b06-143">Die HA-Ressourcen werden zuerst bereitgestellt.</span><span class="sxs-lookup"><span data-stu-id="79b06-143">The HA resources will deploy first.</span></span> <span data-ttu-id="79b06-144">Überwachen Sie die Bereitstellung, und warten Sie, bis sie abgeschlossen ist.</span><span class="sxs-lookup"><span data-stu-id="79b06-144">Monitor the deployment and wait for it to finish.</span></span> <span data-ttu-id="79b06-145">Sobald die Meldung mit dem Hinweis angezeigt wird, dass die HA-Bereitstellung abgeschlossen ist, können Sie im Portal der Azure Stack Hub-Instanz mit HA die bereitgestellten Ressourcen überprüfen.</span><span class="sxs-lookup"><span data-stu-id="79b06-145">Once you have the message stating that the HA deployment is finished, you can check the HA Azure Stack Hub's portal to see the resources deployed.</span></span>

6. <span data-ttu-id="79b06-146">Fahren Sie mit der Bereitstellung von DR-Ressourcen fort, und entscheiden Sie, ob Sie für die Azure Stack Hub-Instanz mit DR eine Jumpbox für die Interaktion mit dem Cluster aktivieren möchten.</span><span class="sxs-lookup"><span data-stu-id="79b06-146">Continue with the deployment of DR resources and decide if you'd like to enable a jump box on the DR Azure Stack Hub to interact with the cluster.</span></span>

7. <span data-ttu-id="79b06-147">Warten Sie, bis die DR-Ressourcenbereitstellung abgeschlossen ist.</span><span class="sxs-lookup"><span data-stu-id="79b06-147">Wait for DR resource deployment to finish.</span></span>

8. <span data-ttu-id="79b06-148">Beenden Sie den Container, nachdem die Bereitstellung der DR-Ressource abgeschlossen wurde.</span><span class="sxs-lookup"><span data-stu-id="79b06-148">Once DR resource deployment has finished, exit the container.</span></span>

  ```powershell
  exit
  ```

## <a name="next-steps"></a><span data-ttu-id="79b06-149">Nächste Schritte</span><span class="sxs-lookup"><span data-stu-id="79b06-149">Next steps</span></span>

- <span data-ttu-id="79b06-150">Wenn Sie die Jumpbox-VM in der Azure Stack Hub-Instanz mit DR aktiviert haben, können Sie eine Verbindung über SSH herstellen und mit dem MongoDB-Cluster interagieren, indem Sie die Mongo-CLI installieren.</span><span class="sxs-lookup"><span data-stu-id="79b06-150">If you enabled the jump box VM on the DR Azure Stack Hub, you can connect via SSH and interact with the MongoDB cluster by installing the mongo CLI.</span></span> <span data-ttu-id="79b06-151">Weitere Informationen zur Interaktion mit MongoDB finden Sie unter [Die Mongo-Shell](https://docs.mongodb.com/manual/mongo/).</span><span class="sxs-lookup"><span data-stu-id="79b06-151">To learn more about interacting with MongoDB, see [The mongo Shell](https://docs.mongodb.com/manual/mongo/).</span></span>
- <span data-ttu-id="79b06-152">Weitere Informationen zu Hybrid Cloud-Apps finden Sie unter [Hybrid Cloud-Lösungen](/azure-stack/user/).</span><span class="sxs-lookup"><span data-stu-id="79b06-152">To learn more about hybrid cloud apps, see [Hybrid Cloud Solutions.](/azure-stack/user/)</span></span>
- <span data-ttu-id="79b06-153">Ändern Sie den Code dieses Beispiels auf [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span><span class="sxs-lookup"><span data-stu-id="79b06-153">Modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>
