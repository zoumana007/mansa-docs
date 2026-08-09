# Rapprochement financier — pagination et résolution manuelle

## Objet

Cette spécification complète le moteur de rapprochement financier avec deux garanties opérationnelles : une pagination par curseur stable pour les lots et les écarts, et une résolution manuelle atomique, idempotente et auditée des écarts.

## Pagination par curseur

Les lectures de rapprochement ne doivent pas utiliser un offset comme mécanisme de référence. Le curseur encode le couple de tri `(createdAt, id)` afin de conserver un ordre déterministe même lorsque plusieurs lignes partagent le même horodatage.

### Lots

Ordre de lecture :

```text
createdAt DESC, id DESC
```

Le curseur suivant désigne la dernière ligne effectivement renvoyée. Une requête suivante ne retourne que les lignes strictement antérieures selon ce couple de tri.

### Items

Ordre de lecture :

```text
createdAt ASC, id ASC
```

Le curseur suivant désigne la dernière ligne effectivement renvoyée. Une requête suivante ne retourne que les lignes strictement postérieures selon ce couple de tri.

### Contraintes

- le curseur est opaque pour le client ;
- un curseur mal formé produit une erreur client et non une erreur serveur générique ;
- les limites restent bornées côté serveur ;
- la réponse expose `data`, `hasNextPage` et, seulement si nécessaire, `nextCursor` ;
- le même ordre et le même format doivent être utilisés par les routes internes et les futurs contrats publics correspondants ;
- aucune donnée sensible ne doit être encodée dans le curseur.

## Résolution manuelle

Un item peut être résolu manuellement uniquement lorsqu’il est dans l’état `MISMATCHED` ou `PARTIALLY_MATCHED`.

États cibles autorisés :

```text
RESOLVED
IGNORED
```

La commande exige :

- `itemId` ;
- état cible ;
- note de résolution non vide ;
- code motif ;
- clé d’idempotence ;
- identifiant de corrélation ;
- identité de l’acteur ;
- type de l’acteur.

## Atomicité

La mutation de l’item, l’incrément du compteur de lot et la création de l’audit opérationnel appartiennent à la même transaction PostgreSQL.

Si l’écriture d’audit échoue, la résolution doit être annulée. Aucun succès ne peut être retourné lorsque l’audit obligatoire n’a pas été persisté.

Pour `RESOLVED`, `resolvedItems` est incrémenté. Pour `IGNORED`, `ignoredItems` est incrémenté. `mismatchedItems` reste le compteur historique des écarts détectés lors du rapprochement ; il n’est pas décrémenté lors d’une résolution.

## Idempotence

`resolutionIdempotencyKey` est unique. Une répétition exacte de la même commande retourne l’état déjà persisté sans nouvel incrément de compteur et sans nouvel audit.

La réutilisation de la même clé pour un autre item ou un autre état cible est refusée.

## Audit

L’audit minimal contient :

```text
correlationId
actorId
actorType
action
resourceType = ReconciliationItem
resourceId
reason
metadata.batchId
metadata.resolutionNote
metadata.mismatchReason
```

Actions :

```text
RECONCILIATION_ITEM_RESOLVED
RECONCILIATION_ITEM_IGNORED
```

## Sécurité

La route reste protégée par l’authentification de service interne tant que le socle n’utilise pas encore une identité de workload attestée définitive. Cette protection transitoire ne doit pas devenir le mécanisme final de production.

À terme, l’identité de l’acteur doit être dérivée d’un contexte authentifié et non acceptée aveuglément depuis le corps HTTP.

## Critères de recette

1. Deux pages successives ne contiennent pas le même item.
2. Une page de taille 1 sur un lot de deux items retourne un `nextCursor`, puis la seconde page termine avec `hasNextPage = false`.
3. Un curseur invalide retourne une erreur 400.
4. Une résolution `MISMATCHED -> RESOLVED` persiste la note, le motif, l’acteur, la corrélation et la clé d’idempotence.
5. Le compteur `resolvedItems` est incrémenté une seule fois.
6. La répétition de la même commande est idempotente.
7. Un seul audit opérationnel est créé pour une commande rejouée.
8. Une clé d’idempotence réutilisée pour une autre opération est refusée.
9. Une tentative de résolution d’un item `MATCHED`, `RESOLVED` ou `IGNORED` est refusée.
10. L’échec de l’audit annule la mutation de l’item et du compteur.

## Correspondance code

Implémentation de référence :

- `mansa-platform/apps/api-gateway/src/reconciliation/reconciliation.repository.ts` ;
- `mansa-platform/apps/api-gateway/src/reconciliation/reconciliation.controller.ts` ;
- `mansa-platform/apps/api-gateway/test/reconciliation-postgres.test.mjs` ;
- `mansa-platform/apps/api-gateway/prisma/schema.prisma`.

La prochaine tranche doit compléter les tests de contrat HTTP, dériver l’identité d’acteur du contexte authentifié et ajouter les filtres du contrat partagé sur les listes de lots et d’items.
