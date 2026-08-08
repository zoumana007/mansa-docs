# Adaptateur de référence pour la pagination du grand livre

## 1. Objectif

Ce lot fournit une implémentation de référence du flux de persistance `listEntries` afin de verrouiller les invariants de pagination du grand livre avant l’intégration PostgreSQL/Prisma.

L’objectif n’est pas de remplacer un adaptateur de base de données de production. L’implémentation en mémoire sert de spécification exécutable pour les futurs adaptateurs de persistance.

## 2. Fichiers ajoutés ou modifiés

Dans `mansa-platform` :

```text
packages/contracts/src/ledger-entry-reference-adapter.ts
packages/contracts/src/ledger-api.ts
packages/contracts/test/ledger-entry-reference-adapter.test.mjs
```

Le nouvel adaptateur est réexporté depuis `ledger-api.ts`, de sorte que les consommateurs passent toujours par l’API publique du module ledger.

## 3. Sémantique de pagination imposée

L’adaptateur de référence applique les règles suivantes :

1. validation de `ListLedgerEntriesQuery` ;
2. filtrage strict par `accountId` ;
3. bornes temporelles `from` et `to` inclusives lorsqu’elles sont fournies ;
4. décodage et validation du curseur opaque ;
5. rejet d’un curseur appartenant à un autre compte ;
6. ordre canonique `(postedAt ASC, entryId ASC)` ;
7. sélection strictement après le curseur courant ;
8. lecture logique de `limit + 1` lignes ;
9. retour de `limit` éléments maximum ;
10. génération du prochain curseur depuis la dernière écriture réellement retournée.

Cette combinaison évite qu’une page soit définie uniquement par `postedAt`, ce qui serait ambigu lorsque plusieurs écritures partagent le même horodatage.

## 4. Codec de curseur

Le contrat introduit `LedgerEntryCursorCodec` :

```text
encode(cursor) -> string
decode(value) -> LedgerEntryCursor | undefined
```

Le package de contrats ne choisit volontairement aucun format de sérialisation de production. L’encodage, la signature et éventuellement le chiffrement du curseur restent des responsabilités d’infrastructure.

Le test de référence utilise un encodage Base64URL de JSON uniquement pour démontrer le flux. Ce choix ne constitue pas une recommandation de sécurité pour la production.

## 5. Erreurs stables de l’adaptateur

L’adaptateur expose trois codes d’erreur :

- `INVALID_QUERY` ;
- `INVALID_CURSOR` ;
- `CURSOR_ACCOUNT_MISMATCH`.

Ils permettent à une couche API d’établir une traduction stable vers les erreurs HTTP internes sans dépendre du détail d’implémentation du stockage.

## 6. Test multi-pages

Le test `ledger-entry-reference-adapter.test.mjs` construit volontairement plusieurs écritures partageant exactement le même `postedAt` et les fournit dans un ordre non trié.

Avec une taille de page de 2 :

- page 1 retourne `entry-1`, `entry-2` ;
- le curseur est construit depuis `entry-2` ;
- page 2 retourne `entry-3`, `entry-4` ;
- aucun identifiant n’est dupliqué ;
- aucun identifiant n’est omis ;
- la dernière page ne produit pas de curseur suivant.

Un second test vérifie qu’un curseur émis pour un autre compte est rejeté.

## 7. Traduction attendue vers PostgreSQL/Prisma

Le futur adaptateur SQL doit conserver exactement la même sémantique. Conceptuellement, après validation du curseur :

```sql
WHERE account_id = :accountId
  AND (:from IS NULL OR posted_at >= :from)
  AND (:to IS NULL OR posted_at <= :to)
  AND (
    :cursorPostedAt IS NULL
    OR posted_at > :cursorPostedAt
    OR (posted_at = :cursorPostedAt AND id > :cursorEntryId)
  )
ORDER BY posted_at ASC, id ASC
LIMIT :limitPlusOne
```

L’index de production doit être conçu pour cet accès, typiquement autour de :

```text
(account_id, posted_at, id)
```

Le schéma exact et les choix d’indexation restent à valider avec le modèle Prisma final et les volumes réels.

## 8. Validation de ce lot

Les changements ont été écrits directement sur `main` dans les deux dépôts concernés par ce lot.

Le dépôt GitHub ne remonte actuellement aucun statut CI ni workflow associé au dernier commit. La présence des tests protège le comportement lors de l’exécution de `pnpm --filter @mansa/contracts test`, mais l’automatisation distante n’a pas fourni de résultat d’exécution à ce stade.

Aucun secret, identifiant bancaire ou paramètre sensible n’a été ajouté.

## 9. Suite recommandée

Le prochain lot doit brancher cette sémantique sur un vrai port de persistance ledger puis sur l’adaptateur Prisma/PostgreSQL, avec :

- un contrat explicite de repository ;
- un index keyset adapté ;
- des tests d’intégration sur plusieurs pages ;
- un cas où de nouvelles écritures sont insérées entre deux lectures ;
- une validation que le curseur reste stable et n’entraîne ni doublon ni omission pour les données déjà ordonnées.
