# Webhooks et événements partenaires

## Objectif

Les webhooks permettent à Mansa de notifier de manière fiable les banques, opérateurs Mobile Money, commerçants, administrations et autres partenaires lorsqu’un événement métier se produit.

## Principes

- Un événement possède un identifiant unique, un type stable, une version de schéma, une date d’occurrence et un identifiant de corrélation.
- Une livraison est créée séparément pour chaque abonnement concerné.
- Le corps signé ne contient aucun secret, code OTP, numéro de carte complet ni document KYC.
- Les secrets de signature sont stockés dans un gestionnaire de secrets et référencés uniquement par identifiant.
- Chaque livraison est idempotente et rejouable de manière contrôlée.
- L’ordre global n’est pas garanti. L’ordre par agrégat doit être préservé lorsque le partenaire l’exige.

## Contrats techniques

Les modèles partagés sont définis dans `packages/contracts/src/webhook.ts` du dépôt `mansa-platform`.

### Abonnement

Un abonnement contient :

- son propriétaire ;
- l’URL HTTPS cible ;
- les types d’événements autorisés ;
- son statut (`ACTIVE`, `PAUSED` ou `DISABLED`) ;
- une référence vers la clé de signature ;
- ses dates de création et de modification.

### Événement

Un événement contient au minimum :

- `id` ;
- `type` ;
- `occurredAt` ;
- `correlationId` ;
- `schemaVersion` ;
- `payload` ;
- éventuellement `aggregateId`.

### Livraison

Une livraison évolue entre les statuts suivants :

`PENDING` → `DELIVERING` → `DELIVERED`

En cas d’échec temporaire :

`DELIVERING` → `RETRY_SCHEDULED` → `DELIVERING`

En cas d’échec définitif ou d’arrêt administratif :

`FAILED` ou `CANCELLED`.

## Signature

Chaque requête doit inclure :

- un identifiant d’événement ;
- un horodatage ;
- une signature HMAC calculée sur l’horodatage et le corps brut ;
- une version de signature.

Le destinataire doit rejeter les horodatages trop anciens, comparer la signature en temps constant et mémoriser les identifiants déjà traités.

## Politique de nouvelle tentative

- délais exponentiels avec gigue ;
- nombre maximal configurable par partenaire et environnement ;
- aucune nouvelle tentative automatique sur les réponses explicitement définitives ;
- bascule en file d’échec après épuisement ;
- relance manuelle réservée aux rôles autorisés et auditée.

## Événements initiaux

- `payment.created`, `payment.succeeded`, `payment.failed`, `payment.refunded` ;
- `transfer.created`, `transfer.completed`, `transfer.failed` ;
- `kyc.submitted`, `kyc.approved`, `kyc.rejected` ;
- `merchant.activated`, `merchant.suspended` ;
- `terminal.activated`, `terminal.disabled` ;
- `public_obligation.created`, `public_obligation.paid` ;
- `settlement.created`, `settlement.completed`, `settlement.failed`.

Chaque type d’événement doit disposer d’une version de schéma, d’exemples anonymisés et d’une politique de compatibilité ascendante.

## Critères d’acceptation

1. La même clé d’idempotence ne crée pas deux abonnements.
2. Un événement est livré au plus une fois avec succès par abonnement, même si plusieurs tentatives réseau ont lieu.
3. Un secret de signature n’apparaît jamais dans les journaux, contrats ou réponses API.
4. Une livraison échouée reste corrélable par événement, abonnement et tentative.
5. Une relance manuelle exige une justification et produit un événement d’audit.
6. La désactivation d’un abonnement empêche toute nouvelle livraison sans supprimer l’historique.
7. Les charges utiles sont validées selon leur version avant émission.
