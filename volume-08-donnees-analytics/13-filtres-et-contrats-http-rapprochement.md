# Filtres de consultation et contrats HTTP du rapprochement

## 1. Objet

Cette tranche aligne les routes internes de rapprochement financier avec les contrats partagés `@mansa/contracts/reconciliation-api` et `@mansa/contracts/reconciliation`.

Elle complète l'isolation organisationnelle déjà appliquée dans PostgreSQL en ajoutant deux garanties de transport :

- les filtres de consultation prévus par le contrat sont réellement propagés jusqu'aux requêtes Prisma ;
- les réponses HTTP ne renvoient plus directement les modèles Prisma complets mais des DTO publics explicitement sérialisés.

## 2. Filtres sur les lots

`GET /v1/internal/reconciliation/batches` accepte désormais, en plus de `organizationId`, `limit` et `cursor` :

```text
providerId
status
periodStartFrom
periodEndTo
```

Les filtres sont appliqués dans la clause `where` Prisma, avec la portée `organizationId` dans la même requête.

Règles :

- `providerId` est normalisé par `trim` ;
- `status` doit appartenir à `RECONCILIATION_BATCH_STATUSES` ;
- `periodStartFrom` devient une borne `periodStart >= ...` ;
- `periodEndTo` devient une borne `periodEnd <= ...` ;
- une date invalide produit une erreur HTTP `400` ;
- une valeur d'énumération non supportée produit une erreur HTTP `400`.

## 3. Filtres sur les items

`GET /v1/internal/reconciliation/batches/:batchId/items` accepte désormais :

```text
providerId
status
mismatchReason
internalReference
providerReference
createdFrom
createdTo
```

Les règles sont les suivantes :

- le lot parent est d'abord vérifié dans la portée de l'organisation ;
- `providerId` est appliqué via la relation vers le lot ;
- `status` doit appartenir à `RECONCILIATION_STATUSES` ;
- `mismatchReason` doit appartenir à `RECONCILIATION_MISMATCH_REASONS` ;
- les références sont comparées après suppression des espaces de bord ;
- `createdFrom` et `createdTo` bornent `createdAt` ;
- tous les filtres restent cumulables avec la pagination keyset.

La pagination reste ordonnée par `createdAt ASC, id ASC` pour les items et `createdAt DESC, id DESC` pour les lots.

## 4. Sérialisation stricte des lots

Les réponses de lot suivent `ReconciliationBatchSummary`.

Champs exposés :

```text
batchId
providerId
sourceFileReference?
periodStart
periodEnd
status
totalItems
matchedItems
mismatchedItems
resolvedItems
ignoredItems
createdAt
startedAt?
completedAt?
failureReason?
```

Ne sont notamment pas exposés par ce DTO :

- `organizationId` ;
- `sourceFingerprint` ;
- `metadata` interne ;
- tout autre champ Prisma non prévu par le contrat.

Les dates sont sérialisées en ISO 8601.

## 5. Sérialisation stricte des items

Les réponses d'item suivent `ReconciliationItem`.

Champs exposés :

```text
itemId
batchId
internalReference?
providerReference?
internalAmountMinor?
providerAmountMinor?
currency
status
mismatchReason?
resolutionNote?
createdAt
updatedAt
```

Ne sont pas exposés :

- `organizationId` ;
- `rawLineFingerprint` ;
- `internalStatus` et `providerStatus` internes ;
- `providerOccurrenceCount` ;
- `resolutionReasonCode` ;
- `resolvedBy` ;
- `resolutionCorrelationId` ;
- `resolutionIdempotencyKey`.

Les montants persistés en `bigint` sont convertis uniquement s'ils restent des entiers sûrs JavaScript non négatifs. Le contrat public ne doit jamais tronquer silencieusement un montant.

## 6. Résolutions

La route de résolution conserve les garanties précédentes :

- portée organisationnelle obligatoire ;
- transition limitée à `RESOLVED | IGNORED` ;
- idempotence persistante ;
- audit transactionnel ;
- refus des résolutions inter-tenant.

Après mutation, la réponse passe également par le même présentateur public que les lectures. Une résolution ne peut donc pas faire fuiter les champs opérationnels internes qui ne font pas partie du contrat.

## 7. Validation

`mansa-platform/apps/api-gateway/test/reconciliation-controller.test.mjs` couvre désormais :

- propagation et normalisation des filtres de lots ;
- rejet des statuts de lot non supportés ;
- rejet des dates invalides ;
- propagation et normalisation des filtres d'items ;
- rejet des statuts et motifs d'écart non supportés ;
- conservation de la portée organisationnelle ;
- sérialisation des dates ;
- conversion des montants `bigint` ;
- absence des champs Prisma internes dans les réponses publiques ;
- sérialisation stricte de la réponse de résolution.

Les tests PostgreSQL d'isolation tenant restent complémentaires : ils démontrent que les filtres de transport ne remplacent pas le scoping au niveau de la persistance.

## 8. État de la tranche

À l'issue de cette tranche, les éléments suivants sont implémentés pour le rapprochement :

1. moteur de comparaison déterministe ;
2. persistance PostgreSQL ;
3. imports idempotents ;
4. résolution atomique et auditée ;
5. pagination keyset ;
6. isolation organisationnelle dans le schéma, le repository et les routes ;
7. filtres de consultation du contrat partagé ;
8. sérialisation HTTP stricte vers les DTO publics.

## 9. Étapes suivantes

La prochaine tranche prioritaire est l'identité workload attestée des routes internes afin que `organizationId` ne soit plus une valeur arbitraire fournie par l'appelant mais une portée dérivée d'une identité de service authentifiée et autorisée.

Ensuite viennent :

1. métriques et alertes de rapprochement ;
2. instrumentation des volumes, écarts et délais de résolution ;
3. premiers adaptateurs de partenaires réels derrière interfaces ;
4. tests de charge et de concurrence élargis ;
5. procédures de recette partenaire sans secret versionné.
