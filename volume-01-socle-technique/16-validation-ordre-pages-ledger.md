# Validation de l’ordre des pages du grand livre

## 1. Objectif

Le grand livre possède désormais un ordre keyset canonique fondé sur `postedAt ASC, entryId ASC`. Ce lot ajoute un validateur explicite des pages retournées par l’API afin de détecter immédiatement toute rupture de cet ordre entre le contrat, l’adaptateur de persistance et les réponses exposées.

La route concernée reste :

```text
GET /v1/internal/ledger/accounts/:accountId/entries
```

## 2. Invariant de page

Pour toute page `LedgerEntryPage`, chaque écriture doit être strictement postérieure à la précédente selon l’ordre :

```text
postedAt ASC, entryId ASC
```

Deux entrées successives ne peuvent donc pas :

- revenir vers un timestamp antérieur ;
- inverser l’ordre des identifiants lorsque le timestamp est identique ;
- occuper exactement la même position keyset.

La validation d’identité, de timestamp, de taille de page et de curseur reste gérée par les validateurs déjà existants. Le nouveau validateur ne traite que l’ordre relatif des éléments.

## 3. Nouveau contrat

Le fichier :

```text
packages/contracts/src/ledger-entry-page-order.ts
```

introduit :

- `LEDGER_ENTRY_PAGE_ORDER_ERROR_CODES` ;
- `LedgerEntryPageOrderErrorCode` ;
- `LedgerEntryPageOrderError` ;
- `LedgerEntryPageOrderValidationResult` ;
- `validateLedgerEntryPageOrder` ;
- `isLedgerEntryPageOrderErrorCode`.

Le code d’erreur stable est :

```text
OUT_OF_ORDER
```

Chaque erreur fournit aussi `entryIndex`, qui désigne l’élément fautif dans la page.

## 4. Réutilisation de l’ordre canonique

Le validateur n’implémente pas un second algorithme de tri. Il réutilise directement :

```ts
compareLedgerEntryPositions
```

Cela évite qu’une définition de l’ordre soit utilisée dans les helpers de curseur et une autre dans les pages de réponse.

## 5. Séquence de validation recommandée

Une implémentation de `listEntries` doit appliquer au minimum la séquence suivante :

1. valider `ListLedgerEntriesQuery` ;
2. décoder et valider le curseur si présent ;
3. vérifier que le curseur appartient au compte demandé ;
4. exécuter la requête keyset avec l’ordre canonique ;
5. construire `LedgerEntryPage` ;
6. exécuter `validateLedgerEntryPage` ;
7. exécuter `validateLedgerEntryPageOrder` ;
8. retourner la page seulement si tous les invariants sont respectés.

## 6. Export public

Les nouveaux helpers sont réexportés depuis :

```text
@mansa/contracts/ledger-api
```

Exemple :

```ts
import {
  validateLedgerEntryPage,
  validateLedgerEntryPageOrder,
} from '@mansa/contracts/ledger-api';
```

## 7. Tests

Le lot ajoute :

- `packages/contracts/test/ledger-entry-page-order.test.mjs` ;
- `packages/contracts/test/ledger-api-entry-page-order-export.test.mjs`.

Les scénarios couvrent :

- une page correctement ordonnée ;
- une régression de timestamp ;
- une inversion d’identifiant à timestamp égal ;
- la reconnaissance du code `OUT_OF_ORDER` ;
- l’export depuis le point d’entrée `ledger-api`.

## 8. Intérêt opérationnel

Cette validation permet de détecter tôt plusieurs classes de défauts :

- `ORDER BY` incomplet dans une requête SQL/Prisma ;
- pagination offset utilisée par erreur à la place du keyset ;
- tri réalisé uniquement sur `postedAt` ;
- transformation applicative qui réordonne les résultats ;
- fusion incorrecte de plusieurs sources d’écritures.

Pour un grand livre financier, une page mal ordonnée peut créer des doublons ou des omissions entre deux requêtes successives. Ce contrôle fait donc partie des invariants de consultation, même s’il ne modifie aucun solde.

## 9. Suite recommandée

Le prochain lot doit rapprocher ces contrats du stockage réel :

- implémenter un adaptateur de pagination keyset au niveau infrastructure ;
- construire le `nextCursor` depuis la dernière écriture effectivement retournée ;
- ajouter des tests d’intégration sur deux pages ou plus ;
- couvrir plusieurs écritures avec le même `postedAt` ;
- vérifier l’absence de doublons et d’omissions entre pages consécutives ;
- intégrer ensuite la protection cryptographique du curseur avec secret injecté hors dépôt.
