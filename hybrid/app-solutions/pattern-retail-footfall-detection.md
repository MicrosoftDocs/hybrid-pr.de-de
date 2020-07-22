---
title: Muster zur Ermittlung der Kundenfrequenz unter Verwendung von Azure und Azure Stack Hub
description: Hier erfahren Sie, wie Sie mithilfe von Azure und Azure Stack Hub eine KI-basierte Lösung zur Ermittlung der Kundenfrequenz implementieren, um den Kundenverkehr in einem Einzelhandelsgeschäft zu analysieren.
author: BryanLa
ms.topic: article
ms.date: 10/31/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 10/31/2019
ms.openlocfilehash: 0bf07bb38537f530a0adb3569c43d53af13b8d56
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910466"
---
# <a name="footfall-detection-pattern"></a><span data-ttu-id="c1f5a-103">Muster zur Ermittlung der Kundenfrequenz</span><span class="sxs-lookup"><span data-stu-id="c1f5a-103">Footfall detection pattern</span></span>

<span data-ttu-id="c1f5a-104">Dieses Muster bietet eine Übersicht über eine Lösung zur Implementierung eines KI-basierten Musters zur Ermittlung der Kundenfrequenz, um den Kundenverkehr in Einzelhandelsgeschäften zu analysieren.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-104">This pattern provides an overview for implementing an AI-based footfall detection solution for analyzing visitor traffic in retail stores.</span></span> <span data-ttu-id="c1f5a-105">Die Lösung generiert mithilfe von Azure, Azure Stack Hub und dem Custom Vision AI Dev Kit Erkenntnisse anhand von Aktionen aus der Praxis.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-105">The solution generates insights from real world actions, using Azure, Azure Stack Hub, and the Custom Vision AI Dev Kit.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="c1f5a-106">Kontext und Problem</span><span class="sxs-lookup"><span data-stu-id="c1f5a-106">Context and problem</span></span>

<span data-ttu-id="c1f5a-107">Die Contoso Stores möchten gerne Erkenntnisse darüber gewinnen, wie Kunden ihre aktuellen Produkte in der derzeitigen Aufteilung und Gestaltung der Ladengeschäfte wahrnehmen.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-107">Contoso Stores would like to gain insights on how customers are receiving their current products in relation to store layout.</span></span> <span data-ttu-id="c1f5a-108">Das Unternehmen kann nicht in jedem Bereich Mitarbeiter platzieren, und es ist nicht effizient, sämtliche Kameraaufnahmen eines gesamten Geschäfts durch ein Analystenteam überprüfen zu lassen.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-108">They're unable to place staff in every section and it's inefficient to have a team of analysts review an entire store's camera footage.</span></span> <span data-ttu-id="c1f5a-109">Darüber hinaus verfügt keines der Ladengeschäfte über genügend Bandbreite, um die Videodaten aller Kameras zu Analysezwecken in die Cloud zu streamen.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-109">In addition, none of their stores have enough bandwidth to stream video from all their cameras to the cloud for analysis.</span></span>

<span data-ttu-id="c1f5a-110">Contoso möchte eine unaufdringliche Möglichkeit finden, demografische Daten, Informationen zur Kundentreue sowie die Reaktionen der Kunden auf Werbeflächen und Produkte im Geschäft zu ermitteln, ohne den Datenschutz für die Kunden zu gefährden.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-110">Contoso would like to find an unobtrusive, privacy-friendly way to determine their customers' demographics, loyalty, and reactions to store displays and products.</span></span>

## <a name="solution"></a><span data-ttu-id="c1f5a-111">Lösung</span><span class="sxs-lookup"><span data-stu-id="c1f5a-111">Solution</span></span>

<span data-ttu-id="c1f5a-112">Das Analysemuster für Einzelhandelsgeschäfte nutzt einen gestaffelten Ansatz im Edge-Bereich zum Ziehen von Rückschlüssen.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-112">This retail analytics pattern uses a tiered approach to inferencing at the edge.</span></span> <span data-ttu-id="c1f5a-113">Mit dem Custom Vision AI Dev Kit werden nur Bilder mit Gesichtern von Menschen zur Analyse an eine private Azure Stack Hub-Instanz gesendet, die Azure Cognitive Services ausführt.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-113">By using the Custom Vision AI Dev Kit, only images with human faces are sent for analysis to a private Azure Stack Hub that runs Azure Cognitive Services.</span></span> <span data-ttu-id="c1f5a-114">Anonymisierte Daten werden aggregiert und an Azure gesendet. Dort werden alle Daten aus allen Geschäften zusammengefasst und in Power BI visualisiert.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-114">Anonymized, aggregated data is sent to Azure for aggregation across all stores and visualization in Power BI.</span></span> <span data-ttu-id="c1f5a-115">Durch eine Kombination aus Edge und öffentlicher Cloud kann Contoso von moderner KI-Technologie profitieren, und gleichzeitig sind die Einhaltung der Unternehmensrichtlinien und der Datenschutz für die Kunden gewährleistet.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-115">Combining the edge and public cloud lets Contoso take advantage of modern AI technology while also remaining in compliance with their corporate policies and respecting their customers' privacy.</span></span>

<span data-ttu-id="c1f5a-116">[![Lösung mit Muster zur Ermittlung der Kundenfrequenz](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)</span><span class="sxs-lookup"><span data-stu-id="c1f5a-116">[![Footfall detection pattern solution](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)</span></span>

<span data-ttu-id="c1f5a-117">So funktioniert die Lösung:</span><span class="sxs-lookup"><span data-stu-id="c1f5a-117">Here's a summary of how the solution works:</span></span>

1. <span data-ttu-id="c1f5a-118">Das Custom Vision AI Dev Kit ruft eine Konfiguration aus IoT Hub an, die die IoT Edge-Runtime und ein ML-Modell installiert.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-118">The Custom Vision AI Dev Kit gets a configuration from IoT Hub, which installs the IoT Edge Runtime and an ML model.</span></span>
2. <span data-ttu-id="c1f5a-119">Wenn das Modell eine Person sieht, nimmt es ein Bild auf und lädt es in Blob Storage für Azure Stack Hub hoch.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-119">If the model sees a person, it takes a picture and uploads it to Azure Stack Hub blob storage.</span></span>
3. <span data-ttu-id="c1f5a-120">Der Blobdienst löst eine Azure-Funktion in Azure Stack Hub aus.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-120">The blob service triggers an Azure Function on Azure Stack Hub.</span></span>
4. <span data-ttu-id="c1f5a-121">Die Azure-Funktion ruft einen Container mit der Gesichtserkennungs-API auf, um demografische und emotionsbezogene Daten aus dem Bild abzurufen.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-121">The Azure Function calls a container with the Face API to get demographic and emotion data from the image.</span></span>
5. <span data-ttu-id="c1f5a-122">Die Daten werden anonymisiert und an einen Azure Event Hubs-Cluster gesendet.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-122">The data is anonymized and sent to an Azure Event Hubs cluster.</span></span>
6. <span data-ttu-id="c1f5a-123">Der Event Hubs-Cluster pusht die Daten in Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-123">The Event Hubs cluster pushes the data to Stream Analytics.</span></span>
7. <span data-ttu-id="c1f5a-124">Stream Analytics aggregiert die Daten und pusht sie in Power BI.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-124">Stream Analytics aggregates the data and pushes it to Power BI.</span></span>

## <a name="components"></a><span data-ttu-id="c1f5a-125">Komponenten</span><span class="sxs-lookup"><span data-stu-id="c1f5a-125">Components</span></span>

<span data-ttu-id="c1f5a-126">Diese Lösung verwendet die folgenden Komponenten:</span><span class="sxs-lookup"><span data-stu-id="c1f5a-126">This solution uses the following components:</span></span>

| <span data-ttu-id="c1f5a-127">Ebene</span><span class="sxs-lookup"><span data-stu-id="c1f5a-127">Layer</span></span> | <span data-ttu-id="c1f5a-128">Komponente</span><span class="sxs-lookup"><span data-stu-id="c1f5a-128">Component</span></span> | <span data-ttu-id="c1f5a-129">BESCHREIBUNG</span><span class="sxs-lookup"><span data-stu-id="c1f5a-129">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="c1f5a-130">Hardware im Ladengeschäft</span><span class="sxs-lookup"><span data-stu-id="c1f5a-130">In-store hardware</span></span> | [<span data-ttu-id="c1f5a-131">Custom Vision AI Dev Kit</span><span class="sxs-lookup"><span data-stu-id="c1f5a-131">Custom Vision AI Dev Kit</span></span>](https://azure.github.io/Vision-AI-DevKit-Pages/) | <span data-ttu-id="c1f5a-132">Diese Komponente führt im Geschäft eine Filterung anhand eines lokalen ML-Modells aus, das nur Bilder von Personen zu Analysezwecken erfasst.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-132">Provides in-store filtering using a local ML model that only captures images of people for analysis.</span></span> <span data-ttu-id="c1f5a-133">Die Daten werden über IoT Hub sicher bereitgestellt und aktualisiert.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-133">Securely provisioned and updated through IoT Hub.</span></span><br><br>|
| <span data-ttu-id="c1f5a-134">Azure</span><span class="sxs-lookup"><span data-stu-id="c1f5a-134">Azure</span></span> | [<span data-ttu-id="c1f5a-135">Azure Event Hubs</span><span class="sxs-lookup"><span data-stu-id="c1f5a-135">Azure Event Hubs</span></span>](/azure/event-hubs/) | <span data-ttu-id="c1f5a-136">Azure Event Hubs bietet eine skalierbare Plattform für die Erfassung anonymisierter Daten, die sich nahtlos in Azure Stream Analytics integrieren lassen.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-136">Azure Event Hubs provides a scalable platform for ingesting anonymized data that integrates neatly with Azure Stream Analytics.</span></span> |
|  | [<span data-ttu-id="c1f5a-137">Azure Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="c1f5a-137">Azure Stream Analytics</span></span>](/azure/stream-analytics/) | <span data-ttu-id="c1f5a-138">Ein Azure Stream Analytics-Auftrag aggregiert die anonymisierten Daten und gruppiert sie zur Visualisierung in 15 Sekunden große Zeitfenster.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-138">An Azure Stream Analytics job aggregates the anonymized data and groups it into 15-second windows for visualization.</span></span> |
|  | [<span data-ttu-id="c1f5a-139">Microsoft Power BI</span><span class="sxs-lookup"><span data-stu-id="c1f5a-139">Microsoft Power BI</span></span>](https://powerbi.microsoft.com/) | <span data-ttu-id="c1f5a-140">Power BI bietet eine benutzerfreundliche Dashboardschnittstelle zur Anzeige der Ausgabe von Azure Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-140">Power BI provides an easy-to-use dashboard interface for viewing the output from Azure Stream Analytics.</span></span> |
| <span data-ttu-id="c1f5a-141">Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="c1f5a-141">Azure Stack Hub</span></span> | [<span data-ttu-id="c1f5a-142">App Service</span><span class="sxs-lookup"><span data-stu-id="c1f5a-142">App Service</span></span>](/azure-stack/operator/azure-stack-app-service-overview.md) | <span data-ttu-id="c1f5a-143">Der App Service-Ressourcenanbieter stellt eine Basis für Edgekomponenten bereit, u. a. Hosting- und Verwaltungsfeatures für Web-Apps/-APIs und Azure Functions.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-143">The App Service resource provider (RP) provides a base for edge components, including hosting and management features for web apps/APIs and Functions.</span></span> |
| | <span data-ttu-id="c1f5a-144">Cluster für die [AKS-Engine](https://github.com/Azure/aks-engine) (Azure Kubernetes Service)</span><span class="sxs-lookup"><span data-stu-id="c1f5a-144">Azure Kubernetes Service [(AKS) Engine](https://github.com/Azure/aks-engine) cluster</span></span> | <span data-ttu-id="c1f5a-145">Der AKS-Ressourcenanbieter mit dem in Azure Stack Hub bereitgestellten AKS-Engine-Cluster bietet eine skalierbare, resiliente Engine für die Ausführung des Containers der Gesichtserkennungs-API.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-145">The AKS RP with AKS-Engine cluster deployed into Azure Stack Hub provides a scalable, resilient engine to run the Face API container.</span></span> |
| | <span data-ttu-id="c1f5a-146">[Container der Gesichtserkennungs-API](/azure/cognitive-services/face/face-how-to-install-containers) in Azure Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="c1f5a-146">Azure Cognitive Services [Face API containers](/azure/cognitive-services/face/face-how-to-install-containers)</span></span>| <span data-ttu-id="c1f5a-147">Der Azure Cognitive Services-Ressourcenanbieter mit Containern der Gesichtserkennungs-API liefert demografische und emotionsbezogene Daten sowie Daten zur Erkennung einmaliger Besucher in das private Netzwerk von Contoso.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-147">The Azure Cognitive Services RP with Face API containers provides demographic, emotion, and unique visitor detection on Contoso's private network.</span></span> |
| | <span data-ttu-id="c1f5a-148">Blob Storage</span><span class="sxs-lookup"><span data-stu-id="c1f5a-148">Blob Storage</span></span> | <span data-ttu-id="c1f5a-149">Die vom AI Dev Kit erfassten Bilder werden in Blob Storage von Azure Stack Hub hochgeladen.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-149">Images captured from the AI Dev Kit are uploaded to Azure Stack Hub's blob storage.</span></span> |
| | <span data-ttu-id="c1f5a-150">Azure-Funktionen</span><span class="sxs-lookup"><span data-stu-id="c1f5a-150">Azure Functions</span></span> | <span data-ttu-id="c1f5a-151">Eine in Azure Stack Hub ausgeführte Azure-Funktion empfängt Eingangsdaten aus dem Blobspeicher und verarbeitet die Interaktionen mit der Gesichtserkennungs-API.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-151">An Azure Function running on Azure Stack Hub receives input from blob storage and manages the interactions with the Face API.</span></span> <span data-ttu-id="c1f5a-152">Die Funktion gibt anonymisierte Daten an einen Event Hubs-Cluster in Azure aus.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-152">It emits anonymized data to an Event Hubs cluster located in Azure.</span></span><br><br>|

## <a name="issues-and-considerations"></a><span data-ttu-id="c1f5a-153">Probleme und Überlegungen</span><span class="sxs-lookup"><span data-stu-id="c1f5a-153">Issues and considerations</span></span>

<span data-ttu-id="c1f5a-154">Beachten Sie die folgenden Punkte bei der Entscheidung, wie diese Lösung implementiert werden soll:</span><span class="sxs-lookup"><span data-stu-id="c1f5a-154">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="c1f5a-155">Skalierbarkeit</span><span class="sxs-lookup"><span data-stu-id="c1f5a-155">Scalability</span></span>

<span data-ttu-id="c1f5a-156">Um diese Lösung auf mehrere Kameras und Standorte zu skalieren, müssen Sie sicherstellen, dass alle Komponenten die entsprechende höhere Last verarbeiten können.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-156">To enable this solution to scale across multiple cameras and locations, you'll need to make sure that all of the components can handle the increased load.</span></span> <span data-ttu-id="c1f5a-157">Möglicherweise müssen Sie Aktionen wie die folgenden ausführen:</span><span class="sxs-lookup"><span data-stu-id="c1f5a-157">You may need to take actions like:</span></span>

- <span data-ttu-id="c1f5a-158">Erhöhen der Anzahl von Stream Analytics-Streamingeinheiten</span><span class="sxs-lookup"><span data-stu-id="c1f5a-158">Increase the number of Stream Analytics streaming units.</span></span>
- <span data-ttu-id="c1f5a-159">Aufskalieren der Bereitstellung der Gesichtserkennungs-API</span><span class="sxs-lookup"><span data-stu-id="c1f5a-159">Scale out the Face API deployment.</span></span>
- <span data-ttu-id="c1f5a-160">Erhöhen des Durchsatzes des Event Hubs-Clusters</span><span class="sxs-lookup"><span data-stu-id="c1f5a-160">Increase the Event Hubs cluster throughput.</span></span>
- <span data-ttu-id="c1f5a-161">In Extremfällen ist möglicherweise eine Migration von Azure Functions zu einem virtuellen Computer erforderlich.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-161">For extreme cases, migrate from Azure Functions to a virtual machine may be necessary.</span></span>

### <a name="availability"></a><span data-ttu-id="c1f5a-162">Verfügbarkeit</span><span class="sxs-lookup"><span data-stu-id="c1f5a-162">Availability</span></span>

<span data-ttu-id="c1f5a-163">Da in dieser Lösung verschiedene Ebenen involviert sind, müssen Sie darüber nachdenken, was bei Netzwerk- oder Stromausfällen passieren soll.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-163">Since this solution is tiered, it's important to think about how to deal with networking or power failures.</span></span> <span data-ttu-id="c1f5a-164">Je nach Geschäftsanforderungen empfiehlt es sich unter Umständen, einen Mechanismus zu implementieren, der Bilder lokal zwischenspeichert und sie dann an Azure Stack Hub weiterleitet, wenn wieder eine Verbindung besteht.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-164">Depending on business needs, you might want to implement a mechanism to cache images locally, then forward to Azure Stack Hub when connectivity returns.</span></span> <span data-ttu-id="c1f5a-165">Wenn der Standort groß genug ist, ist die Bereitstellung einer Data Box Edge-Instanz mit einem auf diesen Standort ausgerichteten Gesichtserkennungs-API-Container möglicherweise eine bessere Option.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-165">If the location is large enough, deploying a Data Box Edge with the Face API container to that location might be a better option.</span></span>

### <a name="manageability"></a><span data-ttu-id="c1f5a-166">Verwaltbarkeit</span><span class="sxs-lookup"><span data-stu-id="c1f5a-166">Manageability</span></span>

<span data-ttu-id="c1f5a-167">Diese Lösung kann eine Vielzahl von Geräten und Standorten umfassen und damit recht unhandlich werden.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-167">This solution can span many devices and locations, which could get unwieldy.</span></span> <span data-ttu-id="c1f5a-168">Um neue Standorte und Geräte automatisch online zu schalten und auf dem neuesten Stand zu halten, können die [IoT-Dienste von Azure](/azure/iot-fundamentals/) verwendet werden.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-168">[Azure's IoT services](/azure/iot-fundamentals/) can be used to automatically bring new locations and devices online and keep them up to date.</span></span>

### <a name="security"></a><span data-ttu-id="c1f5a-169">Sicherheit</span><span class="sxs-lookup"><span data-stu-id="c1f5a-169">Security</span></span>

<span data-ttu-id="c1f5a-170">Da diese Lösung Bilder von Kunden erfasst, ist das Thema Sicherheit von größter Bedeutung.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-170">This solution captures customer images, making security a paramount consideration.</span></span> <span data-ttu-id="c1f5a-171">Stellen Sie sicher, dass alle Speicherkonten mit geeigneten Zugriffsrichtlinien geschützt sind, und rotieren Sie Schlüssel regelmäßig.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-171">Make sure all storage accounts are secured with the proper access policies and rotate keys regularly.</span></span> <span data-ttu-id="c1f5a-172">Sorgen Sie dafür, dass Speicherkonten und Event Hubs über Beibehaltungsrichtlinien verfügen, die die Datenschutzbestimmungen des Unternehmens und des zuständigen Gesetzgebers erfüllen.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-172">Ensure storage accounts and Event Hubs have retention policies that meet corporate and government privacy regulations.</span></span> <span data-ttu-id="c1f5a-173">Vergewissern Sie sich, dass Sie den Benutzerzugriff abstufen.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-173">Also make sure to tier the user access levels.</span></span> <span data-ttu-id="c1f5a-174">Das Abstufen stellt sicher, dass Benutzer nur auf die Daten zugreifen können, die sie für ihre Rolle benötigen.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-174">Tiering ensures that users only have access to the data they need for their role.</span></span>

## <a name="next-steps"></a><span data-ttu-id="c1f5a-175">Nächste Schritte</span><span class="sxs-lookup"><span data-stu-id="c1f5a-175">Next steps</span></span>

<span data-ttu-id="c1f5a-176">Weitere Informationen zu den in diesem Artikel behandelten Themen:</span><span class="sxs-lookup"><span data-stu-id="c1f5a-176">To learn more about the topics introduced in this article:</span></span>

- <span data-ttu-id="c1f5a-177">Sehen Sie sich das [Muster für mehrstufige Daten](https://aka.ms/tiereddatadeploy) an, das vom Muster zur Ermittlung der Kundenfrequenz genutzt wird.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-177">See the [Tiered Data pattern](https://aka.ms/tiereddatadeploy), which is leveraged by the footfall detection pattern.</span></span>
- <span data-ttu-id="c1f5a-178">Weitere Informationen zum benutzerdefinierten maschinellen Sehen finden Sie unter [Custom Vision AI Dev Kit](https://azure.github.io/Vision-AI-DevKit-Pages/).</span><span class="sxs-lookup"><span data-stu-id="c1f5a-178">See the [Custom Vision AI Dev Kit](https://azure.github.io/Vision-AI-DevKit-Pages/) to learn more about using custom vision.</span></span> 

<span data-ttu-id="c1f5a-179">Wenn Sie bereit sind, das Lösungsbeispiel zu testen, fahren Sie mit dem [Bereitstellungsleitfaden für das Muster zur Ermittlung der Kundenfrequenz ](solution-deployment-guide-retail-footfall-detection.md) fort.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-179">When you're ready to test the solution example, continue with the [Footfall detection deployment guide](solution-deployment-guide-retail-footfall-detection.md).</span></span> <span data-ttu-id="c1f5a-180">In diesem Bereitstellungsleitfaden finden Sie detaillierte Anweisungen zum Bereitstellen und Testen der zugehörigen Komponenten.</span><span class="sxs-lookup"><span data-stu-id="c1f5a-180">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>