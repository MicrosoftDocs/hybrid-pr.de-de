---
title: Konfigurieren einer Hybrid Cloud-Identität für Azure- und Azure Stack Hub-Apps
description: Erfahren Sie, wie Sie eine Hybrid Cloud-Identität für Azure- und Azure Stack Hub-Apps konfigurieren.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 1b3c683dd3e4a68413f83fd3cc129d6e6f594e1b
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910519"
---
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a>Konfigurieren einer Hybrid Cloud-Identität für Azure- und Azure Stack Hub-Apps

Erfahren Sie, wie Sie eine Hybrid Cloud-Identität für Ihre Azure- und Azure Stack Hub-Apps konfigurieren.

Der Zugriff auf Ihre Apps in globalen Azure- und Azure Stack Hub-Instanzen kann auf zwei Arten gewährt werden.

 * Wenn Azure Stack Hub über eine ständige Internetverbindung verfügt, können Sie Azure Active Directory (Azure AD) verwenden.
 * Wenn Azure Stack Hub über keine Internetverbindung verfügt, können Sie Azure Directory-Verbunddienste (AD FS) verwenden.

Dienstprinzipale werden verwendet, um den Zugriff auf Ihre Azure Stack Hub-Apps zur Bereitstellung oder Konfiguration über Azure Resource Manager in Azure Stack Hub zu gewähren.

In dieser Lösung erstellen Sie eine Beispielumgebung, die Folgendes ermöglicht:

> [!div class="checklist"]
> - Einrichten einer Hybrididentität in globalen Azure- und Azure Stack Hub-Instanzen
> - Abrufen eines Tokens für den Zugriff auf die Azure Stack Hub-API

Sie benötigen Azure Stack Hub-Bedienerberechtigungen für die Schritte in dieser Lösung.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub ist eine Erweiterung von Azure. Mit Azure Stack Hub holen Sie sich die Agilität und Innovation von Cloud Computing in Ihre lokale Umgebung, indem Sie die einzige Hybrid Cloud aktivieren, mit der Sie Hybrid-Apps überall entwickeln und bereitstellen können.  
> 
> Im Artikel [Überlegungen zum Entwurf von Hybrid-Apps](overview-app-design-considerations.md) werden die wichtigen Aspekte in Bezug auf die Softwarequalität (Platzierung, Skalierbarkeit, Verfügbarkeit, Resilienz, Verwaltbarkeit und Sicherheit) beschrieben, die für das Entwerfen, Bereitstellen und Betreiben von Hybrid-Apps erforderlich sind. Die Überlegungen zum Entwurf dienen als Hilfe beim Optimieren des Designs von Hybrid-Apps, um für Produktionsumgebungen das Auftreten von Problemen zu minimieren.

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a>Erstellen eines Dienstprinzipals für Azure AD über das Portal

Wenn Sie Azure Stack Hub mit Azure AD als Identitätsspeicher bereitgestellt haben, können Sie Dienstprinzipale genauso wie für Azure erstellen. Im Artikel [Verwenden einer App-Identität für den Ressourcenzugriff](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity) erfahren Sie, wie die Schritte über das Portal ausgeführt werden. Vergewissern Sie sich vorher, dass Sie über die [erforderlichen Azure AD-Berechtigungen](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) verfügen.

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a>Erstellen eines Dienstprinzipals für AD FS mithilfe von PowerShell

Wenn Sie Azure Stack Hub mit AD FS bereitgestellt haben, können Sie PowerShell verwenden, um einen Dienstprinzipal zu erstellen, eine Rolle für den Zugriff zuzuweisen und die Anmeldung über PowerShell mit dieser Identität durchzuführen. Im Artikel [Verwenden einer App-Identität für den Ressourcenzugriff](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity) erfahren Sie, wie Sie die erforderlichen Schritte mithilfe von PowerShell ausführen.

## <a name="using-the-azure-stack-hub-api"></a>Verwenden der Azure Stack Hub-API

In der Lösung zur [Azure Stack Hub-API](/azure-stack/user/azure-stack-rest-api-use.md) erfahren Sie Schritt für Schritt, wie Sie ein Token für den Zugriff auf die Azure Stack Hub-API abrufen.

## <a name="connect-to-azure-stack-hub-using-powershell"></a>Herstellen einer Verbindung mit Azure Stack Hub mithilfe von PowerShell

In der Schnellstartanleitung zum [Einrichten von PowerShell in Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install.md) sind die Schritte erläutert, die zum Installieren von Azure PowerShell und zum Herstellen einer Verbindung mit der Azure Stack Hub-Installation ausgeführt werden müssen.

### <a name="prerequisites"></a>Voraussetzungen

Sie benötigen eine mit Azure AD verbundene Azure Stack Hub-Installation mit einem Abonnement, auf das Sie zugreifen können. Falls Sie keine Azure Stack Hub-Installation haben, sehen Sie sich die Anleitung zum Einrichten eines [Azure Stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install.md) an.

#### <a name="connect-to-azure-stack-hub-using-code"></a>Herstellen einer Verbindung mit Azure Stack Hub mithilfe von Code

Wenn Sie mithilfe von Code eine Verbindung mit Azure Stack Hub herstellen möchten, verwenden Sie die Endpunkte-API von Azure Resource Manager, um die Authentifizierung und die Graph-Endpunkte für Ihre Azure Stack Hub-Installation abzurufen. Führen Sie anschließend die Authentifizierung mithilfe von REST-Anforderungen durch. Eine Beispielclientanwendung finden Sie auf [GitHub](https://github.com/shriramnat/HybridARMApplication).

>[!Note]
>Sofern das Azure SDK für Ihre bevorzugte Sprache keine Azure-API-Profile unterstützt, funktioniert das SDK möglicherweise nicht mit Azure Stack Hub. Weitere Informationen zu Azure-API-Profilen finden Sie im Artikel [Verwalten von API-Versionsprofilen in Azure Stack](/azure-stack/user/azure-stack-version-profiles.md).

## <a name="next-steps"></a>Nächste Schritte

- Weitere Informationen zur Behandlung der Identität in Azure Stack Hub finden Sie unter [Identitätsarchitektur für Azure Stack Hub](/azure-stack/operator/azure-stack-identity-architecture.md).
- Weitere Informationen zu Azure-Cloudmustern finden Sie unter [Cloudentwurfsmuster](https://docs.microsoft.com/azure/architecture/patterns).
