# API d’autorisation, rôles et audit

## 1. Objet

Ce catalogue décrit les contrats d’administration nécessaires pour évaluer les autorisations, consulter les permissions effectives, gérer les rôles, suspendre ou restaurer un acteur et consulter le journal d’audit.

La référence technique est `packages/contracts/src/admin-api.ts` dans `mansa-platform`. Le catalogue est aussi exporté par le registre transversal `@mansa/contracts/api-contracts`.

## 2. Préfixe et routes

Préfixe : `/v1/admin`.

| Opération | Méthode | Route | Permission minimale |
| --- | --- | --- | --- |
| Évaluer une autorisation | `POST` | `/authorization/evaluate` | `authorization.evaluate` |
| Lire les permissions effectives | `GET` | `/actors/:actorId/effective-permissions` | `actor.permissions.read` |
| Attribuer un rôle | `POST` | `/actors/:actorId/roles` | `role.assignment.create` |
| Révoquer un rôle | `DELETE` | `/actors/:actorId/roles/:roleId` | `role.assignment.revoke` |
| Consulter les événements d’audit | `GET` | `/audit-events` | `audit.event.read` |
| Suspendre un acteur | `POST` | `/actors/:actorId/suspension` | `actor.suspend` |
| Restaurer un acteur | `POST` | `/actors/:actorId/restoration` | `actor.restore` |

## 3. Matrice de séparation des responsabilités

| Action | Opérateur support | Responsable support | Conformité | Risque | Administrateur sécurité | Super administrateur |
| --- | --- | --- | --- | --- | --- | --- |
| Lire un profil | Oui, périmètre assigné | Oui | Oui | Oui | Oui | Oui |
| Lire les permissions | Non | Oui, périmètre assigné | Oui | Oui | Oui | Oui |
| Attribuer un rôle standard | Non | Proposition | Validation selon périmètre | Non | Oui | Oui |
| Attribuer un rôle privilégié | Non | Non | Co-validation | Co-validation | Proposition | Validation finale |
| Révoquer un rôle | Non | Proposition | Oui si motif conformité | Oui si motif risque | Oui | Oui |
| Suspendre un acteur | Non | Oui, temporaire | Oui | Oui | Oui | Oui |
| Restaurer un acteur | Non | Proposition | Validation si conformité | Validation si risque | Oui | Oui |
| Lire l’audit global | Non | Périmètre support | Oui | Oui | Oui | Oui |
| Modifier une politique | Non | Non | Proposition | Proposition | Oui avec double contrôle | Oui avec double contrôle |

Les intitulés de rôle sont des profils de référence. Les permissions effectives restent calculées à partir du catalogue de permissions, des affectations, des portées, des dates de validité et des politiques actives.

## 4. Règles obligatoires

- Toute mutation exige une clé d’idempotence.
- Toute attribution ou révocation exige un motif explicite.
- Les rôles privilégiés nécessitent une authentification multifacteur et un double contrôle.
- Un administrateur ne peut pas approuver sa propre demande d’élévation.
- Les permissions sont refusées par défaut.
- Une suspension doit invalider les sessions actives et empêcher toute nouvelle authentification sensible.
- Une restauration ne rétablit pas automatiquement les anciennes sessions.
- Les recherches d’audit utilisent une pagination par curseur et des filtres bornés.
- Les événements d’audit ne contiennent ni secret, ni jeton, ni donnée KYC brute.

## 5. Décision d’autorisation

L’évaluation retourne au minimum :

- `allowed` : résultat final ;
- `reasonCode` : justification stable et exploitable ;
- `obligations` : contrôles supplémentaires à satisfaire ;
- `evaluatedPolicyIds` : politiques ayant participé à la décision.

Exemples d’obligations : authentification multifacteur, double approbation, limitation au pays, limitation au commerçant, masquage de données ou justification renforcée.

## 6. Journal d’audit

Chaque événement contient un identifiant, la date, l’acteur, l’action, la ressource, le résultat et l’identifiant de corrélation. Les événements sont append-only. Toute correction métier produit un nouvel événement au lieu de modifier l’historique.

Les accès au journal sont eux-mêmes audités. Les exports volumineux passent par un traitement asynchrone contrôlé, avec expiration du lien et chiffrement du fichier.

## 7. Critères de recette

1. Un acteur sans permission reçoit un refus explicite sans fuite d’information.
2. Une attribution rejouée avec la même clé d’idempotence ne crée pas de doublon.
3. Un rôle expiré disparaît des permissions effectives.
4. Une élévation privilégiée sans double contrôle est refusée.
5. Une suspension invalide les sessions et bloque les actions protégées.
6. Une restauration est auditée et ne recrée aucune session.
7. Les routes, méthodes et types correspondent à `admin-api.ts`.
8. Les montants éventuels sont transmis sous forme décimale sérialisée, jamais en nombre flottant.
9. Les événements peuvent être reliés de bout en bout grâce au `correlationId`.
10. Aucun secret ou donnée personnelle brute n’est présent dans les réponses d’audit.
