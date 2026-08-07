# Validation des requêtes de consultation des écritures comptables

## 1. Objectif

Le contrat `listEntries` du grand livre permet de consulter les écritures d’un compte avec filtres temporels et pagination. Ce lot ajoute une validation explicite de cette requête avant son passage à un adaptateur de stockage.

L’objectif est d’éviter qu’une couche HTTP, un service interne ou un futur adaptateur PostgreSQL interprète différemment les mêmes paramètres.

## 2. Contrat concerné

La route reste :

```text
GET /v1/internal/ledger/accounts/:accountId/entries
```

Le type `ListLedgerEntriesQuery` contient :

- `accountId` obligatoire ;
- `from` optionnel ;
- `to` optionnel ;
- `cursor` optionnel ;
- `limit` optionnel.

La validation est implémentée dans `packages/contracts/src/ledger-entry-query.ts` et réexportée depuis le point d’entrée public `@mansa/contracts/ledger-api`.

## 3. Règles de validation

### 3.1 Compte

`accountId` doit contenir au moins un caractère non blanc.

### 3.2 Filtres temporels

Lorsqu’ils sont fournis, `from` et `to` doivent être des dates-heures interprétables.

Si les deux bornes sont valides et présentes :

```text
from <= to
```

L’égalité est autorisée afin de permettre une requête sur un instant exact.

### 3.3 Curseur

Le curseur reste volontairement opaque au niveau du contrat public. La validation garantit uniquement qu’un curseur fourni n’est pas vide.

Le décodage, la signature éventuelle et la vérification du contenu appartiennent à l’adaptateur de persistance qui émet ce curseur.

### 3.4 Limite de page

Lorsqu’elle est fournie, `limit` doit être un entier compris entre 1 et 200 inclus.

Cette borne protège les futurs adaptateurs contre des lectures non bornées tout en laissant une taille suffisante aux usages internes et aux opérations de réconciliation.

## 4. Codes d’erreur

Les codes stables exposés sont :

- `INVALID_ACCOUNT_ID` ;
- `INVALID_FROM` ;
- `INVALID_TO` ;
- `INVALID_DATE_RANGE` ;
- `INVALID_CURSOR` ;
- `INVALID_LIMIT`.

Le helper `isLedgerEntryQueryValidationErrorCode` permet de vérifier qu’une chaîne appartient bien à cet ensemble.

## 5. API publique

Les consommateurs doivent importer la validation depuis :

```ts
import {
  validateListLedgerEntriesQuery,
  type LedgerEntryQueryValidationResult,
} from '@mansa/contracts/ledger-api';
```

Les chemins internes `src/*` et `dist/*` ne constituent pas des API publiques pour les autres packages du monorepo.

## 6. Tests de non-régression

Deux niveaux sont couverts :

1. `packages/contracts/test/ledger-entry-query.test.mjs` vérifie les invariants métier, les bornes de pagination, les intervalles temporels et les codes d’erreur ;
2. `packages/contracts/test/ledger-api-entry-query-export.test.mjs` vérifie que la route et la méthode restent stables et que la validation est réellement accessible depuis `ledger-api`.

## 7. Règle d’architecture

Toutes les futures implémentations de `listEntries` doivent appeler cette validation, ou appliquer un validateur généré strictement équivalent, avant d’interroger le stockage.

La validation d’un curseur spécifique à PostgreSQL, Redis ou à une autre technologie ne doit pas remonter dans le contrat métier générique.

## 8. Suite recommandée

Le prochain lot du grand livre peut désormais traiter la persistance des projections et écritures avec :

- pagination déterministe ;
- ordre stable des écritures ;
- index adaptés à `accountId` et à la date comptable ;
- reconstruction des soldes ;
- contrôle de cohérence entre écritures et projections ;
- stratégie de réconciliation et réparation des divergences.
