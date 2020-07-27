---
title: Bereitstellen einer Hybrid-App mit lokalen Daten mit cloudübergreifender Skalierung
description: Es wird beschrieben, wie Sie eine App bereitstellen, die lokale Daten verwendet und mithilfe von Azure und Azure Stack Hub cloudübergreifend skaliert wird.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 6de35cb55c4c35a2a9927f9ffc2516ccb00cd89f
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477319"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a><span data-ttu-id="16b6a-103">Bereitstellen einer Hybrid-App mit lokalen Daten mit cloudübergreifender Skalierung</span><span class="sxs-lookup"><span data-stu-id="16b6a-103">Deploy hybrid app with on-premises data that scales cross-cloud</span></span>

<span data-ttu-id="16b6a-104">In diesem Lösungsleitfaden erfahren Sie, wie Sie eine Hybrid-App implementieren, die sowohl für Azure als auch für Azure Stack Hub gilt und für die nur eine gemeinsame lokale Datenquelle verwendet wird.</span><span class="sxs-lookup"><span data-stu-id="16b6a-104">This solution guide shows you how to deploy a hybrid app that spans both Azure and Azure Stack Hub and uses a single on-premises data source.</span></span>

<span data-ttu-id="16b6a-105">Indem Sie eine Hybrid Cloud-Lösung verwenden, können Sie die Compliancevorteile einer privaten Cloud mit der Skalierbarkeit der öffentlichen Cloud verbinden.</span><span class="sxs-lookup"><span data-stu-id="16b6a-105">By using a hybrid cloud solution, you can combine the compliance benefits of a private cloud with the scalability of the public cloud.</span></span> <span data-ttu-id="16b6a-106">Ihre Entwickler können auch das Microsoft-Entwicklerökosystem nutzen und ihre Fähigkeiten auf die Cloudumgebungen und lokalen Umgebungen anwenden.</span><span class="sxs-lookup"><span data-stu-id="16b6a-106">Your developers can also take advantage of the Microsoft developer ecosystem and apply their skills to the cloud and on-premises environments.</span></span>

## <a name="overview-and-assumptions"></a><span data-ttu-id="16b6a-107">Übersicht und Annahmen</span><span class="sxs-lookup"><span data-stu-id="16b6a-107">Overview and assumptions</span></span>

<span data-ttu-id="16b6a-108">Arbeiten Sie dieses Tutorial durch, um einen Workflow einzurichten, bei dem Entwickler eine identische Web-App in einer öffentlichen und einer privaten Cloud bereitstellen können.</span><span class="sxs-lookup"><span data-stu-id="16b6a-108">Follow this tutorial to set up a workflow that lets developers deploy an identical web app to a public cloud and a private cloud.</span></span> <span data-ttu-id="16b6a-109">Diese App kann auf ein Netzwerk zugreifen, das nicht über das Internet geroutet werden kann und in der privaten Cloud gehostet wird.</span><span class="sxs-lookup"><span data-stu-id="16b6a-109">This app can access a non-internet routable network hosted on the private cloud.</span></span> <span data-ttu-id="16b6a-110">Diese Web-Apps werden überwacht. Wenn es zu einer Datenverkehrsspitze kommt, werden die DNS-Einträge mit einem Programm geändert, um Datenverkehr an die öffentliche Cloud umzuleiten.</span><span class="sxs-lookup"><span data-stu-id="16b6a-110">These web apps are monitored and when there's a spike in traffic, a program modifies the DNS records to redirect traffic to the public cloud.</span></span> <span data-ttu-id="16b6a-111">Wenn der Datenverkehr wieder auf die Menge vor der Spitze fällt, wird er wieder an die private Cloud geleitet.</span><span class="sxs-lookup"><span data-stu-id="16b6a-111">When traffic drops to the level before the spike, traffic is routed back to the private cloud.</span></span>

<span data-ttu-id="16b6a-112">Dieses Tutorial enthält die folgenden Aufgaben:</span><span class="sxs-lookup"><span data-stu-id="16b6a-112">This tutorial covers the following tasks:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="16b6a-113">Bereitstellen eines SQL Server-Datenbankservers mit Hybridverbindung</span><span class="sxs-lookup"><span data-stu-id="16b6a-113">Deploy a hybrid-connected SQL Server database server.</span></span>
> - <span data-ttu-id="16b6a-114">Verbinden einer Web-App in der globalen Azure-Umgebung mit einem Hybridnetzwerk</span><span class="sxs-lookup"><span data-stu-id="16b6a-114">Connect a web app in global Azure to a hybrid network.</span></span>
> - <span data-ttu-id="16b6a-115">Konfigurieren von DNS für die cloudübergreifende Skalierung</span><span class="sxs-lookup"><span data-stu-id="16b6a-115">Configure DNS for cross-cloud scaling.</span></span>
> - <span data-ttu-id="16b6a-116">Konfigurieren von SSL-Zertifikaten für die cloudübergreifende Skalierung</span><span class="sxs-lookup"><span data-stu-id="16b6a-116">Configure SSL certificates for cross-cloud scaling.</span></span>
> - <span data-ttu-id="16b6a-117">Konfigurieren und Bereitstellen der Web-App</span><span class="sxs-lookup"><span data-stu-id="16b6a-117">Configure and deploy the web app.</span></span>
> - <span data-ttu-id="16b6a-118">Erstellen eines Traffic Manager-Profils mit anschließender Konfiguration für die cloudübergreifende Skalierung</span><span class="sxs-lookup"><span data-stu-id="16b6a-118">Create a Traffic Manager profile and configure it for cross-cloud scaling.</span></span>
> - <span data-ttu-id="16b6a-119">Einrichten der Application Insights-Überwachung und -Benachrichtigung für erhöhten Datenverkehr</span><span class="sxs-lookup"><span data-stu-id="16b6a-119">Set up Application Insights monitoring and alerting for increased traffic.</span></span>
> - <span data-ttu-id="16b6a-120">Konfigurieren der automatischen Umschaltung des Datenverkehrs zwischen der globalen Azure-Umgebung und Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="16b6a-120">Configure automatic traffic switching between global Azure and Azure Stack Hub.</span></span>

> [!Tip]  
> <span data-ttu-id="16b6a-121">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="16b6a-121">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="16b6a-122">Microsoft Azure Stack Hub ist eine Erweiterung von Azure.</span><span class="sxs-lookup"><span data-stu-id="16b6a-122">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="16b6a-123">Mit Azure Stack Hub holen Sie sich die Agilität und Innovation von Cloud Computing in Ihre lokale Umgebung. Sie erhalten die einzige Hybrid Cloud, mit der Sie Hybrid-Apps überall entwickeln und bereitstellen können.</span><span class="sxs-lookup"><span data-stu-id="16b6a-123">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="16b6a-124">Im Artikel [Überlegungen zum Entwurf von Hybrid-Apps](overview-app-design-considerations.md) werden die wichtigen Aspekte in Bezug auf die Softwarequalität (Platzierung, Skalierbarkeit, Verfügbarkeit, Resilienz, Verwaltbarkeit und Sicherheit) beschrieben, die für das Entwerfen, Bereitstellen und Betreiben von Hybrid-Apps erforderlich sind.</span><span class="sxs-lookup"><span data-stu-id="16b6a-124">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="16b6a-125">Die Überlegungen zum Entwurf dienen als Hilfe beim Optimieren des Designs von Hybrid-Apps, um für Produktionsumgebungen das Auftreten von Problemen zu minimieren.</span><span class="sxs-lookup"><span data-stu-id="16b6a-125">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

### <a name="assumptions"></a><span data-ttu-id="16b6a-126">Annahmen</span><span class="sxs-lookup"><span data-stu-id="16b6a-126">Assumptions</span></span>

<span data-ttu-id="16b6a-127">In diesem Tutorial wird davon ausgegangen, dass Sie bereits über Grundkenntnisse in Bezug auf die globale Azure-Umgebung und Azure Stack Hub verfügen.</span><span class="sxs-lookup"><span data-stu-id="16b6a-127">This tutorial assumes that you have a basic knowledge of global Azure and Azure Stack Hub.</span></span> <span data-ttu-id="16b6a-128">Lesen Sie diese Artikel, falls Sie sich vor Beginn des Tutorials informieren möchten:</span><span class="sxs-lookup"><span data-stu-id="16b6a-128">If you want to learn more before starting the tutorial, review these articles:</span></span>

- [<span data-ttu-id="16b6a-129">Einführung in Azure</span><span class="sxs-lookup"><span data-stu-id="16b6a-129">Introduction to Azure</span></span>](https://azure.microsoft.com/overview/what-is-azure/)
- [<span data-ttu-id="16b6a-130">Übersicht über Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="16b6a-130">Azure Stack Hub Key Concepts</span></span>](/azure-stack/operator/azure-stack-overview.md)

<span data-ttu-id="16b6a-131">In diesem Tutorial wird auch davon ausgegangen, dass Sie über ein Azure-Abonnement verfügen.</span><span class="sxs-lookup"><span data-stu-id="16b6a-131">This tutorial also assumes that you have an Azure subscription.</span></span> <span data-ttu-id="16b6a-132">Wenn Sie kein Abonnement besitzen, müssen Sie ein [kostenloses Konto erstellen](https://azure.microsoft.com/free/), bevor Sie beginnen.</span><span class="sxs-lookup"><span data-stu-id="16b6a-132">If you don't have a subscription, [create a free account](https://azure.microsoft.com/free/) before you begin.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="16b6a-133">Voraussetzungen</span><span class="sxs-lookup"><span data-stu-id="16b6a-133">Prerequisites</span></span>

<span data-ttu-id="16b6a-134">Vergewissern Sie sich zunächst, dass die folgenden Anforderungen erfüllt sind bzw. dass Folgendes vorhanden ist:</span><span class="sxs-lookup"><span data-stu-id="16b6a-134">Before you start this solution, make sure you meet the following requirements:</span></span>

- <span data-ttu-id="16b6a-135">Ein Azure Stack Development Kit (ASDK) oder ein Abonnement für ein integriertes Azure Stack Hub-System.</span><span class="sxs-lookup"><span data-stu-id="16b6a-135">An Azure Stack Development Kit (ASDK) or a subscription on an Azure Stack Hub Integrated System.</span></span> <span data-ttu-id="16b6a-136">Befolgen Sie die Anleitung zum Bereitstellen des ASDK unter [Bereitstellen des ASDK mithilfe des Installationsprogramms](/azure-stack/asdk/asdk-install.md).</span><span class="sxs-lookup"><span data-stu-id="16b6a-136">To deploy the ASDK, follow the instructions in [Deploy the ASDK using the installer](/azure-stack/asdk/asdk-install.md).</span></span>
- <span data-ttu-id="16b6a-137">Für Ihre Azure Stack Hub-Installation muss Folgendes installiert sein:</span><span class="sxs-lookup"><span data-stu-id="16b6a-137">Your Azure Stack Hub installation should have the following installed:</span></span>
  - <span data-ttu-id="16b6a-138">Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="16b6a-138">The Azure App Service.</span></span> <span data-ttu-id="16b6a-139">Arbeiten Sie mit Ihrem Azure Stack Hub-Betreiber zusammen, um Azure App Service in Ihrer Umgebung bereitzustellen und zu konfigurieren.</span><span class="sxs-lookup"><span data-stu-id="16b6a-139">Work with your Azure Stack Hub Operator to deploy and configure the Azure App Service on your environment.</span></span> <span data-ttu-id="16b6a-140">Für dieses Tutorial muss App Service mindestens über eine (1) verfügbare dedizierte Workerrolle verfügen.</span><span class="sxs-lookup"><span data-stu-id="16b6a-140">This tutorial requires the App Service to have at least one (1) available dedicated worker role.</span></span>
  - <span data-ttu-id="16b6a-141">Ein Windows Server 2016-Image.</span><span class="sxs-lookup"><span data-stu-id="16b6a-141">A Windows Server 2016 image.</span></span>
  - <span data-ttu-id="16b6a-142">Eine Windows Server 2016-Instanz mit einem Microsoft SQL Server-Image.</span><span class="sxs-lookup"><span data-stu-id="16b6a-142">A Windows Server 2016 with a Microsoft SQL Server image.</span></span>
  - <span data-ttu-id="16b6a-143">Die entsprechenden Tarife und Angebote.</span><span class="sxs-lookup"><span data-stu-id="16b6a-143">The appropriate plans and offers.</span></span>
  - <span data-ttu-id="16b6a-144">Ein Domänenname für Ihre Web-App.</span><span class="sxs-lookup"><span data-stu-id="16b6a-144">A domain name for your web app.</span></span> <span data-ttu-id="16b6a-145">Wenn Sie keinen Domänennamen besitzen, können Sie einen von einem Domänenanbieter wie GoDaddy, Bluehost oder InMotion erwerben.</span><span class="sxs-lookup"><span data-stu-id="16b6a-145">If you don't have a domain name, you can buy one from a domain provider like GoDaddy, Bluehost, and InMotion.</span></span>
- <span data-ttu-id="16b6a-146">Ein SSL-Zertifikat für Ihre Domäne von einer vertrauenswürdigen Zertifizierungsstelle, z. B. LetsEncrypt.</span><span class="sxs-lookup"><span data-stu-id="16b6a-146">An SSL certificate for your domain from a trusted certificate authority like LetsEncrypt.</span></span>
- <span data-ttu-id="16b6a-147">Eine Web-App, die mit einer SQL Server-Datenbank kommuniziert und Application Insights unterstützt.</span><span class="sxs-lookup"><span data-stu-id="16b6a-147">A web app that communicates with a SQL Server database and supports Application Insights.</span></span> <span data-ttu-id="16b6a-148">Sie können die Beispiel-App [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) von GitHub herunterladen.</span><span class="sxs-lookup"><span data-stu-id="16b6a-148">You can download the [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) sample app from GitHub.</span></span>
- <span data-ttu-id="16b6a-149">Ein Hybridnetzwerk zwischen einem virtuellen Azure-Netzwerk und einem virtuellen Azure Stack Hub-Netzwerk.</span><span class="sxs-lookup"><span data-stu-id="16b6a-149">A hybrid network between an Azure virtual network and Azure Stack Hub virtual network.</span></span> <span data-ttu-id="16b6a-150">Eine ausführliche Anleitung finden Sie unter [Konfigurieren der Hybrid Cloud-Konnektivität mit Azure und Azure Stack Hub](solution-deployment-guide-connectivity.md).</span><span class="sxs-lookup"><span data-stu-id="16b6a-150">For detailed instructions, see [Configure hybrid cloud connectivity with Azure and Azure Stack Hub](solution-deployment-guide-connectivity.md).</span></span>

- <span data-ttu-id="16b6a-151">Eine CI/CD-Hybridpipeline (Continuous Integration/Continuous Deployment) mit einem privaten Build-Agent in Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="16b6a-151">A hybrid continuous integration/continuous deployment (CI/CD) pipeline with a private build agent on Azure Stack Hub.</span></span> <span data-ttu-id="16b6a-152">Eine ausführliche Anleitung finden Sie unter [Konfigurieren einer Hybrid Cloud-Identität mit Azure- und Azure Stack Hub-Apps](solution-deployment-guide-identity.md).</span><span class="sxs-lookup"><span data-stu-id="16b6a-152">For detailed instructions, see [Configure hybrid cloud identity with Azure and Azure Stack Hub apps](solution-deployment-guide-identity.md).</span></span>

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a><span data-ttu-id="16b6a-153">Bereitstellen eines SQL Server-Datenbankservers mit Hybridverbindung</span><span class="sxs-lookup"><span data-stu-id="16b6a-153">Deploy a hybrid-connected SQL Server database server</span></span>

1. <span data-ttu-id="16b6a-154">Melden Sie sich am Azure Stack Hub-Benutzerportal an.</span><span class="sxs-lookup"><span data-stu-id="16b6a-154">Sign to the Azure Stack Hub user portal.</span></span>

2. <span data-ttu-id="16b6a-155">Wählen Sie im **Dashboard** die Option **Marketplace**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-155">On the **Dashboard**, select **Marketplace**.</span></span>

    ![Azure Stack Hub Marketplace](media/solution-deployment-guide-hybrid/image1.png)

3. <span data-ttu-id="16b6a-157">Wählen Sie unter **Marketplace** die Option **Compute** und dann **Mehr**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-157">In **Marketplace**, select **Compute**, and then choose **More**.</span></span> <span data-ttu-id="16b6a-158">Wählen Sie unter **Mehr** das Image **Free SQL Server License: SQL Server 2017 Developer on Windows Server** (Kostenlose SQL Server-Lizenz: SQL Server 2017 Developer unter Windows Server) aus.</span><span class="sxs-lookup"><span data-stu-id="16b6a-158">Under **More**, select the **Free SQL Server License: SQL Server 2017 Developer on Windows Server** image.</span></span>

    ![Auswählen eines VM-Images im Azure Stack Hub-Benutzerportal](media/solution-deployment-guide-hybrid/image2.png)

4. <span data-ttu-id="16b6a-160">Wählen Sie unter **Free SQL Server License: SQL Server 2017 Developer on Windows Server** (Kostenlose SQL Server-Lizenz: SQL Server 2017 Developer unter Windows Server) die Option **Erstellen** aus.</span><span class="sxs-lookup"><span data-stu-id="16b6a-160">On **Free SQL Server License: SQL Server 2017 Developer on Windows Server**, select **Create**.</span></span>

5. <span data-ttu-id="16b6a-161">Geben Sie unter **Grundlagen > Grundeinstellungen konfigurieren** einen **Namen** für den virtuellen Computer (VM) und einen **Benutzernamen** und ein **Kennwort** für den SQL Server-Systemadministrator an.</span><span class="sxs-lookup"><span data-stu-id="16b6a-161">On **Basics > Configure basic settings**, provide a **Name** for the virtual machine (VM), a **User name** for the SQL Server SA, and a **Password** for the SA.</span></span>  <span data-ttu-id="16b6a-162">Wählen Sie in der Dropdownliste **Abonnement** das Abonnement aus, für das Sie die Bereitstellung durchführen möchten.</span><span class="sxs-lookup"><span data-stu-id="16b6a-162">From the **Subscription** drop-down list, select the subscription that you're deploying to.</span></span> <span data-ttu-id="16b6a-163">Wählen Sie unter **Ressourcengruppe** die Option **Vorhandene auswählen**, und ordnen Sie die VM in derselben Ressourcengruppe wie Ihre Azure Stack Hub-Web-App an.</span><span class="sxs-lookup"><span data-stu-id="16b6a-163">For **Resource group**, use **Choose existing** and put the VM in the same resource group as your Azure Stack Hub web app.</span></span>

    ![Konfigurieren der Grundeinstellungen für die VM im Azure Stack Hub-Benutzerportal](media/solution-deployment-guide-hybrid/image3.png)

6. <span data-ttu-id="16b6a-165">Wählen Sie unter **Größe** eine Größe für Ihre VM aus.</span><span class="sxs-lookup"><span data-stu-id="16b6a-165">Under **Size**, pick a size for your VM.</span></span> <span data-ttu-id="16b6a-166">Für dieses Tutorial empfehlen wir „A2_Standard“ oder „DS2_V2_Standard“.</span><span class="sxs-lookup"><span data-stu-id="16b6a-166">For this tutorial, we recommend A2_Standard or a DS2_V2_Standard.</span></span>

7. <span data-ttu-id="16b6a-167">Konfigurieren Sie unter **Einstellungen > Optionale Features konfigurieren** die folgenden Einstellungen:</span><span class="sxs-lookup"><span data-stu-id="16b6a-167">Under **Settings > Configure optional features**, configure the following settings:</span></span>

   - <span data-ttu-id="16b6a-168">**Speicherkonto**: Erstellen Sie ein neues Konto, falls erforderlich.</span><span class="sxs-lookup"><span data-stu-id="16b6a-168">**Storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="16b6a-169">**Virtuelles Netzwerk:**</span><span class="sxs-lookup"><span data-stu-id="16b6a-169">**Virtual network**:</span></span>

     > [!Important]  
     > <span data-ttu-id="16b6a-170">Stellen Sie sicher, dass Ihre SQL Server-VM in demselben virtuellen Netzwerk wie die VPN-Gateways bereitgestellt wird.</span><span class="sxs-lookup"><span data-stu-id="16b6a-170">Make sure your SQL Server VM is deployed on the same  virtual network as the VPN gateways.</span></span>

   - <span data-ttu-id="16b6a-171">**Öffentliche IP-Adresse:** Verwenden Sie die Standardeinstellungen.</span><span class="sxs-lookup"><span data-stu-id="16b6a-171">**Public IP address**: Use the default settings.</span></span>
   - <span data-ttu-id="16b6a-172">**Netzwerksicherheitsgruppe**: (NSG).</span><span class="sxs-lookup"><span data-stu-id="16b6a-172">**Network security group**: (NSG).</span></span> <span data-ttu-id="16b6a-173">Erstellen Sie eine neue NSG.</span><span class="sxs-lookup"><span data-stu-id="16b6a-173">Create a new NSG.</span></span>
   - <span data-ttu-id="16b6a-174">**Extensions and Monitoring** (Erweiterungen und Überwachung): Behalten Sie die Standardeinstellungen bei.</span><span class="sxs-lookup"><span data-stu-id="16b6a-174">**Extensions and Monitoring**: Keep the default settings.</span></span>
   - <span data-ttu-id="16b6a-175">**Diagnosespeicherkonto**: Erstellen Sie ein neues Konto, falls erforderlich.</span><span class="sxs-lookup"><span data-stu-id="16b6a-175">**Diagnostics storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="16b6a-176">Wählen Sie **OK**, um Ihre Konfiguration zu speichern.</span><span class="sxs-lookup"><span data-stu-id="16b6a-176">Select **OK** to save your configuration.</span></span>

     ![Konfigurieren der optionalen VM-Features im Azure Stack Hub-Benutzerportal](media/solution-deployment-guide-hybrid/image4.png)

8. <span data-ttu-id="16b6a-178">Konfigurieren Sie unter **SQL Server-Einstellungen** die folgenden Einstellungen:</span><span class="sxs-lookup"><span data-stu-id="16b6a-178">Under **SQL Server settings**, configure the following settings:</span></span>

   - <span data-ttu-id="16b6a-179">Wählen Sie unter **SQL-Konnektivität** die Option **Öffentlich (Internet)** aus.</span><span class="sxs-lookup"><span data-stu-id="16b6a-179">For **SQL connectivity**, select **Public (Internet)**.</span></span>
   - <span data-ttu-id="16b6a-180">Behalten Sie für **Port** den Standardwert **1433** bei.</span><span class="sxs-lookup"><span data-stu-id="16b6a-180">For **Port**, keep the default, **1433**.</span></span>
   - <span data-ttu-id="16b6a-181">Wählen Sie unter **SQL-Authentifizierung** die Option **Aktivieren**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-181">For **SQL authentication**, select **Enable**.</span></span>

     > [!Note]  
     > <span data-ttu-id="16b6a-182">Wenn Sie die SQL-Authentifizierung aktivieren, sollten automatisch die „SQLAdmin“-Informationen eingefügt werden, die Sie unter **Grundlagen** konfiguriert haben.</span><span class="sxs-lookup"><span data-stu-id="16b6a-182">When you enable SQL authentication, it should auto-populate with the "SQLAdmin" information that you configured in **Basics**.</span></span>

   - <span data-ttu-id="16b6a-183">Behalten Sie für die übrigen Einstellungen die Standardwerte bei.</span><span class="sxs-lookup"><span data-stu-id="16b6a-183">For the rest of the settings, keep the defaults.</span></span> <span data-ttu-id="16b6a-184">Klicken Sie auf **OK**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-184">Select **OK**.</span></span>

     ![Konfigurieren der SQL Server-Einstellungen im Azure Stack Hub-Benutzerportal](media/solution-deployment-guide-hybrid/image5.png)

9. <span data-ttu-id="16b6a-186">Sehen Sie sich unter **Zusammenfassung** die Konfiguration der VM an, und wählen Sie dann **OK** aus, um die Bereitstellung zu starten.</span><span class="sxs-lookup"><span data-stu-id="16b6a-186">On **Summary**, review the VM configuration and then select **OK** to start the deployment.</span></span>

    ![Zusammenfassung der Konfiguration im Azure Stack Hub-Benutzerportal](media/solution-deployment-guide-hybrid/image6.png)

10. <span data-ttu-id="16b6a-188">Die Erstellung der neuen VM dauert einige Zeit.</span><span class="sxs-lookup"><span data-stu-id="16b6a-188">It takes some time to create the new VM.</span></span> <span data-ttu-id="16b6a-189">Sie können den STATUS Ihrer VMs unter **Virtuelle Computer** anzeigen.</span><span class="sxs-lookup"><span data-stu-id="16b6a-189">You can view the STATUS of your VMs in **Virtual machines**.</span></span>

    ![VM-Status im Azure Stack Hub-Benutzerportal](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a><span data-ttu-id="16b6a-191">Erstellen von Web-Apps in Azure und Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="16b6a-191">Create web apps in Azure and Azure Stack Hub</span></span>

<span data-ttu-id="16b6a-192">Azure App Service vereinfacht die Ausführung und Verwaltung einer Web-App.</span><span class="sxs-lookup"><span data-stu-id="16b6a-192">The Azure App Service simplifies running and managing a web app.</span></span> <span data-ttu-id="16b6a-193">Da Azure Stack Hub mit Azure konsistent ist, kann App Service in beiden Umgebungen ausgeführt werden.</span><span class="sxs-lookup"><span data-stu-id="16b6a-193">Because Azure Stack Hub is consistent with Azure,  the App Service can run in both environments.</span></span> <span data-ttu-id="16b6a-194">Sie verwenden App Service zum Hosten Ihrer App.</span><span class="sxs-lookup"><span data-stu-id="16b6a-194">You'll use the App Service to host your app.</span></span>

### <a name="create-web-apps"></a><span data-ttu-id="16b6a-195">Erstellen von Web-Apps</span><span class="sxs-lookup"><span data-stu-id="16b6a-195">Create web apps</span></span>

1. <span data-ttu-id="16b6a-196">Erstellen Sie eine Web-App in Azure, indem Sie die Anleitung unter [Verwalten eines App Service-Plans in Azure](/azure/app-service/app-service-plan-manage#create-an-app-service-plan) befolgen.</span><span class="sxs-lookup"><span data-stu-id="16b6a-196">Create a web app in Azure by following the instructions in [Manage an App Service plan in Azure](/azure/app-service/app-service-plan-manage#create-an-app-service-plan).</span></span> <span data-ttu-id="16b6a-197">Stellen Sie sicher, dass Sie die Web-App unter demselben Abonnement und derselben Ressourcengruppe wie für Ihr Hybridnetzwerk anordnen.</span><span class="sxs-lookup"><span data-stu-id="16b6a-197">Make sure you put the web app in the same subscription and resource group as your hybrid network.</span></span>

2. <span data-ttu-id="16b6a-198">Wiederholen Sie den vorherigen Schritt (1) in Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="16b6a-198">Repeat the previous step (1) in Azure Stack Hub.</span></span>

### <a name="add-route-for-azure-stack-hub"></a><span data-ttu-id="16b6a-199">Hinzufügen der Route für Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="16b6a-199">Add route for Azure Stack Hub</span></span>

<span data-ttu-id="16b6a-200">App Service in Azure Stack Hub muss über das öffentliche Internet geroutet werden können, damit Benutzer auf Ihre App zugreifen können.</span><span class="sxs-lookup"><span data-stu-id="16b6a-200">The App Service on Azure Stack Hub must be routable from the public internet to let users access your app.</span></span> <span data-ttu-id="16b6a-201">Wenn Ihre Azure Stack Hub-Instanz über das Internet zugänglich ist, notieren Sie sich die öffentliche IP-Adresse oder URL der Azure Stack Hub-Web-App.</span><span class="sxs-lookup"><span data-stu-id="16b6a-201">If your Azure Stack Hub is accessible from the internet, make a note of the public-facing IP address or URL for the Azure Stack Hub web app.</span></span>

<span data-ttu-id="16b6a-202">Bei Verwendung eines ASDK können Sie [eine statische NAT-Zuordnung konfigurieren](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal), um App Service außerhalb der virtuellen Umgebung verfügbar zu machen.</span><span class="sxs-lookup"><span data-stu-id="16b6a-202">If you're using an ASDK, you can [configure a static NAT mapping](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) to expose App Service outside the virtual environment.</span></span>

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a><span data-ttu-id="16b6a-203">Verbinden einer Web-App in Azure mit einem Hybridnetzwerk</span><span class="sxs-lookup"><span data-stu-id="16b6a-203">Connect a web app in Azure to a hybrid network</span></span>

<span data-ttu-id="16b6a-204">Um die Konnektivität zwischen dem Web-Front-End in Azure und der SQL Server-Datenbank in Azure Stack Hub bereitzustellen, muss die Web-App mit dem Hybridnetzwerk zwischen Azure und Azure Stack Hub verbunden werden.</span><span class="sxs-lookup"><span data-stu-id="16b6a-204">To provide connectivity between the web front end in Azure and the SQL Server database in Azure Stack Hub, the web app must be connected to the hybrid network between Azure and Azure Stack Hub.</span></span> <span data-ttu-id="16b6a-205">Folgendes ist erforderlich, um die Konnektivität zu aktivieren:</span><span class="sxs-lookup"><span data-stu-id="16b6a-205">To enable connectivity, you'll have to:</span></span>

- <span data-ttu-id="16b6a-206">Konfigurieren der Punkt-zu-Site-Konnektivität.</span><span class="sxs-lookup"><span data-stu-id="16b6a-206">Configure point-to-site connectivity.</span></span>
- <span data-ttu-id="16b6a-207">Konfigurieren der Web-App.</span><span class="sxs-lookup"><span data-stu-id="16b6a-207">Configure the web app.</span></span>
- <span data-ttu-id="16b6a-208">Ändern des Gateways für lokale Netzwerke in Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="16b6a-208">Modify the local network gateway in Azure Stack Hub.</span></span>

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a><span data-ttu-id="16b6a-209">Konfigurieren des virtuellen Azure-Netzwerks für Punkt-zu-Standort-Konnektivität</span><span class="sxs-lookup"><span data-stu-id="16b6a-209">Configure the Azure virtual network for point-to-site connectivity</span></span>

<span data-ttu-id="16b6a-210">Das Gateway für virtuelle Netzwerke auf der Azure-Seite des Hybridnetzwerks muss Point-to-Site-Verbindungen zulassen, um in Azure App Service integriert werden zu können.</span><span class="sxs-lookup"><span data-stu-id="16b6a-210">The virtual network gateway in the Azure side of the hybrid network must allow point-to-site connections to integrate with Azure App Service.</span></span>

1. <span data-ttu-id="16b6a-211">Navigieren Sie in Azure zur Seite für das Gateway für virtuelle Netzwerke.</span><span class="sxs-lookup"><span data-stu-id="16b6a-211">In Azure, go to the virtual network gateway page.</span></span> <span data-ttu-id="16b6a-212">Wählen Sie unter **Einstellungen** die Option **Punkt-zu-Standort-Konfiguration**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-212">Under **Settings**, select **Point-to-site configuration**.</span></span>

    ![Punkt-zu-Standort-Option im Gateway für virtuelle Azure-Netzwerke](media/solution-deployment-guide-hybrid/image8.png)

2. <span data-ttu-id="16b6a-214">Wählen Sie **Jetzt konfigurieren**, um die Punkt-zu-Standort-Konfiguration zu starten.</span><span class="sxs-lookup"><span data-stu-id="16b6a-214">Select **Configure now** to configure point-to-site.</span></span>

    ![Starten der Punkt-zu-Standort-Konfiguration im Gateway für virtuelle Azure-Netzwerke](media/solution-deployment-guide-hybrid/image9.png)

3. <span data-ttu-id="16b6a-216">Geben Sie auf der Konfigurationsseite **Punkt-zu-Standort** im Feld **Adresspool** den privaten IP-Adressbereich ein, den Sie verwenden möchten.</span><span class="sxs-lookup"><span data-stu-id="16b6a-216">On the **Point-to-site** configuration page, enter the private IP address range that you want to use in **Address pool**.</span></span>

   > [!Note]  
   > <span data-ttu-id="16b6a-217">Stellen Sie sicher, dass sich der angegebene Bereich nicht mit Adressbereichen überlappt, die bereits von Subnetzen in den globalen Azure- oder Azure Stack Hub-Komponenten des Hybridnetzwerks verwendet werden.</span><span class="sxs-lookup"><span data-stu-id="16b6a-217">Make sure that the range you specify doesn't overlap with any of the address ranges already used by subnets in the global Azure or Azure Stack Hub components of the hybrid network.</span></span>

   <span data-ttu-id="16b6a-218">Deaktivieren Sie unter **Tunneltyp** die Option **IKEv2-VPN**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-218">Under **Tunnel Type**, uncheck the **IKEv2 VPN**.</span></span> <span data-ttu-id="16b6a-219">Wählen Sie **Speichern**, um die Punkt-zu-Standort-Konfiguration abzuschließen.</span><span class="sxs-lookup"><span data-stu-id="16b6a-219">Select **Save** to finish configuring point-to-site.</span></span>

   ![Punkt-zu-Standort-Einstellungen im Gateway für virtuelle Azure-Netzwerke](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a><span data-ttu-id="16b6a-221">Integrieren der Azure App Service-App in das Hybridnetzwerk</span><span class="sxs-lookup"><span data-stu-id="16b6a-221">Integrate the Azure App Service app with the hybrid network</span></span>

1. <span data-ttu-id="16b6a-222">Befolgen Sie die Anleitung unter [VNET-Integration, die ein Gateway erfordert](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration), um die App mit dem Azure VNET zu verbinden.</span><span class="sxs-lookup"><span data-stu-id="16b6a-222">To connect the app to the Azure VNet, follow the instructions in [Gateway required VNet integration](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).</span></span>

2. <span data-ttu-id="16b6a-223">Navigieren Sie für den App Service-Plan, der die Web-App hostet, zu **Einstellungen**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-223">Go to **Settings** for the App Service plan hosting the web app.</span></span> <span data-ttu-id="16b6a-224">Wählen Sie unter **Einstellungen** die Option **Netzwerk**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-224">In **Settings**, select **Networking**.</span></span>

    ![Konfigurieren des Netzwerks für den App Service-Plan](media/solution-deployment-guide-hybrid/image11.png)

3. <span data-ttu-id="16b6a-226">Wählen Sie unter **VNET-Integration** die Option **Klicken Sie zur Verwaltung hier**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-226">In **VNET Integration**, select **Click here to manage**.</span></span>

    ![Verwalten der VNET-Integration für den App Service-Plan](media/solution-deployment-guide-hybrid/image12.png)

4. <span data-ttu-id="16b6a-228">Wählen Sie das VNET aus, das Sie konfigurieren möchten.</span><span class="sxs-lookup"><span data-stu-id="16b6a-228">Select the VNET that you want to configure.</span></span> <span data-ttu-id="16b6a-229">Geben Sie unter **AN DAS VNET WEITERGELEITETE IP-ADRESSEN** den IP-Adressbereich für das Azure VNET, das Azure Stack Hub VNET und die Punkt-zu-Standort-Adressräume ein.</span><span class="sxs-lookup"><span data-stu-id="16b6a-229">Under **IP ADDRESSES ROUTED TO VNET**, enter the IP address range for the Azure VNet, the Azure Stack Hub VNet, and the point-to-site address spaces.</span></span> <span data-ttu-id="16b6a-230">Wählen Sie **Speichern**, um diese Einstellungen zu überprüfen und zu speichern.</span><span class="sxs-lookup"><span data-stu-id="16b6a-230">Select **Save** to validate and save these settings.</span></span>

    ![IP-Adressbereiche für die Weiterleitung in der VNET-Integration](media/solution-deployment-guide-hybrid/image13.png)

<span data-ttu-id="16b6a-232">Weitere Informationen zur Integration von App Service in Azure VNETs finden Sie unter [Integrieren Ihrer App in ein Azure Virtual Network](/azure/app-service/web-sites-integrate-with-vnet).</span><span class="sxs-lookup"><span data-stu-id="16b6a-232">To learn more about how App Service integrates with Azure VNets, see [Integrate your app with an Azure Virtual Network](/azure/app-service/web-sites-integrate-with-vnet).</span></span>

### <a name="configure-the-azure-stack-hub-virtual-network"></a><span data-ttu-id="16b6a-233">Konfigurieren des virtuellen Azure Stack Hub-Netzwerks</span><span class="sxs-lookup"><span data-stu-id="16b6a-233">Configure the Azure Stack Hub virtual network</span></span>

<span data-ttu-id="16b6a-234">Das Gateway für lokale Netzwerke im virtuellen Azure Stack Hub-Netzwerk muss zum Weiterleiten von Datenverkehr aus dem Punkt-zu-Standort-Adressbereich von App Service konfiguriert werden.</span><span class="sxs-lookup"><span data-stu-id="16b6a-234">The local network gateway in the Azure Stack Hub virtual network needs to be configured to route traffic from the App Service point-to-site address range.</span></span>

1. <span data-ttu-id="16b6a-235">Navigieren Sie in Azure Stack Hub zu **Lokales Netzwerkgateway**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-235">In Azure Stack Hub, go to **Local network gateway**.</span></span> <span data-ttu-id="16b6a-236">Wählen Sie unter **Einstellungen** die Option **Konfiguration**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-236">Under **Settings**, select **Configuration**.</span></span>

    ![Konfigurationsoption für Gateway im lokalen Netzwerkgateway für Azure Stack Hub](media/solution-deployment-guide-hybrid/image14.png)

2. <span data-ttu-id="16b6a-238">Geben Sie unter **Adressraum** den Punkt-zu-Standort-Adressbereich für das Gateway für virtuelle Netzwerke in Azure ein.</span><span class="sxs-lookup"><span data-stu-id="16b6a-238">In **Address space**, enter the point-to-site address range for the virtual network gateway in Azure.</span></span>

    ![Punkt-zu-Standort-Adressraum im lokalen Netzwerkgateway für Azure Stack Hub](media/solution-deployment-guide-hybrid/image15.png)

3. <span data-ttu-id="16b6a-240">Wählen Sie **Speichern** aus, um die Konfiguration zu überprüfen und zu speichern.</span><span class="sxs-lookup"><span data-stu-id="16b6a-240">Select **Save** to validate and save the configuration.</span></span>

## <a name="configure-dns-for-cross-cloud-scaling"></a><span data-ttu-id="16b6a-241">Konfigurieren von DNS für die cloudübergreifende Skalierung</span><span class="sxs-lookup"><span data-stu-id="16b6a-241">Configure DNS for cross-cloud scaling</span></span>

<span data-ttu-id="16b6a-242">Indem Benutzer das DNS für cloudübergreifende Apps richtig konfigurieren, können sie auf die globale Azure-Umgebung und die Azure Stack Hub-Instanzen Ihrer Web-App zugreifen.</span><span class="sxs-lookup"><span data-stu-id="16b6a-242">By properly configuring DNS for cross-cloud apps, users can access the global Azure and Azure Stack Hub instances of your web app.</span></span> <span data-ttu-id="16b6a-243">Mit der DNS-Konfiguration für dieses Tutorial kann Azure Traffic Manager auch Datenverkehr weiterleiten, wenn sich die Last erhöht oder verringert.</span><span class="sxs-lookup"><span data-stu-id="16b6a-243">The DNS configuration for this tutorial also lets Azure Traffic Manager route traffic when the load increases or decreases.</span></span>

<span data-ttu-id="16b6a-244">In diesem Tutorial wird Azure DNS zum Verwalten des DNS verwendet, weil App Service-Domänen nicht funktionieren.</span><span class="sxs-lookup"><span data-stu-id="16b6a-244">This tutorial uses Azure DNS to manage the DNS because App Service domains won't work.</span></span>

### <a name="create-subdomains"></a><span data-ttu-id="16b6a-245">Erstellen von Unterdomänen</span><span class="sxs-lookup"><span data-stu-id="16b6a-245">Create subdomains</span></span>

<span data-ttu-id="16b6a-246">Da für Traffic Manager DNS-CNAME-Einträge verwendet werden, wird eine Unterdomäne benötigt, um Datenverkehr korrekt an die Endpunkte weiterleiten zu können.</span><span class="sxs-lookup"><span data-stu-id="16b6a-246">Because Traffic Manager relies on DNS CNAMEs, a subdomain is needed to properly route traffic to endpoints.</span></span> <span data-ttu-id="16b6a-247">Weitere Informationen zu DNS-Einträgen und zur Domänenzuordnung finden Sie unter [Zuordnen von Domänen mithilfe von Traffic Manager](/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span><span class="sxs-lookup"><span data-stu-id="16b6a-247">For more information about DNS records and domain mapping, see [map domains with Traffic Manager](/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span></span>

<span data-ttu-id="16b6a-248">Für den Azure-Endpunkt erstellen Sie eine Unterdomäne, die Benutzer zum Zugreifen auf Ihre Web-App verwenden können.</span><span class="sxs-lookup"><span data-stu-id="16b6a-248">For the Azure endpoint, you'll create a subdomain that users can use to access your web app.</span></span> <span data-ttu-id="16b6a-249">Für dieses Tutorial können Sie **app.northwind.com** verwenden, aber Sie sollten diesen Wert basierend auf Ihrer eigenen Domäne anpassen.</span><span class="sxs-lookup"><span data-stu-id="16b6a-249">For this tutorial, can use **app.northwind.com**, but you should customize this value based on your own domain.</span></span>

<span data-ttu-id="16b6a-250">Außerdem müssen Sie eine Unterdomäne mit einem A-Eintrag für den Azure Stack Hub-Endpunkt erstellen.</span><span class="sxs-lookup"><span data-stu-id="16b6a-250">You'll also need to create a subdomain with an A record for the Azure Stack Hub endpoint.</span></span> <span data-ttu-id="16b6a-251">Sie können **azurestack.northwind.com** verwenden.</span><span class="sxs-lookup"><span data-stu-id="16b6a-251">You can use **azurestack.northwind.com**.</span></span>

### <a name="configure-a-custom-domain-in-azure"></a><span data-ttu-id="16b6a-252">Konfigurieren einer benutzerdefinierten Domäne in Azure</span><span class="sxs-lookup"><span data-stu-id="16b6a-252">Configure a custom domain in Azure</span></span>

1. <span data-ttu-id="16b6a-253">Fügen Sie den Hostnamen **app.northwind.com** der Azure-Web-App hinzu, indem Sie [Azure App Service einen CNAME-Eintrag zuordnen](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span><span class="sxs-lookup"><span data-stu-id="16b6a-253">Add the **app.northwind.com** hostname to the Azure web app by [mapping a CNAME to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span>

### <a name="configure-custom-domains-in-azure-stack-hub"></a><span data-ttu-id="16b6a-254">Konfigurieren von benutzerdefinierten Domänen in Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="16b6a-254">Configure custom domains in Azure Stack Hub</span></span>

1. <span data-ttu-id="16b6a-255">Fügen Sie den Hostnamen **azurestack.northwind.com** der Azure Stack Hub-Web-App hinzu, indem Sie [Azure App Service einen A-Eintrag zuordnen](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span><span class="sxs-lookup"><span data-stu-id="16b6a-255">Add the **azurestack.northwind.com** hostname to the Azure Stack Hub web app by [mapping an A record to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span></span> <span data-ttu-id="16b6a-256">Verwenden Sie für die App Service-App die IP-Adresse, die über das Internet geroutet werden kann.</span><span class="sxs-lookup"><span data-stu-id="16b6a-256">Use the internet-routable IP address for the App Service app.</span></span>

2. <span data-ttu-id="16b6a-257">Fügen Sie den Hostnamen **app.northwind.com** der Azure Stack Hub-Web-App hinzu, indem Sie [Azure App Service einen CNAME-Eintrag zuordnen](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span><span class="sxs-lookup"><span data-stu-id="16b6a-257">Add the **app.northwind.com** hostname to the Azure Stack Hub web app by [mapping a CNAME to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span> <span data-ttu-id="16b6a-258">Verwenden Sie den Hostnamen, den Sie im vorherigen Schritt (1) konfiguriert haben, als Ziel für den CNAME-Eintrag.</span><span class="sxs-lookup"><span data-stu-id="16b6a-258">Use the hostname you configured in the previous step (1) as the target for the CNAME.</span></span>

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a><span data-ttu-id="16b6a-259">Konfigurieren von SSL-Zertifikaten für die cloudübergreifende Skalierung</span><span class="sxs-lookup"><span data-stu-id="16b6a-259">Configure SSL certificates for cross-cloud scaling</span></span>

<span data-ttu-id="16b6a-260">Es ist wichtig, dass Sie sicherstellen, dass von Ihrer Web-App erfasste vertrauliche Daten bei der Übertragung zur und der Speicherung in der SQL-Datenbank sicher sind.</span><span class="sxs-lookup"><span data-stu-id="16b6a-260">It's important to ensure sensitive data collected by your web app is secure in transit to and when stored on the SQL database.</span></span>

<span data-ttu-id="16b6a-261">Sie konfigurieren Ihre Azure- und Azure Stack Hub-Web-Apps so, dass für den gesamten eingehenden Datenverkehr SSL-Zertifikate verwendet werden.</span><span class="sxs-lookup"><span data-stu-id="16b6a-261">You'll configure your Azure and Azure Stack Hub web apps to use SSL certificates for all incoming traffic.</span></span>

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a><span data-ttu-id="16b6a-262">Hinzufügen von SSL zu Azure und Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="16b6a-262">Add SSL to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="16b6a-263">Fügen Sie SSL für Azure wie folgt hinzu:</span><span class="sxs-lookup"><span data-stu-id="16b6a-263">To add SSL to Azure:</span></span>

1. <span data-ttu-id="16b6a-264">Stellen Sie sicher, dass das beschaffte SSL-Zertifikat für die von Ihnen erstellte Unterdomäne gültig ist.</span><span class="sxs-lookup"><span data-stu-id="16b6a-264">Make sure that the SSL certificate you get is valid for the subdomain you created.</span></span> <span data-ttu-id="16b6a-265">(Hierbei können auch Platzhalterzertifikate verwendet werden.)</span><span class="sxs-lookup"><span data-stu-id="16b6a-265">(It's okay to use wildcard certificates.)</span></span>

2. <span data-ttu-id="16b6a-266">Befolgen Sie in Azure die Anleitung in den Abschnitten **Vorbereiten Ihrer Web-App** und **Binden Ihres SSL-Zertifikats** des Artikels [Binden eines vorhandenen benutzerdefinierten SSL-Zertifikats an Azure-Web-Apps](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="16b6a-266">In Azure, follow the instructions in the **Prepare your web app** and **Bind your SSL certificate** sections of the [Bind an existing custom SSL certificate to Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span> <span data-ttu-id="16b6a-267">Wählen Sie unter **SSL-Typ** die Option **SNI-basiertes SSL**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-267">Select **SNI-based SSL** as the **SSL Type**.</span></span>

3. <span data-ttu-id="16b6a-268">Leiten Sie den gesamten Datenverkehr an den HTTPS-Port um.</span><span class="sxs-lookup"><span data-stu-id="16b6a-268">Redirect all traffic to the HTTPS port.</span></span> <span data-ttu-id="16b6a-269">Befolgen Sie die Anleitung im Abschnitt **Erzwingen von HTTPS** des Artikels [Binden eines vorhandenen benutzerdefinierten SSL-Zertifikats an Azure-Web-Apps](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="16b6a-269">Follow the instructions in the   **Enforce HTTPS** section of the [Bind an existing custom SSL certificate to Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span>

<span data-ttu-id="16b6a-270">Fügen Sie SSL für Azure Stack Hub wie folgt hinzu:</span><span class="sxs-lookup"><span data-stu-id="16b6a-270">To add SSL to Azure Stack Hub:</span></span>

1. <span data-ttu-id="16b6a-271">Wiederholen Sie die Schritte 1 bis 3, die Sie für Azure verwendet haben.</span><span class="sxs-lookup"><span data-stu-id="16b6a-271">Repeat steps 1-3 that you used for Azure.</span></span>

## <a name="configure-and-deploy-the-web-app"></a><span data-ttu-id="16b6a-272">Konfigurieren und Bereitstellen der Web-App</span><span class="sxs-lookup"><span data-stu-id="16b6a-272">Configure and deploy the web app</span></span>

<span data-ttu-id="16b6a-273">Sie konfigurieren den App-Code so, dass die Telemetriedaten an die richtige Application Insights-Instanz gemeldet werden, und konfigurieren die Web-Apps mit den richtigen Verbindungszeichenfolgen.</span><span class="sxs-lookup"><span data-stu-id="16b6a-273">You'll configure the app code to report telemetry to the correct Application Insights instance and configure the web apps with the right connection strings.</span></span> <span data-ttu-id="16b6a-274">Weitere Informationen zu Application Insights finden Sie unter [Was ist Application Insights?](/azure/application-insights/app-insights-overview).</span><span class="sxs-lookup"><span data-stu-id="16b6a-274">To learn more about Application Insights, see [What is Application Insights?](/azure/application-insights/app-insights-overview)</span></span>

### <a name="add-application-insights"></a><span data-ttu-id="16b6a-275">Hinzufügen von Application Insights</span><span class="sxs-lookup"><span data-stu-id="16b6a-275">Add Application Insights</span></span>

1. <span data-ttu-id="16b6a-276">Öffnen Sie Ihre Web-App in Microsoft Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="16b6a-276">Open your web app in Microsoft Visual Studio.</span></span>

2. <span data-ttu-id="16b6a-277">[Fügen Sie Ihrem Projekt Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) hinzu, um die Telemetriedaten zu übertragen, die von Application Insights zum Erstellen von Warnungen verwendet werden, wenn der Webdatenverkehr zunimmt oder sich verringert.</span><span class="sxs-lookup"><span data-stu-id="16b6a-277">[Add Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) to your project to transmit the telemetry that Application Insights uses to create alerts when web traffic increases or decreases.</span></span>

### <a name="configure-dynamic-connection-strings"></a><span data-ttu-id="16b6a-278">Konfigurieren von dynamischen Verbindungszeichenfolgen</span><span class="sxs-lookup"><span data-stu-id="16b6a-278">Configure dynamic connection strings</span></span>

<span data-ttu-id="16b6a-279">Für jede Instanz der Web-App wird eine andere Methode zum Herstellen der Verbindung mit der SQL-Datenbank verwendet.</span><span class="sxs-lookup"><span data-stu-id="16b6a-279">Each instance of the web app will use a different method to connect to the SQL database.</span></span> <span data-ttu-id="16b6a-280">Für die App in Azure wird die private IP-Adresse der SQL Server-VM verwendet, und die App in Azure Stack Hub nutzt die öffentliche IP-Adresse der SQL Server-VM.</span><span class="sxs-lookup"><span data-stu-id="16b6a-280">The app in Azure uses the private IP address of the SQL Server VM and the app in Azure Stack Hub uses the public IP address of the SQL Server VM.</span></span>

> [!Note]  
> <span data-ttu-id="16b6a-281">Auf einem integrierten Azure Stack Hub-System sollte die öffentliche IP-Adresse nicht über das Internet geroutet werden können.</span><span class="sxs-lookup"><span data-stu-id="16b6a-281">On an Azure Stack Hub integrated system, the public IP address shouldn't be internet-routable.</span></span> <span data-ttu-id="16b6a-282">Für ein ASDK kann die öffentliche IP-Adresse nicht außerhalb des ASDK geroutet werden.</span><span class="sxs-lookup"><span data-stu-id="16b6a-282">On an ASDK, the public IP address isn't routable outside the ASDK.</span></span>

<span data-ttu-id="16b6a-283">Sie können die App Service-Umgebungsvariablen verwenden, um an jede Instanz der App eine andere Verbindungszeichenfolge zu übergeben.</span><span class="sxs-lookup"><span data-stu-id="16b6a-283">You can use App Service environment variables to pass a different connection string to each instance of the app.</span></span>

1. <span data-ttu-id="16b6a-284">Öffnen Sie die App in Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="16b6a-284">Open the app in Visual Studio.</span></span>

2. <span data-ttu-id="16b6a-285">Öffnen Sie „Startup.cs“, und suchen Sie nach dem folgenden Codeblock:</span><span class="sxs-lookup"><span data-stu-id="16b6a-285">Open Startup.cs and find the following code block:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. <span data-ttu-id="16b6a-286">Ersetzen Sie den obigen Codeblock durch den folgenden Code, in dem eine Verbindungszeichenfolge verwendet wird, die in der Datei *appsettings.json* definiert ist:</span><span class="sxs-lookup"><span data-stu-id="16b6a-286">Replace the previous code block with the following code, which uses a connection string defined in the *appsettings.json* file:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a><span data-ttu-id="16b6a-287">Konfigurieren von App Service-App-Einstellungen</span><span class="sxs-lookup"><span data-stu-id="16b6a-287">Configure App Service app settings</span></span>

1. <span data-ttu-id="16b6a-288">Erstellen Sie Verbindungszeichenfolgen für Azure und Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="16b6a-288">Create connection strings for Azure and Azure Stack Hub.</span></span> <span data-ttu-id="16b6a-289">Die Zeichenfolgen sollten mit Ausnahme der verwendeten IP-Adressen identisch sein.</span><span class="sxs-lookup"><span data-stu-id="16b6a-289">The strings should be the same, except for the IP addresses that are used.</span></span>

2. <span data-ttu-id="16b6a-290">Fügen Sie in Azure und Azure Stack Hub die entsprechende Verbindungszeichenfolge [als App-Einstellung](/azure/app-service/web-sites-configure) in der Web-App hinzu, indem Sie `SQLCONNSTR\_` als Präfix im Namen verwenden.</span><span class="sxs-lookup"><span data-stu-id="16b6a-290">In Azure and Azure Stack Hub, add the appropriate connection string [as an app setting](/azure/app-service/web-sites-configure) in the web app, using `SQLCONNSTR\_` as a prefix in the name.</span></span>

3. <span data-ttu-id="16b6a-291">**Speichern** Sie die Web-App-Einstellungen, und starten Sie die App neu.</span><span class="sxs-lookup"><span data-stu-id="16b6a-291">**Save** the web app settings and restart the app.</span></span>

## <a name="enable-automatic-scaling-in-global-azure"></a><span data-ttu-id="16b6a-292">Aktivieren der automatischen Skalierung in der globalen Azure-Umgebung</span><span class="sxs-lookup"><span data-stu-id="16b6a-292">Enable automatic scaling in global Azure</span></span>

<span data-ttu-id="16b6a-293">Wenn Sie Ihre Web-App in einer App Service-Umgebung erstellen, beginnt sie mit einer Instanz.</span><span class="sxs-lookup"><span data-stu-id="16b6a-293">When you create your web app in an App Service environment, it starts with one instance.</span></span> <span data-ttu-id="16b6a-294">Sie können automatisch aufskalieren, um Instanzen hinzuzufügen und so für Ihre App mehr Computeressourcen bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="16b6a-294">You can automatically scale out to add instances to provide more compute resources for your app.</span></span> <span data-ttu-id="16b6a-295">Auf ähnliche Weise können Sie automatisch abskalieren und die Anzahl von Instanzen reduzieren, die Ihre App benötigt.</span><span class="sxs-lookup"><span data-stu-id="16b6a-295">Similarly, you can automatically scale in and reduce the number of instances your app needs.</span></span>

> [!Note]  
> <span data-ttu-id="16b6a-296">Sie müssen über einen App Service-Plan verfügen, um das Auf- und Abskalieren zu konfigurieren.</span><span class="sxs-lookup"><span data-stu-id="16b6a-296">You need to have an App Service plan to configure scale out and scale in.</span></span> <span data-ttu-id="16b6a-297">Wenn Sie keinen Plan besitzen, sollten Sie einen erstellen, bevor Sie mit den nächsten Schritten beginnen.</span><span class="sxs-lookup"><span data-stu-id="16b6a-297">If you don't have a plan, create one before starting the next steps.</span></span>

### <a name="enable-automatic-scale-out"></a><span data-ttu-id="16b6a-298">Aktivieren der automatischen horizontalen Skalierung</span><span class="sxs-lookup"><span data-stu-id="16b6a-298">Enable automatic scale-out</span></span>

1. <span data-ttu-id="16b6a-299">Suchen Sie in Azure nach dem App Service-Plan für die Standorte, die Sie aufskalieren möchten, und wählen Sie dann die Option **Aufskalieren (App Service-Plan)** .</span><span class="sxs-lookup"><span data-stu-id="16b6a-299">In Azure, find the App Service plan for the sites you want to scale out, and then select **Scale-out (App Service plan)**.</span></span>

    ![Aufskalieren von Azure App Service](media/solution-deployment-guide-hybrid/image16.png)

2. <span data-ttu-id="16b6a-301">Wählen Sie **Automatische Skalierung aktivieren**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-301">Select **Enable autoscale**.</span></span>

    ![Aktivieren der Autoskalierung in Azure App Service](media/solution-deployment-guide-hybrid/image17.png)

3. <span data-ttu-id="16b6a-303">Geben Sie unter **Name der Einstellung für die automatische Skalierung** einen Namen ein.</span><span class="sxs-lookup"><span data-stu-id="16b6a-303">Enter a name for **Autoscale Setting Name**.</span></span> <span data-ttu-id="16b6a-304">Wählen Sie unter **Standard** für die Standardregel der automatischen Skalierung die Option **Basierend auf einer Metrik skalieren**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-304">For the **Default** auto scale rule, select **Scale based on a metric**.</span></span> <span data-ttu-id="16b6a-305">Legen Sie **Instanzgrenzwerte** auf **Minimum: 1**, **Maximum: 10** und **Standard: 1** fest.</span><span class="sxs-lookup"><span data-stu-id="16b6a-305">Set the **Instance limits** to **Minimum: 1**, **Maximum: 10**, and **Default: 1**.</span></span>

    ![Konfigurieren der Autoskalierung in Azure App Service](media/solution-deployment-guide-hybrid/image18.png)

4. <span data-ttu-id="16b6a-307">Wählen Sie **+ Regel hinzufügen**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-307">Select **+Add a rule**.</span></span>

5. <span data-ttu-id="16b6a-308">Wählen Sie unter **Metrikquelle** die Option **Aktuelle Ressource**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-308">In **Metric Source**, select **Current Resource**.</span></span> <span data-ttu-id="16b6a-309">Verwenden Sie die folgenden Kriterien und Aktionen für die Regel.</span><span class="sxs-lookup"><span data-stu-id="16b6a-309">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="16b6a-310">Kriterien</span><span class="sxs-lookup"><span data-stu-id="16b6a-310">Criteria</span></span>

1. <span data-ttu-id="16b6a-311">Wählen Sie unter **Zeitaggregation** die Option **Durchschnitt**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-311">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="16b6a-312">Wählen Sie unter **Metrikname** die Option **CPU-Prozentsatz**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-312">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="16b6a-313">Wählen Sie unter **Operator** die Option **Größer als**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-313">Under **Operator**, select **Greater than**.</span></span>

   - <span data-ttu-id="16b6a-314">Legen Sie **Schwellenwert** auf **50** fest.</span><span class="sxs-lookup"><span data-stu-id="16b6a-314">Set the **Threshold** to **50**.</span></span>
   - <span data-ttu-id="16b6a-315">Legen Sie die **Dauer** auf **10** fest.</span><span class="sxs-lookup"><span data-stu-id="16b6a-315">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="16b6a-316">Aktion</span><span class="sxs-lookup"><span data-stu-id="16b6a-316">Action</span></span>

1. <span data-ttu-id="16b6a-317">Wählen Sie unter **Vorgang** die Option **Anzahl erhöhen um**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-317">Under **Operation**, select **Increase Count by**.</span></span>

2. <span data-ttu-id="16b6a-318">Legen Sie **Instanzenanzahl** auf **2** fest.</span><span class="sxs-lookup"><span data-stu-id="16b6a-318">Set the **Instance Count** to **2**.</span></span>

3. <span data-ttu-id="16b6a-319">Legen Sie **Abklingen** auf **5** fest.</span><span class="sxs-lookup"><span data-stu-id="16b6a-319">Set the **Cool down** to **5**.</span></span>

4. <span data-ttu-id="16b6a-320">Wählen Sie **Hinzufügen**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-320">Select **Add**.</span></span>

5. <span data-ttu-id="16b6a-321">Wählen Sie **+ Regel hinzufügen**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-321">Select the **+ Add a rule**.</span></span>

6. <span data-ttu-id="16b6a-322">Wählen Sie unter **Metrikquelle** die Option **Aktuelle Ressource**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-322">In **Metric Source**, select **Current Resource.**</span></span>

   > [!Note]  
   > <span data-ttu-id="16b6a-323">Die aktuelle Ressource enthält den Namen bzw. die GUID Ihres App Service-Plans, und die Dropdownlisten **Ressourcentyp** und **Ressource** sind nicht verfügbar.</span><span class="sxs-lookup"><span data-stu-id="16b6a-323">The current resource will contain your App Service plan's name/GUID and the **Resource Type** and **Resource** drop-down lists will be unavailable.</span></span>

### <a name="enable-automatic-scale-in"></a><span data-ttu-id="16b6a-324">Aktivieren des automatischen Abskalierens</span><span class="sxs-lookup"><span data-stu-id="16b6a-324">Enable automatic scale in</span></span>

<span data-ttu-id="16b6a-325">Bei einer Verringerung des Datenverkehrs kann die Azure-Web-App die Anzahl von aktiven Instanzen reduzieren, um die Kosten zu senken.</span><span class="sxs-lookup"><span data-stu-id="16b6a-325">When traffic decreases, the Azure web app can automatically reduce the number of active instances to reduce costs.</span></span> <span data-ttu-id="16b6a-326">Diese Aktion ist weniger aggressiv als die horizontale Skalierung und minimiert die Auswirkungen auf App-Benutzer.</span><span class="sxs-lookup"><span data-stu-id="16b6a-326">This action is less aggressive than scale-out and minimizes the impact on app users.</span></span>

1. <span data-ttu-id="16b6a-327">Navigieren Sie zur Standardbedingung für das Aufskalieren unter **Standard**, und wählen Sie dann **+ Regel hinzufügen** aus.</span><span class="sxs-lookup"><span data-stu-id="16b6a-327">Go to the **Default** scale out condition, then select **+ Add a rule**.</span></span> <span data-ttu-id="16b6a-328">Verwenden Sie die folgenden Kriterien und Aktionen für die Regel.</span><span class="sxs-lookup"><span data-stu-id="16b6a-328">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="16b6a-329">Kriterien</span><span class="sxs-lookup"><span data-stu-id="16b6a-329">Criteria</span></span>

1. <span data-ttu-id="16b6a-330">Wählen Sie unter **Zeitaggregation** die Option **Durchschnitt**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-330">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="16b6a-331">Wählen Sie unter **Metrikname** die Option **CPU-Prozentsatz**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-331">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="16b6a-332">Wählen Sie unter **Operator** die Option **Kleiner als**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-332">Under **Operator**, select **Less than**.</span></span>

   - <span data-ttu-id="16b6a-333">Legen Sie **Schwellenwert** auf **30** fest.</span><span class="sxs-lookup"><span data-stu-id="16b6a-333">Set the **Threshold** to **30**.</span></span>
   - <span data-ttu-id="16b6a-334">Legen Sie die **Dauer** auf **10** fest.</span><span class="sxs-lookup"><span data-stu-id="16b6a-334">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="16b6a-335">Aktion</span><span class="sxs-lookup"><span data-stu-id="16b6a-335">Action</span></span>

1. <span data-ttu-id="16b6a-336">Wählen Sie unter **Vorgang** die Option **Anzahl verringern um**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-336">Under **Operation**, select **Decrease Count by**.</span></span>

   - <span data-ttu-id="16b6a-337">Legen Sie **Instanzenanzahl** auf **1** fest.</span><span class="sxs-lookup"><span data-stu-id="16b6a-337">Set the **Instance Count** to **1**.</span></span>
   - <span data-ttu-id="16b6a-338">Legen Sie **Abklingen** auf **5** fest.</span><span class="sxs-lookup"><span data-stu-id="16b6a-338">Set the **Cool down** to **5**.</span></span>

2. <span data-ttu-id="16b6a-339">Wählen Sie **Hinzufügen**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-339">Select **Add**.</span></span>

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a><span data-ttu-id="16b6a-340">Erstellen eines Traffic Manager-Profils mit anschließender Konfiguration für die cloudübergreifende Skalierung</span><span class="sxs-lookup"><span data-stu-id="16b6a-340">Create a Traffic Manager profile and configure cross-cloud scaling</span></span>

<span data-ttu-id="16b6a-341">Erstellen Sie ein Traffic Manager-Profil in Azure, und konfigurieren Sie dann die Endpunkte, um die cloudübergreifende Skalierung zu ermöglichen.</span><span class="sxs-lookup"><span data-stu-id="16b6a-341">Create a Traffic Manager profile in Azure and then configure endpoints to enable cross-cloud scaling.</span></span>

### <a name="create-traffic-manager-profile"></a><span data-ttu-id="16b6a-342">Erstellen eines Traffic Manager-Profils</span><span class="sxs-lookup"><span data-stu-id="16b6a-342">Create Traffic Manager profile</span></span>

1. <span data-ttu-id="16b6a-343">Wählen Sie **Ressource erstellen**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-343">Select **Create a resource**.</span></span>
2. <span data-ttu-id="16b6a-344">Wählen Sie **Netzwerk** aus.</span><span class="sxs-lookup"><span data-stu-id="16b6a-344">Select **Networking**.</span></span>
3. <span data-ttu-id="16b6a-345">Wählen Sie **Traffic Manager-Profil**, und konfigurieren Sie folgende Einstellungen:</span><span class="sxs-lookup"><span data-stu-id="16b6a-345">Select **Traffic Manager profile** and configure the following settings:</span></span>

   - <span data-ttu-id="16b6a-346">Geben Sie unter **Name** einen Namen für Ihr Profil ein.</span><span class="sxs-lookup"><span data-stu-id="16b6a-346">In **Name**, enter a name for your profile.</span></span> <span data-ttu-id="16b6a-347">Dieser Name **muss** in der Zone „trafficmanager.net“ eindeutig sein und wird genutzt, um einen neuen DNS-Namen zu erstellen (z.B. „northwindstore.trafficmanager.net“).</span><span class="sxs-lookup"><span data-stu-id="16b6a-347">This name **must** be unique in the trafficmanager.net zone and is used to create a new DNS name (for example, northwindstore.trafficmanager.net).</span></span>
   - <span data-ttu-id="16b6a-348">Wählen Sie unter **Routingmethode** die Option **Gewichtet**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-348">For **Routing method**, select the **Weighted**.</span></span>
   - <span data-ttu-id="16b6a-349">Wählen Sie unter **Abonnement** das Abonnement aus, unter dem Sie dieses Profil erstellen möchten.</span><span class="sxs-lookup"><span data-stu-id="16b6a-349">For **Subscription**, select the subscription you want to create  this profile in.</span></span>
   - <span data-ttu-id="16b6a-350">Erstellen Sie unter **Ressourcengruppe** eine neue Ressourcengruppe für dieses Profil.</span><span class="sxs-lookup"><span data-stu-id="16b6a-350">In **Resource Group**, create a new resource group for this profile.</span></span>
   - <span data-ttu-id="16b6a-351">Wählen Sie unter **Ressourcengruppenstandort** den Speicherort für die Ressourcengruppe aus.</span><span class="sxs-lookup"><span data-stu-id="16b6a-351">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="16b6a-352">Diese Einstellung bezieht sich auf den Speicherort der Ressourcengruppe und hat keine Auswirkungen auf das global bereitgestellte Traffic Manager-Profil.</span><span class="sxs-lookup"><span data-stu-id="16b6a-352">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile that's deployed globally.</span></span>

4. <span data-ttu-id="16b6a-353">Klicken Sie auf **Erstellen**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-353">Select **Create**.</span></span>

    ![Erstellen eines Traffic Manager-Profils](media/solution-deployment-guide-hybrid/image19.png)

   <span data-ttu-id="16b6a-355">Wenn die globale Bereitstellung Ihres Traffic Manager-Profils abgeschlossen ist, wird es in der Liste mit den Ressourcen für die Ressourcengruppe angezeigt, unter der Sie es erstellt haben.</span><span class="sxs-lookup"><span data-stu-id="16b6a-355">When the global deployment of your Traffic Manager profile is complete, it's shown in the list of resources for the resource group you created it under.</span></span>

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="16b6a-356">Hinzufügen von Traffic Manager-Endpunkten</span><span class="sxs-lookup"><span data-stu-id="16b6a-356">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="16b6a-357">Suchen Sie nach dem Traffic Manager-Profil, das Sie erstellt haben.</span><span class="sxs-lookup"><span data-stu-id="16b6a-357">Search for the Traffic Manager profile you created.</span></span> <span data-ttu-id="16b6a-358">Wählen Sie das Profil aus, nachdem Sie die Navigation zur Ressourcengruppe für das Profil durchgeführt haben.</span><span class="sxs-lookup"><span data-stu-id="16b6a-358">If you navigated to the resource group for the profile, select the profile.</span></span>

2. <span data-ttu-id="16b6a-359">Wählen Sie im **Traffic Manager-Profil** unter **EINSTELLUNGEN** die Option **Endpunkte**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-359">In **Traffic Manager profile**, under **SETTINGS**, select **Endpoints**.</span></span>

3. <span data-ttu-id="16b6a-360">Wählen Sie **Hinzufügen**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-360">Select **Add**.</span></span>

4. <span data-ttu-id="16b6a-361">Verwenden Sie unter **Endpunkt hinzufügen** die folgenden Einstellungen für Azure Stack Hub:</span><span class="sxs-lookup"><span data-stu-id="16b6a-361">In **Add endpoint**, use the following settings for Azure Stack Hub:</span></span>

   - <span data-ttu-id="16b6a-362">Wählen Sie unter **Typ** die Option **Externer Endpunkt**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-362">For **Type**, select **External endpoint**.</span></span>
   - <span data-ttu-id="16b6a-363">Geben Sie einen **Namen** für den Endpunkt ein.</span><span class="sxs-lookup"><span data-stu-id="16b6a-363">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="16b6a-364">Geben Sie unter **Fully-qualified domain name (FQDN) or IP** (Vollqualifizierter Domänenname [FQDN] oder IP) die externe URL für Ihre Azure Stack Hub-Web-App ein.</span><span class="sxs-lookup"><span data-stu-id="16b6a-364">For **Fully qualified domain name (FQDN) or IP**, enter the external URL for your Azure Stack Hub web app.</span></span>
   - <span data-ttu-id="16b6a-365">Behalten Sie für **Gewichtung** den Standardwert **1** bei.</span><span class="sxs-lookup"><span data-stu-id="16b6a-365">For **Weight**, keep the default, **1**.</span></span> <span data-ttu-id="16b6a-366">Diese Gewichtung bewirkt, dass der gesamte Datenverkehr an diesen Endpunkt geleitet wird, sofern sein Status intakt ist.</span><span class="sxs-lookup"><span data-stu-id="16b6a-366">This weight results in all traffic going to this endpoint if it's healthy.</span></span>
   - <span data-ttu-id="16b6a-367">Lassen Sie **Als deaktiviert hinzufügen** deaktiviert.</span><span class="sxs-lookup"><span data-stu-id="16b6a-367">Leave **Add as disabled** unchecked.</span></span>

5. <span data-ttu-id="16b6a-368">Wählen Sie **OK**, um den Azure Stack Hub-Endpunkt zu speichern.</span><span class="sxs-lookup"><span data-stu-id="16b6a-368">Select **OK** to save the Azure Stack Hub endpoint.</span></span>

<span data-ttu-id="16b6a-369">Als Nächstes konfigurieren Sie den Azure-Endpunkt.</span><span class="sxs-lookup"><span data-stu-id="16b6a-369">You'll configure the Azure endpoint next.</span></span>

1. <span data-ttu-id="16b6a-370">Wählen Sie unter **Traffic Manager-Profil** die Option **Endpunkte**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-370">On **Traffic Manager profile**, select **Endpoints**.</span></span>
2. <span data-ttu-id="16b6a-371">Wählen Sie **+ Hinzufügen** aus.</span><span class="sxs-lookup"><span data-stu-id="16b6a-371">Select **+Add**.</span></span>
3. <span data-ttu-id="16b6a-372">Verwenden Sie unter **Endpunkt hinzufügen** die folgenden Einstellungen für Azure:</span><span class="sxs-lookup"><span data-stu-id="16b6a-372">On **Add endpoint**, use the following settings for Azure:</span></span>

   - <span data-ttu-id="16b6a-373">Wählen Sie für **Typ** die Option **Azure-Endpunkt**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-373">For **Type**, select **Azure endpoint**.</span></span>
   - <span data-ttu-id="16b6a-374">Geben Sie einen **Namen** für den Endpunkt ein.</span><span class="sxs-lookup"><span data-stu-id="16b6a-374">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="16b6a-375">Wählen Sie unter **Zielressourcentyp** die Option **App Service**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-375">For **Target resource type**, select **App Service**.</span></span>
   - <span data-ttu-id="16b6a-376">Wählen Sie unter **Zielressource** die Option **App Service auswählen**, um eine Liste mit den Web-Apps für dasselbe Abonnement anzuzeigen.</span><span class="sxs-lookup"><span data-stu-id="16b6a-376">For **Target resource**, select **Choose an app service** to see a list of Web Apps in the same subscription.</span></span>
   - <span data-ttu-id="16b6a-377">Wählen Sie unter **Ressource** den App Service aus, den Sie als ersten Endpunkt hinzufügen möchten.</span><span class="sxs-lookup"><span data-stu-id="16b6a-377">In **Resource**, pick the App service that you want to add as the first endpoint.</span></span>
   - <span data-ttu-id="16b6a-378">Wählen Sie unter **Gewichtung** den Wert **2**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-378">For **Weight**, select **2**.</span></span> <span data-ttu-id="16b6a-379">Diese Einstellung führt dazu, dass der gesamte Datenverkehr an diesen Endpunkt geleitet wird, wenn der primäre Endpunkt fehlerhaft ist oder Sie über eine Regel oder Warnung verfügen, mit der Datenverkehr bei der Auslösung umgeleitet wird.</span><span class="sxs-lookup"><span data-stu-id="16b6a-379">This setting results in all traffic going to this endpoint if the primary endpoint is unhealthy, or if you have a rule/alert that redirects traffic when triggered.</span></span>
   - <span data-ttu-id="16b6a-380">Lassen Sie **Als deaktiviert hinzufügen** deaktiviert.</span><span class="sxs-lookup"><span data-stu-id="16b6a-380">Leave **Add as disabled** unchecked.</span></span>

4. <span data-ttu-id="16b6a-381">Wählen Sie **OK**, um den Azure-Endpunkt zu speichern.</span><span class="sxs-lookup"><span data-stu-id="16b6a-381">Select **OK** to save the Azure endpoint.</span></span>

<span data-ttu-id="16b6a-382">Nachdem beide Endpunkte konfiguriert wurden, werden sie im **Traffic Manager-Profil** aufgeführt, wenn Sie **Endpunkte** wählen.</span><span class="sxs-lookup"><span data-stu-id="16b6a-382">After both endpoints are configured, they're listed in **Traffic Manager profile** when you select **Endpoints**.</span></span> <span data-ttu-id="16b6a-383">Im Beispiel im folgenden Screenshot werden zwei Endpunkte jeweils mit Status- und Konfigurationsinformationen angezeigt.</span><span class="sxs-lookup"><span data-stu-id="16b6a-383">The example in the following screen capture shows two endpoints, with status and configuration information for each one.</span></span>

![Endpunkte im Traffic Manager-Profil](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting"></a><span data-ttu-id="16b6a-385">Einrichten der Application Insights-Überwachung und -Warnungen</span><span class="sxs-lookup"><span data-stu-id="16b6a-385">Set up Application Insights monitoring and alerting</span></span>

<span data-ttu-id="16b6a-386">Mit Azure Application Insights können Sie Ihre App überwachen und basierend auf den von Ihnen konfigurierten Bedingungen Warnungen senden.</span><span class="sxs-lookup"><span data-stu-id="16b6a-386">Azure Application Insights lets you monitor your app and send alerts based on conditions you configure.</span></span> <span data-ttu-id="16b6a-387">Beispiele: Die App ist nicht verfügbar, weist Fehler auf oder zeigt Leistungsprobleme an.</span><span class="sxs-lookup"><span data-stu-id="16b6a-387">Some examples are: the app is unavailable, is experiencing failures, or is showing performance issues.</span></span>

<span data-ttu-id="16b6a-388">Sie nutzen Application Insights-Metriken, um Warnungen zu erstellen.</span><span class="sxs-lookup"><span data-stu-id="16b6a-388">You'll use Application Insights metrics to create alerts.</span></span> <span data-ttu-id="16b6a-389">Wenn diese Warnungen ausgelöst werden, wird für die Instanz Ihrer Web-App automatisch von Azure Stack Hub zu Azure gewechselt, um aufzuskalieren, und dann zurück zu Azure Stack Hub, um abzuskalieren.</span><span class="sxs-lookup"><span data-stu-id="16b6a-389">When these alerts trigger, your web app's instance will automatically switch from Azure Stack Hub to Azure to scale out, and then back to Azure Stack Hub to scale in.</span></span>

### <a name="create-an-alert-from-metrics"></a><span data-ttu-id="16b6a-390">Warnung auf Grundlage von Metriken erstellen</span><span class="sxs-lookup"><span data-stu-id="16b6a-390">Create an alert from metrics</span></span>

<span data-ttu-id="16b6a-391">Navigieren Sie zur Ressourcengruppe für dieses Tutorial, und wählen Sie dann die Application Insights-Instanz aus, um **Application Insights** zu öffnen.</span><span class="sxs-lookup"><span data-stu-id="16b6a-391">Go to the resource group for this tutorial and then select the Application Insights instance to open **Application Insights**.</span></span>

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

<span data-ttu-id="16b6a-393">Sie verwenden diese Ansicht, um eine Warnung für das Aufskalieren und eine Warnung für das Abskalieren zu erstellen.</span><span class="sxs-lookup"><span data-stu-id="16b6a-393">You'll use this view to create a scale-out alert and a scale-in alert.</span></span>

### <a name="create-the-scale-out-alert"></a><span data-ttu-id="16b6a-394">Erstellen der Warnung für das horizontale Hochskalieren</span><span class="sxs-lookup"><span data-stu-id="16b6a-394">Create the scale-out alert</span></span>

1. <span data-ttu-id="16b6a-395">Wählen Sie unter **KONFIGURIEREN** die Option **Warnungen (klassisch)** .</span><span class="sxs-lookup"><span data-stu-id="16b6a-395">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="16b6a-396">Wählen Sie **Metrikwarnung hinzufügen (klassisch)** .</span><span class="sxs-lookup"><span data-stu-id="16b6a-396">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="16b6a-397">Konfigurieren Sie unter **Regel hinzufügen** folgende Einstellungen:</span><span class="sxs-lookup"><span data-stu-id="16b6a-397">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="16b6a-398">Geben Sie unter **Name** den Namen **Burst into Azure Cloud** ein.</span><span class="sxs-lookup"><span data-stu-id="16b6a-398">For **Name**, enter **Burst into Azure Cloud**.</span></span>
   - <span data-ttu-id="16b6a-399">Das Angeben einer **Beschreibung** ist optional.</span><span class="sxs-lookup"><span data-stu-id="16b6a-399">A **Description** is optional.</span></span>
   - <span data-ttu-id="16b6a-400">Wählen Sie unter **Quelle** > **Warnung bei** die Option **Metriken**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-400">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="16b6a-401">Wählen Sie unter **Kriterien** Ihr Abonnement, die Ressourcengruppe für Ihr Traffic Manager-Profil und den Namen des Traffic Manager-Profils für die Ressource aus.</span><span class="sxs-lookup"><span data-stu-id="16b6a-401">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="16b6a-402">Wählen Sie unter **Metrik** die Option **Anforderungsrate**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-402">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="16b6a-403">Wählen Sie unter **Bedingung** die Option **Größer als**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-403">For **Condition**, select **Greater than**.</span></span>
6. <span data-ttu-id="16b6a-404">Geben Sie unter **Schwellenwert** den Wert **2** ein.</span><span class="sxs-lookup"><span data-stu-id="16b6a-404">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="16b6a-405">Wählen Sie unter **Zeitraum** die Option **In den letzten 5 Minuten**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-405">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="16b6a-406">Unter **Benachrichtigen über**:</span><span class="sxs-lookup"><span data-stu-id="16b6a-406">Under **Notify via**:</span></span>
   - <span data-ttu-id="16b6a-407">Aktivieren Sie das Kontrollkästchen **E-Mail-Besitzer, Mitwirkende und Leser**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-407">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="16b6a-408">Geben Sie unter **Zusätzliche Administrator-E-Mail-Adressen** Ihre E-Mail-Adresse ein.</span><span class="sxs-lookup"><span data-stu-id="16b6a-408">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="16b6a-409">Wählen Sie in der Menüleiste die Option **Speichern**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-409">On the menu bar, select **Save**.</span></span>

### <a name="create-the-scale-in-alert"></a><span data-ttu-id="16b6a-410">Erstellen der Warnung für das Abskalieren</span><span class="sxs-lookup"><span data-stu-id="16b6a-410">Create the scale-in alert</span></span>

1. <span data-ttu-id="16b6a-411">Wählen Sie unter **KONFIGURIEREN** die Option **Warnungen (klassisch)** .</span><span class="sxs-lookup"><span data-stu-id="16b6a-411">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="16b6a-412">Wählen Sie **Metrikwarnung hinzufügen (klassisch)** .</span><span class="sxs-lookup"><span data-stu-id="16b6a-412">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="16b6a-413">Konfigurieren Sie unter **Regel hinzufügen** folgende Einstellungen:</span><span class="sxs-lookup"><span data-stu-id="16b6a-413">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="16b6a-414">Geben Sie unter **Name** den Namen **Scale back into Azure Stack Hub** ein.</span><span class="sxs-lookup"><span data-stu-id="16b6a-414">For **Name**, enter **Scale back into Azure Stack Hub**.</span></span>
   - <span data-ttu-id="16b6a-415">Das Angeben einer **Beschreibung** ist optional.</span><span class="sxs-lookup"><span data-stu-id="16b6a-415">A **Description** is optional.</span></span>
   - <span data-ttu-id="16b6a-416">Wählen Sie unter **Quelle** > **Warnung bei** die Option **Metriken**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-416">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="16b6a-417">Wählen Sie unter **Kriterien** Ihr Abonnement, die Ressourcengruppe für Ihr Traffic Manager-Profil und den Namen des Traffic Manager-Profils für die Ressource aus.</span><span class="sxs-lookup"><span data-stu-id="16b6a-417">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="16b6a-418">Wählen Sie unter **Metrik** die Option **Anforderungsrate**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-418">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="16b6a-419">Wählen Sie unter **Bedingung** die Option **Kleiner als**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-419">For **Condition**, select **Less than**.</span></span>
6. <span data-ttu-id="16b6a-420">Geben Sie unter **Schwellenwert** den Wert **2** ein.</span><span class="sxs-lookup"><span data-stu-id="16b6a-420">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="16b6a-421">Wählen Sie unter **Zeitraum** die Option **In den letzten 5 Minuten**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-421">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="16b6a-422">Unter **Benachrichtigen über**:</span><span class="sxs-lookup"><span data-stu-id="16b6a-422">Under **Notify via**:</span></span>
   - <span data-ttu-id="16b6a-423">Aktivieren Sie das Kontrollkästchen **E-Mail-Besitzer, Mitwirkende und Leser**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-423">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="16b6a-424">Geben Sie unter **Zusätzliche Administrator-E-Mail-Adressen** Ihre E-Mail-Adresse ein.</span><span class="sxs-lookup"><span data-stu-id="16b6a-424">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="16b6a-425">Wählen Sie in der Menüleiste die Option **Speichern**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-425">On the menu bar, select **Save**.</span></span>

<span data-ttu-id="16b6a-426">Im folgenden Screenshot sind die Warnungen für das Auf- und Abskalieren dargestellt.</span><span class="sxs-lookup"><span data-stu-id="16b6a-426">The following screenshot shows the alerts for scale-out and scale-in.</span></span>

   ![Application Insights-Warnungen (klassisch)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a><span data-ttu-id="16b6a-428">Umleiten von Datenverkehr zwischen Azure und Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="16b6a-428">Redirect traffic between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="16b6a-429">Sie können das manuelle oder automatische Umschalten zwischen Azure und Azure Stack Hub für Ihren Web-App-Datenverkehr konfigurieren.</span><span class="sxs-lookup"><span data-stu-id="16b6a-429">You can configure manual or automatic switching of your web app traffic between Azure and Azure Stack Hub.</span></span>

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="16b6a-430">Konfigurieren des manuellen Umschaltens zwischen Azure und Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="16b6a-430">Configure manual switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="16b6a-431">Wenn Ihre Website die von Ihnen konfigurierten Schwellenwerte erreicht, erhalten Sie eine Warnung.</span><span class="sxs-lookup"><span data-stu-id="16b6a-431">When your web site reaches the thresholds that you configure, you'll receive an alert.</span></span> <span data-ttu-id="16b6a-432">Führen Sie die folgenden Schritte aus, um Datenverkehr manuell an Azure umzuleiten.</span><span class="sxs-lookup"><span data-stu-id="16b6a-432">Use the following steps to manually redirect traffic to Azure.</span></span>

1. <span data-ttu-id="16b6a-433">Wählen Sie im Azure-Portal Ihr Traffic Manager-Profil aus.</span><span class="sxs-lookup"><span data-stu-id="16b6a-433">In the Azure portal, select your Traffic Manager profile.</span></span>

    ![Traffic Manager-Endpunkte im Azure-Portal](media/solution-deployment-guide-hybrid/image20.png)

2. <span data-ttu-id="16b6a-435">Wählen Sie **Endpunkte**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-435">Select **Endpoints**.</span></span>
3. <span data-ttu-id="16b6a-436">Wählen Sie die Option **Azure-Endpunkt**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-436">Select the **Azure endpoint**.</span></span>
4. <span data-ttu-id="16b6a-437">Wählen Sie unter **Status** die Option **Aktiviert** und dann **Speichern**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-437">Under **Status**, select **Enabled**, and then select **Save**.</span></span>

    ![Aktivieren des Azure-Endpunkts im Azure-Portal](media/solution-deployment-guide-hybrid/image23.png)

5. <span data-ttu-id="16b6a-439">Wählen Sie unter **Endpunkte** für das Traffic Manager-Profil die Option **Externer Endpunkt**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-439">On **Endpoints** for the Traffic Manager profile, select **External endpoint**.</span></span>
6. <span data-ttu-id="16b6a-440">Wählen Sie unter **Status** die Option **Deaktiviert** und dann **Speichern**.</span><span class="sxs-lookup"><span data-stu-id="16b6a-440">Under **Status**, select **Disabled**, and then select **Save**.</span></span>

    ![Deaktivieren des Azure Stack Hub-Endpunkts im Azure-Portal](media/solution-deployment-guide-hybrid/image24.png)

<span data-ttu-id="16b6a-442">Nachdem die Endpunkte konfiguriert wurden, fließt der App-Datenverkehr nicht mehr zur Azure Stack Hub-Web-App, sondern zu Ihrer horizontal skalierten Azure-Web-App.</span><span class="sxs-lookup"><span data-stu-id="16b6a-442">After the endpoints are configured, app traffic goes to your Azure scale-out web app instead of the Azure Stack Hub web app.</span></span>

 ![Für Azure-Web-App-Datenverkehr geänderte Endpunkte](media/solution-deployment-guide-hybrid/image25.png)

<span data-ttu-id="16b6a-444">Verwenden Sie zum Umkehren des Datenverkehrsflusses zurück zu Azure Stack Hub die vorherigen Schritte, um Folgendes durchzuführen:</span><span class="sxs-lookup"><span data-stu-id="16b6a-444">To reverse the flow back to Azure Stack Hub, use the previous steps to:</span></span>

- <span data-ttu-id="16b6a-445">Aktivieren des Azure Stack Hub-Endpunkts.</span><span class="sxs-lookup"><span data-stu-id="16b6a-445">Enable the Azure Stack Hub endpoint.</span></span>
- <span data-ttu-id="16b6a-446">Deaktivieren des Azure-Endpunkts.</span><span class="sxs-lookup"><span data-stu-id="16b6a-446">Disable the Azure endpoint.</span></span>

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="16b6a-447">Konfigurieren des automatischen Umschaltens zwischen Azure und Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="16b6a-447">Configure automatic switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="16b6a-448">Sie können die Application Insights-Überwachung auch nutzen, wenn Ihre App in einer [serverlosen](https://azure.microsoft.com/overview/serverless-computing/) Umgebung ausgeführt wird, die von Azure Functions bereitgestellt wird.</span><span class="sxs-lookup"><span data-stu-id="16b6a-448">You can also use Application Insights monitoring if your app runs in a [serverless](https://azure.microsoft.com/overview/serverless-computing/) environment provided by Azure Functions.</span></span>

<span data-ttu-id="16b6a-449">In diesem Szenario können Sie Application Insights so konfigurieren, dass ein Webhook zum Aufrufen einer Funktions-App verwendet wird.</span><span class="sxs-lookup"><span data-stu-id="16b6a-449">In this scenario, you can configure Application Insights to use a webhook that calls a function app.</span></span> <span data-ttu-id="16b6a-450">Diese App aktiviert bzw. deaktiviert automatisch einen Endpunkt als Reaktion auf eine Warnung.</span><span class="sxs-lookup"><span data-stu-id="16b6a-450">This app automatically enables or disables an endpoint in response to an alert.</span></span>

<span data-ttu-id="16b6a-451">Verwenden Sie die folgenden Schritte als Anleitung zum Konfigurieren der automatischen Umschaltung von Datenverkehr.</span><span class="sxs-lookup"><span data-stu-id="16b6a-451">Use the following steps as a guide to configure automatic traffic switching.</span></span>

1. <span data-ttu-id="16b6a-452">Erstellen Sie eine Azure-Funktions-App.</span><span class="sxs-lookup"><span data-stu-id="16b6a-452">Create an Azure Function app.</span></span>
2. <span data-ttu-id="16b6a-453">Erstellen Sie eine per HTTP ausgelöste Funktion.</span><span class="sxs-lookup"><span data-stu-id="16b6a-453">Create an HTTP-triggered function.</span></span>
3. <span data-ttu-id="16b6a-454">Importieren Sie die Azure SDKs für Resource Manager, Web-Apps und Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="16b6a-454">Import the Azure SDKs for Resource Manager, Web Apps, and Traffic Manager.</span></span>
4. <span data-ttu-id="16b6a-455">Entwickeln Sie Code für folgende Zwecke:</span><span class="sxs-lookup"><span data-stu-id="16b6a-455">Develop code to:</span></span>

   - <span data-ttu-id="16b6a-456">Durchführen der Authentifizierung für Ihr Azure-Abonnement</span><span class="sxs-lookup"><span data-stu-id="16b6a-456">Authenticate to your Azure subscription.</span></span>
   - <span data-ttu-id="16b6a-457">Verwenden eines Parameters, mit dem die Traffic Manager-Endpunkte zum Weiterleiten von Datenverkehr an Azure oder Azure Stack Hub umgeschaltet werden</span><span class="sxs-lookup"><span data-stu-id="16b6a-457">Use a parameter that toggles the Traffic Manager endpoints to direct traffic to Azure or Azure Stack Hub.</span></span>

5. <span data-ttu-id="16b6a-458">Speichern Sie Ihren Code, und fügen Sie die Funktions-App-URL mit den entsprechenden Parametern dem Abschnitt **Webhook** der Einstellungen für die Application Insights-Warnungsregeln hinzu.</span><span class="sxs-lookup"><span data-stu-id="16b6a-458">Save your code and add the function app's URL with the appropriate parameters to the **Webhook** section of the Application Insights alert rule settings.</span></span>
6. <span data-ttu-id="16b6a-459">Datenverkehr wird automatisch umgeleitet, wenn eine Application Insights-Warnung ausgelöst wird.</span><span class="sxs-lookup"><span data-stu-id="16b6a-459">Traffic is automatically redirected when an Application Insights alert fires.</span></span>

## <a name="next-steps"></a><span data-ttu-id="16b6a-460">Nächste Schritte</span><span class="sxs-lookup"><span data-stu-id="16b6a-460">Next steps</span></span>

- <span data-ttu-id="16b6a-461">Weitere Informationen zu Azure-Cloudmustern finden Sie unter [Cloudentwurfsmuster](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="16b6a-461">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
