---
title: Bereitstellen einer KI-basierten Lösung zur Ermittlung der Kundenfrequenz in Azure und Azure Stack Hub
description: Es wird beschrieben, wie Sie eine KI-basierte Lösung zur Ermittlung der Kundenfrequenz bereitstellen, um mit Azure und Azure Stack Hub den Kundenverkehr in Einzelhandelsfilialen zu analysieren.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: caedbd4758b9ae8c93cf9bb625ed9aac68bfa196
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895362"
---
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a><span data-ttu-id="61586-103">Bereitstellen einer KI-basierten Lösung zur Ermittlung der Kundenfrequenz mithilfe von Azure und Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="61586-103">Deploy an AI-based footfall detection solution using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="61586-104">In diesem Artikel wird beschrieben, wie eine KI-basierte Lösung bereitgestellt wird, die mithilfe von Azure, Azure Stack Hub und dem Custom Vision AI Dev Kit Erkenntnisse aus der Praxis generiert.</span><span class="sxs-lookup"><span data-stu-id="61586-104">This article describes how to deploy an AI-based solution that generates insights from real world actions by using Azure, Azure Stack Hub, and the Custom Vision AI Dev Kit.</span></span>

<span data-ttu-id="61586-105">In diesem Thema lernen Sie Folgendes:</span><span class="sxs-lookup"><span data-stu-id="61586-105">In this solution, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="61586-106">Bereitstellen von nativen cloudbasierten Anwendungspaketen (CNAB) auf Edge-Ebene</span><span class="sxs-lookup"><span data-stu-id="61586-106">Deploy Cloud Native Application Bundles (CNAB) at the edge.</span></span> 
> - <span data-ttu-id="61586-107">Bereitstellen einer cloudgrenzenübergreifenden App</span><span class="sxs-lookup"><span data-stu-id="61586-107">Deploy an app that spans cloud boundaries.</span></span>
> - <span data-ttu-id="61586-108">Verwenden des Custom Vision AI Dev Kit für das Rückschließen auf Edge-Ebene</span><span class="sxs-lookup"><span data-stu-id="61586-108">Use the Custom Vision AI Dev Kit for inference at the edge.</span></span>

> [!Tip]  
> <span data-ttu-id="61586-109">![Diagramm der Hybridsäulen](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="61586-109">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="61586-110">Microsoft Azure Stack Hub ist eine Erweiterung von Azure.</span><span class="sxs-lookup"><span data-stu-id="61586-110">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="61586-111">Mit Azure Stack Hub holen Sie sich die Agilität und Innovation von Cloud Computing in Ihre lokale Umgebung. Sie erhalten die einzige Hybrid Cloud, mit der Sie Hybrid-Apps überall entwickeln und bereitstellen können.</span><span class="sxs-lookup"><span data-stu-id="61586-111">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="61586-112">Im Artikel [Überlegungen zum Entwurf von Hybrid-Apps](overview-app-design-considerations.md) werden die wichtigen Aspekte in Bezug auf die Softwarequalität (Platzierung, Skalierbarkeit, Verfügbarkeit, Resilienz, Verwaltbarkeit und Sicherheit) beschrieben, die für das Entwerfen, Bereitstellen und Betreiben von Hybrid-Apps erforderlich sind.</span><span class="sxs-lookup"><span data-stu-id="61586-112">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="61586-113">Die Überlegungen zum Entwurf dienen als Hilfe beim Optimieren des Designs von Hybrid-Apps, um für Produktionsumgebungen das Auftreten von Problemen zu minimieren.</span><span class="sxs-lookup"><span data-stu-id="61586-113">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="61586-114">Voraussetzungen</span><span class="sxs-lookup"><span data-stu-id="61586-114">Prerequisites</span></span>

<span data-ttu-id="61586-115">Schritte vor dem Beginnen mit diesem Bereitstellungsleitfaden:</span><span class="sxs-lookup"><span data-stu-id="61586-115">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="61586-116">Lesen Sie das Thema [Muster zur Ermittlung der Kundenfrequenz](pattern-retail-footfall-detection.md).</span><span class="sxs-lookup"><span data-stu-id="61586-116">Review the [Footfall detection pattern](pattern-retail-footfall-detection.md) topic.</span></span>
- <span data-ttu-id="61586-117">Verschaffen Sie sich Benutzerzugriff auf ein Azure Stack Development Kit (ASDK) oder auf eine Instanz eines integrierten Azure Stack Hub-Systems:</span><span class="sxs-lookup"><span data-stu-id="61586-117">Obtain user access to an Azure Stack Development Kit (ASDK) or Azure Stack Hub integrated system instance, with:</span></span>
  - <span data-ttu-id="61586-118">Der [Ressourcenanbieter für Azure App Service in Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) muss installiert sein.</span><span class="sxs-lookup"><span data-stu-id="61586-118">The [Azure App Service on Azure Stack Hub resource provider](/azure-stack/operator/azure-stack-app-service-overview) installed.</span></span> <span data-ttu-id="61586-119">Sie benötigen Benutzerzugriff auf Ihre Azure Stack Hub-Instanz oder müssen für die Installation mit Ihrem Administrator zusammenarbeiten.</span><span class="sxs-lookup"><span data-stu-id="61586-119">You need operator access to your Azure Stack Hub instance, or work with your administrator to install.</span></span>
  - <span data-ttu-id="61586-120">Sie benötigen ein Abonnement für ein Angebot, das ein Azure App Service- und Azure Storage-Kontingent bietet.</span><span class="sxs-lookup"><span data-stu-id="61586-120">A subscription to an offer that provides App Service and Storage quota.</span></span> <span data-ttu-id="61586-121">Sie benötigen Benutzerzugriff, um ein Angebot zu erstellen.</span><span class="sxs-lookup"><span data-stu-id="61586-121">You need operator access to create an offer.</span></span>
- <span data-ttu-id="61586-122">Verschaffen Sie sich Zugriff auf ein Azure-Abonnement.</span><span class="sxs-lookup"><span data-stu-id="61586-122">Obtain access to an Azure subscription.</span></span>
  - <span data-ttu-id="61586-123">Wenn Sie kein Azure-Abonnement haben, können Sie sich [für ein kostenloses Testkonto](https://azure.microsoft.com/free/) registrieren, bevor Sie beginnen.</span><span class="sxs-lookup"><span data-stu-id="61586-123">If you don't have an Azure subscription, sign up for a [free trial account](https://azure.microsoft.com/free/) before you begin.</span></span>
- <span data-ttu-id="61586-124">Erstellen Sie in Ihrem Verzeichnis zwei Dienstprinzipale:</span><span class="sxs-lookup"><span data-stu-id="61586-124">Create two service principals in your directory:</span></span>
  - <span data-ttu-id="61586-125">Einen, der für die Nutzung mit Azure-Ressourcen eingerichtet ist und Zugriff auf den Geltungsbereich des Azure-Abonnements hat.</span><span class="sxs-lookup"><span data-stu-id="61586-125">One set up for use with Azure resources, with access at the Azure subscription scope.</span></span>
  - <span data-ttu-id="61586-126">Einen, der für die Nutzung mit Azure Stack Hub-Ressourcen eingerichtet ist und Zugriff auf den Geltungsbereich des Azure Stack Hub-Abonnements hat.</span><span class="sxs-lookup"><span data-stu-id="61586-126">One set up for use with Azure Stack Hub resources, with access at the Azure Stack Hub subscription scope.</span></span>
  - <span data-ttu-id="61586-127">Informationen zum Erstellen von Dienstprinzipalen und Autorisieren des Zugriffs finden Sie unter [Verwenden einer App-Identität für den Ressourcenzugriff](/azure-stack/operator/azure-stack-create-service-principals).</span><span class="sxs-lookup"><span data-stu-id="61586-127">To learn more about creating service principals and authorizing access, see [Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals).</span></span> <span data-ttu-id="61586-128">Wenn Sie die Azure CLI bevorzugen, siehe [Erstellen eines Azure-Dienstprinzipals mit der Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest&preserve-view=true).</span><span class="sxs-lookup"><span data-stu-id="61586-128">If you prefer to use Azure CLI, see [Create an Azure service principal with Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest&preserve-view=true).</span></span>
- <span data-ttu-id="61586-129">Stellen Sie Azure Cognitive Services in Azure oder Azure Stack Hub bereit.</span><span class="sxs-lookup"><span data-stu-id="61586-129">Deploy Azure Cognitive Services in Azure or Azure Stack Hub.</span></span>
  - <span data-ttu-id="61586-130">[Erfahren Sie zunächst mehr über Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).</span><span class="sxs-lookup"><span data-stu-id="61586-130">First, [learn more about Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).</span></span>
  - <span data-ttu-id="61586-131">Lesen Sie dann [Bereitstellen von Azure Cognitive Services in Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services), um Cognitive Services in Azure Stack Hub bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="61586-131">Then visit [Deploy Azure Cognitive Services to Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services) to deploy Cognitive Services on Azure Stack Hub.</span></span> <span data-ttu-id="61586-132">Zuerst müssen Sie sich für den Zugriff auf die Vorschauversion registrieren.</span><span class="sxs-lookup"><span data-stu-id="61586-132">You first need to sign up for access to the preview.</span></span>
- <span data-ttu-id="61586-133">Sie können anschließend ein nicht konfiguriertes Azure Custom Vision AI Dev Kit klonen oder herunterladen.</span><span class="sxs-lookup"><span data-stu-id="61586-133">Clone or download an unconfigured Azure Custom Vision AI Dev Kit.</span></span> <span data-ttu-id="61586-134">Einzelheiten finden Sie unter [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/).</span><span class="sxs-lookup"><span data-stu-id="61586-134">For details, see the [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/).</span></span>
- <span data-ttu-id="61586-135">Registrieren Sie sich für ein Power BI-Konto.</span><span class="sxs-lookup"><span data-stu-id="61586-135">Sign up for a Power BI account.</span></span>
- <span data-ttu-id="61586-136">Holen Sie sich einen Abonnementschlüssel für die Azure Cognitive Services-Gesichtserkennungs-API und eine Endpunkt-URL.</span><span class="sxs-lookup"><span data-stu-id="61586-136">An Azure Cognitive Services Face API subscription key and endpoint URL.</span></span> <span data-ttu-id="61586-137">Sie können beides über die kostenlose Testversion unter [Cognitive Services ausprobieren](https://azure.microsoft.com/try/cognitive-services/?api=face-api) beziehen.</span><span class="sxs-lookup"><span data-stu-id="61586-137">You can get both with the [Try Cognitive Services](https://azure.microsoft.com/try/cognitive-services/?api=face-api) free trial.</span></span> <span data-ttu-id="61586-138">Oder befolgen Sie die Anweisungen unter [Erstellen eines Cognitive Services-Kontos](/azure/cognitive-services/cognitive-services-apis-create-account).</span><span class="sxs-lookup"><span data-stu-id="61586-138">Or, follow the instructions in [Create a Cognitive Services account](/azure/cognitive-services/cognitive-services-apis-create-account).</span></span>
- <span data-ttu-id="61586-139">Installieren Sie die folgenden Entwicklungsressourcen:</span><span class="sxs-lookup"><span data-stu-id="61586-139">Install the following development resources:</span></span>
  - [<span data-ttu-id="61586-140">Azure CLI 2.0</span><span class="sxs-lookup"><span data-stu-id="61586-140">Azure CLI 2.0</span></span>](/azure-stack/user/azure-stack-version-profiles-azurecli2)
  - [<span data-ttu-id="61586-141">Docker CE</span><span class="sxs-lookup"><span data-stu-id="61586-141">Docker CE</span></span>](https://hub.docker.com/search/?type=edition&offering=community)
  - <span data-ttu-id="61586-142">[Porter](https://porter.sh/).</span><span class="sxs-lookup"><span data-stu-id="61586-142">[Porter](https://porter.sh/).</span></span> <span data-ttu-id="61586-143">Mit Porter können Sie Cloud-Apps bereitstellen, indem Sie CNAB-Paketmanifeste verwenden, die für Sie bereitgestellt werden.</span><span class="sxs-lookup"><span data-stu-id="61586-143">You use Porter to deploy cloud apps using CNAB bundle manifests that are provided for you.</span></span>
  - [<span data-ttu-id="61586-144">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="61586-144">Visual Studio Code</span></span>](https://code.visualstudio.com/)
  - [<span data-ttu-id="61586-145">Azure IoT Tools für Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="61586-145">Azure IoT Tools for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - [<span data-ttu-id="61586-146">Python-Erweiterung für Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="61586-146">Python extension for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - [<span data-ttu-id="61586-147">Python</span><span class="sxs-lookup"><span data-stu-id="61586-147">Python</span></span>](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a><span data-ttu-id="61586-148">Bereitstellen der Hybrid Cloud-App</span><span class="sxs-lookup"><span data-stu-id="61586-148">Deploy the hybrid cloud app</span></span>

<span data-ttu-id="61586-149">Zuerst generieren Sie mit der Porter CLI einen Satz mit Anmeldeinformationen und stellen dann die Cloud-App bereit.</span><span class="sxs-lookup"><span data-stu-id="61586-149">First you use the Porter CLI to generate a credential set, then deploy the cloud app.</span></span>  

1. <span data-ttu-id="61586-150">Klonen oder laden Sie den Code des Lösungsbeispiels von https://github.com/azure-samples/azure-intelligent-edge-patterns herunter.</span><span class="sxs-lookup"><span data-stu-id="61586-150">Clone or download the solution sample code from https://github.com/azure-samples/azure-intelligent-edge-patterns.</span></span> 

1. <span data-ttu-id="61586-151">Porter generiert einen Satz mit Anmeldeinformationen, mit dem die Bereitstellung der App automatisiert wird.</span><span class="sxs-lookup"><span data-stu-id="61586-151">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="61586-152">Bevor Sie den Befehl zur Generierung von Anmeldeinformationen ausführen, stellen Sie sicher, dass Sie über Folgendes verfügen:</span><span class="sxs-lookup"><span data-stu-id="61586-152">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="61586-153">Einen Dienstprinzipal für den Zugriff auf Azure-Ressourcen, einschließlich Dienstprinzipal-ID, Schlüssels und DNS des Mandanten.</span><span class="sxs-lookup"><span data-stu-id="61586-153">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="61586-154">Die Abonnement-ID Ihres Azure-Abonnements.</span><span class="sxs-lookup"><span data-stu-id="61586-154">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="61586-155">Einen Dienstprinzipal für den Zugriff auf Azure Stack Hub-Ressourcen, einschließlich Dienstprinzipal-ID, Schlüssel und DNS des Mandanten.</span><span class="sxs-lookup"><span data-stu-id="61586-155">A service principal for accessing Azure Stack Hub resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="61586-156">Die Abonnement-ID Ihres Azure Stack Hub-Abonnements.</span><span class="sxs-lookup"><span data-stu-id="61586-156">The subscription ID for your Azure Stack Hub subscription.</span></span>
    - <span data-ttu-id="61586-157">Den Schlüssel für die Azure Cognitive Services-Gesichtserkennungs-API und die Ressourcenendpunkt-URL.</span><span class="sxs-lookup"><span data-stu-id="61586-157">Your Azure Cognitive Services Face API key and resource endpoint URL.</span></span>

1. <span data-ttu-id="61586-158">Führen Sie den Porter-Prozess zur Erstellung von Anmeldeinformationen aus, und befolgen Sie die Anweisungen:</span><span class="sxs-lookup"><span data-stu-id="61586-158">Run the Porter credential generation process and follow the prompts:</span></span>

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. <span data-ttu-id="61586-159">Porter erfordert auch die Ausführung einer Reihe von Parametern.</span><span class="sxs-lookup"><span data-stu-id="61586-159">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="61586-160">Erstellen Sie eine Parametertextdatei, und geben Sie die folgenden Name-Wert-Paare ein.</span><span class="sxs-lookup"><span data-stu-id="61586-160">Create a parameter text file and enter the following name/value pairs.</span></span> <span data-ttu-id="61586-161">Fragen Sie Ihren Azure Stack Hub-Administrator, wenn Sie Unterstützung bei den erforderlichen Werten benötigen.</span><span class="sxs-lookup"><span data-stu-id="61586-161">Ask your Azure Stack Hub administrator if you need assistance with any of the required values.</span></span>

   > [!NOTE] 
   > <span data-ttu-id="61586-162">Der Wert `resource suffix` wird verwendet, um sicherzustellen, dass die Ressourcen Ihrer Bereitstellung in Azure eindeutige Namen haben.</span><span class="sxs-lookup"><span data-stu-id="61586-162">The `resource suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="61586-163">Es muss sich um eine eindeutige Zeichenfolge aus Buchstaben und Zahlen mit maximal 8 Zeichen handeln.</span><span class="sxs-lookup"><span data-stu-id="61586-163">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    azure_stack_tenant_arm="Your Azure Stack Hub tenant endpoint"
    azure_stack_storage_suffix="Your Azure Stack Hub storage suffix"
    azure_stack_keyvault_suffix="Your Azure Stack Hub keyVault suffix"
    resource_suffix="A unique string to identify your deployment"
    azure_location="A valid Azure region"
    azure_stack_location="Your Azure Stack Hub location identifier"
    powerbi_display_name="Your first and last name"
    powerbi_principal_name="Your Power BI account email address"
    ```
   <span data-ttu-id="61586-164">Speichern Sie die Textdatei, und notieren Sie sich ihren Pfad.</span><span class="sxs-lookup"><span data-stu-id="61586-164">Save the text file and make a note of its path.</span></span>

1. <span data-ttu-id="61586-165">Sie können die Hybrid Cloud-App jetzt mithilfe von Porter bereitstellen.</span><span class="sxs-lookup"><span data-stu-id="61586-165">You're now ready to deploy the hybrid cloud app using Porter.</span></span> <span data-ttu-id="61586-166">Führen Sie den Installationsbefehl aus, und beobachten Sie, wie Ressourcen für Azure und Azure Stack Hub bereitgestellt werden:</span><span class="sxs-lookup"><span data-stu-id="61586-166">Run the install command and watch as resources are deployed to Azure and Azure Stack Hub:</span></span>

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. <span data-ttu-id="61586-167">Notieren Sie sich nach Abschluss der Bereitstellung die folgenden Werte:</span><span class="sxs-lookup"><span data-stu-id="61586-167">Once deployment is complete, make note of the following values:</span></span>
    - <span data-ttu-id="61586-168">Die Verbindungszeichenfolge der Kamera</span><span class="sxs-lookup"><span data-stu-id="61586-168">The camera's connection string.</span></span>
    - <span data-ttu-id="61586-169">Die Verbindungszeichenfolge des Speicherkontos für Bilder</span><span class="sxs-lookup"><span data-stu-id="61586-169">The image storage account connection string.</span></span>
    - <span data-ttu-id="61586-170">Die Ressourcengruppennamen</span><span class="sxs-lookup"><span data-stu-id="61586-170">The resource group names.</span></span>

## <a name="prepare-the-custom-vision-ai-devkit"></a><span data-ttu-id="61586-171">Vorbereiten des Custom Vision AI Dev Kit</span><span class="sxs-lookup"><span data-stu-id="61586-171">Prepare the Custom Vision AI DevKit</span></span>

<span data-ttu-id="61586-172">Richten Sie als Nächstes das Custom Vision AI Dev Kit wie im [Schnellstart zum Vision AI DevKit ](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/) gezeigt ein.</span><span class="sxs-lookup"><span data-stu-id="61586-172">Next, set up the Custom Vision AI Dev Kit as shown in the [Vision AI DevKit quickstart](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/).</span></span> <span data-ttu-id="61586-173">Sie richten auch Ihre Kamera ein, die Sie mit der im vorherigen Schritt angegebenen Verbindungszeichenfolge testen.</span><span class="sxs-lookup"><span data-stu-id="61586-173">You also set up and test your camera, using the connection string provided in the previous step.</span></span>

## <a name="deploy-the-camera-app"></a><span data-ttu-id="61586-174">Bereitstellen der Kamera-App</span><span class="sxs-lookup"><span data-stu-id="61586-174">Deploy the camera app</span></span>

<span data-ttu-id="61586-175">Generieren Sie mit der Porter CLI einen Satz mit Anmeldeinformationen, und stellen Sie dann die Kamera-App bereit.</span><span class="sxs-lookup"><span data-stu-id="61586-175">Use the Porter CLI to generate a credential set, then deploy the camera app.</span></span>

1. <span data-ttu-id="61586-176">Porter generiert einen Satz mit Anmeldeinformationen, mit dem die Bereitstellung der App automatisiert wird.</span><span class="sxs-lookup"><span data-stu-id="61586-176">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="61586-177">Bevor Sie den Befehl zur Generierung von Anmeldeinformationen ausführen, stellen Sie sicher, dass Sie über Folgendes verfügen:</span><span class="sxs-lookup"><span data-stu-id="61586-177">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="61586-178">Einen Dienstprinzipal für den Zugriff auf Azure-Ressourcen, einschließlich Dienstprinzipal-ID, Schlüssels und DNS des Mandanten.</span><span class="sxs-lookup"><span data-stu-id="61586-178">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="61586-179">Die Abonnement-ID Ihres Azure-Abonnements.</span><span class="sxs-lookup"><span data-stu-id="61586-179">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="61586-180">Die beim Bereitstellen der Cloud-App bereitgestellte Verbindungszeichenfolge für das Imagespeicherkonto.</span><span class="sxs-lookup"><span data-stu-id="61586-180">The image storage account connection string provided when you deployed the cloud app.</span></span>

1. <span data-ttu-id="61586-181">Führen Sie den Porter-Prozess zur Erstellung von Anmeldeinformationen aus, und befolgen Sie die Anweisungen:</span><span class="sxs-lookup"><span data-stu-id="61586-181">Run the Porter credential generation process and follow the prompts:</span></span>

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. <span data-ttu-id="61586-182">Porter erfordert auch die Ausführung einer Reihe von Parametern.</span><span class="sxs-lookup"><span data-stu-id="61586-182">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="61586-183">Erstellen Sie eine Parametertextdatei, und geben Sie den folgenden Text ein.</span><span class="sxs-lookup"><span data-stu-id="61586-183">Create a parameter text file and enter the following text.</span></span> <span data-ttu-id="61586-184">Fragen Sie Ihren Azure Stack Hub-Administrator, falls Ihnen einige erforderliche Werte nicht vorliegen.</span><span class="sxs-lookup"><span data-stu-id="61586-184">Ask your Azure Stack Hub administrator if you don't know some of the required values.</span></span>

    > [!NOTE]
    > <span data-ttu-id="61586-185">Der Wert `deployment suffix` wird verwendet, um sicherzustellen, dass die Ressourcen Ihrer Bereitstellung in Azure eindeutige Namen haben.</span><span class="sxs-lookup"><span data-stu-id="61586-185">The `deployment suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="61586-186">Es muss sich um eine eindeutige Zeichenfolge aus Buchstaben und Zahlen mit maximal 8 Zeichen handeln.</span><span class="sxs-lookup"><span data-stu-id="61586-186">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    <span data-ttu-id="61586-187">Speichern Sie die Textdatei, und notieren Sie sich ihren Pfad.</span><span class="sxs-lookup"><span data-stu-id="61586-187">Save the text file and make a note of its path.</span></span>

4. <span data-ttu-id="61586-188">Sie können die Kamera-App jetzt mithilfe von Porter bereitstellen.</span><span class="sxs-lookup"><span data-stu-id="61586-188">You're now ready to deploy the camera app using Porter.</span></span> <span data-ttu-id="61586-189">Führen Sie den Installationsbefehl aus, und beobachten Sie, wie die IoT Edge-Bereitstellung erstellt wird.</span><span class="sxs-lookup"><span data-stu-id="61586-189">Run the install command and watch as the IoT Edge deployment is created.</span></span>

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. <span data-ttu-id="61586-190">Vergewissern Sie sich, dass die Bereitstellung der Kamera abgeschlossen ist, indem Sie den Kamerafeed bei `https://<camera-ip>:3000/` einsehen (`<camara-ip>` steht für die IP-Adresse der Kamera).</span><span class="sxs-lookup"><span data-stu-id="61586-190">Verify that the camera's deployment is complete by viewing the camera feed at `https://<camera-ip>:3000/`, where `<camara-ip>` is the camera IP address.</span></span> <span data-ttu-id="61586-191">Dieser Schritt kann bis zu 10 Minuten dauern.</span><span class="sxs-lookup"><span data-stu-id="61586-191">This step may take up to 10 minutes.</span></span>

## <a name="configure-azure-stream-analytics"></a><span data-ttu-id="61586-192">Konfigurieren von Azure Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="61586-192">Configure Azure Stream Analytics</span></span>

<span data-ttu-id="61586-193">Da nun die Daten von der Kamera zu Azure Stream Analytics übertragen werden, müssen wir sie manuell für die Kommunikation mit Power BI autorisieren.</span><span class="sxs-lookup"><span data-stu-id="61586-193">Now that data is flowing to Azure Stream Analytics from the camera, we need to manually authorize it to communicate with Power BI.</span></span>

1. <span data-ttu-id="61586-194">Öffnen Sie im Azure-Portal **Alle Ressourcen** und dann den Auftrag *process-footfall\[Ihr_Suffix\]* .</span><span class="sxs-lookup"><span data-stu-id="61586-194">From the Azure portal, open **All Resources**, and the *process-footfall\[yoursuffix\]* job.</span></span>

2. <span data-ttu-id="61586-195">Wählen Sie im Bereich „Stream Analytics-Auftrag“ im Abschnitt **Auftragstopologie** die Option **Ausgaben**.</span><span class="sxs-lookup"><span data-stu-id="61586-195">In the **Job Topology** section of the Stream Analytics job pane, select the **Outputs** option.</span></span>

3. <span data-ttu-id="61586-196">Wählen Sie die Ausgabesenke **traffic-output** aus.</span><span class="sxs-lookup"><span data-stu-id="61586-196">Select the **traffic-output** output sink.</span></span>

4. <span data-ttu-id="61586-197">Wählen Sie **Autorisierung erneuern** aus, und melden Sie sich bei Ihrem Power BI-Konto an.</span><span class="sxs-lookup"><span data-stu-id="61586-197">Select **Renew authorization** and sign in to your Power BI account.</span></span>
  
    ![Aufforderung zum Erneuern der Autorisierung in Power BI](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. <span data-ttu-id="61586-199">Speichern Sie die Ausgabeeinstellungen.</span><span class="sxs-lookup"><span data-stu-id="61586-199">Save the output settings.</span></span>

6. <span data-ttu-id="61586-200">Navigieren Sie zum Bereich **Übersicht**, und klicken Sie auf **Starten**, um das Senden von Daten an Power BI zu starten.</span><span class="sxs-lookup"><span data-stu-id="61586-200">Go to the **Overview** pane and select **Start** to start sending data to Power BI.</span></span>

7. <span data-ttu-id="61586-201">Wählen Sie als Startzeit für die Auftragsausgabe die Option **Jetzt** und anschließend **Starten**.</span><span class="sxs-lookup"><span data-stu-id="61586-201">Select **Now** for job output start time and select **Start**.</span></span> <span data-ttu-id="61586-202">Der Auftragsstatus kann über die Benachrichtigungsleiste angezeigt werden.</span><span class="sxs-lookup"><span data-stu-id="61586-202">You can view the job status in the notification bar.</span></span>

## <a name="create-a-power-bi-dashboard"></a><span data-ttu-id="61586-203">Erstellen eines Power BI-Dashboards</span><span class="sxs-lookup"><span data-stu-id="61586-203">Create a Power BI Dashboard</span></span>

1. <span data-ttu-id="61586-204">Navigieren Sie nach erfolgreichem Abschluss des Auftrags zu [Power BI](https://powerbi.com/), und melden Sie sich mit Ihrem Geschäfts-, Schul- oder Unikonto an.</span><span class="sxs-lookup"><span data-stu-id="61586-204">Once the job succeeds, go to [Power BI](https://powerbi.com/) and sign in with your work or school account.</span></span> <span data-ttu-id="61586-205">Wenn die Abfrage des Stream Analytics-Auftrags Ergebnisse ausgibt, befindet sich das von Ihnen erstellte Dataset *footfall-dataset* auf der Registerkarte **Datasets**.</span><span class="sxs-lookup"><span data-stu-id="61586-205">If the Stream Analytics job query is outputting results, the *footfall-dataset* dataset you created exists under the **Datasets** tab.</span></span>

2. <span data-ttu-id="61586-206">Wählen Sie in Ihrem Power BI-Arbeitsbereich die Option **+ Erstellen** aus, um ein neues Dashboard namens *Footfall Analysis* zu erstellen.</span><span class="sxs-lookup"><span data-stu-id="61586-206">From your Power BI workspace, select **+ Create** to create a new dashboard named *Footfall Analysis.*</span></span>

3. <span data-ttu-id="61586-207">Wählen Sie im oberen Fensterbereich **Kachel hinzufügen** aus.</span><span class="sxs-lookup"><span data-stu-id="61586-207">At the top of the window, select **Add tile**.</span></span> <span data-ttu-id="61586-208">Wählen Sie dann **Benutzerdefinierte Streamingdaten** und anschließend **Weiter** aus.</span><span class="sxs-lookup"><span data-stu-id="61586-208">Then select **Custom Streaming Data** and **Next**.</span></span> <span data-ttu-id="61586-209">Wählen Sie unter **Ihre Datasets** das Dataset **footfall-dataset** aus.</span><span class="sxs-lookup"><span data-stu-id="61586-209">Choose the **footfall-dataset** under **Your Datasets**.</span></span> <span data-ttu-id="61586-210">Wählen Sie in der Dropdownliste **Visualisierungstyp** die Option **Karte** aus, und fügen Sie unter **Felder** die Option **age** hinzu.</span><span class="sxs-lookup"><span data-stu-id="61586-210">Select **Card** from the **Visualization type** dropdown, and add **age** to **Fields**.</span></span> <span data-ttu-id="61586-211">Wählen Sie **Weiter** aus, um einen Namen für die Kachel einzugeben, und wählen Sie dann **Übernehmen** aus, um die Kachel zu erstellen.</span><span class="sxs-lookup"><span data-stu-id="61586-211">Select **Next** to enter a name for the tile, and then select **Apply** to create the tile.</span></span>

4. <span data-ttu-id="61586-212">Sie können nach Wunsch weitere Felder und Karten hinzufügen.</span><span class="sxs-lookup"><span data-stu-id="61586-212">You can add additional fields and cards as desired.</span></span>

## <a name="test-your-solution"></a><span data-ttu-id="61586-213">Testen Ihrer Lösung</span><span class="sxs-lookup"><span data-stu-id="61586-213">Test Your Solution</span></span>

<span data-ttu-id="61586-214">Beobachten Sie, wie sich die Daten auf den Karten ändern, die Sie in Power BI erstellt haben, wenn verschiedene Personen an der Kamera vorbeigehen.</span><span class="sxs-lookup"><span data-stu-id="61586-214">Observe how the data in the cards you created in Power BI changes as different people walk in front of the camera.</span></span> <span data-ttu-id="61586-215">Es kann bis zu 20 Sekunden dauern, bis Rückschlüsse angezeigt werden.</span><span class="sxs-lookup"><span data-stu-id="61586-215">Inferences may take up to 20 seconds to appear once recorded.</span></span>

## <a name="remove-your-solution"></a><span data-ttu-id="61586-216">Entfernen der Lösung</span><span class="sxs-lookup"><span data-stu-id="61586-216">Remove Your Solution</span></span>

<span data-ttu-id="61586-217">Führen Sie in Porter die unten angegebenen Befehle aus, wenn Sie Ihre Lösung entfernen möchten. Verwenden Sie hierbei die gleichen Parameterdateien, die Sie für die Bereitstellung erstellt haben:</span><span class="sxs-lookup"><span data-stu-id="61586-217">If you'd like to remove your solution, run the following commands using Porter, using the same parameter files that you created for deployment:</span></span>

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a><span data-ttu-id="61586-218">Nächste Schritte</span><span class="sxs-lookup"><span data-stu-id="61586-218">Next steps</span></span>

- <span data-ttu-id="61586-219">Erfahren Sie mehr über [Überlegungen zum Entwurf von Hybrid-Apps](overview-app-design-considerations.md).</span><span class="sxs-lookup"><span data-stu-id="61586-219">Learn more about [Hybrid app design considerations](overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="61586-220">Überprüfen Sie den [Code dieses Beispiels auf GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis), und schlagen Sie Verbesserungen vor.</span><span class="sxs-lookup"><span data-stu-id="61586-220">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).</span></span>
