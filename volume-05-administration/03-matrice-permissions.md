# Matrice des rôles et permissions

## 1. Objet

Cette matrice définit les droits minimaux attribués aux rôles de la plateforme Mansa. Elle constitue la référence fonctionnelle du fichier `packages/security/src/role-policy.ts` du dépôt `mansa-platform`.

Les droits effectifs restent limités par le périmètre de l’acteur : pays, organisation, commerce, magasin ou agence. Ils peuvent aussi être réduits par des règles ABAC, les limites de montant, le niveau de risque, l’environnement et les mécanismes de double validation.

## 2. Règles générales

- Le moindre privilège est obligatoire.
- Aucun rôle ne contourne les contrôles de périmètre.
- Un utilisateur ne peut pas approuver sa propre opération sensible.
- Les comptes de service reçoivent uniquement des permissions explicites lors du déploiement.
- Toute modification de rôle, permission ou périmètre est auditée.
- La consultation de documents KYC sensibles est réservée aux rôles autorisés.
- Les opérations de grand livre, remboursement, gel de compte et configuration critique peuvent exiger une double validation.

## 3. Rôles plateforme

### SUPER_ADMIN

Accès transversal à l’administration de la plateforme. Ce rôle peut gérer les utilisateurs, le KYC, les comptes, les paiements, le grand livre, les commerçants, les configurations, les partenaires, les services publics et l’audit.

Contraintes :

- usage nominatif uniquement ;
- authentification multifacteur renforcée ;
- session courte ;
- accès privilégié temporaire en Production ;
- double validation pour les ajustements financiers et changements critiques.

### PLATFORM_ADMIN

Administration opérationnelle de la plateforme sans accès automatique aux documents KYC sensibles ni approbation des ajustements de grand livre.

Droits principaux : utilisateurs, comptes, paiements, commerçants, configuration, partenaires, audit, support et lecture des services publics.

### COMPLIANCE_OFFICER

Gestion de la conformité et du KYC.

Droits principaux : lecture utilisateur, revue et décision KYC, lecture de documents sensibles, gel de compte, consultation des paiements, audit et export contrôlé.

### RISK_ANALYST

Analyse du risque et de la fraude.

Droits principaux : lecture des comptes, paiements, grand livre et commerçants, gel de compte, audit et export contrôlé.

### FINANCE_OPERATOR

Exploitation financière et règlements commerçants.

Droits principaux : consultation des comptes et paiements, remboursement, lecture du grand livre, initiation d’ajustement, consultation des règlements et export.

Ce rôle ne peut jamais approuver son propre ajustement.

### SUPPORT_AGENT

Traitement des demandes clients et commerçants.

Droits principaux : lecture limitée des profils, comptes, paiements et commerces, lecture et mise à jour des dossiers de support.

Ce rôle ne peut pas lire les documents KYC sensibles, modifier le grand livre ou déclencher un remboursement.

### AUDITOR

Accès en lecture aux données nécessaires à l’audit.

Droits principaux : utilisateurs, KYC, comptes, paiements, grand livre, commerçants, règlements, configuration, journaux d’audit, support, services publics et exports.

Aucune permission d’écriture métier ne doit être attribuée par défaut.

## 4. Rôles commerçant

### MERCHANT_OWNER

Gestion du commerce, des employés, terminaux, règlements, ventes, remboursements, clôtures de caisse et dossiers de support dans le périmètre de son commerce.

### MERCHANT_MANAGER

Gestion opérationnelle du commerce, des employés, terminaux, ventes, remboursements, clôtures et support. Les changements de propriété et de règlement sensibles restent réservés au propriétaire ou à l’administration.

### MERCHANT_CASHIER

Encaissement, remboursement autorisé et clôture de caisse dans le magasin attribué.

Le remboursement peut être soumis à un plafond et à l’approbation d’un responsable.

## 5. Rôles services publics

### PUBLIC_AGENCY_ADMIN

Administration des services d’une agence publique : configuration, création et annulation de dossiers, encaissement, gestion des agents, audit et exports dans le périmètre de l’agence.

### PUBLIC_AGENT

Création de dossiers et collecte de paiements pour les services publics autorisés.

Ce rôle ne configure pas les tarifs, ne gère pas les agents et ne peut annuler un dossier après paiement sans procédure contrôlée.

## 6. Rôles client et technique

### CUSTOMER

Consultation de ses propres comptes et paiements, création de paiement et gestion de ses dossiers de support.

La règle de propriété interdit l’accès aux ressources d’un autre client.

### SERVICE_ACCOUNT

Aucun droit par défaut. Chaque compte technique reçoit une liste minimale de permissions, un périmètre, une durée de validité, une rotation de secret ou certificat et un journal d’utilisation.

## 7. Permissions sensibles

Les permissions suivantes exigent des contrôles renforcés :

- `identity.document.read_sensitive` : justification, journalisation et masquage des données ;
- `account.freeze` : motif obligatoire et notification selon la procédure applicable ;
- `payment.refund` : plafond, idempotence et référence à la transaction d’origine ;
- `ledger.adjustment.initiate` : pièce justificative et séparation des tâches ;
- `ledger.adjustment.approve` : approbateur différent de l’initiateur ;
- `configuration.update` : versionnement, prévisualisation et possibilité de retour arrière ;
- `fee_rule.manage` et `limit_rule.manage` : date d’effet, historique et double validation en Production ;
- `partner.manage` : validation de sécurité et conformité avant activation ;
- `data.export` : finalité, périmètre, expiration et traçabilité ;
- `public_case.cancel` : motif, autorité compétente et conservation de l’historique.

## 8. Contrôles ABAC obligatoires

L’autorisation finale doit vérifier au minimum :

1. la permission demandée ;
2. le pays ;
3. l’organisation ;
4. le commerce et le magasin ;
5. l’agence publique ;
6. la propriété de la ressource pour un client ;
7. l’environnement ;
8. le niveau de risque ;
9. le montant ;
10. la nécessité d’une double validation.

## 9. Critères d’acceptation

- Chaque rôle du code est documenté ici.
- Chaque permission du code est attribuée explicitement ou laissée volontairement sans rôle par défaut.
- Un client ne peut accéder qu’à ses propres ressources.
- Un acteur hors périmètre reçoit un refus `SCOPE_MISMATCH`.
- Une permission absente produit `MISSING_PERMISSION`.
- Une opération à double validation sans approbateur produit `DUAL_APPROVAL_REQUIRED`.
- L’auto-approbation produit `SELF_APPROVAL_FORBIDDEN`.
- Toute décision sensible est corrélée à un journal d’audit.
