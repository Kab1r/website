---
title: Kubernetes cloud-controller-manager
content_template: templates/concept
---

{{% capture overview %}}

{{< feature-state state="beta" >}}

Kubernetes v1.6 a introduit un nouveau binaire appelé
`cloud-controller-manager`. `cloud-controller-manager` est un démon qui intègre
des boucles de contrôle spécifiques au cloud. Ces boucles de contrôle
spécifiques au cloud étaient à l’origine dans le binaire
`kube-controller-manager`. Étant donné que les fournisseurs de cloud développent
et publient à un rythme différent de celui du projet Kubernetes, fournir une
abstraction du code du `cloud-controller-manager` permet aux fournisseurs de
cloud d’évoluer indépendamment du code Kubernetes principal.

Le `cloud-controller-manager` peut être lié à tout fournisseur de cloud
satisfaisant l'interface
[cloudprovider.Interface](https://github.com/kubernetes/cloud-provider/blob/master/cloud.go).
Pour des raisons de retro-compatibilité, le
[cloud-controller-manager](https://github.com/kubernetes/kubernetes/tree/master/cmd/cloud-controller-manager)
fourni dans le projet de base Kubernetes utilise les mêmes bibliothèques ​​que
`kube-controller-manager`. Les fournisseurs de cloud déjà pris en charge
nativement par Kubernetes devraient utiliser le cloud-controller-manager
​disponible ​dans le code de Kubernetes pour effectuer une transition visant �
faire sortir cette prise en charge du code de Kubernetes. Dans les futures
versions de Kubernetes, tous les cloud-controller-manager seront développés en
dehors du projet de base de Kubernetes géré par des sig leads ou des
fournisseurs de cloud.

{{% /capture %}}

{{% capture body %}}

## Administration

### Pré-requis

Chaque cloud a ses propres exigences pour l'exécution de sa propre intégration,
ces exigences sont similaires à celles requises pour l'exécution de
`kube-controller-manager`. En règle générale, vous aurez besoin de:

- cloud authentification/autorisation: votre cloud peut nécessiter un jeton ou
  des règles IAM pour permettre l'accès à leurs API
- kubernetes authentification/autorisation: cloud-controller-manager peut avoir
  besoin de règles RBAC définies pour parler à l'apiserver kubernetes
- la haute disponibilité: Comme pour kube-controller-manager, vous pouvez
  souhaiter une configuration hautement disponible pour le cloud controller
  mananger en utilisant l'élection de leader (activée par défaut).

### Lancer cloud-controller-manager

L'exécution réussie de cloud-controller-manager nécessite certaines
modifications de la configuration de votre cluster.

- `kube-apiserver` et `kube-controller-manager` NE DOIVENT PAS spécifier
  l'option `--cloud-provider`. Cela garantit qu'il n'exécutera aucune boucle
  spécifique au cloud qui serait exécutée par le cloud-controller-manager. À
  l'avenir, cet indicateur sera rendu obsolète et supprimé.
- `kubelet` doit s'exécuter avec `--cloud-provider=external`. C’est pour nous
  assurer que le kubelet est conscient qu'il doit être initialisé par le
  cloud-controller-manager avant qu'il ne commence à travailler.

N'oubliez pas que la configuration de votre cluster pour utiliser le
cloud-controller-manager changera le comportement de votre cluster de plusieurs
façons:

- Les kubelets lancés avec `--cloud-provider=external` auront un marquage
  `node.cloudprovider.kubernetes.io/uninitialized` avec un effet `NoSchedule`
  pendant l'initialisation. Cela indique que le nœud nécessite une seconde
  initialisation à partir d'un contrôleur externe avant de pouvoir planifier un
  travail. Notez que si le cloud-controller-manager n'est pas disponible, les
  nouveaux nœuds du cluster ne seront pas valides. Le marquage est important car
  le planificateur peut nécessiter des informations spécifiques au cloud �
  propos des nœuds, telles que leur région ou leur type (CPU performant, gpu,
  mémoire importante, instance ponctuelle, etc.).
- Les informations relatives aux nœuds s'exécutant dans le cloud ne seront plus
  récupérées à l'aide de métadonnées locales, mais tous les appels d'API pour
  récupérer les informations de ces nœuds passeront par le
  cloud-controller-manager. Cela peut signifier que vous pouvez restreindre
  l'accès à votre API de cloud sur les kubelets pour une sécurité accrue. Pour
  les clusters de plus grande taille, vous voudrez peut-être déterminer si le
  cloud-controller-manager atteindra les limites de requêtes sur les API de
  votre fournisseur de cloud puisqu'il est désormais responsable de la
  quasi-totalité des appels d'API vers votre cloud depuis le cluster.

À partir de la version 1.8, le cloud-controller-manager peut implémenter:

- contrôleur de nœud - responsable de la mise à jour des nœud kubernetes �
  l’aide des API de cloud et de la suppression des nœud kubernetes supprimés sur
  votre cloud.
- contrôleur de service - responsable des loadbalancers sur votre cloud vers des
  services de type LoadBalancer.
- contrôleur de route - responsable de la configuration des routes réseau sur
  votre cloud
- toute autre fonctionnalité que vous voudriez implémenter si vous exécutez en
  dehors de l'arborescence de Kubernetes.

## Exemples

Si vous utilisez un cloud actuellement pris en charge nativement dans Kubernetes
et souhaitez adopter le cloud-controller-manager, reportez-vous à la section
[cloud-controller-manager dans kubernetes core](https://github.com/kubernetes/kubernetes/tree/master/cmd/cloud-controller-manager).

Pour les cloud-controller-manager ne faisant pas partie de Kubernetes, vous
pouvez trouver les projets respectifs dans des dépôts maintenus par des
fournisseurs de cloud ou des sig leads.

- [DigitalOcean](https://github.com/digitalocean/digitalocean-cloud-controller-manager)
- [keepalived](https://github.com/munnerz/keepalived-cloud-provider)
- [Oracle Cloud Infrastructure](https://github.com/oracle/oci-cloud-controller-manager)
- [Rancher](https://github.com/rancher/rancher-cloud-controller-manager)
- [Scaleway](https://github.com/scaleway/scaleway-cloud-controller-manager)

Pour les fournisseurs qui se trouvent déjà dans Kubernetes, vous pouvez exécuter
le cloud-controller-manager dans l'arborescence en tant que Daemonset dans votre
cluster. Utilisez ce qui suit comme guide:

{{< codenew file="admin/cloud/ccm-example.yaml" >}}

## Limitations

L'exécution du cloud-controller-manager est soumise à quelques limitations. Bien
que ces limitations soient levées dans les prochaines versions, il est important
que vous connaissiez ces limitations pour les charges de travail de production.

### Prise en charge des volumes

Le cloud-controller-manager n'implémente aucun des contrôleurs de volume trouvés
dans `kube-controller-manager` car les intégrations de volume nécessitent
également une coordination avec les kubelets. Au fur et à mesure de l'évolution
de CSI (interface de stockage de conteneur) et de la prise en charge renforcée
des plug-ins de volume flexible, le cloud-controller-manager prendra en charge
le support nécessaire afin que les clouds puissent pleinement s'intégrer aux
volumes. Pour en savoir plus sur les plug-ins de volume CSI en dehors des
sources de Kubernetes consultez
[ceci](https://github.com/kubernetes/features/issues/178).

### Charge sur les APIs cloud

Dans l'architecture précédente pour les fournisseurs de cloud, nous utilisions
des kubelets utilisant un service de métadonnées local pour extraire des
informations sur les nœuds. Avec cette nouvelle architecture, nous comptons
désormais entièrement sur les cloud-controller-manager pour extraire les
informations de tous les nœuds. Pour les très grand clusters, vous devez
envisager les goulots d'étranglement tels que les besoins en ressources et la
limitation de la vitesse des APIs de votre fournisseur cloud.

### Problème de l'oeuf et de la poule

L'objectif du projet des cloud-controller-manager est de dissocier le
développement des fonctionnalités de cloud computing du projet de base
Kubernetes. Malheureusement, de nombreux aspects du projet Kubernetes supposent
que les fonctionnalités de fournisseur de cloud soient étroitement intégrées au
projet. Par conséquent, l'adoption de cette nouvelle architecture peut créer
plusieurs situations dans lesquelles une demande d'informations auprès d'un
fournisseur de cloud est demandée, mais le cloud-controller-manager peut ne pas
être en mesure de renvoyer ces informations sans que la demande d'origine soit
complète.

La fonctionnalité d’amorçage TLS dans Kubelet en est un bon exemple.
Actuellement, l’amorçage TLS suppose que Kubelet aie la possibilité de demander
au fournisseur de cloud (ou à un service de métadonnées local) tous ses types
d’adresses (privé, public, etc.), mais le cloud-controller-manager ne peut pas
définir les types d’adresse d’un nœud sans être initialisé dans le système. Ce
qui nécessite que le kubelet possède des certificats TLS pour communiquer avec
l’apiserver.

À mesure que cette initiative évoluera, des modifications seront apportées pour
résoudre ces problèmes dans les prochaines versions.

## Développer votre propre cloud-controller-manager

Pour créer et développer votre propre cloud-controller-manager, lisez la
documentation
[Développer un cloud-controller-manager](/docs/tasks/administer-cluster/developing-cloud-controller-manager.md).

{{% /capture %}}
