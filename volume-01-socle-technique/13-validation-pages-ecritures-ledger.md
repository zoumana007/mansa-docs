# Validation des pages d’écritures comptables

## 1. Objectif

Le contrat `listEntries` du grand livre retourne une page d’écritures et, lorsqu’une page suivante existe, un curseur opaque. Après la validation des paramètres de requête, ce lot sécurise aussi la forme de la réponse avant qu’elle ne soit transmise à une couche HTTP ou consommée par un autre service.

La validation reste volontairement centrée sur la pagination et les champs d’identité stables. Les invariants monétaires et comptables restent dans les validateurs métier du ledger.

## 2. Contrat concerné

La route reste :

```text
GET /v1/internal/ledger/accounts/:accountId/entries
```

La réponse conserve la forme :

```ts
interface LedgerEntryPage {
  readonly items: readonly LedgerEntry[];
  readonly nextCursor?: string;
}
```

Le validateur est implémenté dans `packages/contracts/src/ledger-entry-page.ts` et réexporté depuis `@mansa/contracts/ledger-api`.

## 3. Règles de validation

### 3.1 Taille maximale

Une page ne peut pas contenir plus de 200 écritures, afin de rester cohérente avec la borne maximale déjà imposée au paramètre `limit` de la requête.

Code d’erreur : `PAGE_TOO_LARGE`.

### 3.2 Curseur suivant

Lorsque `nextCursor` est présent, il doit contenir au moins un caractère non blanc.

Le contrat ne tente pas de décoder le curseur : son format reste une responsabilité de l’adaptateur de stockage qui l’a émis.

Code d’erreur : `EMPTY_NEXT_CURSOR`.

### 3.3 Identité d’une écriture

Chaque écriture de la page doit avoir :

- un `id` non vide ;
- un `transactionId` non vide ;
- un `accountId` non vide.

Les identifiants d’écriture doivent aussi être uniques à l’intérieur de la page afin d’éviter qu’une pagination ou une jointure mal configurée ne duplique silencieusement une même écriture.

Codes d’erreur :

- `INVALID_ENTRY_ID` ;
- `INVALID_TRANSACTION_ID` ;
- `INVALID_ACCOUNT_ID` ;
- `DUPLICATE_ENTRY_ID`.

### 3.4 Séquence

`sequence` doit être un entier strictement positif.

Code d’erreur : `INVALID_SEQUENCE`.

### 3.5 Date de comptabilisation

`postedAt` doit être une date-heure valide et non vide.

Code d’erreur : `INVALID_POSTED_AT`.

## 4. API publique

Les consommateurs doivent utiliser :

```ts
import {
  validateLedgerEntryPage,
  type LedgerEntryPageValidationResult,
} from '@mansa/contracts/ledger-api';
```

Le tableau stable `LEDGER_ENTRY_PAGE_VALIDATION_ERROR_CODES` et le helper `isLedgerEntryPageValidationErrorCode` sont également exposés par ce point d’entrée.

## 5. Séparation des responsabilités

Ce validateur ne remplace pas :

- `validateLedgerEntries`, qui contrôle les invariants comptables d’un ensemble d’écritures ;
- `validateListLedgerEntriesQuery`, qui contrôle les filtres et limites de la requête ;
- la validation spécifique du curseur par l’adaptateur de persistance ;
- les contrôles d’autorisation sur le compte consulté.

Cette séparation évite de mélanger contrat API, règles comptables et détails PostgreSQL ou Redis.

## 6. Tests de non-régression

Le lot ajoute deux niveaux de couverture :

1. `packages/contracts/test/ledger-entry-page.test.mjs` couvre page valide, curseur vide, doublons, identifiants invalides, séquence, date et taille maximale ;
2. `packages/contracts/test/ledger-api-entry-page-export.test.mjs` vérifie que le validateur est réellement accessible depuis le point d’entrée public `ledger-api`.

## 7. Invariant à conserver pour la persistance

Tout futur adaptateur de lecture du grand livre doit construire une `LedgerEntryPage` conforme avant de retourner son résultat.

Une implémentation de pagination ne doit jamais :

- retourner plus de 200 éléments ;
- émettre un curseur vide ;
- retourner deux fois le même identifiant d’écriture dans une page ;
- masquer une écriture sans identité ou sans date comptable valide.

## 8. Suite recommandée

Le prochain lot peut maintenant définir le contrat de pagination déterministe côté persistance :

- ordre de lecture stable ;
- composition et signature du curseur interne ;
- index PostgreSQL adaptés à `accountId` et `postedAt` ;
- tests de passage d’une page à l’autre sans doublon ni omission ;
- stratégie de reconstruction et de réconciliation des soldes.
