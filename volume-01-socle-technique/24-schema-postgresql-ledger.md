# 24 — Schéma PostgreSQL du Ledger

## Objectif

Ce lot matérialise le premier schéma PostgreSQL du grand livre Mansa. Il complète le repository de lecture keyset en fournissant les tables et index nécessaires à une persistance réelle des comptes, transactions et écritures comptables.

La migration de référence est :

`mansa-platform/infra/postgres/migrations/0001_ledger_core.sql`

## Tables créées

### `ledger_accounts`

Cette table conserve les comptes comptables de la plateforme.

Champs principaux :

- `id` : identifiant technique texte ;
- `code` : code comptable unique ;
- `owner_type` : `PLATFORM`, `USER`, `MERCHANT`, `PARTNER` ou `PUBLIC_BODY` ;
- `owner_id` : propriétaire lorsque le compte n’est pas un compte plateforme ;
- `account_type` : `ASSET`, `LIABILITY`, `EQUITY`, `REVENUE` ou `EXPENSE` ;
- `currency` : `XOF`, `EUR` ou `USD` dans le socle actuel ;
- `country_code` : code pays ISO alpha-2 ;
- `name` ;
- `is_system_account` ;
- `created_at`.

Une contrainte empêche la création d’un compte non-plateforme sans `owner_id`.

### `ledger_transactions`

Cette table porte l’enveloppe comptable de chaque mouvement.

Champs principaux :

- `id` ;
- `reference` ;
- `transaction_type` ;
- `status` ;
- `idempotency_key` unique ;
- `correlation_id` ;
- `country_code` ;
- `occurred_at` ;
- `posted_at` ;
- `reversed_by_transaction_id` ;
- `metadata` au format JSONB ;
- `created_at`.

L’unicité de `idempotency_key` fournit une première barrière de persistance contre la double publication d’une même commande.

### `ledger_entries`

Cette table contient les écritures débit/crédit immuables.

Champs :

- `id` ;
- `transaction_id` ;
- `sequence` ;
- `account_id` ;
- `direction` ;
- `amount_minor` ;
- `currency` ;
- `description` ;
- `posted_at` ;
- `created_at`.

`amount_minor` est un `BIGINT` strictement positif. Aucun type flottant n’est utilisé pour les montants.

## Contraintes d’intégrité

La migration impose notamment :

- direction limitée à `DEBIT` ou `CREDIT` ;
- montant strictement positif ;
- devise limitée au catalogue monétaire actuellement exposé par `Money` ;
- statut de transaction limité aux statuts du contrat Ledger ;
- unicité de `(transaction_id, sequence)` ;
- suppression en cascade interdite sur les écritures ;
- référence explicite des écritures vers leur transaction et leur compte ;
- `idempotency_key` unique.

Ces contraintes SQL ne remplacent pas la validation métier. L’équilibre débit/crédit reste vérifié dans le domaine avant publication et devra être garanti dans une transaction PostgreSQL atomique lors de l’implémentation de l’écriture persistée.

## Index keyset

Le repository `PostgresLedgerEntryRepository` lit les écritures dans l’ordre :

```text
posted_at ASC, id ASC
```

avec un filtre obligatoire sur `account_id`.

La migration crée donc :

```sql
CREATE INDEX IF NOT EXISTS ledger_entries_account_posted_id_idx
ON ledger_entries (account_id, posted_at ASC, id ASC);
```

Cet index est aligné avec le prédicat et l’ordre du repository de lecture. Il évite une pagination par offset et permet de reprendre efficacement après un curseur `(postedAt, entryId)`.

## Index secondaires

Sont également créés :

- `ledger_accounts_owner_idx` sur `(owner_type, owner_id)` ;
- `ledger_transactions_reference_idx` sur `reference` ;
- `ledger_transactions_correlation_idx` sur `correlation_id` ;
- `ledger_entries_transaction_idx` sur `(transaction_id, sequence)`.

## Stratégie de migration

La migration est enveloppée dans une transaction SQL explicite et utilise `IF NOT EXISTS` pour les objets de premier niveau.

Avant production, les migrations doivent être exécutées par un outil versionné et contrôlé dans le pipeline de déploiement. Les credentials de base ne doivent jamais être stockés dans Git.

Pour les très gros volumes, certains index pourront ensuite être créés avec une stratégie `CONCURRENTLY` séparée de la transaction principale afin de réduire les verrouillages de production.

## Hypothèses à valider

- Les identifiants sont stockés en `TEXT` pour rester compatibles avec les contrats actuels qui n’imposent pas encore UUID/ULID.
- Le catalogue monétaire initial est limité à `XOF`, `EUR` et `USD`, conformément au type `CurrencyCode` actuel.
- La séquence d’écriture est portée par `BIGINT` et doit être attribuée de manière déterministe par la future couche de publication.
- Une politique d’archivage/partitionnement des écritures pourra être ajoutée lorsque la volumétrie réelle sera connue.

## Sécurité

- aucun secret n’est introduit dans la migration ;
- les contraintes de référence empêchent la suppression silencieuse d’écritures historiques ;
- les montants négatifs ou nuls sont rejetés par PostgreSQL ;
- les statuts, directions et devises hors contrat sont rejetés ;
- l’idempotence dispose d’une contrainte unique au niveau base de données.

## Validation restante

Le schéma doit encore être exécuté contre une instance PostgreSQL réelle dans un test d’intégration automatisé. Ce test devra :

1. appliquer la migration sur une base vide ;
2. insérer un compte et une transaction valides ;
3. insérer plusieurs écritures ;
4. exécuter `PostgresLedgerEntryRepository` contre cette base ;
5. vérifier l’ordre keyset et les filtres ;
6. vérifier le rejet des contraintes critiques ;
7. détruire la base de test.

## Étape suivante

Le prochain lot doit ajouter un environnement PostgreSQL de test reproductible et un test d’intégration du repository réel. Ensuite, la couche de publication devra écrire transaction + écritures de manière atomique, avec contrôle d’idempotence, verrouillage approprié et outbox transactionnelle.
