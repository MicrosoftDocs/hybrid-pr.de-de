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
# <a name="deploy-a-highly-available-mongodb-solution-across-two-azure-stack-hub-environments"></a>Bereitstellen einer MongoDB-Hochverfügbarkeitslösung in zwei Azure Stack Hub-Umgebungen

In diesem Artikel werden Sie schrittweise durch die automatisierte Bereitstellung eines einfachen MongoDB-Hochverfügbarkeitsclusters (HA) mit einem Standort für die Notfallwiederherstellung (DR) in zwei Azure Stack Hub-Umgebungen geführt. Weitere Informationen zu MongoDB und zur Hochverfügbarkeit finden Sie unter [Replikatgruppenmitglieder](https://docs.mongodb.com/manual/core/replica-set-members/).

In dieser Lösung erstellen Sie eine Beispielumgebung, die Folgendes ermöglicht:

> [!div class="checklist"]
> - Orchestrieren einer Bereitstellung in zwei Azure Stack Hub-Instanzen
> - Verwenden von Docker zur Minimierung von Abhängigkeitsproblemen mit Azure-API-Profilen
> - Bereitstellen eines einfachen MongoDB-Hochverfügbarkeitsclusters mit einem Standort für die Notfallwiederherstellung

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub ist eine Erweiterung von Azure. Mit Azure Stack Hub holen Sie sich die Agilität und Innovation von Cloud Computing in Ihre lokale Umgebung, indem Sie die einzige Hybrid Cloud aktivieren, mit der Sie Hybrid-Apps überall entwickeln und bereitstellen können.  
> 
> Im Artikel [Überlegungen zum Entwurf von Hybrid-Apps](overview-app-design-considerations.md) werden die wichtigen Aspekte in Bezug auf die Softwarequalität (Platzierung, Skalierbarkeit, Verfügbarkeit, Resilienz, Verwaltbarkeit und Sicherheit) beschrieben, die für das Entwerfen, Bereitstellen und Betreiben von Hybrid-Apps erforderlich sind. Die Überlegungen zum Entwurf dienen als Hilfe beim Optimieren des Designs von Hybrid-Apps, um für Produktionsumgebungen das Auftreten von Problemen zu minimieren.

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a>Architektur für MongoDB mit Azure Stack Hub

![Hoch verfügbare MongoDB-Architektur in Azure Stack Hub](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a>Voraussetzungen für MongoDB mit Azure Stack Hub

- Zwei verbundene integrierte Azure Stack Hub-Systeme (Azure Stack Hub). Diese Bereitstellung funktioniert nicht für das Azure Stack Development Kit (ASDK). Weitere Informationen zu Azure Stack finden Sie unter [Was ist Azure Stack Hub?](https://azure.microsoft.com/products/azure-stack/hub/).
  - Ein Mandantenabonnement für jede Azure Stack-Instanz. 
  - **Notieren Sie sich jede Abonnement-ID und den Azure Resource Manager-Endpunkt für jede Azure Stack Hub-Instanz.**
- Ein Dienstprinzipal für Azure Active Directory (Azure AD), der über Berechtigungen für das Mandantenabonnement für jede Azure Stack Hub-Instanz verfügt. Möglicherweise müssen Sie zwei Dienstprinzipale erstellen, wenn die Azure Stack Hub-Instanzen in unterschiedlichen Azure AD-Mandanten bereitgestellt sind. Informationen zur Erstellung eines Dienstprinzipals für Azure Stack Hub finden Sie unter [Verwenden einer App-Identität für den Zugriff auf Azure Stack Hub-Ressourcen](/azure-stack/user/azure-stack-create-service-principals).
  - **Notieren Sie sich die Anwendungs-ID, den geheimen Clientschlüssel und den Mandantennamen (xxxxx.onmicrosoft.com) jedes Dienstprinzipals.**
- Ubuntu 16.04 syndiziert in jeden Marketplace von Azure Stack Hub. Weitere Informationen zur Marketplace-Syndikation finden Sie unter [Herunterladen von Marketplace-Elementen in Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).
- [Docker für Windows](https://docs.docker.com/docker-for-windows/), auf Ihrem lokalen Computer installiert.

## <a name="get-the-docker-image"></a>Abrufen des Docker-Images

Docker-Images für jede Bereitstellung beseitigen Abhängigkeitsprobleme zwischen verschiedenen Versionen von Azure PowerShell.

1. Stellen Sie sicher, dass Docker für Windows Windows-Container verwendet.
2. Führen Sie den folgenden Befehl an einer Eingabeaufforderung mit erhöhten Rechten aus, um den Docker-Container mit den Bereitstellungsskripts abzurufen.

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a>Bereitstellen der Cluster

1. Sobald das Containerimage erfolgreich abgerufen wurde, starten Sie das Image.

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. Nachdem der Container gestartet wurde, erhalten Sie im Container ein PowerShell-Terminal mit erhöhten Rechten. Wechseln Sie die Verzeichnisse, um zu dem Bereitstellungsskript zu gelangen.

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. Führen Sie die Bereitstellung aus. Geben Sie die Anmeldeinformationen und Ressourcennamen an, wenn erforderlich. Hochverfügbarkeit bezieht sich auf die Azure Stack Hub-Instanz, in der der hoch verfügbare Cluster bereitgestellt wird. Notfallwiederherstellung bezieht sich auf die Azure Stack Hub-Instanz, in der der Notfallwiederherstellungscluster bereitgestellt wird.

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

4. Geben Sie `Y` ein, um die Installation des NuGet-Anbieters zuzulassen, wodurch die Installation der „2018-03-01-Hybrid“-Module des API-Profils ausgelöst wird.

5. Die HA-Ressourcen werden zuerst bereitgestellt. Überwachen Sie die Bereitstellung, und warten Sie, bis sie abgeschlossen ist. Sobald die Meldung mit dem Hinweis angezeigt wird, dass die HA-Bereitstellung abgeschlossen ist, können Sie im Portal der Azure Stack Hub-Instanz mit HA die bereitgestellten Ressourcen überprüfen.

6. Fahren Sie mit der Bereitstellung von DR-Ressourcen fort, und entscheiden Sie, ob Sie für die Azure Stack Hub-Instanz mit DR eine Jumpbox für die Interaktion mit dem Cluster aktivieren möchten.

7. Warten Sie, bis die DR-Ressourcenbereitstellung abgeschlossen ist.

8. Beenden Sie den Container, nachdem die Bereitstellung der DR-Ressource abgeschlossen wurde.

  ```powershell
  exit
  ```

## <a name="next-steps"></a>Nächste Schritte

- Wenn Sie die Jumpbox-VM in der Azure Stack Hub-Instanz mit DR aktiviert haben, können Sie eine Verbindung über SSH herstellen und mit dem MongoDB-Cluster interagieren, indem Sie die Mongo-CLI installieren. Weitere Informationen zur Interaktion mit MongoDB finden Sie unter [Die Mongo-Shell](https://docs.mongodb.com/manual/mongo/).
- Weitere Informationen zu Hybrid Cloud-Apps finden Sie unter [Hybrid Cloud-Lösungen](/azure-stack/user/).
- Ändern Sie den Code dieses Beispiels auf [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).
