# Matrice RBAC/ABAC et séparation des responsabilités

## 1. Objectif

Cette spécification définit le socle d’autorisation commun aux applications Mansa. Le contrôle d’accès combine :

- **RBAC** : permissions attribuées par rôle ;
- **ABAC** : restrictions selon le pays, l’organisation, le point de vente, l’environnement, le propriétaire de la ressource, le niveau de risque et le montant ;
- **séparation des responsabilités** : certaines opérations nécessitent un initiateur et un approbateur distincts ;
- **audit obligatoire** : toute décision d’autorisation sensible doit être traçable.

Le refus est la règle par défaut. Une permission absente, un contexte incomplet ou une portée incompatible entraîne un refus.

## 2. Rôles de référence

| Rôle | Portée habituelle | Finalité |
|---|---|---|
| `SUPER_ADMIN` | Plateforme | Paramétrage global, pays, partenaires et gouvernance |
| `PLATFORM_ADMIN` | Plateforme ou pays | Exploitation courante sans accès automatique aux secrets |
| `COMPLIANCE_OFFICER` | Pays/entité | KYC, LCB-FT, sanctions et dossiers réglementaires |
| `RISK_ANALYST` | Pays/entité | Fraude, scoring, blocages et limites |
| `FINANCE_OPERATOR` | Entité | Rapprochement, règlements, exports et opérations financières |
| `SUPPORT_AGENT` | Entité | Assistance client avec accès limité aux données sensibles |
| `AUDITOR` | Portée attribuée | Lecture seule des journaux, décisions et preuves |
| `MERCHANT_OWNER` | Commerce | Gestion du commerce, employés, terminaux et règlements |
| `MERCHANT_MANAGER` | Commerce/point de vente | Exploitation d’un ou plusieurs points de vente |
| `MERCHANT_CASHIER` | Point de vente | Encaissement, remboursement limité et clôture de caisse |
| `PUBLIC_AGENCY_ADMIN` | Administration publique | Paramétrage d’un service public et gestion des agents |
| `PUBLIC_AGENT` | Service/zone | Constat, émission ou encaissement selon mandat |
| `CUSTOMER` | Ressources propres | Utilisation des fonctions client autorisées |
| `SERVICE_ACCOUNT` | Intégration précise | Appels machine-à-machine strictement limités |

Un rôle ne constitue jamais, à lui seul, une autorisation suffisante pour une action à risque élevé.

## 3. Catalogue minimal de permissions

### Identité et conformité

- `identity.user.read`
- `identity.user.update`
- `identity.user.suspend`
- `identity.kyc.read`
- `identity.kyc.review`
- `identity.kyc.approve`
- `identity.kyc.reject`
- `identity.document.read_sensitive`

### Comptes et paiements

- `account.read`
- `account.freeze`
- `payment.read`
- `payment.create`
- `payment.refund`
- `payment.cancel`
- `ledger.read`
- `ledger.adjustment.initiate`
- `ledger.adjustment.approve`

### Commerce et TPE

- `merchant.read`
- `merchant.update`
- `merchant.employee.manage`
- `merchant.terminal.manage`
- `merchant.settlement.read`
- `merchant.settlement.configure`
- `pos.sale.create`
- `pos.refund.create`
- `pos.shift.close`

### Administration et exploitation

- `configuration.read`
- `configuration.update`
- `feature_flag.manage`
- `fee_rule.manage`
- `limit_rule.manage`
- `partner.manage`
- `audit.read`
- `support.case.read`
- `support.case.update`
- `data.export`

### Services publics

- `public_service.read`
- `public_service.configure`
- `public_case.create`
- `public_case.cancel`
- `public_payment.collect`
- `public_agent.manage`

## 4. Attributs de contexte obligatoires

La décision d’autorisation peut utiliser :

- `actorId`, `actorType`, rôles et permissions effectives ;
- `countryCode`, `organizationId`, `merchantId`, `storeId`, `agencyId` ;
- `environment` : `DEMO`, `STAGING` ou `PRODUCTION` ;
- propriétaire de la ressource ;
- montant en unité mineure et devise ;
- niveau de risque de l’acteur, de la session et de l’opération ;
- état KYC ;
- terminal, adresse réseau, appareil et canal ;
- plage horaire ou zone géographique lorsque le mandat l’exige ;
- indicateur d’approbation et identité de l’approbateur.

Les attributs fournis par le client ne sont jamais considérés comme fiables sans résolution côté serveur.

## 5. Règles de portée

1. Un acteur limité à un pays ne peut accéder à aucune ressource d’un autre pays.
2. Un employé de commerce ne peut agir que sur les commerces et points de vente auxquels il est affecté.
3. Un client ne peut lire ou modifier que ses propres ressources, sauf délégation explicite et révocable.
4. Un agent public ne peut intervenir que pour son administration, son service et sa zone de mandat.
5. Un compte de service ne reçoit aucune permission interactive et ne peut élargir sa propre portée.
6. La portée Production doit être accordée explicitement ; une permission Démo ou Recette ne s’y transpose pas.

## 6. Actions à double validation

Les opérations suivantes nécessitent deux personnes distinctes lorsque les seuils configurés sont atteints :

- ajustement manuel du grand livre ;
- changement de règles de frais ou de limites en Production ;
- activation d’un partenaire financier ;
- export massif de données sensibles ;
- déblocage d’un compte à risque élevé ;
- remboursement supérieur au seuil du commerce ;
- annulation d’une opération publique déjà constatée ;
- modification d’une politique d’autorisation.

L’initiateur ne peut pas approuver sa propre demande. L’approbation expire après une durée configurable et devient invalide si la demande change.

## 7. Matrice synthétique

| Action | Client | Caissier | Propriétaire commerce | Support | Conformité | Finance | Admin plateforme | Auditeur |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| Lire ses propres paiements | Oui | Non | Selon commerce | Limité | Oui | Oui | Oui | Oui |
| Créer une vente TPE | Non | Oui | Oui | Non | Non | Non | Non | Non |
| Rembourser une vente | Non | Seuil limité | Oui | Non | Non | Selon mandat | Non | Lecture |
| Examiner un dossier KYC | Non | Non | Non | Lecture masquée | Oui | Non | Administration limitée | Lecture |
| Approuver un KYC | Non | Non | Non | Non | Oui | Non | Non par défaut | Lecture |
| Ajuster le ledger | Non | Non | Non | Non | Non | Initier/approuver séparément | Selon mandat | Lecture |
| Modifier les frais | Non | Non | Non | Non | Consultation | Consultation | Double validation | Lecture |
| Lire les audits | Ses événements exposés | Commerce limité | Commerce limité | Dossiers liés | Oui | Oui | Oui | Oui |

Cette table est un résumé. Les permissions et attributs précis priment.

## 8. Journalisation

Chaque décision sensible enregistre au minimum :

- identifiant de corrélation ;
- acteur, session et rôles évalués ;
- permission demandée ;
- ressource et portée ;
- décision `ALLOW` ou `DENY` ;
- règle ou motif principal ;
- date, environnement et canal ;
- identifiant de demande d’approbation, le cas échéant.

Les journaux ne doivent pas contenir de secret, de code PIN, de jeton complet ni de document KYC brut.

## 9. Critères d’acceptation

- Une permission inconnue est refusée.
- Un rôle sans permission explicite est refusé.
- Une portée pays, organisation ou commerce incompatible est refusée.
- Un client ne peut jamais fournir lui-même une portée lui donnant davantage de droits.
- Une action à double validation est refusée sans approbateur distinct.
- Les décisions sensibles produisent un événement d’audit corrélé.
- Les tests couvrent les cas autorisés et refusés, notamment les accès inter-pays et inter-commerces.

## 10. Correspondance avec le code

Le catalogue canonique et les primitives d’évaluation sont implémentés dans `mansa-platform/packages/security`. Les applications et services ne doivent pas redéfinir localement des chaînes de permissions concurrentes.

Les fichiers de référence sont :

- `packages/security/src/index.ts` pour les rôles et permissions ;
- `packages/security/src/role-policy.ts` pour l’attribution minimale par rôle ;
- `packages/security/src/permission-policy.ts` pour le domaine, la sensibilité, le motif obligatoire et la double approbation en Production.

## 11. Politique canonique des actions critiques

Les permissions suivantes exigent systématiquement une double approbation en Production :

- `ledger.adjustment.approve` ;
- `merchant.settlement.configure` ;
- `feature_flag.manage` ;
- `fee_rule.manage` ;
- `limit_rule.manage` ;
- `partner.manage` ;
- `public_service.configure`.

Toute permission marquée `productionDualApproval` doit également être marquée `sensitive` et `requiresReason`. L’initiateur et l’approbateur doivent être distincts, autorisés et dans un périmètre compatible.

Le motif est notamment obligatoire pour les suspensions, décisions KYC, lectures de documents sensibles, gels de comptes, remboursements, annulations, ajustements, modifications de configuration, exports et annulations de dossiers publics.

Les tests `role-policy.test.ts` et `permission-policy.test.ts` garantissent que :

- chaque rôle et chaque permission possèdent une politique explicite ;
- aucune permission inconnue ou dupliquée n’est accordée ;
- les comptes de service restent sans privilège implicite ;
- la séparation des tâches financières est maintenue ;
- toute opération à double approbation est sensible et exige un motif.
