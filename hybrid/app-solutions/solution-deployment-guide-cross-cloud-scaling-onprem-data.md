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
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a>Bereitstellen einer Hybrid-App mit lokalen Daten mit cloudübergreifender Skalierung

In diesem Lösungsleitfaden erfahren Sie, wie Sie eine Hybrid-App implementieren, die sowohl für Azure als auch für Azure Stack Hub gilt und für die nur eine gemeinsame lokale Datenquelle verwendet wird.

Indem Sie eine Hybrid Cloud-Lösung verwenden, können Sie die Compliancevorteile einer privaten Cloud mit der Skalierbarkeit der öffentlichen Cloud verbinden. Ihre Entwickler können auch das Microsoft-Entwicklerökosystem nutzen und ihre Fähigkeiten auf die Cloudumgebungen und lokalen Umgebungen anwenden.

## <a name="overview-and-assumptions"></a>Übersicht und Annahmen

Arbeiten Sie dieses Tutorial durch, um einen Workflow einzurichten, bei dem Entwickler eine identische Web-App in einer öffentlichen und einer privaten Cloud bereitstellen können. Diese App kann auf ein Netzwerk zugreifen, das nicht über das Internet geroutet werden kann und in der privaten Cloud gehostet wird. Diese Web-Apps werden überwacht. Wenn es zu einer Datenverkehrsspitze kommt, werden die DNS-Einträge mit einem Programm geändert, um Datenverkehr an die öffentliche Cloud umzuleiten. Wenn der Datenverkehr wieder auf die Menge vor der Spitze fällt, wird er wieder an die private Cloud geleitet.

Dieses Tutorial enthält die folgenden Aufgaben:

> [!div class="checklist"]
> - Bereitstellen eines SQL Server-Datenbankservers mit Hybridverbindung
> - Verbinden einer Web-App in der globalen Azure-Umgebung mit einem Hybridnetzwerk
> - Konfigurieren von DNS für die cloudübergreifende Skalierung
> - Konfigurieren von SSL-Zertifikaten für die cloudübergreifende Skalierung
> - Konfigurieren und Bereitstellen der Web-App
> - Erstellen eines Traffic Manager-Profils mit anschließender Konfiguration für die cloudübergreifende Skalierung
> - Einrichten der Application Insights-Überwachung und -Benachrichtigung für erhöhten Datenverkehr
> - Konfigurieren der automatischen Umschaltung des Datenverkehrs zwischen der globalen Azure-Umgebung und Azure Stack Hub

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub ist eine Erweiterung von Azure. Mit Azure Stack Hub holen Sie sich die Agilität und Innovation von Cloud Computing in Ihre lokale Umgebung. Sie erhalten die einzige Hybrid Cloud, mit der Sie Hybrid-Apps überall entwickeln und bereitstellen können.  
> 
> Im Artikel [Überlegungen zum Entwurf von Hybrid-Apps](overview-app-design-considerations.md) werden die wichtigen Aspekte in Bezug auf die Softwarequalität (Platzierung, Skalierbarkeit, Verfügbarkeit, Resilienz, Verwaltbarkeit und Sicherheit) beschrieben, die für das Entwerfen, Bereitstellen und Betreiben von Hybrid-Apps erforderlich sind. Die Überlegungen zum Entwurf dienen als Hilfe beim Optimieren des Designs von Hybrid-Apps, um für Produktionsumgebungen das Auftreten von Problemen zu minimieren.

### <a name="assumptions"></a>Annahmen

In diesem Tutorial wird davon ausgegangen, dass Sie bereits über Grundkenntnisse in Bezug auf die globale Azure-Umgebung und Azure Stack Hub verfügen. Lesen Sie diese Artikel, falls Sie sich vor Beginn des Tutorials informieren möchten:

- [Einführung in Azure](https://azure.microsoft.com/overview/what-is-azure/)
- [Übersicht über Azure Stack Hub](/azure-stack/operator/azure-stack-overview.md)

In diesem Tutorial wird auch davon ausgegangen, dass Sie über ein Azure-Abonnement verfügen. Wenn Sie kein Abonnement besitzen, müssen Sie ein [kostenloses Konto erstellen](https://azure.microsoft.com/free/), bevor Sie beginnen.

## <a name="prerequisites"></a>Voraussetzungen

Vergewissern Sie sich zunächst, dass die folgenden Anforderungen erfüllt sind bzw. dass Folgendes vorhanden ist:

- Ein Azure Stack Development Kit (ASDK) oder ein Abonnement für ein integriertes Azure Stack Hub-System. Befolgen Sie die Anleitung zum Bereitstellen des ASDK unter [Bereitstellen des ASDK mithilfe des Installationsprogramms](/azure-stack/asdk/asdk-install.md).
- Für Ihre Azure Stack Hub-Installation muss Folgendes installiert sein:
  - Azure App Service. Arbeiten Sie mit Ihrem Azure Stack Hub-Betreiber zusammen, um Azure App Service in Ihrer Umgebung bereitzustellen und zu konfigurieren. Für dieses Tutorial muss App Service mindestens über eine (1) verfügbare dedizierte Workerrolle verfügen.
  - Ein Windows Server 2016-Image.
  - Eine Windows Server 2016-Instanz mit einem Microsoft SQL Server-Image.
  - Die entsprechenden Tarife und Angebote.
  - Ein Domänenname für Ihre Web-App. Wenn Sie keinen Domänennamen besitzen, können Sie einen von einem Domänenanbieter wie GoDaddy, Bluehost oder InMotion erwerben.
- Ein SSL-Zertifikat für Ihre Domäne von einer vertrauenswürdigen Zertifizierungsstelle, z. B. LetsEncrypt.
- Eine Web-App, die mit einer SQL Server-Datenbank kommuniziert und Application Insights unterstützt. Sie können die Beispiel-App [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) von GitHub herunterladen.
- Ein Hybridnetzwerk zwischen einem virtuellen Azure-Netzwerk und einem virtuellen Azure Stack Hub-Netzwerk. Eine ausführliche Anleitung finden Sie unter [Konfigurieren der Hybrid Cloud-Konnektivität mit Azure und Azure Stack Hub](solution-deployment-guide-connectivity.md).

- Eine CI/CD-Hybridpipeline (Continuous Integration/Continuous Deployment) mit einem privaten Build-Agent in Azure Stack Hub. Eine ausführliche Anleitung finden Sie unter [Konfigurieren einer Hybrid Cloud-Identität mit Azure- und Azure Stack Hub-Apps](solution-deployment-guide-identity.md).

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a>Bereitstellen eines SQL Server-Datenbankservers mit Hybridverbindung

1. Melden Sie sich am Azure Stack Hub-Benutzerportal an.

2. Wählen Sie im **Dashboard** die Option **Marketplace**.

    ![Azure Stack Hub Marketplace](media/solution-deployment-guide-hybrid/image1.png)

3. Wählen Sie unter **Marketplace** die Option **Compute** und dann **Mehr**. Wählen Sie unter **Mehr** das Image **Free SQL Server License: SQL Server 2017 Developer on Windows Server** (Kostenlose SQL Server-Lizenz: SQL Server 2017 Developer unter Windows Server) aus.

    ![Auswählen eines VM-Images im Azure Stack Hub-Benutzerportal](media/solution-deployment-guide-hybrid/image2.png)

4. Wählen Sie unter **Free SQL Server License: SQL Server 2017 Developer on Windows Server** (Kostenlose SQL Server-Lizenz: SQL Server 2017 Developer unter Windows Server) die Option **Erstellen** aus.

5. Geben Sie unter **Grundlagen > Grundeinstellungen konfigurieren** einen **Namen** für den virtuellen Computer (VM) und einen **Benutzernamen** und ein **Kennwort** für den SQL Server-Systemadministrator an.  Wählen Sie in der Dropdownliste **Abonnement** das Abonnement aus, für das Sie die Bereitstellung durchführen möchten. Wählen Sie unter **Ressourcengruppe** die Option **Vorhandene auswählen**, und ordnen Sie die VM in derselben Ressourcengruppe wie Ihre Azure Stack Hub-Web-App an.

    ![Konfigurieren der Grundeinstellungen für die VM im Azure Stack Hub-Benutzerportal](media/solution-deployment-guide-hybrid/image3.png)

6. Wählen Sie unter **Größe** eine Größe für Ihre VM aus. Für dieses Tutorial empfehlen wir „A2_Standard“ oder „DS2_V2_Standard“.

7. Konfigurieren Sie unter **Einstellungen > Optionale Features konfigurieren** die folgenden Einstellungen:

   - **Speicherkonto**: Erstellen Sie ein neues Konto, falls erforderlich.
   - **Virtuelles Netzwerk:**

     > [!Important]  
     > Stellen Sie sicher, dass Ihre SQL Server-VM in demselben virtuellen Netzwerk wie die VPN-Gateways bereitgestellt wird.

   - **Öffentliche IP-Adresse:** Verwenden Sie die Standardeinstellungen.
   - **Netzwerksicherheitsgruppe**: (NSG). Erstellen Sie eine neue NSG.
   - **Extensions and Monitoring** (Erweiterungen und Überwachung): Behalten Sie die Standardeinstellungen bei.
   - **Diagnosespeicherkonto**: Erstellen Sie ein neues Konto, falls erforderlich.
   - Wählen Sie **OK**, um Ihre Konfiguration zu speichern.

     ![Konfigurieren der optionalen VM-Features im Azure Stack Hub-Benutzerportal](media/solution-deployment-guide-hybrid/image4.png)

8. Konfigurieren Sie unter **SQL Server-Einstellungen** die folgenden Einstellungen:

   - Wählen Sie unter **SQL-Konnektivität** die Option **Öffentlich (Internet)** aus.
   - Behalten Sie für **Port** den Standardwert **1433** bei.
   - Wählen Sie unter **SQL-Authentifizierung** die Option **Aktivieren**.

     > [!Note]  
     > Wenn Sie die SQL-Authentifizierung aktivieren, sollten automatisch die „SQLAdmin“-Informationen eingefügt werden, die Sie unter **Grundlagen** konfiguriert haben.

   - Behalten Sie für die übrigen Einstellungen die Standardwerte bei. Klicken Sie auf **OK**.

     ![Konfigurieren der SQL Server-Einstellungen im Azure Stack Hub-Benutzerportal](media/solution-deployment-guide-hybrid/image5.png)

9. Sehen Sie sich unter **Zusammenfassung** die Konfiguration der VM an, und wählen Sie dann **OK** aus, um die Bereitstellung zu starten.

    ![Zusammenfassung der Konfiguration im Azure Stack Hub-Benutzerportal](media/solution-deployment-guide-hybrid/image6.png)

10. Die Erstellung der neuen VM dauert einige Zeit. Sie können den STATUS Ihrer VMs unter **Virtuelle Computer** anzeigen.

    ![VM-Status im Azure Stack Hub-Benutzerportal](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a>Erstellen von Web-Apps in Azure und Azure Stack Hub

Azure App Service vereinfacht die Ausführung und Verwaltung einer Web-App. Da Azure Stack Hub mit Azure konsistent ist, kann App Service in beiden Umgebungen ausgeführt werden. Sie verwenden App Service zum Hosten Ihrer App.

### <a name="create-web-apps"></a>Erstellen von Web-Apps

1. Erstellen Sie eine Web-App in Azure, indem Sie die Anleitung unter [Verwalten eines App Service-Plans in Azure](/azure/app-service/app-service-plan-manage#create-an-app-service-plan) befolgen. Stellen Sie sicher, dass Sie die Web-App unter demselben Abonnement und derselben Ressourcengruppe wie für Ihr Hybridnetzwerk anordnen.

2. Wiederholen Sie den vorherigen Schritt (1) in Azure Stack Hub.

### <a name="add-route-for-azure-stack-hub"></a>Hinzufügen der Route für Azure Stack Hub

App Service in Azure Stack Hub muss über das öffentliche Internet geroutet werden können, damit Benutzer auf Ihre App zugreifen können. Wenn Ihre Azure Stack Hub-Instanz über das Internet zugänglich ist, notieren Sie sich die öffentliche IP-Adresse oder URL der Azure Stack Hub-Web-App.

Bei Verwendung eines ASDK können Sie [eine statische NAT-Zuordnung konfigurieren](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal), um App Service außerhalb der virtuellen Umgebung verfügbar zu machen.

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a>Verbinden einer Web-App in Azure mit einem Hybridnetzwerk

Um die Konnektivität zwischen dem Web-Front-End in Azure und der SQL Server-Datenbank in Azure Stack Hub bereitzustellen, muss die Web-App mit dem Hybridnetzwerk zwischen Azure und Azure Stack Hub verbunden werden. Folgendes ist erforderlich, um die Konnektivität zu aktivieren:

- Konfigurieren der Punkt-zu-Site-Konnektivität.
- Konfigurieren der Web-App.
- Ändern des Gateways für lokale Netzwerke in Azure Stack Hub.

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a>Konfigurieren des virtuellen Azure-Netzwerks für Punkt-zu-Standort-Konnektivität

Das Gateway für virtuelle Netzwerke auf der Azure-Seite des Hybridnetzwerks muss Point-to-Site-Verbindungen zulassen, um in Azure App Service integriert werden zu können.

1. Navigieren Sie in Azure zur Seite für das Gateway für virtuelle Netzwerke. Wählen Sie unter **Einstellungen** die Option **Punkt-zu-Standort-Konfiguration**.

    ![Punkt-zu-Standort-Option im Gateway für virtuelle Azure-Netzwerke](media/solution-deployment-guide-hybrid/image8.png)

2. Wählen Sie **Jetzt konfigurieren**, um die Punkt-zu-Standort-Konfiguration zu starten.

    ![Starten der Punkt-zu-Standort-Konfiguration im Gateway für virtuelle Azure-Netzwerke](media/solution-deployment-guide-hybrid/image9.png)

3. Geben Sie auf der Konfigurationsseite **Punkt-zu-Standort** im Feld **Adresspool** den privaten IP-Adressbereich ein, den Sie verwenden möchten.

   > [!Note]  
   > Stellen Sie sicher, dass sich der angegebene Bereich nicht mit Adressbereichen überlappt, die bereits von Subnetzen in den globalen Azure- oder Azure Stack Hub-Komponenten des Hybridnetzwerks verwendet werden.

   Deaktivieren Sie unter **Tunneltyp** die Option **IKEv2-VPN**. Wählen Sie **Speichern**, um die Punkt-zu-Standort-Konfiguration abzuschließen.

   ![Punkt-zu-Standort-Einstellungen im Gateway für virtuelle Azure-Netzwerke](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a>Integrieren der Azure App Service-App in das Hybridnetzwerk

1. Befolgen Sie die Anleitung unter [VNET-Integration, die ein Gateway erfordert](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration), um die App mit dem Azure VNET zu verbinden.

2. Navigieren Sie für den App Service-Plan, der die Web-App hostet, zu **Einstellungen**. Wählen Sie unter **Einstellungen** die Option **Netzwerk**.

    ![Konfigurieren des Netzwerks für den App Service-Plan](media/solution-deployment-guide-hybrid/image11.png)

3. Wählen Sie unter **VNET-Integration** die Option **Klicken Sie zur Verwaltung hier**.

    ![Verwalten der VNET-Integration für den App Service-Plan](media/solution-deployment-guide-hybrid/image12.png)

4. Wählen Sie das VNET aus, das Sie konfigurieren möchten. Geben Sie unter **AN DAS VNET WEITERGELEITETE IP-ADRESSEN** den IP-Adressbereich für das Azure VNET, das Azure Stack Hub VNET und die Punkt-zu-Standort-Adressräume ein. Wählen Sie **Speichern**, um diese Einstellungen zu überprüfen und zu speichern.

    ![IP-Adressbereiche für die Weiterleitung in der VNET-Integration](media/solution-deployment-guide-hybrid/image13.png)

Weitere Informationen zur Integration von App Service in Azure VNETs finden Sie unter [Integrieren Ihrer App in ein Azure Virtual Network](/azure/app-service/web-sites-integrate-with-vnet).

### <a name="configure-the-azure-stack-hub-virtual-network"></a>Konfigurieren des virtuellen Azure Stack Hub-Netzwerks

Das Gateway für lokale Netzwerke im virtuellen Azure Stack Hub-Netzwerk muss zum Weiterleiten von Datenverkehr aus dem Punkt-zu-Standort-Adressbereich von App Service konfiguriert werden.

1. Navigieren Sie in Azure Stack Hub zu **Lokales Netzwerkgateway**. Wählen Sie unter **Einstellungen** die Option **Konfiguration**.

    ![Konfigurationsoption für Gateway im lokalen Netzwerkgateway für Azure Stack Hub](media/solution-deployment-guide-hybrid/image14.png)

2. Geben Sie unter **Adressraum** den Punkt-zu-Standort-Adressbereich für das Gateway für virtuelle Netzwerke in Azure ein.

    ![Punkt-zu-Standort-Adressraum im lokalen Netzwerkgateway für Azure Stack Hub](media/solution-deployment-guide-hybrid/image15.png)

3. Wählen Sie **Speichern** aus, um die Konfiguration zu überprüfen und zu speichern.

## <a name="configure-dns-for-cross-cloud-scaling"></a>Konfigurieren von DNS für die cloudübergreifende Skalierung

Indem Benutzer das DNS für cloudübergreifende Apps richtig konfigurieren, können sie auf die globale Azure-Umgebung und die Azure Stack Hub-Instanzen Ihrer Web-App zugreifen. Mit der DNS-Konfiguration für dieses Tutorial kann Azure Traffic Manager auch Datenverkehr weiterleiten, wenn sich die Last erhöht oder verringert.

In diesem Tutorial wird Azure DNS zum Verwalten des DNS verwendet, weil App Service-Domänen nicht funktionieren.

### <a name="create-subdomains"></a>Erstellen von Unterdomänen

Da für Traffic Manager DNS-CNAME-Einträge verwendet werden, wird eine Unterdomäne benötigt, um Datenverkehr korrekt an die Endpunkte weiterleiten zu können. Weitere Informationen zu DNS-Einträgen und zur Domänenzuordnung finden Sie unter [Zuordnen von Domänen mithilfe von Traffic Manager](/azure/app-service/web-sites-traffic-manager-custom-domain-name).

Für den Azure-Endpunkt erstellen Sie eine Unterdomäne, die Benutzer zum Zugreifen auf Ihre Web-App verwenden können. Für dieses Tutorial können Sie **app.northwind.com** verwenden, aber Sie sollten diesen Wert basierend auf Ihrer eigenen Domäne anpassen.

Außerdem müssen Sie eine Unterdomäne mit einem A-Eintrag für den Azure Stack Hub-Endpunkt erstellen. Sie können **azurestack.northwind.com** verwenden.

### <a name="configure-a-custom-domain-in-azure"></a>Konfigurieren einer benutzerdefinierten Domäne in Azure

1. Fügen Sie den Hostnamen **app.northwind.com** der Azure-Web-App hinzu, indem Sie [Azure App Service einen CNAME-Eintrag zuordnen](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).

### <a name="configure-custom-domains-in-azure-stack-hub"></a>Konfigurieren von benutzerdefinierten Domänen in Azure Stack Hub

1. Fügen Sie den Hostnamen **azurestack.northwind.com** der Azure Stack Hub-Web-App hinzu, indem Sie [Azure App Service einen A-Eintrag zuordnen](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record). Verwenden Sie für die App Service-App die IP-Adresse, die über das Internet geroutet werden kann.

2. Fügen Sie den Hostnamen **app.northwind.com** der Azure Stack Hub-Web-App hinzu, indem Sie [Azure App Service einen CNAME-Eintrag zuordnen](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record). Verwenden Sie den Hostnamen, den Sie im vorherigen Schritt (1) konfiguriert haben, als Ziel für den CNAME-Eintrag.

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a>Konfigurieren von SSL-Zertifikaten für die cloudübergreifende Skalierung

Es ist wichtig, dass Sie sicherstellen, dass von Ihrer Web-App erfasste vertrauliche Daten bei der Übertragung zur und der Speicherung in der SQL-Datenbank sicher sind.

Sie konfigurieren Ihre Azure- und Azure Stack Hub-Web-Apps so, dass für den gesamten eingehenden Datenverkehr SSL-Zertifikate verwendet werden.

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a>Hinzufügen von SSL zu Azure und Azure Stack Hub

Fügen Sie SSL für Azure wie folgt hinzu:

1. Stellen Sie sicher, dass das beschaffte SSL-Zertifikat für die von Ihnen erstellte Unterdomäne gültig ist. (Hierbei können auch Platzhalterzertifikate verwendet werden.)

2. Befolgen Sie in Azure die Anleitung in den Abschnitten **Vorbereiten Ihrer Web-App** und **Binden Ihres SSL-Zertifikats** des Artikels [Binden eines vorhandenen benutzerdefinierten SSL-Zertifikats an Azure-Web-Apps](/azure/app-service/app-service-web-tutorial-custom-ssl). Wählen Sie unter **SSL-Typ** die Option **SNI-basiertes SSL**.

3. Leiten Sie den gesamten Datenverkehr an den HTTPS-Port um. Befolgen Sie die Anleitung im Abschnitt **Erzwingen von HTTPS** des Artikels [Binden eines vorhandenen benutzerdefinierten SSL-Zertifikats an Azure-Web-Apps](/azure/app-service/app-service-web-tutorial-custom-ssl).

Fügen Sie SSL für Azure Stack Hub wie folgt hinzu:

1. Wiederholen Sie die Schritte 1 bis 3, die Sie für Azure verwendet haben.

## <a name="configure-and-deploy-the-web-app"></a>Konfigurieren und Bereitstellen der Web-App

Sie konfigurieren den App-Code so, dass die Telemetriedaten an die richtige Application Insights-Instanz gemeldet werden, und konfigurieren die Web-Apps mit den richtigen Verbindungszeichenfolgen. Weitere Informationen zu Application Insights finden Sie unter [Was ist Application Insights?](/azure/application-insights/app-insights-overview).

### <a name="add-application-insights"></a>Hinzufügen von Application Insights

1. Öffnen Sie Ihre Web-App in Microsoft Visual Studio.

2. [Fügen Sie Ihrem Projekt Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) hinzu, um die Telemetriedaten zu übertragen, die von Application Insights zum Erstellen von Warnungen verwendet werden, wenn der Webdatenverkehr zunimmt oder sich verringert.

### <a name="configure-dynamic-connection-strings"></a>Konfigurieren von dynamischen Verbindungszeichenfolgen

Für jede Instanz der Web-App wird eine andere Methode zum Herstellen der Verbindung mit der SQL-Datenbank verwendet. Für die App in Azure wird die private IP-Adresse der SQL Server-VM verwendet, und die App in Azure Stack Hub nutzt die öffentliche IP-Adresse der SQL Server-VM.

> [!Note]  
> Auf einem integrierten Azure Stack Hub-System sollte die öffentliche IP-Adresse nicht über das Internet geroutet werden können. Für ein ASDK kann die öffentliche IP-Adresse nicht außerhalb des ASDK geroutet werden.

Sie können die App Service-Umgebungsvariablen verwenden, um an jede Instanz der App eine andere Verbindungszeichenfolge zu übergeben.

1. Öffnen Sie die App in Visual Studio.

2. Öffnen Sie „Startup.cs“, und suchen Sie nach dem folgenden Codeblock:

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. Ersetzen Sie den obigen Codeblock durch den folgenden Code, in dem eine Verbindungszeichenfolge verwendet wird, die in der Datei *appsettings.json* definiert ist:

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a>Konfigurieren von App Service-App-Einstellungen

1. Erstellen Sie Verbindungszeichenfolgen für Azure und Azure Stack Hub. Die Zeichenfolgen sollten mit Ausnahme der verwendeten IP-Adressen identisch sein.

2. Fügen Sie in Azure und Azure Stack Hub die entsprechende Verbindungszeichenfolge [als App-Einstellung](/azure/app-service/web-sites-configure) in der Web-App hinzu, indem Sie `SQLCONNSTR\_` als Präfix im Namen verwenden.

3. **Speichern** Sie die Web-App-Einstellungen, und starten Sie die App neu.

## <a name="enable-automatic-scaling-in-global-azure"></a>Aktivieren der automatischen Skalierung in der globalen Azure-Umgebung

Wenn Sie Ihre Web-App in einer App Service-Umgebung erstellen, beginnt sie mit einer Instanz. Sie können automatisch aufskalieren, um Instanzen hinzuzufügen und so für Ihre App mehr Computeressourcen bereitzustellen. Auf ähnliche Weise können Sie automatisch abskalieren und die Anzahl von Instanzen reduzieren, die Ihre App benötigt.

> [!Note]  
> Sie müssen über einen App Service-Plan verfügen, um das Auf- und Abskalieren zu konfigurieren. Wenn Sie keinen Plan besitzen, sollten Sie einen erstellen, bevor Sie mit den nächsten Schritten beginnen.

### <a name="enable-automatic-scale-out"></a>Aktivieren der automatischen horizontalen Skalierung

1. Suchen Sie in Azure nach dem App Service-Plan für die Standorte, die Sie aufskalieren möchten, und wählen Sie dann die Option **Aufskalieren (App Service-Plan)** .

    ![Aufskalieren von Azure App Service](media/solution-deployment-guide-hybrid/image16.png)

2. Wählen Sie **Automatische Skalierung aktivieren**.

    ![Aktivieren der Autoskalierung in Azure App Service](media/solution-deployment-guide-hybrid/image17.png)

3. Geben Sie unter **Name der Einstellung für die automatische Skalierung** einen Namen ein. Wählen Sie unter **Standard** für die Standardregel der automatischen Skalierung die Option **Basierend auf einer Metrik skalieren**. Legen Sie **Instanzgrenzwerte** auf **Minimum: 1**, **Maximum: 10** und **Standard: 1** fest.

    ![Konfigurieren der Autoskalierung in Azure App Service](media/solution-deployment-guide-hybrid/image18.png)

4. Wählen Sie **+ Regel hinzufügen**.

5. Wählen Sie unter **Metrikquelle** die Option **Aktuelle Ressource**. Verwenden Sie die folgenden Kriterien und Aktionen für die Regel.

#### <a name="criteria"></a>Kriterien

1. Wählen Sie unter **Zeitaggregation** die Option **Durchschnitt**.

2. Wählen Sie unter **Metrikname** die Option **CPU-Prozentsatz**.

3. Wählen Sie unter **Operator** die Option **Größer als**.

   - Legen Sie **Schwellenwert** auf **50** fest.
   - Legen Sie die **Dauer** auf **10** fest.

#### <a name="action"></a>Aktion

1. Wählen Sie unter **Vorgang** die Option **Anzahl erhöhen um**.

2. Legen Sie **Instanzenanzahl** auf **2** fest.

3. Legen Sie **Abklingen** auf **5** fest.

4. Wählen Sie **Hinzufügen**.

5. Wählen Sie **+ Regel hinzufügen**.

6. Wählen Sie unter **Metrikquelle** die Option **Aktuelle Ressource**.

   > [!Note]  
   > Die aktuelle Ressource enthält den Namen bzw. die GUID Ihres App Service-Plans, und die Dropdownlisten **Ressourcentyp** und **Ressource** sind nicht verfügbar.

### <a name="enable-automatic-scale-in"></a>Aktivieren des automatischen Abskalierens

Bei einer Verringerung des Datenverkehrs kann die Azure-Web-App die Anzahl von aktiven Instanzen reduzieren, um die Kosten zu senken. Diese Aktion ist weniger aggressiv als die horizontale Skalierung und minimiert die Auswirkungen auf App-Benutzer.

1. Navigieren Sie zur Standardbedingung für das Aufskalieren unter **Standard**, und wählen Sie dann **+ Regel hinzufügen** aus. Verwenden Sie die folgenden Kriterien und Aktionen für die Regel.

#### <a name="criteria"></a>Kriterien

1. Wählen Sie unter **Zeitaggregation** die Option **Durchschnitt**.

2. Wählen Sie unter **Metrikname** die Option **CPU-Prozentsatz**.

3. Wählen Sie unter **Operator** die Option **Kleiner als**.

   - Legen Sie **Schwellenwert** auf **30** fest.
   - Legen Sie die **Dauer** auf **10** fest.

#### <a name="action"></a>Aktion

1. Wählen Sie unter **Vorgang** die Option **Anzahl verringern um**.

   - Legen Sie **Instanzenanzahl** auf **1** fest.
   - Legen Sie **Abklingen** auf **5** fest.

2. Wählen Sie **Hinzufügen**.

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a>Erstellen eines Traffic Manager-Profils mit anschließender Konfiguration für die cloudübergreifende Skalierung

Erstellen Sie ein Traffic Manager-Profil in Azure, und konfigurieren Sie dann die Endpunkte, um die cloudübergreifende Skalierung zu ermöglichen.

### <a name="create-traffic-manager-profile"></a>Erstellen eines Traffic Manager-Profils

1. Wählen Sie **Ressource erstellen**.
2. Wählen Sie **Netzwerk** aus.
3. Wählen Sie **Traffic Manager-Profil**, und konfigurieren Sie folgende Einstellungen:

   - Geben Sie unter **Name** einen Namen für Ihr Profil ein. Dieser Name **muss** in der Zone „trafficmanager.net“ eindeutig sein und wird genutzt, um einen neuen DNS-Namen zu erstellen (z.B. „northwindstore.trafficmanager.net“).
   - Wählen Sie unter **Routingmethode** die Option **Gewichtet**.
   - Wählen Sie unter **Abonnement** das Abonnement aus, unter dem Sie dieses Profil erstellen möchten.
   - Erstellen Sie unter **Ressourcengruppe** eine neue Ressourcengruppe für dieses Profil.
   - Wählen Sie unter **Ressourcengruppenstandort** den Speicherort für die Ressourcengruppe aus. Diese Einstellung bezieht sich auf den Speicherort der Ressourcengruppe und hat keine Auswirkungen auf das global bereitgestellte Traffic Manager-Profil.

4. Klicken Sie auf **Erstellen**.

    ![Erstellen eines Traffic Manager-Profils](media/solution-deployment-guide-hybrid/image19.png)

   Wenn die globale Bereitstellung Ihres Traffic Manager-Profils abgeschlossen ist, wird es in der Liste mit den Ressourcen für die Ressourcengruppe angezeigt, unter der Sie es erstellt haben.

### <a name="add-traffic-manager-endpoints"></a>Hinzufügen von Traffic Manager-Endpunkten

1. Suchen Sie nach dem Traffic Manager-Profil, das Sie erstellt haben. Wählen Sie das Profil aus, nachdem Sie die Navigation zur Ressourcengruppe für das Profil durchgeführt haben.

2. Wählen Sie im **Traffic Manager-Profil** unter **EINSTELLUNGEN** die Option **Endpunkte**.

3. Wählen Sie **Hinzufügen**.

4. Verwenden Sie unter **Endpunkt hinzufügen** die folgenden Einstellungen für Azure Stack Hub:

   - Wählen Sie unter **Typ** die Option **Externer Endpunkt**.
   - Geben Sie einen **Namen** für den Endpunkt ein.
   - Geben Sie unter **Fully-qualified domain name (FQDN) or IP** (Vollqualifizierter Domänenname [FQDN] oder IP) die externe URL für Ihre Azure Stack Hub-Web-App ein.
   - Behalten Sie für **Gewichtung** den Standardwert **1** bei. Diese Gewichtung bewirkt, dass der gesamte Datenverkehr an diesen Endpunkt geleitet wird, sofern sein Status intakt ist.
   - Lassen Sie **Als deaktiviert hinzufügen** deaktiviert.

5. Wählen Sie **OK**, um den Azure Stack Hub-Endpunkt zu speichern.

Als Nächstes konfigurieren Sie den Azure-Endpunkt.

1. Wählen Sie unter **Traffic Manager-Profil** die Option **Endpunkte**.
2. Wählen Sie **+ Hinzufügen** aus.
3. Verwenden Sie unter **Endpunkt hinzufügen** die folgenden Einstellungen für Azure:

   - Wählen Sie für **Typ** die Option **Azure-Endpunkt**.
   - Geben Sie einen **Namen** für den Endpunkt ein.
   - Wählen Sie unter **Zielressourcentyp** die Option **App Service**.
   - Wählen Sie unter **Zielressource** die Option **App Service auswählen**, um eine Liste mit den Web-Apps für dasselbe Abonnement anzuzeigen.
   - Wählen Sie unter **Ressource** den App Service aus, den Sie als ersten Endpunkt hinzufügen möchten.
   - Wählen Sie unter **Gewichtung** den Wert **2**. Diese Einstellung führt dazu, dass der gesamte Datenverkehr an diesen Endpunkt geleitet wird, wenn der primäre Endpunkt fehlerhaft ist oder Sie über eine Regel oder Warnung verfügen, mit der Datenverkehr bei der Auslösung umgeleitet wird.
   - Lassen Sie **Als deaktiviert hinzufügen** deaktiviert.

4. Wählen Sie **OK**, um den Azure-Endpunkt zu speichern.

Nachdem beide Endpunkte konfiguriert wurden, werden sie im **Traffic Manager-Profil** aufgeführt, wenn Sie **Endpunkte** wählen. Im Beispiel im folgenden Screenshot werden zwei Endpunkte jeweils mit Status- und Konfigurationsinformationen angezeigt.

![Endpunkte im Traffic Manager-Profil](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting"></a>Einrichten der Application Insights-Überwachung und -Warnungen

Mit Azure Application Insights können Sie Ihre App überwachen und basierend auf den von Ihnen konfigurierten Bedingungen Warnungen senden. Beispiele: Die App ist nicht verfügbar, weist Fehler auf oder zeigt Leistungsprobleme an.

Sie nutzen Application Insights-Metriken, um Warnungen zu erstellen. Wenn diese Warnungen ausgelöst werden, wird für die Instanz Ihrer Web-App automatisch von Azure Stack Hub zu Azure gewechselt, um aufzuskalieren, und dann zurück zu Azure Stack Hub, um abzuskalieren.

### <a name="create-an-alert-from-metrics"></a>Warnung auf Grundlage von Metriken erstellen

Navigieren Sie zur Ressourcengruppe für dieses Tutorial, und wählen Sie dann die Application Insights-Instanz aus, um **Application Insights** zu öffnen.

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

Sie verwenden diese Ansicht, um eine Warnung für das Aufskalieren und eine Warnung für das Abskalieren zu erstellen.

### <a name="create-the-scale-out-alert"></a>Erstellen der Warnung für das horizontale Hochskalieren

1. Wählen Sie unter **KONFIGURIEREN** die Option **Warnungen (klassisch)** .
2. Wählen Sie **Metrikwarnung hinzufügen (klassisch)** .
3. Konfigurieren Sie unter **Regel hinzufügen** folgende Einstellungen:

   - Geben Sie unter **Name** den Namen **Burst into Azure Cloud** ein.
   - Das Angeben einer **Beschreibung** ist optional.
   - Wählen Sie unter **Quelle** > **Warnung bei** die Option **Metriken**.
   - Wählen Sie unter **Kriterien** Ihr Abonnement, die Ressourcengruppe für Ihr Traffic Manager-Profil und den Namen des Traffic Manager-Profils für die Ressource aus.

4. Wählen Sie unter **Metrik** die Option **Anforderungsrate**.
5. Wählen Sie unter **Bedingung** die Option **Größer als**.
6. Geben Sie unter **Schwellenwert** den Wert **2** ein.
7. Wählen Sie unter **Zeitraum** die Option **In den letzten 5 Minuten**.
8. Unter **Benachrichtigen über**:
   - Aktivieren Sie das Kontrollkästchen **E-Mail-Besitzer, Mitwirkende und Leser**.
   - Geben Sie unter **Zusätzliche Administrator-E-Mail-Adressen** Ihre E-Mail-Adresse ein.

9. Wählen Sie in der Menüleiste die Option **Speichern**.

### <a name="create-the-scale-in-alert"></a>Erstellen der Warnung für das Abskalieren

1. Wählen Sie unter **KONFIGURIEREN** die Option **Warnungen (klassisch)** .
2. Wählen Sie **Metrikwarnung hinzufügen (klassisch)** .
3. Konfigurieren Sie unter **Regel hinzufügen** folgende Einstellungen:

   - Geben Sie unter **Name** den Namen **Scale back into Azure Stack Hub** ein.
   - Das Angeben einer **Beschreibung** ist optional.
   - Wählen Sie unter **Quelle** > **Warnung bei** die Option **Metriken**.
   - Wählen Sie unter **Kriterien** Ihr Abonnement, die Ressourcengruppe für Ihr Traffic Manager-Profil und den Namen des Traffic Manager-Profils für die Ressource aus.

4. Wählen Sie unter **Metrik** die Option **Anforderungsrate**.
5. Wählen Sie unter **Bedingung** die Option **Kleiner als**.
6. Geben Sie unter **Schwellenwert** den Wert **2** ein.
7. Wählen Sie unter **Zeitraum** die Option **In den letzten 5 Minuten**.
8. Unter **Benachrichtigen über**:
   - Aktivieren Sie das Kontrollkästchen **E-Mail-Besitzer, Mitwirkende und Leser**.
   - Geben Sie unter **Zusätzliche Administrator-E-Mail-Adressen** Ihre E-Mail-Adresse ein.

9. Wählen Sie in der Menüleiste die Option **Speichern**.

Im folgenden Screenshot sind die Warnungen für das Auf- und Abskalieren dargestellt.

   ![Application Insights-Warnungen (klassisch)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a>Umleiten von Datenverkehr zwischen Azure und Azure Stack Hub

Sie können das manuelle oder automatische Umschalten zwischen Azure und Azure Stack Hub für Ihren Web-App-Datenverkehr konfigurieren.

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a>Konfigurieren des manuellen Umschaltens zwischen Azure und Azure Stack Hub

Wenn Ihre Website die von Ihnen konfigurierten Schwellenwerte erreicht, erhalten Sie eine Warnung. Führen Sie die folgenden Schritte aus, um Datenverkehr manuell an Azure umzuleiten.

1. Wählen Sie im Azure-Portal Ihr Traffic Manager-Profil aus.

    ![Traffic Manager-Endpunkte im Azure-Portal](media/solution-deployment-guide-hybrid/image20.png)

2. Wählen Sie **Endpunkte**.
3. Wählen Sie die Option **Azure-Endpunkt**.
4. Wählen Sie unter **Status** die Option **Aktiviert** und dann **Speichern**.

    ![Aktivieren des Azure-Endpunkts im Azure-Portal](media/solution-deployment-guide-hybrid/image23.png)

5. Wählen Sie unter **Endpunkte** für das Traffic Manager-Profil die Option **Externer Endpunkt**.
6. Wählen Sie unter **Status** die Option **Deaktiviert** und dann **Speichern**.

    ![Deaktivieren des Azure Stack Hub-Endpunkts im Azure-Portal](media/solution-deployment-guide-hybrid/image24.png)

Nachdem die Endpunkte konfiguriert wurden, fließt der App-Datenverkehr nicht mehr zur Azure Stack Hub-Web-App, sondern zu Ihrer horizontal skalierten Azure-Web-App.

 ![Für Azure-Web-App-Datenverkehr geänderte Endpunkte](media/solution-deployment-guide-hybrid/image25.png)

Verwenden Sie zum Umkehren des Datenverkehrsflusses zurück zu Azure Stack Hub die vorherigen Schritte, um Folgendes durchzuführen:

- Aktivieren des Azure Stack Hub-Endpunkts.
- Deaktivieren des Azure-Endpunkts.

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a>Konfigurieren des automatischen Umschaltens zwischen Azure und Azure Stack Hub

Sie können die Application Insights-Überwachung auch nutzen, wenn Ihre App in einer [serverlosen](https://azure.microsoft.com/overview/serverless-computing/) Umgebung ausgeführt wird, die von Azure Functions bereitgestellt wird.

In diesem Szenario können Sie Application Insights so konfigurieren, dass ein Webhook zum Aufrufen einer Funktions-App verwendet wird. Diese App aktiviert bzw. deaktiviert automatisch einen Endpunkt als Reaktion auf eine Warnung.

Verwenden Sie die folgenden Schritte als Anleitung zum Konfigurieren der automatischen Umschaltung von Datenverkehr.

1. Erstellen Sie eine Azure-Funktions-App.
2. Erstellen Sie eine per HTTP ausgelöste Funktion.
3. Importieren Sie die Azure SDKs für Resource Manager, Web-Apps und Traffic Manager.
4. Entwickeln Sie Code für folgende Zwecke:

   - Durchführen der Authentifizierung für Ihr Azure-Abonnement
   - Verwenden eines Parameters, mit dem die Traffic Manager-Endpunkte zum Weiterleiten von Datenverkehr an Azure oder Azure Stack Hub umgeschaltet werden

5. Speichern Sie Ihren Code, und fügen Sie die Funktions-App-URL mit den entsprechenden Parametern dem Abschnitt **Webhook** der Einstellungen für die Application Insights-Warnungsregeln hinzu.
6. Datenverkehr wird automatisch umgeleitet, wenn eine Application Insights-Warnung ausgelöst wird.

## <a name="next-steps"></a>Nächste Schritte

- Weitere Informationen zu Azure-Cloudmustern finden Sie unter [Cloudentwurfsmuster](/azure/architecture/patterns).
