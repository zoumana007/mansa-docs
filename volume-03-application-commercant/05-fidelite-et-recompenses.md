# Fidélité et récompenses

## 1. Objectif

Le module de fidélité permet à un commerçant de créer un programme de points, d’inscrire ses clients, d’accorder des points après des opérations éligibles et de proposer des récompenses consommables. Le même socle doit fonctionner pour un commerce unique, une chaîne ou un réseau partenaire.

## 2. Principes métier

- Les points sont des unités entières non monétaires.
- Une opération d’attribution ou de dépense ne modifie jamais directement un solde : elle crée une écriture de fidélité.
- Toute écriture est corrélée à une référence métier lorsqu’elle provient d’un paiement, d’un remboursement ou d’une action administrative.
- Une même référence ne doit pas attribuer deux fois les mêmes points.
- Les annulations et remboursements sont traités par des écritures de sens inverse.
- Les programmes, récompenses et comptes peuvent être suspendus sans supprimer l’historique.
- Les codes de consommation ne doivent jamais être prédictibles.

## 3. Cycle de vie du programme

Un programme passe par les statuts suivants :

- `DRAFT` : configuration non visible des clients ;
- `ACTIVE` : inscriptions, gains et échanges autorisés ;
- `PAUSED` : aucune nouvelle attribution ni dépense, historique consultable ;
- `ENDED` : programme clos, traitement des soldes restants selon la politique annoncée.

Le commerçant configure le nom des points, le taux de gain, le montant minimal éventuel, la date de début, la date de fin et la durée d’expiration des points.

## 4. Compte de fidélité

Un compte relie un client à un programme. Il conserve :

- le solde disponible ;
- le solde en attente ;
- le cumul gagné ;
- le cumul dépensé ;
- le statut et les dates d’inscription et de mise à jour.

Les statuts sont `ACTIVE`, `SUSPENDED` et `CLOSED`. Une fermeture n’efface jamais les transactions.

## 5. Écritures de points

Les types d’écriture sont :

- `EARN` : gain confirmé ;
- `REDEEM` : dépense pour une récompense ;
- `ADJUSTMENT` : correction administrative auditée ;
- `EXPIRATION` : retrait automatique de points expirés ;
- `REVERSAL` : annulation d’une écriture précédente.

Les points doivent être des entiers positifs ou nuls. Toute correction administrative exige un motif, une autorisation adaptée et un événement d’audit.

## 6. Récompenses

Les types initiaux sont `DISCOUNT`, `CASHBACK`, `VOUCHER`, `GIFT` et `POINTS`. Une récompense définit son coût en points, sa valeur monétaire éventuelle, son stock, sa limite par client et sa période de validité.

Les statuts sont `DRAFT`, `ACTIVE`, `PAUSED`, `EXHAUSTED` et `ENDED`.

Une dépense produit un objet de consommation avec un code, une date d’émission, une éventuelle expiration, une date de consommation et une éventuelle annulation.

## 7. API de référence

Les contrats partagés sont définis dans :

- `packages/contracts/src/loyalty.ts` ;
- `packages/contracts/src/loyalty-api.ts`.

Les routes prévues sont :

- `GET|POST /v1/loyalty/programs` ;
- `GET|PATCH /v1/loyalty/programs/:programId` ;
- `POST /v1/loyalty/accounts` ;
- `GET /v1/loyalty/accounts/:accountId` ;
- `GET /v1/loyalty/accounts/:accountId/transactions` ;
- `POST /v1/loyalty/accounts/:accountId/earn` ;
- `GET /v1/loyalty/programs/:programId/rewards` ;
- `POST /v1/loyalty/rewards/:rewardId/redemptions` ;
- `GET /v1/loyalty/redemptions/:redemptionId`.

## 8. Sécurité et anti-fraude

- Clé d’idempotence obligatoire pour gain, échange, annulation et correction.
- Contrôle du commerçant, du programme et du client sur chaque opération.
- Interdiction d’exposer des codes complets dans les journaux.
- Limites configurables par transaction, client, programme et période.
- Détection des gains répétés, références dupliquées, volumes anormaux et corrections excessives.
- Séparation des rôles entre création d’une récompense, activation et ajustement manuel important.

## 9. Critères d’acceptation minimaux

1. Un paiement éligible ne peut créer qu’une attribution de points.
2. Un remboursement peut produire une écriture inverse traçable.
3. Un échange échoue sans modifier le solde si les points ou le stock sont insuffisants.
4. Deux requêtes portant la même clé d’idempotence retournent le même résultat métier.
5. Un programme suspendu refuse les nouveaux gains et échanges.
6. Les points expirés restent visibles dans l’historique.
7. Une récompense consommée ne peut pas être consommée une seconde fois.
8. Toute correction administrative est autorisée et auditée.

## 10. Alignement documentation-code

Les noms de statuts, types et routes de ce document correspondent aux constantes et interfaces des deux fichiers de contrats. Leur export public depuis `packages/contracts/src/index.ts`, les tests unitaires et l’implémentation persistante restent à réaliser dans les prochains lots.
