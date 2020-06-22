---
title: Muster zum Trainieren eines Machine Learning-Modells im Edge-Bereich
description: Hier erfahren Sie, wie Sie mit Azure und Azure Stack Hub ein Machine Learning-Modell (ML) im Edge-Bereich trainieren.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: da1012cc8847f221de6bb540ba4e191e43920f41
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910412"
---
# <a name="train-machine-learning-model-at-the-edge-pattern"></a><span data-ttu-id="771fa-103">Muster zum Trainieren eines Machine Learning-Modells im Edge-Bereich</span><span class="sxs-lookup"><span data-stu-id="771fa-103">Train machine learning model at the edge pattern</span></span>

<span data-ttu-id="771fa-104">Generieren Sie portierbare Machine Learning-Modelle (ML) anhand von Daten, die nur lokal vorhanden sind.</span><span class="sxs-lookup"><span data-stu-id="771fa-104">Generate portable machine learning (ML) models from data that only exists on-premises.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="771fa-105">Kontext und Problem</span><span class="sxs-lookup"><span data-stu-id="771fa-105">Context and problem</span></span>

<span data-ttu-id="771fa-106">Viele Organisationen möchten mithilfe von Tools, die ihre Datenanalysten verstehen, Erkenntnisse aus ihren lokalen oder älteren Datenbeständen gewinnen.</span><span class="sxs-lookup"><span data-stu-id="771fa-106">Many organizations would like to unlock insights from their on-premises or legacy data using tools that their data scientists understand.</span></span> <span data-ttu-id="771fa-107">[Azure Machine Learning](/azure/machine-learning/) bietet cloudnative Tools zum Trainieren, Optimieren und Bereitstellen von ML- und Deep Learning-Modellen.</span><span class="sxs-lookup"><span data-stu-id="771fa-107">[Azure Machine Learning](/azure/machine-learning/) provides cloud-native tooling to train, tune, and deploy ML and deep learning models.</span></span>  

<span data-ttu-id="771fa-108">Einige Datenmengen sind jedoch zu groß, um sie in die Cloud zu senden, oder können aus rechtlichen Gründen nicht in die Cloud übertragen werden.</span><span class="sxs-lookup"><span data-stu-id="771fa-108">However, some data is too large send to the cloud or can't be sent to the cloud for regulatory reasons.</span></span> <span data-ttu-id="771fa-109">Anhand dieses Musters können Datenanalysten mit Azure Machine Learning Modelle trainieren, die lokale Daten und Computeressourcen nutzen.</span><span class="sxs-lookup"><span data-stu-id="771fa-109">Using this pattern, data scientists can use Azure Machine Learning to train models using on-premises data and compute.</span></span>

## <a name="solution"></a><span data-ttu-id="771fa-110">Lösung</span><span class="sxs-lookup"><span data-stu-id="771fa-110">Solution</span></span>

<span data-ttu-id="771fa-111">Für das Muster zum Training im Edge-Bereich wird ein in Azure Stack Hub ausgeführter virtueller Computer (VM) verwendet.</span><span class="sxs-lookup"><span data-stu-id="771fa-111">The training at the edge pattern uses a virtual machine (VM) running on Azure Stack Hub.</span></span> <span data-ttu-id="771fa-112">Die VM ist als Computeziel in Azure Machine Learning registriert, sodass sie auf Daten zugreifen kann, die nur lokal verfügbar sind.</span><span class="sxs-lookup"><span data-stu-id="771fa-112">The VM is registered as a compute target in Azure ML, letting it access data only available on-premises.</span></span> <span data-ttu-id="771fa-113">In diesem Fall werden die Daten im Blobspeicher von Azure Stack Hub gespeichert.</span><span class="sxs-lookup"><span data-stu-id="771fa-113">In this case, the data is stored in Azure Stack Hub's blob storage.</span></span>

<span data-ttu-id="771fa-114">Nach dem Training des Modells wird es bei Azure ML registriert, in Containern organisiert und zur Bereitstellung einer Azure Container Registry-Instanz hinzugefügt.</span><span class="sxs-lookup"><span data-stu-id="771fa-114">Once the model is trained, it's registered with Azure ML, containerized, and added to an Azure Container Registry for deployment.</span></span> <span data-ttu-id="771fa-115">Für diese Iteration des Musters muss die Azure Stack Hub-Trainings-VM über das öffentliche Internet erreichbar sein.</span><span class="sxs-lookup"><span data-stu-id="771fa-115">For this iteration of the pattern, the Azure Stack Hub training VM must be reachable over the public internet.</span></span>

<span data-ttu-id="771fa-116">[![Architektur für das Trainieren von ML-Modellen im Edge-Bereich](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span><span class="sxs-lookup"><span data-stu-id="771fa-116">[![Train ML model at the edge architecture](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span></span>

<span data-ttu-id="771fa-117">Das Muster funktioniert wie folgt:</span><span class="sxs-lookup"><span data-stu-id="771fa-117">Here's how the pattern works:</span></span>

1. <span data-ttu-id="771fa-118">Die Azure Stack Hub-VM wird als Computeziel bei Azure ML bereitgestellt und registriert.</span><span class="sxs-lookup"><span data-stu-id="771fa-118">The Azure Stack Hub VM is deployed and registered as a compute target with Azure ML.</span></span>
2. <span data-ttu-id="771fa-119">In Azure ML wird ein Experiment erstellt, das die Azure Stack Hub-VM als Computeziel verwendet.</span><span class="sxs-lookup"><span data-stu-id="771fa-119">An experiment is created in Azure ML that uses the Azure Stack Hub VM as a compute target.</span></span>
3. <span data-ttu-id="771fa-120">Sobald das Modell trainiert ist, wird es registriert und in Containern organisiert.</span><span class="sxs-lookup"><span data-stu-id="771fa-120">Once the model is trained, it's registered and containerized.</span></span>
4. <span data-ttu-id="771fa-121">Das Modell kann nun an Speicherorten bereitgestellt werden, die sich entweder lokal oder in der Cloud befinden.</span><span class="sxs-lookup"><span data-stu-id="771fa-121">The model can now be deployed to locations that are either on-premises or in the cloud.</span></span>

## <a name="components"></a><span data-ttu-id="771fa-122">Komponenten</span><span class="sxs-lookup"><span data-stu-id="771fa-122">Components</span></span>

<span data-ttu-id="771fa-123">Diese Lösung verwendet die folgenden Komponenten:</span><span class="sxs-lookup"><span data-stu-id="771fa-123">This solution uses the following components:</span></span>

| <span data-ttu-id="771fa-124">Ebene</span><span class="sxs-lookup"><span data-stu-id="771fa-124">Layer</span></span> | <span data-ttu-id="771fa-125">Komponente</span><span class="sxs-lookup"><span data-stu-id="771fa-125">Component</span></span> | <span data-ttu-id="771fa-126">BESCHREIBUNG</span><span class="sxs-lookup"><span data-stu-id="771fa-126">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="771fa-127">Azure</span><span class="sxs-lookup"><span data-stu-id="771fa-127">Azure</span></span> | <span data-ttu-id="771fa-128">Azure Machine Learning</span><span class="sxs-lookup"><span data-stu-id="771fa-128">Azure Machine Learning</span></span> | <span data-ttu-id="771fa-129">[Azure Machine Learning](/azure/machine-learning/) orchestriert das Training des ML-Modells.</span><span class="sxs-lookup"><span data-stu-id="771fa-129">[Azure Machine Learning](/azure/machine-learning/) orchestrates the training of the ML model.</span></span> |
| | <span data-ttu-id="771fa-130">Azure Container Registry</span><span class="sxs-lookup"><span data-stu-id="771fa-130">Azure Container Registry</span></span> | <span data-ttu-id="771fa-131">Azure ML packt das Modell in einen Container und speichert es für die Bereitstellung in einer [Azure Container Registry](/azure/container-registry/)-Instanz.</span><span class="sxs-lookup"><span data-stu-id="771fa-131">Azure ML packages the model into a container and stores it in an [Azure Container Registry](/azure/container-registry/) for deployment.</span></span>|
| <span data-ttu-id="771fa-132">Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="771fa-132">Azure Stack Hub</span></span> | <span data-ttu-id="771fa-133">App Service</span><span class="sxs-lookup"><span data-stu-id="771fa-133">App Service</span></span> | <span data-ttu-id="771fa-134">[Azure Stack Hub mit App Service](/azure-stack/operator/azure-stack-app-service-overview) stellt die Basis für die Komponenten im Edge-Bereich bereit.</span><span class="sxs-lookup"><span data-stu-id="771fa-134">[Azure Stack Hub with App Service](/azure-stack/operator/azure-stack-app-service-overview) provides the base for the components at the edge.</span></span> |
| | <span data-ttu-id="771fa-135">Compute</span><span class="sxs-lookup"><span data-stu-id="771fa-135">Compute</span></span> | <span data-ttu-id="771fa-136">Eine Azure Stack Hub-VM, auf der Ubuntu mit Docker ausgeführt wird, wird zum Trainieren des ML-Modells eingesetzt.</span><span class="sxs-lookup"><span data-stu-id="771fa-136">An Azure Stack Hub VM running Ubuntu with Docker is used to train the ML model.</span></span> |
| | <span data-ttu-id="771fa-137">Storage</span><span class="sxs-lookup"><span data-stu-id="771fa-137">Storage</span></span> | <span data-ttu-id="771fa-138">Private Daten können im Blob Storage von Azure Stack Hub gehostet werden.</span><span class="sxs-lookup"><span data-stu-id="771fa-138">Private data can be hosted in Azure Stack Hub blob storage.</span></span> |

## <a name="issues-and-considerations"></a><span data-ttu-id="771fa-139">Probleme und Überlegungen</span><span class="sxs-lookup"><span data-stu-id="771fa-139">Issues and considerations</span></span>

<span data-ttu-id="771fa-140">Beachten Sie die folgenden Punkte bei der Entscheidung, wie diese Lösung implementiert werden soll:</span><span class="sxs-lookup"><span data-stu-id="771fa-140">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="771fa-141">Skalierbarkeit</span><span class="sxs-lookup"><span data-stu-id="771fa-141">Scalability</span></span>

<span data-ttu-id="771fa-142">Damit diese Lösung skalierbar ist, müssen Sie in Azure Stack Hub eine entsprechend große VM für das Training erstellen.</span><span class="sxs-lookup"><span data-stu-id="771fa-142">To enable this solution to scale, you'll need to create an appropriately sized VM on Azure Stack Hub for training.</span></span>

### <a name="availability"></a><span data-ttu-id="771fa-143">Verfügbarkeit</span><span class="sxs-lookup"><span data-stu-id="771fa-143">Availability</span></span>

<span data-ttu-id="771fa-144">Stellen Sie sicher, dass die Trainingsskripts und die Azure Stack Hub-VM Zugriff auf die für das Training verwendeten lokalen Daten haben.</span><span class="sxs-lookup"><span data-stu-id="771fa-144">Ensure that the training scripts and Azure Stack Hub VM have access to the on-premises data used for training.</span></span>

### <a name="manageability"></a><span data-ttu-id="771fa-145">Verwaltbarkeit</span><span class="sxs-lookup"><span data-stu-id="771fa-145">Manageability</span></span>

<span data-ttu-id="771fa-146">Sorgen Sie dafür, dass Modelle und Experimente ordnungsgemäß registriert sowie mit Versionsangaben und mit Tags versehen sind, um Verwechslungen bei der Modellbereitstellung zu vermeiden.</span><span class="sxs-lookup"><span data-stu-id="771fa-146">Ensure that models and experiments are appropriately registered, versioned, and tagged to avoid confusion during model deployment.</span></span>

### <a name="security"></a><span data-ttu-id="771fa-147">Sicherheit</span><span class="sxs-lookup"><span data-stu-id="771fa-147">Security</span></span>

<span data-ttu-id="771fa-148">Dieses Muster ermöglicht Azure Machine Learning den Zugriff auf lokale Daten, die möglicherweise vertraulich sind.</span><span class="sxs-lookup"><span data-stu-id="771fa-148">This pattern lets Azure ML access possible sensitive data on-premises.</span></span> <span data-ttu-id="771fa-149">Vergewissern Sie sich, dass das Konto, das für die SSH-Verbindung mit der Azure Stack Hub-VM verwendet wird, über ein sicheres Kennwort verfügt und dass Trainingsskripts keine Daten speichern oder in die Cloud hochladen.</span><span class="sxs-lookup"><span data-stu-id="771fa-149">Ensure the account used to SSH into Azure Stack Hub VM has a strong password and training scripts don't preserve or upload data to the cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="771fa-150">Nächste Schritte</span><span class="sxs-lookup"><span data-stu-id="771fa-150">Next steps</span></span>

<span data-ttu-id="771fa-151">Weitere Informationen zu den in diesem Artikel behandelten Themen:</span><span class="sxs-lookup"><span data-stu-id="771fa-151">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="771fa-152">Eine Übersicht über ML und verwandte Themen finden Sie in der [Azure Machine Learning-Dokumentation](/azure/machine-learning).</span><span class="sxs-lookup"><span data-stu-id="771fa-152">See the [Azure Machine Learning documentation](/azure/machine-learning) for an overview of ML and related topics.</span></span>
- <span data-ttu-id="771fa-153">Unter [Azure Container Registry](/azure/container-registry/) erfahren Sie, wie Sie Images für Containerbereitstellungen erstellen, speichern und verwalten.</span><span class="sxs-lookup"><span data-stu-id="771fa-153">See [Azure Container Registry](/azure/container-registry/) to learn how to build, store, and manage images for container deployments.</span></span>
- <span data-ttu-id="771fa-154">Weitere Informationen zum Ressourcenanbieter und zur Bereitstellung finden Sie unter [App Service in Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview).</span><span class="sxs-lookup"><span data-stu-id="771fa-154">Refer to [App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) to learn more about the resource provider and how to deploy.</span></span>
- <span data-ttu-id="771fa-155">Unter [Überlegungen zum Entwurf von Hybrid-Apps](overview-app-design-considerations.md) finden Sie weitere Informationen zu den bewährten Methoden und Antworten auf Ihre Fragen.</span><span class="sxs-lookup"><span data-stu-id="771fa-155">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get any additional questions answered.</span></span>
- <span data-ttu-id="771fa-156">Im Artikel zur [Azure Stack-Familie mit Produkten und Lösungen](/azure-stack) erfahren Sie mehr über das gesamte Portfolio von Produkten und Lösungen.</span><span class="sxs-lookup"><span data-stu-id="771fa-156">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="771fa-157">Wenn Sie bereit sind, das Lösungsbeispiel zu testen, fahren Sie mit dem [Bereitstellungsleitfaden zum Trainieren eines ML-Modells im Edge-Bereich](https://aka.ms/edgetrainingdeploy) fort.</span><span class="sxs-lookup"><span data-stu-id="771fa-157">When you're ready to test the solution example, continue with the [Train ML model at the edge deployment guide](https://aka.ms/edgetrainingdeploy).</span></span> <span data-ttu-id="771fa-158">In diesem Bereitstellungsleitfaden finden Sie detaillierte Anweisungen zum Bereitstellen und Testen der zugehörigen Komponenten.</span><span class="sxs-lookup"><span data-stu-id="771fa-158">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>
