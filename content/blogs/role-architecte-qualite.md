+++
title = "Rôle de l'Architecte Qualité (FR)"
LastModifierDisplayName = "Alain Bouchard"
LastModifierEmail = "abouchard@live.ca"
disableToc = "false"
+++

{{< toc >}}

## Role

L'architecte est responsable de deux grands types d'activités à haut niveau:

1. Stratégique (fondations): s'occuper de la vision technique à moyen à long terme (un, deux ans ou même plus) ;
1. Tactique (opérations): support technique des équipes, résolution de problèmes, preuve de concept, etc ;

## Stratégique

1. Définir la “Vision Qualité”, les guides (ou "guidelines"), documents techniques ([ADR], Technical Design, Terminologie/glossaire technique, etc.) tout en tenant compte des règles de sécuritées, [PII], etc. ;
1. Évaluer les technologies : Recherches, comparatifs, preuves de concepts, publications, etc.;
1. Promouvoir les bonnes pratiques pour les technologies choisies;
1. Mentorat et “Coaching”;
1. [Promouvoir la qualité] lors des six étapes du développement logiciel, [SDLC] ou "Software Development Life Cycle" : "Planning" -> "Defining" -> "Designing" -> "Building" -> "Testing" -> "Deployment" ;
1. Assister ou diriger des groupes de travail (ou "Work Groups") pour des sujets précis, définir le but et les livrables, événements périodiques et sur plusieurs mois ;
1. Assister ou diriger des ateliers de travail (ou “Workshops”) pour des sujets précis, définir le but et les livrables, événement sur quelques heures ou quelques jours ;
1. Audit de la couverture de tests et corrélation des données avec les services problématiques en production (décision basée sur les données ou “Data-Driven”);
1. Analyser la stratégie de tests actuelle, caractérisation des tests, définir un plan, promouvoir les bonnes pratiques de tests, la pyramide de tests "pratique" (en anglais pour faciliter) :
    1. [Unit] ("solitary" et "sociable") testing ;
    1. [Component] ("in-process" et "out-of-process") testing ;
    1. [Contract] testing ;
    1. [Integration] ("narrow" et "broad") testing ;
    1. UI End-to-end ([E2E]) et API E2E (ou [Subcutaneous]) ;
    1. [User journey] tests (and [Synthetic Traffic] tests) ;
    1. [Non functional] tests : load, scalability, webvital metrics [LCP], chaos/robustness, etc. ;
1. Création de projets archétypes et création de matériel de formation (on-demand training) suivi de projets pilotes ou réels, écriture de tests, documentation, etc. ;
1. Alignment sur une stratégie de tests avec la procédure de déploiement ([CI]/[CD]) avec une procédure de “[branching]” (examples : gitflow ou trunk-based workflows), [blue/green deployment], smoke tests, etc. ;
1. Participe au processus de recrutement pour le volet “qualité” ;

## Tactique

1. Appliquer la stratégie de tests : développer des projets tests, ajouter les projets au processus (build, [CI]/[CD], [cluster/kubernetes]. etc), documenter, etc. ;
1. Supporter le développement de tests: supporter les QAs et développeurs qui ajoutent des tests, PR reviews, etc. ;
1. Ajouter une couverture de tests : javascript (exemple : cypress.io), python/pytest, java/rest-assured, etc. ;
1. Ajout de “[tests de trafic synthétique][Synthetic Traffic]” et vérification des flux et scénarios clients (ou “customer flows”) ;
1. Analyser les performances systèmes (example: load tests) et selon les standards définis (example: *"Given the data load X and the number of users Y, an API should respond within Z seconds"*), [LCP] (ou “Largest Contentful Paint”, example : *"une page devrait être considérée utilisable en moins de 2.5 secondes"*) ;
1. Utilisation d’outils d’observation pour comprendre les problèmes : [datadoghq], [splunk/signalfx], [prometheus], [grafana], etc. ;
1. Collabore horizontalement avec les différents “domaines” de l’entreprises et des fournisseurs pour améliorer la qualité globale de l’entreprise ;

[ADR]: https://adr.github.io
[blue/green deployment]: https://martinfowler.com/bliki/BlueGreenDeployment.html
[branching]: https://medium.com/@vafrcor2009/gitflow-vs-trunk-based-development-3beff578030b
[CD]: https://www.atlassian.com/continuous-delivery/continuous-deployment
[CI]: https://www.atlassian.com/continuous-delivery/continuous-integration
[cluster/kubernetes]: https://www.vmware.com/topics/glossary/content/kubernetes-cluster.html
[component]: https://martinfowler.com/articles/microservice-testing/#testing-component-introduction
[contract]: https://martinfowler.com/bliki/ContractTest.html
[datadoghq]: https://www.datadoghq.com/
[E2E]: https://martinfowler.com/articles/microservice-testing/#testing-end-to-end-introduction
[grafana]: https://grafana.com/
[integration]: https://martinfowler.com/articles/microservice-testing/#testing-integration-introduction
[LCP]: https://web.dev/lcp
[non functional]: https://www.guru99.com/non-functional-testing.html
[PII]: https://en.wikipedia.org/wiki/Personal_data
[prometheus]: https://prometheus.io/
[Promouvoir la qualité]: https://github.com/AlainBouchard/engineering-notes/blob/master/how-to-promote-the-quality-best-practices.md
[SDLC]: https://www.tutorialspoint.com/sdlc/sdlc_overview.htm
[splunk/signalfx]: https://www.splunk.com/
[Synthetic Traffic]: https://www.datadoghq.com/knowledge-center/synthetic-testing
[subcutaneous]: https://martinfowler.com/bliki/SubcutaneousTest.html
[unit]: https://martinfowler.com/articles/microservice-testing/#testing-unit-introduction
[user journey]: https://martinfowler.com/bliki/UserJourneyTest.html
