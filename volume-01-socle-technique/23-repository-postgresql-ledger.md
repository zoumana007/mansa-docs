# 23 — Repository PostgreSQL Ledger

## Objectif

Ce lot fournit l’implémentation PostgreSQL concrète du port `LedgerEntryRepository` introduit précédemment. L’objectif est d’obtenir une lecture persistée, déterministe et indexable des écritures Ledger sans faire dépendre le domaine d’un ORM particulier.

L’implémentation de référence se trouve dans `packages/contracts/src/ledger-entry-postgres-repository.ts` du dépôt `mansa-platform`.

## Choix d’architecture

Le dépôt ne dépend pas encore de `@prisma/client` ni de `pg`. Pour ne pas introduire une dépendance infrastructure prématurée, le repository dépend d’une surface minimale injectable :

```ts
interface PostgresLedgerEntryClient {
  query<Row>(text: string, values: readonly unknown[]): Promise<{ rows: readonly Row[] }>;
}
```

Cette interface peut être adaptée à `pg.Pool`, à une couche Prisma utilisant une requête SQL typée, ou à un autre client PostgreSQL sans changer le contrat métier `LedgerEntryRepository`.

## Requête keyset canonique

Le repository applique l’ordre :

```text
posted_at ASC, id ASC
```

et filtre toujours par :

```text
account_id = ?
```

Lorsqu’un curseur est présent, le prédicat keyset est :

```sql
posted_at > :postedAt
OR (posted_at = :postedAt AND id > :entryId)
```

Cette règle doit rester identique à la construction et à la validation des curseurs publics.

## Paramétrage SQL

Aucune valeur fournie par l’utilisateur n’est concaténée dans la chaîne SQL.

Le générateur `buildLedgerEntryPostgresStatement` produit uniquement des placeholders PostgreSQL `$1`, `$2`, etc., et place les données dans le tableau `values`.

Cette propriété est testée avec un identifiant de compte contenant une tentative d’injection SQL : la valeur reste exclusivement dans les paramètres et n’apparaît jamais dans le texte SQL.

## Filtres supportés

Le repository accepte :

- `accountId` obligatoire ;
- `from` optionnel avec `posted_at >= ...` ;
- `to` optionnel avec `posted_at <= ...` ;
- `after` optionnel pour la pagination keyset ;
- `take` obligatoire pour limiter strictement le nombre de lignes retournées.

La validation métier de ces paramètres reste dans `validateLedgerEntryRepositoryQuery` avant l’accès au stockage.

## Mapping PostgreSQL vers domaine

Les colonnes infrastructure sont converties vers `LedgerEntry` :

- `transaction_id` → `transactionId` ;
- `account_id` → `accountId` ;
- `amount_minor` → `Money.amountMinor` en `bigint` ;
- `posted_at` → date ISO ;
- `description = NULL` → propriété absente ;
- `direction` doit être `DEBIT` ou `CREDIT` ;
- `currency` doit respecter un code à trois lettres majuscules.

Une ligne PostgreSQL invalide provoque une erreur d’infrastructure au lieu de laisser entrer une donnée incohérente dans le domaine. L’orchestrateur supérieur la convertit ensuite en `REPOSITORY_FAILURE`.

## Index PostgreSQL requis

Avant une mise en production, la table `ledger_entries` doit disposer au minimum d’un index compatible avec le filtre et l’ordre de pagination :

```sql
CREATE INDEX CONCURRENTLY IF NOT EXISTS ledger_entries_account_posted_id_idx
ON ledger_entries (account_id, posted_at ASC, id ASC);
```

L’index doit être créé via une migration contrôlée. `CONCURRENTLY` nécessite une stratégie de migration adaptée car cette commande ne peut pas être exécutée dans toutes les transactions de migration classiques.

## Schéma de colonnes attendu

Le repository suppose les colonnes suivantes :

```text
id
transaction_id
sequence
account_id
direction
amount_minor
currency
description
posted_at
```

`amount_minor` doit être stocké dans un type entier suffisamment large (`BIGINT` recommandé). Aucun montant monétaire ne doit être stocké en flottant.

## Tests ajoutés

`packages/contracts/test/ledger-entry-postgres-repository.test.mjs` couvre :

- la génération du SQL keyset paramétré ;
- l’absence d’interpolation d’une valeur utilisateur dans le SQL ;
- les filtres temporels ;
- l’ordre canonique `posted_at ASC, id ASC` ;
- la limite paramétrée ;
- le mapping d’une ligne PostgreSQL vers `LedgerEntry` ;
- la conversion de `amount_minor` vers `bigint` ;
- le rejet d’une direction infrastructure invalide.

## Sécurité et conformité

- aucun secret de base de données n’est présent dans le code ;
- les credentials PostgreSQL devront être injectés par variables d’environnement ou gestionnaire de secrets ;
- le repository ne retourne que les écritures du `accountId` explicitement demandé ;
- le curseur est déjà validé et lié au compte par l’orchestrateur avant l’appel ;
- les erreurs SQL brutes ne doivent jamais être exposées à l’API publique.

## Étape suivante

Le prochain lot doit introduire le schéma/migration PostgreSQL de `ledger_entries` et l’index `(account_id, posted_at, id)`, puis ajouter un test d’intégration réel contre PostgreSQL. Une fois la couche Prisma effectivement ajoutée au monorepo, un adaptateur Prisma pourra implémenter la même interface sans modifier les contrats Ledger.
