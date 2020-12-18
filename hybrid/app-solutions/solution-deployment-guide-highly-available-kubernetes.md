---
title: Bereitstellen eines hochverfügbaren Kubernetes-Clusters in Azure Stack Hub
description: Hier wird beschrieben, wie Sie mit Azure und Azure Stack Hub eine Kubernetes-Clusterlösung für Hochverfügbarkeit bereitstellen.
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: 91f5856aa670bf3810baa5e5f07dbb7dafc9e3f3
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 12/09/2020
ms.locfileid: "96911921"
---
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a><span data-ttu-id="3c52f-103">Bereitstellen eines Kubernetes-Hochverfügbarkeitsclusters in Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="3c52f-103">Deploy a high availability Kubernetes cluster on Azure Stack Hub</span></span>

<span data-ttu-id="3c52f-104">In diesem Artikel erfahren Sie, wie Sie eine hochverfügbare Kubernetes-Clusterumgebung erstellen, die in mehreren Azure Stack Hub-Instanzen an unterschiedlichen physischen Standorten bereitgestellt wird.</span><span class="sxs-lookup"><span data-stu-id="3c52f-104">This article will show you how to build a highly available Kubernetes cluster environment, deployed on multiple Azure Stack Hub instances, in different physical locations.</span></span>

<span data-ttu-id="3c52f-105">In diesem Lösungsbereitstellungshandbuch lernen Sie Folgendes:</span><span class="sxs-lookup"><span data-stu-id="3c52f-105">In this solution deployment guide, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="3c52f-106">Herunterladen und Vorbereiten der AKS-Engine</span><span class="sxs-lookup"><span data-stu-id="3c52f-106">Download and prepare the AKS Engine</span></span>
> - <span data-ttu-id="3c52f-107">Herstellen einer Verbindung mit der Hilfs-VM der AKS-Engine</span><span class="sxs-lookup"><span data-stu-id="3c52f-107">Connect to the AKS Engine Helper VM</span></span>
> - <span data-ttu-id="3c52f-108">Bereitstellen eines Kubernetes-Clusters</span><span class="sxs-lookup"><span data-stu-id="3c52f-108">Deploy a Kubernetes cluster</span></span>
> - <span data-ttu-id="3c52f-109">Herstellen einer Verbindung mit dem Kubernetes-Cluster</span><span class="sxs-lookup"><span data-stu-id="3c52f-109">Connect to the Kubernetes cluster</span></span>
> - <span data-ttu-id="3c52f-110">Herstellen einer Verbindung zwischen Azure Pipelines und dem Kubernetes-Cluster</span><span class="sxs-lookup"><span data-stu-id="3c52f-110">Connect Azure Pipelines to Kubernetes cluster</span></span>
> - <span data-ttu-id="3c52f-111">Konfigurieren der Überwachung</span><span class="sxs-lookup"><span data-stu-id="3c52f-111">Configure monitoring</span></span>
> - <span data-ttu-id="3c52f-112">Bereitstellen der Anwendung</span><span class="sxs-lookup"><span data-stu-id="3c52f-112">Deploy application</span></span>
> - <span data-ttu-id="3c52f-113">Autoskalieren der Anwendung</span><span class="sxs-lookup"><span data-stu-id="3c52f-113">Autoscale application</span></span>
> - <span data-ttu-id="3c52f-114">Traffic Manager konfigurieren</span><span class="sxs-lookup"><span data-stu-id="3c52f-114">Configure Traffic Manager</span></span>
> - <span data-ttu-id="3c52f-115">Kubernetes aktualisieren</span><span class="sxs-lookup"><span data-stu-id="3c52f-115">Upgrade Kubernetes</span></span>
> - <span data-ttu-id="3c52f-116">Skalieren von Kubernetes</span><span class="sxs-lookup"><span data-stu-id="3c52f-116">Scale Kubernetes</span></span>

> [!Tip]  
> <span data-ttu-id="3c52f-117">![Hybridsäulen](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="3c52f-117">![Hybrid pillars](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="3c52f-118">Microsoft Azure Stack Hub ist eine Erweiterung von Azure.</span><span class="sxs-lookup"><span data-stu-id="3c52f-118">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="3c52f-119">Mit Azure Stack Hub holen Sie sich die Agilität und Innovation von Cloud Computing in Ihre lokale Umgebung. Sie erhalten die einzige Hybrid Cloud, mit der Sie Hybrid-Apps überall entwickeln und bereitstellen können.</span><span class="sxs-lookup"><span data-stu-id="3c52f-119">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="3c52f-120">Im Artikel [Überlegungen zum Entwurf von Hybrid-Apps](overview-app-design-considerations.md) werden die wichtigen Aspekte in Bezug auf die Softwarequalität (Platzierung, Skalierbarkeit, Verfügbarkeit, Resilienz, Verwaltbarkeit und Sicherheit) beschrieben, die für das Entwerfen, Bereitstellen und Betreiben von Hybrid-Apps erforderlich sind.</span><span class="sxs-lookup"><span data-stu-id="3c52f-120">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="3c52f-121">Die Überlegungen zum Entwurf dienen als Hilfe beim Optimieren des Designs von Hybrid-Apps, um für Produktionsumgebungen das Auftreten von Problemen zu minimieren.</span><span class="sxs-lookup"><span data-stu-id="3c52f-121">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="3c52f-122">Voraussetzungen</span><span class="sxs-lookup"><span data-stu-id="3c52f-122">Prerequisites</span></span>

<span data-ttu-id="3c52f-123">Schritte vor dem Beginnen mit diesem Bereitstellungsleitfaden:</span><span class="sxs-lookup"><span data-stu-id="3c52f-123">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="3c52f-124">Lesen Sie den Artikel [Kubernetes-Clustermuster mit Hochverfügbarkeit](pattern-highly-available-kubernetes.md).</span><span class="sxs-lookup"><span data-stu-id="3c52f-124">Review the [High availability Kubernetes cluster pattern](pattern-highly-available-kubernetes.md) article.</span></span>
- <span data-ttu-id="3c52f-125">Sehen Sie sich den Inhalt des [zugehörigen GitHub-Repositorys](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub) an, das zusätzliche Ressourcen enthält, auf die in diesem Artikel verwiesen wird.</span><span class="sxs-lookup"><span data-stu-id="3c52f-125">Review the contents of the [companion GitHub repository](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), which contains additional assets referenced in this article.</span></span>
- <span data-ttu-id="3c52f-126">Sie benötigen ein Konto, mit dem sie auf das [Azure Stack Hub-Benutzerportal](/azure-stack/user/azure-stack-use-portal) zugreifen können und das mindestens die [Berechtigung „Mitwirkender“](/azure-stack/user/azure-stack-manage-permissions) hat.</span><span class="sxs-lookup"><span data-stu-id="3c52f-126">Have an account that can access the [Azure Stack Hub user portal](/azure-stack/user/azure-stack-use-portal), with at least ["contributor" permissions](/azure-stack/user/azure-stack-manage-permissions).</span></span>

## <a name="download-and-prepare-aks-engine"></a><span data-ttu-id="3c52f-127">Herunterladen und Vorbereiten der AKS-Engine</span><span class="sxs-lookup"><span data-stu-id="3c52f-127">Download and prepare AKS Engine</span></span>

<span data-ttu-id="3c52f-128">Die AKS-Engine ist eine Binärdatei, die auf jedem Windows- oder Linux-Host verwendet werden kann, der die Azure Resource Manager-Endpunkte für Azure Stack Hub erreichen kann.</span><span class="sxs-lookup"><span data-stu-id="3c52f-128">AKS Engine is a binary that can be used from any Windows or Linux host that can reach the Azure Stack Hub Azure Resource Manager endpoints.</span></span> <span data-ttu-id="3c52f-129">In diesem Leitfaden wird beschrieben, wie Sie einen neuen virtuellen Linux- oder Windows-Computer in Azure Stack Hub bereitstellen.</span><span class="sxs-lookup"><span data-stu-id="3c52f-129">This guide describes deploying a new Linux (or Windows) VM on Azure Stack Hub.</span></span> <span data-ttu-id="3c52f-130">Er wird später verwendet, wenn die AKS-Engine die Kubernetes-Cluster bereitstellt.</span><span class="sxs-lookup"><span data-stu-id="3c52f-130">It will be used later when AKS Engine deploys the Kubernetes clusters.</span></span>

> [!NOTE]
> <span data-ttu-id="3c52f-131">Sie können auch einen vorhandenen virtuellen Windows- oder Linux-Computer verwenden, um einen Kubernetes-Cluster mithilfe der AKS-Engine in Azure Stack Hub bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="3c52f-131">You can also use an existing Windows or Linux VM to deploy a Kubernetes cluster on Azure Stack Hub using AKS Engine.</span></span>

<span data-ttu-id="3c52f-132">Die Schritt-für-Schritt-Anleitung und die Anforderungen für die AKS-Engine sind hier dokumentiert:</span><span class="sxs-lookup"><span data-stu-id="3c52f-132">The step-by-step process and requirements for AKS Engine are documented here:</span></span>

* <span data-ttu-id="3c52f-133">[Installieren der AKS-Engine unter Linux in Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (oder mithilfe von [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))</span><span class="sxs-lookup"><span data-stu-id="3c52f-133">[Install the AKS Engine on Linux in Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (or using [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))</span></span>

<span data-ttu-id="3c52f-134">Die AKS-Engine ist ein Hilfsprogramm zum Bereitstellen und Betreiben von (nicht verwalteten) Kubernetes-Clustern (in Azure und Azure Stack Hub).</span><span class="sxs-lookup"><span data-stu-id="3c52f-134">AKS Engine is a helper tool to deploy and operate (unmanaged) Kubernetes clusters (in Azure and Azure Stack Hub).</span></span>

<span data-ttu-id="3c52f-135">Die Details und Unterschiede der AKS-Engine in Azure Stack Hub sind nachfolgend beschrieben:</span><span class="sxs-lookup"><span data-stu-id="3c52f-135">The details and differences of AKS Engine on Azure Stack Hub are described here:</span></span>

* [<span data-ttu-id="3c52f-136">Was ist die AKS-Engine in Azure Stack Hub?</span><span class="sxs-lookup"><span data-stu-id="3c52f-136">What is the AKS Engine on Azure Stack Hub?</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* <span data-ttu-id="3c52f-137">[AKS-Engine in Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (auf GitHub)</span><span class="sxs-lookup"><span data-stu-id="3c52f-137">[AKS Engine on Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (on GitHub)</span></span>

<span data-ttu-id="3c52f-138">In der Beispielumgebung wird Terraform zum Automatisieren der Bereitstellung der AKS-Engine-VM verwendet.</span><span class="sxs-lookup"><span data-stu-id="3c52f-138">The sample environment will use Terraform to automate the deployment of the AKS Engine VM.</span></span> <span data-ttu-id="3c52f-139">Sie finden [die Details und den Code im zugehörigen GitHub-Repository](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).</span><span class="sxs-lookup"><span data-stu-id="3c52f-139">You can find the [details and code in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).</span></span>

<span data-ttu-id="3c52f-140">Das Ergebnis dieses Schritts ist eine neue Ressourcengruppe in Azure Stack Hub, die die Hilfs-VM der AKS-Engine und zugehörige Ressourcen enthält:</span><span class="sxs-lookup"><span data-stu-id="3c52f-140">The result of this step is a new resource group on Azure Stack Hub that contains the AKS Engine helper VM and related resources:</span></span>

![AKS-Engine-VM-Ressourcen in Azure Stack Hub](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> <span data-ttu-id="3c52f-142">Wenn Sie die AKS-Engine in einer getrennten Air-Gap-Umgebung bereitstellen müssen, finden Sie unter [Getrennte Azure Stack Hub-Instanzen](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) weitere Informationen.</span><span class="sxs-lookup"><span data-stu-id="3c52f-142">If you have to deploy AKS Engine in a disconnected air-gapped environment, review [Disconnected Azure Stack Hub Instances](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) to learn more.</span></span>

<span data-ttu-id="3c52f-143">Im nächsten Schritt verwenden Sie die neu bereitgestellte AKS-Engine-VM zum Bereitstellen eines Kubernetes-Clusters.</span><span class="sxs-lookup"><span data-stu-id="3c52f-143">In the next step, we'll use the newly deployed AKS Engine VM to deploy a Kubernetes cluster.</span></span>

## <a name="connect-to-the-aks-engine-helper-vm"></a><span data-ttu-id="3c52f-144">Herstellen einer Verbindung mit der Hilfs-VM der AKS-Engine</span><span class="sxs-lookup"><span data-stu-id="3c52f-144">Connect to the AKS Engine helper VM</span></span>

<span data-ttu-id="3c52f-145">Zunächst müssen Sie eine Verbindung mit der zuvor erstellten Hilfs-VM der AKS-Engine herstellen.</span><span class="sxs-lookup"><span data-stu-id="3c52f-145">First you must connect to the previously created AKS Engine helper VM.</span></span>

<span data-ttu-id="3c52f-146">Die VM sollte über eine öffentliche IP-Adresse verfügen und über SSH (Port 22/TCP) erreichbar sein.</span><span class="sxs-lookup"><span data-stu-id="3c52f-146">The VM should have a Public IP Address and should be accessible via SSH (Port 22/TCP).</span></span>

![Übersichtsseite der AKS-Engine-VM](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> <span data-ttu-id="3c52f-148">Sie können ein Tool Ihrer Wahl wie MobaXterm, puTTY oder PowerShell unter Windows 10 verwenden, um per SSH eine Verbindung mit einem virtuellen Linux-Computer herzustellen.</span><span class="sxs-lookup"><span data-stu-id="3c52f-148">You can use a tool of your choice like MobaXterm, puTTY or PowerShell in Windows 10 to connect to a Linux VM using SSH.</span></span>

```console
ssh <username>@<ipaddress>
```

<span data-ttu-id="3c52f-149">Führen Sie nach der Verbindungsherstellung den Befehl `aks-engine` aus.</span><span class="sxs-lookup"><span data-stu-id="3c52f-149">After connecting, run the command `aks-engine`.</span></span> <span data-ttu-id="3c52f-150">Unter [Unterstützte AKS-Engine-Versionen](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) finden Sie weitere Informationen zu den AKS-Engine- und Kubernetes-Versionen.</span><span class="sxs-lookup"><span data-stu-id="3c52f-150">Go to [Supported AKS Engine Versions](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) to learn more about the AKS Engine and Kubernetes versions.</span></span>

![Befehlszeilenbeispiel für „aks-engine“](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a><span data-ttu-id="3c52f-152">Bereitstellen eines Kubernetes-Clusters</span><span class="sxs-lookup"><span data-stu-id="3c52f-152">Deploy a Kubernetes cluster</span></span>

<span data-ttu-id="3c52f-153">Die Hilfs-VM der AKS-Engine selbst hat noch keinen Kubernetes-Cluster in Azure Stack Hub erstellt.</span><span class="sxs-lookup"><span data-stu-id="3c52f-153">The AKS Engine helper VM itself hasn't created a Kubernetes cluster on our Azure Stack Hub, yet.</span></span> <span data-ttu-id="3c52f-154">Die Clustererstellung ist die erste Aktion, die auf der Hilfs-VM der AKS-Engine ausgeführt wird.</span><span class="sxs-lookup"><span data-stu-id="3c52f-154">Creating the cluster is the first action to take in the AKS Engine helper VM.</span></span>

<span data-ttu-id="3c52f-155">Die Vorgehensweise ist hier Schritt für Schritt dokumentiert:</span><span class="sxs-lookup"><span data-stu-id="3c52f-155">The step-by-step process is documented here:</span></span>

* [<span data-ttu-id="3c52f-156">Bereitstellen eines Kubernetes-Cluster mit der AKS-Engine in Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="3c52f-156">Deploy a Kubernetes cluster with the AKS engine on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

<span data-ttu-id="3c52f-157">Das Ergebnis des Befehls `aks-engine deploy` und der Vorbereitungen in den vorherigen Schritten ist ein Kubernetes-Cluster mit vollem Funktionsumfang, der im Mandantenraum der ersten Azure Stack Hub-Instanz bereitgestellt wird.</span><span class="sxs-lookup"><span data-stu-id="3c52f-157">The end result of the `aks-engine deploy` command and the preparations in the previous steps is a fully featured Kubernetes cluster deployed into the tenant space of the first Azure Stack Hub instance.</span></span> <span data-ttu-id="3c52f-158">Der Cluster selbst besteht aus Azure-IaaS-Komponenten wie virtuellen Computern, Lastenausgleichsmodulen, VNETs, Datenträgern usw.</span><span class="sxs-lookup"><span data-stu-id="3c52f-158">The cluster itself consists of Azure IaaS components like VMs, load balancers, VNets, disks, and so on.</span></span>

![Cluster-IaaS-Komponenten: Azure Stack Hub-Portal](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) <span data-ttu-id="3c52f-160">Azure-Lastenausgleich (K8s-API-Endpunkt)</span><span class="sxs-lookup"><span data-stu-id="3c52f-160">Azure load balancer (K8s API Endpoint)</span></span>
2) <span data-ttu-id="3c52f-161">Workerknoten (Agentpool)</span><span class="sxs-lookup"><span data-stu-id="3c52f-161">Worker Nodes (Agent Pool)</span></span>
3) <span data-ttu-id="3c52f-162">Masterknoten</span><span class="sxs-lookup"><span data-stu-id="3c52f-162">Master Nodes</span></span>

<span data-ttu-id="3c52f-163">Der Cluster wird jetzt ordnungsgemäß ausgeführt. Im nächsten Schritt stellen Sie eine Verbindung mit dem Cluster her.</span><span class="sxs-lookup"><span data-stu-id="3c52f-163">The cluster is now up-and-running and in the next step we'll connect to it.</span></span>

## <a name="connect-to-the-kubernetes-cluster"></a><span data-ttu-id="3c52f-164">Herstellen einer Verbindung mit dem Kubernetes-Cluster</span><span class="sxs-lookup"><span data-stu-id="3c52f-164">Connect to the Kubernetes cluster</span></span>

<span data-ttu-id="3c52f-165">Sie können nun eine Verbindung mit dem zuvor erstellten Kubernetes-Cluster herstellen. Verwenden Sie dazu entweder SSH (unter Verwendung des SSH-Schlüssels, der im Rahmen der Bereitstellung angegeben wurde), oder `kubectl` (empfohlen).</span><span class="sxs-lookup"><span data-stu-id="3c52f-165">You can now connect to the previously created Kubernetes cluster, either via SSH (using the SSH key specified as part of the deployment) or via `kubectl` (recommended).</span></span> <span data-ttu-id="3c52f-166">Das Kubernetes-Befehlszeilentool `kubectl` steht [hier](https://kubernetes.io/docs/tasks/tools/install-kubectl/) für Windows, Linux und macOS zur Verfügung.</span><span class="sxs-lookup"><span data-stu-id="3c52f-166">The Kubernetes command-line tool `kubectl` is available for Windows, Linux, and macOS [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).</span></span> <span data-ttu-id="3c52f-167">Es ist bereits vorinstalliert und auf den Masterknoten des Clusters konfiguriert.</span><span class="sxs-lookup"><span data-stu-id="3c52f-167">It's already pre-installed and configured on the master nodes of our cluster.</span></span>

```console
ssh azureuser@<k8s-master-lb-ip>
```

![Ausführen von kubectl auf dem Masterknoten](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

<span data-ttu-id="3c52f-169">Es wird nicht empfohlen, den Masterknoten als Jumpbox für administrative Aufgaben zu verwenden.</span><span class="sxs-lookup"><span data-stu-id="3c52f-169">It's not recommended to use the master node as a jumpbox for administrative tasks.</span></span> <span data-ttu-id="3c52f-170">Die `kubectl`-Konfiguration wird auf den Masterknoten in `.kube/config` sowie auf der AKS-Engine-VM gespeichert.</span><span class="sxs-lookup"><span data-stu-id="3c52f-170">The `kubectl` configuration is stored in `.kube/config` on the master node(s) as well as on the AKS Engine VM.</span></span> <span data-ttu-id="3c52f-171">Sie können die Konfiguration auf einen Administratorcomputer mit Verbindung mit dem Kubernetes-Cluster kopieren und dort den Befehl `kubectl` verwenden.</span><span class="sxs-lookup"><span data-stu-id="3c52f-171">You can copy the configuration to an admin machine with connectivity to the Kubernetes cluster and use the `kubectl` command there.</span></span> <span data-ttu-id="3c52f-172">Die Datei `.kube/config` wird außerdem später zum Konfigurieren einer Dienstverbindung in Azure Pipelines verwendet.</span><span class="sxs-lookup"><span data-stu-id="3c52f-172">The `.kube/config` file is also used later to configure a service connection in Azure Pipelines.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="3c52f-173">Schützen Sie diese Dateien, da sie die Anmeldeinformationen für Ihren Kubernetes-Cluster enthalten.</span><span class="sxs-lookup"><span data-stu-id="3c52f-173">Keep these files secure because they contain the credentials for your Kubernetes cluster.</span></span> <span data-ttu-id="3c52f-174">Ein Angreifer mit Zugriff auf die Datei verfügt über genügend Informationen, um Administratorzugriff zu erhalten.</span><span class="sxs-lookup"><span data-stu-id="3c52f-174">An attacker with access to the file has enough information to gain administrator access to it.</span></span> <span data-ttu-id="3c52f-175">Alle Aktionen, die mit der anfänglichen Datei `.kube/config` durchgeführt werden, werden mithilfe eines Clusteradministratorkontos ausgeführt.</span><span class="sxs-lookup"><span data-stu-id="3c52f-175">All actions that are done using the initial `.kube/config` file are done using a cluster-admin account.</span></span>

<span data-ttu-id="3c52f-176">Sie können jetzt unter Verwendung von `kubectl` verschiedene Befehle ausprobieren, um den Status Ihres Clusters zu überprüfen.</span><span class="sxs-lookup"><span data-stu-id="3c52f-176">You can now try various commands using `kubectl` to check the status of your cluster.</span></span>

```bash
kubectl get nodes
```

```console
NAME                       STATUS   ROLE     VERSION
k8s-linuxpool-35064155-0   Ready    agent    v1.14.8
k8s-linuxpool-35064155-1   Ready    agent    v1.14.8
k8s-linuxpool-35064155-2   Ready    agent    v1.14.8
k8s-master-35064155-0      Ready    master   v1.14.8
```

```bash
kubectl cluster-info
```

```console
Kubernetes master is running at https://aks.***
CoreDNS is running at https://aks.**_/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
kubernetes-dashboard is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy
Metrics-server is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
```

> [!IMPORTANT]
> <span data-ttu-id="3c52f-177">Kubernetes verfügt über ein eigenes Modell für die _ *rollenbasierte Zugriffssteuerung (Role-Based Access Control, RBAC)* \*, mit dem Sie differenzierte Rollendefinitionen und Rollenbindungen erstellen können.</span><span class="sxs-lookup"><span data-stu-id="3c52f-177">Kubernetes has its own _ *Role-based Access Control (RBAC)*\* model that allows you to create fine-grained role definitions and role bindings.</span></span> <span data-ttu-id="3c52f-178">Verwenden Sie nach Möglichkeit diese Methode, um den Zugriff auf den Cluster zu steuern, statt Clusteradministratorberechtigungen zuzuweisen.</span><span class="sxs-lookup"><span data-stu-id="3c52f-178">This is the preferable way to control access to the cluster instead of handing out cluster-admin permissions.</span></span>

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a><span data-ttu-id="3c52f-179">Herstellen einer Verbindung zwischen Azure Pipelines und Kubernetes-Clustern</span><span class="sxs-lookup"><span data-stu-id="3c52f-179">Connect Azure Pipelines to Kubernetes clusters</span></span>

<span data-ttu-id="3c52f-180">Zum Herstellen einer Verbindung zwischen Azure Pipelines und dem neu bereitgestellten Kubernetes-Cluster benötigen Sie die Datei mit der Kube-Konfiguration (`.kube/config`), wie im vorherigen Schritt erläutert.</span><span class="sxs-lookup"><span data-stu-id="3c52f-180">To connect Azure Pipelines to the newly deployed Kubernetes cluster, we need its kube config (`.kube/config`) file as explained in the previous step.</span></span>

* <span data-ttu-id="3c52f-181">Stellen Sie eine Verbindung mit einem der Masterknoten Ihres Kubernetes-Clusters her.</span><span class="sxs-lookup"><span data-stu-id="3c52f-181">Connect to one of the master nodes of your Kubernetes cluster.</span></span>
* <span data-ttu-id="3c52f-182">Kopieren Sie den Inhalt der Datei `.kube/config`.</span><span class="sxs-lookup"><span data-stu-id="3c52f-182">Copy the content of the `.kube/config` file.</span></span>
* <span data-ttu-id="3c52f-183">Navigieren Sie zu „Azure DevOps“ > „Projekteinstellungen“ > „Dienstverbindungen“, um eine neue Kubernetes-Dienstverbindung zu erstellen. (Verwenden Sie „KubeConfig“ als Authentifizierungsmethode.)</span><span class="sxs-lookup"><span data-stu-id="3c52f-183">Go to Azure DevOps > Project Settings > Service Connections to create a new "Kubernetes" service connection (use KubeConfig as Authentication method)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="3c52f-184">Azure Pipelines (oder die zugehörigen Build-Agents) müssen Zugriff auf die Kubernetes-API haben.</span><span class="sxs-lookup"><span data-stu-id="3c52f-184">Azure Pipelines (or its build agents) must have access to the Kubernetes API.</span></span> <span data-ttu-id="3c52f-185">Besteht eine Internetverbindung zwischen Azure Pipelines und dem Azure Stack Hub-Kubernetes-Cluster, müssen Sie einen selbstgehosteten Azure Pipelines-Build-Agent bereitstellen.</span><span class="sxs-lookup"><span data-stu-id="3c52f-185">If there is an Internet connection from Azure Pipelines to the Azure Stack Hub Kubernetes clusetr, you'll need to deploy a self-hosted Azure Pipelines Build Agent.</span></span>

<span data-ttu-id="3c52f-186">Beim Bereitstellen von selbstgehosteten Agents für Azure Pipelines können Sie die Bereitstellung entweder in Azure Stack Hub oder auf einem Computer mit Netzwerkkonnektivität mit allen erforderlichen Verwaltungsendpunkten durchführen.</span><span class="sxs-lookup"><span data-stu-id="3c52f-186">When deploying self-hosted Agents for Azure Pipelines, you may deploy either on Azure Stack Hub, or on a machine with network connectivity to all required management endpoints.</span></span> <span data-ttu-id="3c52f-187">Ausführliche Informationen finden Sie hier:</span><span class="sxs-lookup"><span data-stu-id="3c52f-187">See the details here:</span></span>

* <span data-ttu-id="3c52f-188">[Azure Pipelines-Agents](/azure/devops/pipelines/agents/agents) unter [Windows](/azure/devops/pipelines/agents/v2-windows) oder [Linux](/azure/devops/pipelines/agents/v2-linux)</span><span class="sxs-lookup"><span data-stu-id="3c52f-188">[Azure Pipelines agents](/azure/devops/pipelines/agents/agents) on [Windows](/azure/devops/pipelines/agents/v2-windows) or [Linux](/azure/devops/pipelines/agents/v2-linux)</span></span>

<span data-ttu-id="3c52f-189">Der Abschnitt [Bereitstellungsaspekte (CI/CD)](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) enthält einen Entscheidungsfluss, anhand dessen Sie nachvollziehen können, ob Sie von Microsoft gehostete Agents oder selbstgehostete Agents verwenden sollten.</span><span class="sxs-lookup"><span data-stu-id="3c52f-189">The pattern [Deployment (CI/CD) considerations](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) section contains a decision flow that helps you to understand whether to use Microsoft-hosted agents or self-hosted agents:</span></span>

<span data-ttu-id="3c52f-190">[![Entscheidungsfluss für selbstgehostete Agents](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="3c52f-190">[![decision flow self hosted agents](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span></span>

<span data-ttu-id="3c52f-191">In dieser Beispiellösung enthält die Topologie einen selbstgehosteten Build-Agent in jeder Azure Stack Hub-Instanz.</span><span class="sxs-lookup"><span data-stu-id="3c52f-191">In this sample solution, the topology includes a self-hosted build agent on each Azure Stack Hub instance.</span></span> <span data-ttu-id="3c52f-192">Der Agent kann auf die Azure Stack Hub-Verwaltungsendpunkte und die Kubernetes-Cluster-API-Endpunkte zugreifen.</span><span class="sxs-lookup"><span data-stu-id="3c52f-192">The agent can access the Azure Stack Hub Management Endpoints and the Kubernetes cluster API endpoints.</span></span>

<span data-ttu-id="3c52f-193">[![Nur ausgehender Datenverkehr](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="3c52f-193">[![only outbound traffic](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span></span>

<span data-ttu-id="3c52f-194">Dieser Entwurf erfüllt eine gängige gesetzliche Anforderung, die vorgibt, dass nur ausgehende Verbindungen von der Anwendungslösung hergestellt werden dürfen.</span><span class="sxs-lookup"><span data-stu-id="3c52f-194">This design fulfills a common regulatory requirement, which is to have only outbound connections from the application solution.</span></span>

## <a name="configure-monitoring"></a><span data-ttu-id="3c52f-195">Konfigurieren der Überwachung</span><span class="sxs-lookup"><span data-stu-id="3c52f-195">Configure monitoring</span></span>

<span data-ttu-id="3c52f-196">Sie können [Azure Monitor](/azure/azure-monitor/) für Container verwenden, um die Container in der Lösung zu überwachen.</span><span class="sxs-lookup"><span data-stu-id="3c52f-196">You can use [Azure Monitor](/azure/azure-monitor/) for containers to monitor the containers in the solution.</span></span> <span data-ttu-id="3c52f-197">Dadurch wird Azure Monitor auf den von der AKS-Engine bereitgestellten Kubernetes-Cluster in Azure Stack Hub verwiesen.</span><span class="sxs-lookup"><span data-stu-id="3c52f-197">This points Azure Monitor to the AKS Engine-deployed Kubernetes cluster on Azure Stack Hub.</span></span>

<span data-ttu-id="3c52f-198">Um Azure Monitor für Ihren Cluster zu aktivieren, stehen Ihnen zwei Methoden zur Auswahl.</span><span class="sxs-lookup"><span data-stu-id="3c52f-198">There are two ways to enable Azure Monitor on your cluster.</span></span> <span data-ttu-id="3c52f-199">Beide Methoden erfordern, dass Sie einen Log Analytics-Arbeitsbereich in Azure einrichten.</span><span class="sxs-lookup"><span data-stu-id="3c52f-199">Both ways require you to set up a Log Analytics workspace in Azure.</span></span>

* <span data-ttu-id="3c52f-200">[Methode 1](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) verwendet ein Helm-Chart.</span><span class="sxs-lookup"><span data-stu-id="3c52f-200">[Method one](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) uses a Helm Chart</span></span>
* <span data-ttu-id="3c52f-201">[Methode 2](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) ist Teil der AKS-Engine-Clusterspezifikation.</span><span class="sxs-lookup"><span data-stu-id="3c52f-201">[Method two](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) as part of the AKS Engine cluster specification</span></span>

<span data-ttu-id="3c52f-202">In der Beispieltopologie wird Methode 1 verwendet, die die Automatisierung des Prozesses und eine einfachere Installation von Updates ermöglicht.</span><span class="sxs-lookup"><span data-stu-id="3c52f-202">In the sample topology, "Method one" is used, which allows automation of the process and updates can be installed more easily.</span></span>

<span data-ttu-id="3c52f-203">Für den nächsten Schritt benötigen Sie einen Azure Log Analytics-Arbeitsbereich (ID und Schlüssel), `Helm` (Version 3) und `kubectl` auf Ihrem Computer.</span><span class="sxs-lookup"><span data-stu-id="3c52f-203">For the next step, you need an Azure LogAnalytics Workspace (ID and Key), `Helm` (version 3), and `kubectl` on your machine.</span></span>

<span data-ttu-id="3c52f-204">Helm ist ein Kubernetes-Paket-Manager, der als Binärdatei verfügbar ist und unter macOS, Windows und Linux ausgeführt wird.</span><span class="sxs-lookup"><span data-stu-id="3c52f-204">Helm is a Kubernetes package manager, available as a binary that is runs on macOS, Windows, and Linux.</span></span> <span data-ttu-id="3c52f-205">Er kann hier heruntergeladen werden: [helm.sh](https://helm.sh/docs/intro/quickstart/). Helm nutzt die Kubernetes-Konfigurationsdatei, die für den `kubectl`-Befehl verwendet wird.</span><span class="sxs-lookup"><span data-stu-id="3c52f-205">It can be downloaded here: [helm.sh](https://helm.sh/docs/intro/quickstart/) Helm relies on the Kubernetes configuration file used for the `kubectl` command.</span></span>

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

<span data-ttu-id="3c52f-206">Mit diesem Befehl wird der Azure Monitor-Agent in Ihrem Kubernetes-Cluster installiert:</span><span class="sxs-lookup"><span data-stu-id="3c52f-206">This command will install the Azure Monitor agent on your Kubernetes cluster:</span></span>

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

<span data-ttu-id="3c52f-207">Der OMS-Agent (Operations Management Suite) in Ihrem Kubernetes-Cluster sendet Überwachungsdaten an Ihren Azure Log Analytics-Arbeitsbereich (über eine ausgehende HTTPS-Verbindung).</span><span class="sxs-lookup"><span data-stu-id="3c52f-207">The Operations Management Suite (OMS) Agent on your Kubernetes cluster will send monitoring data to your Azure Log Analytics Workspace (using outbound HTTPS).</span></span> <span data-ttu-id="3c52f-208">Sie können nun Azure Monitor nutzen, um umfangreichere Informationen zu Ihren Kubernetes-Clustern in Azure Stack Hub zu erhalten.</span><span class="sxs-lookup"><span data-stu-id="3c52f-208">You can now use Azure Monitor to get deeper insights about your Kubernetes clusters on Azure Stack Hub.</span></span> <span data-ttu-id="3c52f-209">Dieser Entwurf ist eine leistungsstarke Möglichkeit, die Leistungsfähigkeit von Analysen zu veranschaulichen, die automatisch mit den Clustern Ihrer Anwendung bereitgestellt werden können.</span><span class="sxs-lookup"><span data-stu-id="3c52f-209">This design is a powerful way to demonstrate the power of analytics that can be automatically deployed with your application's clusters.</span></span>

<span data-ttu-id="3c52f-210">[![Azure Stack Hub-Cluster in Azure Monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="3c52f-210">[![Azure Stack Hub clusters in Azure monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span></span>

<span data-ttu-id="3c52f-211">[![Details zu Azure Monitor-Clustern](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="3c52f-211">[![Azure Monitor cluster details](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="3c52f-212">Wenn in Azure Monitor keine Azure Stack Hub-Daten angezeigt werden, vergewissern Sie sich, dass Sie die Anweisungen zum [Hinzufügen der Lösung „AzureMonitor-Containers“ zu einem Azure Log Analytics-Arbeitsbereich](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) befolgt haben.</span><span class="sxs-lookup"><span data-stu-id="3c52f-212">If Azure Monitor does not show any Azure Stack Hub data, please make sure that you have followed the instructions on [how to add AzureMonitor-Containers solution to a Azure Loganalytics workspace](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) carefully.</span></span>

## <a name="deploy-the-application"></a><span data-ttu-id="3c52f-213">Bereitstellen der Anwendung</span><span class="sxs-lookup"><span data-stu-id="3c52f-213">Deploy the application</span></span>

<span data-ttu-id="3c52f-214">Vor der Installation der Beispielanwendung ist ein weiterer Schritt zum Konfigurieren des NGINX-basierten Eingangsdatencontrollers im Kubernetes-Cluster erforderlich.</span><span class="sxs-lookup"><span data-stu-id="3c52f-214">Before installing our sample application, there's another step to configure the nginx-based Ingress controller on our Kubernetes cluster.</span></span> <span data-ttu-id="3c52f-215">Der Eingangsdatencontroller wird als Layer-7-Lastenausgleich verwendet, um den Datenverkehr im Cluster auf der Grundlage von Host, Pfad oder Protokoll weiterzuleiten.</span><span class="sxs-lookup"><span data-stu-id="3c52f-215">The Ingress controller is used as a layer 7 load balancer to route traffic in our cluster based on host, path, or protocol.</span></span> <span data-ttu-id="3c52f-216">„nginx-ingress“ ist als Helm-Chart verfügbar.</span><span class="sxs-lookup"><span data-stu-id="3c52f-216">Nginx-ingress is available as a Helm Chart.</span></span> <span data-ttu-id="3c52f-217">Ausführliche Anweisungen finden Sie im [GitHub-Repository mit dem Helm-Chart](https://github.com/helm/charts/tree/master/stable/nginx-ingress).</span><span class="sxs-lookup"><span data-stu-id="3c52f-217">For detailed instructions, refer to the [Helm Chart GitHub repository](https://github.com/helm/charts/tree/master/stable/nginx-ingress).</span></span>

<span data-ttu-id="3c52f-218">Die Beispielanwendung wird ebenfalls als Helm-Chart gepackt, etwa der [Azure-Überwachungs-Agent](#configure-monitoring) im vorherigen Schritt.</span><span class="sxs-lookup"><span data-stu-id="3c52f-218">Our sample application is also packaged as a Helm Chart, like the [Azure Monitoring Agent](#configure-monitoring) in the previous step.</span></span> <span data-ttu-id="3c52f-219">Daher ist es einfach, die Anwendung im Kubernetes-Cluster bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="3c52f-219">As such, it's straightforward to deploy the application onto our Kubernetes cluster.</span></span> <span data-ttu-id="3c52f-220">Sie finden die [Helm-Chart-Dateien im zugehörigen GitHub-Repository](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm).</span><span class="sxs-lookup"><span data-stu-id="3c52f-220">You can find the [Helm Chart files in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)</span></span>

<span data-ttu-id="3c52f-221">Bei der Beispielanwendung handelt es sich um eine Anwendung mit drei Ebenen, die in einem Kubernetes-Cluster in zwei Azure Stack Hub-Instanzen bereitgestellt wird.</span><span class="sxs-lookup"><span data-stu-id="3c52f-221">The sample application is a three tier application, deployed onto a Kubernetes cluster on each of two Azure Stack Hub instances.</span></span> <span data-ttu-id="3c52f-222">Die Anwendung nutzt eine MongoDB-Datenbank.</span><span class="sxs-lookup"><span data-stu-id="3c52f-222">The application uses a MongoDB database.</span></span> <span data-ttu-id="3c52f-223">Weitere Informationen zum Replizieren der Daten über mehrere Instanzen hinweg finden Sie im Muster [Überlegungen zu Daten und Speicher](pattern-highly-available-kubernetes.md#data-and-storage-considerations).</span><span class="sxs-lookup"><span data-stu-id="3c52f-223">You can learn more about how to get the data replicated across multiple instances in the pattern [Data and Storage considerations](pattern-highly-available-kubernetes.md#data-and-storage-considerations).</span></span>

<span data-ttu-id="3c52f-224">Nach der Bereitstellung des Helm-Charts für die Anwendung werden alle drei Ebenen der Anwendung als Bereitstellungen und StatefulSets (für die Datenbank) mit einem einzelnen Pod dargestellt:</span><span class="sxs-lookup"><span data-stu-id="3c52f-224">After deploying the Helm Chart for the application, you'll see all three tiers of your application represented as deployments and stateful sets (for the database) with a single pod:</span></span>

```kubectl
kubectl get pod,deployment,statefulset
```

```console
NAME                                         READY   STATUS
pod/ratings-api-569d7f7b54-mrv5d             1/1     Running
pod/ratings-mongodb-0                        1/1     Running
pod/ratings-web-85667bfb86-l6vxz             1/1     Running

NAME                                         READY
deployment.extensions/ratings-api            1/1
deployment.extensions/ratings-web            1/1

NAME                                         READY
statefulset.apps/ratings-mongodb             1/1
```

<span data-ttu-id="3c52f-225">Auf der Dienstseite finden Sie den NGINX-basierten Eingangsdatencontroller und seine öffentliche IP-Adresse:</span><span class="sxs-lookup"><span data-stu-id="3c52f-225">On the services, side you'll find the nginx-based Ingress Controller and its public IP address:</span></span>

```kubectl
kubectl get service
```

```console
NAME                                         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)
kubernetes                                   ClusterIP      10.0.0.1       <none>        443/TCP
nginx-ingress-1588931383-controller          LoadBalancer   10.0.114.180   *public-ip*   443:30667/TCP
nginx-ingress-1588931383-default-backend     ClusterIP      10.0.76.54     <none>        80/TCP
ratings-api                                  ClusterIP      10.0.46.69     <none>        80/TCP
ratings-web                                  ClusterIP      10.0.161.124   <none>        80/TCP
```

<span data-ttu-id="3c52f-226">Die externe IP-Adresse ist der Anwendungsendpunkt.</span><span class="sxs-lookup"><span data-stu-id="3c52f-226">The "External IP" address is our "application endpoint".</span></span> <span data-ttu-id="3c52f-227">Damit wird festgelegt, wie Benutzer eine Verbindung zum Öffnen der Anwendung herstellen, und er wird außerdem als Endpunkt für den nächsten Schritt ([Konfigurieren von Traffic Manager](#configure-traffic-manager)) verwendet.</span><span class="sxs-lookup"><span data-stu-id="3c52f-227">It's how users will connect to open the application and will also be used as the endpoint for our next step [Configure Traffic Manager](#configure-traffic-manager).</span></span>

## <a name="autoscale-the-application"></a><span data-ttu-id="3c52f-228">Autoskalieren der Anwendung</span><span class="sxs-lookup"><span data-stu-id="3c52f-228">Autoscale the application</span></span>
<span data-ttu-id="3c52f-229">Optional können Sie die [horizontale automatische Podskalierung](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) (Horizontal Pod AutoScaler, HPA) konfigurieren, um basierend auf bestimmten Metriken wie CPU-Auslastung zentral hoch- oder herunterzuskalieren.</span><span class="sxs-lookup"><span data-stu-id="3c52f-229">You can optionally configure the [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) to scale up or down based on certain metrics like CPU utilization.</span></span> <span data-ttu-id="3c52f-230">Mit dem folgenden Befehl wird eine horizontale automatische Podskalierung erstellt, die ein bis zehn Replikate der Pods verwaltet, die von der Bereitstellung „ratings-web“ gesteuert werden.</span><span class="sxs-lookup"><span data-stu-id="3c52f-230">The following command will create a Horizontal Pod Autoscaler that maintains 1 to 10 replicas of the Pods controlled by the ratings-web deployment.</span></span> <span data-ttu-id="3c52f-231">Die HPA erhöht und verringert die Anzahl von Replikaten (über die Bereitstellung), um eine durchschnittliche CPU-Auslastung von 80 Prozent über alle Pods hinweg aufrechtzuerhalten.</span><span class="sxs-lookup"><span data-stu-id="3c52f-231">HPA will increase and decrease the number of replicas (via the deployment) to maintain an average CPU utilization across all Pods of 80%.</span></span>

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
<span data-ttu-id="3c52f-232">Sie können den aktuellen Status der Autoskalierung überprüfen, indem Sie Folgendes ausführen:</span><span class="sxs-lookup"><span data-stu-id="3c52f-232">You may check the current status of autoscaler by running:</span></span>

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a><span data-ttu-id="3c52f-233">Traffic Manager konfigurieren</span><span class="sxs-lookup"><span data-stu-id="3c52f-233">Configure Traffic Manager</span></span>

<span data-ttu-id="3c52f-234">Zum Verteilen des Datenverkehrs zwischen zwei (oder mehr) Bereitstellungen der Anwendung verwenden Sie [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview).</span><span class="sxs-lookup"><span data-stu-id="3c52f-234">To distribute traffic between two (or more) deployments of the application, we'll use [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview).</span></span> <span data-ttu-id="3c52f-235">Azure Traffic Manager ist ein auf DNS basierender Lastenausgleichsdienst in Azure.</span><span class="sxs-lookup"><span data-stu-id="3c52f-235">Azure Traffic Manager is a DNS-based traffic load balancer in Azure.</span></span>

> [!NOTE]
> <span data-ttu-id="3c52f-236">Traffic Manager verwendet DNS, um Clientanforderungen auf der Grundlage einer Datenverkehrsrouting-Methode und der Integrität der Endpunkte an den optimalen Endpunkt weiterzuleiten.</span><span class="sxs-lookup"><span data-stu-id="3c52f-236">Traffic Manager uses DNS to direct client requests to the most appropriate service endpoint, based on a traffic-routing method and the health of the endpoints.</span></span>

<span data-ttu-id="3c52f-237">Anstelle von Azure Traffic Manager können Sie auch andere globale Lastenausgleichslösungen verwenden, die lokal gehostet werden.</span><span class="sxs-lookup"><span data-stu-id="3c52f-237">Instead of using Azure Traffic Manager you can also use other global load-balancing solutions hosted on-premises.</span></span> <span data-ttu-id="3c52f-238">Im Beispielszenario verwenden Sie Azure Traffic Manager, um den Datenverkehr zwischen zwei Instanzen Ihrer Anwendung zu verteilen.</span><span class="sxs-lookup"><span data-stu-id="3c52f-238">In the sample scenario, we'll use Azure Traffic Manager to distribute traffic between two instances of our application.</span></span> <span data-ttu-id="3c52f-239">Diese können in Azure Stack Hub-Instanzen am gleichen Standort oder an verschiedenen Standorten ausgeführt werden:</span><span class="sxs-lookup"><span data-stu-id="3c52f-239">They can run on Azure Stack Hub instances in the same or different locations:</span></span>

![Lokale Traffic Manager-Instanz](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

<span data-ttu-id="3c52f-241">Sie konfigurieren Traffic Manager in Azure, um auf die beiden unterschiedlichen Instanzen der Anwendung zu verweisen:</span><span class="sxs-lookup"><span data-stu-id="3c52f-241">In Azure, we configure Traffic Manager to point to the two different instances of our application:</span></span>

<span data-ttu-id="3c52f-242">[![TM-Endpunktprofil](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="3c52f-242">[![TM endpoint profile](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span></span>

<span data-ttu-id="3c52f-243">Wie Sie sehen, verweisen die beiden Endpunkte auf die beiden Instanzen der bereitgestellten Anwendung aus dem [vorherigen Abschnitt](#deploy-the-application).</span><span class="sxs-lookup"><span data-stu-id="3c52f-243">As you can see, the two endpoints point to the two instances of the deployed application from the [previous section](#deploy-the-application).</span></span>

<span data-ttu-id="3c52f-244">Zu diesem Zeitpunkt gilt Folgendes:</span><span class="sxs-lookup"><span data-stu-id="3c52f-244">At this point:</span></span>
- <span data-ttu-id="3c52f-245">Die Kubernetes-Infrastruktur, einschließlich eines Eingangsdatencontrollers, wurde erstellt.</span><span class="sxs-lookup"><span data-stu-id="3c52f-245">The Kubernetes infrastructure has been created, including an Ingress Controller.</span></span>
- <span data-ttu-id="3c52f-246">Cluster wurden in zwei Azure Stack Hub-Instanzen bereitgestellt.</span><span class="sxs-lookup"><span data-stu-id="3c52f-246">Clusters have been deployed across two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="3c52f-247">Die Überwachung wurde konfiguriert.</span><span class="sxs-lookup"><span data-stu-id="3c52f-247">Monitoring has been configured.</span></span>
- <span data-ttu-id="3c52f-248">Azure Traffic Manager nimmt einen Lastausgleich für den Datenverkehr zwischen den beiden Azure Stack Hub-Instanzen vor.</span><span class="sxs-lookup"><span data-stu-id="3c52f-248">Azure Traffic Manager will load balance traffic across the two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="3c52f-249">Zusätzlich zu dieser Infrastruktur wurde die Beispielanwendung mit drei Ebenen mithilfe von Helm-Charts automatisiert bereitgestellt.</span><span class="sxs-lookup"><span data-stu-id="3c52f-249">On top of this infrastructure, the sample three-tier application has been deployed in an automated way using Helm Charts.</span></span> 

<span data-ttu-id="3c52f-250">Die Lösung sollte nun einsatzbereit und für Benutzer verfügbar sein.</span><span class="sxs-lookup"><span data-stu-id="3c52f-250">The solution should now be up and accessible to users!</span></span>

<span data-ttu-id="3c52f-251">Außerdem gibt es einige wichtige Überlegungen zum Betrieb nach der Bereitstellung, die in den nächsten zwei Abschnitten behandelt werden.</span><span class="sxs-lookup"><span data-stu-id="3c52f-251">There are also some post-deployment operational considerations worth discussing, which are covered in the next two sections.</span></span>

## <a name="upgrade-kubernetes"></a><span data-ttu-id="3c52f-252">Kubernetes aktualisieren</span><span class="sxs-lookup"><span data-stu-id="3c52f-252">Upgrade Kubernetes</span></span>

<span data-ttu-id="3c52f-253">Beachten Sie bei der Aktualisierung des Kubernetes-Clusters die folgenden Punkte:</span><span class="sxs-lookup"><span data-stu-id="3c52f-253">Consider the following topics when upgrading the Kubernetes cluster:</span></span>

- <span data-ttu-id="3c52f-254">Das Upgrade eines Kubernetes-Clusters ist ein komplexer Vorgang am 2. Tag, der mithilfe der AKS-Engine ausgeführt werden kann.</span><span class="sxs-lookup"><span data-stu-id="3c52f-254">Upgrading a Kubernetes cluster is a complex Day 2 operation that can be done using AKS Engine.</span></span> <span data-ttu-id="3c52f-255">Weitere Informationen finden Sie unter [Aktualisieren eines Kubernetes-Clusters in Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).</span><span class="sxs-lookup"><span data-stu-id="3c52f-255">For more information, see [Upgrade a Kubernetes cluster on Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).</span></span>
- <span data-ttu-id="3c52f-256">Mithilfe der AKS-Engine können Sie Cluster auf neuere Versionen von Kubernetes und des Betriebssystem-Basisimages aktualisieren.</span><span class="sxs-lookup"><span data-stu-id="3c52f-256">AKS Engine allows you to upgrade clusters to newer Kubernetes and base OS image versions.</span></span> <span data-ttu-id="3c52f-257">Weitere Informationen finden Sie unter [Schritte zum Durchführen eines Upgrades auf eine neuere Kubernetes-Version](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version).</span><span class="sxs-lookup"><span data-stu-id="3c52f-257">For more information, see [Steps to upgrade to a newer Kubernetes version](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version).</span></span> 
- <span data-ttu-id="3c52f-258">Sie können auch nur die zugrunde liegenden Knoten auf neuere Versionen des Betriebssystem-Basisimages aktualisieren.</span><span class="sxs-lookup"><span data-stu-id="3c52f-258">You can also upgrade only the underlaying nodes to newer base OS image versions.</span></span> <span data-ttu-id="3c52f-259">Weitere Informationen finden Sie unter [Schritte zum alleinigen Upgrade des Betriebssystemimages](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).</span><span class="sxs-lookup"><span data-stu-id="3c52f-259">For more information, see [Steps to only upgrade the OS image](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).</span></span>

<span data-ttu-id="3c52f-260">Neuere Betriebssystem-Basisimages enthalten Sicherheits- und Kernelupdates.</span><span class="sxs-lookup"><span data-stu-id="3c52f-260">Newer base OS images contain security and kernel updates.</span></span> <span data-ttu-id="3c52f-261">Es liegt in der Verantwortung des Clusteroperators, die Verfügbarkeit von neueren Kubernetes- und Betriebssystemversionen zu überwachen.</span><span class="sxs-lookup"><span data-stu-id="3c52f-261">It's the cluster operator's responsibility to monitor the availability of newer Kubernetes Versions and OS Images.</span></span> <span data-ttu-id="3c52f-262">Der Operator sollte diese Upgrades mithilfe der AKS-Engine planen und ausführen.</span><span class="sxs-lookup"><span data-stu-id="3c52f-262">The operator should plan and execute these upgrades using AKS Engine.</span></span> <span data-ttu-id="3c52f-263">Die Betriebssystem-Basisimages müssen vom Azure Stack Hub-Operator aus dem Azure Stack Hub-Marketplace heruntergeladen werden.</span><span class="sxs-lookup"><span data-stu-id="3c52f-263">The base OS images must be downloaded from the Azure Stack Hub Marketplace by the Azure Stack Hub Operator.</span></span>

## <a name="scale-kubernetes"></a><span data-ttu-id="3c52f-264">Skalieren von Kubernetes</span><span class="sxs-lookup"><span data-stu-id="3c52f-264">Scale Kubernetes</span></span>

<span data-ttu-id="3c52f-265">Die Skalierung ist ein weiterer Vorgang am 2. Tag, der mithilfe der AKS-Engine orchestriert werden kann.</span><span class="sxs-lookup"><span data-stu-id="3c52f-265">Scale is another Day 2 operation that can be orchestrated using AKS Engine.</span></span>

<span data-ttu-id="3c52f-266">Vom Skalierungsbefehl wird Ihre Clusterkonfigurationsdatei (apimodel.json) im Ausgabeverzeichnis als Eingabe für eine neue Azure Resource Manager-Bereitstellung wiederverwendet.</span><span class="sxs-lookup"><span data-stu-id="3c52f-266">The scale command reuses your cluster configuration file (apimodel.json) in the output directory, as input for a new Azure Resource Manager deployment.</span></span> <span data-ttu-id="3c52f-267">Die AKS-Engine führt den Skalierungsvorgang für einen bestimmten Agentpool aus.</span><span class="sxs-lookup"><span data-stu-id="3c52f-267">AKS Engine executes the scale operation against a specific agent pool.</span></span> <span data-ttu-id="3c52f-268">Wenn der Skalierungsvorgang beendet ist, aktualisiert die AKS-Engine die Clusterdefinition in derselben Datei vom Typ „apimodel.json“.</span><span class="sxs-lookup"><span data-stu-id="3c52f-268">When the scale operation is complete, AKS Engine updates the cluster definition in that same apimodel.json file.</span></span> <span data-ttu-id="3c52f-269">Die Clusterdefinition spiegelt die neue Knotenanzahl und damit die aktualisierte (derzeitige) Clusterkonfiguration wider.</span><span class="sxs-lookup"><span data-stu-id="3c52f-269">The cluster definition reflects the new node count in order to reflect the updated, current cluster configuration.</span></span>

- [<span data-ttu-id="3c52f-270">Skalieren eines Kubernetes-Cluster in Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="3c52f-270">Scale a Kubernetes cluster on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a><span data-ttu-id="3c52f-271">Nächste Schritte</span><span class="sxs-lookup"><span data-stu-id="3c52f-271">Next steps</span></span>

- <span data-ttu-id="3c52f-272">Erfahren Sie mehr über [Überlegungen zum Entwurf von Hybrid-Apps](overview-app-design-considerations.md).</span><span class="sxs-lookup"><span data-stu-id="3c52f-272">Learn more about [Hybrid app design considerations](overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="3c52f-273">Überprüfen Sie den [Code dieses Beispiels auf GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), und schlagen Sie Verbesserungen vor.</span><span class="sxs-lookup"><span data-stu-id="3c52f-273">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).</span></span>