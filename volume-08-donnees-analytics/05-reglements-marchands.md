# Règlements marchands

## Objectif

Le règlement marchand transforme les encaissements éligibles en un versement net vers une destination autorisée du commerçant. Il reste distinct du paiement client, du rapprochement partenaire et du grand livre.

## Cycle de vie

Les statuts sont `DRAFT`, `READY`, `PROCESSING`, `PAID`, `PARTIALLY_PAID`, `FAILED` et `CANCELLED`.

Transitions permises :

- `DRAFT` vers `READY` ou `CANCELLED` ;
- `READY` vers `PROCESSING` ou `CANCELLED` ;
- `PROCESSING` vers `PAID`, `PARTIALLY_PAID` ou `FAILED` ;
- `PARTIALLY_PAID` vers `PROCESSING`, `PAID` ou `FAILED` ;
- `FAILED` vers `READY` ou `CANCELLED`.

`PAID` et `CANCELLED` sont finaux.

## Règles

- Les montants utilisent les unités mineures et des entiers sûrs.
- Le montant net correspond au brut diminué des frais puis corrigé par les ajustements validés.
- Le montant net ne peut pas être négatif.
- Une opération source ne peut appartenir qu’à un seul règlement actif.
- La devise des opérations doit correspondre à celle du règlement.
- Les opérations sources doivent être comptabilisées, rapprochées et éligibles.
- Les destinations prises en charge sont `BANK_ACCOUNT`, `MOBILE_MONEY` et `MANSA_WALLET`.
- Toute création ou transition sensible utilise une clé d’idempotence et un identifiant de corrélation.
- Une référence partenaire est obligatoire pour `PAID` et `PARTIALLY_PAID`.
- Un motif explicite est obligatoire pour `FAILED`.

## Contrats techniques

Le domaine est défini dans `packages/contracts/src/settlement.ts` et le transport dans `packages/contracts/src/settlement-api.ts`.

Routes initiales :

- `GET /v1/settlements` ;
- `POST /v1/settlements` ;
- `GET /v1/settlements/:settlementId` ;
- `POST /v1/settlements/:settlementId/status`.

## Audit et sécurité

Les références de destination sont masquées dans les journaux. Toute création, transition, reprise ou annulation est auditée. Les ajustements manuels et reprises après échec peuvent exiger une double validation selon les seuils configurés.

## Critères d’acceptation

- Une même clé d’idempotence ne crée pas deux règlements.
- Une transition interdite est rejetée sans modification.
- Un règlement payé sans référence partenaire est refusé.
- Un échec sans motif est refusé.
- Un montant net négatif est refusé.
- Chaque règlement est traçable jusqu’aux opérations sources, écritures comptables, rapprochements et réponses partenaires.

## Suite

Le prochain lot doit ajouter le module NestJS persistant, le worker d’exécution partenaire, les politiques d’autorisation, les écritures comptables, les tests et les interfaces d’administration et commerçant.
