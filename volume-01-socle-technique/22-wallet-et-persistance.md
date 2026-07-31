# Wallet et port de persistance

## Objectif

Le wallet représente les fonds immédiatement disponibles d’un propriétaire dans une devise donnée. Il constitue un agrégat métier distinct du grand livre comptable : il expose une vue opérationnelle du solde disponible tandis que le grand livre reste la source comptable détaillée et auditable.

## Identité et propriétés

Un wallet contient :

- `id`, identifiant technique unique ;
- `ownerId`, identifiant du client, commerçant ou autre propriétaire ;
- `currency`, devise unique du wallet ;
- `availableBalance`, montant immédiatement utilisable ;
- `status`, état du cycle de vie ;
- `createdAt` et `updatedAt`.

Un propriétaire peut posséder plusieurs wallets, notamment un par devise ou par produit autorisé.

## Cycle de vie

Les états supportés sont :

- `ACTIVE` : crédits et débits autorisés ;
- `SUSPENDED` : opérations financières refusées, consultation autorisée ;
- `CLOSED` : état terminal.

La fermeture exige un solde nul. Un wallet fermé ne peut être ni réactivé ni suspendu.

## Invariants financiers

- le solde ne peut jamais devenir négatif ;
- un crédit ou un débit doit être strictement positif ;
- la devise de l’opération doit correspondre à celle du wallet ;
- les opérations financières sont refusées lorsque le wallet n’est pas actif ;
- la date de mise à jour ne peut pas revenir en arrière ;
- aucun montant n’est représenté en nombre flottant.

## Port de persistance

Le contrat `WalletRepository` isole le domaine de Prisma, PostgreSQL et de tout autre stockage. Il expose :

- `findById(id)` ;
- `findByOwnerId(ownerId)` ;
- `search(criteria)` ;
- `save(wallet)`.

Les critères de recherche couvrent le propriétaire, la devise et le statut. Les adaptateurs de production devront ajouter pagination, indexation, contrôle de version et gestion explicite de la concurrence.

## Implémentation en mémoire

`InMemoryWalletRepository` sert de double de test et d’adaptateur local minimal. Il ne doit pas être utilisé en production car il ne fournit ni durabilité, ni isolation transactionnelle, ni verrouillage optimiste, ni protection contre plusieurs instances.

## Cohérence avec le code

L’implémentation de référence se trouve dans :

- `packages/domain/src/wallet.ts` ;
- `packages/domain/src/wallet-repository.ts` ;
- `packages/domain/src/index.ts` ;
- `packages/domain/test/wallet.test.mjs` ;
- `packages/domain/test/wallet-repository.test.mjs`.

## Exigences de l’adaptateur Prisma

L’adaptateur de production devra :

1. persister les montants en unités mineures entières ;
2. imposer l’unicité de l’identifiant ;
3. indexer `ownerId`, `currency` et `status` ;
4. utiliser un champ de version pour le verrouillage optimiste ;
5. reconstruire un nouvel agrégat à chaque lecture ;
6. refuser une mise à jour fondée sur une version obsolète ;
7. participer à la même transaction de base que les écritures du grand livre ;
8. ne jamais journaliser de données personnelles ou de secrets.

## Critères d’acceptation

- un wallet est créé actif avec un solde nul par défaut ;
- les identifiants vides sont refusés ;
- les devises incompatibles sont refusées ;
- un débit supérieur au solde disponible échoue ;
- un wallet suspendu ne peut pas être crédité ou débité ;
- un wallet non nul ne peut pas être fermé ;
- la recherche par propriétaire, devise et statut est couverte par des tests ;
- le package domaine n’importe aucune technologie de persistance.

## Étapes suivantes

- définir les événements de cycle de vie du wallet ;
- relier les mouvements du wallet aux écritures du grand livre ;
- créer l’adaptateur Prisma avec verrouillage optimiste ;
- ajouter pagination et recherche administrative ;
- définir les règles multi-wallet et wallet principal ;
- ajouter les limites, réservations et soldes en attente.
