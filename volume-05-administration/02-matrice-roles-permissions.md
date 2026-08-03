# Matrice des rôles et permissions

## Objectif

Cette matrice définit le socle RBAC/ABAC commun à toutes les applications Mansa. Elle doit rester alignée avec `packages/security/src/index.ts` dans `mansa-platform`.

## Principes

- Une permission autorise une capacité précise ; un rôle regroupe plusieurs permissions.
- Les permissions seules ne suffisent pas : le périmètre doit aussi correspondre au pays, à l’organisation, au commerçant, au point de vente ou à l’agence.
- Un client ne peut agir que sur ses propres ressources.
- Toute opération critique peut imposer une double validation.
- L’initiateur ne peut jamais être son propre approbateur.
- Les accès aux données sensibles, exports et ajustements comptables sont journalisés.
- Les comptes de service disposent uniquement des permissions techniques nécessaires à leur fonction.

## Rôles de référence

| Rôle | Finalité | Périmètre attendu |
|---|---|---|
| `SUPER_ADMIN` | Administration globale exceptionnelle | Plateforme entière, accès fortement contrôlé |
| `PLATFORM_ADMIN` | Configuration opérationnelle de la plateforme | Pays et organisations autorisés |
| `COMPLIANCE_OFFICER` | Revue KYC, conformité et décisions réglementaires | Pays ou entité de conformité |
| `RISK_ANALYST` | Analyse risque, fraude et surveillance | Pays, portefeuille ou partenaire |
| `FINANCE_OPERATOR` | Règlements, rapprochements et opérations financières | Organisation et comptes autorisés |
| `SUPPORT_AGENT` | Traitement des demandes clients et commerçants | Files et pays affectés |
| `AUDITOR` | Consultation des journaux et preuves | Lecture seule, périmètre d’audit |
| `MERCHANT_OWNER` | Gestion complète d’un commerce | Commerçant propriétaire |
| `MERCHANT_MANAGER` | Gestion opérationnelle d’un commerce | Commerçant ou magasins affectés |
| `MERCHANT_CASHIER` | Vente, encaissement et clôture de caisse | Magasin ou terminal affecté |
| `PUBLIC_AGENCY_ADMIN` | Configuration d’une administration publique | Agence publique autorisée |
| `PUBLIC_AGENT` | Création de dossiers et encaissement public | Agence et service affectés |
| `CUSTOMER` | Utilisation de l’application client | Ressources appartenant au client |
| `SERVICE_ACCOUNT` | Intégrations et traitements automatisés | Portée technique minimale |

## Matrice minimale

Légende : `A` autorisé par défaut, `C` autorisé sous condition de périmètre, `D` double validation requise, `—` non attribué par défaut.

| Domaine / permission | Super admin | Platform admin | Conformité | Risque | Finance | Support | Auditeur | Merchant owner | Manager | Cashier | Agency admin | Public agent | Client | Service |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| `identity.user.read` | A | C | C | C | — | C | C | C | C | — | C | C | C | C |
| `identity.user.update` | D | C | — | — | — | C | — | C | C | — | C | — | C | C |
| `identity.user.suspend` | D | D | C | C | — | — | — | C | — | — | C | — | — | — |
| `identity.kyc.read` | A | C | C | C | — | C | C | — | — | — | — | — | C | C |
| `identity.kyc.review` | D | — | C | C | — | — | — | — | — | — | — | — | — | — |
| `identity.kyc.approve` | D | — | D | — | — | — | — | — | — | — | — | — | — | — |
| `identity.kyc.reject` | D | — | D | — | — | — | — | — | — | — | — | — | — | — |
| `identity.document.read_sensitive` | D | — | C | C | — | — | C | — | — | — | — | — | C | — |
| `account.read` | A | C | C | C | C | C | C | C | C | C | C | C | C | C |
| `account.freeze` | D | D | C | C | C | — | — | C | — | — | C | — | — | C |
| `payment.read` | A | C | C | C | C | C | C | C | C | C | C | C | C | C |
| `payment.create` | — | — | — | — | C | — | — | C | C | C | — | C | C | C |
| `payment.refund` | D | D | — | C | C | C | C | C | C | C | — | C | — | C |
| `payment.cancel` | D | C | — | C | C | C | C | C | C | C | C | C | C | C |
| `ledger.read` | A | C | C | C | C | — | C | C | C | — | C | — | C | C |
| `ledger.adjustment.initiate` | D | — | — | — | D | — | — | — | — | — | — | — | — | C |
| `ledger.adjustment.approve` | D | — | — | — | D | — | C | — | — | — | — | — | — | — |
| `merchant.employee.manage` | D | C | — | — | — | — | — | C | C | — | — | — | — | C |
| `merchant.terminal.manage` | D | C | — | C | — | C | — | C | C | — | — | — | — | C |
| `merchant.settlement.configure` | D | D | — | C | D | — | C | C | — | — | — | — | — | C |
| `pos.sale.create` | — | — | — | — | — | — | — | C | C | C | — | C | — | C |
| `pos.refund.create` | — | — | — | C | C | C | — | C | C | C | — | C | — | C |
| `pos.shift.close` | — | — | — | — | C | — | C | C | C | C | — | C | — | C |
| `configuration.update` | D | D | — | — | — | — | C | — | — | — | C | — | — | C |
| `feature_flag.manage` | D | D | — | C | — | — | C | — | — | — | — | — | — | C |
| `fee_rule.manage` | D | D | — | C | D | — | C | — | — | — | C | — | — | C |
| `limit_rule.manage` | D | D | C | C | D | — | C | — | — | — | C | — | — | C |
| `partner.manage` | D | D | C | C | C | — | C | — | — | — | C | — | — | C |
| `audit.read` | A | C | C | C | C | — | C | C | C | — | C | — | C | C |
| `support.case.read` | A | C | C | C | C | C | C | C | C | — | C | C | C | C |
| `support.case.update` | D | C | — | — | — | C | — | C | C | — | C | C | C | C |
| `data.export` | D | D | C | C | C | — | C | C | — | — | C | — | C | C |
| `public_service.configure` | D | C | — | C | C | — | C | — | — | — | C | — | — | C |
| `public_case.create` | — | — | — | — | — | — | — | — | — | — | C | C | — | C |
| `public_case.cancel` | D | C | — | C | C | C | C | — | — | — | C | C | C | C |
| `public_payment.collect` | — | — | — | — | C | — | C | — | — | — | C | C | C | C |
| `public_agent.manage` | D | C | — | C | — | — | C | — | — | — | C | — | — | C |

## Règles ABAC

Une autorisation doit être refusée dès qu’un attribut requis ne correspond pas :

1. `countryCode` pour le pays.
2. `organizationId` pour l’entreprise ou institution.
3. `merchantId` pour le commerçant.
4. `storeId` pour le magasin.
5. `agencyId` pour l’administration publique.
6. `resourceOwnerId` pour une ressource appartenant à un client.

Un attribut absent de la ressource n’impose pas de contrainte supplémentaire. Un attribut présent doit correspondre exactement au périmètre de l’acteur.

## Actions à double validation

La double validation est obligatoire au minimum pour :

- ajustement du grand livre ;
- modification des frais ou limites en Production ;
- gel ou dégel manuel d’un compte sensible ;
- approbation KYC à risque élevé ;
- modification d’un partenaire financier ;
- export massif de données ;
- remboursement supérieur au seuil configuré ;
- activation ou désactivation globale d’une fonctionnalité critique.

## Critères d’acceptation

- Une permission absente entraîne `MISSING_PERMISSION`.
- Un périmètre incompatible entraîne `SCOPE_MISMATCH`.
- Un client visant la ressource d’un tiers reçoit `OWNER_MISMATCH`.
- Une opération exigeant une deuxième personne sans approbateur reçoit `DUAL_APPROVAL_REQUIRED`.
- L’auto-approbation reçoit `SELF_APPROVAL_FORBIDDEN`.
- Les décisions d’autorisation sont testées automatiquement dans `packages/security/src/authorization.test.ts`.
