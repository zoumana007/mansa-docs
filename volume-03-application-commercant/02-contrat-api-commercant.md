# Contrat API — application commerçant

## Objectif

Ce document définit le contrat minimal partagé entre l’application commerçant, le portail administrateur et le backend Mansa. La source TypeScript correspondante est `packages/contracts/src/merchant-api.ts` dans `zoumana007/mansa-platform`.

## Ressources principales

- **Merchant** : établissement juridique et commercial, rattaché à un pays et à une devise par défaut.
- **MerchantLocation** : point de vente ou agence exploité par un commerçant.
- **MerchantMember** : utilisateur autorisé à agir pour le commerçant avec un rôle et un périmètre de points de vente.
- **Settlement** : règlement financier d’une période, avec ventes brutes, remboursements, frais, ajustements et montant net.
- **MerchantDashboardSummary** : agrégat de pilotage sur une période donnée.

## Routes versionnées

| Opération | Méthode | Route |
|---|---|---|
| Créer un commerçant | `POST` | `/v1/merchants` |
| Lister les commerçants | `GET` | `/v1/merchants` |
| Lire un commerçant | `GET` | `/v1/merchants/:merchantId` |
| Créer un point de vente | `POST` | `/v1/merchants/:merchantId/locations` |
| Lister les points de vente | `GET` | `/v1/merchants/:merchantId/locations` |
| Inviter un membre | `POST` | `/v1/merchants/:merchantId/members` |
| Lister les membres | `GET` | `/v1/merchants/:merchantId/members` |
| Lister les règlements | `GET` | `/v1/merchants/:merchantId/settlements` |
| Lire le tableau de bord | `GET` | `/v1/merchants/:merchantId/dashboard` |

## Pagination et filtres

Toutes les listes utilisent le contrat commun `PageRequest` / `PageResponse<T>`. Les filtres autorisés sont :

- commerçants : propriétaire, pays et statut ;
- points de vente : état actif ou inactif ;
- membres : rôle, statut et point de vente ;
- règlements : statut et période.

Le serveur doit imposer une limite maximale de page et un ordre stable. Un client ne doit jamais supposer que la totalité des résultats tient dans une seule réponse.

## Idempotence

Les opérations de création et d’invitation exigent une `idempotencyKey` :

- création d’un commerçant ;
- création d’un point de vente ;
- invitation d’un membre.

Une nouvelle requête portant la même clé et le même contenu retourne le résultat initial. Une même clé réutilisée avec un contenu différent est rejetée et auditée.

## Autorisations minimales

- Un propriétaire peut administrer l’ensemble du commerçant.
- Un responsable agit uniquement sur les points de vente qui lui sont attribués.
- Un caissier peut consulter les opérations nécessaires à l’encaissement, sans modifier la configuration financière.
- Un comptable peut consulter règlements, frais et exports, sans gérer les membres.
- Le support dispose d’un accès limité aux dossiers autorisés et toutes ses actions sont auditées.
- Les actions sensibles suivent la matrice RBAC du volume 5 et peuvent nécessiter une approbation distincte.

## Règlements

Le montant net respecte la relation suivante :

`net = brut - remboursements - frais + ajustements`

Les montants utilisent le type partagé `Money`. Les règlements sont filtrables par période et statut. Un règlement `PAID`, `FAILED`, `HELD` ou `CANCELLED` reste historisé et ne doit pas être supprimé physiquement.

## Tableau de bord

Le tableau de bord reçoit obligatoirement `periodStart` et `periodEnd`, avec un `locationId` facultatif. Il retourne : ventes brutes, remboursements, frais, ventes nettes, nombre de paiements réussis et nombre de paiements échoués.

Les agrégats sont dérivés des écritures et transactions persistées. Ils ne constituent pas la source comptable de vérité.

## Sécurité et confidentialité

- Aucun secret d’acquisition, jeton de terminal ou identifiant bancaire complet n’est exposé.
- Les réponses sont limitées au périmètre autorisé du membre.
- Toute modification de membres, points de vente ou configuration produit un événement d’audit.
- Les données exportées doivent être chiffrées au repos, temporaires et associées à une date d’expiration.

## Critères d’acceptation

1. Toutes les routes sont versionnées sous `/v1` et correspondent au contrat TypeScript.
2. Les listes sont paginées et filtrées côté serveur.
3. Les écritures créatrices sont idempotentes.
4. Un membre ne peut jamais accéder à un point de vente hors de son périmètre.
5. Les montants de règlement restent cohérents et utilisent une seule devise.
6. Les tableaux de bord peuvent être recalculés à partir des transactions persistées.
7. Les erreurs utilisent le contrat commun `ApiErrorResponse` et ne divulguent aucune donnée sensible.
