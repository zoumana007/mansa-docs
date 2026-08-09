# Contrat API interne du rapprochement financier

## 1. Objet

Ce document fixe le contrat de transport du rapprochement financier Mansa et aligne explicitement la documentation avec les routes réellement exposées par l’API Gateway.

Le rapprochement est un service interne. Aucune route publique non authentifiée ne doit permettre de consulter ou modifier les lots et écarts.

## 2. Espace de routes

Toutes les routes de cette tranche sont protégées par l’identité de service interne et commencent par :

```text
/v1/internal/reconciliation
```

Routes de référence :

```text
GET  /v1/internal/reconciliation/batches
GET  /v1/internal/reconciliation/batches/:batchId
GET  /v1/internal/reconciliation/batches/:batchId/items
GET  /v1/internal/reconciliation/items/:itemId
POST /v1/internal/reconciliation/items/:itemId/resolve
```

Les constantes TypeScript correspondantes sont maintenues dans `mansa-platform/packages/contracts/src/reconciliation-api.ts`.

## 3. Pagination par curseur

Les listes utilisent le contrat transverse `PageResponse<T>` défini dans `packages/contracts/src/pagination.ts`.

Réponse :

```json
{
  "data": [],
  "page": {
    "hasNextPage": false,
    "nextCursor": "opaque-et-optionnel"
  }
}
```

Règles :

- `nextCursor` est absent lorsqu’il n’existe pas de page suivante ;
- le curseur est opaque pour le consommateur ;
- le consommateur ne doit jamais reconstruire ou modifier le curseur ;
- les lots sont ordonnés par `createdAt DESC, id DESC` ;
- les items d’un lot sont ordonnés par `createdAt ASC, id ASC` ;
- le couple horodatage + identifiant sert de frontière stable en cas d’égalité de date ;
- un curseur malformé produit une erreur client et n’est jamais interprété partiellement.

Limites actuelles :

- lots : défaut `50`, maximum `100` ;
- items d’un lot : défaut `100`, maximum `500`.

## 4. Consultation des lots

### `GET /v1/internal/reconciliation/batches`

Paramètres actuels :

```text
limit?
cursor?
```

Les filtres fournisseur, statut et période existent dans le contrat cible mais ne doivent être annoncés comme disponibles qu’après leur implémentation côté repository/controller.

### `GET /v1/internal/reconciliation/batches/:batchId`

`batchId` est un UUID v4.

Une ressource inconnue renvoie une erreur `404`.

## 5. Consultation des items

### `GET /v1/internal/reconciliation/batches/:batchId/items`

La liste est toujours rattachée explicitement à un lot. Le `batchId` est donc obligatoire dans le chemin et ne doit pas être remplacé par un filtre libre tant que l’isolation de lecture transverse n’est pas implémentée.

Paramètres :

```text
limit?
cursor?
```

### `GET /v1/internal/reconciliation/items/:itemId`

Permet une lecture ponctuelle d’un item par UUID v4.

## 6. Résolution manuelle

### `POST /v1/internal/reconciliation/items/:itemId/resolve`

Corps minimal :

```json
{
  "status": "RESOLVED",
  "resolutionNote": "Écart confirmé avec le fournisseur.",
  "reasonCode": "PROVIDER_CONFIRMED",
  "idempotencyKey": "opaque-key",
  "correlationId": "opaque-correlation",
  "actorId": "service-or-user-id",
  "actorType": "SERVICE_ACCOUNT"
}
```

`status` accepte uniquement :

```text
RESOLVED
IGNORED
```

Règles obligatoires :

1. seuls les états `MISMATCHED` et `PARTIALLY_MATCHED` sont résolubles ;
2. `resolutionNote`, `reasonCode`, `idempotencyKey`, `correlationId`, `actorId` et `actorType` sont non vides ;
3. la même clé d’idempotence rejouée pour le même item et le même statut retourne le résultat existant ;
4. une même clé réutilisée pour un autre item ou un autre statut est refusée ;
5. la mutation de l’item, l’incrément du compteur du lot et la création du journal d’audit appartiennent à la même transaction PostgreSQL ;
6. si l’audit échoue, la résolution entière est annulée ;
7. le motif d’écart initial n’est jamais supprimé ;
8. `IGNORED` reste visible dans les statistiques et dans l’historique d’audit.

## 7. Audit opérationnel

Une résolution produit une entrée `OperationalAuditLog` avec :

```text
correlationId
actorId
actorType
action
resourceType = ReconciliationItem
resourceId
reason
metadata.batchId
metadata.mismatchReason
metadata.resolutionNote
```

Actions :

```text
RECONCILIATION_ITEM_RESOLVED
RECONCILIATION_ITEM_IGNORED
```

Les logs ne doivent jamais contenir de secret fournisseur, numéro de carte complet, credential bancaire ou payload brut inutile.

## 8. Sécurité

Les routes utilisent `InternalServiceGuard` dans la tranche actuelle.

Avant production, cette identité interne transitoire doit être remplacée ou renforcée par une identité de workload attestée et une politique d’autorisation explicite par service appelant.

La résolution manuelle doit ensuite distinguer clairement :

- l’identité technique du service qui transporte la requête ;
- l’identité de l’opérateur humain lorsque la décision provient d’un portail d’administration ;
- les permissions métier de résolution ;
- les exigences de double validation éventuelles selon montant, fournisseur ou type d’écart.

## 9. Cohérence code/documentation

Références de code :

```text
packages/contracts/src/pagination.ts
packages/contracts/src/reconciliation.ts
packages/contracts/src/reconciliation-api.ts
apps/api-gateway/src/reconciliation/reconciliation.controller.ts
apps/api-gateway/src/reconciliation/reconciliation.repository.ts
apps/api-gateway/test/reconciliation-postgres.test.mjs
```

À partir de cette version, le contrat partagé utilise les mêmes chemins internes que le contrôleur et le repository renvoie l’enveloppe transverse `data + page`.

## 10. Tests requis

Déjà couverts au niveau PostgreSQL :

- persistance et normalisation ;
- idempotence séquentielle d’import ;
- idempotence concurrente d’import ;
- pagination stable ;
- limite de page ;
- résolution manuelle ;
- audit unique ;
- replay idempotent.

Reste à ajouter dans une tranche suivante :

- tests HTTP réels des cinq routes ;
- refus du guard sans identité valide ;
- validation UUID au niveau HTTP ;
- traduction stable des erreurs métier en statuts HTTP ;
- filtres de lots/items prévus par le contrat cible ;
- tests d’autorisation par rôle lorsque l’identité opérateur sera disponible.
