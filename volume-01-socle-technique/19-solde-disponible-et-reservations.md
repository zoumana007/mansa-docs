# Solde disponible et réservations

## Objectif

Le solde comptable ne suffit pas pour décider si une nouvelle opération peut être acceptée. Mansa distingue le solde issu du grand livre des montants temporairement réservés par des autorisations, paiements en cours ou contrôles réglementaires.

La primitive `calculateAvailableBalance` est exposée par `@mansa/domain` depuis `packages/domain/src/available-balance.ts`.

## Définitions

- `ledgerBalanceMinor` : solde comptable calculé depuis les écritures validées du grand livre.
- `reservedMinor` : somme des réservations encore actives.
- `availableMinor` : montant immédiatement mobilisable.

```text
availableMinor = ledgerBalanceMinor - reservedMinor
```

Tous les montants sont représentés en `bigint` et en unité monétaire mineure.

## Cycle de vie d’une réservation

Une réservation porte un identifiant stable, un montant positif et un statut :

- `ACTIVE` : le montant réduit le solde disponible ;
- `RELEASED` : la réservation a été libérée sans débit définitif ;
- `CAPTURED` : l’autorisation a été transformée en écriture comptable ;
- `EXPIRED` : la réservation a dépassé sa durée de validité.

Seules les réservations `ACTIVE` sont additionnées. Deux réservations actives ne peuvent pas partager le même identifiant.

## Règles d’intégrité

- L’identifiant de réservation est obligatoire.
- Le montant d’une réservation est strictement positif.
- Le montant demandé pour un nouveau débit est strictement positif.
- Un débit est refusé lorsque le solde disponible est inférieur au montant demandé.
- Une valeur disponible négative n’est pas masquée : elle doit déclencher une alerte et un rapprochement.
- La devise, le wallet et le périmètre de compte doivent être filtrés avant le calcul.

## Concurrence

Le contrôle de disponibilité et la création de la réservation doivent appartenir à la même transaction logique. Lire le solde puis réserver dans une seconde opération indépendante crée un risque de double dépense.

Les implémentations autorisées comprennent :

- verrouillage pessimiste du wallet ;
- contrôle de version optimiste avec nouvelle tentative bornée ;
- sérialisation des commandes par wallet ;
- écriture atomique conditionnelle côté base de données.

Le choix définitif doit être documenté dans l’adaptateur de persistance et testé sous concurrence.

## Idempotence

L’identifiant de réservation doit dériver d’une clé d’idempotence stable fournie par le canal appelant ou générée à l’entrée de la plateforme. Une répétition réseau ne doit jamais créer une seconde réservation.

La libération, l’expiration et la capture doivent également être idempotentes. Une réservation déjà terminale ne doit pas revenir à l’état actif.

## Expiration et capture

Chaque réservation persistée doit comporter au minimum :

- la date de création ;
- la date d’expiration ;
- la référence de l’opération ;
- le canal et le partenaire ;
- le statut courant ;
- les traces de capture ou de libération.

Un traitement planifié peut expirer les réservations anciennes, mais la vérification à la lecture reste nécessaire afin qu’un retard du traitement planifié ne bloque pas durablement le client.

Lors d’une capture, l’écriture comptable définitive et le passage de la réservation à `CAPTURED` doivent être atomiques ou coordonnés par un workflow idempotent compensable.

## Critères d’acceptation

- Un solde comptable de 10 000 avec une réservation active de 2 000 produit un solde disponible de 8 000.
- Les réservations libérées, capturées ou expirées ne réduisent pas le disponible.
- Une réservation active dupliquée est rejetée.
- Un identifiant vide ou un montant non positif est rejeté.
- Une demande égale au solde disponible est acceptée.
- Une demande supérieure au solde disponible est refusée.
- Les tests automatisés couvrent le calcul, les statuts, les doublons et le contrôle d’insuffisance.

## Hypothèses à valider

Les durées d’autorisation, les règles de découvert, les réservations liées aux cartes, les frais estimés et les retenues réglementaires devront être confirmés avec la banque partenaire, le processeur de paiement et les opérateurs Mobile Money. Ces règles doivent rester configurables par produit, canal, partenaire et pays.
