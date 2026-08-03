# Matrice des rôles et permissions

## 1. Objet

Cette matrice définit les droits de base attribués aux rôles Mansa. Elle correspond au contrat technique `packages/security/src/role-policy.ts` du dépôt `mansa-platform`.

Les permissions attribuées par rôle ne suffisent jamais seules à autoriser une action. L’autorisation finale combine :

- les permissions RBAC du rôle ;
- le périmètre ABAC de l’acteur (`countryCode`, `organizationId`, `merchantId`, `storeId`, `agencyId`) ;
- la propriété de la ressource pour un client ;
- le niveau de risque ;
- les limites métier et financières ;
- la double validation lorsqu’elle est exigée.

## 2. Rôles plateforme

### `SUPER_ADMIN`

Rôle exceptionnel de gouvernance globale. Il peut administrer la plateforme, les partenaires, la configuration, les fonctions publiques et les règles financières. Les opérations critiques restent soumises à double validation et audit.

### `PLATFORM_ADMIN`

Administre les utilisateurs, comptes, commerçants, terminaux, fonctions et configurations dans son périmètre. Il ne dispose pas des droits d’approbation KYC ni d’ajustement comptable final par défaut.

### `COMPLIANCE_OFFICER`

Consulte et traite les dossiers KYC, accède aux documents sensibles, peut geler un compte et exporter les données nécessaires aux contrôles de conformité.

### `RISK_ANALYST`

Analyse utilisateurs, comptes, paiements, ledger et commerçants. Il peut geler un compte selon les procédures de risque mais ne modifie pas la configuration métier.

### `FINANCE_OPERATOR`

Consulte les comptes, paiements, règlements et le ledger. Il peut initier un ajustement comptable ou un remboursement, mais ne peut pas approuver son propre ajustement.

### `SUPPORT_AGENT`

Accède uniquement aux données nécessaires au traitement d’un dossier support. Il ne peut ni lire les documents KYC sensibles, ni modifier le ledger, ni configurer des commissions.

### `AUDITOR`

Rôle majoritairement en lecture : identité, KYC, comptes, paiements, ledger, commerçants, configuration, audit, services publics et exports. Aucune mutation financière ou opérationnelle n’est accordée par défaut.

## 3. Rôles commerçant

### `MERCHANT_OWNER`

Gère son commerce, ses employés, ses terminaux, ses règlements et les opérations TPE autorisées dans son propre périmètre.

### `MERCHANT_MANAGER`

Gère les employés et terminaux du ou des points de vente qui lui sont affectés. Il consulte les règlements et réalise les opérations de vente, remboursement et clôture de caisse autorisées.

### `MERCHANT_CASHIER`

Réalise les ventes, remboursements permis et clôtures de caisse sur le magasin et le terminal affectés. Il ne gère ni les employés, ni les paramètres de règlement.

## 4. Rôles services publics

### `PUBLIC_AGENCY_ADMIN`

Configure les services de son organisme, gère les agents, crée ou annule des dossiers publics, supervise les encaissements et accède aux exports et journaux de son agence.

### `PUBLIC_AGENT`

Consulte les services disponibles, crée les dossiers autorisés et collecte les paiements. Il ne peut ni modifier les barèmes, ni gérer les autres agents, ni annuler librement une opération finalisée.

## 5. Rôles externes

### `CUSTOMER`

Consulte ses propres comptes et paiements, initie les paiements permis et gère ses dossiers support. Toute tentative d’accès à la ressource d’un autre client doit être refusée.

### `SERVICE_ACCOUNT`

Aucun droit implicite. Chaque compte de service reçoit uniquement des permissions explicites, limitées à une intégration et un environnement précis.

## 6. Séparation des responsabilités

Les combinaisons suivantes exigent au minimum deux acteurs distincts :

- initiation et approbation d’un ajustement ledger ;
- création et activation d’une règle de commission sensible ;
- modification des limites de production au-dessus d’un seuil défini ;
- remboursement exceptionnel et approbation de ce remboursement ;
- activation d’un nouveau partenaire financier en production ;
- export massif de données sensibles et validation de l’export.

Un acteur ne peut jamais être son propre approbateur.

## 7. Règles de périmètre

1. Un rôle commerçant est toujours limité à un `merchantId`, et si nécessaire à un `storeId`.
2. Un rôle public est toujours limité à une `agencyId` et à un pays.
3. Un administrateur régional est limité à un `countryCode` ou à une organisation.
4. Un client ne peut agir que sur ses propres ressources, sauf mandat formel géré par un module dédié.
5. Une permission accordée sans périmètre obligatoire doit être rejetée lors de la création de l’acteur.

## 8. Audit minimum

Chaque décision sensible doit enregistrer : acteur, rôles, permission demandée, périmètre, ressource, environnement, résultat, motif du refus, identifiant de corrélation, date, appareil ou service appelant et approbateur éventuel.

## 9. Critères d’acceptation

- Chaque rôle déclaré dans le code possède une entrée dans la politique de rôles.
- Aucune permission inconnue ne peut être attribuée.
- Les permissions de plusieurs rôles sont fusionnées sans doublon.
- Les contraintes de périmètre restent appliquées après fusion des rôles.
- `SERVICE_ACCOUNT` ne reçoit aucun droit implicite.
- Un client ne peut pas lire ou modifier la ressource d’un autre client.
- Une opération à double validation est refusée sans approbateur distinct.
- Les changements de politique sont revus conjointement par sécurité, conformité et propriétaire métier.

## 10. Référence technique

La liste canonique des rôles et permissions demeure déclarée dans `packages/security/src/index.ts`. La table `ROLE_PERMISSIONS`, la fonction `permissionsForRoles` et la fonction `roleHasPermission` sont définies dans `packages/security/src/role-policy.ts`.
