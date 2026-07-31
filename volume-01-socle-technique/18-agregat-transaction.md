# Agrégat Transaction

## Objectif

L’agrégat `Transaction` regroupe les invariants minimaux communs aux paiements, transferts, cash-in, cash-out et remboursements. Il fournit un objet métier indépendant de NestJS, Prisma, HTTP et des partenaires externes.

## Données canoniques

Chaque transaction contient :

- une référence publique Mansa valide ;
- un type d’opération ;
- un montant strictement positif représenté par `Money` ;
- un état du cycle de vie canonique ;
- une date UTC de création ;
- une date UTC de dernière modification.

Les types initiaux sont `PAYMENT`, `TRANSFER`, `CASH_IN`, `CASH_OUT` et `REFUND`.

## Création

Une nouvelle transaction :

1. valide et normalise sa référence publique ;
2. refuse un montant nul ou négatif ;
3. démarre obligatoirement dans l’état `PENDING` ;
4. initialise `createdAt` et `updatedAt` avec la même date ;
5. ne produit encore aucune écriture comptable tant qu’un cas d’usage applicatif ne l’ordonne pas.

## Restauration

La restauration depuis la persistance valide les mêmes invariants que la création. Elle refuse notamment une date `updatedAt` antérieure à `createdAt`. Les objets `Date` sont copiés afin d’empêcher une mutation externe de l’état interne.

## Transitions

La méthode `transition` délègue la validation à la machine d’états documentée dans `17-cycle-vie-transactions.md`. Elle refuse :

- toute transition interdite ;
- toute date de transition antérieure à la dernière modification ;
- toute modification directe de l’état sans passage par le domaine.

L’agrégat ne persiste pas lui-même l’historique. Le cas d’usage applicatif devra enregistrer atomiquement l’état courant, l’événement de transition et les écritures comptables associées.

## API de référence

Le package `@mansa/domain` expose :

- `Transaction.create` pour créer une nouvelle opération ;
- `Transaction.restore` pour reconstruire une opération persistée ;
- `Transaction.transition` pour appliquer une transition valide ;
- `Transaction.current` pour obtenir un instantané défensif ;
- `InvalidTransactionAmountError` pour les montants non positifs ;
- `TransactionKind` et `TransactionSnapshot` pour les contrats TypeScript.

## Critères d’acceptation

- Une transaction valide est créée en `PENDING`.
- Un montant nul ou négatif est refusé.
- Une référence invalide est refusée par le composant de référence publique.
- Les transitions autorisées modifient l’état et `updatedAt`.
- Une transition invalide ne modifie pas l’agrégat.
- Une date antérieure à `updatedAt` est refusée.
- La restauration normalise la référence et protège les dates internes.
- Les tests du package domaine couvrent création, montant, transitions, temps et restauration.

## Cohérence avec le code

L’implémentation de référence se trouve dans :

- `packages/domain/src/transaction.ts` ;
- `packages/domain/src/transaction-state.ts` ;
- `packages/domain/src/transaction-reference.ts` ;
- `packages/domain/src/money.ts` ;
- `packages/domain/test/transaction.test.mjs`.

## Étapes suivantes

- Définir les événements métier produits par l’agrégat.
- Ajouter un dépôt applicatif et une unité de travail transactionnelle.
- Relier les transitions réussies au grand livre double entrée.
- Persister l’historique append-only des changements d’état.
- Ajouter les métadonnées de canal, initiateur, bénéficiaire et partenaire sans exposer de données sensibles inutiles.
