---
title: Service
feature:
  title: Découverte de services et équilibrage de charge
  description: >
    Pas besoin de modifier votre application pour utiliser un mécanisme de
    découverte de services inconnu. Kubernetes donne aux pods leurs propres
    adresses IP et un nom DNS unique pour un ensemble de pods, et peut
    équilibrer la charge entre eux.

content_template: templates/concept
weight: 10
---

{{% capture overview %}}

{{< glossary_definition term_id="service" length="short" >}}

Avec Kubernetes, vous n'avez pas besoin de modifier votre application pour
utiliser un mécanisme de découverte de services inconnu. Kubernetes donne aux
pods leurs propres adresses IP et un nom DNS unique pour un ensemble de pods, et
peut équilibrer la charge entre eux.

{{% /capture %}}

{{% capture body %}}

## Motivation

Les {{< glossary_tooltip term_id="pod" text="Pods" >}} Kubernetes sont mortels.
Ils naissent et lorsqu'ils meurent, ils ne ressuscitent pas. Si vous utilisez un
{{< glossary_tooltip term_id="deployment" >}} pour exécuter votre application,
il peut créer et détruire dynamiquement des pods.

Chaque pod obtient sa propre adresse IP, mais dans un déploiement, l'ensemble de
pods s'exécutant en un instant peut être différent de l'ensemble de pods
exécutant cette application un instant plus tard.

Cela conduit à un problème: si un ensemble de pods (appelez-les «backends»)
fournit des fonctionnalités à d'autres pods (appelez-les «frontends») �
l'intérieur de votre cluster, comment les frontends peuvent-ils trouver et
suivre l'adresse IP à laquelle se connecter, afin que le frontend puisse
utiliser la partie backend de la charge de travail?

C'est là où les _Services_ rentrent en jeu.

## La ressource Service {#service-resource}

Dans Kubernetes, un service est une abstraction qui définit un ensemble logique
de pods et une politique permettant d'y accéder (parfois ce modèle est appelé un
micro-service). L'ensemble des pods ciblés par un service est généralement
déterminé par un {{< glossary_tooltip text="selector" term_id="selector" >}}
(voir [ci-dessous](#services-without-selectors) pourquoi vous voudrez peut-être
un service _sans_ un sélecteur).

Par exemple, considérons un backend de traitement d'image sans état qui
s'exécute avec 3 replicas. Ces réplicas sont fongibles et les frontends ne se
soucient pas du backend qu'ils utilisent. Bien que les pods réels qui composent
l'ensemble backend puissent changer, les clients frontends ne devraient pas
avoir besoin de le savoir, pas plus qu'ils ne doivent suivre eux-mêmes
l'ensemble des backends.

L'abstraction du service permet ce découplage.

### Découverte de services native du cloud

Si vous pouvez utiliser les API Kubernetes pour la découverte de services dans
votre application, vous pouvez interroger
l'{{< glossary_tooltip text="API server" term_id="kube-apiserver" >}} pour les
Endpoints, qui sont mis à jour chaque fois que l'ensemble des pods d'un service
change.

Pour les applications non natives, Kubernetes propose des moyens de placer un
port réseau ou un load balancer entre votre application et les modules backend.

## Définition d'un service

Un service dans Kubernetes est un objet REST, semblable à un pod. Comme tous les
objets REST, vous pouvez effectuer un `POST` d'une définition de service sur le
serveur API pour créer une nouvelle instance.

Par exemple, supposons que vous ayez un ensemble de pods qui écoutent chacun sur
le port TCP 9376 et portent une étiquette `app=MyApp`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

Cette spécification crée un nouvel objet Service nommé «my-service», qui cible
le port TCP 9376 sur n'importe quel pod avec l'étiquette «app=MyApp».

Kubernetes attribue à ce service une adresse IP (parfois appelé l'"IP cluster"),
qui est utilisé par les proxies Service (voir
[IP virtuelles et proxy de service](#virtual-ips-and-service-proxies)).

Le contrôleur de service recherche en continu les pods qui correspondent à son
sélecteur, puis POST toutes les mises à jour d'un objet Endpoint également
appelé "my-service".

{{< note >}} Un service peut mapper _n'importe quel_ `port` entrant vers un
`targetPort`. Par défaut et pour plus de commodité, le `targetPort` a la même
valeur que le champ `port`. {{< /note >}}

Les définitions de port dans les pods ont des noms, et vous pouvez référencer
ces noms dans l'attribut `targetPort` d'un service. Cela fonctionne même s'il
existe un mélange de pods dans le service utilisant un seul nom configuré, avec
le même protocole réseau disponible via différents numéros de port. Cela offre
beaucoup de flexibilité pour déployer et faire évoluer vos services. Par
exemple, vous pouvez modifier les numéros de port que les pods exposent dans la
prochaine version de votre logiciel principal, sans casser les clients.

Le protocole par défaut pour les services est TCP; vous pouvez également
utiliser tout autre [protocole pris en charge](#protocol-support).

Comme de nombreux services doivent exposer plus d'un port, Kubernetes prend en
charge plusieurs définitions de port sur un objet Service. Chaque définition de
port peut avoir le même protocole, ou un autre.

### Services sans sélecteurs

Les services abritent le plus souvent l'accès aux pods Kubernetes, mais ils
peuvent également abstraire d'autres types de backends. Par exemple:

- Vous voulez avoir un cluster de base de données externe en production, mais
  dans votre environnement de test, vous utilisez vos propres bases de données.
- Vous souhaitez pointer votre service vers un service dans un autre
  {{< glossary_tooltip term_id="namespace" >}} ou sur un autre cluster.
- Vous migrez une charge de travail vers Kubernetes. Lors de l'évaluation de
  l'approche, vous exécutez uniquement une partie de vos backends dans
  Kubernetes.

Dans n'importe lequel de ces scénarios, vous pouvez définir un service _sans_ un
sélecteur de pod. Par exemple:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

Étant donné que ce service n'a pas de sélecteur, l'objet Endpoint correspondant
n'est _pas_ créé automatiquement. Vous pouvez mapper manuellement le service �
l'adresse réseau et au port où il s'exécute, en ajoutant manuellement un objet
Endpoint:

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: my-service
subsets:
  - addresses:
      - ip: 192.0.2.42
    ports:
      - port: 9376
```

{{< note >}} Les IP de noeud final ne doivent pas être: loopback (127.0.0.0/8
pour IPv4, ::1/128 pour IPv6), ou link-local (169.254.0.0/16 et 224.0.0.0/24
pour IPv4, fe80::/64 pour IPv6).

Les adresses IP de noeud final ne peuvent pas être les adresses IP de cluster
d'autres services Kubernetes, car {{< glossary_tooltip term_id="kube-proxy" >}}
ne prend pas en charge les adresses IP virtuelles en tant que destination.
{{< /note >}}

L'accès à un service sans sélecteur fonctionne de la même manière que s'il avait
un sélecteur. Dans l'exemple ci-dessus, le trafic est routé vers le Endpoint
unique défini dans le YAML: `192.0.2.42:9376` (TCP).

Un service ExternalName est un cas spécial de service qui n'a pas de sélecteurs
et utilise des noms DNS à la place. Pour plus d'informations, consultez la
section [ExternalName](#externalname) plus loin dans ce document.

### Endpoint Slices

{{< feature-state for_k8s_version="v1.17" state="beta" >}}

Un Endpoint Slices est une ressource API qui peut fournir une alternative plus
évolutive au Endpoints. Bien que conceptuellement assez similaire aux Endpoints,
les Endpoint Slices permettent la distribution des endpoints réseau sur
plusieurs ressources. Par défaut, un Endpoint Slice est considéré comme "plein"
une fois qu'il atteint 100 endpoints, au delà, des Endpoint Slices addtionnels
seront crées pour stocker tout autre endpoints.

Les Endpoint Slices fournissent des attributs et des fonctionnalités
supplémentaires qui sont décrits en détail dans
[Endpoint Slices](/docs/concepts/services-networking/endpoint-slices/).

## IP virtuelles et proxy de service

Chaque nœud d'un cluster Kubernetes exécute un `kube-proxy`. `kube-proxy` est
responsable de l'implémentation d'une forme d'IP virtuelle pour les `Services`
qui ne sont pas de type [`ExternalName`](#externalname).

### Pourquoi ne pas utiliser le DNS round-robin ?

Une question qui apparaît de temps en temps est pourquoi Kubernetes s'appuie sur
le proxy pour transférer le trafic entrant vers les backends. Et les autres
approches? Par exemple, serait-il possible de configurer des enregistrements DNS
qui ont plusieurs valeurs A (ou AAAA pour IPv6), et de s'appuyer sur la
résolution de nom à tour de rôle (round-robin)?

Il existe plusieurs raisons d'utiliser le proxy pour les services:

- Il existe une longue histoire d'implémentations DNS ne respectant pas les TTL
  d'enregistrement et mettant en cache les résultats des recherches de noms
  après leur expiration.
- Certaines applications n'effectuent des recherches DNS qu'une seule fois et
  mettent en cache les résultats indéfiniment.
- Même si les applications et les bibliothèques ont fait une bonne résolution,
  les TTL faibles ou nuls sur les enregistrements DNS pourraient imposer une
  charge élevée sur DNS qui devient alors difficile à gérer.

### User space proxy mode {#proxy-mode-userspace}

Dans ce mode, kube-proxy surveille le maître Kubernetes pour l'ajout et la
suppression d'objets Service et Endpoint. Pour chaque service, il ouvre un port
(choisi au hasard) sur le nœud local. Toutes les connexions à ce "port proxy"
sont transmises par proxy à l'un des modules backend du service (comme indiqué
via les Endpoints). kube-proxy prend en compte le paramètre `SessionAffinity` du
service pour décider quel pod backend utiliser.

Enfin, le proxy de l'espace utilisateur installe des règles iptables qui
capturent le trafic vers le service `clusterIP` (qui est virtuel) et `port`. Les
règles redirigent ce trafic vers le port proxy qui fait office de proxy pour le
Pod de backend.

Par défaut, kube-proxy en mode espace utilisateur choisit un backend via un
algorithme round-robin.

![Diagramme de vue d'ensemble des services pour le proxy de l'espace utilisateur](/images/docs/services-userspace-overview.svg)

### `iptables` proxy mode {#proxy-mode-iptables}

Dans ce mode, kube-proxy surveille le plan de contrôle Kubernetes pour l'ajout
et la suppression d'objets Service et Endpoint. Pour chaque service, il installe
des règles iptables, qui capturent le trafic vers le «clusterIP» et le «port» du
service, et redirigent ce trafic vers l'un des ensembles principaux du service.
Pour chaque objet Endpoint, il installe des règles iptables qui sélectionnent un
Pod de backend.

Par défaut, kube-proxy en mode iptables choisit un backend au hasard.

L'utilisation d'iptables pour gérer le trafic a un coût système inférieur, car
le trafic est géré par Linux netfilter sans avoir besoin de basculer entre
l'espace utilisateur et l'espace noyau. Cette approche est également susceptible
d'être plus fiable.

Si kube-proxy s'exécute en mode iptables et que le premier pod sélectionné ne
répond pas, la connexion échoue. C'est différent du mode espace utilisateur:
dans ce scénario, kube-proxy détecterait que la connexion au premier pod avait
échoué et réessayerait automatiquement avec un pod backend différent.

Vous pouvez utiliser les
[readiness probes](/docs/concepts/workloads/pods/pod-lifecycle/#container-probes)
d'un Pod pour vérifier que les pods backend fonctionnent correctement, de sorte
que kube-proxy en mode iptables ne voit que les backends testés comme sains.
Cela signifie que vous évitez d'envoyer du trafic via kube-proxy vers un pod
connu pour avoir échoué.

![Diagramme de présentation des services pour le proxy iptables](/images/docs/services-iptables-overview.svg)

### IPVS proxy mode {#proxy-mode-ipvs}

{{< feature-state for_k8s_version="v1.11" state="stable" >}}

En mode `ipvs`, kube-proxy surveille les Services et Endpoints Kubernetes.
kube-proxy appelle l'interface`netlink` pour créer les règles IPVS en
conséquence et synchronise périodiquement les règles IPVS avec les Services et
Endpoints Kubernetes. Cette boucle de contrôle garantit que l'état IPVS
correspond à l'état souhaité. Lors de l'accès à un service, IPVS dirige le
trafic vers l'un des pods backend.

Le mode proxy IPVS est basé sur des fonctions hooks de netfilter qui est
similaire au mode iptables, mais utilise la table de hachage comme structure de
données sous-jacente et fonctionne dans l'espace du noyau. Cela signifie que
kube-proxy en mode IPVS redirige le trafic avec une latence plus faible que
kube-proxy en mode iptables, avec de bien meilleures performances lors de la
synchronisation des règles de proxy. Par rapport aux autres modes proxy, le mode
IPVS prend également en charge un débit plus élevé de trafic réseau.

IPVS offre plus d'options pour équilibrer le trafic vers les pods
d'arrière-plan; ceux-ci sont:

- `rr`: round-robin
- `lc`: least connection (plus petit nombre de connexions ouvertes)
- `dh`: destination hashing
- `sh`: source hashing
- `sed`: shortest expected delay
- `nq`: never queue

{{< note >}} Pour exécuter kube-proxy en mode IPVS, vous devez rendre IPVS Linux
disponible sur le nœud avant de démarrer kube-proxy.

Lorsque kube-proxy démarre en mode proxy IPVS, il vérifie si les modules du
noyau IPVS sont disponibles. Si les modules du noyau IPVS ne sont pas détectés,
alors kube-proxy revient à fonctionner en mode proxy iptables. {{< /note >}}

![Diagramme de vue d'ensemble des services pour le proxy IPVS](/images/docs/services-ipvs-overview.svg)

Dans ces modèles de proxy, le trafic lié à l'IP: Port du service est dirigé vers
un backend approprié sans que les clients ne sachent quoi que ce soit sur
Kubernetes, les services ou les pods.

Si vous souhaitez vous assurer que les connexions d'un client particulier sont
transmises à chaque fois au même pod, vous pouvez sélectionner l'affinité de
session en fonction des adresses IP du client en définissant
`service.spec.sessionAffinity` sur" ClientIP "(la valeur par défaut est" None").
Vous pouvez également définir la durée maximale de session persistante en
définissant `service.spec.sessionAffinityConfig.clientIP.timeoutSeconds` de
manière appropriée (la valeur par défaut est 10800, ce qui correspond à 3
heures).

## Services multi-ports

Pour certains services, vous devez exposer plusieurs ports. Kubernetes vous
permet de configurer plusieurs définitions de port sur un objet Service. Lorsque
vous utilisez plusieurs ports pour un service, vous devez donner tous vos noms
de ports afin qu'ils ne soient pas ambigus. Par exemple:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 9376
    - name: https
      protocol: TCP
      port: 443
      targetPort: 9377
```

{{< note >}} Comme pour tous les
{{< glossary_tooltip term_id="name" text="names">}} Kubernetes en général, les
noms de ports ne doivent contenir que des caractères alphanumériques en
minuscules et `-`. Les noms de port doivent également commencer et se terminer
par un caractère alphanumérique.

Par exemple, les noms `123-abc` et `web` sont valides, mais `123_abc` et `-web`
ne le sont pas. {{< /note >}}

## Choisir sa propre adresse IP

Vous pouvez spécifier votre propre adresse IP de cluster dans le cadre d'une
demande de création de Service. Pour ce faire, définissez le champ
`.spec.clusterIP`. Par exemple, si vous avez déjà une entrée DNS existante que
vous souhaitez réutiliser, ou des systèmes existants qui sont configurés pour
une adresse IP spécifique et difficiles à reconfigurer.

L'adresse IP que vous choisissez doit être une adresse IPv4 ou IPv6 valide dans
la plage CIDR `service-cluster-ip-range` configurée pour le serveur API. Si vous
essayez de créer un service avec une valeur d'adresse de clusterIP non valide,
le serveur API retournera un code d'état HTTP 422 pour indiquer qu'il y a un
problème.

## Découvrir les services

Kubernetes prend en charge 2 modes principaux de recherche d'un service: les
variables d'environnement et DNS.

### Variables d'environnement

Lorsqu'un pod est exécuté sur un nœud, le kubelet ajoute un ensemble de
variables d'environnement pour chaque service actif. Il prend en charge à la
fois les variables
[Docker links](https://docs.docker.com/userguide/dockerlinks/) (voir
[makeLinkVariables](http://releases.k8s.io/{{< param
"githubbranch" >}}/pkg/kubelet/envvars/envvars.go#L49)) et plus simplement les
variables `{SVCNAME}_SERVICE_HOST` et `{SVCNAME}_SERVICE_PORT`, où le nom du
service est en majuscules et les tirets sont convertis en underscore.

Par exemple, le service `redis-master` qui expose le port TCP 6379 et a reçu
l'adresse IP de cluster 10.0.0.11, produit les variables d'environnement
suivantes:

```shell
REDIS_MASTER_SERVICE_HOST=10.0.0.11
REDIS_MASTER_SERVICE_PORT=6379
REDIS_MASTER_PORT=tcp://10.0.0.11:6379
REDIS_MASTER_PORT_6379_TCP=tcp://10.0.0.11:6379
REDIS_MASTER_PORT_6379_TCP_PROTO=tcp
REDIS_MASTER_PORT_6379_TCP_PORT=6379
REDIS_MASTER_PORT_6379_TCP_ADDR=10.0.0.11
```

{{< note >}} Lorsque vous avez un pod qui doit accéder à un service et que vous
utilisez la méthode des variables d'environnement pour publier le port et l'IP
du cluster sur les pods clients, vous devez créer le service _avant_ que les
pods clients n'existent. Sinon, ces pods clients n'auront pas leurs variables
d'environnement remplies.

Si vous utilisez uniquement DNS pour découvrir l'IP du cluster pour un service,
vous n'avez pas à vous soucier de ce problème de commande. {{< /note >}}

### DNS

Vous pouvez (et devriez presque toujours) configurer un service DNS pour votre
cluster Kubernetes à l'aide d'un
[add-on](/docs/concepts/cluster-administration/addons/).

Un serveur DNS prenant en charge les clusters, tel que CoreDNS, surveille l'API
Kubernetes pour les nouveaux services et crée un ensemble d'enregistrements DNS
pour chacun. Si le DNS a été activé dans votre cluster, tous les pods devraient
automatiquement être en mesure de résoudre les services par leur nom DNS.

Par exemple, si vous avez un service appelé `"my-service"` dans un namespace
Kubernetes `"my-ns"`, le plan de contrôle et le service DNS agissant ensemble et
créent un enregistrement DNS pour `"my-service.my-ns"`. Les Pods dans le
Namespace `"my-ns"` devrait être en mesure de le trouver en faisant simplement
une recherche de nom pour `my-service` (`"my-service.my-ns"` fonctionnerait
également).

Les pods dans d'autres namespaces doivent utiliser le nom de `my-service.my-ns`.
Ces noms seront résolus en IP de cluster attribuée pour le service.

Kubernetes prend également en charge les enregistrements DNS SRV (Service) pour
les ports nommés. Si le service `"my-service.my-ns"` a un port nommé `http` avec
un protocole défini sur `TCP`, vous pouvez effectuer une requête DNS SRV pour
`_http._tcp.my-service.my-ns` pour découvrir le numéro de port de `http`, ainsi
que l'adresse IP.

Le serveur DNS Kubernetes est le seul moyen d'accéder aux services
`ExternalName`. Vous pouvez trouver plus d'informations sur la résolution de
`ExternalName` dans
[DNS Pods et Services](/docs/concepts/services-networking/dns-pod-service/).

## Headless Services

Parfois, vous n'avez pas besoin de load-balancing et d'une seule IP de Service.
Dans ce cas, vous pouvez créer ce que l'on appelle des services "headless", en
spécifiant explicitement "None" pour l'IP du cluster (`.spec.clusterIP`).

Vous pouvez utiliser un service headless pour interfacer avec d'autres
mécanismes de découverte de service, sans être lié à l'implémentation de
Kubernetes.

Pour les services headless, une IP de cluster n'est pas allouée, kube-proxy ne
gère pas ces services et aucun load-balancing ou proxy n'est effectué par la
plateforme pour eux. La configuration automatique de DNS dépend de la définition
ou non de sélecteurs par le service:

### Avec sélecteurs

Pour les services headless qui définissent des sélecteurs, le controlleur des
Endpoints crée des enregistrements `Endpoints` dans l'API, et modifie la
configuration DNS pour renvoyer des enregistrements (adresses) qui pointent
directement vers les `Pods` visés par le `Service`.

### Sans sélecteurs

Pour les services headless qui ne définissent pas de sélecteurs, le contrôleur
des Endpoints ne crée pas d'enregistrements `Endpoints`. Cependant, le système
DNS recherche et configure soit:

- Enregistrements CNAME pour les services de type
  [`ExternalName`](#externalname).
- Un enregistrement pour tous les «Endpoints» qui partagent un nom avec le
  Service, pour tous les autres types.

## Services de publication (ServiceTypes) {#publishing-services-service-types}

Pour certaines parties de votre application (par exemple, les frontaux), vous
souhaiterez peut-être exposer un service sur une adresse IP externe, qui est en
dehors de votre cluster.

Les «ServiceTypes» de Kubernetes vous permettent de spécifier le type de service
que vous souhaitez. La valeur par défaut est «ClusterIP».

Les valeurs de `Type` et leurs comportements sont:

- `ClusterIP`: Expose le service sur une IP interne au cluster. Le choix de
  cette valeur rend le service uniquement accessible à partir du cluster. Il
  s'agit du `ServiceType` par défaut.
- [`NodePort`](#nodeport): Expose le service sur l'IP de chaque nœud sur un port
  statique (le `NodePort`). Un service `ClusterIP`, vers lequel le
  service`NodePort` est automatiquement créé. Vous pourrez contacter le service
  `NodePort`, depuis l'extérieur du cluster, en demandant
  `<NodeIP>: <NodePort>`.
- [`LoadBalancer`](#loadbalancer): Expose le service en externe à l'aide de
  l'équilibreur de charge d'un fournisseur de cloud. Les services `NodePort` et
  `ClusterIP`, vers lesquels les itinéraires de l'équilibreur de charge externe,
  sont automatiquement créés.
- [`ExternalName`](#externalname): Mappe le service au contenu du champ
  `externalName` (par exemple`foo.bar.example.com`), en renvoyant un
  enregistrement `CNAME` avec sa valeur. Aucun proxy d'aucune sorte n'est mis en
  place. {{< note >}} Vous avez besoin de CoreDNS version 1.7 ou supérieure pour
  utiliser le type `ExternalName`. {{< /note >}}

Vous pouvez également utiliser
[Ingress](/fr/docs/concepts/services-networking/ingress) pour exposer votre
service. Ingress n'est pas un type de service, mais il sert de point d'entrée
pour votre cluster. Il vous permet de consolider vos règles de routage en une
seule ressource car il peut exposer plusieurs services sous la même adresse IP.

### Type NodePort {#nodeport}

Si vous définissez le champ `type` sur`NodePort`, le plan de contrôle Kubernetes
alloue un port à partir d'une plage spécifiée par l'indicateur
`--service-node-port-range` (par défaut: 30000-32767). Chaque nœud assure le
proxy de ce port (le même numéro de port sur chaque nœud) vers votre service.
Votre service signale le port alloué dans son champ `.spec.ports[*].nodePort`.

Si vous souhaitez spécifier une ou des adresses IP particulières pour proxyfier
le port, vous pouvez définir l'indicateur `--nodeport-addresses` dans kube-proxy
sur des blocs IP particuliers; cela est pris en charge depuis Kubernetes v1.10.
Cet indicateur prend une liste délimitée par des virgules de blocs IP (par
exemple 10.0.0.0/8, 192.0.2.0/25) pour spécifier les plages d'adresses IP que
kube-proxy doit considérer comme locales pour ce nœud.

Par exemple, si vous démarrez kube-proxy avec l'indicateur
`--nodeport-addresses=127.0.0.0/8`, kube-proxy sélectionne uniquement
l'interface de boucle locale pour les services NodePort. La valeur par défaut
pour `--nodeport-addresses` est une liste vide. Cela signifie que kube-proxy
doit prendre en compte toutes les interfaces réseau disponibles pour NodePort
(qui est également compatible avec les versions antérieures de Kubernetes).

Si vous voulez un numéro de port spécifique, vous pouvez spécifier une valeur
dans le champ `nodePort`. Le plan de contrôle vous attribuera ce port ou
signalera l'échec de la transaction API. Cela signifie que vous devez vous
occuper vous-même des éventuelles collisions de ports. Vous devez également
utiliser un numéro de port valide, celui qui se trouve dans la plage configurée
pour l'utilisation de NodePort.

L'utilisation d'un NodePort vous donne la liberté de configurer votre propre
solution d'équilibrage de charge, de configurer des environnements qui ne sont
pas entièrement pris en charge par Kubernetes, ou même d'exposer directement les
adresses IP d'un ou plusieurs nœuds.

Notez que ce service est visible en tant que `<NodeIP>: spec.ports[*].nodePort`
et `.spec.clusterIP: spec.ports[*].Port`. (Si l'indicateur
`--nodeport-addresses` dans kube-proxy est défini, <NodeIP> serait filtré
NodeIP(s).)

### Type LoadBalancer {#loadbalancer}

Sur les fournisseurs de cloud qui prennent en charge les load balancers
externes, la définition du champ `type` sur`LoadBalancer` provisionne un load
balancer pour votre service. La création réelle du load balancer se produit de
manière asynchrone et les informations sur le load balancer provisionné sont
publiées dans le champ `.status.loadBalancer`. Par exemple:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
  clusterIP: 10.0.171.239
  type: LoadBalancer
status:
  loadBalancer:
    ingress:
      - ip: 192.0.2.127
```

Le trafic provenant du load balancer externe est dirigé vers les Pods backend.
Le fournisseur de cloud décide de la répartition de la charge.

Certains fournisseurs de cloud vous permettent de spécifier le `loadBalancerIP`.
Dans ces cas, le load balancer est créé avec le `loadBalancerIP` spécifié par
l'utilisateur. Si le champ `loadBalancerIP` n'est pas spécifié, le loadBalancer
est configuré avec une adresse IP éphémère. Si vous spécifiez un
`loadBalancerIP` mais que votre fournisseur de cloud ne prend pas en charge la
fonctionnalité, le champ `loadBalancerIP` que vous définissez est ignoré.

{{< note >}} Si vous utilisez SCTP, voir le
[caveat](#caveat-sctp-loadbalancer-service-type) ci-dessous sur le type de
service `LoadBalancer`. {{< /note >}}

{{< note >}}

Sur **Azure**, si vous souhaitez utiliser un type public spécifié par
l'utilisateur `loadBalancerIP`, vous devez d'abord créer une ressource d'adresse
IP publique de type statique. Cette ressource d'adresse IP publique doit se
trouver dans le même groupe de ressources que les autres ressources créées
automatiquement du cluster. Par exemple,
`MC_myResourceGroup_myAKSCluster_eastus`.

Spécifiez l'adresse IP attribuée en tant que loadBalancerIP. Assurez-vous
d'avoir mis à jour le securityGroupName dans le fichier de configuration du
fournisseur de cloud. Pour plus d'informations sur le dépannage
`CreatingLoadBalancerFailed` relatif aux permissions consultez:
[Use a static IP address with the Azure Kubernetes Service (AKS) load balancer](https://docs.microsoft.com/en-us/azure/aks/static-ip)
ou
[CreatingLoadBalancerFailed on AKS cluster with advanced networking](https://github.com/Azure/AKS/issues/357).

{{< /note >}}

#### Load Balancer interne

Dans un environnement mixte, il est parfois nécessaire d'acheminer le trafic des
services à l'intérieur du même bloc d'adresse réseau (virtuel).

Dans un environnement DNS à horizon divisé, vous auriez besoin de deux services
pour pouvoir acheminer le trafic externe et interne vers vos endpoints.

Vous pouvez y parvenir en ajoutant une des annotations suivantes à un service.
L'annotation à ajouter dépend du fournisseur de services cloud que vous
utilisez.

{{< tabs name="service_tabs" >}} {{% tab name="Default" %}} Sélectionnez l'un
des onglets. {{% /tab %}} {{% tab name="GCP" %}}

```yaml
[...]
metadata:
    name: my-service
    annotations:
        cloud.google.com/load-balancer-type: "Internal"
[...]
```

{{% /tab %}} {{% tab name="AWS" %}}

```yaml
[...]
metadata:
    name: my-service
    annotations:
        service.beta.kubernetes.io/aws-load-balancer-internal: "true"
[...]
```

{{% /tab %}} {{% tab name="Azure" %}}

```yaml
[...]
metadata:
    name: my-service
    annotations:
        service.beta.kubernetes.io/azure-load-balancer-internal: "true"
[...]
```

{{% /tab %}} {{% tab name="OpenStack" %}}

```yaml
[...]
metadata:
    name: my-service
    annotations:
        service.beta.kubernetes.io/openstack-internal-load-balancer: "true"
[...]
```

{{% /tab %}} {{% tab name="Baidu Cloud" %}}

```yaml
[...]
metadata:
    name: my-service
    annotations:
        service.beta.kubernetes.io/cce-load-balancer-internal-vpc: "true"
[...]
```

{{% /tab %}} {{% tab name="Tencent Cloud" %}}

```yaml
[...]
metadata:
  annotations:
    service.kubernetes.io/qcloud-loadbalancer-internal-subnetid: subnet-xxxxx
[...]
```

{{% /tab %}} {{< /tabs >}}

#### Prise en charge TLS sur AWS {#ssl-support-on-aws}

Pour une prise en charge partielle de TLS / SSL sur des clusters exécutés sur
AWS, vous pouvez ajouter trois annotations à un service `LoadBalancer`:

```yaml
metadata:
  name: my-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-ssl-cert: arn:aws:acm:us-east-1:123456789012:certificate/12345678-1234-1234-1234-123456789012
```

Le premier spécifie l'ARN du certificat à utiliser. Il peut s'agir soit d'un
certificat d'un émetteur tiers qui a été téléchargé sur IAM, soit d'un
certificat créé dans AWS Certificate Manager.

```yaml
metadata:
  name: my-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: (https|http|ssl|tcp)
```

La deuxième annotation spécifie le protocole utilisé par un pod. Pour HTTPS et
SSL, l'ELB s'attend à ce que le pod s'authentifie sur la connexion chiffrée, �
l'aide d'un certificat.

HTTP et HTTPS sélectionnent le proxy de couche 7: l'ELB met fin à la connexion
avec l'utilisateur, analyse les en-têtes et injecte l'en-tête `X-Forwarded-For`
avec l'adresse IP de l'utilisateur (les pods ne voient que l'adresse IP de l'ELB
à l'autre extrémité de sa connexion) lors du transfert des demandes.

TCP et SSL sélectionnent le proxy de couche 4: l'ELB transfère le trafic sans
modifier les en-têtes.

Dans un environnement à usage mixte où certains ports sont sécurisés et d'autres
non chiffrés, vous pouvez utiliser les annotations suivantes:

```yaml
metadata:
  name: my-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: http
    service.beta.kubernetes.io/aws-load-balancer-ssl-ports: "443,8443"
```

Dans l'exemple ci-dessus, si le service contenait trois ports, «80», «443» et
«8443», alors «443» et «8443» utiliseraient le certificat SSL, mais «80» serait
simplement un proxy HTTP.

A partir de Kubernetes v1.9, vous pouvez utiliser des
[stratégies SSL AWS prédéfinies](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-security-policy-table.html)
avec des écouteurs HTTPS ou SSL pour vos services. Pour voir quelles politiques
sont disponibles, vous pouvez utiliser l'outil de ligne de commande `aws`:

```bash
aws elb describe-load-balancer-policies --query 'PolicyDescriptions[].PolicyName'
```

Vous pouvez ensuite spécifier l'une de ces stratégies à l'aide de l'annotation
"`service.beta.kubernetes.io/aws-load-balancer-ssl-negotiation-policy`"; par
exemple:

```yaml
metadata:
  name: my-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-ssl-negotiation-policy: "ELBSecurityPolicy-TLS-1-2-2017-01"
```

#### Prise en charge du protocole PROXY sur AWS

Pour activer
[protocole PROXY](https://www.haproxy.org/download/1.8/doc/proxy-protocol.txt)
prise en charge des clusters exécutés sur AWS, vous pouvez utiliser l'annotation
de service suivante:

```yaml
metadata:
  name: my-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-proxy-protocol: "*"
```

Depuis la version 1.3.0, l'utilisation de cette annotation s'applique à tous les
ports mandatés par l'ELB et ne peut pas être configurée autrement.

#### Journaux d'accès ELB sur AWS

Il existe plusieurs annotations pour gérer les journaux d'accès aux services ELB
sur AWS.

L'annotation `service.beta.kubernetes.io/aws-load-balancer-access-log-enabled`
contrôle si les journaux d'accès sont activés.

L'annotation
`service.beta.kubernetes.io/aws-load-balancer-access-log-emit-interval` contrôle
l'intervalle en minutes pour la publication des journaux d'accès. Vous pouvez
spécifier un intervalle de 5 ou 60 minutes.

L'annotation
`service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-name`
contrôle le nom du bucket Amazon S3 où les journaux d'accès au load balancer
sont stockés.

L'annotation
`service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-prefix`
spécifie la hiérarchie logique que vous avez créée pour votre bucket Amazon S3.

```yaml
metadata:
  name: my-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-access-log-enabled: "true"
    # Spécifie si les journaux d'accès sont activés pour le load balancer

    service.beta.kubernetes.io/aws-load-balancer-access-log-emit-interval: "60"
    # L'intervalle de publication des journaux d'accès.
    # Vous pouvez spécifier un intervalle de 5 ou 60 (minutes).

    service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-name: "my-bucket"
    # Le nom du bucket Amazon S3 où les journaux d'accès sont stockés

    service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-prefix: "my-bucket-prefix/prod"
    # La hiérarchie logique que vous avez créée pour votre bucket Amazon S3, par exemple `my-bucket-prefix/prod`
```

#### Drainage de connexion sur AWS

Le drainage des connexions pour les ELB classiques peut être géré avec
l'annotation
`service.beta.kubernetes.io / aws-load-balancer-connection-draining-enabled`
définie sur la valeur `true`. L'annotation
`service.beta.kubernetes.io / aws-load-balancer-connection-draining-timeout`
peut également être utilisée pour définir la durée maximale, en secondes, pour
garder les connexions existantes ouvertes avant de désenregistrer les instances.

```yaml
metadata:
  name: my-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-connection-draining-enabled: "true"
    service.beta.kubernetes.io/aws-load-balancer-connection-draining-timeout: "60"
```

#### Autres annotations ELB

Il existe d'autres annotations pour gérer les Elastic Load Balancers décrits
ci-dessous.

```yaml
metadata:
  name: my-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-connection-idle-timeout: "60"
    # Délai, en secondes, pendant lequel la connexion peut être inactive (aucune donnée n'a été envoyée via la connexion) avant d'être fermée par le load balancer

    service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: "true"
    # Spécifie si le load balancing inter-zones est activé pour le load balancer

    service.beta.kubernetes.io/aws-load-balancer-additional-resource-tags: "environment=prod,owner=devops"
    # Une liste de paires clé-valeur séparées par des virgules qui seront enregistrées en tant que balises supplémentaires dans l'ELB.

    service.beta.kubernetes.io/aws-load-balancer-healthcheck-healthy-threshold: ""
    # Nombre de contrôles de santé successifs réussis requis pour qu'un backend soit considéré comme sain pour le trafic.
    # La valeur par défaut est 2, doit être comprise entre 2 et 10

    service.beta.kubernetes.io/aws-load-balancer-healthcheck-unhealthy-threshold: "3"
    # Nombre de contrôles de santé infructueux requis pour qu'un backend soit considéré comme inapte pour le trafic.
    # La valeur par défaut est 6, doit être comprise entre 2 et 10

    service.beta.kubernetes.io/aws-load-balancer-healthcheck-interval: "20"
    # Intervalle approximatif, en secondes, entre les contrôles d'intégrité d'une instance individuelle.
    # La valeur par défaut est 10, doit être comprise entre 5 et 300

    service.beta.kubernetes.io/aws-load-balancer-healthcheck-timeout: "5"
    # Durée, en secondes, pendant laquelle aucune réponse ne signifie l'échec d'un contrôle de santé.
    # Cette valeur doit être inférieure à la valeur service.beta.kubernetes.io/aws-load-balancer-healthcheck-interval.
    # La valeur par défaut est 5, doit être comprise entre 2 et 60

    service.beta.kubernetes.io/aws-load-balancer-extra-security-groups: "sg-53fae93f,sg-42efd82e"
    # Une liste de groupes de sécurité supplémentaires à ajouter à l'ELB
```

#### Prise en charge du load balancer réseau sur AWS {#aws-nlb-support}

{{< feature-state for_k8s_version="v1.15" state="beta" >}}

Pour utiliser un load balancer réseau sur AWS, utilisez l'annotation
`service.beta.kubernetes.io/aws-load-balancer-type` avec la valeur définie sur
`nlb`.

```yaml
metadata:
  name: my-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
```

{{< note >}} NLB ne fonctionne qu'avec certaines classes d'instance; voir la
[documentation AWS](http://docs.aws.amazon.com/elasticloadbalancing/latest/network/target-group-register-targets.html#register-deregister-targets)
sur Elastic Load Balancing pour une liste des types d'instances pris en charge.
{{< /note >}}

Contrairement aux équilibreurs de charge élastiques classiques, les équilibreurs
de charge réseau (NLB) transfèrent l'adresse IP du client jusqu'au nœud. Si un
service est `.spec.externalTrafficPolicy` est réglé sur `Cluster`, l'adresse IP
du client n'est pas propagée aux pods finaux.

En définissant `.spec.externalTrafficPolicy` à `Local`, les adresses IP des
clients sont propagées aux pods finaux, mais cela peut entraîner une répartition
inégale du trafic. Les nœuds sans pods pour un service LoadBalancer particulier
échoueront au contrôle de santé du groupe cible NLB sur le
`.spec.healthCheckNodePort` attribué automatiquement et ne recevront aucun
trafic.

Pour obtenir un trafic uniforme, utilisez un DaemonSet ou spécifiez un
[pod anti-affinity](/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity)
pour ne pas localiser sur le même noeud.

Vous pouvez également utiliser les services NLB avec l'annotation
[load balancer internal](/docs/concepts/services-networking/service/#internal-load-balancer).

Pour que le trafic client atteigne des instances derrière un NLB, les groupes de
sécurité du nœud sont modifiés avec les règles IP suivantes:

| Rule           | Protocol | Port(s)                                                                             | IpRange(s)                                                 | IpRange Description                                |
| -------------- | -------- | ----------------------------------------------------------------------------------- | ---------------------------------------------------------- | -------------------------------------------------- |
| Health Check   | TCP      | NodePort(s) (`.spec.healthCheckNodePort` for `.spec.externalTrafficPolicy = Local`) | VPC CIDR                                                   | kubernetes.io/rule/nlb/health=\<loadBalancerName\> |
| Client Traffic | TCP      | NodePort(s)                                                                         | `.spec.loadBalancerSourceRanges` (defaults to `0.0.0.0/0`) | kubernetes.io/rule/nlb/client=\<loadBalancerName\> |
| MTU Discovery  | ICMP     | 3,4                                                                                 | `.spec.loadBalancerSourceRanges` (defaults to `0.0.0.0/0`) | kubernetes.io/rule/nlb/mtu=\<loadBalancerName\>    |

Afin de limiter les IP clientes pouvant accéder à l'équilibreur de charge
réseau, spécifiez `loadBalancerSourceRanges`.

```yaml
spec:
  loadBalancerSourceRanges:
    - "143.231.0.0/16"
```

{{< note >}} Si `.spec.loadBalancerSourceRanges` n'est pas défini, Kubernetes
autorise le trafic de `0.0.0.0/0` vers les groupes de sécurité des nœuds. Si les
nœuds ont des adresses IP publiques, sachez que le trafic non NLB peut également
atteindre toutes les instances de ces groupes de sécurité modifiés.

{{< /note >}}

#### Autres annotations CLB sur Tencent Kubernetes Engine (TKE)

Il existe d'autres annotations pour la gestion des équilibreurs de charge cloud
sur TKE, comme indiqué ci-dessous.

```yaml
    metadata:
      name: my-service
      annotations:
        # Lier des load balancers avec des nœuds spécifiques
        service.kubernetes.io/qcloud-loadbalancer-backends-label: key in (value1, value2)

        # ID d'un load balancer existant
        service.kubernetes.io/tke-existed-lbid：lb-6swtxxxx

        # Paramètres personnalisés pour le load balancer (LB), ne prend pas encore en charge la modification du type LB
        service.kubernetes.io/service.extensiveParameters: ""

        # Paramètres personnalisés pour le listener LB
        service.kubernetes.io/service.listenerParameters: ""

        # Spécifie le type de Load balancer;
        # valeurs valides: classic (Classic Cloud Load Balancer) ou application (Application Cloud Load Balancer)
        service.kubernetes.io/loadbalance-type: xxxxx

        # Spécifie la méthode de facturation de la bande passante du réseau public;
        # valid values: TRAFFIC_POSTPAID_BY_HOUR(bill-by-traffic) and BANDWIDTH_POSTPAID_BY_HOUR (bill-by-bandwidth).
        service.kubernetes.io/qcloud-loadbalancer-internet-charge-type: xxxxxx

        # Spécifie la valeur de bande passante (plage de valeurs: [1,2000] Mbps).
        service.kubernetes.io/qcloud-loadbalancer-internet-max-bandwidth-out: "10"

        # Lorsque cette annotation est définie, les équilibreurs de charge n'enregistrent que les nœuds sur lesquels le pod s'exécute, sinon tous les nœuds seront enregistrés.
        service.kubernetes.io/local-svc-only-bind-node-with-pod: true
```

### Type ExternalName {#externalname}

Les services de type ExternalName mappent un service à un nom DNS, et non à un
sélecteur standard tel que `my-service` ou `cassandra`. Vous spécifiez ces
services avec le paramètre `spec.externalName`.

Cette définition de service, par exemple, mappe le service `my-service` dans
l'espace de noms`prod` à `my.database.example.com`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
```

{{< note >}} ExternalName accepte une chaîne d'adresse IPv4, mais en tant que
noms DNS composés de chiffres, et non en tant qu'adresse IP. Les noms externes
qui ressemblent aux adresses IPv4 ne sont pas résolus par CoreDNS ou
ingress-nginx car ExternalName est destiné à spécifier un nom DNS canonique.
Pour coder en dur une adresse IP, pensez à utiliser des
[Services headless](#headless-services). {{< /note >}}

Lors de la recherche de l'hôte `my-service.prod.svc.cluster.local`, le service
DNS du cluster renvoie un enregistrement`CNAME` avec la valeur
`my.database.example.com`. L'accès à «mon-service» fonctionne de la même manière
que les autres services, mais avec la différence cruciale que la redirection se
produit au niveau DNS plutôt que via un proxy ou un transfert. Si vous décidez
ultérieurement de déplacer votre base de données dans votre cluster, vous pouvez
démarrer ses pods, ajouter des sélecteurs ou des Endpoints appropriés et
modifier le `type` du service.

{{< warning >}} Vous pouvez rencontrer des difficultés à utiliser ExternalName
pour certains protocoles courants, notamment HTTP et HTTPS. Si vous utilisez
ExternalName, le nom d'hôte utilisé par les clients à l'intérieur de votre
cluster est différent du nom référencé par ExternalName.

Pour les protocoles qui utilisent des noms d'hôtes, cette différence peut
entraîner des erreurs ou des réponses inattendues. Les requêtes HTTP auront un
en-tête `Host:` que le serveur d'origine ne reconnaît pas; Les serveurs TLS ne
pourront pas fournir de certificat correspondant au nom d'hôte auquel le client
s'est connecté. {{< /warning >}}

{{< note >}} Cette section est redevable à l'article
[Kubernetes Tips - Part 1](https://akomljen.com/kubernetes-tips-part-1/)
d'[Alen Komljen](https://akomljen.com/). {{< /note >}}

### IP externes

S'il existe des adresses IP externes qui acheminent vers un ou plusieurs nœuds
de cluster, les services Kubernetes peuvent être exposés sur ces "IP externes".
Le trafic qui pénètre dans le cluster avec l'IP externe (en tant qu'IP de
destination), sur le port de service, sera routé vers l'un des Endpoints de
service. Les `externalIPs` ne sont pas gérées par Kubernetes et relèvent de la
responsabilité de l'administrateur du cluster.

Dans la spécification de service, «externalIPs» peut être spécifié avec
n'importe lequel des «ServiceTypes». Dans l'exemple ci-dessous, "`my-service`"
peut être consulté par les clients sur "`80.11.12.10:80`" (`externalIP:port`)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 9376
  externalIPs:
    - 80.11.12.10
```

## Lacunes

Le proxy fonctionnant dans l'espace utilisateur pour les VIP peut fonctionner �
petite ou moyenne échelle, mais montrera ses limites dans de très grands
clusters avec des milliers de services. La
[proposition de conception originale pour les portails](http://issue.k8s.io/1107)
a plus de détails à ce sujet.

L'utilisation du proxy de l'espace utilisateur masque l'adresse IP source d'un
paquet accédant à un service. Cela rend certains types de filtrage réseau
(pare-feu) impossibles. Le mode proxy iptables n'obscurcit pas les adresses IP
source dans le cluster, mais il affecte toujours les clients passant par un
`LoadBalancer` ou un `NodePort`.

Le champ `Type` est conçu comme une fonctionnalité imbriquée - chaque niveau
s'ajoute au précédent. Cela n'est pas strictement requis sur tous les
fournisseurs de cloud (par exemple, Google Compute Engine n'a pas besoin
d'allouer un `NodePort` pour faire fonctionner `LoadBalancer`, mais AWS le fait)
mais l'API actuelle le requiert.

## Implémentation IP virtuelle {#the-gory-details-of-virtual-ips}

Les informations précédentes devraient être suffisantes pour de nombreuses
personnes qui souhaitent simplement utiliser les Services. Cependant, il se
passe beaucoup de choses dans les coulisses qui méritent d'être comprises.

### Éviter les collisions

L'une des principales philosophies de Kubernetes est que vous ne devez pas être
exposé à des situations qui pourraient entraîner l'échec de vos actions sans
aucune faute de votre part. Pour la conception de la ressource Service, cela
signifie de ne pas vous faire choisir votre propre numéro de port si ce choix
pourrait entrer en collision avec le choix de quelqu'un d'autre. C'est un échec
d'isolement.

Afin de vous permettre de choisir un numéro de port pour vos Services, nous
devons nous assurer qu'aucun deux Services ne peuvent entrer en collision.
Kubernetes le fait en attribuant à chaque service sa propre adresse IP.

Pour garantir que chaque service reçoit une adresse IP unique, un allocateur
interne met à jour atomiquement une carte d'allocation globale dans
{{< glossary_tooltip term_id="etcd" >}} avant de créer chaque service. L'objet
de mappage doit exister dans le registre pour que les services obtiennent des
affectations d'adresse IP, sinon les créations échoueront avec un message
indiquant qu'une adresse IP n'a pas pu être allouée.

Dans le plan de contrôle, un contrôleur d'arrière-plan est responsable de la
création de cette carte (nécessaire pour prendre en charge la migration à partir
d'anciennes versions de Kubernetes qui utilisaient le verrouillage en mémoire).
Kubernetes utilise également des contrôleurs pour vérifier les affectations non
valides (par exemple en raison d'une intervention de l'administrateur) et pour
nettoyer les adresses IP allouées qui ne sont plus utilisées par aucun service.

### Service IP addresses {#ips-and-vips}

Contrairement aux adresses IP des pods, qui acheminent réellement vers une
destination fixe, les adresses IP des services ne sont pas réellement répondues
par un seul hôte. Au lieu de cela, kube-proxy utilise iptables (logique de
traitement des paquets sous Linux) pour définir les adresses IP _virtual_ qui
sont redirigées de manière transparente selon les besoins. Lorsque les clients
se connectent au VIP, leur trafic est automatiquement transporté vers un
Endpoint approprié. Les variables d'environnement et DNS pour les services sont
en fait remplis en termes d'adresse IP virtuelle (et de port) du service.

kube-proxy prend en charge trois modes proxy &mdash; espace utilisateur,
iptables et IPVS &mdash; qui fonctionnent chacun légèrement différemment.

#### Userspace

À titre d'exemple, considérons l'application de traitement d'image décrite
ci-dessus. Lorsque le service backend est créé, le maître Kubernetes attribue
une adresse IP virtuelle, par exemple 10.0.0.1. En supposant que le port de
service est 1234, le service est observé par toutes les instances kube-proxy
dans le cluster. Lorsqu'un proxy voit un nouveau service, il ouvre un nouveau
port aléatoire, établit une redirection iptables de l'adresse IP virtuelle vers
ce nouveau port et commence à accepter les connexions sur celui-ci.

Lorsqu'un client se connecte à l'adresse IP virtuelle du service, la règle
iptables entre en jeu et redirige les paquets vers le propre port du proxy. Le
“Service proxy” choisit un backend, et commence le proxy du trafic du client
vers le backend.

Cela signifie que les propriétaires de services peuvent choisir le port de leur
choix sans risque de collision. Les clients peuvent simplement se connecter �
une adresse IP et à un port, sans savoir à quels pods ils accèdent réellement.

#### iptables

Considérons à nouveau l'application de traitement d'image décrite ci-dessus.
Lorsque le service backend est créé, le plan de contrôle Kubernetes attribue une
adresse IP virtuelle, par exemple 10.0.0.1. En supposant que le port de service
est 1234, le service est observé par toutes les instances de kube-proxy dans le
cluster. Lorsqu'un proxy voit un nouveau service, il installe une série de
règles iptables qui redirigent de l'adresse IP virtuelle vers des règles par
service. Les règles par service sont liées aux règles des Endpoints qui
redirigent le trafic (à l'aide du NAT de destination) vers les backends.

Lorsqu'un client se connecte à l'adresse IP virtuelle du service, la règle
iptables entre en jeu. Un backend est choisi (soit en fonction de l'affinité de
la session, soit au hasard) et les paquets sont redirigés vers le backend.
Contrairement au proxy de l'espace utilisateur, les paquets ne sont jamais
copiés dans l'espace utilisateur, le proxy de kube n'a pas besoin d'être exécuté
pour que l'adresse IP virtuelle fonctionne et les nœuds voient le trafic
provenant de l'adresse IP du client non modifiée.

Ce même flux de base s'exécute lorsque le trafic arrive via un port de nœud ou
via un load balancer, bien que dans ces cas, l'adresse IP du client soit
modifiée.

#### IPVS

Les opérations iptables ralentissent considérablement dans un cluster à grande
échelle, par exemple 10000 services. IPVS est conçu pour l'équilibrage de charge
et basé sur des tables de hachage dans le noyau. Ainsi, vous pouvez obtenir une
cohérence des performances dans un grand nombre de services à partir d'un
kube-proxy basé sur IPVS. De plus, kube-proxy basé sur IPVS a des algorithmes
d'équilibrage de charge plus sophistiqués (le moins de connexions, localité,
pondéré, persistance).

## Objet API

Le service est une ressource de niveau supérieur dans l'API REST Kubernetes.
Vous pouvez trouver plus de détails sur l'objet API sur: [Service API
object](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#service-v1-core).

## Protocoles pris en charge {#protocol-support}

### TCP

{{< feature-state for_k8s_version="v1.0" state="stable" >}}

Vous pouvez utiliser TCP pour tout type de service, et c'est le protocole réseau
par défaut.

### UDP

{{< feature-state for_k8s_version="v1.0" state="stable" >}}

Vous pouvez utiliser UDP pour la plupart des services. Pour Services de type
LoadBalancer, la prise en charge UDP dépend du fournisseur de cloud offrant
cette fonctionnalité.

### HTTP

{{< feature-state for_k8s_version="v1.1" state="stable" >}}

Si votre fournisseur de cloud le prend en charge, vous pouvez utiliser un
service dans le mode LoadBalancer pour configurer le proxy inverse HTTP / HTTPS
externe, transmis au Endpoints du Service.

{{< note >}} Vous pouvez aussi utiliser
{{< glossary_tooltip term_id="ingress" >}} à la place du service pour exposer
les services HTTP/HTTPS. {{< /note >}}

### Protocole PROXY

{{< feature-state for_k8s_version="v1.1" state="stable" >}}

Si votre fournisseur de cloud le prend en charge(eg,
[AWS](/docs/concepts/cluster-administration/cloud-providers/#aws)), vous pouvez
utiliser un service en mode LoadBalancer pour configurer un load balancer en
dehors de Kubernetes lui-même, qui transmettra les connexions préfixées par
[PROXY protocol](https://www.haproxy.org/download/1.8/doc/proxy-protocol.txt).

Le load balancer enverra une première série d'octets décrivant la connexion
entrante, similaire à cet exemple

```
PROXY TCP4 192.0.2.202 10.0.42.7 12345 7\r\n
```

suivi des données du client.

### SCTP

{{< feature-state for_k8s_version="v1.12" state="alpha" >}}

Kubernetes prend en charge SCTP en tant que valeur de «protocole» dans les
définitions de Service, Endpoint, NetworkPolicy et Pod en tant que
fonctionnalité alpha. Pour activer cette fonction, l'administrateur du cluster
doit activer le flag `SCTPSupport` sur l'apiserver, par exemple,
`--feature-gates=SCTPSupport=true,…`.

When the feature gate is enabled, you can set the `protocol` field of a Service,
Endpoint, NetworkPolicy or Pod to `SCTP`. Kubernetes sets up the network
accordingly for the SCTP associations, just like it does for TCP connections.

#### Avertissements {#caveat-sctp-overview}

##### Prise en charge des associations SCTP multi-hôtes {#caveat-sctp-multihomed}

{{< warning >}} La prise en charge des associations SCTP multi-hôtes nécessite
que le plug-in CNI puisse prendre en charge l'attribution de plusieurs
interfaces et adresses IP à un pod.

Le NAT pour les associations SCTP multi-hôtes nécessite une logique spéciale
dans les modules de noyau correspondants. {{< /warning >}}

##### Service avec type=LoadBalancer {#caveat-sctp-loadbalancer-service-type}

{{< warning >}} Vous ne pouvez créer un service de type LoadBalancer avec SCTP
que si le fournisseur de load balancer supporte SCTP comme protocole. Sinon, la
demande de création de service est rejetée. L'ensemble actuel de fournisseurs de
load balancer cloud (Azure, AWS, CloudStack, GCE, OpenStack) ne prennent pas en
charge SCTP. {{< /warning >}}

##### Windows {#caveat-sctp-windows-os}

{{< warning >}} SCTP n'est pas pris en charge sur les nœuds Windows.
{{< /warning >}}

##### Userspace kube-proxy {#caveat-sctp-kube-proxy-userspace}

{{< warning >}} Le kube-proxy ne prend pas en charge la gestion des associations
SCTP lorsqu'il est en mode userspace. {{< /warning >}}

## Futurs développements

À l'avenir, la stratégie de proxy pour les services peut devenir plus nuancée
que le simple équilibrage alterné, par exemple master-elected ou sharded. Nous
prévoyons également que certains services auront des load balancer «réels»,
auquel cas l'adresse IP virtuelle y transportera simplement les paquets.

Le projet Kubernetes vise à améliorer la prise en charge des services L7 (HTTP).

Le projet Kubernetes prévoit d'avoir des modes d'entrée plus flexibles pour les
services, qui englobent les modes ClusterIP, NodePort et LoadBalancer actuels et
plus encore.

{{% /capture %}}

{{% capture whatsnext %}}

- Voir
  [Connecting Applications with Services](/docs/concepts/services-networking/connect-applications-service/)
- Voir [Ingress](/docs/concepts/services-networking/ingress/)
- Voir [Endpoint Slices](/docs/concepts/services-networking/endpoint-slices/)

{{% /capture %}}
