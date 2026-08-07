# Pagination déterministe du grand livre

## 1. Objectif

Après la validation des paramètres de lecture et des pages retournées par `listEntries`, le grand livre doit garantir une pagination stable : une écriture ne doit ni disparaître ni apparaître deux fois lorsqu’un consommateur parcourt plusieurs pages.

Ce lot introduit le contrat du curseur interne utilisé pour construire les curseurs opaques exposés par l’API ledger.

## 2. Route concernée

La route reste :

```text
GET /v1/internal/ledger/accounts/:accountId/entries
```

Le paramètre `cursor` reste une chaîne opaque pour le consommateur. Le client ne doit jamais construire, modifier ou interpréter ce curseur.

## 3. Charge utile canonique du curseur

Le contrat interne est défini dans `packages/contracts/src/ledger-entry-cursor.ts` :

```ts
interface LedgerEntryCursor {
  readonly version: 1;
  readonly accountId: string;
  readonly postedAt: string;
  readonly entryId: string;
}
```

Le triplet `accountId`, `postedAt`, `entryId` fournit une position stable dans le flux d’écritures d’un compte.

`postedAt` détermine l’ordre temporel principal et `entryId` fournit un départage déterministe lorsque plusieurs écritures partagent exactement la même date de comptabilisation.

## 4. Versionnement

Le curseur possède un champ `version`.

La seule version acceptée actuellement est `1`, exposée via :

```ts
LEDGER_ENTRY_CURSOR_VERSIONS
```

L’ajout futur d’une nouvelle représentation doit préserver la capacité à identifier explicitement la version décodée. Une ancienne version ne doit pas être réinterprétée silencieusement selon un nouveau format.

## 5. Validation

`validateLedgerEntryCursor` vérifie :

- que la version est supportée ;
- que `accountId` n’est pas vide ;
- que `postedAt` est une date-heure valide ;
- que `entryId` n’est pas vide.

Codes d’erreur stables :

- `UNSUPPORTED_VERSION` ;
- `INVALID_ACCOUNT_ID` ;
- `INVALID_POSTED_AT` ;
- `INVALID_ENTRY_ID`.

Le tableau `LEDGER_ENTRY_CURSOR_VALIDATION_ERROR_CODES` et le helper `isLedgerEntryCursorValidationErrorCode` permettent aux consommateurs internes de traiter ces erreurs sans comparer des messages humains.

## 6. API publique des contrats

Le contrat est réexporté depuis `@mansa/contracts/ledger-api` :

```ts
import {
  validateLedgerEntryCursor,
  type LedgerEntryCursor,
} from '@mansa/contracts/ledger-api';
```

L’exposition depuis `ledger-api` évite aux adaptateurs d’infrastructure de dépendre d’un chemin source privé.

## 7. Encodage et sécurité

Le package `@mansa/contracts` ne sérialise pas et ne signe pas le curseur. Il décrit uniquement sa charge utile canonique.

L’adaptateur de persistance doit :

1. construire un `LedgerEntryCursor` valide à partir de la dernière écriture retournée ;
2. le sérialiser dans un format déterministe ;
3. protéger le curseur contre la modification par le client, par exemple avec une signature authentifiée gérée par l’infrastructure ;
4. produire une chaîne opaque destinée à `nextCursor` ;
5. vérifier l’intégrité avant de réutiliser un curseur entrant.

Aucune clé, aucun secret de signature et aucun exemple de secret ne doit être stocké dans le dépôt.

## 8. Ordre de lecture recommandé

L’adaptateur PostgreSQL doit utiliser un ordre total stable, par exemple :

```text
posted_at ASC, id ASC
```

Pour la page suivante, la condition de reprise doit être strictement postérieure à la dernière position :

```text
posted_at > cursor.postedAt
OR (posted_at = cursor.postedAt AND id > cursor.entryId)
```

Le filtre `accountId` doit toujours être appliqué avant la comparaison de position.

## 9. Index PostgreSQL recommandé

L’index cible pour ce parcours est :

```text
(account_id, posted_at, id)
```

Le nom exact de table et d’index dépendra du schéma Prisma final, mais l’ordre des colonnes doit rester aligné avec le filtre et l’ordre de pagination.

## 10. Invariants

Une implémentation conforme doit garantir :

- aucun doublon entre deux pages successives ;
- aucune omission d’une écriture déjà comptabilisée dans l’ordre parcouru ;
- un curseur lié au compte demandé ;
- un ordre total même lorsque plusieurs écritures possèdent le même `postedAt` ;
- le rejet d’un curseur mal formé, d’une version inconnue ou d’un curseur dont l’intégrité n’est pas vérifiée.

## 11. Tests ajoutés

Le lot ajoute :

- `packages/contracts/test/ledger-entry-cursor.test.mjs`, qui couvre la validation du payload ;
- `packages/contracts/test/ledger-api-entry-cursor-export.test.mjs`, qui vérifie l’export depuis le point d’entrée public `ledger-api`.

## 12. Suite recommandée

Le prochain lot peut implémenter l’adaptateur PostgreSQL/Prisma de pagination :

- requête keyset réelle ;
- encodeur/décodeur de curseur dans la couche infrastructure ;
- protection cryptographique via secret injecté ;
- tests d’intégration avec plusieurs écritures partageant le même timestamp ;
- tests de passage de page sans doublon ni omission.
