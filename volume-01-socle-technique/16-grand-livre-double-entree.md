# Grand livre comptable en partie double

## Objectif

Le grand livre Mansa constitue la source de vérité financière interne. Toute opération qui déplace de la valeur produit une écriture comptable immuable et équilibrée avant d’être considérée comme comptabilisée.

La primitive actuelle est `JournalEntry`, exposée par `@mansa/domain` depuis `packages/domain/src/ledger.ts`.

## Principes obligatoires

- Une écriture contient au minimum deux lignes.
- Chaque ligne est un débit ou un crédit.
- Le total des débits est strictement égal au total des crédits.
- Une écriture ne mélange jamais plusieurs devises.
- Les montants sont stockés en unités mineures entières avec `bigint`.
- Chaque montant de ligne est strictement positif.
- Une écriture validée est immuable.
- Une correction comptable se fait par contre-passation, jamais par modification destructive.
- La clé d’idempotence empêche la comptabilisation répétée d’une même opération.

## Structure canonique

Une écriture comprend :

- `id` : identifiant unique de l’écriture ;
- `transactionId` : opération métier à l’origine de l’écriture ;
- `idempotencyKey` : clé stable de déduplication ;
- `createdAt` : date de création ;
- `currency` : devise commune normalisée ;
- `lines` : lignes comptables gelées après validation.

Chaque ligne contient :

- `accountId` : compte comptable concerné ;
- `side` : `DEBIT` ou `CREDIT` ;
- `amountMinor` : montant positif en unité mineure ;
- `currency` : code ISO 4217 sur trois lettres.

## Exemple de paiement commerçant

Pour un paiement de 10 000 XOF sans frais :

```text
Débit   wallet:user-1       10 000 XOF
Crédit  wallet:merchant-1   10 000 XOF
```

Avec des frais de 350 XOF supportés par le commerçant, l’écriture de règlement pourra être décomposée ainsi :

```text
Débit   wallet:user-1       10 000 XOF
Crédit  wallet:merchant-1    9 650 XOF
Crédit  revenue:fees           350 XOF
```

Le choix exact des comptes dépend du plan comptable validé. Le moteur de grand livre ne doit pas inventer cette ventilation : elle provient d’une règle métier versionnée.

## Idempotence

Une même `idempotencyKey` ne doit produire qu’une seule écriture persistée. Cette garantie doit être renforcée par une contrainte unique en base de données et par une transaction atomique.

Exemples de clés :

```text
payment:<paymentId>:capture
transfer:<transferId>:settlement
refund:<refundId>:posting
```

## Cycle d’une opération

1. Le service métier valide la commande et les permissions.
2. Il calcule les montants, frais et taxes avec les primitives canoniques.
3. Il construit une écriture équilibrée.
4. Le dépôt comptable vérifie l’idempotence.
5. L’écriture et les changements de statut sont persistés dans une transaction atomique.
6. Un événement de domaine est publié après confirmation de la transaction.
7. Les soldes de lecture sont mis à jour ou recalculables depuis le grand livre.

## Soldes

Le solde affiché à un utilisateur est une projection. Le grand livre reste la source de vérité. Les soldes disponibles, comptables, réservés et en attente doivent être distingués.

Une projection de solde doit pouvoir être reconstruite à partir des écritures persistées. Les caches ou tables de synthèse ne doivent jamais devenir l’unique preuve financière.

## Sécurité et audit

- Les écritures comptables ne sont jamais supprimées.
- Les accès en écriture sont réservés aux services backend autorisés.
- Toute action administrative sensible est journalisée.
- Les exports doivent inclure les identifiants d’écriture, de transaction et de corrélation.
- Les horodatages sont conservés en UTC.
- Les comptes techniques, revenus, frais, suspens et règlement sont séparés des wallets utilisateurs.

## Hypothèse à valider

Le plan comptable détaillé, les règles de cantonnement, les comptes de règlement bancaire et les traitements réglementaires doivent être validés avec la banque partenaire, les commissaires aux comptes et les autorités compétentes avant mise en production.

## Critères d’acceptation

- Une écriture équilibrée et mono-devise est acceptée.
- Une écriture avec moins de deux lignes est rejetée.
- Un identifiant, une transaction ou une clé d’idempotence vide est rejeté.
- Une ligne sans compte est rejetée.
- Un montant nul ou négatif est rejeté.
- Une valeur de côté autre que `DEBIT` ou `CREDIT` est rejetée à l’exécution.
- Une écriture déséquilibrée est rejetée.
- Un mélange de devises est rejeté.
- La devise est normalisée en majuscules.
- La liste de lignes et la date exposées par l’objet ne permettent pas de muter l’état interne.
- Les tests automatisés couvrent les invariants précédents.
