---
title: Init Containers
content_template: templates/concept
weight: 40
---

{{% capture overview %}} Cette page fournit une vue d'ensemble des _conteneurs
d'initialisation_ (init containers) : des conteneurs spécialisés qui s'exécutent
avant les conteneurs d'application dans un
{{< glossary_tooltip text="Pod" term_id="pod" >}}. Les init containers peuvent
contenir des utilitaires ou des scripts d'installation qui ne sont pas présents
dans une image d'application.

Vous pouvez spécifier des init containers dans la spécification du Pod à côté du
tableau `containers` (qui décrit les conteneurs d'application) {{% /capture %}}

{{% capture body %}}

## Comprendre les init containers

Un {{< glossary_tooltip text="Pod" term_id="pod" >}} peut avoir plusieurs
conteneurs exécutant des applications mais peut aussi avoir un ou plusieurs init
containers, qui sont exécutés avant que les conteneurs d'application ne
démarrent.

Les init containers se comportent comme les conteneurs réguliers, avec quelques
différences :

- Les init containers s'exécutent toujours jusqu'à la complétion.
- Chaque init container doit se terminer avec succès avant que le prochain ne
  démarre.

Si le init container d'un Pod échoue, Kubernetes redémarre le Pod à répétition
jusqu'à ce que le init container se termine avec succès. Cependant, si le Pod a
une `restartPolicy` à "Never", Kubernetes ne redémarre pas le Pod.

Afin de spécifier un init container pour un Pod, il faut ajouter le champ
`initContainers` dans la spécification du Pod, comme un tableau d'objets de type
[Container](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#container-v1-core),
au même niveau que le tableau d'applications `containers`. Le statut des init
containers est retourné dans le champ `.status.initContainerStatuses` comme un
tableau des statuts du conteneur (comparable au champ
`.status.containerStatuses`).

### Différences avec les conteneurs réguliers

Les init containers supportent tous les champs et fonctionnalités des conteneurs
d'application incluant les limites de ressources, les volumes et les paramètres
de sécurité. Cependant, les demandes de ressources pour un init container sont
gérées différemment des limites de ressources, tel que documenté dans
[Ressources](#ressources).

De plus, les init containers ne supportent pas les readiness probes parce que
ces conteneurs s'exécutent jusqu'au bout avant que le Pod soit prêt.

Si l'on spécifie plusieurs init containers pour un Pod, Kubelet exécute chaque
init container de manière séquentielle. Chaque init container doit se terminer
avec succès avant que le prochain ne puisse s'exécuter. Lorsque tous les init
containers se sont exécutés jusqu'au bout, Kubelet initialise les conteneurs
d'application pour le Pod et les exécute comme d'habitude.

## Utiliser les init containers

Puisque les init containers ont des images séparées des conteneurs
d'application, ils apportent certains avantages pour du code de mise en route :

- Les init containers peuvent contenir des utilitaires ou du code de
  configuration personnalisé qui ne sont pas présents dans une image
  d'application. Par exemple, il n'y a pas besoin de faire hériter une image
  d'une autre (`FROM`) seulement pour utiliser un outil comme `sed`, `awk`,
  `python`, ou `dig` pendant l'installation.
- Les init containers peuvent exécuter en toute sécurité des utilitaires qui
  rendraient moins sécurisée une image de conteneur d'application.
- Les rôles "builder" et "deployer" d'une image d'application peuvent travailler
  indépendamment sans qu'il n'y ait besoin de créer conjointement une seule
  image d'application.
- Les init containers peuvent s'exécuter avec une vue du système de fichiers
  différente de celle des conteneurs d'application dans le même Pod. Par
  conséquent, on peut leur donner accès aux
  {{< glossary_tooltip text="Secrets" term_id="secret" >}}, auxquels les
  conteneurs d'application n'ont pas accès.
- Puisque les init containers s'exécutent jusqu'à la complétion avant qu'un
  conteneur d'application ne démarre, les init containers offrent un mécanisme
  pour bloquer ou retarder le démarrage d'un conteneur d'application tant qu'un
  ensemble de préconditions n'est pas respecté. Une fois que les préconditions
  sont respectées, tous les conteneurs d'application dans un Pod peuvent
  démarrer en parallèle.

### Exemples

Voici plusieurs idées pour utiliser les init containers :

- Attendre qu'un {{< glossary_tooltip text="Service" term_id="service">}} soit
  créé, en utilisant une commande shell d'une ligne telle que :

  ```shell
  for i in {1..100}; do sleep 1; if dig myservice; then exit 0; fi; done; exit 1
  ```

- Enregistrer ce Pod à un serveur distant depuis l'API downward avec une
  commande telle que :

  ```shell
  curl -X POST http://$MANAGEMENT_SERVICE_HOST:$MANAGEMENT_SERVICE_PORT/register -d 'instance=$(<POD_NAME>)&ip=$(<POD_IP>)'
  ```

- Attendre un certain temps avant de démarrer le conteneur d'application avec
  une commande telle que :

  ```shell
  sleep 60
  ```

- Cloner un dépôt Git dans un
  {{< glossary_tooltip text="Volume" term_id="volume" >}}

- Placer des valeurs dans un fichier de configuration et exécuter un outil de
  templating pour générer dynamiquement un fichier de configuration pour le
  conteneur d'application principal. Par exemple, placer la valeur `POD_IP` dans
  une configuration et générer le fichier de configuration de l'application
  principale en utilisant Jinja.

#### Les init containers en utilisation

Cet exemple définit un simple Pod possédant deux init containers. Le premier
attend `myservice` et le second attend `mydb`. Une fois que les deux init
containers terminent leur exécution, le Pod exécute le conteneur d'application
décrit dans sa section `spec`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
    - name: myapp-container
      image: busybox:1.28
      command: ["sh", "-c", 'echo "L''app s''exécute!" && sleep 3600']
  initContainers:
    - name: init-myservice
      image: busybox:1.28
      command:
        [
          "sh",
          "-c",
          "until nslookup myservice.$(cat
          /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local;
          do echo en attente de myservice; sleep 2; done",
        ]
    - name: init-mydb
      image: busybox:1.28
      command:
        [
          "sh",
          "-c",
          "until nslookup mydb.$(cat
          /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local;
          do echo en attente de mydb; sleep 2; done",
        ]
```

Les fichiers YAML suivants résument les services `mydb` et `myservice` :

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
---
apiVersion: v1
kind: Service
metadata:
  name: mydb
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9377
```

Vous pouvez démarrer ce Pod en exécutant :

```shell
kubectl apply -f myapp.yaml
```

```
pod/myapp-pod created
```

Et vérifier son statut avec :

```shell
kubectl get -f myapp.yaml
```

```
NAME        READY     STATUS     RESTARTS   AGE
myapp-pod   0/1       Init:0/2   0          6m
```

ou pour plus de détails :

```shell
kubectl describe -f myapp.yaml
```

```
Name:          myapp-pod
Namespace:     default
[...]
Labels:        app=myapp
Status:        Pending
[...]
Init Containers:
  init-myservice:
[...]
    State:         Running
[...]
  init-mydb:
[...]
    State:         Waiting
      Reason:      PodInitializing
    Ready:         False
[...]
Containers:
  myapp-container:
[...]
    State:         Waiting
      Reason:      PodInitializing
    Ready:         False
[...]
Events:
  FirstSeen    LastSeen    Count    From                      SubObjectPath                           Type          Reason        Message
  ---------    --------    -----    ----                      -------------                           --------      ------        -------
  16s          16s         1        {default-scheduler }                                              Normal        Scheduled     Successfully assigned myapp-pod to 172.17.4.201
  16s          16s         1        {kubelet 172.17.4.201}    spec.initContainers{init-myservice}     Normal        Pulling       pulling image "busybox"
  13s          13s         1        {kubelet 172.17.4.201}    spec.initContainers{init-myservice}     Normal        Pulled        Successfully pulled image "busybox"
  13s          13s         1        {kubelet 172.17.4.201}    spec.initContainers{init-myservice}     Normal        Created       Created container with docker id 5ced34a04634; Security:[seccomp=unconfined]
  13s          13s         1        {kubelet 172.17.4.201}    spec.initContainers{init-myservice}     Normal        Started       Started container with docker id 5ced34a04634
```

Pour voir les logs des init containers dans ce Pod, exécuter :

```shell
kubectl logs myapp-pod -c init-myservice # Inspecter le premier init container
kubectl logs myapp-pod -c init-mydb      # Inspecter le second init container
```

À ce stade, ces init containers attendent de découvrir les services nommés
`mydb` et `myservice`.

Voici une configuration que vous pouvez utiliser pour faire apparaître ces
Services :

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
---
apiVersion: v1
kind: Service
metadata:
  name: mydb
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9377
```

Pour créer les services `mydb` et `myservice` :

```shell
kubectl apply -f services.yaml
```

```
service/myservice created
service/mydb created
```

Vous verrez ensuite que ces init containers se terminent et que le Pod
`myapp-pod` évolue vers l'état "Running" (en exécution) :

```shell
kubectl get -f myapp.yaml
```

```
NAME        READY     STATUS    RESTARTS   AGE
myapp-pod   1/1       Running   0          9m
```

Cet exemple simple devrait suffire à vous inspirer pour créer vos propres init
containers. [A suivre](#a-suivre) contient un lien vers un exemple plus
détaillé.

## Comportement détaillé

Pendant le démarrage d'un Pod, chaque init container démarre en ordre, après que
le réseau et les volumes ont été initialisés. Chaque conteneur doit se terminer
avec succès avant que le prochain ne démarre. Si un conteneur n'arrive pas �
démarrer à cause d'un problème d'exécution ou se termine avec un échec, il est
redémarré selon la `restartPolicy` du Pod. Toutefois, si la `restartPolicy` du
Pod est configurée à "Always", les init containers utilisent la `restartPolicy`
"OnFailure".

Un Pod ne peut pas être `Ready` tant que tous les init containers ne se sont pas
exécutés avec succès. Les ports d'un init container ne sont pas agrégés sous un
Service. Un Pod qui s'initialise est dans l'état `Pending` mais devrait avoir
une condition `Initialized` configurée à "true".

Si le Pod [redémarre](#raisons-du-redémarrage-d-un-pod) ou est redémarré, tous
les init containers doivent s'exécuter à nouveau.

Les changements aux spec d'un init containers sont limités au champ image du
conteneur. Changer le champ image d'un init container équivaut à redémarrer le
Pod.

Puisque les init containers peuvent être redémarrés, réessayés ou ré-exécutés,
leur code doit être idempotent. En particulier, le code qui écrit dans des
fichiers sur `EmptyDirs` devrait être préparé à la possibilité qu'un fichier de
sortie existe déjà.

Les init containers ont tous les champs d'un conteneur d'application. Cependant,
Kubernetes interdit l'utilisation de `readinessProbe` parce que les init
containers ne peuvent pas définir une "readiness" distincte de la complétion.
Ceci est appliqué lors de la validation.

L'utilisation de `activeDeadlineSeconds` sur le Pod et `livenessProbe` sur le
conteneur permet d'empêcher les init containers d'échouer tout le temps. La
deadline active inclut les init containers.

Le nom de chaque application et init container dans un Pod doit être unique; une
erreur de validation est générée pour tout conteneur partageant un nom avec un
autre.

### Ressources

Étant donné l'ordonnancement et l'exécution des init containers, les règles
suivantes s'appliquent pour l'utilisation des ressources :

- La plus haute requête ou limite particulière de ressource définie pour tous
  les init containers est la _limite/requête d'initialisation effective_
- La _limite/requête effective_ d'un Pod pour une ressource est la plus haute
  parmis :
  - la somme de toutes les requêtes/limites des conteneurs d'application pour
    une ressource
  - la limite/requête d'initialisation effective pour une ressource
- Le Scheduling est effectué sur la base des requêtes/limites effectives, ce qui
  signifie que les init containers peuvent réserver des ressources pour
  l'initialisation qui ne sont pas utilisées durant le cycle de vie du Pod.
- La QoS (qualité de service) tierce de la _QoS tierce effective_ d'un Pod est
  la QoS tierce aussi bien pour les init containers que pour les conteneurs
  d'application.

Les quotas et limites sont appliqués sur la base de la requête/limite effective
d'un Pod.

Les groupes de contrôle au niveau du Pod
({{< glossary_tooltip text="cgroups" term_id="cgroup" >}}) sont basés sur la
requête/limite effective de Pod, la même que celle du scheduler.

### Raisons du redémarrage d'un Pod

Un Pod peut redémarrer, ce qui cause la ré-exécution des init containers, pour
les raisons suivantes :

- Un utilisateur met à jour les spécifications du Pod, ce qui cause le
  changement de l'image de l'init container. Tout changement à l'image du init
  container redémarre le Pod. Les changements au conteneur d'application
  entraînent seulement le redémarrage du conteneur d'application.
- Le conteneur d'infrastructure Pod est redémarré. Ceci est peu commun et serait
  effectué par une personne ayant un accès root aux nœuds.
- Tous les conteneurs dans un Pod sont terminés tandis que `restartPolicy` est
  configurée à "Always", ce qui force le redémarrage, et l'enregistrement de
  complétion du init container a été perdu à cause d'une opération de garbage
  collection (récupération de mémoire).

{{% /capture %}}

{{% capture whatsnext %}}

- Lire à propos de la
  [création d'un Pod ayant un init container](/docs/tasks/configure-pod-container/configure-pod-initialization/#creating-a-pod-that-has-an-init-container)
- Apprendre �
  [debugger les init containers](/docs/tasks/debug-application-cluster/debug-init-containers/)

{{% /capture %}}
