---
title: Aperçu de kubectl
description: kubectl référence
content_template: templates/concept
weight: 20
card:
  name: reference
  weight: 20
---

{{% capture overview %}} Kubectl est une interface en ligne de commande qui
permet d'exécuter des commandes sur des clusters Kubernetes. `kubectl` recherche
un fichier appelé config dans le répertoire \$HOME/.kube. Vous pouvez spécifier
d'autres fichiers [kubeconfig](https://kube
rnetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/) en
définissant la variable d'environnement KUBECONFIG ou en utilisant le paramètre
[`--kubeconfig`](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/).

Cet aperçu couvre la syntaxe `kubectl`, décrit les opérations et fournit des
exemples classiques. Pour des détails sur chaque commande, incluant toutes les
options et sous-commandes autorisées, voir la documentation de référence de
[kubectl](/docs/reference/generated/kubectl/kubectl-commands/). Pour des
instructions d'installation, voir
[installer kubectl](/docs/tasks/kubectl/install/).

{{% /capture %}}

{{% capture body %}}

## Syntaxe

Utilisez la syntaxe suivante pour exécuter des commandes `kubectl` depuis votre
fenêtre de terminal :

```shell
kubectl [commande] [TYPE] [NOM] [flags]
```

où `commande`, `TYPE`, `NOM` et `flags` sont :

- `commande`: Indique l'opération que vous désirez exécuter sur une ou plusieurs
  ressources, par exemple `create`, `get`, `describe`, `delete`.

- `TYPE`: Indique le [type de ressource](#resource-types). Les types de
  ressources sont insensibles à la casse et vous pouvez utiliser les formes
  singulier, pluriel ou abrégé. Par exemple, les commandes suivantes produisent
  le même résultat :

      ```shell
      $ kubectl get pod pod1
      $ kubectl get pods pod1
      $ kubectl get po pod1
      ```

- `NOM`: Indique le nom de la ressource. Les noms sont sensibles à la casse. Si
  le nom est omis, des détails pour toutes les ressources sont affichés, par
  exemple `$ kubectl get pods`.

  En effectuant une opération sur plusieurs ressources, vous pouvez soit
  indiquer chaque ressource par leur type et nom soit indiquer un ou plusieurs
  fichiers :

  - Pour indiquer des ressources par leur type et nom :

    - Pour regrouper des ressources si elles ont toutes le même type :
      `TYPE1 nom1 nom2 nom<#>`.<br/> Example:
      `$ kubectl get pod exemple-pod1 exemple-pod2`

    - Pour indiquer plusieurs types de ressources individuellement :
      `TYPE1/nom1 TYPE1/nom2 TYPE2/nom3 TYPE<#>/nom<#>`.<br/> Exemple:
      `$ kubectl get pod/exemple-pod1 replicationcontroller/exemple-rc1`

  - Pour indiquer des ressources avec un ou plusieurs fichiers :
    `-f fichier1 -f fichier2 -f fichier<#>`

    - [Utilisez YAML plutôt que JSON](/docs/concepts/configuration/overview/#general-configuration-tips),
      YAML tendant à être plus facile à utiliser, particulièrement pour des
      fichiers de configuration.<br/> Exemple: `$ kubectl get pod -f ./pod.yaml`

- `flags`: Indique des flags optionnels. Par exemple, vous pouvez utiliser les
  flags `-s` ou `--server` pour indiquer l'adresse et le port de l'API server
  Kubernetes.<br/>

{{< caution >}} Les flags indiqués en ligne de commande écrasent les valeurs par
défaut et les variables d'environnement correspondantes. {{< /caution >}}

Si vous avez besoin d'aide, exécutez `kubectl help` depuis la fenêtre de
terminal.

## Opérations

Le tableau suivant inclut une courte description et la syntaxe générale pour
chaque opération `kubectl` :

| Opération        | Syntaxe                                                                                                                                                                                                              | Description                                                                                                                                                 |
| ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `annotate`       | <code>kubectl annotate (-f FICHIER &#124; TYPE NOM &#124; TYPE/NOM) CLE_1=VAL_1 ... CLE_N=VAL_N [--overwrite][--all] [--resource-version=version][flags]</code>                                                      | Ajoute ou modifie les annotations d'une ou plusieurs ressources.                                                                                            |
| `api-versions`   | `kubectl api-versions [flags]`                                                                                                                                                                                       | Liste les versions d'API disponibles.                                                                                                                       |
| `apply`          | `kubectl apply -f FICHIER [flags]`                                                                                                                                                                                   | Applique un changement de configuration à une ressource depuis un fichier ou stdin.                                                                         |
| `attach`         | `kubectl attach POD -c CONTENEUR [-i] [-t] [flags]`                                                                                                                                                                  | Attache à un conteneur en cours d'exécution soit pour voir la sortie standard soit pour interagir avec le conteneur (stdin).                                |
| `autoscale`      | <code>kubectl autoscale (-f FICHIER &#124; TYPE NOM &#124; TYPE/NOM) [--min=MINPODS] --max=MAXPODS [--cpu-percent=CPU][flags]</code>                                                                                 | Scale automatiquement l'ensemble des pods gérés par un replication controller.                                                                              |
| `cluster-info`   | `kubectl cluster-info [flags]`                                                                                                                                                                                       | Affiche les informations des endpoints du master et des services du cluster.                                                                                |
| `config`         | `kubectl config SOUS-COMMANDE [flags]`                                                                                                                                                                               | Modifie les fichiers kubeconfig. Voir les sous-commandes individuelles pour plus de détails.                                                                |
| `create`         | `kubectl create -f FICHIER [flags]`                                                                                                                                                                                  | Crée une ou plusieurs ressources depuis un fichier ou stdin.                                                                                                |
| `delete`         | <code>kubectl delete (-f FICHIER &#124; TYPE [NOM &#124; /NOM &#124; -l label &#124; --all]) [flags]</code>                                                                                                          | Supprime des ressources soit depuis un fichier ou stdin, ou en indiquant des sélecteurs de label, des noms, des sélecteurs de ressources ou des ressources. |
| `describe`       | <code>kubectl describe (-f FICHIER &#124; TYPE [PREFIXE_NOM &#124; /NOM &#124; -l label]) [flags]</code>                                                                                                             | Affiche l'état détaillé d'une ou plusieurs ressources.                                                                                                      |
| `diff`           | `kubectl diff -f FICHIER [flags]`                                                                                                                                                                                    | Diff un fichier ou stdin par rapport à la configuration en cours (**BETA**)                                                                                 |
| `edit`           | <code>kubectl edit (-f FICHIER &#124; TYPE NOM &#124; TYPE/NOM) [flags]</code>                                                                                                                                       | Édite et met à jour la définition d'une ou plusieurs ressources sur le serveur en utilisant l'éditeur par défaut.                                           |
| `exec`           | `kubectl exec POD [-c CONTENEUR] [-i] [-t] [flags] [-- COMMANDE [args...]]`                                                                                                                                          | Exécute une commande à l'intérieur d'un conteneur dans un pod.                                                                                              |
| `explain`        | `kubectl explain [--recursive=false] [flags]`                                                                                                                                                                        | Obtient des informations sur différentes ressources. Par exemple pods, nœuds, services, etc.                                                                |
| `expose`         | <code>kubectl expose (-f FICHIER &#124; TYPE NOM &#124; TYPE/NOM) [--port=port][--protocol=tcp&#124;udp] [--target-port=nombre-ou-nom][--name=nom] [--external-ip=ip-externe-ou-service][--type=type] [flags]</code> | Expose un replication controller, service ou pod comme un nouveau service Kubernetes.                                                                       |
| `get`            | <code>kubectl get (-f FICHIER &#124; TYPE [NOM &#124; /NOM &#124; -l label]) [--watch][--sort-by=champ] [[-o &#124; --output]=FORMAT_AFFICHAGE][flags]</code>                                                        | Liste une ou plusieurs ressources.                                                                                                                          |
| `label`          | <code>kubectl label (-f FICHIER &#124; TYPE NOM &#124; TYPE/NOM) CLE_1=VAL_1 ... CLE_N=VAL_N [--overwrite][--all] [--resource-version=version][flags]</code>                                                         | Ajoute ou met à jour les labels d'une ou plusieurs ressources.                                                                                              |
| `logs`           | `kubectl logs POD [-c CONTENEUR] [--follow] [flags]`                                                                                                                                                                 | Affiche les logs d'un conteneur dans un pod.                                                                                                                |
| `patch`          | <code>kubectl patch (-f FICHIER &#124; TYPE NOM &#124; TYPE/NOM) --patch PATCH [flags]</code>                                                                                                                        | Met à jour un ou plusieurs champs d'une resource en utilisant le processus de merge patch stratégique.                                                      |
| `port-forward`   | `kubectl port-forward POD [PORT_LOCAL:]PORT_DISTANT [...[PORT_LOCAL_N:]PORT_DISTANT_N] [flags]`                                                                                                                      | Transfère un ou plusieurs ports locaux vers un pod.                                                                                                         |
| `proxy`          | `kubectl proxy [--port=PORT] [--www=static-dir] [--www-prefix=prefix] [--api-prefix=prefix] [flags]`                                                                                                                 | Exécute un proxy vers un API server Kubernetes.                                                                                                             |
| `replace`        | `kubectl replace -f FICHIER`                                                                                                                                                                                         | Remplace une ressource depuis un fichier ou stdin.                                                                                                          |
| `rolling-update` | <code>kubectl rolling-update ANCIEN_NOM_CONTROLEUR ([NOUVEAU_NOM_CONTROLEUR] --image=NOUVELLE_IMAGE_CONTENEUR &#124; -f NOUVELLE_SPEC_CONTROLEUR) [flags]</code>                                                     | Exécute un rolling update en remplaçant graduellement le replication controller indiqué et ses pods.                                                        |
| `run`            | `kubectl run NOM --image=image [--env="cle=valeur"] [--port=port] [--replicas=replicas] [--dry-run=bool] [--overrides=inline-json] [flags]`                                                                          | Exécute dans le cluster l'image indiquée.                                                                                                                   |
| `scale`          | <code>kubectl scale (-f FICHIER &#124; TYPE NOM &#124; TYPE/NOM) --replicas=QUANTITE [--resource-version=version][--current-replicas=quantité] [flags]</code>                                                        | Met à jour la taille du replication controller indiqué.                                                                                                     |
| `version`        | `kubectl version [--client] [flags]`                                                                                                                                                                                 | Affiche la version de Kubernetes du serveur et du client.                                                                                                   |

Rappelez-vous : Pour tout savoir sur les opérations, voir la documentation de
référence de [kubectl](/docs/user-guide/kubectl/).

## Types de ressources

Le tableau suivant inclut la liste de tous les types de ressources pris en
charge et leurs alias abrégés.

(cette sortie peut être obtenue depuis `kubectl api-resources`, et correspond �
Kubernetes 1.13.3.)

| Nom de la ressource               | Noms abrégés  | Groupe API                   | Par namespace | Genre de la ressource          |
| --------------------------------- | ------------- | ---------------------------- | ------------- | ------------------------------ |
| `componentstatuses`               | `cs`          |                              | false         | ComponentStatus                |
| `configmaps`                      | `cm`          |                              | true          | ConfigMap                      |
| `endpoints`                       | `ep`          |                              | true          | Endpoints                      |
| `limitranges`                     | `limits`      |                              | true          | LimitRange                     |
| `namespaces`                      | `ns`          |                              | false         | Namespace                      |
| `nodes`                           | `no`          |                              | false         | Node                           |
| `persistentvolumeclaims`          | `pvc`         |                              | true          | PersistentVolumeClaim          |
| `persistentvolumes`               | `pv`          |                              | false         | PersistentVolume               |
| `pods`                            | `po`          |                              | true          | Pod                            |
| `podtemplates`                    |               |                              | true          | PodTemplate                    |
| `replicationcontrollers`          | `rc`          |                              | true          | ReplicationController          |
| `resourcequotas`                  | `quota`       |                              | true          | ResourceQuota                  |
| `secrets`                         |               |                              | true          | Secret                         |
| `serviceaccounts`                 | `sa`          |                              | true          | ServiceAccount                 |
| `services`                        | `svc`         |                              | true          | Service                        |
| `mutatingwebhookconfigurations`   |               | admissionregistration.k8s.io | false         | MutatingWebhookConfiguration   |
| `validatingwebhookconfigurations` |               | admissionregistration.k8s.io | false         | ValidatingWebhookConfiguration |
| `customresourcedefinitions`       | `crd`, `crds` | apiextensions.k8s.io         | false         | CustomResourceDefinition       |
| `apiservices`                     |               | apiregistration.k8s.io       | false         | APIService                     |
| `controllerrevisions`             |               | apps                         | true          | ControllerRevision             |
| `daemonsets`                      | `ds`          | apps                         | true          | DaemonSet                      |
| `deployments`                     | `deploy`      | apps                         | true          | Deployment                     |
| `replicasets`                     | `rs`          | apps                         | true          | ReplicaSet                     |
| `statefulsets`                    | `sts`         | apps                         | true          | StatefulSet                    |
| `tokenreviews`                    |               | authentication.k8s.io        | false         | TokenReview                    |
| `localsubjectaccessreviews`       |               | authorization.k8s.io         | true          | LocalSubjectAccessReview       |
| `selfsubjectaccessreviews`        |               | authorization.k8s.io         | false         | SelfSubjectAccessReview        |
| `selfsubjectrulesreviews`         |               | authorization.k8s.io         | false         | SelfSubjectRulesReview         |
| `subjectaccessreviews`            |               | authorization.k8s.io         | false         | SubjectAccessReview            |
| `horizontalpodautoscalers`        | `hpa`         | autoscaling                  | true          | HorizontalPodAutoscaler        |
| `cronjobs`                        | `cj`          | batch                        | true          | CronJob                        |
| `jobs`                            |               | batch                        | true          | Job                            |
| `certificatesigningrequests`      | `csr`         | certificates.k8s.io          | false         | CertificateSigningRequest      |
| `leases`                          |               | coordination.k8s.io          | true          | Lease                          |
| `events`                          | `ev`          | events.k8s.io                | true          | Event                          |
| `ingresses`                       | `ing`         | extensions                   | true          | Ingress                        |
| `networkpolicies`                 | `netpol`      | networking.k8s.io            | true          | NetworkPolicy                  |
| `poddisruptionbudgets`            | `pdb`         | policy                       | true          | PodDisruptionBudget            |
| `podsecuritypolicies`             | `psp`         | policy                       | false         | PodSecurityPolicy              |
| `clusterrolebindings`             |               | rbac.authorization.k8s.io    | false         | ClusterRoleBinding             |
| `clusterroles`                    |               | rbac.authorization.k8s.io    | false         | ClusterRole                    |
| `rolebindings`                    |               | rbac.authorization.k8s.io    | true          | RoleBinding                    |
| `roles`                           |               | rbac.authorization.k8s.io    | true          | Role                           |
| `priorityclasses`                 | `pc`          | scheduling.k8s.io            | false         | PriorityClass                  |
| `storageclasses`                  | `sc`          | storage.k8s.io               | false         | StorageClass                   |
| `volumeattachments`               |               | storage.k8s.io               | false         | VolumeAttachment               |

## Options de sortie

Utilisez les sections suivantes pour savoir comment vous pouvez formater ou
ordonner les sorties de certaines commandes. Pour savoir exactgement quelles
commandes prennent en charge quelles options de sortie, voir la documentation de
référence de [kubectl](/docs/user-guide/kubectl/).

### Formater la sortie

Le format de sortie par défaut pour toutes les commandes `kubectl` est le format
texte lisible par l'utilisateur. Pour afficher des détails dans votre fenêtre de
terminal dans un format spécifique, vous pouvez ajouter une des options `-o` ou
`--output` à une des commandes `kubectl` les prenant en charge.

#### Syntaxe

```shell
kubectl [commande] [TYPE] [NOM] -o <format_sortie>
```

Selon l'opération `kubectl`, les formats de sortie suivants sont pris en charge
:

| Format de sortie                   | Description                                                                                                              |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `-o custom-columns=<spec>`         | Affiche un tableau en utilisant une liste de [colonnes personnalisées](#custom-columns) séparées par des virgules.       |
| `-o custom-columns-file=<fichier>` | Affiche un tableau en utilisant un modèle de [colonnes personnalisées](#custom-columns) dans le fichier `<fichier>`.     |
| `-o json`                          | Affiche un objet de l'API formaté en JSON.                                                                               |
| `-o jsonpath=<modèle>`             | Affiche les champs définis par une expression [jsonpath](/docs/reference/kubectl/jsonpath/).                             |
| `-o jsonpath-file=<ffichier>`      | Affiche les champs définis par une expression [jsonpath](/docs/reference/kubectl/jsonpath/) dans le fichier `<fichier>`. |
| `-o name`                          | Affiche uniquement le nom de la ressource et rien de plus.                                                               |
| `-o wide`                          | Affiche dans le format texte avec toute information supplémentaire. Pour les pods, le nom du nœud est inclus.            |
| `-o yaml`                          | Affiche un objet de l'API formaté en YAML.                                                                               |

##### Exemple

Dans cet exemple, la commande suivante affiche les détails d'un unique pod sous
forme d'un objet formaté en YAML :

```shell
$ kubectl get pod web-pod-13je7 -o yaml
```

Souvenez-vous : Voir la documentation de référence de
[kubectl](/docs/user-guide/kubectl/) pour voir quels formats de sortie sont pris
en charge par chaque commande.

#### Colonnes personnalisées

Pour définir des colonnes personnalisées et afficher uniquement les détails
voulus dans un tableau, vous pouvez utiliser l'option `custom-columns`. Vous
pouvez choisir de définir les colonnes personnalisées soit en ligne soit dans un
fichier modèle : `-o custom-columns=<spec>` ou
`-o custom-columns-file=<fichier>`.

##### Exemples

En ligne :

```shell
$ kubectl get pods <nom-pod> -o custom-columns=NOM:.metadata.name,RSRC:.metadata.resourceVersion
```

Fichier modèle :

```shell
$ kubectl get pods <nom-pod> -o custom-columns-file=modele.txt
```

où le fichier `modele.txt` contient :

```
NOM           RSRC
metadata.name metadata.resourceVersion
```

Le résultat de ces commandes est :

```shell
NOM            RSRC
submit-queue   610995
```

#### Colonnes côté serveur

`kubectl` est capable de recevoir des informations de colonnes spécifiques
d'objets depuis le serveur. Cela veut dire que pour toute ressource donnée, le
serveur va retourner les colonnes et lignes pour cette ressource, que le client
pourra afficher. Cela permet un affichage de sortie lisible par l'utilisateur
cohérent entre les clients utilisés sur le même cluster, le serveur encapsulant
les détails d'affichage.

Cette fonctionnalité est activée par défaut dans `kubectl` version 1.11 et
suivantes. Pour la désactiver, ajoutez l'option `--server-print=false` à la
commande `kubectl get`.

##### Exemples

Pour afficher les informations sur le status d'un pod, utilisez une commande
similaire à :

```shell
kubectl get pods <nom-pod> --server-print=false
```

La sortie ressemble à :

```shell
NAME       READY     STATUS              RESTARTS   AGE
nom-pod    1/1       Running             0          1m
```

### Ordonner les listes d'objets

Pour afficher les objets dans une liste ordonnée dans une fenêtre de terminal,
vous pouvez ajouter l'option `--sort-by` à une commande `kubectl` qui la prend
en charge. Ordonnez vos objets en spécifiant n'importe quel champ numérique ou
textuel avec l'option `--sort-by`. Pour spécifier un champ, utilisez une
expression [jsonpath](/docs/reference/kubectl/jsonpath/).

#### Syntaxe

```shell
kubectl [commande] [TYPE] [NOM] --sort-by=<exp_jsonpath>
```

##### Exemple

Pour afficher une liste de pods ordonnés par nom, exécutez :

```shell
$ kubectl get pods --sort-by=.metadata.name
```

## Exemples : Opérations courantes

Utilisez les exemples suivants pour vous familiariser avec les opérations de
`kubectl` fréquemment utilisées :

`kubectl apply` - Créer une ressource depuis un fichier ou stdin.

```shell
# Crée un service en utilisant la définition dans exemple-service.yaml.
$ kubectl apply -f exemple-service.yaml

# Crée un replication controller en utilisant la définition dans exemple-controller.yaml.
$ kubectl apply -f exemple-controller.yaml

# Crée les objets qui sont définis dans les fichiers .yaml, .yml ou .json du répertoire <répertoire>.
$ kubectl apply -f <répertoire>
```

`kubectl get` - Liste une ou plusieurs ressources.

```shell
# Liste tous les pods dans le format de sortie texte.
$ kubectl get pods

# Liste tous les pods dans le format de sortie texte et inclut des informations additionnelles (comme le nom du nœud).
$ kubectl get pods -o wide

# Liste le replication controller ayant le nom donné dans le format de sortie texte.
# Astuce : Vous pouvez raccourcir et remplacer le type de ressource 'replicationcontroller' avec l'alias 'rc'.
$ kubectl get replicationcontroller <nom-rc>

# Liste ensemble tous les replication controller et les services dans le format de sortie texte.
$ kubectl get rc,services

# Liste tous les daemon sets, dont ceux non initialisés, dans le format de sortie texte.
$ kubectl get ds --include-uninitialized

# Liste tous les pods s'exécutant sur le nœud serveur01
$ kubectl get pods --field-selector=spec.nodeName=serveur01
```

`kubectl describe` - Affiche l'état détaillé d'une ou plusieurs ressources, en
incluant par défaut les ressources non initialisées.

```shell
# Affiche les détails du nœud ayant le nom <nom-nœud>.
$ kubectl describe nodes <nom-nœud>

# Affiche les détails du pod ayant le nom <nom-pod>.
$ kubectl describe pods/<nom-pod>

# Affiche les détails de tous les pods gérés par le replication controller dont le nom est <nom-rc>.
# Rappelez-vous : les noms des pods étant créés par un replication controller sont préfixés par le nom du replication controller.
$ kubectl describe pods <nom-rc>

# Décrit tous les pods, sans inclure les non initialisés
$ kubectl describe pods --include-uninitialized=false
```

{{< note >}} La commande `kubectl get` est habituellement utilisée pour afficher
une ou plusieurs ressources d'un même type. Elle propose un ensemble complet
d'options permettant de personnaliser le format de sortie avec les options `-o`
ou `--output`, par exemple. Vous pouvez utiliser les options `-w` ou `--watch`
pour initier l'écoute des modifications d'un objet particulier. La commande
`kubectl describe` est elle plutôt utilisée pour décrire les divers aspects
d'une ressource voulue. Elle peut invoquer plusieurs appels d'API à l'API server
pour construire une vue complète pour l'utilisateur. Par exemple, la commande
`kubectl describe node` retourne non seulement les informations sur les nœuds,
mais aussi un résumé des pods s'exécutant dessus, les événements générés pour
chaque nœud, etc.nœud {{< /note >}}

`kubectl delete` - Supprime des ressources soit depuis un fichier, stdin, ou en
spécifiant des sélecteurs de labels, des noms, des sélecteurs de ressource ou
des ressources.

```shell
# Supprime un pod en utilisant le type et le nom spécifiés dans le fichier pod.yaml.
$ kubectl delete -f pod.yaml

# Supprime tous les pods et services ayant le label name=<label-name>.
$ kubectl delete pods,services -l name=<label-name>

# Supprime tous les pods et services ayant le label name=<label-name>, en incluant les non initialisés.
$ kubectl delete pods,services -l name=<label-name> --include-uninitialized

# Supprime tous les pods, en incluant les non initialisés.
$ kubectl delete pods --all
```

`kubectl exec` - Exécute une commande depuis un conteneur d'un pod.

```shell
# Affiche la sortie de la commande 'date' depuis le pod <nom-pod>. Par défaut, la sortie se fait depuis le premier conteneur.
$ kubectl exec <nom-pod> date

# Affiche la sortie de la commande 'date' depuis le conteneur <nom-conteneur> du pod <nom-pod>.
$ kubectl exec <nom-pod> -c <nom-conteneur> date

# Obtient un TTY interactif et exécute /bin/bash depuis le pod <nom-pod>. Par défaut, la sortie se fait depuis le premier conteneur.
$ kubectl exec -ti <nom-pod> /bin/bash
```

`kubectl logs` - Affiche les logs d'un conteneur dans un pod.

```shell
# Retourne un instantané des logs du pod <nom-pod>.
$ kubectl logs <nom-pod>

# Commence à streamer les logs du pod <nom-pod>. Ceci est similaire à la commande Linux 'tail -f'.
$ kubectl logs -f <nom-pod>
```

## Exemples : Créer et utiliser des plugins

Utilisez les exemples suivants pour vous familiariser avec l'écriture et
l'utilisation de plugins `kubectl` :

```shell
# créez un plugin simple dans n'importe quel langage et nommez
# l'exécutable de telle sorte qu'il commence par "kubectl-"
$ cat ./kubectl-hello
#!/bin/bash

# ce plugin affiche les mots "hello world"
echo "hello world"

# une fois votre plugin écrit, rendez-le exécutable
$ sudo chmod +x ./kubectl-hello

# et déplacez-le dans un répertoire de votre PATH
$ sudo mv ./kubectl-hello /usr/local/bin

# vous avez maintenant créé et "installé" un plugin kubectl.
# vous pouvez commencer à l'utiliser en l'invoquant depuis kubectl
# comme s'il s'agissait d'une commande ordinaire
$ kubectl hello
hello world

# vous pouvez "désinstaller" un plugin,
# simplement en le supprimant de votre PATH
$ sudo rm /usr/local/bin/kubectl-hello
```

Pour voir tous les plugins disponibles pour `kubectl`, vous pouvez utiliser la
sous-commande `kubectl plugin list` :

```shell
$ kubectl plugin list
The following kubectl-compatible plugins are available:

/usr/local/bin/kubectl-hello
/usr/local/bin/kubectl-foo
/usr/local/bin/kubectl-bar

# cette commande peut aussi vous avertir de plugins qui ne sont pas exécutables,
# ou qui sont cachés par d'autres plugins, par exemple :
$ sudo chmod -x /usr/local/bin/kubectl-foo
$ kubectl plugin list
The following kubectl-compatible plugins are available:

/usr/local/bin/kubectl-hello
/usr/local/bin/kubectl-foo
  - warning: /usr/local/bin/kubectl-foo identified as a plugin, but it is not executable
/usr/local/bin/kubectl-bar

error: one plugin warning was found
```

Vous pouvez voir les plugins comme un moyen de construire des fonctionnalités
plus complexes au dessus des commandes kubectl existantes :

```shell
$ cat ./kubectl-whoami
#!/bin/bash

# ce plugin utilise la commande `kubectl config` pour afficher
# l'information sur l'utilisateur courant, en se basant sur
# le contexte couramment sélectionné
kubectl config view --template='{{ range .contexts }}{{ if eq .name "'$(kubectl config current-context)'" }}Current user: {{ .context.user }}{{ end }}{{ end }}'
```

Exécuter le plugin ci-dessus vous donne une sortie contenant l'utilisateur du
contexte couramment sélectionné dans votre fichier KUBECONFIG :

```shell
# rendre le fichier exécutable executable
$ sudo chmod +x ./kubectl-whoami

# et le déplacer dans le PATH
$ sudo mv ./kubectl-whoami /usr/local/bin

$ kubectl whoami
Current user: plugins-user
```

Pour en savoir plus sur les plugins, examinez
[l'exemple de plugin CLI](https://github.com/kubernetes/sample-cli-plugin).

{{% /capture %}}

{{% capture whatsnext %}}

Commencez à utiliser les commandes
[kubectl](/docs/reference/generated/kubectl/kubectl-commands/).

{{% /capture %}}
