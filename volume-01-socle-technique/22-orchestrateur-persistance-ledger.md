# 22 — Orchestrateur de persistance Ledger

## Objectif

Ce lot relie le contrat HTTP de consultation des écritures Ledger au port de persistance défini dans le lot précédent, sans faire fuiter Prisma ou PostgreSQL dans les contrats métier.

L’orchestrateur de référence est implémenté dans `packages/contracts/src/ledger-entry-repository-adapter.ts` du dépôt `mansa-platform`.

## Responsabilités

L’orchestrateur doit, dans cet ordre :

1. valider la requête publique `ListLedgerEntriesQuery` ;
2. décoder le curseur opaque lorsqu’il est présent ;
3. valider la structure et la version du curseur ;
4. vérifier que le curseur appartient bien au compte demandé ;
5. convertir le curseur public en position keyset `{ postedAt, entryId }` ;
6. construire une `LedgerEntryRepositoryQuery` indépendante du stockage ;
7. demander `limit + 1` lignes au repository ;
8. transformer les lignes reçues en fenêtre publique ;
9. générer le prochain curseur uniquement lorsqu’une page suivante existe ;
10. convertir les erreurs d’infrastructure en codes stables côté application.

## Contrat de dépendances

L’adaptateur dépend uniquement de :

- `LedgerEntryRepository`, port de lecture des écritures ;
- `LedgerEntryCursorCodec`, responsable de l’encodage/décodage opaque du curseur.

Le codec peut ensuite être implémenté avec une représentation signée ou chiffrée sans changer l’orchestrateur.

## Construction de la requête repository

Pour une requête publique :

```text
accountId = account-1
limit = 50
cursor = opaque(...)
```

le repository reçoit une requête de cette forme :

```text
accountId = account-1
after = { postedAt, entryId }
take = 51
```

Les bornes `from` et `to` restent optionnelles et sont transmises telles quelles après validation.

## Pourquoi `limit + 1`

Le stockage retourne au maximum une ligne supplémentaire. Cette ligne n’est pas exposée au client ; elle sert uniquement à déterminer si une page suivante existe.

Cette stratégie évite un `COUNT(*)` supplémentaire et conserve une pagination déterministe même lorsque le Ledger devient volumineux.

## Codes d’erreur stables

L’adaptateur expose les catégories suivantes :

- `INVALID_QUERY` : requête publique invalide ;
- `INVALID_CURSOR` : curseur illisible ou structure invalide ;
- `CURSOR_ACCOUNT_MISMATCH` : tentative de réutiliser un curseur sur un autre compte ;
- `INVALID_REPOSITORY_QUERY` : garde-fou si la transformation vers le port de persistance produit une requête non valide ;
- `REPOSITORY_FAILURE` : échec d’accès au stockage ;
- `INVALID_FETCH_WINDOW` : résultat du repository incompatible avec la fenêtre de pagination attendue.

Aucun message brut de base de données ne doit être remonté au consommateur de l’API.

## Sécurité

Le contrôle `CURSOR_ACCOUNT_MISMATCH` doit avoir lieu avant tout accès au repository. Cela empêche un curseur provenant d’un autre compte d’être utilisé comme primitive de navigation ou de découverte de données.

Le curseur reste opaque côté client. Son contenu décodé n’est jamais accepté sans validation.

## Tests ajoutés

Le dépôt `mansa-platform` contient `packages/contracts/test/ledger-entry-repository-adapter.test.mjs` couvrant notamment :

- la transformation du curseur en position keyset ;
- la requête `limit + 1` ;
- la génération du prochain curseur ;
- le rejet d’un curseur lié à un autre compte avant l’accès persistence ;
- la conversion d’une exception du repository en `REPOSITORY_FAILURE`.

## Étape suivante

Le prochain lot doit fournir l’adaptateur concret PostgreSQL/Prisma implémentant `LedgerEntryRepository`, avec requête keyset indexable selon l’ordre canonique `(accountId, postedAt, id)` et tests d’intégration contre PostgreSQL.
