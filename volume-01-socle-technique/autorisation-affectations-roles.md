# Affectations de rôles et portées d’autorisation

## Objectif

Les rôles définissent un ensemble stable de permissions. Les affectations relient temporairement ou durablement un acteur à un rôle dans une portée précise. Cette séparation évite de dupliquer les permissions sur chaque utilisateur et permet de révoquer un accès sans modifier le catalogue de rôles.

## Contrats partagés

Le dépôt `mansa-platform` expose les contrats dans `packages/contracts/src/role-assignment.ts` et le sous-chemin `@mansa/contracts/role-assignment`.

Les contrats principaux sont :

- `RoleDefinition` : rôle nommé et son catalogue de permissions ;
- `RoleScope` : portée dans laquelle le rôle produit ses effets ;
- `RoleAssignment` : affectation historisée d’un rôle à un acteur ;
- `AssignRoleCommand` : demande contrôlée d’affectation ;
- `RevokeRoleAssignmentCommand` : révocation motivée.

## Portées supportées

- `PLATFORM` : ensemble de la plateforme, réservé aux rôles centraux ;
- `COUNTRY` : toutes les ressources d’un pays ;
- `ORGANIZATION` : organisation partenaire ou interne ;
- `MERCHANT` : commerçant déterminé ;
- `LOCATION` : point de vente déterminé ;
- `PUBLIC_ORGANIZATION` : administration, université ou collectivité déterminée.

Une portée autre que `PLATFORM` doit fournir un identifiant de ressource. Une portée pays doit fournir un code pays normalisé. Une affectation ne doit jamais donner accès à une ressource située hors de sa portée.

## Cycle de vie

- `PENDING` : affectation créée mais pas encore effective ;
- `ACTIVE` : affectation utilisable ;
- `SUSPENDED` : suspension réversible ;
- `EXPIRED` : date de fin dépassée ;
- `REVOKED` : révocation définitive et historisée.

Le statut effectif doit aussi tenir compte de `validFrom` et `validUntil`. Une affectation active dont la période n’est pas valide ne produit aucun droit.

## Règles de sécurité

1. Une affectation ne peut être créée que par un acteur autorisé à attribuer le rôle concerné.
2. Un acteur ne peut pas déléguer un rôle plus privilégié que son propre niveau d’autorisation.
3. Les rôles système ne sont modifiables que par migration ou procédure d’administration renforcée.
4. Les affectations globales et les rôles à risque élevé nécessitent une authentification forte et, lorsque configuré, une double approbation.
5. Toute création, suspension, expiration ou révocation génère un événement d’audit.
6. La révocation prend effet immédiatement dans les caches et sessions d’autorisation.
7. Les permissions finales résultent de l’union des rôles actifs dans la portée demandée, puis de l’application des politiques `DENY` prioritaires.

## Modèle de données minimal

Une affectation conserve au minimum : acteur, type d’acteur, rôle, portée, statut, période de validité, auteur de l’affectation, motif, date de création et informations de révocation.

Les suppressions physiques sont interdites pour les affectations ayant déjà été actives. Les données sont conservées selon la politique d’audit et de rétention applicable.

## Critères d’acceptation

- Les statuts et types de portée sont centralisés, typés et sans doublon.
- Le sous-chemin documenté est exporté par le package de contrats.
- Une affectation expirée ou révoquée ne produit aucune permission.
- Une permission limitée à un commerçant ou un point de vente ne s’applique pas à un autre périmètre.
- Les opérations sensibles sont refusées lorsqu’aucune affectation valide ne les autorise.
- Toute modification d’affectation est traçable avec acteur, motif, horodatage et identifiant de corrélation.
