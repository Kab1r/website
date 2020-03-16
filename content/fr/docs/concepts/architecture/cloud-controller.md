---
title: Concepts sous-jacents au Cloud Controller Manager
content_template: templates/concept
weight: 30
---

{{% capture overview %}}

Le concept de cloud controller manager (CCM) (ne pas confondre avec le binaire)
a été créé à l'origine pour permettre au code de fournisseur spécifique de cloud
et au noyau Kubernetes d'évoluer indépendamment les uns des autres. Le
gestionnaire de contrôleur de cloud fonctionne aux côtés d'autres composants
principaux, tels que le gestionnaire de contrôleur Kubernetes, le serveur d'API
et le planificateur. Il peut également être démarré en tant qu’addon Kubernetes,
auquel cas il s’exécute sur Kubernetes.

La conception du gestionnaire de contrôleur de cloud repose sur un mécanisme de
plugin qui permet aux nouveaux fournisseurs de cloud de s'intégrer facilement �
Kubernetes à l'aide de plugins. Des plans sont en place pour intégrer de
nouveaux fournisseurs de cloud sur Kubernetes et pour migrer les fournisseurs de
cloud de l'ancien modèle vers le nouveau modèle CCM.

Ce document discute des concepts derrière le cloud controller manager et donne
des détails sur ses fonctions associées.

Voici l'architecture d'un cluster Kubernetes sans le cloud controller manager:

![Pre CCM Kube Arch](/images/docs/pre-ccm-arch.png)

{{% /capture %}}

{{% capture body %}}

## Conception

Dans le diagramme précédent, Kubernetes et le fournisseur de cloud sont intégrés
via plusieurs composants différents:

- Kubelet
- Kubernetes controller manager
- Kubernetes API server

Le CCM consolide toute la logique dépendante du cloud des trois composants
précédents pour créer un point d’intégration unique avec le cloud. La nouvelle
architecture avec le CCM se présente comme suit:

![CCM Kube Arch](/images/docs/post-ccm-arch.png)

## Composants du CCM

Le CCM rompt certaines fonctionnalités du Kubernetes Controller Manager (KCM) et
les exécute en tant que processus séparé. Plus précisément, il sépare les
contrôleurs du KCM qui dépendent du cloud. Le KCM comporte les boucles de
contrôle dépendant du cloud suivantes:

- Contrôleur de nœud
- Contrôleur de volume
- Contrôleur de route
- Contrôleur de service

Dans la version 1.9, le CCM exécute les contrôleurs suivants de la liste
précédente:

- Contrôleur de nœud
- Contrôleur de route
- Contrôleur de service

{{< note >}} Le contrôleur de volume a été délibérément choisi pour ne pas faire
partie de CCM. En raison de la complexité du processus et des efforts déployés
pour supprimer la logique de volume spécifique au fournisseur, il a été décidé
que le contrôleur de volume ne serait pas déplacé vers CCM. {{< /note >}}

Le plan initial de prise en charge des volumes à l'aide de CCM consistait �
utiliser des volumes Flex pour prendre en charge des volumes pouvant être
connectés. Cependant, un effort concurrentiel appelé CSI est prévu pour
remplacer Flex.

Compte tenu de cette dynamique, nous avons décidé d'avoir une mesure de
transition intermédiaire jusqu'à ce que le CSI soit prêt.

## Fonctions du CCM

Le CCM hérite ses fonctions des composants de Kubernetes qui dépendent d'un
fournisseur de cloud. Cette section est structurée en fonction de ces
composants.

### 1. Kubernetes controller manager

La majorité des fonctions du CCM sont dérivées du KCM. Comme indiqué dans la
section précédente, le CCM exécute les boucles de contrôle suivantes:

- Contrôleur de nœud
- Contrôleur de route
- Contrôleur de service

#### Contrôleur de nœud

Le contrôleur de noeud est responsable de l'initialisation d'un noeud en
obtenant des informations sur les noeuds s'exécutant dans le cluster auprès du
fournisseur de cloud. Le contrôleur de noeud exécute les fonctions suivantes:

1. Il initialise le nœud avec des labels de zone/région spécifiques au cloud.
2. Il initialise le nœud avec des détails d'instance spécifiques au cloud, tels
   que le type et la taille de l'instance.
3. Il obtient les adresses réseau et le nom d'hôte du nœud.
4. Si un nœud ne répond plus, le controlleur vérifie avec le cloud pour voir
   s'il a été supprimé du cloud. Si le nœud a été supprimé du cloud, le
   controlleur supprime l'objet Kubernetes Node.

#### Contrôleur de route

Le contrôleur de route est responsable de la configuration appropriée des
itinéraires dans le cloud afin que les conteneurs situés sur différents nœuds du
cluster Kubernetes puissent communiquer entre eux. Le contrôleur de route ne
s'applique qu'aux clusters Google Compute Engine.

#### Contrôleur de service

Le contrôleur de service est chargé d'écouter les événements de création, de
mise à jour et de suppression de service. En fonction de l'état actuel des
services dans Kubernetes, il configure les équilibreurs de charge dans le cloud
(tels que ELB, Google LB ou Oracle Cloud Infrastructure LB) pour refléter l'état
des services dans Kubernetes. De plus, cela garantit que les services de base
des services pour les load balancers dans le cloud sont à jour.

### 2. Kubelet

Le contrôleur de noeud contient les fonctionnalités du kubelet dépendant du
cloud. Avant l'introduction du CCM, la sous-unité était responsable de
l'initialisation d'un nœud avec des détails spécifiques au cloud, tels que les
adresses IP, les étiquettes de région / zone et les informations de type
d'instance. L’introduction du CCM a déplacé cette opération d’initialisation du
kubelet vers le CCM.

Dans ce nouveau modèle, le kubelet initialise un nœud sans informations
spécifiques au cloud. Cependant, il ajoute un marquage au nœud nouvellement
créé, qui rend le nœud non planifiable jusqu'à ce que le CCM initialise le nœud
avec des informations spécifiques au cloud. Il supprime ensuite ce marquage.

## Mécanisme de plugin

Le cloud controller manager utilise des interfaces Go pour autoriser la mise en
œuvre d'implémentations depuis n'importe quel cloud. Plus précisément, il
utilise l'interface CloudProvider définie
[ici](https://github.com/kubernetes/cloud-provider/blob/9b77dc1c384685cb732b3025ed5689dd597a5971/cloud.go#L42-L62).

La mise en œuvre des quatre contrôleurs partagés soulignés ci-dessus, ainsi que
certaines configurations ainsi que l'interface partagée du fournisseur de cloud,
resteront dans le noyau Kubernetes. Les implémentations spécifiques aux
fournisseurs de cloud seront construites en dehors du noyau et implémenteront
les interfaces définies dans le noyau.

Pour plus d’informations sur le développement de plugins, consultez
[Developing Cloud Controller Manager](/docs/tasks/administer-cluster/developing-cloud-controller-manager/).

## Autorisation

Cette section détaille les accès requis par le CCM sur divers objets API pour
effectuer ses opérations.

### Contrôleur de nœud

Le contrôleur de noeud ne fonctionne qu'avec les objets de noeud. Il nécessite
un accès complet aux objets Node via get, list, create, update, patch, watch et
delete.

v1/Node:

- Get
- List
- Create
- Update
- Patch
- Watch
- Delete

### Contrôleur de route

Le contrôleur de route écoute les évenements de création d'objet Node et
configure les routes de manière appropriée. Cela nécessite un accès get aux
objets Node.

v1/Node:

- Get

### Contrôleur de Service

Le contrôleur de service écoute les évenements de création, suppression et mises
à jour des objets Service et configure les endpoints pour ces Services de
manière appropriée.

Pour accéder aux Services, il faut les accès list et watch. Pour mettre à jour
les Services, il faut les accès patch et update.

Pour configurer des points de terminaison pour les services, vous devez avoir
accès au create, list, get, watch, et update.

v1/Service:

- List
- Get
- Watch
- Patch
- Update

### Autres

La mise en œuvre du CCM nécessite un accès pour créer des événements, et pour
garantir un fonctionnement sécurisé, un accès est nécessaire pour créer
ServiceAccounts.

v1/Event:

- Create
- Patch
- Update

v1/ServiceAccount:

- Create

Le ClusterRole RBAC pour le CCM ressemble à ceci:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cloud-controller-manager
rules:
  - apiGroups:
      - ""
    resources:
      - events
    verbs:
      - create
      - patch
      - update
  - apiGroups:
      - ""
    resources:
      - nodes
    verbs:
      - "*"
  - apiGroups:
      - ""
    resources:
      - nodes/status
    verbs:
      - patch
  - apiGroups:
      - ""
    resources:
      - services
    verbs:
      - list
      - patch
      - update
      - watch
  - apiGroups:
      - ""
    resources:
      - serviceaccounts
    verbs:
      - create
  - apiGroups:
      - ""
    resources:
      - persistentvolumes
    verbs:
      - get
      - list
      - update
      - watch
  - apiGroups:
      - ""
    resources:
      - endpoints
    verbs:
      - create
      - get
      - list
      - watch
      - update
```

## Implémentations des fournisseurs de cloud

Les fournisseurs de cloud suivants ont implémenté leur CCM:

- [Digital Ocean](https://github.com/digitalocean/digitalocean-cloud-controller-manager)
- [Oracle](https://github.com/oracle/oci-cloud-controller-manager)
- [Azure](https://github.com/kubernetes/cloud-provider-azure)
- [GCP](https://github.com/kubernetes/cloud-provider-gcp)
- [AWS](https://github.com/kubernetes/cloud-provider-aws)
- [BaiduCloud](https://github.com/baidu/cloud-provider-baiducloud)
- [Linode](https://github.com/linode/linode-cloud-controller-manager)
- [Scaleway](https://github.com/scaleway/scaleway-cloud-controller-manager)

## Administration de cluster

Des instructions complètes pour la configuration et l'exécution du CCM sont
fournies
[ici](/docs/tasks/administer-cluster/running-cloud-controller/#cloud-controller-manager).

{{% /capture %}}
