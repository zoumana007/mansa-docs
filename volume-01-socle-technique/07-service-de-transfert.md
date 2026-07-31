# Service de transfert

## Objectif

`TransferService` orchestre l’exécution idempotente d’une `TransferCommand`. Il sépare la décision métier de l’implémentation de persistance et de la mutation atomique du grand livre.

## Dépendances

Le service reçoit :

- un `TransferRepository` pour rechercher un résultat par `transferId` ou `idempotencyKey` et enregistrer un transfert terminé ;
- une fonction `executeAtomically` qui réalise la mutation financière et renvoie le `transactionId` ;
- une horloge injectable pour produire des tests déterministes.

## Algorithme

1. Rechercher en parallèle un résultat existant par identifiant de transfert et par clé d’idempotence.
2. Lorsqu’un résultat cohérent existe, renvoyer un nouveau `TransferResult` avec le statut `REPLAYED` sans toucher au ledger.
3. Lorsqu’une clé ou un identifiant pointe vers un autre transfert, lever `TransferIdentityConflictError`.
4. En l’absence de résultat, appeler l’exécuteur atomique.
5. Produire un résultat `COMPLETED` avec la référence de transaction obtenue.
6. Enregistrer la commande et le résultat via le dépôt.

## Invariants de production

L’implémentation d’infrastructure doit garantir que la mutation du ledger, l’écriture du transfert terminé et les événements outbox sont validés dans une seule transaction de base de données. Les contraintes uniques sur `transferId` et `idempotencyKey` constituent la dernière protection contre deux requêtes concurrentes.

Le service de domaine effectue une détection précoce et déterministe des rejouements, mais ne remplace pas les verrous, contraintes et niveaux d’isolation de la base.

## Gestion des conflits

Un conflit est signalé lorsque :

- la clé d’idempotence est déjà associée à un autre `transferId` ;
- les recherches par identifiant et par clé retournent deux transactions différentes ;
- un identifiant fonctionnel est réutilisé avec une identité de requête incompatible.

La couche API devra traduire cette erreur en réponse de conflit stable, sans exposer de données internes.

## État du code

Le paquet `@mansa/domain` expose désormais `TransferService`, `TransferExecutor`, `TransferServiceDependencies` et `TransferIdentityConflictError`. Les tests couvrent une nouvelle exécution, le rejeu sans mutation et le refus d’une clé déjà liée à un autre transfert.

## Étape suivante

Le prochain lot doit relier ce service au grand livre : chargement et verrouillage des wallets, vérification de devise et de solde disponible, création de la transaction équilibrée, persistance atomique et publication outbox.
