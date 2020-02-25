---
title:
  Installer Kubernetes avec Kubespray (on-premises et fournisseurs de cloud)
description: Installation de Kubernetes avec Kubespray
content_template: templates/concept
---

{{% capture overview %}}

Cette documentation permet d'installer rapidement un cluster Kubernetes hébergé
sur GCE, Azure, Openstack, AWS, vSphere, Oracle Cloud Infrastructure
(expérimental) ou sur des serveurs physiques (bare metal) grâce �
[Kubespray](https://github.com/kubernetes-incubator/kubespray).

Kubespray se base sur des outils de provisioning, des
[paramètres](https://github.com/kubernetes-incubator/kubespray/blob/master/docs/ansible.md)
et playbooks [Ansible](http://docs.ansible.com/) ainsi que sur des connaissances
spécifiques à Kubernetes et l'installation de systèmes d'exploitation afin de
fournir:

- Un cluster en haute disponibilité
- des composants modulables
- Le support des principales distributions Linux:
  - Container Linux de CoreOS
  - Debian Jessie, Stretch, Wheezy
  - Ubuntu 16.04, 18.04
  - CentOS/RHEL 7
  - Fedora/CentOS Atomic
  - openSUSE Leap 42.3/Tumbleweed
- des tests d'intégration continue

Afin de choisir l'outil le mieux adapté à votre besoin, veuillez lire
[cette comparaison](https://github.com/kubernetes-incubator/kubespray/blob/master/docs/comparisons.md)
avec [kubeadm](/docs/admin/kubeadm/) et [kops](../kops).

{{% /capture %}}

{{% capture body %}}

## Créer un cluster

### (1/5) Prérequis

Les serveurs doivent être installés en s'assurant des éléments suivants:

- **Ansible v2.6 (ou version plus récente) et python-netaddr installés sur la
  machine qui exécutera les commandes Ansible**
- **Jinja 2.9 (ou version plus récente) est nécessaire pour exécuter les
  playbooks Ansible**
- Les serveurs cibles doivent avoir **accès à Internet** afin de télécharger les
  images Docker. Autrement, une configuration supplémentaire est nécessaire, (se
  référer �
  [Offline Environment](https://github.com/kubernetes-sigs/kubespray/blob/master/docs/downloads.md#offline-environment))
- Les serveurs cibles doivent être configurés afin d'autoriser le transfert IPv4
  (**IPv4 forwarding**)
- **Votre clé ssh doit être copiée** sur tous les serveurs faisant partie de
  votre inventaire Ansible.
- La configuration du **pare-feu n'est pas gérée**. Vous devrez vous en charger
  en utilisant votre méthode habituelle. Afin d'éviter tout problème pendant
  l'installation nous vous conseillons de le désacttiver.
- Si Kubespray est exécuté avec un utilisateur autre que "root", une méthode
  d'autorisation appropriée devra être configurée sur les serveurs cibles
  (exemple: sudo). Il faudra aussi utiliser le paramètre `ansible_become` ou
  ajouter `--become` ou `b` à la ligne de commande.

Afin de vous aider à préparer votre de votre environnement, Kubespray fournit
les outils suivants:

- Scripts [Terraform](https://www.terraform.io/) pour les fournisseurs de cloud
  suivants:
  - [AWS](https://github.com/kubernetes-incubator/kubespray/tree/master/contrib/terraform/aws)
  - [OpenStack](https://github.com/kubernetes-incubator/kubespray/tree/master/contrib/terraform/openstack)

### (2/5) Construire un fichier d'inventaire Ansible

Lorsque vos serveurs sont disponibles, créez un fichier d'inventaire Ansible
([inventory](http://docs.ansible.com/ansible/intro_inventory.html)). Vous pouvez
le créer manuellement ou en utilisant un script d'inventaire dynamique. Pour
plus d'informations se référer �
[Building your own inventory](https://github.com/kubernetes-incubator/kubespray/blob/master/docs/getting-started.md#building-your-own-inventory).

### (3/5) Préparation au déploiement de votre cluster

Kubespray permet de personnaliser de nombreux éléments:

- Choix du mode: kubeadm ou non-kubeadm
- Plugins CNI (réseau)
- Configuration du DNS
- Choix du control plane: natif/binaire ou dans un conteneur docker/rkt
- Version de chaque composant
- "route reflectors" Calico
- Moteur de conteneur
  - docker
  - rkt
  - cri-o
- Méthode de génération des certificats (**Vault n'étant plus maintenu**)

Ces paramètres Kubespray peuvent être définis dans un fichier de
[variables](http://docs.ansible.com/ansible/playbooks_variables.html). Si vous
venez juste de commencer à utiliser Kubespray nous vous recommandons d'utiliser
les paramètres par défaut pour déployer votre cluster et découvrir Kubernetes

### (4/5) Déployer un Cluster

Vous pouvez ensuite lancer le déploiement de votre cluster:

Déploiement du cluster en utilisant l'outil en ligne de commande
[ansible-playbook](https://github.com/kubernetes-incubator/kubespray/blob/master/docs/getting-started.md#starting-custom-deployment).

```shell
ansible-playbook -i your/inventory/hosts.ini cluster.yml -b -v \
  --private-key=~/.ssh/private_key
```

Pour des déploiements plus importants (>100 noeuds) quelques
[ajustements](https://github.com/kubernetes-incubator/kubespray/blob/master/docs/large-deployments.md)
peuvent être nécessaires afin d'obtenir de meilleurs résultats.

### (5/5) Vérifier le déploiement

Kubespray fournit le moyen de vérifier la connectivité inter-pods ainsi que la
résolution DNS grâce �
[Netchecker](https://github.com/kubernetes-incubator/kubespray/blob/master/docs/netcheck.md).
Les pods netchecker-agents s'assurent que la résolution DNS (services
Kubernetes) ainsi que le ping entre les pods fonctionnent correctement. Ces pods
reproduisent un comportement similaire à celui des autres applications et
offrent un indicateur de santé du cluster.

## Opérations sur le cluster

Kubespray fournit des playbooks supplémentaires qui permettent de gérer votre
cluster: _scale_ et _upgrade_.

### Mise à l'échelle du cluster

Vous pouvez ajouter des noeuds à votre cluster en exécutant le playbook `scale`.
Pour plus d'informations se référer �
[Adding nodes](https://github.com/kubernetes-incubator/kubespray/blob/master/docs/getting-started.md#adding-nodes).
vous pouvez retirer des noeuds de votre cluster en exécutant le playbook
`remove-node`. Se référer �
[Remove nodes](https://github.com/kubernetes-incubator/kubespray/blob/master/docs/getting-started.md#remove-nodes).

### Mise à jour du cluster

Vous pouvez mettre à jour votre cluster en exécutant le playbook
`upgrade-cluster`. Pour plus d'informations se référer �
[Upgrades](https://github.com/kubernetes-incubator/kubespray/blob/master/docs/upgrades.md).

## Nettoyage

Vous pouvez réinitialiser vos noeuds et supprimer tous les composants installés
par Kubespray en utilisant le playbook
[reset](https://github.com/kubernetes-incubator/kubespray/blob/master/reset.yml).

{{< caution >}} Quand vous utilisez le playbook `reset`, assurez-vous de ne pas
cibler accidentellement un cluster de production ! {{< /caution >}}

## Retours

- Channel Slack: [#kubespray](https://kubernetes.slack.com/messages/kubespray/)
- [Issues GitHub](https://github.com/kubernetes-incubator/kubespray/issues)

{{% /capture %}}

{{% capture whatsnext %}}

Jetez un oeil aux travaux prévus sur Kubespray:
[roadmap](https://github.com/kubernetes-incubator/kubespray/blob/master/docs/roadmap.md).

{{% /capture %}}
