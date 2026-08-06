# Fidélité et récompenses commerçant

## Objectif

Le module de fidélité permet à un commerçant de créer un programme de points, d’inscrire ses clients, d’attribuer ou retirer des points de manière traçable et de proposer des récompenses échangeables. Il doit rester cohérent avec les paiements, les remboursements, les annulations et les règles configurées dans l’administration.

## Périmètre initial

Le premier socle couvre :

- les programmes liés à un commerçant ;
- l’inscription d’un client dans un programme ;
- les comptes de fidélité et leurs soldes ;
- l’acquisition, l’utilisation, l’ajustement, l’expiration et l’annulation de points ;
- les récompenses de type remise, cashback, bon, cadeau ou points ;
- les limites globales et par client ;
- les périodes d’activation ;
- l’historique paginé des mouvements ;
- l’émission et la consommation d’un code de récompense.

## Statuts

### Programme

- `DRAFT` : programme modifiable mais non disponible ;
- `ACTIVE` : acquisition et utilisation autorisées ;
- `PAUSED` : nouvelles opérations suspendues sans suppression de l’historique ;
- `ENDED` : programme terminé et non réactivable par le parcours normal.

### Compte client

- `ACTIVE` : compte utilisable ;
- `SUSPENDED` : opérations bloquées avec conservation des données ;
- `CLOSED` : compte clôturé selon la politique de conservation.

### Récompense

- `DRAFT` : non visible ;
- `ACTIVE` : échange autorisé ;
- `PAUSED` : échange temporairement bloqué ;
- `EXHAUSTED` : stock ou plafond atteint ;
- `ENDED` : période terminée.

## Calcul des points

Le commerçant configure le taux d’acquisition, le montant minimal éventuel et la durée de validité. Le backend reste autoritatif pour le calcul final.

Les règles suivantes s’appliquent :

1. Les points sont des entiers sûrs, jamais des nombres flottants représentant de l’argent.
2. Le montant financier utilisé pour calculer les points respecte le type `Money` et les unités mineures.
3. Une opération de paiement ne crédite les points qu’une fois grâce à une référence métier et une clé d’idempotence.
4. Un remboursement ou une annulation produit une écriture inverse liée à l’écriture d’origine.
5. Le solde disponible ne peut pas devenir négatif lors d’un échange normal.
6. Les points en attente restent séparés des points disponibles jusqu’à la confirmation de l’opération source.

## Journal de fidélité

Chaque mouvement possède :

- un identifiant unique ;
- un compte de fidélité ;
- un type `EARN`, `REDEEM`, `ADJUSTMENT`, `EXPIRATION` ou `REVERSAL` ;
- un nombre entier de points ;
- une référence métier lorsqu’elle existe ;
- une justification pour les ajustements administratifs ;
- une date de création et, si nécessaire, une date d’expiration.

Les mouvements publiés ne sont pas modifiés. Une correction utilise un mouvement compensatoire et conserve le lien avec l’origine.

## Récompenses

Une récompense définit au minimum son programme, son nom, son type, son coût en points, sa période de validité et son statut. Elle peut également définir :

- une valeur monétaire ;
- un stock total ;
- une limite par client ;
- une date d’expiration après émission ;
- des conditions d’utilisation par point de vente, produit ou canal.

Le code d’échange ne doit pas contenir de donnée personnelle. Il doit être difficile à deviner, utilisable une seule fois et vérifié côté serveur.

## Intégration avec les paiements

L’attribution de points intervient après confirmation du paiement ou après une période configurable lorsque le risque d’annulation est élevé. L’application TPE ou commerçant ne modifie jamais directement le solde.

Pour chaque paiement éligible, le service de fidélité reçoit un événement idempotent. Les remboursements partiels annulent les points selon une règle déterministe et auditée.

Une récompense monétaire appliquée à une vente doit apparaître dans le détail du paiement et dans le reçu, sans masquer les taxes, frais ou remises réglementaires.

## API initiale

Les routes partagées sont :

- `GET /v1/loyalty/programs` ;
- `POST /v1/loyalty/programs` ;
- `GET /v1/loyalty/programs/:programId` ;
- `PATCH /v1/loyalty/programs/:programId` ;
- `POST /v1/loyalty/accounts` ;
- `GET /v1/loyalty/accounts/:accountId` ;
- `GET /v1/loyalty/accounts/:accountId/transactions` ;
- `POST /v1/loyalty/accounts/:accountId/earn` ;
- `GET /v1/loyalty/programs/:programId/rewards` ;
- `POST /v1/loyalty/rewards/:rewardId/redemptions` ;
- `GET /v1/loyalty/redemptions/:redemptionId`.

Les listes sont paginées. Les mutations financières ou assimilées utilisent une clé d’idempotence dans l’implémentation backend, même lorsque le contrat partagé n’expose pas encore tous les en-têtes HTTP.

## Contrats techniques

Le domaine partagé se trouve dans `mansa-platform/packages/contracts/src/loyalty.ts`.

Le contrat d’API se trouve dans `mansa-platform/packages/contracts/src/loyalty-api.ts` et est agrégé dans `packages/contracts/src/api-contracts.ts`.

Les applications concernées sont principalement `apps/mobile-client`, `apps/mobile-merchant`, `apps/mobile-tpe` et `apps/business-web`. Le backend autoritatif reste `apps/api-gateway`.

## Autorisations et audit

- Un commerçant ne consulte et ne configure que ses propres programmes.
- Un employé doit disposer d’une permission explicite pour attribuer, consommer ou ajuster des points.
- Un ajustement manuel exige un motif, un acteur et un événement d’audit.
- Les suspensions de compte et changements de statut sont audités.
- Les données de fidélité ne doivent pas révéler l’historique complet de paiement à un commerçant non autorisé.
- Les exports respectent les règles de minimisation et de conservation des données.

## Prévention de la fraude

Le backend contrôle notamment :

- les acquisitions répétées sur la même référence ;
- les échanges successifs dépassant le solde ;
- les ajustements anormalement fréquents ;
- les comptes liés à des paiements annulés ;
- les codes de récompense réutilisés ;
- les opérations hors point de vente ou canal autorisé ;
- les volumes dépassant les seuils configurés.

Une alerte ne modifie pas automatiquement un solde sans règle explicite. Elle peut placer l’opération en attente ou demander une validation.

## Critères d’acceptation

1. Une même opération de paiement ne crédite les points qu’une seule fois.
2. Un compte suspendu ne peut ni acquérir ni utiliser de points.
3. Un programme en pause conserve les soldes et l’historique.
4. Un échange est rejeté lorsque le solde disponible est insuffisant.
5. Une récompense hors période ou épuisée ne peut pas être émise.
6. Un code consommé ne peut pas être réutilisé.
7. Un remboursement génère une correction corrélée aux points initialement acquis.
8. Un ajustement manuel contient obligatoirement une justification et une trace d’audit.
9. Les mouvements sont paginés et ordonnés de manière stable.
10. Les routes, types et statuts documentés correspondent aux fichiers `loyalty.ts` et `loyalty-api.ts`.
