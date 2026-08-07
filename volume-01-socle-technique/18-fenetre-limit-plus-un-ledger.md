# Fenêtre `limit + 1` du grand livre

## 1. Objectif

Ce lot formalise la technique utilisée par l’infrastructure pour déterminer de façon fiable si une page suivante d’écritures existe sans exposer une ligne supplémentaire au consommateur.

La route reste :

```text
GET /v1/internal/ledger/accounts/:accountId/entries
```

Le stockage peut lire jusqu’à `limit + 1` écritures dans l’ordre keyset canonique. Le contrat réduit ensuite cette fenêtre à `limit` éléments publics et utilise uniquement la présence de la ligne supplémentaire pour calculer `hasNextPage`.

## 2. Invariant principal

Pour une limite `N` :

- le stockage demande au maximum `N + 1` lignes ;
- les `N` premières lignes au maximum deviennent `items` ;
- la ligne `N + 1`, lorsqu’elle existe, sert uniquement à établir `hasNextPage = true` ;
- cette ligne supplémentaire n’est jamais retournée dans `items` ;
- le curseur suivant est construit depuis la dernière ligne effectivement retournée, jamais depuis la ligne supplémentaire.

Cette règle empêche les omissions lors du passage à la page suivante.

## 3. Nouveau helper

Le fichier :

```text
packages/contracts/src/ledger-entry-fetch-window.ts
```

introduit :

- `createLedgerEntryFetchWindow` ;
- `LEDGER_ENTRY_FETCH_WINDOW_ERROR_CODES` ;
- `LedgerEntryFetchWindowErrorCode` ;
- `LedgerEntryFetchWindowError` ;
- `LedgerEntryFetchWindowResult` ;
- `isLedgerEntryFetchWindowErrorCode`.

Le résultat expose :

- `items` : uniquement les écritures réellement visibles ;
- `hasNextPage` : indique si la lecture contenait une ligne supplémentaire ;
- `nextCursor` : payload stable construit depuis le dernier élément visible lorsque `hasNextPage` vaut `true` ;
- `errors` : erreurs de contrat.

## 4. Validation de la limite

La limite doit être un entier compris entre `1` et `200` inclus.

Le code stable suivant est défini :

```text
INVALID_LIMIT
```

Une limite invalide produit une fenêtre vide, sans curseur et sans page suivante.

## 5. Pages terminales

Deux situations ne produisent pas de curseur suivant :

1. aucune écriture n’est retournée ;
2. le nombre d’écritures lues est inférieur ou égal à `limit`.

Dans ces deux cas :

```text
hasNextPage = false
nextCursor = undefined
```

Une page vide est une page terminale valide et n’est pas considérée comme une erreur.

## 6. Cohérence avec le curseur keyset

Lorsque `hasNextPage = true`, le helper réutilise `createLedgerEntryCursorFromEntry`.

Le payload reprend donc :

```text
version
accountId
postedAt
entryId
```

à partir de la dernière écriture contenue dans `items`.

La ligne supplémentaire ne doit jamais devenir la position du curseur, sinon la première écriture de la page suivante serait sautée.

## 7. Séquence recommandée côté stockage

1. valider `ListLedgerEntriesQuery` ;
2. normaliser `limit` ;
3. décoder et valider le curseur entrant lorsqu’il existe ;
4. vérifier que le curseur appartient au compte demandé ;
5. exécuter la requête keyset en ordre `postedAt ASC, entryId ASC` ;
6. demander au maximum `limit + 1` lignes ;
7. transmettre ces lignes à `createLedgerEntryFetchWindow` ;
8. valider l’ordre et le contenu des `items` retournés ;
9. encoder le payload `nextCursor` dans la couche infrastructure lorsqu’il existe ;
10. retourner la page publique.

## 8. Tests

Le lot ajoute :

```text
packages/contracts/test/ledger-entry-fetch-window.test.mjs
```

Les scénarios couvrent :

- lecture de trois lignes pour une limite de deux ;
- exclusion correcte de la ligne supplémentaire ;
- construction du curseur depuis la deuxième ligne, donc la dernière ligne visible ;
- absence de curseur lorsque le nombre de lignes est exactement égal à la limite ;
- page terminale vide ;
- limites `0`, `201` et non entières ;
- reconnaissance du code `INVALID_LIMIT`.

## 9. Sécurité et séparation des responsabilités

Ce helper ne réalise aucun encodage opaque et ne manipule aucun secret.

La signature, le chiffrement éventuel, la rotation de clés et la représentation transport du curseur restent dans la couche infrastructure. Aucun secret ne doit être ajouté au dépôt.

## 10. Suite recommandée

Le prochain lot doit brancher cette fenêtre sur un adaptateur de persistance de référence et couvrir plusieurs pages successives, notamment le cas où plusieurs écritures partagent le même `postedAt` afin de démontrer l’absence de doublon et d’omission grâce au couple `(postedAt, entryId)`.
