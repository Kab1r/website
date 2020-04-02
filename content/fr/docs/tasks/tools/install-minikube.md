---
title: Installer Minikube
content_template: templates/task
weight: 20
card:
  name: tasks
  weight: 10
---

{{% capture overview %}}

Cette page vous montre comment installer
[Minikube](/fr/docs/tutorials/hello-minikube/), qui est un outil qui fait
tourner un cluster Kubernetes à un noeud unique dans une machine virtuelle sur
votre machine.

{{% /capture %}}

{{% capture prerequisites %}}

{{< tabs name="minikube_before_you_begin" >}} {{% tab name="Linux" %}} Pour
vérifier si la virtualisation est prise en charge sur Linux, exécutez la
commande suivante et vérifiez que la sortie n'est pas vide :

```
grep -E --color 'vmx|svm' /proc/cpuinfo
```

{{% /tab %}}

{{% tab name="macOS" %}} Pour vérifier si la virtualisation est prise en charge
sur macOS, exécutez la commande suivante sur votre terminal.

```
sysctl -a | grep -E --color 'machdep.cpu.features|VMX'
```

Si vous trouvez `VMX` dans la sortie, la fonction VT-x est supportée sur votre
OS. {{% /tab %}}

{{% tab name="Windows" %}} Pour vérifier si la virtualisation est prise en
charge sur Windows 8 et au-delà, exécutez la commande suivante sur votre
terminal Windows ou à l'invite de commande.

```
systeminfo
```

Si vous obtenez la sortie suivant, la virtualisation est prise en charge sur
Windows.

```
Hyper-V Requirements:     VM Monitor Mode Extensions: Yes
                          Virtualization Enabled In Firmware: Yes
                          Second Level Address Translation: Yes
                          Data Execution Prevention Available: Yes
```

Si vous voyez la sortie suivante, votre système a déjà un hyperviseur installé
et vous pouvez ignorer l'étape suivante.

```
Configuration requise pour Hyper-V: un hyperviseur a été détecté. Les fonctionnalités requises pour Hyper-V ne seront pas affichées.
```

{{% /tab %}} {{< /tabs >}}

{{% /capture %}}

{{% capture steps %}}

# Installer Minikube

{{< tabs name="tab_with_md" >}} {{% tab name="Linux" %}}

### Installer kubectl

Installez kubectl en suivant les instructions de la section
[Installer et configurer kubectl](/fr/docs/tasks/tools/install-kubectl/#install-kubectl-on-linux).

### Installer un hyperviseur

Si vous n'avez pas déjà un hyperviseur installé, installez-le maintenant pour
votre système d'exploitation :

• [KVM](http://www.linux-kvm.org/), qui utilise également QEMU

• [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

Minikube supporte également une option `--vm-driver=none` qui exécute les
composants Kubernetes sur la machine hôte et dans pas dans une VM. L'utilisation
de ce pilote nécessite [Docker](https://www.docker.com/products/docker-desktop)
et un environnement Linux mais pas un hyperviseur.

Si vous utilisez le pilote `none` dans Debian ou un dérivé, utilisez les
paquets`.deb` pour Docker plutôt que le package snap, qui ne fonctionne pas avec
Minikube. Vous pouvez télécharger les packages `.deb` depuis
[Docker](https://www.docker.com/products/docker-desktop).

{{< caution >}} Le pilote VM `none` peut entraîner des problèmes de sécurité et
de perte de données. Avant d'utiliser `--driver=none`, consultez
[cette documentation](https://minikube.sigs.k8s.io/docs/reference/drivers/none/)
pour plus d'informations. {{</ caution >}}

Minikube prend également en charge un `vm-driver=podman` similaire au pilote
Docker. Podman est exécuté en tant que superutilisateur (utilisateur root),
c'est le meilleur moyen de garantir que vos conteneurs ont un accès complet �
toutes les fonctionnalités disponibles sur votre système.

{{< caution >}} Le pilote `podman` nécessite l’exécution des conteneurs en tant
que root car les comptes d’utilisateurs normaux n’ont pas un accès complet �
toutes les fonctionnalités du système d’exploitation que leurs conteneurs
pourraient avoir besoin d’exécuter. {{</ caution >}}

### Installer Minikube à l'aide d'un package

Il existe des packages _ expérimentaux _ pour Minikube; vous pouvez trouver des
packages Linux (AMD64) depuis la page
[releases](https://github.com/kubernetes/minikube/releases) de Minikube sur
GitHub.

Utilisez l'outil de package de votre distribution Linux pour installer un
package approprié.

### Installez Minikube par téléchargement direct

Si vous n'installez pas via un package, vous pouvez télécharger un binaire
autonome et l'utiliser.

```shell
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube
```

Voici un moyen simple d'ajouter l'exécutable Minikube à votre path :

```shell
sudo mkdir -p /usr/local/bin/
sudo install minikube /usr/local/bin/
```

### Installer Minikube en utilisant Homebrew

Une autre alternative, vous pouvez installer Minikube en utilisant Linux
[Homebrew](https://docs.brew.sh/Homebrew-on-Linux) :

```shell
brew install minikube
```

{{% /tab %}} {{% tab name="macOS" %}}

### Installer kubectl

Installez kubectl en suivant les instructions de la section
[Installer et configurer kubectl](/fr/docs/tasks/tools/install-kubectl/#install-kubectl-on-macos).

### Installer un hyperviseur

Si vous n'avez pas encore installé d'hyperviseur, installez-en un maintenant :

• [HyperKit](https://github.com/moby/hyperkit)

• [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

• [VMware Fusion](https://www.vmware.com/products/fusion)

### Installer Minikube

La façon la plus simple d'installer Minikube sur macOS est d'utiliser
[Homebrew](https://brew.sh):

```shell
brew install minikube
```

Vous pouvez aussi l'installer sur macOS en téléchargeant un binaire statique :

```shell
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64 \
  && chmod +x minikube
```

Voici une façon simple d'ajouter l'exécutable de Minikube à votre path :

```shell
sudo mv minikube /usr/local/bin
```

{{% /tab %}} {{% tab name="Windows" %}}

### Installer kubectl

Installez kubectl en suivant les instructions de la section
[Installer et configurer kubectl](/fr/docs/tasks/tools/install-kubectl/#install-kubectl-on-windows).

### Installer un hyperviseur

Si vous n'avez pas encore installé d'hyperviseur, installez-en un maintenant :

•
[Hyper-V](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_install)

• [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

{{< note >}} Hyper-V peut fonctionner sur trois versions de Windows 10: Windows
10 Entreprise, Windows 10 Professionnel et Windows 10 Éducation. {{</ note >}}

### Installer Minikube en utilisant Chocolatey

La façon la plus simple d'installer Minikube sur Windows est d'utiliser
[Chocolatey](https://chocolatey.org/) (exécuté avec les droits administrateur) :

```shell
choco install minikube
```

Une fois l'installation de Minikube terminée, fermez la session CLI en cours et
redémarrez. Minikube devrait avoir été ajouté à votre path automatiquement.

### Installer Minikube avec Windows Installer

Pour installer manuellement Minikube sur Windows à l'aide de
[Windows Installer](https://docs.microsoft.com/en-us/windows/desktop/msi/windows-installer-portal),
téléchargez
[`minikube-installer.exe`](https://github.com/kubernetes/minikube/releases/latest)
et exécutez l'Installer.

#### Installer Minikube manuellement

Pour installer Minikube manuellement sur Windows, téléchargez
[`minikube-windows-amd64`](https://github.com/kubernetes/minikube/releases/latest),
renommez-le en `minikube.exe`, et ajoutez-le à votre path.

{{% /tab %}} {{< /tabs >}}

{{% /capture %}}

{{% capture whatsnext %}}

- [Exécutez Kubernetes localement via Minikube](/fr/docs/setup/learning-environment/minikube/)

{{% /capture %}}

## Confirmer l'installation

Pour confirmer la réussite de l'installation d'un hyperviseur et d'un mini-cube,
vous pouvez exécuter la commande suivante pour démarrer un cluster Kubernetes
local :

{{< note >}}

Pour définir le `--driver` avec`minikube start`, entrez le nom de l'hyperviseur
que vous avez installé en minuscules où `<driver_name>` est mentionné
ci-dessous. Une liste complète des valeurs `--driver` est disponible dans
[la documentation spécifiant le pilote VM](https://kubernetes.io/docs/setup/learning-environment/minikube/#specifying-the-vm-driver).

{{</ note >}}

```shell
minikube start --driver=<driver_name>
```

Une fois `minikube start` terminé, exécutez la commande ci-dessous pour vérifier
l'état du cluster :

```shell
minikube status
```

Si votre cluster est en cours d'exécution, la sortie de `minikube status`
devrait être similaire à :

```
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

Après avoir vérifié si Minikube fonctionne avec l'hyperviseur choisi, vous
pouvez continuer à utiliser Minikube ou arrêter votre cluster. Pour arrêter
votre cluster, exécutez :

```shell
minikube stop
```

## Tout nettoyer pour recommencer à zéro

Si vous avez déjà installé minikube, exécutez :

```shell
minikube start
```

Si cette commande renvoie une erreur :

```shell
machine does not exist
```

Vous devez supprimer les fichiers de configuration :

```shell
rm -rf ~/.minikube
```
