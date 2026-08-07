# Exposition publique de la validation des soldes comptables

## 1. Objectif

Le contrat de validation des projections de solde existe dans `packages/contracts/src/ledger-balance.ts`. Pour éviter que les consommateurs du package `@mansa/contracts` importent directement un fichier interne non garanti par la carte d’exports du package, ce lot rend ce contrat accessible depuis le sous-chemin public déjà stable `@mansa/contracts/ledger-api`.

Cette décision maintient la cohérence avec le reste du grand livre : les routes, méthodes, types d’API et règles de validation liées aux soldes sont regroupés derrière le même point d’entrée public.

## 2. Contrat exposé

Le module `packages/contracts/src/ledger-api.ts` réexporte désormais :

- `LEDGER_BALANCE_VALIDATION_ERROR_CODES` ;
- `validateLedgerBalance` ;
- `isLedgerBalanceValidationErrorCode` ;
- `LedgerBalanceValidationError` ;
- `LedgerBalanceValidationErrorCode` ;
- `LedgerBalanceValidationResult`.

Les consommateurs doivent privilégier :

```ts
import {
  validateLedgerBalance,
  type LedgerBalanceValidationResult,
} from '@mansa/contracts/ledger-api';
```

Ils ne doivent pas dépendre de chemins internes tels que `src/ledger-balance.ts` ou `dist/ledger-balance.js`.

## 3. Pourquoi utiliser `ledger-api`

Le `package.json` de `@mansa/contracts` publie déjà le sous-chemin `./ledger-api`. Réexporter la validation depuis ce point d’entrée permet :

1. de garantir un import supporté par la carte `exports` Node ;
2. d’éviter l’ajout d’un sous-chemin supplémentaire uniquement pour une fonction de validation liée au grand livre ;
3. de garder une surface publique réduite et cohérente ;
4. de permettre une évolution interne de l’implémentation sans casser les consommateurs.

## 4. Test de non-régression

Le test `packages/contracts/test/ledger-api-balance-export.test.mjs` vérifie que :

- la route `getBalance` reste `/v1/internal/ledger/accounts/:accountId/balance` ;
- sa méthode reste `GET` ;
- `validateLedgerBalance` est importable depuis le module compilé `ledger-api.js` ;
- les codes de validation sont accessibles depuis le même point d’entrée ;
- une projection valide est acceptée via cette surface publique.

Le script de tests du package compile d’abord les contrats TypeScript, puis exécute les tests Node. Le test échouera donc si une réexportation est supprimée ou si le module ne compile plus.

## 5. Règle d’architecture

À partir de ce lot, tout nouveau consommateur externe au package `contracts` doit utiliser les sous-chemins explicitement déclarés dans `packages/contracts/package.json`.

Les imports relatifs vers `src/*` ou les imports directs vers `dist/*` sont réservés aux tests internes du package et ne constituent pas une API publique.

## 6. Suite

Les prochains lots du grand livre peuvent poursuivre avec :

- la persistance PostgreSQL des projections de solde ;
- un numéro de version ou une séquence monotone par projection ;
- la reconstruction déterministe des soldes depuis les écritures ;
- la vérification de cohérence entre devise du compte et devise de la projection ;
- la concurrence optimiste ou les verrous adaptés aux mises à jour simultanées ;
- la réconciliation périodique et la réparation des projections divergentes.
