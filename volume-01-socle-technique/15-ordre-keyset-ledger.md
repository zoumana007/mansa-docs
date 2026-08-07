# Ordre keyset du grand livre

## 1. Objectif

Le contrat de curseur du grand livre définit déjà une position stable avec `postedAt` et `entryId`. Ce lot formalise maintenant l’ordre total utilisé pour comparer ces positions et empêcher les incohérences entre l’API, les adaptateurs de persistance et les tests.

La pagination reste fondée sur la route :

```text
GET /v1/internal/ledger/accounts/:accountId/entries
```

Le curseur transmis au client demeure opaque.

## 2. Ordre canonique

L’ordre keyset de référence est :

```text
postedAt ASC, entryId ASC
```

Une position A est strictement antérieure à une position B lorsque :

1. `A.postedAt < B.postedAt` ; ou
2. les timestamps sont identiques et `A.entryId < B.entryId`.

Cet ordre total évite les doublons et les omissions lorsque plusieurs écritures partagent exactement le même timestamp.

## 3. Nouveau contrat

Le fichier `packages/contracts/src/ledger-entry-keyset.ts` introduit :

- `LedgerEntryPosition` ;
- `compareLedgerEntryPositions` ;
- `isLedgerEntryAfterCursor` ;
- `validateLedgerEntryCursorAccount` ;
- `LEDGER_ENTRY_KEYSET_MATCH_ERROR_CODES` ;
- `isLedgerEntryKeysetMatchErrorCode`.

Le helper `compareLedgerEntryPositions` retourne `-1`, `0` ou `1` et constitue la définition applicative de l’ordre attendu par l’adaptateur de persistance.

## 4. Liaison du curseur au compte

Un curseur émis pour un compte ne doit jamais être réutilisé pour parcourir les écritures d’un autre compte.

`validateLedgerEntryCursorAccount(accountId, cursor)` impose cette invariant et retourne le code stable :

```text
ACCOUNT_MISMATCH
```

Le contrôle doit être exécuté après décodage et vérification d’intégrité du curseur, avant toute requête de reprise.

## 5. Sémantique de reprise

`isLedgerEntryAfterCursor(entry, cursor)` retourne vrai uniquement lorsqu’une écriture est strictement postérieure à la position du curseur.

La traduction SQL/Prisma cible reste équivalente à :

```text
account_id = :accountId
AND (
  posted_at > :cursorPostedAt
  OR (posted_at = :cursorPostedAt AND id > :cursorEntryId)
)
ORDER BY posted_at ASC, id ASC
LIMIT :limit
```

L’adaptateur réel devra produire une requête équivalente sans réinterpréter différemment l’ordre défini par le contrat.

## 6. Export public

Les helpers sont réexportés depuis `@mansa/contracts/ledger-api` afin que les couches API et infrastructure partagent la même définition.

Exemple :

```ts
import {
  compareLedgerEntryPositions,
  isLedgerEntryAfterCursor,
  validateLedgerEntryCursorAccount,
} from '@mansa/contracts/ledger-api';
```

## 7. Tests

Le lot ajoute :

- `packages/contracts/test/ledger-entry-keyset.test.mjs` ;
- `packages/contracts/test/ledger-api-entry-keyset-export.test.mjs`.

Les tests couvrent :

- l’ordre par timestamp ;
- le départage par identifiant ;
- l’égalité d’une position ;
- la reprise strictement postérieure ;
- le rejet d’un curseur lié à un autre compte ;
- l’export depuis le point d’entrée public `ledger-api`.

## 8. Sécurité et invariants

- Le client ne doit jamais fournir directement un payload `LedgerEntryCursor` non authentifié.
- L’intégrité du curseur doit être vérifiée avant `validateLedgerEntryCursorAccount`.
- Aucune clé de signature n’est stockée dans les dépôts.
- Le comparateur ne remplace pas la validation des dates du curseur ; `validateLedgerEntryCursor` doit être exécuté en amont.
- L’ordre de la base doit rester cohérent avec l’ordre du contrat.

## 9. Suite recommandée

Le prochain lot doit passer de la définition contractuelle à l’adaptateur réel :

- encodeur/décodeur opaque de curseur ;
- signature ou MAC avec secret injecté ;
- requête Prisma/PostgreSQL keyset ;
- tests d’intégration sur plusieurs pages ;
- scénario avec plusieurs écritures partageant le même `postedAt` ;
- scénario de curseur falsifié et de curseur d’un autre compte.
