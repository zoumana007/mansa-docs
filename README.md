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
- `volume-10-tests-documentation-roadmap/` : stratégie de tests, matrice de recette transverse, documentation finale et feuille de route.

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

## Référence de recette

La matrice commune se trouve dans `volume-10-tests-documentation-roadmap/02-matrice-de-recette-transverse.md`. Elle définit les scénarios minimaux et les conditions de passage vers Recette et Production pour l’authentification, les autorisations, le ledger, les paiements, le KYC, les applications, la sécurité, la reprise et l’observabilité.

## Notifications et assistance

Le socle partagé définit désormais les contrats de notification multicanale et de support client. Les règles suivantes s’appliquent à toutes les applications :

- chaque envoi utilise une clé d’idempotence et peut produire plusieurs livraisons, une par canal ;
- les canaux autorisés sont `IN_APP`, `PUSH`, `SMS`, `EMAIL` et `WHATSAPP` ;
- les coordonnées exposées dans les journaux et réponses sont masquées ;
- les statuts, références fournisseur, dates d’envoi et erreurs restent corrélables sans stocker de secret ;
- une livraison planifiée ne peut être annulée que tant que le fournisseur ne l’a pas rendue irréversible ;
- les tickets de support sont filtrables par demandeur, agent, catégorie, priorité, statut et période ;
- les pièces jointes sont référencées par identifiant et doivent être contrôlées par un service de stockage sécurisé ;
- toute modification administrative d’un ticket ou d’une livraison est auditée ;
- les messages automatiques ne doivent jamais révéler un code OTP, un numéro de carte complet, un secret ou un document KYC.

Dans le dépôt plateforme, les modèles métier sont maintenus dans `packages/contracts/src/notification.ts` et `packages/contracts/src/support.ts`. Les routes, méthodes HTTP et contrats de transport sont séparés dans `packages/contracts/src/notification-api.ts` et `packages/contracts/src/support-api.ts`. Ils couvrent notamment les routes `/v1/notifications`, `/v1/notifications/deliveries` et `/v1/support/tickets`, ainsi que les opérations de consultation, annulation, relance, mise à jour et ajout de message.

## Règles de contenu

Aucun secret, identifiant réel, document KYC réel ou donnée personnelle ne doit être ajouté. Les hypothèses juridiques et réglementaires restent à valider avec les banques, opérateurs, autorités et conseils compétents avant toute mise en production.

## Statut

Les volumes 1 à 10 disposent désormais d’une première spécification structurée sur leurs domaines principaux. Le socle inclut une première série de décisions d’architecture alignées avec le monorepo, une matrice transverse de recette, une matrice RBAC et les contrats initiaux de notifications et de support. La documentation doit encore être enrichie avec les catalogues d’API complets, runbooks spécialisés, parcours détaillés des applications et décisions complémentaires.

Le dépôt plateforme contient le socle du monorepo et plusieurs contrats métier partagés. Les applications exécutables, le grand livre persistant, les modules NestJS complets, les interfaces mobiles et web, les adaptateurs partenaires ainsi que les validations de production restent à construire progressivement.
