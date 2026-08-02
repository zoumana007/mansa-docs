# Résolution des permissions effectives

## Objet

La résolution des permissions effectives transforme les affectations de rôles actives d’un acteur en droits réellement exploitables par les services. Elle constitue l’étape préalable à l’évaluation des politiques d’autorisation.

## Règles de calcul

Une affectation contribue aux permissions uniquement lorsque :

- son `actorId` correspond à l’acteur authentifié ;
- son statut est `ACTIVE` ;
- sa date de début est atteinte ;
- sa date de fin éventuelle n’est pas dépassée ;
- le rôle appartient au catalogue des rôles de référence ;
- le type d’acteur est autorisé par le profil du rôle ;
- le type de périmètre est autorisé par le profil du rôle.

Les affectations suspendues, révoquées, expirées, futures ou incompatibles sont ignorées. Cette exclusion doit être traçable dans les composants d’autorisation sans exposer d’informations sensibles au client.

## Périmètres

Les permissions restent liées au périmètre de l’affectation : plateforme, pays, organisation, commerce, point de vente ou organisme public. Une permission accordée au niveau `PLATFORM` couvre les périmètres inférieurs. Une permission limitée à un pays, commerce ou emplacement ne doit pas être réutilisée pour une autre ressource.

## Déduplication et provenance

La liste finale des permissions est dédupliquée, mais chaque droit conserve ses preuves d’origine : rôle, identifiant d’affectation et périmètre. Cette provenance permet l’audit, l’explication des décisions et la révocation ciblée.

## Séparation des responsabilités

Le calcul des permissions effectives ne suffit pas à autoriser une action sensible. Le moteur doit encore vérifier le niveau d’authentification, les politiques de refus, les limites, l’état de la ressource, l’environnement et les obligations de double approbation ou de revue.

## Alignement avec le code

Le contrat est implémenté dans `packages/contracts/src/effective-permissions.ts` du dépôt `mansa-platform`. Il est exposé par `@mansa/contracts/effective-permissions` et par l’export principal du paquet.

Les fonctions de référence sont :

- `resolveEffectivePermissions` pour produire les droits et leurs provenances ;
- `hasEffectivePermission` pour contrôler un droit dans un périmètre donné.

## Critères d’acceptation

- Une affectation expirée ou future n’accorde aucun droit.
- Une affectation d’un autre acteur n’accorde aucun droit.
- Un rôle incompatible avec le type d’acteur ou le périmètre est rejeté.
- Les permissions identiques sont dédupliquées dans la vue synthétique.
- La provenance de chaque permission reste disponible.
- Une portée pays ne couvre pas un autre pays.
- Une portée plateforme couvre les ressources inférieures.
- Les tests couvrent au minimum la validité temporelle, l’identité de l’acteur et l’isolation des périmètres.
