# Profils des rôles de référence

## Objet

Ce document transforme la matrice fonctionnelle des rôles en profils techniques minimaux. Le dépôt `mansa-platform` doit exposer ces profils sous forme de contrats versionnés afin que l’API, les applications, les tests et l’administration partagent la même base.

Un profil de rôle n’accorde jamais à lui seul un accès définitif. La décision finale combine le profil, le périmètre affecté, les attributs de l’acteur, le niveau d’authentification, l’environnement, le pays, les limites et les éventuelles approbations.

## Règles de construction

- Chaque rôle de référence possède exactement un profil.
- Chaque permission d’un profil appartient au catalogue officiel.
- Chaque type de périmètre appartient au catalogue `ROLE_SCOPE_TYPES`.
- Un rôle interactif et un rôle de service ne sont jamais interchangeables.
- Un rôle système ne peut pas être créé, supprimé ou renommé depuis l’administration.
- Un rôle à forte portée est affecté pour une durée limitée lorsque cela est possible.
- Les permissions non listées sont refusées par défaut.

## Profils particuliers

### Utilisateurs et commerçants

- `CLIENT` : profil propre à l’utilisateur, limité à ses ressources.
- `MERCHANT_OWNER` : gestion du commerce, encaissement, remboursements et consultation des règlements.
- `MERCHANT_MANAGER` : opérations du commerce sans pouvoirs réservés au propriétaire.
- `MERCHANT_CASHIER` : encaissement sur les emplacements et terminaux affectés.
- `MERCHANT_ACCOUNTANT` : lecture des règlements et informations financières autorisées.
- `MERCHANT_SUPPORT` : consultation opérationnelle sans pouvoir financier.

### Services publics

- `PUBLIC_AGENT_COLLECTOR` : recherche d’obligations, constatation et collecte.
- `PUBLIC_AGENT_SUPERVISOR` : supervision et demandes de correction selon procédure.
- `PUBLIC_ORG_ADMIN` : administration de l’organisme public et de son catalogue.
- `SCHOLARSHIP_REVIEWER` : instruction et décision de dossiers de bourse.
- `STUDENT_CARD_OPERATOR` : émission de cartes pour les établissements affectés.

### Administration Mansa

Les profils `SUPPORT_AGENT`, `SUPPORT_SUPERVISOR`, `KYC_REVIEWER`, `COMPLIANCE_OFFICER`, `RISK_ANALYST`, `FINANCE_OPERATOR`, `FINANCE_APPROVER`, `PARTNER_MANAGER`, `PRODUCT_ADMIN`, `SECURITY_ADMIN`, `AUDITOR`, `COUNTRY_ADMIN` et `SUPER_ADMIN` sont limités à leur fonction et à leur portée explicite.

`SUPER_ADMIN` ne signifie pas contournement des contrôles. Les opérations marquées comme sensibles conservent les exigences d’authentification renforcée, d’approbation et de journalisation.

### Comptes de service

Les profils `SERVICE_API_GATEWAY`, `SERVICE_WORKER`, `SERVICE_NOTIFICATION`, `SERVICE_RECONCILIATION` et `SERVICE_AI` sont réservés aux identités non interactives. Leur accès réel doit être réduit par service, environnement, file, sujet d’événement et finalité.

## Périmètres autorisés

| Famille | Périmètres usuels |
|---|---|
| Client | `PLATFORM` avec filtrage sur les ressources propres |
| Commerçant | `MERCHANT`, `LOCATION` |
| Agent public | `PUBLIC_ORGANIZATION`, éventuellement `COUNTRY` |
| Administration pays | `COUNTRY`, `ORGANIZATION` |
| Administration centrale | `PLATFORM`, `COUNTRY`, `ORGANIZATION` |
| Service technique | `PLATFORM` ou périmètre technique explicitement réduit |

## Séparation des tâches

- `FINANCE_OPERATOR` propose ; `FINANCE_APPROVER` approuve.
- Un auteur ne valide jamais sa propre demande.
- Un agent de bourse ne déclenche pas directement le paiement qu’il a décidé.
- Un opérateur de collecte publique ne corrige pas seul une collecte finalisée.
- Un administrateur produit ne reçoit pas automatiquement les pouvoirs de sécurité ou de finance.
- Un compte de service ne reçoit pas de rôle humain.

## Alignement avec le code

La source technique de référence est `packages/contracts/src/role-profiles.ts` dans `mansa-platform`. Elle doit :

1. couvrir tous les codes de `REFERENCE_ROLES` ;
2. référencer uniquement des valeurs de `PERMISSIONS` ;
3. exposer un accès par code de rôle ;
4. distinguer les rôles système des rôles métier ;
5. indiquer les types d’acteurs et périmètres acceptés ;
6. être couverte par des tests automatisés.

## Critères d’acceptation

- Aucun rôle de référence n’est absent du registre technique.
- Aucun profil ne contient une permission inconnue.
- Les profils finance conservent la séparation proposition/approbation.
- Les rôles de service sont marqués comme gérés par le système.
- Les profils `CLIENT`, commerçant et agent public ne peuvent pas être affectés à un type d’acteur incompatible.
- L’ajout d’un rôle ou d’une permission sans mise à jour cohérente fait échouer la validation automatisée.
