# Exécution wallet des transferts

## Objectif

Le lot relie la commande de transfert aux agrégats `Wallet` sans introduire de dépendance vers une base de données particulière. La fabrique `createWalletTransferExecutor` produit un `TransferExecutor` compatible avec `TransferService`.

## Déroulement

1. Charger en parallèle le wallet source et le wallet destinataire.
2. Refuser l’opération lorsqu’un wallet n’existe pas.
3. Vérifier que la devise du montant, du wallet source et du wallet destinataire est identique.
4. Débiter le wallet source avec les contrôles d’état et de solde de l’agrégat.
5. Créditer le wallet destinataire à la même date métier.
6. Persister les deux agrégats dans l’unité de travail courante.
7. Retourner un identifiant de transaction non vide fourni par l’infrastructure.

## Erreurs métier

- `TransferWalletNotFoundError` identifie précisément le wallet absent.
- `TransferCurrencyMismatchError` expose les trois devises incompatibles pour permettre une journalisation interne structurée.
- Les erreurs de wallet existantes restent responsables des wallets suspendus, fermés ou insuffisamment provisionnés.

La couche API devra traduire ces erreurs vers des codes publics stables sans publier les détails internes sensibles.

## Atomicité exigée

L’exécuteur ne constitue pas à lui seul une transaction de base de données. L’adaptateur de production doit lui fournir un `WalletRepository` limité à l’unité de travail courante et garantir :

- verrouillage des deux wallets dans un ordre déterministe ;
- contrôle de concurrence optimiste ou pessimiste ;
- persistance des deux soldes ;
- création de la transaction comptable équilibrée ;
- enregistrement du résultat idempotent ;
- écriture des événements outbox ;
- validation ou annulation de l’ensemble en une seule transaction.

## Cohérence avec le code

Le paquet `@mansa/domain` exporte désormais :

- `createWalletTransferExecutor` ;
- `WalletTransferExecutorDependencies` ;
- `TransferWalletNotFoundError` ;
- `TransferCurrencyMismatchError`.

Le code ne contient aucun secret ni paramètre de production. La génération d’identifiants, l’horloge et la persistance restent injectées.

## Étape suivante

Le prochain lot doit ajouter les tests unitaires de l’exécuteur, puis construire l’unité de travail transactionnelle qui associe wallets, ledger, résultat de transfert et outbox.
