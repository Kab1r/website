---
title: Traduction de la documentation Kubernetes
content_template: templates/concept
card:
  name: contribute
  weight: 30
  title: Translating the docs
---

{{% capture overview %}}

La documentation de Kubernetes est disponible dans plusieurs langues. Nous vous
encourageons à ajouter de nouvelles
[traductions](https://blog.mozilla.org/l10n/2011/12/14/i18n-vs-l10n-whats-the-diff/)!

{{% /capture %}}

{{% capture body %}}

## Commencer

Les traductions doivent avoir certains pré-requis pour le workflow (_comment_
traduire) et la sortie (_quoi_ traduire) avant de publier.

Pour ajouter une nouvelle localisation de la documentation de Kubernetes, vous
devrez mettre à jour le site web en modifiant le paramètre
[site configuration](#modify-the-site-configuration) et
[directory structure](#add-a-new-localization-directory). Alors vous pouvez
commencer la [traduction de documents](#translating-documents)!

{{< note >}} Pour un exemple lié à la localisation
[pull request](../create-pull-request), consultez
[cette pull request](https://github.com/kubernetes/website/pull/8636) vers le
[dépôt Kubernetes website](https://github.com/kubernetes/website) et concernant
l'ajout de la localisation coréenne à la documentation de Kubernetes.
{{< /note >}}

Indiquez à Kubernetes SIG Docs que vous souhaitez créer une traduction!
Rejoignez le canal Slack
[SIG Docs](https://kubernetes.slack.com/messages/C1J0BPD2M/). Nous sommes
heureux de vous aider à démarrer et à répondre à toutes vos questions.

Toutes les équipes de traduction doivent être autonomes avec leurs propres
ressources. Nous sommes heureux d'accueillir votre travail, mais nous ne pouvons
pas le traduire pour vous.

### Fork et cloner le dépôt

D'abord, [créez votre fork](https://help.github.com/articles/fork-a-repo/) du
dépôt [kubernetes/website](https://github.com/kubernetes/website).

Ensuite, clonez ce dépôt et mettez vous dedans (avec une commande `cd`):

```shell
git clone https://github.com/kubernetes/website
cd website
```

{{< note >}} Les contributeurs de `k/website` doivent
[créer un fork](/docs/contribute/start/#improve-existing-content) à partir
duquel les pull requests seront ouvertes. Pour les localisations, nous demandons
en outre que :

1. Les approbateurs d'équipe ouvrent des branches de développement directement �
   partir de https://github.com/kubernetes/website.
2. Les contributeurs à la localisation travaillent à partir de forks, avec des
   branches basées sur la branche de développement actuelle.

Cela s'explique par le fait que les projets de localisation sont des efforts de
collaboration sur des branches à long terme, similaires aux branches de
développement pour le cycle de release de Kubernetes. Pour plus d'informations
sur les pull request de localisation, voir
["branching strategy"](#branching-strategy). {{< /note >}}

### Trouvez votre code de langue à deux lettres

Consultez la
[norme ISO 639-1](https://www.loc.gov/standards/iso639-2/php/code_list.php) pour
le code de pays en deux lettres de votre localisation. Par exemple, le code �
deux lettres pour l'allemand est `de`.

{{< note >}} These instructions use the
[ISO 639-1](https://www.loc.gov/standards/iso639-2/php/code_list.php) language
code for German (`de`) as an example. {{< /note >}}

### Modifier la configuration du site

Le site web de Kubernetes utilise Hugo comme son web framework. La configuration
Hugo du site Web se trouve dans le fichier
[`config.toml`](https://github.com/kubernetes/website/tree/master/config.toml).
Pour prendre en charge une nouvelle localisation, vous devrez modifier
`config.toml`.

Ajoutez un bloc de configuration pour la nouvelle langue dans `config.toml`,
sous le bloc `[languages]` existant. Le bloc allemand, par exemple, ressemble �
:

```toml
[languages.de]
title = "Kubernetes"
description = "Produktionsreife Container-Verwaltung"
languageName = "Deutsch"
contentDir = "content/de"
weight = 3
```

Lors de l'attribution d'un paramètre de `weight` à votre bloc, trouvez le bloc
de langue ayant le `weight` le plus élevé et ajoutez 1 à cette valeur.

Pour plus d'informations sur le support multilingue de Hugo, voir
"[Multilingual Mode](https://gohugo.io/content-management/multilingual/)".

### Ajouter un nouveau répertoire de localisation

Ajoutez un sous-répertoire spécifique à la langue dans le répertoire
[`content`](https://github.com/kubernetes/website/tree/master/content) du dépôt.
Par exemple, le code à deux lettres pour l'allemand est "de" :

```shell
mkdir content/de
```

### Ajouter un README localisé

Pour guider les autres contributeurs à la localisation, ajoutez un nouveau
[`README-**.md`](https://help.github.com/articles/about-readmes/) au plus haut
niveau de k/website, où `**` est le code de langue à deux lettres. Par exemple,
un fichier README allemand serait `README-de.md`.

Fournir des conseils aux contributeurs à la localisation dans le fichier
localisé `README-**.md`. Incluez les mêmes informations que celles contenues
dans `README.md`ainsi que :

- Un point de contact pour le projet de localisation
- Toute information spécifique à la localisation

Après avoir créé le fichier README localisé, ajoutez un lien vers le fichier �
partir du fichier anglais principal, [`README.md`'s Localizing Kubernetes
Documentation] et incluez les coordonnées des personnes-ressources en anglais.
Vous pouvez fournir un identifiant GitHub, une adresse e-mail,
[Slack channel](https://slack.com/), ou toute autre méthode de contact.

## Translating documents

Localiser _toute_ la documentation de Kubernetes est une tâche énorme. Il n'y a
pas de mal à commencer petit et progresser avec le temps.

Au minimum, toutes les localisations doivent inclure :

| Description  | URLs                                                                                                                             |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------- |
| Home         | [All heading and subheading URLs](/docs/home/)                                                                                   |
| Setup        | [All heading and subheading URLs](/docs/setup/)                                                                                  |
| Tutorials    | [Kubernetes Basics](/docs/tutorials/kubernetes-basics/), [Hello Minikube](/docs/tutorials/stateless-application/hello-minikube/) |
| Site strings | [All site strings in a new localized TOML file](https://github.com/kubernetes/website/tree/master/i18n)                          |

Les documents traduits doivent résider dans leur propre sous-répertoire
`content/**/`, mais sinon suivre le même chemin URL que la source anglaise. Par
exemple, pour préparer le tutoriel
[Kubernetes Basics](/docs/tutorials/kubernetes-basics/) à traduire en allemand,
créez un sous-dossier sous le dossier `content/de/' et copiez la source anglaise
:

```shell
mkdir -p content/de/docs/tutorials
cp content/en/docs/tutorials/kubernetes-basics.md content/de/docs/tutorials/kubernetes-basics.md
```

Pour un exemple de demande liée à la localisation
[pull request](.../create-pull-request),
[this pull request](https://github.com/kubernetes/website/pull/10471) au
[Kubernetes website repo](https://github.com/kubernetes/website) a ajouté la
localisation coréenne aux documents Kubernetes.

### Fichiers sources

Les localisations doivent utiliser les fichiers anglais de la version la plus
récente comme source. La version la plus récente est **{{< latest-version >}}**.

Pour trouver les fichiers sources de la version la plus récente :

1. Accédez au dépôt du site web de Kubernetes à l'adresse suivante
   https://github.com/kubernetes/website.
2. Sélectionnez la branche `release-1.X' pour la version la plus récente.

La dernière version est **{{< latest-version >}}**, donc la branche de la
release la plus récente est [`{{< release-branch
>}}`](https://github.com/kubernetes/website/tree/{{< release-branch >}}).

### Chaînes de sites en i18n/

Les localisations doivent inclure le contenu des éléments suivants
[`i18n/en.toml`](https://github.com/kubernetes/website/blob/master/i18n/en.toml)
dans un nouveau fichier spécifique à la langue. Prenons l'allemand comme exemple
: `i18n/de.toml`.

Ajouter un nouveau fichier de localisation dans `i18n/`. Par exemple, avec
l'allemand (de) :

```shell
cp i18n/en.toml i18n/de.toml
```

Traduisez ensuite la valeur de chaque chaîne de caractères :

```TOML
[docs_label_i_am]
other = "ICH BIN..."
```

La localisation des chaînes de caractères du site vous permet de personnaliser
le texte et les fonctionnalités du site : par exemple, le texte du copyright
légal dans le pied de page de chaque page.

## Logistique de projet

### Contactez les responsables du SIG Docs

Contactez l'un des présidents du Kubernetes
[SIG Docs](https://github.com/kubernetes/community/tree/master/sig-docs#chairs)
lorsque vous démarrez une nouvelle localisation.

### Mainteneurs

Chaque traduction doit fournir ses propres responsables. Les responsables
peuvent appartenir à une ou plusieurs organisations. Dans la mesure du possible,
les pull requests de traduction doivent être approuvées par un relecteur d'une
organisation différente de celle du traducteur.

Une traduction doit avoir un minimum de deux mainteneurs. (Il n'est pas possible
de relire et d'approuver son propre travail.)

### Gestion des branches

Étant donné que les projets de traduction sont des efforts hautement
collaboratifs, nous encourageons les équipes à travailler à partir d’une branche
de développement partagée.

Pour collaborer sur une branche de développement:

1. A team member opens a development branch, usually by opening a new pull
   request against a source branch on https://github.com/kubernetes/website.

   Nous recommandons le schéma de nommage de branche suivant :

   `dev-<source version>-<language code>.<team milestone>`

   Par exemple, un approbateur d'une équipe de localisation allemande ouvre la
   branche développement `dev-1.12-de.1` directement contre le dépôt k/website,
   basé sur la branche source pour Kubernetes v1.12.

2. Les contributeurs individuels ouvrent des branches de fonctionnalités basées
   sur la branche de développement.

   Par exemple, un contributeur allemand ouvre une pull request avec les
   modifications suivantes `kubernetes:dev-1.12-de.1` sur
   `username:local-branch-name`.

3. Les approbateurs examinent et mergent les branches de fonctionnalités dans la
   branche de développement.

4. Périodiquement, un approbateur fusionne la branche de développement à sa
   branche source.

Répétez les étapes 1 à 4 au besoin jusqu'à ce que la localisation soit terminée.
Par exemple, les branches de développement allemandes suivantes le seraient :
`dev-1.12-de.2`, `dev-1.12-de.3`, etc.

Les équipes doivent fusionner le contenu localisé dans la même branche de
publication d'où provient le contenu. Par exemple, une direction du
développement provenant de {{< release-branch >}} doit se fonder sur
{{< release-branch >}}.

Un approbateur doit maintenir une branche de développement en la tenant à jour
avec sa branche source et en résolvant les conflits entre les branches. Plus une
branche de développement reste ouverte longtemps, plus elle nécessite
généralement de maintenance. Envisagez de merger périodiquement les branches de
développement et d’en ouvrir de nouvelles, plutôt que de conserver une branche
de développement extrêmement ancienne.

Seuls les approbateurs peuvent accepter les pull requests, mais n'importe qui
peut en ouvrir une avec une nouvelle branche de développement. Aucune
autorisation spéciale n'est requise.

Pour plus d'informations sur le travail à partir de forks ou directement �
partir du dépôt, voir ["fork and clone the repo"](#fork-and-clone-the-repo).

### Upstream contributions

SIG Docs souhaite la bienvenue aux
[contributions et corrections upstream](/docs/contribute/intermediate#localize-content)
à la source anglaise.

{{% /capture %}}

{{% capture whatsnext %}}

Une fois qu'une traduction répond aux exigences de logistique et à une
couverture admissible, le SIG docs se chargera des taches suivantes:

- Activer la sélection de la langue sur le site Web
- Publier la disponibilité de la traduction via les canaux de la
  [Cloud Native Computing Foundation](https://www.cncf.io/), y compris sur le
  blog de [Kubernetes](https://kubernetes.io/blog/).

{{% /capture %}}
