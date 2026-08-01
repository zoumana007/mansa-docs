# Application Commerçant — Affectations aux points de vente

## 1. Objectif

Une permission liée au rôle général d’un employé ne suffit pas à autoriser une opération dans tous les points de vente. Une affectation explicite relie un employé commerçant à un point de vente et définit sa portée opérationnelle locale.

Le contrôle final doit combiner :

1. le statut du profil commerçant ;
2. le statut du point de vente ;
3. le statut de l’employé ;
4. les permissions de son rôle ;
5. son affectation au point de vente ;
6. les permissions et limites propres à cette affectation.

## 2. Modèle de domaine

Le dépôt plateforme expose `MerchantStaffAssignment` dans `packages/domain/src/merchant-staff-assignment.ts`.

Champs principaux :

- `id` : identifiant interne de l’affectation ;
- `merchantId` : commerce propriétaire ;
- `staffMemberId` : employé concerné ;
- `locationId` : point de vente autorisé ;
- `permissions` : permissions locales accordées ;
- `maxTransactionAmountMinor` : plafond par opération en unité monétaire mineure ;
- `status` : état courant de l’affectation ;
- `statusReason` : motif de suspension ou révocation ;
- `createdAt` : date de création.

Les montants sont représentés par des entiers `bigint` dans le domaine et sérialisés sous forme de chaînes. Aucun montant financier n’utilise de nombre flottant.

## 3. Cycle de vie

| État | Effet |
|---|---|
| `active` | Les permissions locales peuvent être utilisées |
| `suspended` | Toute opération locale est temporairement refusée |
| `revoked` | L’affectation est définitivement invalidée |

Une affectation suspendue peut être réactivée. Une affectation révoquée ne peut plus être modifiée ni réactivée ; une nouvelle affectation doit être créée pour préserver l’historique.

## 4. Permissions et limites

Les permissions sont normalisées en minuscules, dédupliquées et comparées exactement. Exemples :

- `payments.accept` ;
- `transactions.read` ;
- `refunds.create` ;
- `reports.read` ;
- `cash.register.open` ;
- `cash.register.close`.

La méthode de domaine `allows(permission, amountMinor?)` refuse l’opération lorsque :

- l’affectation n’est pas active ;
- la permission locale est absente ;
- le montant est négatif ;
- le montant dépasse le plafond local configuré.

L’absence de plafond signifie que ce modèle local n’ajoute pas de limite. Les plafonds globaux du commerce, du rôle, du terminal, du moyen de paiement et de la conformité continuent cependant de s’appliquer.

## 5. Règle d’autorisation composée

Une opération dans un point de vente est autorisée uniquement lorsque tous les contrôles sont positifs :

```text
profil commerçant actif
ET point de vente actif
ET employé actif
ET rôle autorisé
ET affectation active pour ce point
ET permission locale présente
ET montant sous toutes les limites applicables
ET terminal autorisé, lorsque nécessaire
```

L’interface peut masquer les actions indisponibles, mais l’API reste l’autorité finale.

## 6. Sécurité et audit

- La création, modification, suspension, réactivation et révocation d’une affectation sont auditées.
- Le motif est obligatoire pour toute suspension ou révocation.
- Une modification de permissions ou de plafond prend effet côté serveur sans nécessiter une nouvelle version de l’application.
- Les identifiants du commerce, de l’employé et du point de vente doivent être vérifiés comme appartenant au même périmètre.
- Les autorisations déjà chargées sur un appareil doivent avoir une durée de cache courte et être invalidables.
- Aucun code PIN, secret de terminal ou jeton d’accès n’est stocké dans l’agrégat.

## 7. Critères d’acceptation

- Une affectation active autorise uniquement ses permissions déclarées.
- Une permission est insensible aux espaces extérieurs et à la casse lors de sa normalisation.
- Les doublons de permission sont éliminés.
- Un plafond négatif est refusé.
- Un montant égal au plafond est accepté ; un montant supérieur est refusé.
- Une affectation suspendue ou révoquée n’autorise aucune opération.
- Une affectation suspendue peut être réactivée et perd alors son motif.
- Une affectation révoquée ne peut plus modifier ses permissions ni son plafond.
- Les identifiants et permissions vides sont refusés.
- La sérialisation du plafond ne produit pas de nombre flottant.

## 8. Cohérence avec le code

L’agrégat est exporté par `packages/domain/src/index.ts` et couvert par `packages/domain/test/merchant-staff-assignment.test.mjs`.

Le prochain lot cohérent doit introduire l’enregistrement des terminaux et leur rattachement aux points de vente, afin de compléter le triplet employé–point de vente–terminal avant les flux d’encaissement.
