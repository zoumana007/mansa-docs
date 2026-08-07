# Construction du curseur suivant du grand livre

## 1. Objectif

Ce lot formalise la construction du payload de curseur keyset utilisé pour demander la page suivante des écritures du grand livre.

La route concernée reste :

```text
GET /v1/internal/ledger/accounts/:accountId/entries
```

Le principe est volontairement simple : le curseur suivant est toujours construit depuis la dernière écriture effectivement retournée au client, et non depuis la limite demandée, un offset, une estimation ou une ligne non incluse dans la réponse.

## 2. Invariant principal

Pour une page non vide, le payload du curseur suivant reprend exactement :

- `version: 1` ;
- `accountId` de la dernière écriture retournée ;
- `postedAt` de cette écriture ;
- `entryId` égal à son identifiant.

Avec l’ordre canonique :

```text
postedAt ASC, entryId ASC
```

la requête suivante doit ensuite sélectionner uniquement les écritures strictement postérieures à cette position.

## 3. Nouveau contrat

Le fichier :

```text
packages/contracts/src/ledger-entry-next-cursor.ts
```

introduit :

- `createLedgerEntryCursorFromEntry` ;
- `createLedgerEntryNextCursor` ;
- `LEDGER_ENTRY_NEXT_CURSOR_ERROR_CODES` ;
- `LedgerEntryNextCursorErrorCode` ;
- `LedgerEntryNextCursorError` ;
- `LedgerEntryNextCursorResult` ;
- `isLedgerEntryNextCursorErrorCode`.

Le code d’erreur stable actuellement défini est :

```text
EMPTY_PAGE
```

Une page vide ne produit pas de curseur suivant artificiel.

## 4. Séparation entre payload et représentation opaque

Le package de contrats construit uniquement le payload stable `LedgerEntryCursor`.

Il ne réalise pas :

- l’encodage Base64 ou autre ;
- la signature cryptographique ;
- le chiffrement ;
- la gestion ou rotation de secrets.

Ces opérations appartiennent à la couche infrastructure. Les secrets doivent être injectés par l’environnement ou un gestionnaire de secrets et ne doivent jamais être ajoutés au dépôt.

## 5. Pourquoi partir de la dernière écriture retournée

Le curseur doit représenter la frontière réelle visible par le consommateur. Cela évite plusieurs défauts :

- omission d’une écriture lorsque le backend lit `limit + 1` lignes pour détecter une page suivante ;
- duplication lorsqu’un curseur est construit depuis une ligne antérieure ;
- incohérence entre le contenu de la page et la prochaine requête ;
- dépendance à un offset instable sous écritures concurrentes.

Si l’infrastructure lit une ligne supplémentaire uniquement pour déterminer `hasNextPage`, cette ligne supplémentaire ne doit pas devenir la position du curseur. Le curseur reste fondé sur la dernière ligne réellement incluse dans `items`.

## 6. Export public

Les helpers sont réexportés depuis :

```text
@mansa/contracts/ledger-api
```

Exemple :

```ts
import {
  createLedgerEntryNextCursor,
  validateLedgerEntryPageOrder,
} from '@mansa/contracts/ledger-api';
```

## 7. Séquence recommandée côté infrastructure

Pour construire une page :

1. valider la requête `ListLedgerEntriesQuery` ;
2. décoder et valider le curseur entrant lorsqu’il existe ;
3. vérifier son compte ;
4. exécuter la requête keyset avec `postedAt ASC, entryId ASC` ;
5. déterminer les éléments effectivement retournés ;
6. valider la forme et l’ordre de la page ;
7. si une page suivante existe, appeler `createLedgerEntryNextCursor(items)` ;
8. encoder et protéger ce payload dans la couche infrastructure ;
9. exposer la chaîne opaque dans `LedgerEntryPage.nextCursor`.

## 8. Tests ajoutés

Le lot ajoute :

```text
packages/contracts/test/ledger-entry-next-cursor.test.mjs
packages/contracts/test/ledger-api-entry-next-cursor-export.test.mjs
```

Les scénarios couvrent :

- création déterministe d’un curseur depuis une écriture ;
- utilisation de la dernière écriture d’une page ;
- absence de curseur inventé pour une page vide ;
- reconnaissance du code `EMPTY_PAGE` ;
- disponibilité des helpers depuis le point d’entrée public `ledger-api`.

## 9. Cohérence avec les lots précédents

Ce lot complète les contrats déjà présents pour :

- la validation du payload de curseur ;
- l’appartenance du curseur au compte demandé ;
- la comparaison keyset ;
- la validation d’une page ;
- la validation de l’ordre strict de ses écritures.

L’ensemble définit maintenant le comportement attendu autour de la pagination sans imposer encore une base de données ou un format cryptographique précis.

## 10. Suite recommandée

Le prochain lot doit porter cette logique dans une implémentation d’infrastructure testable :

- construire une requête keyset réelle ou un adaptateur de référence ;
- lire `limit + 1` éléments pour détecter proprement l’existence d’une page suivante ;
- retourner seulement `limit` éléments ;
- fabriquer le curseur depuis le dernier élément retourné ;
- ajouter des tests d’intégration sur au moins deux pages consécutives ;
- vérifier l’absence de doublon et d’omission lorsque plusieurs écritures partagent le même `postedAt`.
