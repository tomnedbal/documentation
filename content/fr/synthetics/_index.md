---
title: Surveillance Synthetic
kind: documentation
description: Utilisez des tests automatisés pour vous assurer que les aspects les plus importants de vos systèmes et applications fonctionnent correctement à différents endroits du monde.
disable_toc: true
aliases:
  - /fr/integrations/synthetics/
further_reading:
  - link: 'https://www.datadoghq.com/blog/introducing-synthetic-monitoring/'
    tag: Blog
    text: Présentation de la surveillance Datadog Synthetic
  - link: 'https://www.datadoghq.com/blog/browser-tests/'
    tag: Blog
    text: Surveillance de l'expérience utilisateur avec les tests Browser de Datadog
  - link: 'https://www.datadoghq.com/blog/test-creation-best-practices/'
    tag: Blog
    text: Pratiques recommandées pour la création de tests de bout en bout
  - link: /synthetics/guide/
    tag: Documentation
    text: Guides d'utilisation de la surveillance Synthetic
  - link: /synthetics/troubleshooting/
    tag: Documentation
    text: Dépannage de la surveillance Synthetics
---
{{< img src="synthetics/synthetics_home.png" alt="Page d'accueil Surveillance Synthetic" >}}

Les tests Synthetic vous permettent d'observer le fonctionnement de vos systèmes et applications à l'aide de **requêtes et actions simulées dans le monde entier**. Datadog **surveille le fonctionnement de vos pages Web et de vos API** du backend au frontend, et à différents niveaux du réseau (HTTP, TCP, SSL et DNS) de manière contrôlée et stable, en vous envoyant des alertes en cas de comportement anormal (régression, fonctionnalité défaillante, temps de réponse élevé, code de statut inattendu, etc.). **Les tests de bout en bout dans les environnements de production et d'intégration continue** améliorent la rapidité des équipes de développement, car ils éliminent tout risque que le code défectueux soit mis en production. **La configuration de SLO** sur les principaux endpoints et parcours utilisateurs permet de mieux répondre aux objectifs de performance de l'application et, à terme, de proposer une expérience client robuste.

## Prise en main
Créez votre premier test Synthetic et commencez à surveiller des applications Web pour améliorer leurs performances en quelques minutes seulement.

### Configurer des tests API

Les tests API vous permettent de lancer des requêtes et d'effectuer des vérifications sur vos applications Web à différents niveaux du réseau : HTTP, TCP, SSL et DNS. Créez votre premier [test HTTP][1], [test TCP][2], [test SSL][3] et [test DNS][4] pour commencer à surveiller vos API et votre réseau.

{{< img src="synthetics/api_test.png" alt="Tests API"  style="width:100%;">}}

### Enregistrer des tests Browser

Enregistrez des tests de bout en bout pour surveiller l'expérience de vos clients sur vos pages Web dans le monde entier en utilisant des [tests Browser Synthetic][5].

{{< img src="synthetics/browser_test.gif" alt="Tests Browser"  style="width:100%;">}}

### Lancer des emplacements privés

Utilisez des [emplacements privés Synthetic][6] pour surveiller des API et sites Web internes ou pour créer des emplacements personnalisés dans des zones stratégiques pour votre entreprise.

{{< img src="synthetics/private_locations.png" alt="Emplacements privés"  style="width:100%;">}}

### Exécuter des tests au sein de vos processus d'intégration et de déploiement

Utilisez vos tests Synthetic comme [tests Canary][7] ou exécutez-les directement dans vos [pipelines d'intégration continue][7] pour déployer votre code sans crainte de dégrader l'expérience de vos utilisateurs.

 {{< img src="synthetics/ci.png" alt="Tests d'intégration continue"  style="width:100%;">}}

### Associer vos données de surveillance Synthetic à vos traces

Datadog vous permet d'[intégrer vos tests Synthetic à vos traces d'APM][8] pour identifier l'origine des échecs parmi vos requêtes frontend, réseau et backend.

{{< img src="synthetics/synthetics_traces.gif" alt="Surveillance Synthetic" style="width:100%;">}}

## Pour aller plus loin

{{< partial name="whats-next/whats-next.html" >}}

[1]: /fr/getting_started/synthetics/api_test
[2]: /fr/synthetics/api_tests/?tab=tcptest
[3]: /fr/synthetics/api_tests/?tab=ssltest
[4]: /fr/synthetics/api_tests/?tab=dnstest
[5]: /fr/getting_started/synthetics/browser_test
[6]: /fr/getting_started/synthetics/private_location
[7]: /fr/synthetics/ci/
[8]: /fr/synthetics/apm/