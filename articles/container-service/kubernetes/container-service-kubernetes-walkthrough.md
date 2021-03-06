---
title: Snelstartgids - Azure Kubernetes-cluster voor Linux | Microsoft Docs
description: Leer snel een Kubernetes-cluster te maken voor Linux-containers in Azure Container Service met de Azure CLI.
services: container-service
documentationcenter: 
author: neilpeterson
manager: timlt
editor: 
tags: acs, azure-container-service, kubernetes
keywords: 
ms.assetid: 8da267e8-2aeb-4c24-9a7a-65bdca3a82d6
ms.service: container-service
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/25/2017
ms.author: nepeters
ms.custom: H1Hack27Feb2017
ms.translationtype: HT
ms.sourcegitcommit: bfd49ea68c597b109a2c6823b7a8115608fa26c3
ms.openlocfilehash: 51c70dcacfba82255532f3222ecb391a43eccbb4
ms.contentlocale: nl-nl
ms.lasthandoff: 07/25/2017

---

# <a name="deploy-kubernetes-cluster-for-linux-containers"></a>Kubernetes-cluster voor Linux-containers implementeren

In deze snelstartgids wordt een Kubernetes-cluster geïmplementeerd met behulp van de Azure CLI. Vervolgens wordt er een toepassing met meerdere containers uitgevoerd die bestaat uit een web-front-end en een Redis-exemplaar op het cluster. Zodra de toepassing is voltooid, is deze toegankelijk via internet.

![Afbeelding van browsen naar Azure Vote](media/container-service-kubernetes-walkthrough/azure-vote.png)

In deze snelstartgids wordt ervan uitgegaan dat u over basiskennis van Kubernetes-concepten beschikt. Zie de [Kubernetes-documentatie]( https://kubernetes.io/docs/home/) voor gedetailleerde informatie over Kubernetes.

Als u nog geen abonnement op Azure hebt, maak dan een [gratis account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) aan voordat u begint.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Als u ervoor kiest om de CLI lokaal te installeren en te gebruiken, moet u voor deze Quickstart gebruikmaken van Azure CLI versie 2.0.4 of hoger. Voer `az --version` uit om de versie te bekijken. Als u Azure CLI 2.0 wilt installeren of upgraden, raadpleegt u [Azure CLI 2.0 installeren]( /cli/azure/install-azure-cli). 

## <a name="create-a-resource-group"></a>Een resourcegroep maken

Een resourcegroep maken met de opdracht [az group create](/cli/azure/group#create). Een Azure-resourcegroep is een logische groep waarin Azure-resources worden geïmplementeerd en beheerd. 

In het volgende voorbeeld wordt een resourcegroep met de naam *myResourceGroup* gemaakt op de locatie *VS Oost*.

```azurecli-interactive 
az group create --name myResourceGroup --location eastus
```

Uitvoer:

```json
{
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup",
  "location": "eastus",
  "managedBy": null,
  "name": "myResourceGroup",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null
}
```

## <a name="create-kubernetes-cluster"></a>Een Kubernetes-cluster maken

Maak een Kubernetes-cluster in Azure Container Service met de opdracht [az acs create](/cli/azure/acs#create). In het volgende voorbeeld wordt een cluster gemaakt met de naam *myK8sCluster* met een Linux-hoofdknooppunt en drie knooppunten van de Linux-agent.

```azurecli-interactive 
az acs create --orchestrator-type=kubernetes --resource-group myResourceGroup --name=myK8sCluster --generate-ssh-keys 
```

Na enkele minuten is de opdracht voltooid en retourneert deze informatie over het cluster in json-indeling. 

## <a name="connect-to-the-cluster"></a>Verbinding maken met het cluster

Als u een Kubernetes-cluster wilt beheren, gebruikt u [kubectl](https://kubernetes.io/docs/user-guide/kubectl/), de Kubernetes-opdrachtregelclient. 

Als u Azure CloudShell gebruikt, is kubectl al geïnstalleerd. Als u lokaal wilt installeren, kunt u de opdracht [az acs kubernetes install-cli](/cli/azure/acs/kubernetes#install-cli) gebruiken.

Als u kubectl zo wilt configureren dat de client verbinding maakt met uw Kubernetes-cluster, voert u de opdracht [az acs kubernetes get-credentials](/cli/azure/acs/kubernetes#get-credentials) uit.

```azurecli-interactive 
az acs kubernetes get-credentials --resource-group=myResourceGroup --name=myK8sCluster
```

Als u de verbinding met uw cluster wilt controleren, gebruikt u de opdracht [kubectl get](https://kubernetes.io/docs/user-guide/kubectl/v1.6/#get) om een lijst met clusterknooppunten te retourneren.

```azurecli-interactive
kubectl get nodes
```

Uitvoer:

```bash
NAME                    STATUS                     AGE       VERSION
k8s-agent-14ad53a1-0    Ready                      10m       v1.6.6
k8s-agent-14ad53a1-1    Ready                      10m       v1.6.6
k8s-agent-14ad53a1-2    Ready                      10m       v1.6.6
k8s-master-14ad53a1-0   Ready,SchedulingDisabled   10m       v1.6.6
```

## <a name="run-the-application"></a>De toepassing uitvoeren

Een Kubernetes-manifestbestand definieert een gewenste status voor het cluster, inclusief zaken zoals welke containerinstallatiekopieën moeten worden uitgevoerd. In dit voorbeeld worden met behulp van een manifest alle objecten gemaakt die nodig zijn om de Azure Vote-toepassing uit te voeren. 

Maak een bestand met de naam `azure-vote.yaml` en kopieer hierin de volgende YAML.

```yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: azure-vote-back
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: azure-vote-back
    spec:
      containers:
      - name: azure-vote-back
        image: redis
        ports:
        - containerPort: 6379
          name: redis
---
apiVersion: v1
kind: Service
metadata:
  name: azure-vote-back
spec:
  ports:
  - port: 6379
  selector:
    app: azure-vote-back
---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: azure-vote-front
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: azure-vote-front
    spec:
      containers:
      - name: azure-vote-front
        image: microsoft/azure-vote-front:redis-v1
        ports:
        - containerPort: 80
        env:
        - name: REDIS
          value: "azure-vote-back"
---
apiVersion: v1
kind: Service
metadata:
  name: azure-vote-front
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: azure-vote-front
```

Gebruik de opdracht [kubectl create](https://kubernetes.io/docs/user-guide/kubectl/v1.6/#create) om de toepassing uit te voeren.

```azurecli-interactive
kubectl create -f azure-vote.yaml
```

Uitvoer:

```bash
deployment "azure-vote-back" created
service "azure-vote-back" created
deployment "azure-vote-front" created
service "azure-vote-front" created
```

## <a name="test-the-application"></a>De toepassing testen

Terwijl de toepassing wordt uitgevoerd, wordt er een [Kubernetes-service](https://kubernetes.io/docs/concepts/services-networking/service/) gemaakt die de front-end van de toepassing beschikbaar maakt op internet. Dit proces kan enkele minuten duren. 

Gebruik de opdracht [kubectl get service](https://kubernetes.io/docs/user-guide/kubectl/v1.6/#get) met het argument `--watch` om de voortgang te controleren.

```azurecli-interactive
kubectl get service azure-vote-front --watch
```

Eerst wordt het **externe IP-adres** voor de service *azure-vote-front* weergegeven als *in behandeling*. Nadat het externe IP-adres is gewijzigd van *in behandeling* naar een *IP-adres*, gebruikt u `CTRL-C` om het controleproces van kubectl te stoppen. 
  
```bash
azure-vote-front   10.0.34.242   <pending>     80:30676/TCP   7s
azure-vote-front   10.0.34.242   52.179.23.131   80:30676/TCP   2m
```

Nu kunt u naar het externe IP-adres browsen voor een overzicht van de Azure Vote-toepassing.

![Afbeelding van browsen naar Azure Vote](media/container-service-kubernetes-walkthrough/azure-vote.png)  

## <a name="delete-cluster"></a>Cluster verwijderen
U kunt de opdracht [az group delete](/cli/azure/group#delete) gebruiken om de resourcegroep, de containerservice en alle gerelateerde resources te verwijderen wanneer u het cluster niet meer nodig hebt.

```azurecli-interactive 
az group delete --name myResourceGroup --yes --no-wait
```

## <a name="get-the-code"></a>Code ophalen

In deze snelstartgids zijn vooraf gemaakte containerinstallatiekopieën gebruikt om een Kubernetes-implementatie te maken. De gerelateerde toepassingscode, Dockerfile en het Kubernetes-manifestbestand zijn beschikbaar op GitHub.

[Azure Vote-toepassing met Redis](https://github.com/Azure-Samples/azure-voting-app-redis.git)

## <a name="next-steps"></a>Volgende stappen

In deze snelstartgids hebt u een Kubernetes-cluster geïmplementeerd en vervolgens een toepassing met meerdere containers naar het cluster geïmplementeerd. 

Voor meer informatie over Azure Container Service en een volledig voorbeeld van code tot implementatie gaat u naar de zelfstudie Kubernetes-cluster.

> [!div class="nextstepaction"]
> [Een ACS Kubernetes-cluster beheren](./container-service-tutorial-kubernetes-prepare-app.md)
