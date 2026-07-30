# Volume 1 — Modèle de données initial

## 1. Objectif

Le modèle initial fournit les fondations nécessaires à l’identité, aux portefeuilles et au grand livre. Il reste volontairement limité afin d’éviter de figer trop tôt les modules cartes, commerçants, services publics et partenaires.

## 2. Principes

- PostgreSQL est la source transactionnelle de référence.
- Les identifiants sont des UUID générés côté base ou application.
- Tous les montants sont stockés en unités mineures avec un entier 64 bits.
- La devise est enregistrée sur chaque compte et écriture.
- Une transaction de grand livre est immuable après validation.
- Une écriture ne peut pas changer de montant ou de compte après publication.
- Les suppressions physiques d’éléments financiers sont interdites.
- Les dates sont stockées en UTC.

## 3. Entités du premier socle

### User

Représente une identité humaine ou technique. L’état du compte permet de suspendre les accès sans supprimer l’historique.

### Wallet

Regroupe les comptes financiers d’un utilisateur. Un utilisateur peut disposer de plusieurs portefeuilles selon le pays, l’usage ou le produit.

### LedgerAccount

Compte comptable portant une devise et une nature : actif, passif, produit, charge ou capitaux propres.

### LedgerTransaction

En-tête d’une opération comptable. Il contient une référence unique, un statut et une clé d’idempotence optionnelle mais unique lorsqu’elle est fournie.

### LedgerEntry

Ligne débit ou crédit liée à un compte. La somme des débits doit égaler la somme des crédits pour chaque transaction et chaque devise.

## 4. Contraintes obligatoires

- `User.email` et `User.phone` sont uniques lorsqu’ils sont présents.
- La référence d’une transaction est unique.
- La clé d’idempotence est unique.
- Le montant d’une écriture est strictement positif.
- Une transaction publiée contient au minimum deux écritures.
- Les écritures d’une transaction utilisent une seule devise dans le premier socle.
- La validation de l’équilibre est effectuée dans le service métier avant commit atomique.

## 5. Évolution prévue

Les prochains lots ajouteront : KYC, bénéficiaires, intentions de paiement, transferts, commerçants, terminaux, cartes, frais, limites, webhooks, rapprochements, services publics et journaux d’audit.

Chaque ajout doit être accompagné d’une migration, d’une règle métier documentée et d’un test de non-régression.
