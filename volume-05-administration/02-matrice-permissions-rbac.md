# Matrice des permissions RBAC et contrôles sensibles

## 1. Objectif

Cette matrice définit le socle d’autorisation commun aux interfaces d’administration, aux API et aux applications internes de Mansa. Elle complète l’authentification et doit être appliquée côté serveur : masquer un bouton dans une interface ne constitue jamais un contrôle d’accès suffisant.

## 2. Principes

- Refus par défaut.
- Moindre privilège.
- Séparation des rôles métier et techniques.
- Double validation pour les opérations à fort impact.
- Portée explicite par pays, entité, partenaire, commerce ou agence.
- Journal d’audit immuable pour toute action sensible.
- Réauthentification forte pour les actions critiques.
- Permissions temporaires avec date d’expiration lorsque nécessaire.

## 3. Rôles de référence

| Rôle | Périmètre principal |
|---|---|
| `SUPER_ADMIN` | Configuration globale, pays, partenaires et gouvernance |
| `COUNTRY_ADMIN` | Administration d’un pays autorisé |
| `OPERATIONS_MANAGER` | Opérations, incidents et supervision des transactions |
| `COMPLIANCE_OFFICER` | KYC, AML, sanctions, dossiers et décisions de conformité |
| `RISK_ANALYST` | Risque, fraude, règles de détection et revues |
| `FINANCE_OPERATOR` | Rapprochements, règlements, commissions et exports financiers |
| `SUPPORT_AGENT` | Assistance client avec accès limité aux données nécessaires |
| `MERCHANT_MANAGER` | Gestion des commerçants et terminaux autorisés |
| `PUBLIC_SERVICE_AGENT` | Services publics dans une entité et une zone définies |
| `AUDITOR` | Consultation des journaux, décisions et preuves sans modification |
| `DEVELOPER_READONLY` | Diagnostic technique non productif et données masquées |

## 4. Permissions normalisées

Les permissions utilisent la forme `domaine.ressource.action`.

Exemples :

- `identity.user.read`
- `identity.user.suspend`
- `compliance.kyc.review`
- `payments.transaction.read`
- `payments.transaction.refund.request`
- `payments.transaction.refund.approve`
- `ledger.entry.read`
- `merchant.profile.update`
- `configuration.fee.update`
- `public-service.fine.issue`
- `audit.event.export`

## 5. Matrice minimale

| Domaine / action | Super Admin | Country Admin | Operations | Conformité | Risque | Finance | Support | Audit |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| Consulter un utilisateur | Oui | Portée pays | Oui | Oui | Oui | Limité | Limité | Oui |
| Modifier les données d’identité | Non direct | Non direct | Non | Workflow | Non | Non | Non | Non |
| Suspendre un compte | Approbation | Demande/approbation | Demande | Demande/approbation | Demande | Non | Demande limitée | Non |
| Examiner un dossier KYC | Non | Lecture | Lecture | Oui | Lecture | Non | Lecture masquée | Oui |
| Approuver ou refuser un KYC | Non | Non | Non | Oui | Non | Non | Non | Non |
| Consulter une transaction | Oui | Portée pays | Oui | Oui | Oui | Oui | Limitée | Oui |
| Demander un remboursement | Oui | Oui | Oui | Non | Non | Oui | Selon seuil | Non |
| Approuver un remboursement | Double contrôle | Selon seuil | Selon seuil | Non | Non | Selon seuil | Non | Non |
| Modifier une commission | Double contrôle | Demande | Non | Non | Non | Demande | Non | Non |
| Modifier une limite produit | Double contrôle | Demande | Non | Avis | Avis | Avis | Non | Non |
| Consulter le ledger | Oui | Portée pays | Lecture | Lecture | Lecture | Oui | Non | Oui |
| Corriger le ledger | Jamais directement | Jamais | Jamais | Jamais | Jamais | Jamais | Jamais | Jamais |
| Exporter des données personnelles | Double contrôle | Limité | Limité | Oui | Limité | Limité | Non | Selon mandat |
| Consulter l’audit | Oui | Portée pays | Limité | Oui | Oui | Oui | Non | Oui |
| Modifier ou supprimer l’audit | Jamais | Jamais | Jamais | Jamais | Jamais | Jamais | Jamais | Jamais |

## 6. Contrôles ABAC complémentaires

Une permission RBAC n’autorise l’action que si les attributs suivants sont également valides :

- pays et entité de rattachement ;
- environnement (`demo`, `staging`, `production`) ;
- devise et produit ;
- montant et seuil d’autorisation ;
- niveau de risque du compte ou de la transaction ;
- état du KYC ;
- horaire, adresse réseau et terminal de confiance ;
- absence de conflit d’intérêts ;
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
- opération manuelle de compensation ou de règlement.

Le demandeur ne peut pas approuver sa propre demande.

## 8. Journal d’audit obligatoire

Chaque action sensible enregistre :

- identifiant de l’acteur et rôle effectif ;
- permission évaluée ;
- portée et attributs utilisés ;
- ressource concernée ;
- état avant et après, avec masquage des secrets ;
- justification saisie ;
- identifiant de corrélation ;
- date, terminal, adresse réseau et environnement ;
- résultat de l’action ;
- approbateurs lorsque la double validation s’applique.

## 9. Critères d’acceptation

1. Une API refuse une action sans permission même si l’interface l’affiche par erreur.
2. Un administrateur d’un pays ne peut pas accéder aux ressources d’un autre pays.
3. Le demandeur d’une opération à double validation ne peut pas l’approuver.
4. Une permission expirée est refusée immédiatement.
5. Toute action critique produit un événement d’audit corrélé.
6. Les exports masquent ou excluent les champs non autorisés.
7. Les tests automatisés couvrent au minimum les refus inter-pays, l’escalade de privilèges et la séparation des tâches.
