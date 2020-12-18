---
title: Bereitstellen einer SQL Server 2016-Verfügbarkeitsgruppe in Azure und Azure Stack Hub
description: Es wird beschrieben, wie Sie eine SQL Server 2016-Verfügbarkeitsgruppe in Azure und Azure Stack Hub bereitstellen.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 0f857515a44ece7f967ade3dee8f493481709851
ms.sourcegitcommit: c890f2c5e5e5f9f93c921f02dd1a6ca5026d5289
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 12/10/2020
ms.locfileid: "97091801"
---
# <a name="deploy-a-sql-server-2016-availability-group-across-two-azure-stack-hub-environments"></a>Bereitstellen einer SQL Server 2016-Verfügbarkeitsgruppe in zwei Azure Stack Hub-Umgebungen

In diesem Artikel werden Sie schrittweise durch die automatisierte Bereitstellung eines einfachen SQL Server 2016 Enterprise-Hochverfügbarkeitsclusters (HA) mit einem asynchronen Standort für die Notfallwiederherstellung (DR) in zwei Azure Stack Hub-Umgebungen geführt. Weitere Informationen zu SQL Server 2016 und zur Hochverfügbarkeit finden Sie unter [Always On-Verfügbarkeitsgruppen: eine Lösung mit Hochverfügbarkeit und Notfallwiederherstellung](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).

In dieser Lösung erstellen Sie eine Beispielumgebung, die Folgendes ermöglicht:

> [!div class="checklist"]
> - Orchestrieren einer Bereitstellung in zwei Azure Stack Hub-Instanzen
> - Verwenden von Docker zur Minimierung von Abhängigkeitsproblemen mit Azure-API-Profilen
> - Bereitstellen eines einfachen SQL Server 2016 Enterprise-Hochverfügbarkeitsclusters mit einem Standort für die Notfallwiederherstellung

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub ist eine Erweiterung von Azure. Mit Azure Stack Hub holen Sie sich die Agilität und Innovation von Cloud Computing in Ihre lokale Umgebung, indem Sie die einzige Hybrid Cloud aktivieren, mit der Sie Hybrid-Apps überall entwickeln und bereitstellen können.  
> 
> Im Artikel [Überlegungen zum Entwurf von Hybrid-Apps](overview-app-design-considerations.md) werden die wichtigen Aspekte in Bezug auf die Softwarequalität (Platzierung, Skalierbarkeit, Verfügbarkeit, Resilienz, Verwaltbarkeit und Sicherheit) beschrieben, die für das Entwerfen, Bereitstellen und Betreiben von Hybrid-Apps erforderlich sind. Die Überlegungen zum Entwurf dienen als Hilfe beim Optimieren des Designs von Hybrid-Apps, um für Produktionsumgebungen das Auftreten von Problemen zu minimieren.

## <a name="architecture-for-sql-server-2016"></a>Architektur für SQLServer 2016

![SQL Server 2016 SQL HA Azure Stack Hub](media/solution-deployment-guide-sql-ha/image1.png)

## <a name="prerequisites-for-sql-server-2016"></a>Voraussetzungen für SQL Server 2016

- Zwei verbundene integrierte Azure Stack Hub-Systeme (Azure Stack Hub). Diese Bereitstellung funktioniert nicht für das Azure Stack Development Kit (ASDK). Weitere Informationen zu Azure Stack Hub finden Sie in der [Übersicht zu Azure Stack](https://azure.microsoft.com/overview/azure-stack/).
- Ein Mandantenabonnement für jede Azure Stack-Instanz.
  - **Notieren Sie sich jede Abonnement-ID und den Azure Resource Manager-Endpunkt für jede Azure Stack Hub-Instanz.**
- Ein Dienstprinzipal für Azure Active Directory (Azure AD), der über Berechtigungen für das Mandantenabonnement für jede Azure Stack Hub-Instanz verfügt. Möglicherweise müssen Sie zwei Dienstprinzipale erstellen, wenn die Azure Stack Hub-Instanzen in unterschiedlichen Azure AD-Mandanten bereitgestellt sind. Informationen zum Erstellen eines Dienstprinzipals für Azure Stack Hub finden Sie unter [Verwenden einer App-Identität für den Zugriff auf Azure Stack Hub-Ressourcen](/azure-stack/user/azure-stack-create-service-principals).
  - **Notieren Sie sich die Anwendungs-ID, den geheimen Clientschlüssel und den Mandantennamen (xxxxx.onmicrosoft.com) jedes Dienstprinzipals.**
- SQL Server 2016 Enterprise syndiziert in jeden Marketplace von Azure Stack Hub. Weitere Informationen zur Marketplace-Syndikation finden Sie unter [Herunterladen von Marketplace-Elementen in Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).
    **Stellen Sie sicher, dass Ihre Organisation über die geeigneten SQL-Lizenzen verfügt.**
- [Docker für Windows](https://docs.docker.com/docker-for-windows/), auf Ihrem lokalen Computer installiert.

## <a name="get-the-docker-image"></a>Abrufen des Docker-Images

Docker-Images für jede Bereitstellung beseitigen Abhängigkeitsprobleme zwischen verschiedenen Versionen von Azure PowerShell.

1. Stellen Sie sicher, dass Docker für Windows Windows-Container verwendet.
2. Führen Sie das folgende Skript an einer Eingabeaufforderung mit erhöhten Rechten aus, um den Docker-Container mit den Bereitstellungsskripts abzurufen.

    ```powershell  
    docker pull intelligentedge/sqlserver2016-hadr:1.0.0
    ```

## <a name="deploy-the-availability-group"></a>Bereitstellen der Verfügbarkeitsgruppe

1. Sobald das Containerimage erfolgreich abgerufen wurde, starten Sie das Image.

      ```powershell  
      docker run -it intelligentedge/sqlserver2016-hadr:1.0.0 powershell
      ```

2. Nachdem der Container gestartet wurde, erhalten Sie im Container ein PowerShell-Terminal mit erhöhten Rechten. Wechseln Sie die Verzeichnisse, um zu dem Bereitstellungsskript zu gelangen.

      ```powershell  
      cd .\SQLHADRDemo\
      ```

3. Führen Sie die Bereitstellung aus. Geben Sie die Anmeldeinformationen und Ressourcennamen an, wenn erforderlich. Hochverfügbarkeit bezieht sich auf die Azure Stack Hub-Instanz, in der der hoch verfügbare Cluster bereitgestellt wird. Notfallwiederherstellung bezieht sich auf die Azure Stack Hub-Instanz, in der der Notfallwiederherstellungscluster bereitgestellt wird.

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

4. Geben Sie `Y` ein, um die Installation des NuGet-Anbieters zuzulassen, wodurch die Installation der „2018-03-01-Hybrid“-Module des API-Profils ausgelöst wird.

5. Warten Sie, bis die Ressourcenbereitstellung abgeschlossen ist.

6. Nach Abschluss der Bereitstellung der DR-Ressource beenden Sie den Container.

      ```powershell
      exit
      ```

7. Überprüfen Sie die Bereitstellung, indem Sie sich die Ressourcen im Portal jeder Azure Stack Hub-Instanz ansehen. Stellen Sie eine Verbindung mit einer SQL-Instanz in der Hochverfügbarkeitsumgebung her, und untersuchen Sie die Verfügbarkeitsgruppe über SQL Server Management Studio (SSMS).

    ![SQL Server 2016 SQL HA](media/solution-deployment-guide-sql-ha/image2.png)

## <a name="next-steps"></a>Nächste Schritte

- Verwenden Sie SQL Server Management Studio, um das Failover für den Cluster manuell auszuführen. Weitere Informationen finden Sie unter [Ausführen eines erzwungenen manuellen Failovers einer Always On-Verfügbarkeitsgruppe (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017).
- Informieren Sie sich über Hybrid Cloud-Apps. Sehen Sie sich die Informationen zu [Hybrid Cloud-Lösungen](/azure-stack/user/) an.
- Ändern Sie den Code dieses Beispiels auf [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).
