# Matrice des permissions RBAC et contrôles sensibles

## 1. Objectif

Cette matrice définit le socle d’autorisation commun aux interfaces d’administration, aux API et aux applications internes de Mansa. Elle complète l’authentification et doit être appliquée côté serveur : masquer un bouton dans une interface ne constitue jamais un contrôle d’accès suffisant.

La source exécutable correspondante se trouve dans `mansa-platform/packages/security/src/index.ts` et `mansa-platform/packages/security/src/role-policy.ts`.

## 2. Principes

- Refus par défaut.
- Moindre privilège.
- Séparation des rôles métier et techniques.
- Double validation pour les opérations à fort impact.
- Portée explicite par pays, organisation, commerce, magasin ou agence.
- Journal d’audit immuable pour toute action sensible.
- Réauthentification forte pour les actions critiques.
- Permissions temporaires avec date d’expiration lorsque nécessaire.

## 3. Rôles de référence

Les identifiants ci-dessous doivent rester strictement alignés avec le type `Role` du package `@mansa/security`.

| Rôle | Périmètre principal |
|---|---|
| `SUPER_ADMIN` | Configuration globale, partenaires et gouvernance |
| `PLATFORM_ADMIN` | Administration opérationnelle de la plateforme dans une portée autorisée |
| `COMPLIANCE_OFFICER` | KYC, conformité, sanctions et décisions réglementaires |
| `RISK_ANALYST` | Risque, fraude, surveillance et revues |
| `FINANCE_OPERATOR` | Rapprochements, règlements, remboursements et exports financiers |
| `SUPPORT_AGENT` | Assistance client avec accès limité aux données nécessaires |
| `AUDITOR` | Consultation des journaux, décisions et preuves sans modification |
| `MERCHANT_OWNER` | Propriétaire d’un commerce et gestion de ses ressources |
| `MERCHANT_MANAGER` | Gestion déléguée d’un commerce, de ses employés et terminaux |
| `MERCHANT_CASHIER` | Encaissement, remboursement autorisé et clôture de caisse |
| `PUBLIC_AGENCY_ADMIN` | Administration d’une agence ou entité publique |
| `PUBLIC_AGENT` | Création de dossiers et collecte de paiements publics autorisés |
| `CUSTOMER` | Accès du client à ses propres comptes, paiements et demandes de support |
| `SERVICE_ACCOUNT` | Identité technique sans permission implicite ; droits accordés explicitement |

La restriction par pays n’est pas un rôle séparé : elle est portée par `AuthorizationScope.countryCode` et les autres attributs de portée.

## 4. Permissions normalisées

Les permissions utilisent les identifiants exacts du package de sécurité. Principaux groupes :

- identité : `identity.user.read`, `identity.user.update`, `identity.user.suspend`, `identity.kyc.read`, `identity.kyc.review`, `identity.kyc.approve`, `identity.kyc.reject`, `identity.document.read_sensitive` ;
- comptes et paiements : `account.read`, `account.freeze`, `payment.read`, `payment.create`, `payment.refund`, `payment.cancel` ;
- ledger : `ledger.read`, `ledger.adjustment.initiate`, `ledger.adjustment.approve` ;
- commerçants et TPE : `merchant.read`, `merchant.update`, `merchant.employee.manage`, `merchant.terminal.manage`, `merchant.settlement.read`, `merchant.settlement.configure`, `pos.sale.create`, `pos.refund.create`, `pos.shift.close` ;
- configuration : `configuration.read`, `configuration.update`, `feature_flag.manage`, `fee_rule.manage`, `limit_rule.manage`, `partner.manage` ;
- audit et support : `audit.read`, `support.case.read`, `support.case.update`, `data.export` ;
- services publics : `public_service.read`, `public_service.configure`, `public_case.create`, `public_case.cancel`, `public_payment.collect`, `public_agent.manage`.

Toute nouvelle permission doit être ajoutée au type `Permission`, attribuée explicitement dans `ROLE_PERMISSIONS`, testée et documentée dans le même lot.

## 5. Matrice métier minimale

| Domaine / action | Super Admin | Platform Admin | Conformité | Risque | Finance | Support | Audit |
|---|---:|---:|---:|---:|---:|---:|---:|
| Consulter un utilisateur | Oui | Oui, portée limitée | Oui | Oui | Non | Oui, minimum nécessaire | Oui |
| Modifier un utilisateur | Oui | Oui, portée limitée | Non | Non | Non | Non | Non |
| Suspendre un utilisateur | Oui | Oui, selon workflow | Non | Non | Non | Non | Non |
| Examiner un dossier KYC | Oui | Non | Oui | Lecture | Non | Non | Lecture |
| Approuver ou refuser un KYC | Oui | Non | Oui | Non | Non | Non | Non |
| Geler un compte | Oui | Oui | Oui | Oui | Non | Non | Non |
| Consulter une transaction | Oui | Oui | Oui | Oui | Oui | Oui, limitée | Oui |
| Créer un remboursement | Oui | Non par défaut | Non | Non | Oui | Non | Non |
| Consulter le ledger | Oui | Oui | Lecture | Lecture | Oui | Non | Oui |
| Initier un ajustement ledger | Oui | Non | Non | Non | Oui | Non | Non |
| Approuver un ajustement ledger | Oui, acteur distinct | Non | Non | Non | Non par défaut | Non | Non |
| Modifier une commission | Oui | Oui selon délégation | Non | Non | Non | Non | Non |
| Modifier une limite produit | Oui | Oui selon délégation | Non | Non | Non | Non | Non |
| Exporter des données | Oui | Non par défaut | Oui | Oui | Oui | Non | Oui selon mandat |
| Consulter l’audit | Oui | Oui | Oui | Oui | Non par défaut | Non | Oui |
| Modifier ou supprimer l’audit | Jamais | Jamais | Jamais | Jamais | Jamais | Jamais | Jamais |

Cette table décrit le comportement attendu. La liste exécutable exacte reste `ROLE_PERMISSIONS`.

## 6. Contrôles ABAC complémentaires

Une permission RBAC n’autorise l’action que si les attributs suivants sont également valides :

- pays (`countryCode`) ;
- organisation (`organizationId`) ;
- commerce (`merchantId`) ;
- magasin (`storeId`) ;
- agence publique (`agencyId`) ;
- environnement (`DEMO`, `STAGING`, `PRODUCTION`) ;
- montant en unité mineure et seuil d’autorisation ;
- niveau de risque (`LOW`, `MEDIUM`, `HIGH`, `CRITICAL`) ;
- propriétaire de la ressource pour les clients ;
- règle de séparation entre demandeur et approbateur.

## 7. Double validation

Les actions suivantes exigent au minimum deux acteurs distincts :

- changement de commissions ou de tarification ;
- modification de limites globales ;
- remboursement supérieur au seuil configuré ;
- déblocage d’un compte à risque élevé ;
- export massif de données ;
- activation d’un partenaire de paiement ;
- changement d’une règle de conformité en production ;
- rotation ou révocation d’un secret partenaire ;
- ajustement manuel du ledger ;
- opération manuelle de compensation ou de règlement.

Le demandeur ne peut pas approuver sa propre demande. Le moteur d’autorisation doit renvoyer `DUAL_APPROVAL_REQUIRED` si l’approbateur manque et `SELF_APPROVAL_FORBIDDEN` si les identités sont identiques.

## 8. Journal d’audit obligatoire

Chaque action sensible enregistre :

- identifiant de l’acteur et rôles effectifs ;
- permission évaluée ;
- portée et attributs utilisés ;
- ressource concernée ;
- état avant et après, avec masquage des secrets ;
- justification saisie ;
- identifiant de corrélation ;
- date, terminal, adresse réseau et environnement ;
- résultat de l’action et motif de refus éventuel ;
- approbateurs lorsque la double validation s’applique.

## 9. Critères d’acceptation

1. Une API refuse une action sans permission même si l’interface l’affiche par erreur.
2. Un acteur limité à un pays, une organisation, un commerce, un magasin ou une agence ne peut pas sortir de sa portée.
3. Un client ne peut lire que ses propres ressources lorsque `resourceOwnerId` est évalué.
4. Le demandeur d’une opération à double validation ne peut pas l’approuver.
5. Toute action critique produit un événement d’audit corrélé.
6. Les exports masquent ou excluent les champs non autorisés.
7. Les tests automatisés couvrent au minimum les permissions manquantes, les refus de portée, le refus propriétaire, l’absence de second approbateur et l’auto-approbation.
