---
title: Konfigurieren der Hybrid Cloud-Konnektivität in Azure und Azure Stack Hub
description: Erfahren Sie, wie Sie die Hybrid Cloud-Konnektivität mithilfe von Azure und Azure Stack Hub konfigurieren.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 16c5d7820e8c865a9f88cb00da5cc7c854379414
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477285"
---
# <a name="configure-hybrid-cloud-connectivity-using-azure-and-azure-stack-hub"></a>Konfigurieren der Hybrid Cloud-Konnektivität mithilfe von Azure und Azure Stack Hub

Mithilfe des Hybridkonnektivitätsmusters können Sie auf Ressourcen mit Sicherheit in globalen Azure- und Azure Stack Hub-Instanzen zugreifen.

In dieser Lösung erstellen Sie eine Beispielumgebung, die Folgendes ermöglicht:

> [!div class="checklist"]
> - Lokale Aufbewahrung von Daten zur Erfüllung datenschutzbezogener und rechtlicher Anforderungen bei Erhaltung des Zugriffs auf globale Azure-Ressourcen.
> - Verwendung eines Legacysystems sowie einer über die Cloud skalierten App-Bereitstellung und entsprechender Ressourcen in globalen Azure-Instanzen

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub ist eine Erweiterung von Azure. Mit Azure Stack Hub holen Sie sich die Agilität und Innovation von Cloud Computing in Ihre lokale Umgebung. Sie erhalten die einzige Hybrid Cloud, mit der Sie Hybrid-Apps überall entwickeln und bereitstellen können.  
> 
> Im Artikel [Überlegungen zum Entwurf von Hybrid-Apps](overview-app-design-considerations.md) werden die wichtigen Aspekte in Bezug auf die Softwarequalität (Platzierung, Skalierbarkeit, Verfügbarkeit, Resilienz, Verwaltbarkeit und Sicherheit) beschrieben, die für das Entwerfen, Bereitstellen und Betreiben von Hybrid-Apps erforderlich sind. Die Überlegungen zum Entwurf dienen als Hilfe beim Optimieren des Designs von Hybrid-Apps, um für Produktionsumgebungen das Auftreten von Problemen zu minimieren.

## <a name="prerequisites"></a>Voraussetzungen

Es sind einige Komponenten erforderlich, um eine Hybrid-Verbindungsbereitstellung zu erstellen. Da die Vorbereitung für einige dieser Komponenten länger dauert, sollten Sie entsprechend planen.

### <a name="azure"></a>Azure

- Wenn Sie kein Azure-Abonnement besitzen, können Sie ein [kostenloses Konto](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) erstellen, bevor Sie beginnen.
- Erstellen Sie eine [Web-App](/vsts/build-release/apps/cd/azure/aspnet-core-to-azure-webapp?tabs=vsts&view=vsts) in Azure. Notieren Sie sich die Web-App-URL, da Sie sie in der Lösung benötigen.

### <a name="azure-stack-hub"></a>Azure Stack Hub

Ein Azure-OEM/-Hardwarepartner kann eine Azure Stack Hub-Produktionsumgebung bereitstellen. Ein Azure Stack Development Kit (ASDK) kann von allen Benutzern bereitgestellt werden.

- Verwenden Sie Ihre Produktionsinstanz von Azure Stack Hub, oder stellen Sie das ASDK bereit.
   >[!Note]
   >Da die Bereitstellung des ASDK bis zu sieben Stunden dauern kann, sollten Sie entsprechend planen.

- Stellen Sie PaaS-Dienste als [App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) für Azure Stack Hub bereit.
- [Erstellen Sie Pläne und Angebote](/azure-stack/operator/service-plan-offer-subscription-overview.md) in der Azure Stack Hub-Umgebung.
- [Erstellen Sie ein Mandantenabonnement](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) in Ihrer Azure Stack Hub-Umgebung.

### <a name="azure-stack-hub-components"></a>Azure Stack Hub-Komponenten

Ein Azure Stack Hub-Bediener muss App Service bereitstellen, Pläne, Angebote und ein Mandantenabonnement erstellen und das Windows Server 2016-Image hinzufügen. Wenn Sie bereits über diese Komponenten verfügen, sollten Sie vor dem Beginn dieser Lösung sicherstellen, dass diese die Anforderungen erfüllen.

In diesem Lösungsbeispiel wird davon ausgegangen, dass Sie bereits über Grundkenntnisse in Bezug auf Azure und Azure Stack Hub verfügen. Lesen Sie die folgenden Artikel, um vor dem Starten der Lösung weitere Informationen zu erhalten:

- [Einführung in Azure](https://azure.microsoft.com/overview/what-is-azure/)
- [Übersicht über Azure Stack Hub](/azure-stack/operator/azure-stack-overview.md)

### <a name="before-you-begin"></a>Voraussetzungen

Überprüfen Sie, ob Sie die folgenden Kriterien erfüllen, bevor Sie mit der Konfiguration der Hybrid Cloud-Konnektivität beginnen:

- Sie benötigen eine extern zugängliche, öffentliche IPv4-Adresse für Ihr VPN-Gerät. Diese IP-Adresse darf sich nicht hinter einer NAT (Netzwerkadressenübersetzung) befinden.
- Alle Ressourcen werden in derselben Region bzw. an demselben Standort bereitgestellt.

#### <a name="solution-example-values"></a>Beispielwerte für die Lösung

In den Beispielen dieser Lösung werden die folgenden Werte verwendet. Sie können diese Werte zum Erstellen einer Testumgebung verwenden oder zum besseren Verständnis der Beispiele heranziehen. Weitere Informationen zu den VPN Gateway-Einstellungen finden Sie unter [Informationen zu VPN Gateway-Einstellungen](/azure/vpn-gateway/vpn-gateway-about-vpn-gateway-settings).

Verbindungsangaben:

- **VPN-Typ**: routenbasiert
- **Verbindungstyp**: Site-to-Site (IPsec)
- **Gatewaytyp**: VPN
- **Azure-Verbindungsname**: Azure-Gateway-AzureStack-S2SGateway (automatisch vom Portal ausgefüllt)
- **Azure Stack Hub-Verbindungsname**: AzureStack-Gateway-Azure-S2SGateway (automatisch vom Portal ausgefüllt)
- **Gemeinsam verwendeter Schlüssel**: beliebiger, mit VPN-Hardware kompatibler Schlüssel mit entsprechenden Werten auf beiden Seiten der Verbindung
- **Abonnement**: beliebiges bevorzugtes Abonnement
- **Ressourcengruppe**: Test-Infra

Netzwerk- und Subnetz-IP-Adressen:

| Azure-/Azure Stack Hub-Verbindung | Name | Subnet | IP-Adresse |
|---|---|---|---|
| Azure: VNet | ApplicationvNet<br>10.100.102.9/23 | ApplicationSubnet<br>10.100.102.0/24 |  |
|  |  | GatewaySubnet<br>10.100.103.0/24 |  |
| Azure Stack Hub VNET | ApplicationvNet<br>10.100.100.0/23 | ApplicationSubnet <br>10.100.100.0/24 |  |
|  |  | GatewaySubnet <br>10.100101.0/24 |  |
| Azure: Gateway für virtuelle Netzwerke | Azure-Gateway |  |  |
| Azure Stack Hub: Gateway für virtuelle Netzwerke | AzureStack-Gateway |  |  |
| Azure: öffentliche IP-Adresse | Azure-GatewayPublicIP |  | Bei Erstellung bestimmt |
| Azure Stack Hub: öffentliche IP-Adresse | AzureStack-GatewayPublicIP |  | Bei Erstellung bestimmt |
| Azure: lokales Netzwerkgateway | AzureStack-S2SGateway<br>   10.100.100.0/23 |  | Azure Stack Hub: Wert der öffentlichen IP-Adresse |
| Azure Stack Hub: Gateway für lokale Netzwerke | Azure-S2SGateway<br>10.100.102.0/23 |  | Azure: öffentlicher IP-Wert |

## <a name="create-a-virtual-network-in-global-azure-and-azure-stack-hub"></a>Erstellen eines virtuellen Netzwerks in globalen Azure- und Azure Stack Hub-Instanzen

Führen Sie die folgenden Schritte aus, um mit dem Portal ein virtuelles Netzwerk zu erstellen. Sie können diese [Beispielwerte](/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal#values) verwenden, wenn Sie diesen Artikel nur als Lösung nutzen. Falls Sie den Artikel für die Konfiguration einer Produktionsumgebung verwenden, müssen Sie die Beispieleinstellungen durch Ihre eigenen Werte ersetzen.

> [!IMPORTANT]
> Die IP-Adressen in den VNet-Adressräumen von Azure oder Azure Stack Hub dürfen sich nicht überschneiden.

Erstellen Sie in Azure wie folgt ein VNet:

1. Nutzen Sie Ihren Browser, um eine Verbindung mit dem [Azure-Portal](https://portal.azure.com/) herzustellen, und melden Sie sich mit Ihrem Azure-Konto an.
2. Wählen Sie **Ressource erstellen**. Geben Sie im Feld **Marketplace durchsuchen** „Virtuelles Netzwerk“ ein. Wählen Sie in den Ergebnissen **Virtuelles Netzwerk** aus.
3. Wählen Sie in der Liste **Bereitstellungsmodell auswählen** die Option **Resource Manager** und dann **Erstellen**.
4. Konfigurieren Sie unter **Virtuelles Netzwerk erstellen** die VNet-Einstellungen. Die erforderlichen Feldnamen sind durch ein vorangestelltes rotes Sternchen gekennzeichnet.  Wenn Sie einen gültigen Wert eingeben, wird das Sternchen in ein grünes Häkchen geändert.

Erstellen Sie wie folgt in Azure Stack Hub ein VNET:

1. Wiederholen Sie die oben angegebenen Schritte (1 bis 4) im **Mandantenportal** von Azure Stack Hub.

## <a name="add-a-gateway-subnet"></a>Hinzufügen eines Gatewaysubnetzes

Bevor Sie das virtuelle Netzwerk mit einem Gateway verbinden, müssen Sie das Gatewaysubnetz für das virtuelle Netzwerk erstellen, mit dem Sie eine Verbindung herstellen möchten. Für die Gatewaydienste werden die IP-Adressen verwendet, die Sie im Gatewaysubnetz angeben.

Navigieren Sie im [Azure-Portal](https://portal.azure.com/) zum virtuellen Resource Manager-Netzwerk, in dem Sie ein virtuelles Netzwerkgateway erstellen möchten.

1. Wählen Sie das VNet aus, um die Seite **Virtuelles Netzwerk** zu öffnen.
2. Wählen Sie unter **EINSTELLUNGEN** die Option **Subnetze**.
3. Klicken Sie auf der Seite **Subnetze** auf **+Gatewaysubnetz**, um die Seite **Subnetz hinzufügen** zu öffnen.

    ![Gatewaysubnetz hinzufügen](media/solution-deployment-guide-connectivity/image4.png)

4. Als **Name** für das Subnetz wird automatisch der Wert „GatewaySubnet“ eingefügt. Dieser Wert ist erforderlich, damit Azure das Subnetz als Gatewaysubnetz erkennt.
5. Passen Sie die angegebenen Werte für **Adressbereich** an Ihre Konfigurationsanforderungen an, und wählen Sie anschließend **OK** aus.

## <a name="create-a-virtual-network-gateway-in-azure-and-azure-stack"></a>Erstellen eines Gateways für virtuelle Netzwerke in Azure und Azure Stack

Führen Sie die folgenden Schritte aus, um in Azure ein Gateway für virtuelle Netzwerke zu erstellen.

1. Wählen Sie im Portal auf der linken Seite die Option **+** , und geben Sie im Suchfeld als Suchbegriff „Gateway für virtuelle Netzwerke“ ein.
2. Wählen Sie unter **Ergebnisse** die Option **Gateway für virtuelle Netzwerke**.
3. Wählen Sie unter **Gateway für virtuelle Netzwerke** die Option **Erstellen**, um die Seite **Gateway für virtuelle Netzwerke erstellen** zu öffnen.
4. Geben Sie unter **Gateway für virtuelle Netzwerke erstellen** die Werte für Ihr Netzwerkgateway an, indem Sie **Beispielwerte für Tutorial** verwenden. Fügen Sie die folgenden zusätzlichen Werte ein:

   - **SKU:** Basic
   - **Virtuelles Netzwerk**: Wählen Sie das zuvor erstellte virtuelle Netzwerk aus. Das von Ihnen erstellte Gatewaysubnetz wird automatisch ausgewählt.
   - **Erste IP-Konfiguration**:  Die öffentliche IP-Adresse Ihres Gateways.
     - Wählen Sie **Gateway-IP-Konfiguration erstellen**. Sie gelangen auf die Seite **Öffentliche IP-Adresse wählen**.
     - Wählen Sie **+Neu erstellen**, um die Seite **Öffentliche IP-Adresse erstellen** zu öffnen.
     - Geben Sie unter **Name** einen Namen für die öffentliche IP-Adresse ein. Behalten Sie für die SKU die Einstellung **Basic** bei, und wählen Sie dann **OK**, um Ihre Änderungen zu speichern.

       > [!Note]
       > Derzeit unterstützt VPN Gateway nur die dynamische Zuweisung öffentlicher IP-Adressen. Dies bedeutet aber nicht, dass sich die IP-Adresse ändert, nachdem sie Ihrem VPN-Gateway zugewiesen wurde. Die öffentliche IP-Adresse wird nur geändert, wenn das Gateway gelöscht und neu erstellt wird. Die IP-Adresse ändert sich nicht, wenn die Größe geändert wird, das VPN-Gateway zurückgesetzt wird oder andere interne Wartungs-/Upgradevorgänge für das VPN-Gateway durchgeführt werden.

5. Überprüfen Sie Ihre Gatewayeinstellungen.
6. Wählen Sie **Erstellen**, um das VPN-Gateway zu erstellen. Die Gatewayeinstellungen werden überprüft, und die Kachel „Gateway des virtuellen Netzwerks wird bereitgestellt“ wird auf Ihrem Dashboard angezeigt.

   >[!Note]
   >Die Erstellung eines Gateways kann bis zu 45 Minuten dauern. Unter Umständen müssen Sie die Portalseite aktualisieren, um den Status „Abgeschlossen“ anzuzeigen.

    Nach der Erstellung des Gateways wird die zugewiesene IP-Adresse unter dem virtuellen Netzwerk im Portal angezeigt. Das Gateway wird als verbundenes Gerät angezeigt. Wählen Sie das Gerät aus, um weitere Informationen zum Gateway anzuzeigen.

7. Wiederholen Sie die vorherigen Schritte (1-5) für Ihre Azure Stack Hub-Bereitstellung.

## <a name="create-the-local-network-gateway-in-azure-and-azure-stack-hub"></a>Erstellen des lokalen Netzwerkgateways in Azure und Azure Stack Hub

Mit dem Gateway des lokalen Netzwerks ist normalerweise Ihr lokaler Standort gemeint. Sie geben dem Standort einen Namen, auf den von Azure oder Azure Stack Hub verwiesen werden kann, und geben dann Folgendes an:

- Die IP-Adresse des lokalen VPN-Geräts, für das Sie eine Verbindung erstellen.
- Die IP-Adresspräfixe, die über das VPN-Gateway an das VPN-Gerät weitergeleitet werden. Die von Ihnen angegebenen Adresspräfixe befinden sich in Ihrem lokalen Netzwerk.

  >[!Note]
  >Wenn am lokalen Netzwerk Änderungen vorgenommen werden oder Sie die öffentliche IP-Adresse des VPN-Geräts ändern müssen, können Sie diese Werte später aktualisieren.

1. Klicken Sie im Portal auf **+Ressource erstellen**.
2. Geben Sie im Suchfeld den Text **Gateway für lokales Netzwerk** ein, und drücken Sie dann die **EINGABETASTE**, um die Suche zu starten. Eine Liste mit Ergebnissen wird angezeigt.
3. Wählen Sie **Gateway für lokales Netzwerk** und anschließend **Erstellen**, um die Seite **Lokales Netzwerkgateway erstellen** zu öffnen.
4. Geben Sie unter **Lokales Netzwerkgateway erstellen** die Werte für Ihr Gateway für das lokale Netzwerk an, indem Sie **Beispielwerte für Tutorial** verwenden. Fügen Sie die folgenden zusätzlichen Werte ein:

    - **IP-Adresse**: Die öffentliche IP-Adresse des VPN-Geräts, mit dem Azure oder Azure Stack Hub eine Verbindung herstellen soll. Geben Sie eine gültige öffentliche IP-Adresse an, die nicht hinter einer Netzwerkadressenübersetzung angeordnet ist, damit Azure die Adresse erreichen kann. Falls Ihnen die IP-Adresse gerade nicht vorliegt, können Sie einen Wert aus dem Beispiel als Platzhalter verwenden. Sie müssen dann später den Platzhalter durch die öffentliche IP-Adresse Ihres VPN-Geräts ersetzen. Azure kann erst dann eine Verbindung mit dem Gerät herstellen, wenn Sie eine gültige Adresse angeben.
    - **Adressraum**: der Adressbereich für das Netzwerk, das dieses lokale Netzwerk darstellt. Sie können mehrere Adressraumbereiche hinzufügen. Achten Sie darauf, dass sich die angegebenen Bereiche nicht mit den Bereichen anderer Netzwerke überschneiden, mit denen Sie eine Verbindung herstellen möchten. Azure leitet den Adressbereich, den Sie angeben, an die lokale IP-Adresse des VPN-Geräts weiter. Verwenden Sie Ihre eigenen Werte, wenn Sie eine Verbindung mit Ihrem lokalen Standort herstellen möchten, und keinen Beispielwert.
    - **BGP-Einstellungen konfigurieren**: Nur beim Konfigurieren von BGP verwenden. Lassen Sie diese Option andernfalls deaktiviert.
    - **Abonnement**: Vergewissern Sie sich, dass das richtige Abonnement angezeigt wird.
    - **Ressourcengruppe**: Wählen Sie die Ressourcengruppe aus, die Sie verwenden möchten. Sie können entweder eine neue Ressourcengruppe erstellen oder eine bereits erstellte auswählen.
    - **Standort**: Wählen Sie den Standort aus, an dem dieses Objekt erstellt wird. Es empfiehlt sich unter Umständen, den gleichen Ort auszuwählen, an dem sich auch Ihr VNet befindet, aber dies ist nicht zwingend erforderlich.
5. Wenn Sie mit dem Angeben der erforderlichen Werte fertig sind, können Sie **Erstellen** wählen, um das Gateway für das lokale Netzwerk zu erstellen.
6. Wiederholen Sie diese Schritte (1-5) für Ihre Azure Stack Hub-Bereitstellung.

## <a name="configure-your-connection"></a>Konfigurieren der Verbindung

Für Site-to-Site-Verbindungen mit einem lokalen Netzwerk ist ein VPN-Gerät erforderlich. Das von Ihnen konfigurierte VPN-Gerät wird als „Verbindung“ bezeichnet. Sie benötigen Folgendes, um die Verbindung zu konfigurieren:

- Einen gemeinsam verwendeten Schlüssel. Dieser Schlüssel ist derselbe gemeinsame Schlüssel, den Sie beim Erstellen Ihrer Site-to-Site-VPN-Verbindung angeben. In unseren Beispielen verwenden wir einen einfachen gemeinsamen Schlüssel. Es wird empfohlen, einen komplexeren Schlüssel zu generieren.
- Die öffentliche IP-Adresse Ihres Gateways für virtuelle Netzwerke. Sie können die öffentliche IP-Adresse mit dem Azure-Portal, mit PowerShell oder mit der CLI anzeigen. Die öffentliche IP-Adresse Ihres VPN-Gateways können Sie über das Azure-Portal ermitteln, indem Sie zu „Gateways für virtuelle Netzwerke“ navigieren und auf den Namen Ihres Gateways klicken.

Führen Sie die folgenden Schritte aus, um eine Site-to-Site-VPN-Verbindung zwischen dem Gateway des virtuellen Netzwerks und Ihrem lokalen VPN-Gerät herzustellen.

1. Wählen Sie im Azure-Portal die Option **+ Ressource erstellen**.
2. Suchen Sie nach **Verbindungen**.
3. Wählen Sie unter **Ergebnisse** die Option **Verbindungen**.
4. Wählen Sie unter **Verbindung** die Option **Erstellen**.
5. Konfigurieren Sie unter **Verbindung erstellen** die folgenden Einstellungen:

    - **Verbindungstyp**: Wählen Sie „Site-to-Site (IPsec)“ aus.
    - **Ressourcengruppe**: Wählen Sie Ihre Testressourcengruppe aus.
    - **Gateway für virtuelle Netzwerke**: Wählen Sie das erstellte Gateway für virtuelle Netzwerke aus.
    - **Lokales Netzwerkgateway**: Wählen Sie das erstellte lokale Netzwerkgateway aus.
    - **Verbindungsname**: Dieser Name wird automatisch mit den Werten der beiden Gateways ausgefüllt.
    - **Gemeinsam verwendeter Schlüssel**: Dieser Wert muss dem Wert entsprechen, den Sie für Ihr lokales VPN-Gerät verwenden. Im Tutorialbeispiel wird „abc123“ verwendet, aber Sie sollten einen komplexeren Wert verwenden. Entscheidend ist Folgendes: Dieser Wert *muss* dem Wert entsprechen, den Sie beim Konfigurieren Ihres VPN-Geräts angeben.
    - Die Werte für **Abonnement**, **Ressourcengruppe** und **Standort** sind festgelegt.

6. Wählen Sie **OK**, um die Verbindung zu erstellen.

Die Verbindung wird auf der Seite **Verbindungen** des Gateways für virtuelle Netzwerke angezeigt. Der Status wechselt von *Unbekannt* zu *Verbindung wird hergestellt* und dann zu *Erfolgreich*.

## <a name="next-steps"></a>Nächste Schritte

- Weitere Informationen zu Azure-Cloudmustern finden Sie unter [Cloudentwurfsmuster](/azure/architecture/patterns).
