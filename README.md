# Mansa — Documentation officielle

Ce dépôt contient le cahier des charges, l’architecture, les règles métier, les critères de recette et les instructions de développement de la plateforme fintech Mansa. Le code source correspondant est conservé dans `zoumana007/mansa-platform`.

## Organisation

- `volume-01-socle-technique/` : vision, architecture, monorepo, backend, données, sécurité, décisions d’architecture et CI/CD.
- `volume-02-application-client/` : application mobile client.
- `volume-03-application-commercant/` : application commerçant.
- `volume-04-application-tpe/` : application TPE Android.
- `volume-05-administration/` : portail administrateur et gouvernance.
- `volume-06-module-etat/` : amendes, taxes, bourses, cartes étudiantes et services publics.
- `volume-07-intelligence-artificielle/` : assistant Jini, fraude, support et recommandations.
- `volume-08-donnees-analytics/` : reporting, comptabilité, BI et exports.
- `volume-09-infrastructure-production/` : déploiement, observabilité, sauvegardes et haute disponibilité.
- `volume-10-tests-documentation-roadmap/` : stratégie de tests, documentation finale et feuille de route.

## Principes directeurs

1. Toutes les fonctions sensibles sont configurables depuis l’administration.
2. Toute action critique est tracée dans un journal d’audit immuable.
3. La plateforme est conçue pour plusieurs pays, devises, langues et partenaires.
4. Les intégrations externes passent par des adaptateurs isolés.
5. Aucun secret ni identifiant de production n’est stocké dans le dépôt.
6. Le développement progresse module par module, avec validation technique avant intégration.
7. Chaque exigence doit être testable ou accompagnée d’un critère d’acceptation.
8. Les chemins, scripts et noms de modules documentés doivent correspondre au dépôt plateforme.

## Ordre de construction

Le travail suit les dépendances réelles : socle technique, identité et conformité, comptes et grand livre, paiements, applications, administration, services publics, données, exploitation puis recette de production.

## Règles de contenu

Aucun secret, identifiant réel, document KYC réel ou donnée personnelle ne doit être ajouté. Les hypothèses juridiques et réglementaires restent à valider avec les banques, opérateurs, autorités et conseils compétents avant toute mise en production.

## Statut

Les volumes 1 à 10 disposent désormais d’une première spécification structurée sur leurs domaines principaux. Le socle inclut une première série de décisions d’architecture alignées avec le monorepo. La documentation doit encore être enrichie avec les critères de recette détaillés, matrices de permissions, catalogues d’API, runbooks spécialisés et décisions complémentaires.

Le dépôt plateforme contient le socle du monorepo et plusieurs contrats métier partagés. Les applications exécutables, le grand livre persistant, les modules NestJS complets, les interfaces mobiles et web, les adaptateurs partenaires ainsi que les validations de production restent à construire progressivement.
