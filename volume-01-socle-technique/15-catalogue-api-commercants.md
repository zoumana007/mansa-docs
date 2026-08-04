# Catalogue API — commerçants

## 1. Portée

Ce catalogue décrit les routes du domaine commerçant. Les chemins, méthodes et types partagés sont déclarés dans `mansa-platform/packages/contracts/src/merchant-api.ts` et `merchant.ts`.

Préfixe : `/v1`.

Le domaine couvre l’organisation commerçante, ses points de vente, ses membres, son tableau de bord et ses règlements. Les paiements unitaires restent exposés par le domaine paiements ; ce catalogue organise leur consultation et leur exploitation dans le contexte commerçant.

## 2. Routes

### `POST /v1/merchants`

Crée un commerçant à partir de `CreateMerchantCommand` et retourne un `Merchant`.

Règles minimales :

- exiger une clé d’idempotence ;
- vérifier l’identité et le niveau KYC/KYB du propriétaire ;
- valider le pays, la devise, la catégorie d’activité et les identifiants déclarés ;
- empêcher la duplication d’une même entreprise selon les clés de rapprochement configurées ;
- créer le propriétaire avec le rôle `OWNER` dans une opération cohérente ;
- placer le commerçant dans le statut approprié selon la politique de revue.

### `GET /v1/merchants`

Liste les commerçants accessibles à l’acteur. Les filtres supportés sont `status` et `countryCode` via `ListMerchantsQuery`.

Un utilisateur standard ne voit que les organisations auxquelles il appartient. Un administrateur doit être limité à sa portée pays, organisation et environnement.

### `GET /v1/merchants/:merchantId`

Retourne un commerçant accessible. Une ressource hors portée ne doit pas révéler son existence.

### `POST /v1/merchants/:merchantId/locations`

Crée un point de vente à partir de `CreateMerchantLocationCommand` et retourne un `MerchantLocation`.

Le backend vérifie que le pays du point de vente est autorisé, que l’acteur dispose du droit de gestion et que le nom est unique dans la portée choisie lorsque cette contrainte est activée.

### `GET /v1/merchants/:merchantId/locations`

Liste les points de vente. Le filtre `isActive` est exposé par `ListMerchantLocationsQuery`.

### `POST /v1/merchants/:merchantId/members`

Invite un membre à partir de `InviteMerchantMemberCommand` et retourne un `MerchantMember`.

Règles minimales :

- vérifier le droit d’invitation et la séparation des tâches ;
- limiter les rôles attribuables selon le rôle de l’invitant ;
- vérifier que chaque `locationId` appartient au commerçant ;
- ne jamais permettre l’auto-attribution silencieuse du rôle `OWNER` ;
- tracer l’invitation, l’activation, la suspension et la révocation.

### `GET /v1/merchants/:merchantId/members`

Liste les membres accessibles. Le filtre `locationId` est exposé par `ListMerchantMembersQuery`.

Les données de contact doivent être minimisées selon le rôle de l’appelant.

### `GET /v1/merchants/:merchantId/dashboard`

Retourne un `MerchantDashboardSummary` pour l’intervalle demandé par `MerchantDashboardQuery`.

`periodStart` et `periodEnd` sont obligatoires. `locationId` est facultatif. Les montants doivent provenir de données financières rapprochées ou clairement signalées comme provisoires.

### `GET /v1/merchants/:merchantId/settlements`

Liste les `Settlement` selon `ListSettlementsQuery`, avec filtres facultatifs `status`, `periodStart` et `periodEnd`.

Un règlement ne doit jamais être recalculé silencieusement après finalisation. Toute correction passe par une écriture d’ajustement traçable.

## 3. Statuts

Les statuts commerçant sont définis par `MERCHANT_STATUSES` :

- `DRAFT`
- `UNDER_REVIEW`
- `ACTIVE`
- `RESTRICTED`
- `SUSPENDED`
- `CLOSED`

Les rôles membres sont définis par `MERCHANT_MEMBER_ROLES` : `OWNER`, `MANAGER`, `CASHIER`, `ACCOUNTANT`, `SUPPORT`.

Les statuts de règlement sont définis par `SETTLEMENT_STATUSES` : `SCHEDULED`, `PROCESSING`, `PAID`, `FAILED`, `HELD`, `CANCELLED`.

Les transitions de statut doivent être pilotées côté serveur, auditées et soumises à approbation lorsque le niveau de risque l’exige.

## 4. Sécurité et contrôle

- Authentification obligatoire sur toutes les routes.
- Autorisation par rôle, organisation, point de vente, pays et environnement.
- Authentification renforcée pour les invitations sensibles et changements de propriétaire.
- Double validation pour les changements de destination de règlement et opérations à risque.
- Journal d’audit avec acteur, ressource, portée, ancienne valeur, nouvelle valeur et corrélation.
- Idempotence obligatoire pour les créations et invitations.
- Limitation de débit et détection d’abus sur les invitations.
- Aucun secret partenaire, jeton de paiement ou donnée bancaire complète dans les réponses et journaux.

## 5. Cohérence technique

La source canonique est constituée de :

- `MERCHANT_API_ROUTES`
- `MERCHANT_API_METHODS`
- `MerchantApiContract`
- `ListMerchantsQuery`
- `ListMerchantLocationsQuery`
- `ListMerchantMembersQuery`
- `MerchantDashboardQuery`
- `ListSettlementsQuery`
- les types du fichier `merchant.ts`

Le paquet `@mansa/contracts` expose le catalogue via `@mansa/contracts/merchant-api`. Les contrôleurs NestJS et les clients mobiles ou web doivent importer ce contrat au lieu de dupliquer les routes.

## 6. Critères d’acceptation

- Chaque route documentée possède la même méthode et le même chemin dans le catalogue TypeScript.
- La création d’un commerçant est idempotente et associe correctement son propriétaire.
- Un membre ne peut accéder qu’aux organisations et points de vente autorisés.
- Les invitations respectent les règles de délégation et la séparation des tâches.
- Les périodes du tableau de bord sont validées et les montants conservent leur devise.
- Les règlements finalisés ne sont jamais modifiés silencieusement.
- Toute action sensible est auditée et corrélée.
- Les applications utilisent les types partagés du paquet de contrats.
