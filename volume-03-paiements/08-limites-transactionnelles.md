# Limites transactionnelles

## Objet

Les limites transactionnelles protègent les clients, les commerçants, les partenaires et la plateforme contre les erreurs, la fraude et les usages incompatibles avec le niveau de vérification du titulaire. Elles sont appliquées avant l’autorisation financière et ne remplacent ni les contrôles KYC, ni les règles de risque, ni les contrôles réglementaires.

## Portées prises en charge

Une limite peut être définie pour :

- un utilisateur ;
- un portefeuille ;
- un commerçant ;
- un terminal ;
- un pays.

Les limites de plusieurs portées peuvent s’appliquer à une même opération. L’opération est refusée dès qu’une seule limite obligatoire est dépassée ou inactive.

## Périodes

Le socle reconnaît les périodes suivantes :

- `PER_TRANSACTION` : montant maximal pour une opération ;
- `DAILY` : cumul sur une journée métier ;
- `WEEKLY` : cumul sur une semaine métier ;
- `MONTHLY` : cumul sur un mois métier.

Le début et la fin de période doivent être calculés dans le fuseau métier du pays concerné, puis persistés sous forme d’instants UTC.

## Montants et devises

Tous les montants sont représentés dans l’unité mineure de la devise et utilisent un entier. Aucun nombre flottant n’est autorisé.

Une consommation et une demande doivent utiliser la même devise que la limite. Une différence de devise provoque un refus explicite avant tout calcul de conversion.

## États

- `ACTIVE` : la limite est applicable pendant sa période de validité ;
- `SUSPENDED` : la limite ne peut pas autoriser d’opération ;
- `EXPIRED` : la limite n’est plus applicable.

Une limite future ou arrivée à expiration est considérée inactive au moment de l’évaluation.

## Évaluation

L’évaluation reçoit :

1. la définition de la limite ;
2. la consommation courante de la période ;
3. le montant demandé ;
4. l’instant d’évaluation.

Le résultat expose :

- `allowed`, décision finale ;
- `reason`, motif stable ;
- `remainingAmountMinor`, montant encore disponible avant l’opération.

Les motifs de référence sont :

- `WITHIN_LIMIT` ;
- `LIMIT_EXCEEDED` ;
- `CURRENCY_MISMATCH` ;
- `LIMIT_INACTIVE`.

## Concurrence et atomicité

La lecture de la consommation, la réservation du montant et l’écriture de l’opération doivent être atomiques. Deux opérations concurrentes ne doivent pas pouvoir consommer le même solde de limite.

L’implémentation applicative devra utiliser un verrou transactionnel, une écriture conditionnelle ou un mécanisme de réservation idempotent adapté au stockage retenu.

## Administration

Toute création, modification, suspension ou expiration manuelle d’une limite doit :

- être soumise à une permission dédiée ;
- être journalisée ;
- comporter un motif ;
- respecter la séparation des tâches pour les modifications à risque élevé ;
- être versionnée afin de reconstruire la règle applicable à une date donnée.

## Alignement avec le code

Le contrat de référence est implémenté dans `packages/contracts/src/transaction-limits.ts` du dépôt `mansa-platform`.

Il expose notamment :

- les catalogues de périodes, portées et statuts ;
- `evaluateTransactionLimit` ;
- `createTransactionLimitAmount` ;
- les types `TransactionLimit`, `TransactionLimitConsumption` et `TransactionLimitEvaluation`.

## Critères d’acceptation

- Une opération inférieure ou égale au montant restant est autorisée.
- Une opération supérieure au montant restant est refusée.
- Une devise différente est refusée.
- Une limite suspendue, expirée ou future est refusée.
- Le montant restant ne devient jamais négatif.
- Une limite négative ne peut pas être créée.
- Les tests automatisés couvrent les décisions et les principaux cas d’erreur.
