# Matrice des rôles et permissions

## Objet

Cette matrice définit le modèle d’autorisation commun à la plateforme Mansa. Elle complète les contrats techniques `AuthorizationActor`, `AuthorizationResource` et `AuthorizationContext` du dépôt `mansa-platform`.

Les rôles fournissent un ensemble de permissions par défaut. L’autorisation finale tient également compte du périmètre de l’acteur, du pays, de l’organisation, du commerçant, du point de vente, de l’environnement, du niveau d’authentification, du montant et des obligations de double validation.

## Convention de nommage

Une permission suit le format `domaine.ressource.action`.

Exemples :

- `identity.user.read`
- `merchant.payment.refund`
- `administration.feature-flag.update`
- `public-service.fine.collect`
- `finance.reconciliation.approve`

Les permissions sont stables, versionnées et ne doivent jamais être construites à partir d’un libellé traduit.

## Types d’acteurs

| Type technique | Description | Périmètre minimal |
|---|---|---|
| `USER` | Client particulier | Propre identité et propres comptes |
| `MERCHANT_MEMBER` | Propriétaire ou employé d’un commerce | Commerçant et points de vente assignés |
| `PUBLIC_AGENT` | Agent d’un organisme public | Organisation, service et zone assignés |
| `ADMIN` | Collaborateur Mansa ou partenaire habilité | Portée explicite par rôle et attributs |
| `SERVICE` | Service technique ou worker | Compte de service, environnement et finalité |

## Rôles de référence

### Client

| Rôle | Permissions principales | Restrictions |
|---|---|---|
| `CLIENT` | Consulter profil, comptes, cartes, transactions ; créer transferts et paiements ; gérer bénéficiaires | Ressources propres uniquement ; limites KYC et risque |

### Commerçant

| Rôle | Permissions principales | Restrictions |
|---|---|---|
| `MERCHANT_OWNER` | Gérer commerce, membres, points de vente, TPE, paiements, remboursements, règlements et exports | Commerçant possédé ; authentification renforcée pour actions sensibles |
| `MERCHANT_MANAGER` | Gérer opérations, employés, catalogue et rapports | Pas de changement de propriétaire ni de coordonnées de règlement sans approbation |
| `MERCHANT_CASHIER` | Encaisser, consulter ses opérations, réimprimer un reçu | Point de vente et terminal assignés ; aucun export massif |
| `MERCHANT_ACCOUNTANT` | Consulter règlements, rapprochements, factures et exports financiers | Lecture financière ; aucun encaissement ni changement de configuration |
| `MERCHANT_SUPPORT` | Consulter état opérationnel et ouvrir des tickets | Données financières sensibles masquées |

### Services publics

| Rôle | Permissions principales | Restrictions |
|---|---|---|
| `PUBLIC_AGENT_COLLECTOR` | Rechercher une obligation, constater et collecter un paiement, émettre un reçu | Organisation, service et zone assignés ; aucune modification du barème |
| `PUBLIC_AGENT_SUPERVISOR` | Superviser agents, annuler selon procédure, consulter rapports | Double validation pour annulation et correction |
| `PUBLIC_ORG_ADMIN` | Gérer agents, services, barèmes, catalogues et habilitations | Organisation propre ; changements critiques approuvés |
| `SCHOLARSHIP_REVIEWER` | Examiner dossiers de bourse et proposer une décision | Aucun paiement direct ; séparation entre instruction et paiement |
| `STUDENT_CARD_OPERATOR` | Vérifier l’éligibilité et lancer l’émission d’une carte étudiante | Établissements assignés ; données minimales nécessaires |

### Administration Mansa

| Rôle | Permissions principales | Restrictions |
|---|---|---|
| `SUPPORT_AGENT` | Rechercher client, consulter état, gérer tickets, déclencher procédures autorisées | Données masquées ; aucune écriture financière directe |
| `SUPPORT_SUPERVISOR` | Superviser support, déverrouiller selon procédure, approuver certaines actions | Actions sensibles auditées et limitées |
| `KYC_REVIEWER` | Examiner documents et décider du niveau KYC | Accès KYC justifié, journalisé et limité au pays assigné |
| `COMPLIANCE_OFFICER` | Gérer alertes, restrictions et déclarations internes | Séparation avec les opérations commerciales |
| `RISK_ANALYST` | Consulter signaux de risque, créer règles et recommandations | Activation d’une règle bloquante soumise à approbation |
| `FINANCE_OPERATOR` | Rapprochement, règlements, écritures d’ajustement proposées | Ne peut pas approuver sa propre proposition |
| `FINANCE_APPROVER` | Approuver règlements et ajustements | Authentification renforcée et principe des quatre yeux |
| `PARTNER_MANAGER` | Gérer banques, opérateurs et paramètres non secrets | Aucun secret visible ; activation Production approuvée |
| `PRODUCT_ADMIN` | Gérer produits, limites, frais, contenu et fonctionnalités | Changements Production critiques soumis à double validation |
| `SECURITY_ADMIN` | Gérer politiques de sécurité, sessions, incidents et habilitations techniques | Aucun pouvoir financier métier par défaut |
| `AUDITOR` | Lecture des audits, configurations, décisions et preuves | Lecture seule ; exports tracés et filtrés |
| `COUNTRY_ADMIN` | Administrer un pays dans les limites déléguées | Aucun accès aux autres pays |
| `SUPER_ADMIN` | Administrer la plateforme et déléguer des rôles | Usage exceptionnel ; authentification matérielle ; double validation ; accès temporaire recommandé |

### Services techniques

| Rôle | Permissions principales | Restrictions |
|---|---|---|
| `SERVICE_API_GATEWAY` | Valider identité, autoriser requêtes et orchestrer appels | Aucun accès interactif ; environnement fixé |
| `SERVICE_WORKER` | Consommer événements et exécuter tâches ciblées | Permission par file et type de tâche |
| `SERVICE_NOTIFICATION` | Envoyer notifications demandées | Aucun accès au solde complet ni aux secrets d’authentification |
| `SERVICE_RECONCILIATION` | Lire flux financiers et produire rapprochements | Ne peut pas approuver les écarts |
| `SERVICE_AI` | Traiter les données minimales autorisées pour Jini, fraude ou support | Finalité, rétention et champs explicitement limités |

## Catalogue minimal des permissions

### Identité et conformité

| Permission | Description | Niveau minimal |
|---|---|---|
| `identity.profile.read-own` | Lire son propre profil | Facteur principal |
| `identity.profile.update-own` | Modifier son propre profil | Facteur principal |
| `identity.session.revoke-own` | Révoquer une de ses sessions | Facteur principal |
| `identity.user.read` | Consulter une identité selon périmètre | Multi-facteur pour administration |
| `identity.user.restrict` | Restreindre un compte | Multi-facteur + justification |
| `kyc.case.read` | Consulter un dossier KYC | Multi-facteur + motif audité |
| `kyc.case.review` | Examiner un dossier KYC | Multi-facteur |
| `kyc.case.decide` | Décider du statut KYC | Multi-facteur ; séparation configurable |

### Portefeuille, paiement et cartes

| Permission | Description | Niveau minimal |
|---|---|---|
| `wallet.balance.read-own` | Lire ses soldes | Facteur principal |
| `payment.create-own` | Initier un paiement | Facteur principal ou renforcé selon risque |
| `transfer.create-own` | Initier un transfert | Facteur principal ou renforcé selon montant |
| `beneficiary.manage-own` | Gérer ses bénéficiaires | Renforcement pour nouveau bénéficiaire |
| `card.manage-own` | Gérer statut, limites et contrôles de ses cartes | Multi-facteur pour actions sensibles |
| `merchant.payment.collect` | Encaisser un paiement | Terminal ou session autorisée |
| `merchant.payment.refund` | Initier un remboursement | Multi-facteur selon montant |
| `merchant.settlement.read` | Consulter les règlements | Facteur principal |
| `finance.adjustment.propose` | Proposer un ajustement | Multi-facteur |
| `finance.adjustment.approve` | Approuver un ajustement | Multi-facteur ; auteur différent |
| `finance.reconciliation.approve` | Valider un rapprochement | Multi-facteur ; auteur différent |

### Administration et configuration

| Permission | Description | Niveau minimal |
|---|---|---|
| `administration.role.read` | Lire rôles et permissions | Multi-facteur |
| `administration.role.assign` | Affecter ou retirer un rôle | Multi-facteur ; approbation selon portée |
| `administration.feature-flag.update` | Modifier un drapeau de fonctionnalité | Multi-facteur ; approbation en Production |
| `administration.fee.update` | Modifier frais et commissions | Multi-facteur ; double validation en Production |
| `administration.limit.update` | Modifier limites produit ou risque | Multi-facteur ; double validation en Production |
| `administration.partner.activate` | Activer une intégration partenaire | Multi-facteur ; validation technique et métier |
| `audit.event.read` | Consulter le journal d’audit | Multi-facteur ; périmètre limité |
| `audit.export.create` | Exporter des événements d’audit | Multi-facteur ; export tracé |

### Services publics

| Permission | Description | Niveau minimal |
|---|---|---|
| `public-service.obligation.read` | Rechercher une obligation | Facteur principal sur appareil autorisé |
| `public-service.fine.create` | Constater une amende | Appareil et agent identifiés |
| `public-service.payment.collect` | Collecter le paiement d’une obligation | Appareil autorisé ; reçu obligatoire |
| `public-service.payment.cancel` | Annuler ou corriger une collecte | Multi-facteur ; double validation |
| `public-service.catalog.update` | Modifier services, barèmes ou règles | Multi-facteur ; double validation en Production |
| `public-service.scholarship.review` | Examiner une bourse | Multi-facteur |
| `public-service.scholarship.decide` | Décider une bourse | Multi-facteur ; séparation avec paiement |
| `public-service.student-card.issue` | Émettre une carte étudiante | Multi-facteur ; établissement assigné |

## Règles ABAC obligatoires

Une permission seule ne suffit pas. La décision vérifie au minimum :

1. le statut actif de l’acteur et de sa session ;
2. le pays et l’environnement autorisés ;
3. l’organisation, le commerçant, le point de vente ou l’établissement assigné ;
4. le niveau d’authentification requis ;
5. les limites de montant et de fréquence ;
6. le statut KYC, risque ou conformité applicable ;
7. les horaires, appareils ou réseaux autorisés lorsque configurés ;
8. l’absence de conflit de séparation des tâches ;
9. la présence d’une approbation encore valide ;
10. la finalité déclarée pour l’accès aux données sensibles.

## Actions nécessitant une double validation

La double validation est obligatoire en Production pour :

- modification des frais, limites globales ou règles bloquantes ;
- activation ou désactivation d’un partenaire financier ;
- ajustement financier ou règlement exceptionnel ;
- annulation d’une collecte publique après finalisation ;
- attribution d’un rôle à forte portée, notamment `SUPER_ADMIN` ;
- export massif de données sensibles ;
- changement de politique de sécurité critique ;
- bascule d’une fonctionnalité financière à 100 % des utilisateurs.

L’auteur d’une demande ne peut jamais l’approuver lui-même.

## Journalisation obligatoire

Toute décision d’autorisation conserve :

- identifiant de corrélation ;
- acteur, type d’acteur et rôles actifs ;
- permission et ressource demandées ;
- périmètre appliqué ;
- niveau d’authentification ;
- décision et code de raison ;
- politiques évaluées ;
- obligations imposées ;
- date, environnement et pays ;
- approbation associée lorsqu’elle existe.

Les refus sur actions sensibles sont audités sans enregistrer de secret, jeton ou donnée de paiement complète.

## Critères d’acceptation

- Une route protégée refuse par défaut toute permission inconnue.
- Un rôle retiré cesse d’accorder ses permissions sans attendre une reconnexion complète.
- Un acteur ne peut accéder à une ressource hors de son périmètre même s’il possède le nom de permission.
- Les actions Production critiques exigent le niveau d’authentification et les approbations configurés.
- Les comptes de service ne peuvent pas être utilisés comme comptes interactifs.
- Les exports appliquent masquage, filtrage et journalisation.
- Chaque permission du code est référencée dans un catalogue versionné.
- Les scénarios `REC-AUTHZ-001` à `REC-AUTHZ-004` de la matrice transverse sont automatisés ou accompagnés d’une preuve de recette.
