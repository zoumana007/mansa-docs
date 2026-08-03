# Matrice RBAC et séparation des tâches

## 1. Objectif

Cette spécification définit les rôles administratifs de Mansa, leurs permissions minimales et les contrôles empêchant qu'une seule personne puisse initier, approuver et dissimuler une opération sensible.

Le contrôle d'accès repose sur trois niveaux complémentaires :

1. **RBAC** : permissions attribuées à des rôles.
2. **Portée** : pays, entité, partenaire, agence, commerce ou portefeuille concerné.
3. **Contexte** : environnement, niveau de risque, montant, heure, appareil et étape d'approbation.

Aucune permission sensible ne doit être accordée directement à un utilisateur. Elle doit toujours provenir d'un rôle versionné et audité.

## 2. Convention de nommage

Les permissions suivent le format `domaine.ressource.action`.

Exemples :

- `identity.user.read`
- `identity.user.suspend`
- `payments.transaction.refund.request`
- `payments.transaction.refund.approve`
- `risk.rule.update`
- `platform.feature-flag.update`
- `audit.event.export`

Les actions standard sont : `create`, `read`, `update`, `delete`, `request`, `approve`, `reject`, `execute`, `export`, `suspend` et `restore`.

## 3. Rôles de référence

| Rôle | Finalité | Portée par défaut |
|---|---|---|
| `SUPER_ADMIN` | Administration technique exceptionnelle | Globale, accès fortement contrôlé |
| `PLATFORM_ADMIN` | Configuration générale de la plateforme | Globale hors secrets et approbations financières |
| `COUNTRY_ADMIN` | Pilotage d'un pays | Un pays |
| `COMPLIANCE_OFFICER` | KYC, AML et obligations réglementaires | Pays ou entité réglementée |
| `RISK_MANAGER` | Règles de risque, fraude et limites | Pays ou produit |
| `FINANCE_OPERATOR` | Rapprochements, règlements et opérations financières | Entité comptable |
| `FINANCE_APPROVER` | Approbation des opérations financières sensibles | Entité comptable |
| `SUPPORT_AGENT` | Assistance client de niveau 1 | File ou pays assigné |
| `SUPPORT_MANAGER` | Escalade et actions support sensibles | Pays ou centre de support |
| `MERCHANT_OPERATIONS` | Gestion des commerçants et terminaux | Réseau ou pays |
| `PUBLIC_SERVICE_OPERATOR` | Traitement des services publics | Administration et service assignés |
| `PUBLIC_SERVICE_APPROVER` | Approbation des annulations et corrections publiques | Administration assignée |
| `AUDITOR` | Consultation et export des preuves | Périmètre d'audit |
| `READ_ONLY_ANALYST` | Lecture des données agrégées | Domaine analytique autorisé |

## 4. Matrice fonctionnelle minimale

Légende : `L` lecture, `D` demande, `A` approbation, `E` exécution, `C` configuration, `—` interdit.

| Domaine | Super admin | Platform admin | Country admin | Conformité | Risque | Finance opérateur | Finance approbateur | Support | Audit |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Utilisateurs et profils | L/C | L/C | L/C | L | L | L | L | L limité | L |
| Suspension d'un compte | A/E | D | D | D/A | D/A | — | — | D | L |
| Dossiers KYC | L | L | L | L/C/A | L | — | — | L masqué | L |
| Règles AML | L | L | L | C/A | L | — | — | — | L |
| Règles de fraude | L | L | L | L | C/A | — | — | — | L |
| Limites produit | L | C | D | L | D/A | L | L | — | L |
| Transactions | L | L | L | L | L | L | L | L limité | L |
| Remboursement | — | — | D | D | D | D/E | A | D sous seuil | L |
| Ajustement de ledger | — | — | — | — | — | D/E | A | — | L |
| Rapprochement | L | L | L | L | L | C/E | A | — | L |
| Commissions et frais | L | C | D | L | L | D | A | — | L |
| Fonctionnalités activables | L | C | D | L | L | — | — | — | L |
| Secrets et clés | Rotation uniquement via système dédié | — | — | — | — | — | — | — | Métadonnées seulement |
| Journaux d'audit | L | L | L | L | L | L | L | L limité | L/export |
| Suppression de journaux | — | — | — | — | — | — | — | — | — |

## 5. Séparation des tâches

Les couples suivants sont incompatibles pour un même utilisateur dans une même portée :

- `FINANCE_OPERATOR` et `FINANCE_APPROVER`.
- `PUBLIC_SERVICE_OPERATOR` et `PUBLIC_SERVICE_APPROVER`.
- création d'une règle de risque et approbation de cette même version.
- demande d'ajustement du grand livre et approbation du même ajustement.
- création d'un barème de commission et approbation de sa mise en production.
- création d'un bénéficiaire de règlement et validation de son premier paiement.
- traitement d'un dossier KYC et approbation finale lorsque le dossier est classé à risque élevé.

Le backend doit refuser l'approbation lorsque `requestedByUserId == approvedByUserId`, même si l'utilisateur possède les deux permissions à la suite d'une erreur de configuration.

## 6. Actions nécessitant une double approbation

Une double approbation indépendante est obligatoire pour :

- ajustement manuel du ledger ;
- remboursement supérieur au seuil configuré ;
- modification rétroactive de frais ou de commissions ;
- désactivation globale d'un contrôle antifraude ;
- changement d'un compte bancaire de règlement ;
- export massif de données personnelles ;
- déblocage d'un compte classé à risque élevé ;
- bascule d'une intégration partenaire vers Production ;
- rotation ou révocation d'une clé de signature de production.

Chaque demande possède un statut : `DRAFT`, `PENDING_APPROVAL`, `APPROVED`, `REJECTED`, `EXECUTED`, `EXPIRED` ou `CANCELLED`.

## 7. Contraintes techniques

Le jeton d'accès ne contient que les identifiants de rôles et de portée nécessaires. Les permissions effectives sont résolues côté serveur afin qu'une révocation soit appliquée sans attendre l'expiration d'un jeton long.

Toute décision d'autorisation enregistre au minimum :

- `actorUserId` ;
- `actorRoleIds` ;
- `tenantId` ;
- `countryCode` ;
- `resourceType` et `resourceId` ;
- `permission` ;
- `decision` ;
- `reasonCode` ;
- `requestId` ;
- `ipAddress` et empreinte d'appareil lorsque disponibles ;
- horodatage UTC.

Les refus d'autorisation sont journalisés pour les actions sensibles et agrégés dans les alertes de sécurité.

## 8. Gestion du cycle de vie des accès

- Les accès temporaires possèdent une date d'expiration obligatoire.
- Les comptes inactifs sont suspendus après la durée configurée.
- Les rôles sensibles sont revus périodiquement par un responsable différent du propriétaire du compte.
- Le départ d'un collaborateur révoque immédiatement sessions, clés, appareils et rôles.
- L'accès d'urgence est limité dans le temps, exige une justification et déclenche une alerte.
- Les comptes de service utilisent des identités techniques distinctes, sans connexion interactive.

## 9. Critères d'acceptation

1. Un agent support ne peut pas consulter les documents KYC non masqués.
2. Un opérateur financier ne peut pas approuver sa propre demande.
3. Une permission retirée devient ineffective sur la requête suivante.
4. Une action hors de la portée pays ou entité est refusée.
5. Toute approbation sensible produit une preuve d'audit complète.
6. Aucun rôle applicatif ne permet la suppression des événements d'audit.
7. L'interface masque les actions interdites, mais le backend reste l'autorité finale.
8. Les tests automatisés couvrent les autorisations positives, négatives et les conflits de séparation des tâches.

## 10. Correspondance attendue dans `mansa-platform`

La plateforme doit exposer des constantes ou types partagés pour les rôles, permissions, portées et statuts d'approbation. Les gardes backend doivent vérifier permission et portée. Les modules administratifs doivent réutiliser ces contrats plutôt que définir leurs propres chaînes de caractères.
