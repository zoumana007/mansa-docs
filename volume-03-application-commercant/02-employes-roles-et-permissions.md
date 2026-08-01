# Application Commerçant — Employés, rôles et permissions

## 1. Objectif

Un profil commerçant peut déléguer des tâches sans partager le compte du propriétaire. Chaque employé possède une identité utilisateur propre, un rôle explicite, un état de cycle de vie et des permissions évaluées côté serveur.

## 2. Modèle de domaine

Le dépôt plateforme expose `MerchantStaffMember` dans `packages/domain/src/merchant-staff-member.ts`.

Champs principaux :

- `id` : identifiant interne du rattachement ;
- `merchantId` : profil commerçant concerné ;
- `userId` : utilisateur Mansa rattaché ;
- `role` : rôle métier courant ;
- `status` : état du rattachement ;
- `statusReason` : motif de suspension ou révocation ;
- `createdAt` : date de création.

Un rattachement ne contient aucun mot de passe, code PIN, jeton de session ou secret de terminal.

## 3. Rôles initiaux

| Rôle | Capacités principales |
|---|---|
| `owner` | Gestion du commerce, des employés, des paiements, remboursements et rapports |
| `manager` | Consultation des employés, encaissement, remboursement et rapports |
| `cashier` | Encaissement et consultation limitée des transactions |
| `support` | Consultation des transactions et assistance client |

Les permissions exactes sont définies dans le domaine et doivent ensuite être complétées par des règles de portée : point de vente, terminal, montant maximal, horaires et type d’opération.

## 4. Cycle de vie

| État | Description |
|---|---|
| `invited` | Invitation créée mais non acceptée |
| `active` | Employé autorisé à utiliser ses permissions |
| `suspended` | Accès temporairement bloqué avec motif |
| `revoked` | Rattachement supprimé définitivement avec motif |

Transitions autorisées :

1. `invited` vers `active` après acceptation et authentification.
2. `active` vers `suspended` avec motif obligatoire.
3. `suspended` vers `active` après réactivation.
4. Tout état non révoqué vers `revoked` avec motif obligatoire.

Un employé révoqué ne peut plus changer de rôle ni retrouver un accès. Une nouvelle invitation doit créer un nouveau rattachement auditable.

## 5. Règles de sécurité

- Les permissions sont refusées tant que le statut n’est pas `active`.
- Le contrôle côté interface est uniquement ergonomique ; l’API reste l’autorité.
- Le propriétaire ne doit jamais transmettre ses propres identifiants à un employé.
- Toute modification de rôle, suspension, réactivation ou révocation produit un événement d’audit.
- Les opérations sensibles peuvent demander une authentification renforcée.
- Les rôles et permissions doivent être configurables par pays et type de commerce sans réduire les contrôles minimaux du domaine.
- Le principe du moindre privilège s’applique à toute invitation.

## 6. Critères d’acceptation

- Un employé invité ne peut pas encaisser.
- Un caissier actif peut accepter un paiement mais ne peut pas créer un remboursement.
- Un responsable actif peut créer un remboursement selon les limites configurées.
- Un employé suspendu ou révoqué ne possède aucune permission effective.
- Un motif vide est refusé pour toute suspension ou révocation.
- Les identifiants de l’employé, du commerce et de l’utilisateur sont obligatoires.
- La sérialisation publique ne contient ni secret ni liste de permissions persistée.

## 7. Cohérence avec le code

L’agrégat est exporté par `packages/domain/src/index.ts` et couvert par `packages/domain/test/merchant-staff-member.test.mjs`. Le prochain lot du volume commerçant doit ajouter les points de vente et rattacher les permissions des employés à une ou plusieurs portées opérationnelles.
