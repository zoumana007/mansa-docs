# Litiges et contestations

## Objet

Un litige représente la contestation formelle d’un paiement par un client, un réseau de cartes, une banque ou un prestataire. Il est distinct d’un remboursement volontaire : le litige suit un calendrier imposé, nécessite des preuves et peut entraîner une perte financière forcée.

## Principes

- Chaque litige référence le paiement d’origine.
- Le montant est exprimé en unité monétaire mineure et dans la devise du paiement.
- Une échéance de réponse est obligatoire et doit être postérieure à l’ouverture.
- Les preuves sont référencées dans un stockage sécurisé ; leur contenu sensible n’est pas placé dans les journaux.
- Une décision gagnée ou perdue est finale.
- Toute transition et tout dépôt de preuve sont audités.
- Le système distingue clairement remboursement, litige et fraude.

## Motifs

- `CARDHOLDER_NOT_PRESENT` : titulaire absent ou paiement non reconnu dans un contexte carte non présente.
- `DUPLICATE_CHARGE` : débit en double.
- `FRAUDULENT_TRANSACTION` : transaction déclarée frauduleuse.
- `GOODS_NOT_RECEIVED` : bien ou service non reçu.
- `GOODS_NOT_AS_DESCRIBED` : bien ou service non conforme.
- `CREDIT_NOT_PROCESSED` : remboursement ou avoir non traité.
- `CANCELLED_RECURRING_PAYMENT` : paiement récurrent maintenu après annulation.
- `OTHER` : autre motif documenté.

## États

- `OPENED` : litige enregistré.
- `EVIDENCE_REQUIRED` : preuves attendues.
- `UNDER_REVIEW` : dossier complet en cours d’examen.
- `WON` : décision favorable au commerçant ou à Mansa.
- `LOST` : décision défavorable et perte confirmée.
- `WITHDRAWN` : contestation retirée.
- `EXPIRED` : délai de réponse dépassé sans traitement recevable.

Les états finaux sont `WON`, `LOST`, `WITHDRAWN` et `EXPIRED`.

## Transitions autorisées

| État courant | États suivants autorisés |
|---|---|
| `OPENED` | `EVIDENCE_REQUIRED`, `UNDER_REVIEW`, `WITHDRAWN`, `EXPIRED` |
| `EVIDENCE_REQUIRED` | `UNDER_REVIEW`, `WITHDRAWN`, `EXPIRED` |
| `UNDER_REVIEW` | `WON`, `LOST`, `WITHDRAWN` |
| `WON` | aucun |
| `LOST` | aucun |
| `WITHDRAWN` | aucun |
| `EXPIRED` | aucun |

Le passage à `UNDER_REVIEW` exige au moins une preuve. Les décisions `WON` et `LOST` exigent une note de résolution.

## Preuves

Les types de preuves pris en charge sont :

- `TRANSACTION_RECEIPT` ;
- `DELIVERY_PROOF` ;
- `CUSTOMER_COMMUNICATION` ;
- `REFUND_PROOF` ;
- `TERMS_ACCEPTANCE` ;
- `IDENTITY_VERIFICATION` ;
- `OTHER`.

Chaque preuve possède un identifiant unique dans le litige, une référence de stockage, l’acteur ayant effectué le dépôt et son horodatage. Une preuve ne peut plus être ajoutée après la clôture.

## Échéances et alertes

Le service calcule si la réponse est en retard à partir de `responseDeadlineAt`. Des alertes doivent être produites avant l’échéance, notamment à J-7, J-3, J-1 et au dépassement. Les délais réels restent configurables par réseau et partenaire.

Une tâche opérationnelle peut classer automatiquement le dossier `EXPIRED` uniquement après vérification des règles du prestataire et émission d’un événement auditable.

## Contrôles métier

Avant l’ouverture ou la transition, le système vérifie :

1. l’existence du paiement ;
2. la cohérence du montant et de la devise ;
3. la validité de la période de contestation ;
4. l’unicité de la référence prestataire ;
5. les autorisations de l’acteur ;
6. l’absence de preuve dupliquée ;
7. la présence d’au moins une preuve avant revue ;
8. la présence d’une justification avant décision finale.

## Audit, sécurité et conformité

Les journaux conservent les identifiants, états, motifs, dates, acteurs et références techniques. Les pièces jointes, numéros de carte complets, documents d’identité et conversations privées restent dans un stockage chiffré avec accès limité et politique de rétention.

Les actions sensibles doivent respecter la séparation des tâches : la personne qui prépare les preuves ne doit pas nécessairement être celle qui valide une concession financière importante.

## Effets financiers

Une décision `LOST` peut déclencher un débit du compte de règlement commerçant, des frais de litige et une écriture comptable dédiée. Ces effets doivent être exécutés par le grand livre et non directement par le contrat de domaine.

Une décision `WON` clôt le risque sans remboursement automatique. Les remboursements volontaires restent gérés par le module `refund`.

## Alignement avec le code

Le contrat de référence est `packages/contracts/src/dispute.ts` dans `mansa-platform`.

Il expose :

- `DISPUTE_STATUSES`, `DISPUTE_REASONS` et `DISPUTE_EVIDENCE_TYPES` ;
- `Dispute`, `DisputeEvidence` et les commandes associées ;
- `openDispute`, `addDisputeEvidence` et `transitionDispute` ;
- les gardes de type et règles de transition ;
- `isDisputeResponseOverdue` ;
- le sous-chemin public `@mansa/contracts/dispute`.

## Critères d’acceptation

- Le montant est un entier sûr strictement positif.
- La devise est normalisée sur trois lettres.
- L’échéance est postérieure à l’ouverture.
- Une preuve possède un identifiant unique.
- Une revue ne démarre pas sans preuve.
- Une décision gagnée ou perdue contient une justification.
- Les états finaux ne peuvent plus être modifiés.
- Les tests couvrent ouverture, preuve, cycle nominal, échéance et entrées invalides.
- La documentation et les noms exportés restent synchronisés.
