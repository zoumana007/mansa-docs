# Remboursements

## Objet

Un remboursement restitue tout ou partie d’un paiement confirmé. Il constitue une opération financière distincte, liée au paiement d’origine et traçable de bout en bout.

## Principes

- Un remboursement référence toujours le paiement d’origine.
- Le montant est exprimé en unité monétaire mineure et dans la même devise que le paiement.
- Le cumul des remboursements réussis ne doit jamais dépasser le montant payé.
- Chaque demande possède une clé d’idempotence.
- Les demandes sensibles peuvent imposer une revue et une approbation séparées.
- Une opération terminée ne peut pas être réouverte.
- Les références prestataire et codes d’échec sont conservés sans stocker de secret.

## Motifs

- `CUSTOMER_REQUEST` : demande du client.
- `DUPLICATE_PAYMENT` : paiement dupliqué.
- `FRAUD_CONFIRMED` : fraude confirmée après traitement.
- `ORDER_CANCELLED` : commande annulée.
- `SERVICE_NOT_DELIVERED` : bien ou service non livré.
- `MERCHANT_GESTURE` : geste commercial.
- `TECHNICAL_CORRECTION` : correction d’une anomalie technique.
- `OTHER` : autre motif documenté dans le dossier métier.

## États

- `REQUESTED` : demande enregistrée.
- `REVIEW_REQUIRED` : contrôle humain obligatoire.
- `APPROVED` : demande autorisée.
- `PROCESSING` : instruction envoyée ou en cours chez le prestataire.
- `SUCCEEDED` : restitution confirmée.
- `FAILED` : échec final avec code obligatoire.
- `CANCELLED` : demande annulée avant traitement irréversible.

Les états finaux sont `SUCCEEDED`, `FAILED` et `CANCELLED`.

## Transitions autorisées

| État courant | États suivants autorisés |
|---|---|
| `REQUESTED` | `REVIEW_REQUIRED`, `APPROVED`, `CANCELLED` |
| `REVIEW_REQUIRED` | `APPROVED`, `CANCELLED` |
| `APPROVED` | `PROCESSING`, `CANCELLED` |
| `PROCESSING` | `SUCCEEDED`, `FAILED` |
| `SUCCEEDED` | aucun |
| `FAILED` | aucun |
| `CANCELLED` | aucun |

## Contrôles métier

Avant approbation, le service vérifie :

1. que le paiement est confirmé et remboursable ;
2. que la devise correspond à celle du paiement ;
3. que le montant demandé est positif ;
4. que le montant demandé ne dépasse pas le solde remboursable ;
5. qu’aucune demande identique n’existe pour la même clé d’idempotence ;
6. que le demandeur possède les droits nécessaires ;
7. que les règles de séparation des tâches sont respectées lorsque la revue est obligatoire ;
8. qu’aucun litige, gel conformité ou blocage risque n’interdit l’opération.

Le solde remboursable est calculé comme le montant du paiement moins la somme des remboursements `SUCCEEDED`.

## Idempotence et concurrence

La création est idempotente par paiement et clé de demande. La réservation du montant remboursable doit être atomique afin d’empêcher deux remboursements concurrents de dépasser le montant initial.

Les webhooks prestataire sont corrélés par identifiant de remboursement, paiement, référence externe et identifiant de corrélation.

## Audit et observabilité

Chaque création, approbation, annulation et transition conserve :

- le paiement et le remboursement concernés ;
- le montant et la devise ;
- le motif ;
- l’ancien et le nouvel état ;
- le demandeur, le réviseur et l’approbateur lorsqu’ils existent ;
- la référence prestataire et le code d’échec ;
- les horodatages et l’identifiant de corrélation.

Les métriques couvrent le taux de remboursement, les montants par motif, les délais d’approbation, les échecs prestataire et les tentatives de dépassement du montant remboursable.

## Alignement avec le code

Le contrat de référence est `packages/contracts/src/refund.ts` dans `mansa-platform`.

Il expose :

- `REFUND_STATUSES` et `REFUND_REASONS` ;
- `Refund`, `CreateRefundCommand` et `TransitionRefundCommand` ;
- `createRefund` et `transitionRefund` ;
- `canTransitionRefund`, `isFinalRefundStatus`, `isRefundStatus` et `isRefundReason` ;
- `remainingRefundableAmount` ;
- le sous-chemin public `@mansa/contracts/refund`.

## Critères d’acceptation

- Les montants utilisent des entiers sûrs strictement positifs.
- La devise est normalisée en code de trois lettres.
- Une demande peut être créée avec ou sans revue obligatoire.
- Seules les transitions documentées sont acceptées.
- Un échec possède obligatoirement un code.
- Le solde remboursable ne devient jamais négatif.
- Les tests couvrent création, revue, cycle nominal, échec, calcul du solde et entrées invalides.
- Les noms documentés correspondent aux exports du paquet de contrats.
