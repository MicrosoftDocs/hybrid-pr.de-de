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
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a>Bereitstellen eines Kubernetes-Hochverfügbarkeitsclusters in Azure Stack Hub

In diesem Artikel erfahren Sie, wie Sie eine hochverfügbare Kubernetes-Clusterumgebung erstellen, die in mehreren Azure Stack Hub-Instanzen an unterschiedlichen physischen Standorten bereitgestellt wird.

In diesem Lösungsbereitstellungshandbuch lernen Sie Folgendes:

> [!div class="checklist"]
> - Herunterladen und Vorbereiten der AKS-Engine
> - Herstellen einer Verbindung mit der Hilfs-VM der AKS-Engine
> - Bereitstellen eines Kubernetes-Clusters
> - Herstellen einer Verbindung mit dem Kubernetes-Cluster
> - Herstellen einer Verbindung zwischen Azure Pipelines und dem Kubernetes-Cluster
> - Konfigurieren der Überwachung
> - Bereitstellen der Anwendung
> - Autoskalieren der Anwendung
> - Traffic Manager konfigurieren
> - Kubernetes aktualisieren
> - Skalieren von Kubernetes

> [!Tip]  
> ![Hybridsäulen](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub ist eine Erweiterung von Azure. Mit Azure Stack Hub holen Sie sich die Agilität und Innovation von Cloud Computing in Ihre lokale Umgebung. Sie erhalten die einzige Hybrid Cloud, mit der Sie Hybrid-Apps überall entwickeln und bereitstellen können.  
> 
> Im Artikel [Überlegungen zum Entwurf von Hybrid-Apps](overview-app-design-considerations.md) werden die wichtigen Aspekte in Bezug auf die Softwarequalität (Platzierung, Skalierbarkeit, Verfügbarkeit, Resilienz, Verwaltbarkeit und Sicherheit) beschrieben, die für das Entwerfen, Bereitstellen und Betreiben von Hybrid-Apps erforderlich sind. Die Überlegungen zum Entwurf dienen als Hilfe beim Optimieren des Designs von Hybrid-Apps, um für Produktionsumgebungen das Auftreten von Problemen zu minimieren.

## <a name="prerequisites"></a>Voraussetzungen

Schritte vor dem Beginnen mit diesem Bereitstellungsleitfaden:

- Lesen Sie den Artikel [Kubernetes-Clustermuster mit Hochverfügbarkeit](pattern-highly-available-kubernetes.md).
- Sehen Sie sich den Inhalt des [zugehörigen GitHub-Repositorys](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub) an, das zusätzliche Ressourcen enthält, auf die in diesem Artikel verwiesen wird.
- Sie benötigen ein Konto, mit dem sie auf das [Azure Stack Hub-Benutzerportal](/azure-stack/user/azure-stack-use-portal) zugreifen können und das mindestens die [Berechtigung „Mitwirkender“](/azure-stack/user/azure-stack-manage-permissions) hat.

## <a name="download-and-prepare-aks-engine"></a>Herunterladen und Vorbereiten der AKS-Engine

Die AKS-Engine ist eine Binärdatei, die auf jedem Windows- oder Linux-Host verwendet werden kann, der die Azure Resource Manager-Endpunkte für Azure Stack Hub erreichen kann. In diesem Leitfaden wird beschrieben, wie Sie einen neuen virtuellen Linux- oder Windows-Computer in Azure Stack Hub bereitstellen. Er wird später verwendet, wenn die AKS-Engine die Kubernetes-Cluster bereitstellt.

> [!NOTE]
> Sie können auch einen vorhandenen virtuellen Windows- oder Linux-Computer verwenden, um einen Kubernetes-Cluster mithilfe der AKS-Engine in Azure Stack Hub bereitzustellen.

Die Schritt-für-Schritt-Anleitung und die Anforderungen für die AKS-Engine sind hier dokumentiert:

* [Installieren der AKS-Engine unter Linux in Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (oder mithilfe von [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))

Die AKS-Engine ist ein Hilfsprogramm zum Bereitstellen und Betreiben von (nicht verwalteten) Kubernetes-Clustern (in Azure und Azure Stack Hub).

Die Details und Unterschiede der AKS-Engine in Azure Stack Hub sind nachfolgend beschrieben:

* [Was ist die AKS-Engine in Azure Stack Hub?](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* [AKS-Engine in Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (auf GitHub)

In der Beispielumgebung wird Terraform zum Automatisieren der Bereitstellung der AKS-Engine-VM verwendet. Sie finden [die Details und den Code im zugehörigen GitHub-Repository](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).

Das Ergebnis dieses Schritts ist eine neue Ressourcengruppe in Azure Stack Hub, die die Hilfs-VM der AKS-Engine und zugehörige Ressourcen enthält:

![AKS-Engine-VM-Ressourcen in Azure Stack Hub](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> Wenn Sie die AKS-Engine in einer getrennten Air-Gap-Umgebung bereitstellen müssen, finden Sie unter [Getrennte Azure Stack Hub-Instanzen](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) weitere Informationen.

Im nächsten Schritt verwenden Sie die neu bereitgestellte AKS-Engine-VM zum Bereitstellen eines Kubernetes-Clusters.

## <a name="connect-to-the-aks-engine-helper-vm"></a>Herstellen einer Verbindung mit der Hilfs-VM der AKS-Engine

Zunächst müssen Sie eine Verbindung mit der zuvor erstellten Hilfs-VM der AKS-Engine herstellen.

Die VM sollte über eine öffentliche IP-Adresse verfügen und über SSH (Port 22/TCP) erreichbar sein.

![Übersichtsseite der AKS-Engine-VM](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> Sie können ein Tool Ihrer Wahl wie MobaXterm, puTTY oder PowerShell unter Windows 10 verwenden, um per SSH eine Verbindung mit einem virtuellen Linux-Computer herzustellen.

```console
ssh <username>@<ipaddress>
```

Führen Sie nach der Verbindungsherstellung den Befehl `aks-engine` aus. Unter [Unterstützte AKS-Engine-Versionen](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) finden Sie weitere Informationen zu den AKS-Engine- und Kubernetes-Versionen.

![Befehlszeilenbeispiel für „aks-engine“](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a>Bereitstellen eines Kubernetes-Clusters

Die Hilfs-VM der AKS-Engine selbst hat noch keinen Kubernetes-Cluster in Azure Stack Hub erstellt. Die Clustererstellung ist die erste Aktion, die auf der Hilfs-VM der AKS-Engine ausgeführt wird.

Die Vorgehensweise ist hier Schritt für Schritt dokumentiert:

* [Bereitstellen eines Kubernetes-Cluster mit der AKS-Engine in Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

Das Ergebnis des Befehls `aks-engine deploy` und der Vorbereitungen in den vorherigen Schritten ist ein Kubernetes-Cluster mit vollem Funktionsumfang, der im Mandantenraum der ersten Azure Stack Hub-Instanz bereitgestellt wird. Der Cluster selbst besteht aus Azure-IaaS-Komponenten wie virtuellen Computern, Lastenausgleichsmodulen, VNETs, Datenträgern usw.

![Cluster-IaaS-Komponenten: Azure Stack Hub-Portal](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) Azure-Lastenausgleich (K8s-API-Endpunkt)
2) Workerknoten (Agentpool)
3) Masterknoten

Der Cluster wird jetzt ordnungsgemäß ausgeführt. Im nächsten Schritt stellen Sie eine Verbindung mit dem Cluster her.

## <a name="connect-to-the-kubernetes-cluster"></a>Herstellen einer Verbindung mit dem Kubernetes-Cluster

Sie können nun eine Verbindung mit dem zuvor erstellten Kubernetes-Cluster herstellen. Verwenden Sie dazu entweder SSH (unter Verwendung des SSH-Schlüssels, der im Rahmen der Bereitstellung angegeben wurde), oder `kubectl` (empfohlen). Das Kubernetes-Befehlszeilentool `kubectl` steht [hier](https://kubernetes.io/docs/tasks/tools/install-kubectl/) für Windows, Linux und macOS zur Verfügung. Es ist bereits vorinstalliert und auf den Masterknoten des Clusters konfiguriert.

```console
ssh azureuser@<k8s-master-lb-ip>
```

![Ausführen von kubectl auf dem Masterknoten](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

Es wird nicht empfohlen, den Masterknoten als Jumpbox für administrative Aufgaben zu verwenden. Die `kubectl`-Konfiguration wird auf den Masterknoten in `.kube/config` sowie auf der AKS-Engine-VM gespeichert. Sie können die Konfiguration auf einen Administratorcomputer mit Verbindung mit dem Kubernetes-Cluster kopieren und dort den Befehl `kubectl` verwenden. Die Datei `.kube/config` wird außerdem später zum Konfigurieren einer Dienstverbindung in Azure Pipelines verwendet.

> [!IMPORTANT]
> Schützen Sie diese Dateien, da sie die Anmeldeinformationen für Ihren Kubernetes-Cluster enthalten. Ein Angreifer mit Zugriff auf die Datei verfügt über genügend Informationen, um Administratorzugriff zu erhalten. Alle Aktionen, die mit der anfänglichen Datei `.kube/config` durchgeführt werden, werden mithilfe eines Clusteradministratorkontos ausgeführt.

Sie können jetzt unter Verwendung von `kubectl` verschiedene Befehle ausprobieren, um den Status Ihres Clusters zu überprüfen.

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
> Kubernetes verfügt über ein eigenes Modell für die _ *rollenbasierte Zugriffssteuerung (Role-Based Access Control, RBAC)* *, mit dem Sie differenzierte Rollendefinitionen und Rollenbindungen erstellen können. Verwenden Sie nach Möglichkeit diese Methode, um den Zugriff auf den Cluster zu steuern, statt Clusteradministratorberechtigungen zuzuweisen.

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a>Herstellen einer Verbindung zwischen Azure Pipelines und Kubernetes-Clustern

Zum Herstellen einer Verbindung zwischen Azure Pipelines und dem neu bereitgestellten Kubernetes-Cluster benötigen Sie die Datei mit der Kube-Konfiguration (`.kube/config`), wie im vorherigen Schritt erläutert.

* Stellen Sie eine Verbindung mit einem der Masterknoten Ihres Kubernetes-Clusters her.
* Kopieren Sie den Inhalt der Datei `.kube/config`.
* Navigieren Sie zu „Azure DevOps“ > „Projekteinstellungen“ > „Dienstverbindungen“, um eine neue Kubernetes-Dienstverbindung zu erstellen. (Verwenden Sie „KubeConfig“ als Authentifizierungsmethode.)

> [!IMPORTANT]
> Azure Pipelines (oder die zugehörigen Build-Agents) müssen Zugriff auf die Kubernetes-API haben. Besteht eine Internetverbindung zwischen Azure Pipelines und dem Azure Stack Hub-Kubernetes-Cluster, müssen Sie einen selbstgehosteten Azure Pipelines-Build-Agent bereitstellen.

Beim Bereitstellen von selbstgehosteten Agents für Azure Pipelines können Sie die Bereitstellung entweder in Azure Stack Hub oder auf einem Computer mit Netzwerkkonnektivität mit allen erforderlichen Verwaltungsendpunkten durchführen. Ausführliche Informationen finden Sie hier:

* [Azure Pipelines-Agents](/azure/devops/pipelines/agents/agents) unter [Windows](/azure/devops/pipelines/agents/v2-windows) oder [Linux](/azure/devops/pipelines/agents/v2-linux)

Der Abschnitt [Bereitstellungsaspekte (CI/CD)](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) enthält einen Entscheidungsfluss, anhand dessen Sie nachvollziehen können, ob Sie von Microsoft gehostete Agents oder selbstgehostete Agents verwenden sollten.

[![Entscheidungsfluss für selbstgehostete Agents](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)

In dieser Beispiellösung enthält die Topologie einen selbstgehosteten Build-Agent in jeder Azure Stack Hub-Instanz. Der Agent kann auf die Azure Stack Hub-Verwaltungsendpunkte und die Kubernetes-Cluster-API-Endpunkte zugreifen.

[![Nur ausgehender Datenverkehr](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)

Dieser Entwurf erfüllt eine gängige gesetzliche Anforderung, die vorgibt, dass nur ausgehende Verbindungen von der Anwendungslösung hergestellt werden dürfen.

## <a name="configure-monitoring"></a>Konfigurieren der Überwachung

Sie können [Azure Monitor](/azure/azure-monitor/) für Container verwenden, um die Container in der Lösung zu überwachen. Dadurch wird Azure Monitor auf den von der AKS-Engine bereitgestellten Kubernetes-Cluster in Azure Stack Hub verwiesen.

Um Azure Monitor für Ihren Cluster zu aktivieren, stehen Ihnen zwei Methoden zur Auswahl. Beide Methoden erfordern, dass Sie einen Log Analytics-Arbeitsbereich in Azure einrichten.

* [Methode 1](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) verwendet ein Helm-Chart.
* [Methode 2](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) ist Teil der AKS-Engine-Clusterspezifikation.

In der Beispieltopologie wird Methode 1 verwendet, die die Automatisierung des Prozesses und eine einfachere Installation von Updates ermöglicht.

Für den nächsten Schritt benötigen Sie einen Azure Log Analytics-Arbeitsbereich (ID und Schlüssel), `Helm` (Version 3) und `kubectl` auf Ihrem Computer.

Helm ist ein Kubernetes-Paket-Manager, der als Binärdatei verfügbar ist und unter macOS, Windows und Linux ausgeführt wird. Er kann hier heruntergeladen werden: [helm.sh](https://helm.sh/docs/intro/quickstart/). Helm nutzt die Kubernetes-Konfigurationsdatei, die für den `kubectl`-Befehl verwendet wird.

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

Mit diesem Befehl wird der Azure Monitor-Agent in Ihrem Kubernetes-Cluster installiert:

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

Der OMS-Agent (Operations Management Suite) in Ihrem Kubernetes-Cluster sendet Überwachungsdaten an Ihren Azure Log Analytics-Arbeitsbereich (über eine ausgehende HTTPS-Verbindung). Sie können nun Azure Monitor nutzen, um umfangreichere Informationen zu Ihren Kubernetes-Clustern in Azure Stack Hub zu erhalten. Dieser Entwurf ist eine leistungsstarke Möglichkeit, die Leistungsfähigkeit von Analysen zu veranschaulichen, die automatisch mit den Clustern Ihrer Anwendung bereitgestellt werden können.

[![Azure Stack Hub-Cluster in Azure Monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)

[![Details zu Azure Monitor-Clustern](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)

> [!IMPORTANT]
> Wenn in Azure Monitor keine Azure Stack Hub-Daten angezeigt werden, vergewissern Sie sich, dass Sie die Anweisungen zum [Hinzufügen der Lösung „AzureMonitor-Containers“ zu einem Azure Log Analytics-Arbeitsbereich](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) befolgt haben.

## <a name="deploy-the-application"></a>Bereitstellen der Anwendung

Vor der Installation der Beispielanwendung ist ein weiterer Schritt zum Konfigurieren des NGINX-basierten Eingangsdatencontrollers im Kubernetes-Cluster erforderlich. Der Eingangsdatencontroller wird als Layer-7-Lastenausgleich verwendet, um den Datenverkehr im Cluster auf der Grundlage von Host, Pfad oder Protokoll weiterzuleiten. „nginx-ingress“ ist als Helm-Chart verfügbar. Ausführliche Anweisungen finden Sie im [GitHub-Repository mit dem Helm-Chart](https://github.com/helm/charts/tree/master/stable/nginx-ingress).

Die Beispielanwendung wird ebenfalls als Helm-Chart gepackt, etwa der [Azure-Überwachungs-Agent](#configure-monitoring) im vorherigen Schritt. Daher ist es einfach, die Anwendung im Kubernetes-Cluster bereitzustellen. Sie finden die [Helm-Chart-Dateien im zugehörigen GitHub-Repository](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm).

Bei der Beispielanwendung handelt es sich um eine Anwendung mit drei Ebenen, die in einem Kubernetes-Cluster in zwei Azure Stack Hub-Instanzen bereitgestellt wird. Die Anwendung nutzt eine MongoDB-Datenbank. Weitere Informationen zum Replizieren der Daten über mehrere Instanzen hinweg finden Sie im Muster [Überlegungen zu Daten und Speicher](pattern-highly-available-kubernetes.md#data-and-storage-considerations).

Nach der Bereitstellung des Helm-Charts für die Anwendung werden alle drei Ebenen der Anwendung als Bereitstellungen und StatefulSets (für die Datenbank) mit einem einzelnen Pod dargestellt:

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

Auf der Dienstseite finden Sie den NGINX-basierten Eingangsdatencontroller und seine öffentliche IP-Adresse:

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

Die externe IP-Adresse ist der Anwendungsendpunkt. Damit wird festgelegt, wie Benutzer eine Verbindung zum Öffnen der Anwendung herstellen, und er wird außerdem als Endpunkt für den nächsten Schritt ([Konfigurieren von Traffic Manager](#configure-traffic-manager)) verwendet.

## <a name="autoscale-the-application"></a>Autoskalieren der Anwendung
Optional können Sie die [horizontale automatische Podskalierung](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) (Horizontal Pod AutoScaler, HPA) konfigurieren, um basierend auf bestimmten Metriken wie CPU-Auslastung zentral hoch- oder herunterzuskalieren. Mit dem folgenden Befehl wird eine horizontale automatische Podskalierung erstellt, die ein bis zehn Replikate der Pods verwaltet, die von der Bereitstellung „ratings-web“ gesteuert werden. Die HPA erhöht und verringert die Anzahl von Replikaten (über die Bereitstellung), um eine durchschnittliche CPU-Auslastung von 80 Prozent über alle Pods hinweg aufrechtzuerhalten.

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
Sie können den aktuellen Status der Autoskalierung überprüfen, indem Sie Folgendes ausführen:

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a>Traffic Manager konfigurieren

Zum Verteilen des Datenverkehrs zwischen zwei (oder mehr) Bereitstellungen der Anwendung verwenden Sie [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview). Azure Traffic Manager ist ein auf DNS basierender Lastenausgleichsdienst in Azure.

> [!NOTE]
> Traffic Manager verwendet DNS, um Clientanforderungen auf der Grundlage einer Datenverkehrsrouting-Methode und der Integrität der Endpunkte an den optimalen Endpunkt weiterzuleiten.

Anstelle von Azure Traffic Manager können Sie auch andere globale Lastenausgleichslösungen verwenden, die lokal gehostet werden. Im Beispielszenario verwenden Sie Azure Traffic Manager, um den Datenverkehr zwischen zwei Instanzen Ihrer Anwendung zu verteilen. Diese können in Azure Stack Hub-Instanzen am gleichen Standort oder an verschiedenen Standorten ausgeführt werden:

![Lokale Traffic Manager-Instanz](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

Sie konfigurieren Traffic Manager in Azure, um auf die beiden unterschiedlichen Instanzen der Anwendung zu verweisen:

[![TM-Endpunktprofil](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)

Wie Sie sehen, verweisen die beiden Endpunkte auf die beiden Instanzen der bereitgestellten Anwendung aus dem [vorherigen Abschnitt](#deploy-the-application).

Zu diesem Zeitpunkt gilt Folgendes:
- Die Kubernetes-Infrastruktur, einschließlich eines Eingangsdatencontrollers, wurde erstellt.
- Cluster wurden in zwei Azure Stack Hub-Instanzen bereitgestellt.
- Die Überwachung wurde konfiguriert.
- Azure Traffic Manager nimmt einen Lastausgleich für den Datenverkehr zwischen den beiden Azure Stack Hub-Instanzen vor.
- Zusätzlich zu dieser Infrastruktur wurde die Beispielanwendung mit drei Ebenen mithilfe von Helm-Charts automatisiert bereitgestellt. 

Die Lösung sollte nun einsatzbereit und für Benutzer verfügbar sein.

Außerdem gibt es einige wichtige Überlegungen zum Betrieb nach der Bereitstellung, die in den nächsten zwei Abschnitten behandelt werden.

## <a name="upgrade-kubernetes"></a>Kubernetes aktualisieren

Beachten Sie bei der Aktualisierung des Kubernetes-Clusters die folgenden Punkte:

- Das Upgrade eines Kubernetes-Clusters ist ein komplexer Vorgang am 2. Tag, der mithilfe der AKS-Engine ausgeführt werden kann. Weitere Informationen finden Sie unter [Aktualisieren eines Kubernetes-Clusters in Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).
- Mithilfe der AKS-Engine können Sie Cluster auf neuere Versionen von Kubernetes und des Betriebssystem-Basisimages aktualisieren. Weitere Informationen finden Sie unter [Schritte zum Durchführen eines Upgrades auf eine neuere Kubernetes-Version](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version). 
- Sie können auch nur die zugrunde liegenden Knoten auf neuere Versionen des Betriebssystem-Basisimages aktualisieren. Weitere Informationen finden Sie unter [Schritte zum alleinigen Upgrade des Betriebssystemimages](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).

Neuere Betriebssystem-Basisimages enthalten Sicherheits- und Kernelupdates. Es liegt in der Verantwortung des Clusteroperators, die Verfügbarkeit von neueren Kubernetes- und Betriebssystemversionen zu überwachen. Der Operator sollte diese Upgrades mithilfe der AKS-Engine planen und ausführen. Die Betriebssystem-Basisimages müssen vom Azure Stack Hub-Operator aus dem Azure Stack Hub-Marketplace heruntergeladen werden.

## <a name="scale-kubernetes"></a>Skalieren von Kubernetes

Die Skalierung ist ein weiterer Vorgang am 2. Tag, der mithilfe der AKS-Engine orchestriert werden kann.

Vom Skalierungsbefehl wird Ihre Clusterkonfigurationsdatei (apimodel.json) im Ausgabeverzeichnis als Eingabe für eine neue Azure Resource Manager-Bereitstellung wiederverwendet. Die AKS-Engine führt den Skalierungsvorgang für einen bestimmten Agentpool aus. Wenn der Skalierungsvorgang beendet ist, aktualisiert die AKS-Engine die Clusterdefinition in derselben Datei vom Typ „apimodel.json“. Die Clusterdefinition spiegelt die neue Knotenanzahl und damit die aktualisierte (derzeitige) Clusterkonfiguration wider.

- [Skalieren eines Kubernetes-Cluster in Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a>Nächste Schritte

- Erfahren Sie mehr über [Überlegungen zum Entwurf von Hybrid-Apps](overview-app-design-considerations.md).
- Überprüfen Sie den [Code dieses Beispiels auf GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), und schlagen Sie Verbesserungen vor.