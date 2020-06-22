---
title: Hybridrelaymuster in Azure und Azure Stack Hub
description: Verwenden Sie das Hybridrelaymuster in Azure und Azure Stack Hub, um eine Verbindung mit von Firewalls geschützten Edgeressourcen herzustellen.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 03b20a20a04f620c977fb20e1ea26f5982e42721
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910435"
---
# <a name="hybrid-relay-pattern"></a><span data-ttu-id="cab8f-103">Hybridrelaymuster</span><span class="sxs-lookup"><span data-stu-id="cab8f-103">Hybrid relay pattern</span></span>

<span data-ttu-id="cab8f-104">Erfahren Sie, wie Sie mit dem Hybrid Relay-Muster und Azure Relay eine Verbindung mit Edge-Ressourcen oder Geräten herstellen, die von Firewalls geschützt werden</span><span class="sxs-lookup"><span data-stu-id="cab8f-104">Learn how to connect to edge resources or devices protected by firewalls using the hybrid relay pattern and Azure Relay.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="cab8f-105">Kontext und Problem</span><span class="sxs-lookup"><span data-stu-id="cab8f-105">Context and problem</span></span>

<span data-ttu-id="cab8f-106">Edge-Geräte befinden sich oft hinter einer Unternehmensfirewall oder einem NAT-Gerät.</span><span class="sxs-lookup"><span data-stu-id="cab8f-106">Edge devices are often behind a corporate firewall or NAT device.</span></span> <span data-ttu-id="cab8f-107">Obwohl sie sicher sind, können sie möglicherweise nicht mit der öffentlichen Cloud oder Edgegeräten in anderen Unternehmensnetzwerken kommunizieren.</span><span class="sxs-lookup"><span data-stu-id="cab8f-107">Although they're secure, they may be unable to communicate with the public cloud or edge devices on other corporate networks.</span></span> <span data-ttu-id="cab8f-108">Es kann notwendig sein, bestimmte Ports und Funktionen Benutzern in der öffentlichen Cloud auf sichere Weise verfügbar zu machen.</span><span class="sxs-lookup"><span data-stu-id="cab8f-108">It may be necessary to expose certain ports and functionality to users in the public cloud in a secure manner.</span></span>

## <a name="solution"></a><span data-ttu-id="cab8f-109">Lösung</span><span class="sxs-lookup"><span data-stu-id="cab8f-109">Solution</span></span>

<span data-ttu-id="cab8f-110">Das Hybrid Relay-Muster verwendet Azure Relay, um einen websockets-Tunnel zwischen zwei Endpunkten einzurichten, die nicht direkt miteinander kommunizieren können.</span><span class="sxs-lookup"><span data-stu-id="cab8f-110">The hybrid relay pattern uses Azure Relay to establish a WebSockets tunnel between two endpoints that can't directly communicate.</span></span> <span data-ttu-id="cab8f-111">Geräte, die nicht lokal sind, aber eine Verbindung mit einem lokalen Endpunkt herstellen müssen, werden mit einem Endpunkt in der öffentlichen Cloud verbunden.</span><span class="sxs-lookup"><span data-stu-id="cab8f-111">Devices that aren't on-premises but need to connect to an on-premises endpoint will connect to an endpoint in the public cloud.</span></span> <span data-ttu-id="cab8f-112">Dieser Endpunkt leitet den Datenverkehr auf vordefinierten Routen über einen sicheren Kanal um.</span><span class="sxs-lookup"><span data-stu-id="cab8f-112">This endpoint will redirect the traffic on predefined routes over a secure channel.</span></span> <span data-ttu-id="cab8f-113">Ein Endpunkt innerhalb der lokalen Umgebung empfängt den Datenverkehr und leitet ihn an das gewünschte Ziel weiter.</span><span class="sxs-lookup"><span data-stu-id="cab8f-113">An endpoint inside the on-premises environment receives the traffic and routes it to the correct destination.</span></span>

![Hybridrelaymuster: Lösungsarchitektur](media/pattern-hybrid-relay/solution-architecture.png)

<span data-ttu-id="cab8f-115">So funktioniert das Hybridrelaymuster:</span><span class="sxs-lookup"><span data-stu-id="cab8f-115">Here's how the hybrid relay pattern works:</span></span>

1. <span data-ttu-id="cab8f-116">Ein Gerät stellt an einem vordefinierten Port eine Verbindung mit einem virtuellen Computer (VM) in Azure her.</span><span class="sxs-lookup"><span data-stu-id="cab8f-116">A device connects to the virtual machine (VM) in Azure, on a predefined port.</span></span>
2. <span data-ttu-id="cab8f-117">Der Datenverkehr wird an die Azure Relay in Azure weitergeleitet.</span><span class="sxs-lookup"><span data-stu-id="cab8f-117">Traffic is forwarded to the Azure Relay in Azure.</span></span>
3. <span data-ttu-id="cab8f-118">Der virtuelle Computer auf Azure Stack Hub, der bereits eine langlebige Verbindung mit dem Azure Relay hergestellt hat, empfängt den Datenverkehr und leitet ihn an das Ziel weiter.</span><span class="sxs-lookup"><span data-stu-id="cab8f-118">The VM on Azure Stack Hub, which has already established a long-lived connection to the Azure Relay, receives the traffic and forwards it on to the destination.</span></span>
4. <span data-ttu-id="cab8f-119">Der lokale Dienst oder Endpunkt verarbeitet die Anforderung.</span><span class="sxs-lookup"><span data-stu-id="cab8f-119">The on-premises service or endpoint processes the request.</span></span>

## <a name="components"></a><span data-ttu-id="cab8f-120">Komponenten</span><span class="sxs-lookup"><span data-stu-id="cab8f-120">Components</span></span>

<span data-ttu-id="cab8f-121">Diese Lösung verwendet die folgenden Komponenten:</span><span class="sxs-lookup"><span data-stu-id="cab8f-121">This solution uses the following components:</span></span>

| <span data-ttu-id="cab8f-122">Ebene</span><span class="sxs-lookup"><span data-stu-id="cab8f-122">Layer</span></span> | <span data-ttu-id="cab8f-123">Komponente</span><span class="sxs-lookup"><span data-stu-id="cab8f-123">Component</span></span> | <span data-ttu-id="cab8f-124">BESCHREIBUNG</span><span class="sxs-lookup"><span data-stu-id="cab8f-124">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="cab8f-125">Azure</span><span class="sxs-lookup"><span data-stu-id="cab8f-125">Azure</span></span> | <span data-ttu-id="cab8f-126">Azure VM</span><span class="sxs-lookup"><span data-stu-id="cab8f-126">Azure VM</span></span> | <span data-ttu-id="cab8f-127">Eine Azure-VM bietet einen öffentlich zugänglichen Endpunkt für die lokale Ressource.</span><span class="sxs-lookup"><span data-stu-id="cab8f-127">An Azure VM provides a publicly accessible endpoint for the on-premises resource.</span></span> |
| | <span data-ttu-id="cab8f-128">Azure Relay</span><span class="sxs-lookup"><span data-stu-id="cab8f-128">Azure Relay</span></span> | <span data-ttu-id="cab8f-129">Eine [Azure Relay](/azure/azure-relay/) stellt die Infrastruktur zum Verwalten des Tunnels und der Verbindung zwischen dem virtuellen Azure-Computer und Azure Stack Hub-VM bereit.</span><span class="sxs-lookup"><span data-stu-id="cab8f-129">An [Azure Relay](/azure/azure-relay/) provides the infrastructure for maintaining the tunnel and connection between the Azure VM and Azure Stack Hub VM.</span></span>|
| <span data-ttu-id="cab8f-130">Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="cab8f-130">Azure Stack Hub</span></span> | <span data-ttu-id="cab8f-131">Compute</span><span class="sxs-lookup"><span data-stu-id="cab8f-131">Compute</span></span> | <span data-ttu-id="cab8f-132">Eine Azure Stack Hub-VM stellt die Serverseite des Hybridrelaytunnels bereit.</span><span class="sxs-lookup"><span data-stu-id="cab8f-132">An Azure Stack Hub VM provides the server-side of the Hybrid Relay tunnel.</span></span> |
| | <span data-ttu-id="cab8f-133">Storage</span><span class="sxs-lookup"><span data-stu-id="cab8f-133">Storage</span></span> | <span data-ttu-id="cab8f-134">Der in Azure Stack Hub bereitgestellte AKS-Engine-Cluster bietet eine skalierbare, resiliente Engine für die Ausführung des Gesichtserkennungs-API-Containers.</span><span class="sxs-lookup"><span data-stu-id="cab8f-134">The AKS engine cluster deployed into Azure Stack Hub provides a scalable, resilient engine to run the Face API container.</span></span>|

## <a name="issues-and-considerations"></a><span data-ttu-id="cab8f-135">Probleme und Überlegungen</span><span class="sxs-lookup"><span data-stu-id="cab8f-135">Issues and considerations</span></span>

<span data-ttu-id="cab8f-136">Beachten Sie die folgenden Punkte bei der Entscheidung, wie diese Lösung implementiert werden soll:</span><span class="sxs-lookup"><span data-stu-id="cab8f-136">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="cab8f-137">Skalierbarkeit</span><span class="sxs-lookup"><span data-stu-id="cab8f-137">Scalability</span></span>

<span data-ttu-id="cab8f-138">Dieses Muster erlaubt auf dem Client und Server nur 1:1-Portzuordnungen.</span><span class="sxs-lookup"><span data-stu-id="cab8f-138">This pattern only allows for 1:1 port mappings on the client and server.</span></span> <span data-ttu-id="cab8f-139">Wenn beispielsweise Port 80 für einen Dienst auf dem Azure-Endpunkt getunnelt ist, kann er nicht für einen anderen Dienst verwendet werden.</span><span class="sxs-lookup"><span data-stu-id="cab8f-139">For example, if port 80 is tunneled for one service on the Azure endpoint, it can't be used for another service.</span></span> <span data-ttu-id="cab8f-140">Portzuordnungen müssen entsprechend geplant werden.</span><span class="sxs-lookup"><span data-stu-id="cab8f-140">Port mappings should be planned accordingly.</span></span> <span data-ttu-id="cab8f-141">Die Azure Relay und VMS sollten entsprechend skaliert werden, um den Datenverkehr zu verarbeiten.</span><span class="sxs-lookup"><span data-stu-id="cab8f-141">The Azure Relay and VMs should be appropriately scaled to handle traffic.</span></span>

### <a name="availability"></a><span data-ttu-id="cab8f-142">Verfügbarkeit</span><span class="sxs-lookup"><span data-stu-id="cab8f-142">Availability</span></span>

<span data-ttu-id="cab8f-143">Diese Tunnel und Verbindungen sind nicht redundant.</span><span class="sxs-lookup"><span data-stu-id="cab8f-143">These tunnels and connections aren't redundant.</span></span> <span data-ttu-id="cab8f-144">Um Hochverfügbarkeit sicherzustellen, sollten Sie Code zur Prüfung auf Fehler implementieren.</span><span class="sxs-lookup"><span data-stu-id="cab8f-144">To ensure high-availability, you may want to implement error checking code.</span></span> <span data-ttu-id="cab8f-145">Eine andere Möglichkeit besteht darin, einen Pool mit Azure Relay verbundenen VMS hinter einem Load Balancer zu haben.</span><span class="sxs-lookup"><span data-stu-id="cab8f-145">Another option is to have a pool of Azure Relay-connected VMs behind a load balancer.</span></span>

### <a name="manageability"></a><span data-ttu-id="cab8f-146">Verwaltbarkeit</span><span class="sxs-lookup"><span data-stu-id="cab8f-146">Manageability</span></span>

<span data-ttu-id="cab8f-147">Diese Lösung kann eine Vielzahl von Geräten und Standorten umfassen und damit recht unhandlich werden.</span><span class="sxs-lookup"><span data-stu-id="cab8f-147">This solution can span many devices and locations, which could get unwieldy.</span></span> <span data-ttu-id="cab8f-148">Mit den IoT-Diensten von Azure können neue Standorte und Geräte automatisch online geschaltet und auf dem neuesten Stand gehalten werden.</span><span class="sxs-lookup"><span data-stu-id="cab8f-148">Azure's IoT services can automatically bring new locations and devices online and keep them up to date.</span></span>

### <a name="security"></a><span data-ttu-id="cab8f-149">Sicherheit</span><span class="sxs-lookup"><span data-stu-id="cab8f-149">Security</span></span>

<span data-ttu-id="cab8f-150">Dieses hier gezeigte Muster ermöglicht im Edge-Bereich den ungehinderten Zugriff auf einen Port auf einem internen Gerät.</span><span class="sxs-lookup"><span data-stu-id="cab8f-150">This pattern as shown allows for unfettered access to a port on an internal device from the edge.</span></span> <span data-ttu-id="cab8f-151">Erwägen Sie, dem Dienst auf dem internen Gerät oder vor dem Hybridrelayendpunkt einen Authentifizierungsmechanismus hinzuzufügen.</span><span class="sxs-lookup"><span data-stu-id="cab8f-151">Consider adding an authentication mechanism to the service on the internal device, or in front of the hybrid relay endpoint.</span></span>

## <a name="next-steps"></a><span data-ttu-id="cab8f-152">Nächste Schritte</span><span class="sxs-lookup"><span data-stu-id="cab8f-152">Next steps</span></span>

<span data-ttu-id="cab8f-153">Weitere Informationen zu den in diesem Artikel behandelten Themen:</span><span class="sxs-lookup"><span data-stu-id="cab8f-153">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="cab8f-154">Dieses Muster verwendet Azure Relay.</span><span class="sxs-lookup"><span data-stu-id="cab8f-154">This pattern uses Azure Relay.</span></span> <span data-ttu-id="cab8f-155">Weitere Informationen finden Sie in der [Azure Relay-Dokumentation](/azure/azure-relay/).</span><span class="sxs-lookup"><span data-stu-id="cab8f-155">For more information, see the [Azure Relay documentation](/azure/azure-relay/).</span></span>
- <span data-ttu-id="cab8f-156">Unter [Überlegungen zum Entwurf von Hybrid-Apps](overview-app-design-considerations.md) erfahren Sie mehr zu bewährten Methoden und erhalten Antworten auf weitere Fragen.</span><span class="sxs-lookup"><span data-stu-id="cab8f-156">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and get answers to any additional questions.</span></span>
- <span data-ttu-id="cab8f-157">Im Artikel zur [Azure Stack-Familie mit Produkten und Lösungen](/azure-stack) erfahren Sie mehr über das gesamte Portfolio von Produkten und Lösungen.</span><span class="sxs-lookup"><span data-stu-id="cab8f-157">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="cab8f-158">Wenn Sie bereit sind, das Lösungsbeispiel zu testen, fahren Sie mit dem [Bereitstellungsleitfaden für eine Hybridrelaylösung](https://aka.ms/hybridrelaydeployment) fort.</span><span class="sxs-lookup"><span data-stu-id="cab8f-158">When you're ready to test the solution example, continue with the [Hybrid relay solution deployment guide](https://aka.ms/hybridrelaydeployment).</span></span> <span data-ttu-id="cab8f-159">In diesem Bereitstellungsleitfaden finden Sie detaillierte Anweisungen zum Bereitstellen und Testen der zugehörigen Komponenten.</span><span class="sxs-lookup"><span data-stu-id="cab8f-159">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>