# Résultats de transfert

## Objectif

Le résultat de transfert est la réponse stable produite après le traitement d’une `TransferCommand`. Il permet aux API et aux applications de distinguer une nouvelle exécution réussie d’une réponse rejouée par le mécanisme d’idempotence, sans exposer les détails internes du ledger.

## Données exposées

- `transferId` : identifiant fonctionnel du transfert demandé ;
- `transactionId` : référence de la transaction financière enregistrée ;
- `status` : `COMPLETED` pour une nouvelle exécution ou `REPLAYED` pour une réponse déjà produite ;
- `completedAt` : date UTC ISO 8601 associée au résultat persistant.

## Invariants

1. Les identifiants de transfert et de transaction sont obligatoires et normalisés.
2. Le statut appartient uniquement à l’ensemble `COMPLETED | REPLAYED`.
3. La date de complétion est valide et copiée défensivement.
4. Un résultat rejoué référence la transaction originale et n’entraîne aucune nouvelle mutation du ledger.
5. La sérialisation est stable afin de servir de contrat entre le domaine, l’API et les clients.

## Sémantique d’idempotence

Lorsqu’une clé d’idempotence est reçue pour la première fois et que le transfert aboutit, le service renvoie un résultat `COMPLETED` puis conserve la référence de transaction dans l’enregistrement d’idempotence.

Lorsqu’une nouvelle requête présente la même clé et le même contenu, le service renvoie `REPLAYED` avec la même référence de transaction. Une même clé associée à un contenu différent constitue un conflit et ne produit pas de `TransferResult` réussi.

## Contrat API cible

La couche HTTP pourra traduire le résultat de la façon suivante :

```json
{
  "transferId": "transfer-2026-000001",
  "transactionId": "transaction-2026-000001",
  "status": "COMPLETED",
  "completedAt": "2026-07-31T17:30:00.000Z"
}
```

Le code HTTP exact, les en-têtes d’idempotence et la politique de rétention seront définis dans le catalogue API. Le domaine ne dépend pas de HTTP.

## État du code

Le paquet `@mansa/domain` expose `TransferResult`, `TransferResultStatus` et `InvalidTransferResultError`. Des constructeurs explicites couvrent les cas `completed` et `replayed`. Les tests vérifient la normalisation, la sérialisation, la copie défensive de la date et le rejet des identifiants, dates et statuts invalides.

## Étape suivante

Le prochain lot doit introduire le contrat de persistance de l’idempotence et le service applicatif qui orchestre la commande, le verrouillage des wallets, l’écriture atomique du ledger et la production du résultat.
