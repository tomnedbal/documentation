---
title: Kubernetes
kind: documentation
aliases:
  - /fr/guides/basic_agent_usage/kubernetes
  - /fr/agent/basic_agent_usage/kubernetes
  - /fr/tracing/kubernetes/
  - /fr/tracing/setup/kubernetes
  - /fr/integrations/faq/using-rbac-permission-with-your-kubernetes-integration
  - /fr/integrations/faq/gathering-kubernetes-events
  - /fr/agent/kubernetes/event_collection
  - /fr/agent/kubernetes/daemonset_setup
  - /fr/agent/kubernetes/helm
  - /fr/agent/autodiscovery
further_reading:
  - link: agent/kubernetes/log
    tag: Documentation
    text: Recueillir les logs de votre application
  - link: /agent/kubernetes/apm
    tag: Documentation
    text: Recueillir les traces de votre application
  - link: /agent/kubernetes/prometheus
    tag: Documentation
    text: Recueillir vos métriques Prometheus
  - link: /agent/kubernetes/integrations
    tag: Documentation
    text: Recueillir automatiquement les métriques et les logs de vos applications
  - link: /agent/guide/autodiscovery-management
    tag: Documentation
    text: Limiter la collecte de données à un seul sous-ensemble de conteneurs
  - link: /agent/kubernetes/tag
    tag: Documentation
    text: Attribuer des tags à toutes les données émises par un conteneur
---
Exécutez l'Agent Datadog dans votre cluster Kubernetes en tant que DaemonSet pour commencer à recueillir des métriques, des traces et des logs sur votre cluster et vos applications. Vous pouvez déployer l'Agent à l'aide d'un [chart Helm](?tab=helm) ou directement avec une définition YAML d'objet [DaemonSet](?tab=daemonset).

**Remarque** : les versions 6.0 et ultérieures de l'Agent prennent seulement en charge les versions de Kubernetes ultérieures à 1.7.6. Pour les versions antérieures de Kubernetes, consultez la [section Versions antérieures de Kubernetes][1].

## Installation

{{< tabs >}}
{{% tab "Helm" %}}

Pour installer le chart avec un nom de version `<RELEASE_NAME>` personnalisé (p. ex., `datadog-agent`) :

1. [Installez Helm][1].
2. Téléchargez le [fichier de configuration `values.yaml` Datadog][2].
3. S'il s'agit d'une nouvelle installation, ajoutez le référentiel du Helm Datadog et le référentiel de Helm stable (pour le chart Kube State Metrics) :
    ```bash
    helm repo add datadog https://helm.datadoghq.com
    helm repo add stable https://kubernetes-charts.storage.googleapis.com/
    helm repo update
    ```
4. Récupérez votre clé d'API Datadog à partir des [instructions d'installation de l'Agent][3] et exécutez ce qui suit :

- **Helm v3 et ultérieur**

    ```bash
    helm install <RELEASE_NAME> -f values.yaml  --set datadog.apiKey=<DATADOG_API_KEY> datadog/datadog --set targetSystem=<TARGET_SYSTEM>
    ```

    Remplacez `<TARGET_SYSTEM>` par le nom de votre système d'exploitation : `linux` ou `windows`.

- **Helm v1 ou v2**

    ```bash
    helm install -f values.yaml --name <RELEASE_NAME> --set datadog.apiKey=<DATADOG_API_KEY> datadog/datadog
    ```

Ce chart ajoute l'Agent Datadog à l'ensemble des nœuds dans votre cluster via un DaemonSet. Il peut également déployer le [chart kube-state-metrics][4] et l'utiliser comme source supplémentaire de métriques concernant le cluster. Quelques minutes après l'installation, Datadog commence à transmettre les hosts et les métriques.

Activez ensuite les fonctionnalités Datadog que vous souhaitez utiliser, comme l'[APM][5] ou les [logs][6].

**Remarque** : pour obtenir la liste complète des paramètres disponibles pour le chart Datadog et leurs valeurs par défaut, consultez le [README du référentiel Helm Datadog][7].

### Mise à niveau depuis le chart v1.x

Le chart Datadog a été réusiné dans la v2.0 afin de regrouper plus logiquement les paramètres du fichier `values.yaml`.

Si vous déployez actuellement un chart antérieur à `v2.0.0`, suivez le [guide de migration][8] (en anglais) afin de mapper vos anciens paramètres avec les nouveaux champs.


[1]: https://v3.helm.sh/docs/intro/install/
[2]: https://github.com/DataDog/helm-charts/blob/master/charts/datadog/values.yaml
[3]: https://app.datadoghq.com/account/settings#api
[4]: https://github.com/helm/charts/tree/master/stable/kube-state-metrics
[5]: /fr/agent/kubernetes/apm?tab=helm
[6]: /fr/agent/kubernetes/log?tab=helm
[7]: https://github.com/DataDog/helm-charts/blob/master/charts/datadog
[8]: https://github.com/DataDog/helm-charts/blob/master/charts/datadog/docs/Migration_1.x_to_2.x.md
{{% /tab %}}
{{% tab "DaemonSet" %}}

Tirez profit des DaemonSets pour déployer l'Agent Datadog sur l'ensemble de vos nœuds (ou sur un nœud donné grâce [aux nodeSelectors][1]). 

Pour installer l'Agent Datadog sur votre cluster Kubernetes :

1. **Configurez les autorisations de l'Agent** : si le contrôle d'accès basé sur des rôles (RBAC) est activé pour votre déploiement Kubernetes, configurez les autorisations RBAC pour le compte de service de votre Agent Datadog. Depuis la version 1.6 de Kubernetes, le RBAC est activé par défaut. Créez les ClusterRole, ServiceAccount et ClusterRoleBinding appropriés à l'aide de la commande suivante :

    ```shell
    kubectl apply -f "https://raw.githubusercontent.com/DataDog/datadog-agent/master/Dockerfiles/manifests/rbac/clusterrole.yaml"

    kubectl apply -f "https://raw.githubusercontent.com/DataDog/datadog-agent/master/Dockerfiles/manifests/rbac/serviceaccount.yaml"

    kubectl apply -f "https://raw.githubusercontent.com/DataDog/datadog-agent/master/Dockerfiles/manifests/rbac/clusterrolebinding.yaml"
    ```

    **Remarque** : ces configurations sont par défaut définies pour l'espace de nommage `default`. Si vous utilisez un espace de nommage personnalisé, modifiez le paramètre `namespace` avant d'appliquer les configurations.

2. **Créez un secret qui contient votre clé d'API Datadog**. Remplacez `<DATADOG_API_KEY>` ci-dessous par [la clé d'API de votre organisation][2]. Ce secret est utilisé dans le manifeste pour déployer l'Agent Datadog.

    ```shell
    kubectl create secret generic datadog-agent --from-literal api-key="<DATADOG_API_KEY>" --namespace="default"
    ```

     **Remarque** : cette opération crée un secret dans l'espace de nommage `default`. SI vous utilisez un espace de nommage personnalisé, modifiez le paramètre `namespace` de la commande avant de l'exécuter.

3. **Créez le manifeste de l'Agent Datadog**. Créez le manifeste `datadog-agent.yaml` à partir de l'un des modèles suivants :

    | Métriques | Logs | APM | Processus | NPM | Linux                  | Windows                 |
    |---------|------|-----|---------|-----|------------------------|-------------------------|
    | X       | X    | X   | X       |     | [Modèle du manifeste][3] | [Modèle du manifeste][4] |
    | X       | X    | X   |         |     | [Modèle du manifeste][5] | [Modèle du manifeste][6] |
    | X       | X    |     |         |     | [Modèle du manifeste][7] | [Modèle du manifeste][8] |
    | X       |      | X   |         |     | [Modèle du manifeste][9] | [Modèle du manifeste][10] |
    |         |      |     |         | X   | [Modèle du manifeste][11] | Aucun modèle             |
    | X       |      |     |         |     | [Modèle du manifeste][12] | [Modèle du manifeste][13] |

     Pour activer toutes les fonctionnalités de collecte de traces, [vous devez suivre plusieurs étapes supplémentaires lors de la configuration des pods de votre application][14]. Consultez également les sections relatives aux [logs][15], à l'[APM][16], aux [processus][17] et à la [surveillance des performances réseau][18] pour découvrir comment activer chacune de ces fonctionnalités.

     **Remarque** : ces manifestes sont par défaut définis pour l'espace de nommage `default`. Si vous utilisez un espace de nommage personnalisé, modifiez le paramètre `metadata.namespace` avant d'appliquer les manifestes.

4. **Définissez votre site Datadog** (facultatif). Si vous utilisez le site européen de Datadog, définissez la variable d'environnement `DD_SITE` sur `datadoghq.eu` dans le manifeste `datadog-agent.yaml`.

5. **Déployez le DaemonSet** avec cette commande :

    ```shell
    kubectl apply -f datadog-agent.yaml
    ```

6. **Vérification** : pour vérifier que l'Agent Datadog s'exécute dans votre environnement en tant que DaemonSet, exécutez :

    ```shell
    kubectl get daemonset
    ```

     Si l'Agent est déployé, une sortie similaire au texte ci-dessous s'affiche. Les valeurs `DESIRED` et `CURRENT` correspondent au nombre de nœuds exécutés dans votre cluster.

    ```shell
    NAME            DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
    datadog-agent   2         2         2         2            2           <none>          10s
    ```

7. **Configurez des métriques Kubernetes State** (facultatif) : téléchargez le [dossier de manifestes Kube-State][19], puis appliquez-les à votre cluster Kubernetes pour recueillir automatiquement des [métriques kube-state][20] :

    ```shell
    kubectl apply -f <NAME_OF_THE_KUBE_STATE_MANIFESTS_FOLDER>
    ```

[1]: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector
[2]: https://app.datadoghq.com/account/settings#api
[3]: /resources/yaml/datadog-agent-all-features.yaml
[4]: /resources/yaml/datadog-agent-windows-all-features.yaml
[5]: /resources/yaml/datadog-agent-logs-apm.yaml
[6]: /resources/yaml/datadog-agent-windows-logs-apm.yaml
[7]: /resources/yaml/datadog-agent-logs.yaml
[8]: /resources/yaml/datadog-agent-windows-logs.yaml
[9]: /resources/yaml/datadog-agent-apm.yaml
[10]: /resources/yaml/datadog-agent-windows-apm.yaml
[11]: /resources/yaml/datadog-agent-npm.yaml
[12]: /resources/yaml/datadog-agent-vanilla.yaml
[13]: /resources/yaml/datadog-agent-windows-vanilla.yaml
[14]: /fr/agent/kubernetes/apm/#setup
[15]: /fr/agent/kubernetes/log/
[16]: /fr/agent/kubernetes/apm/
[17]: /fr/infrastructure/process/?tab=kubernetes#installation
[18]: /fr/network_performance_monitoring/installation/
[19]: https://github.com/kubernetes/kube-state-metrics/tree/master/examples/standard
[20]: /fr/agent/kubernetes/data_collected/#kube-state-metrics
{{% /tab %}}
{{% tab "Operator" %}}

<div class="alert alert-warning">L'Operator Datadog est en bêta publique. Si vous souhaitez nous faire part de vos remarques ou de vos questions, contactez l'<a href="/help">assistance Datadog</a>.</div>

[L'Operator Datadog][1] est une fonctionnalité permettant de déployer l'Agent Datadog sur Kubernetes et OpenShift. L'Operator transmet des données sur le statut, la santé et les erreurs du déploiement dans le statut de sa ressource personnalisée. Ses paramètres de niveau supérieur permettent également de réduire les erreurs de configuration.

## Prérequis

L'utilisation de l'Operator Datadog nécessite les prérequis suivants :

- **Cluster Kubernetes version >= v1.14.X** : les tests ont été réalisés sur les versions >= `1.14.0`. Néanmoins, les versions `>= v1.11.0` devraient également fonctionner. Pour les versions plus anciennes, en raison de la prise en charge limitée du CRD, il se peut que l'Operator ne fonctionne pas comme prévu.
- [`Helm`][2] pour le déploiement de `datadog-operator`.
- [Interface de ligne de commande `Kubectl`][3] pour l'installation de `datadog-agent`.


## Déployer un Agent avec l'Operator

Pour déployer un Agent Datadog avec l'Operator en un minimum d'étapes, utilisez le chart Helm [`datadog-agent-with-operator`][4].
Voici les étapes à suivre :

1. [Téléchargez le chart][5] :

   ```shell
   curl -Lo datadog-agent-with-operator.tar.gz https://github.com/DataDog/datadog-operator/releases/latest/download/datadog-agent-with-operator.tar.gz
   ```

2. Créez un fichier avec les spécifications de l'Agent. La configuration la plus simple est la suivante :

   ```yaml
   credentials:
     apiKey: <DATADOG_API_KEY>
     appKey: <DATADOG_APP_KEY>
   agent:
     image:
       name: "datadog/agent:latest"
   ```

   Remplacez `<CLÉ_API_DATADOG>` et `<CLÉ_APPLICATION_DATADOG>` par vos [clés d'API et d'application Datadog][6].

3. Déployez l'Agent Datadog avec le fichier de configuration ci-dessus :
   ```shell
   helm install --set-file agent_spec=/path/to/your/datadog-agent.yaml datadog datadog-agent-with-operator.tar.gz
   ```

## Nettoyage

La commande suivante permet de supprimer toutes les ressources Kubernetes créées par les instructions ci-dessus :

```shell
kubectl delete datadogagent datadog
helm delete datadog
```

Pour en savoir plus sur la configuration de l'Operator, notamment sur l'utilisation de tolérances, consultez le [guide de configuration avancée de l'Operator Datadog][7].

[1]: https://github.com/DataDog/datadog-operator
[2]: https://helm.sh
[3]: https://kubernetes.io/docs/tasks/tools/install-kubectl/
[4]: https://github.com/DataDog/datadog-operator/tree/master/chart/datadog-agent-with-operator
[5]: https://github.com/DataDog/datadog-operator/releases/latest/download/datadog-agent-with-operator.tar.gz
[6]: https://app.datadoghq.com/account/settings#api
[7]: /fr/agent/guide/operator-advanced
{{% /tab %}}
{{< /tabs >}}

### Sans privilèges

(Facultatif) Pour exécuter une installation sans privilèges, ajoutez le bloc suivant à votre [modèle de pod][2] :

```text
  spec:
    securityContext:
      runAsUser: <ID_UTILISATEUR>
      fsGroup: <ID_GROUPE_DOCKER>
```

## Collecte d'événements

{{< tabs >}}
{{% tab "Helm" %}}

Définissez les options `datadog.leaderElection`, `datadog.collectEvents` et `agents.rbac.create` sur `true` dans votre fichier `value.yaml` afin d'activer la collecte d'événements Kubernetes.

{{% /tab %}}
{{% tab "DaemonSet" %}}

Si vous souhaitez recueillir des événements à partir de votre cluster Kubernetes, définissez les variables d'environnement `DD_COLLECT_KUBERNETES_EVENTS` et `DD_LEADER_ELECTION` sur `true` dans le manifeste de votre Agent. Vous pouvez également utiliser le [processus de collecte d'événements de l'Agent de cluster Datadog][1].

[1]: /fr/agent/cluster_agent/event_collection/
{{% /tab %}}
{{% tab "Operator" %}}

Définissez `agent.config.collectEvents` sur `true` dans votre manifeste `datadog-agent.yaml`.

Par exemple :

```
agent:
  config:
    collectEvents: true
```

{{% /tab %}}
{{< /tabs >}}


## Intégrations

Dès lors que votre Agent s'exécute dans votre cluster, vous pouvez utiliser la [fonctionnalité Autodiscovery de Datadog][3] pour recueillir automatiquement des métriques et des logs à partir de vos pods.

## Variables d'environnement

Vous trouverez ci-dessous la liste des variables d'environnement disponibles pour l'Agent Datadog. Si vous souhaitez configurer ces variables avec Helm, consultez la liste complète des options de configuration pour le fichier `datadog-value.yaml` dans le [référentiel helm/charts GitHub][4].

### Options globales

| Variable d'environnement       | Description                                                                                                                                                                                                                                                                                                                                      |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `DD_API_KEY`       | Votre clé d'API Datadog (**obligatoire**).                                                                                                                                                                                                                                                                                                              |
| `DD_ENV`          | Définit le tag `env` global pour toutes les données émises.                                                                                                                                                                                                                                                                 |
| `DD_HOSTNAME`      | Le hostname à utiliser pour les métriques (si la détection automatique échoue).                                                                                                                                                                                                                                                                                             |
| `DD_TAGS`          | Tags de host séparés par des espaces. Exemple : `tag-simple-0 clé-tag-1:valeur-tag-1`.                                                                                                                                                                                                                                                                 |
| `DD_SITE`          | Le site auquel vous transmettez vos métriques, traces et logs. Vous pouvez choisir entre `datadoghq.com` pour le site américain de Datadog et `datadoghq.eu` pour le site européen.                                                                                                                                                                                      |
| `DD_DD_URL`        | Paramètre facultatif pour remplacer l'URL utilisée pour l'envoi de métriques.                                                                                                                                                                                                                                                                                      |
| `DD_CHECK_RUNNERS` | Par défaut, l'Agent exécute tous les checks simultanément (valeur par défaut : `4` runners). Pour exécuter les checks de manière séquentielle, définissez la valeur sur `1`. Si vous devez exécuter un grand nombre de checks (ou plusieurs checks lents), le composant `collector-queue` peut prendre du retard et renvoyer un échec au check de santé. Vous pouvez accroître le nombre de runners pour exécuter davantage de checks en parallèle. |
| `DD_LEADER_ELECTION` | Si vous exécutez plusieurs Agents dans votre cluster, définissez cette variable sur `true` pour éviter de recueillir deux fois chaque événement. |

### Paramètres de proxy

À partir de l'Agent v6.4.0 (et v6.5.0 pour l'Agent de trace), vous pouvez remplacer les paramètres de proxy de l'Agent via les variables d'environnement suivantes :

| Variable d'environnement        | Description                                                       |
| ------------------- | ----------------------------------------------------------------- |
| `DD_PROXY_HTTP`     | URL HTTP à utiliser comme proxy pour les requêtes `http`.                |
| `DD_PROXY_HTTPS`    | URL HTTPS à utiliser comme proxy pour les requêtes `https`.              |
| `DD_PROXY_NO_PROXY` | Une liste d'URL, séparées par des espaces, pour lesquelles aucun proxy ne doit être utilisé. |
| `DD_SKIP_SSL_VALIDATION` | Option de test si l'Agent a des difficultés à se connecter à Datadog. |

Pour en savoir plus sur les paramètres de proxy, consultez la [documentation relative au proxy de l'Agent v6][5].

### Agents de collecte facultatifs

Par défaut, les Agents de collecte facultatifs sont désactivés pour des raisons de sécurité et de performance. Utilisez les variables d'environnement suivantes pour les activer :

| Variable d'environnement               | Description                                                                                                                                                                                                                                                  |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `DD_APM_ENABLED`           | Active la [collecte de traces][6] via l'Agent de trace.                                                                                                                                                                                                           |
| `DD_LOGS_ENABLED`          | Active la [collecte de logs][7] via l'Agent de log.                                                                                                                                                                                                              |
| `DD_PROCESS_AGENT_ENABLED` | Active la [collecte de live processes][8] via l'Agent de processus. Par défaut, la [vue Live Container][9] est déjà activée si le socket Docker est disponible. Si définie sur `false`, la [collecte de live processes][8] et la [vue Live Container][9] sont désactivées. |
| `DD_COLLECT_KUBERNETES_EVENTS ` | Active la collecte d'événements via l'Agent. Si vous exécutez plusieurs Agents dans votre cluster, définissez également `DD_LEADER_ELECTION` sur `true`. |

Pour activer la vue Live Container, assurez-vous d'exécuter l'Agent de processus en ayant défini DD_PROCESS_AGENT_ENABLED sur `true`.

### DogStatsD (métriques custom)

Envoyez des métriques custom avec le [protocole StatsD][10] :

| Variable d'environnement                     | Description                                                                                                                                                |
| -------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `DD_DOGSTATSD_NON_LOCAL_TRAFFIC` | Effectue une écoute des paquets DogStatsD issus d'autres conteneurs (requis pour envoyer des métriques custom).                                                                       |
| `DD_HISTOGRAM_PERCENTILES`       | Les centiles de l'histogramme à calculer (séparés par des espaces). Valeur par défaut :  `0.95`.                                                                         |
| `DD_HISTOGRAM_AGGREGATES`        | Les agrégations à calculer pour l'histogramme (séparées par des espaces). Valeur par défaut : « max median avg count ».                                                          |
| `DD_DOGSTATSD_SOCKET`            | Le chemin vers le socket Unix à écouter. Doit être dans le volume `rw` monté.                                                                                    |
| `DD_DOGSTATSD_ORIGIN_DETECTION`  | Active la détection de conteneurs et l'ajout de tags pour les métriques de socket Unix.                                                                                            |
| `DD_DOGSTATSD_TAGS`              | Les tags supplémentaires à ajouter à l'ensemble des métriques, événements et checks de service reçus par ce serveur DogStatsD. Par exemple : `["env:golden", "group:retrievers"]`. |

En savoir plus sur l'utilisation de [DogStatsD sur des sockets de domaine Unix][11].

### Tagging

Datadog recueille automatiquement les tags courants à partir de Kubernetes. Pour extraire des tags supplémentaires, utilisez les options suivantes :

| Variable d'environnement                            | Description             |
| --------------------------------------- | ----------------------- |
| `DD_KUBERNETES_POD_LABELS_AS_TAGS`      | Extrait les étiquettes de pod.      |
| `DD_KUBERNETES_POD_ANNOTATIONS_AS_TAGS` | Extrait les annotations de pod. |

Consultez la documentation relative à [l'extraction de tags Kubernetes][12] pour en savoir plus.

### Utiliser des secrets

Les identifiants des intégrations peuvent être conservés dans des secrets Docker ou Kubernetes et utilisés dans les modèles Autodiscovery. Pour en savoir plus, consultez la [documentation sur la gestion des secrets][13].

### Ignorer des conteneurs

Excluez les conteneurs de la collecte de logs, de la collecte de métriques et d'Autodiscovery. Par défaut, Datadog exclut les conteneurs `pause` de Kubernetes et d'OpenShift. Ces listes d'inclusion et d'exclusion s'appliquent uniquement à Autodiscovery. Elles n'ont aucun impact sur les traces ni sur DogStatsD. La valeur de ces variables d'environnement prend en charge les expressions régulières.

| Variable d'environnement    | Description                                                                                                                                                                                                        |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `DD_CONTAINER_INCLUDE` | Liste des conteneurs à inclure (séparés par des espaces). Utilisez `.*` pour tous les inclure. Exemple : `"image:nom_image_1 image:nom_image_2"`, `image:.*`.  |
| `DD_CONTAINER_EXCLUDE` | Liste des conteneurs à exclure (séparés par des espaces). Utilisez `.*` pour tous les exclure. Exemple : `"image:nom_image_3 image:nom_image_4"` (cette variable est seulement traitée pour Autodiscovery), `image:.*`. |
| `DD_CONTAINER_INCLUDE_METRICS` | Liste des conteneurs dont vous souhaitez inclure les métriques.  |
| `DD_CONTAINER_EXCLUDE_METRICS` | Liste des conteneurs dont vous souhaitez exclure les métriques. |
| `DD_CONTAINER_INCLUDE_LOGS` | Liste des conteneurs dont vous souhaitez inclure les logs.  |
| `DD_CONTAINER_EXCLUDE_LOGS` | Liste des conteneurs dont vous souhaitez exclure les logs. |
| `DD_AC_INCLUDE` | **Obsolète**. Liste des conteneurs à inclure (séparés par des espaces). Utilisez `.*` pour tous les inclure. Exemple : `"image:nom_image_1 image:nom_image_2"`, `image:.*`.  |
| `DD_AC_EXCLUDE` | **Obsolète**. Liste des conteneurs à exclure (séparés par des espaces). Utilisez `.*` pour tous les exclure. Exemple : `"image:nom_image_3 image:nom_image_4"` (cette variable est seulement traitée pour Autodiscovery), `image:.*`. |

Des exemples supplémentaires sont disponibles sur la page [Gestion de la découverte de conteneurs][14].

**Remarque** : ces paramètres n'ont aucun effet sur les métriques `docker.containers.running`, `.stopped`, `.running.total` et `.stopped.total`, qui prennent en compte l'ensemble des conteneurs. Cela n'a aucune incidence sur le nombre de conteneurs facturés.

### Divers

| Variable d'environnement                        | Description                                                                                                      |
| ----------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `DD_PROCESS_AGENT_CONTAINER_SOURCE` | Remplace la détection automatique des sources de conteneurs par une source unique, comme `"docker"`, `"ecs_fargate"` ou `"kubelet"`. |
| `DD_HEALTH_PORT`                    | Définissez cette variable sur `5555` pour exposer le check de santé de l'Agent sur le port `5555`.                                              |

Vous pouvez ajouter d'autres écouteurs et fournisseurs de configuration à l'aide des variables d'environnement `DD_EXTRA_LISTENERS` et `DD_EXTRA_CONFIG_PROVIDERS`. Elles viennent s'ajouter aux variables définies dans les sections `listeners` et `config_providers` du fichier de configuration `datadog.yaml`.

## Commandes

Consultez les [guides sur les commandes de l'Agent][15] pour découvrir toutes les commandes de l'Agent Docker.

## Pour aller plus loin

{{< partial name="whats-next/whats-next.html" >}}

[1]: /fr/agent/faq/kubernetes-legacy/
[2]: https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/#pod-templates
[3]: /fr/agent/kubernetes/integrations/
[4]: https://github.com/DataDog/helm-charts/tree/master/charts/datadog#all-configuration-options
[5]: /fr/agent/proxy/#agent-v6
[6]: /fr/agent/kubernetes/apm/
[7]: /fr/agent/kubernetes/log/
[8]: /fr/infrastructure/process/
[9]: /fr/infrastructure/livecontainers/
[10]: /fr/developers/dogstatsd/
[11]: /fr/developers/dogstatsd/unix_socket/
[12]: /fr/agent/kubernetes/tag/
[13]: /fr/security/agent/#secrets-management
[14]: /fr/agent/guide/autodiscovery-management/
[15]: /fr/agent/guide/agent-commands/