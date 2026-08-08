# Export public de la fenêtre de pagination du grand livre

## 1. Objectif

Ce lot ferme l’écart entre l’implémentation de la fenêtre `limit + 1` et le contrat public du module ledger.

Le helper `createLedgerEntryFetchWindow` existait déjà dans :

```text
packages/contracts/src/ledger-entry-fetch-window.ts
```

mais il n’était pas encore réexporté depuis l’agrégateur public :

```text
packages/contracts/src/ledger-api.ts
```

Un consommateur du package aurait donc dû importer directement le fichier interne, ce qui cassait la frontière publique du module.

## 2. Export public ajouté

`ledger-api.ts` expose désormais :

- `LEDGER_ENTRY_FETCH_WINDOW_ERROR_CODES` ;
- `createLedgerEntryFetchWindow` ;
- `isLedgerEntryFetchWindowErrorCode` ;
- `LedgerEntryFetchWindowError` ;
- `LedgerEntryFetchWindowErrorCode` ;
- `LedgerEntryFetchWindowResult`.

Les applications et adaptateurs doivent importer ces symboles via le point d’entrée public du ledger, et non via un chemin interne du package.

## 3. Pourquoi cet export est important

La pagination du grand livre est une règle de contrat partagée entre la couche API et la persistance :

1. le stockage lit jusqu’à `limit + 1` lignes ;
2. `createLedgerEntryFetchWindow` réduit la fenêtre à la page visible ;
3. `hasNextPage` est calculé depuis la ligne supplémentaire ;
4. `nextCursor` est construit depuis la dernière écriture réellement retournée.

En exposant cette logique depuis `ledger-api.ts`, tous les adaptateurs utilisent la même règle et évitent de réimplémenter localement la pagination.

## 4. Compatibilité

Le changement est additif :

- aucune route existante n’est modifiée ;
- aucun type existant n’est supprimé ;
- aucune signature précédente n’est changée ;
- aucun secret ni paramètre d’infrastructure n’est introduit.

## 5. Test de non-régression

Le fichier suivant est ajouté :

```text
packages/contracts/test/ledger-api-entry-fetch-window-export.test.mjs
```

Il vérifie que le build public de `ledger-api` expose bien :

- le code d’erreur stable `INVALID_LIMIT` ;
- `createLedgerEntryFetchWindow` ;
- `isLedgerEntryFetchWindowErrorCode`.

Ce test protège contre une suppression accidentelle de l’export lors d’un futur refactoring.

## 6. Règle d’architecture

Les consommateurs externes au sous-module ledger ne doivent pas importer directement :

```text
./ledger-entry-fetch-window.js
```

Ils doivent passer par :

```text
./ledger-api.js
```

Cette règle maintient une API publique explicite et réduit le couplage aux fichiers internes.

## 7. Suite recommandée

Le prochain lot doit implémenter un adaptateur de persistance de référence pour `listEntries` et démontrer sur plusieurs pages que l’ordre keyset `(postedAt, entryId)` ne crée ni doublon ni omission, y compris lorsque plusieurs écritures partagent exactement le même `postedAt`.
