# Moteur de politiques d’autorisation

## Objectif

Le moteur de politiques complète le RBAC par des règles contextuelles de type ABAC. Il produit une décision explicite, traçable et reproductible pour toute opération sensible sans disperser les règles d’accès dans les contrôleurs ou les interfaces.

## Contrats partagés

Le dépôt `mansa-platform` expose les contrats dans `packages/contracts/src/policy.ts` et le sous-chemin `@mansa/contracts/policy`.

Les éléments principaux sont :

- `AuthorizationPolicy` : définition versionnée d’une politique ;
- `PolicyCondition` : condition portant sur un attribut de l’acteur, de la ressource ou du contexte ;
- `EvaluateAuthorizationCommand` : demande d’évaluation ;
- `AuthorizationEvaluationResult` : décision et trace d’évaluation ;
- `PolicyEvaluationTrace` : résultat détaillé pour chaque politique examinée.

## Règles de décision

1. Une politique `DENY` correspondante est prioritaire sur une politique `ALLOW`.
2. L’absence de politique autorisant explicitement l’action produit un refus par défaut.
3. Les politiques sont évaluées par priorité décroissante, puis par identifiant stable.
4. Une politique inactive, archivée ou en brouillon ne participe jamais à une décision de production.
5. Le niveau d’authentification requis doit être atteint avant l’évaluation des conditions métier.
6. Les obligations retournées doivent être exécutées avant l’action : journalisation renforcée, approbation, authentification forte ou limitation de montant.
7. La version du catalogue évalué est enregistrée avec la décision afin de permettre sa reproduction lors d’un audit.

## Attributs autorisés

Le moteur peut lire uniquement une liste blanche d’attributs :

- acteur : type, rôles, permissions, organisation, commerçant, point de vente et pays ;
- ressource : type, propriétaire, organisation, commerçant, point de vente, pays et environnement ;
- contexte : niveau d’authentification, montant en unité mineure, devise, heure, canal et identifiant de corrélation.

Une politique ne doit jamais exécuter de code arbitraire ni effectuer directement un appel réseau.

## Cycle de vie

- `DRAFT` : préparation et tests sans effet ;
- `ACTIVE` : utilisable dans les environnements ciblés ;
- `DISABLED` : désactivation réversible ;
- `ARCHIVED` : conservation historique non réactivable sans nouvelle version.

Toute activation ou désactivation en production est auditée. Les politiques à risque élevé nécessitent une double approbation selon les règles d’administration.

## Cache et cohérence

Les politiques actives peuvent être mises en cache avec une durée courte. Le cache est indexé par version de catalogue. Une activation invalide immédiatement la version précédente. En cas d’indisponibilité du moteur ou de catalogue invalide, le comportement par défaut est le refus pour les actions sensibles.

## Journalisation

Chaque décision conserve au minimum :

- l’identifiant de corrélation ;
- l’acteur et l’action ;
- le type et l’identifiant de ressource ;
- la décision, le code motif et les obligations ;
- la version du catalogue ;
- les politiques évaluées ;
- l’horodatage et l’environnement.

Les journaux ne doivent pas contenir de secret, de mot de passe, de jeton complet ou de document KYC brut.

## Critères d’acceptation

- Les catalogues d’effets, statuts et opérateurs sont typés et testés.
- Une décision identique est obtenue pour un même contexte et une même version de politiques.
- Une règle de refus prioritaire ne peut pas être contournée par une règle d’autorisation moins prioritaire.
- Une politique désactivée cesse de produire des effets après invalidation du cache.
- Toutes les décisions sensibles sont liées à un événement d’audit.
- Les contrats compilent sans dépendance applicative et sont importables depuis le sous-chemin documenté.
